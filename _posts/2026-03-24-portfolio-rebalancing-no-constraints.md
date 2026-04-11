---
layout: post
title: "260324 포트폴리오 리밸런싱 무제약 설계"
comments: true
tags:
- portfolio
- rebalancing
- no
- constraints
---

# 260324 포트폴리오 리밸런싱 무제약 설계

안녕하세요 👋  
이번 글은 **최근 7일, 3000+ 문서(HsStockDoc + sentiment_list)**를 입력으로 받아,  
**pydantic.ai + gpt-4o-mini** 기반으로 포트폴리오 비중을 계산하는 실전 설계안입니다.

핵심 요구는 명확합니다.

- 문서 소스: `kr(뉴스/증권뉴스/DART/리서치)`, `us(뉴스/SEC/레딧)`
- 문서별 `sentiment_list` 사전 계산 완료
- 출력: 자산별 비중 (예: 삼성전자/MSFT/금/현금/채권)
- 추가 요청: **리밸런싱 제약(하한/상한/현금최소/턴오버 제한) 제거**

---

## 1) 결론: 결정론 코어 + AI 보정 (하이브리드) ✅

완전 AI 직출력(문서 -> 바로 비중)은 재현성과 감사를 통과하기 어렵습니다.  
반대로 완전 결정론은 레짐 전환 대응이 둔합니다.

그래서 아래 구조를 권장합니다.

1. **결정론 코어**: 문서 신호를 수치화해 자산 점수 산출
2. **AI 보정**: 레짐/이벤트를 반영해 가중치 벡터를 조정
3. **무제약 변환**: 점수 -> 비중 변환 (하드 제약 없이)

즉, AI는 "판단 보조", 수치계산은 "일관 엔진"이 담당합니다. 🔧

---

## 2) 가중치 우선순위와 배율 (초기 권장)

요구하신 우선순위를 실제 운영 관점으로 정리하면 다음이 안정적입니다.

### 우선순위

$$
\text{신뢰성} > \text{최신성} > \text{감성강도} > \text{인기} > \text{문서수}
$$

이유:

- 신뢰성: 신호의 품질 자체를 결정
- 최신성: 7일 윈도우에서 시장 반응속도 반영
- 감성강도: 방향성 직접 반영
- 인기/문서수: 조작/바이어스 가능성이 높아 보조 신호

### 초기 배율 (베이스라인)

문서 단위 점수:

$$
S_d = 0.45R_d + 0.25T_d + 0.15E_d + 0.10P_d + 0.05V_d
$$

- $R_d$ (신뢰성): 소스 등급 점수
  - 공시(SEC/DART) `1.00`
  - 리서치 `0.70`
  - 뉴스 `0.40`
  - 커뮤니티(레딧) `0.20`
- $T_d$ (최신성): 시간감쇠
  - $T_d = \exp(-\Delta h/\tau_{source})$
  - 예: 공시 72h, 리서치 120h, 뉴스 48h, 레딧 24h
- $E_d$ (감성강도): `sentiment_list` 집계값(정규화)
- $P_d$ (인기): `log1p(likes + 0.5*comments)` 후 소스별 표준화
- $V_d$ (문서수): 자산/소스 버킷의 `log1p(count)`

> 포인트: 문서수/인기는 반드시 포화함수(log/sqrt)로 눌러야 왜곡이 줄어듭니다.

---

## 3) "무제약" 리밸런싱에서의 비중 계산

요청대로 하드 제약은 제거합니다.  
다만 포트폴리오로 쓰려면 최소한 **정규화 방식**은 정해야 합니다.

### A안: Long-only 무제약 정규화 (가장 실무적)

자산별 최종 알파 $\alpha_i$가 있을 때:

$$
w_i = \frac{\exp(\beta \alpha_i)}{\sum_j \exp(\beta \alpha_j)}
$$

- 장점: 자동으로 $w_i \ge 0$, 합 1
- 의미: 하드 상/하한 없이도 "비중"이라는 출력 형식을 자연스럽게 만족
- 추천: 기본 운영안

### B안: 완전 무제약(롱숏/레버리지 허용)

$$
w_i = \frac{\alpha_i}{\sum_j |\alpha_j|}
$$

- 장점: 음수 비중 가능(숏 시그널 반영)
- 주의: 실제 계좌 반영 시 별도 레버리지/마진 정책 필요

---

## 4) pydantic.ai 아키텍처 설계

3000개 문서를 LLM에 한 번에 넣지 않고, **Map-Reduce 피처화** 후 에이전트에 전달합니다.

### 단계별 파이프라인

1. **Ingest/Feature Builder (결정론)**
   - 문서 -> 자산 매핑
   - 소스 신뢰성, 최신성, 인기, sentiment 피처 계산
   - 자산별 요약 피처 생성

2. **RegimeAgent (gpt-4o-mini)**
   - 입력: 시장 요약 피처
   - 출력: `risk_on/neutral/risk_off`, confidence, 근거

3. **WeightTunerAgent (gpt-4o-mini)**
   - 입력: 기본 가중치 + 레짐
   - 출력: 조정 가중치(합 1)

4. **Allocator (결정론)**
   - 문서 점수 집계 -> 자산 알파
   - 알파 -> 비중(softmax 또는 sign-normalization)

5. **SanityAgent (gpt-4o-mini)**
   - 출력 비중/근거 텍스트 검토
   - 이상치 플래그만 제공(최종 값은 결정론 결과 우선)

---

## 5) Pydantic 모델 스키마 예시

```python
from typing import Literal
from pydantic import BaseModel, Field, confloat


class RegimeOut(BaseModel):
    regime: Literal["risk_on", "neutral", "risk_off"]
    confidence: confloat(ge=0.0, le=1.0)
    reason: str = Field(min_length=1, max_length=400)


class WeightOut(BaseModel):
    reliability: confloat(ge=0.0, le=1.0)
    recency: confloat(ge=0.0, le=1.0)
    sentiment: confloat(ge=0.0, le=1.0)
    popularity: confloat(ge=0.0, le=1.0)
    volume: confloat(ge=0.0, le=1.0)
    # 런타임에서 합=1 검증


class AllocationItem(BaseModel):
    asset: str
    weight: float


class AllocationOut(BaseModel):
    mode: Literal["long_only_softmax", "unconstrained_signed"]
    items: list[AllocationItem]
    explanation: str
```

핵심 운영 규칙:

- LLM 출력은 반드시 Pydantic 검증 통과
- 실패 시 재시도 1회 후 베이스라인 가중치 폴백
- `temperature=0.0~0.2`로 낮춰 일관성 확보

---

## 6) pydantic.ai 연결 예시 (개념 코드)

```python
from dataclasses import dataclass
from pydantic_ai import Agent, RunContext


@dataclass
class Deps:
    feature_snapshot: dict


regime_agent = Agent(
    "openai:gpt-4o-mini",
    deps_type=Deps,
    output_type=RegimeOut,
    instructions=(
        "Classify current market regime from summarized cross-asset features. "
        "Return concise, evidence-based output only."
    ),
)


weight_agent = Agent(
    "openai:gpt-4o-mini",
    deps_type=Deps,
    output_type=WeightOut,
    instructions=(
        "Tune weights for reliability/recency/sentiment/popularity/volume. "
        "Weights must sum to 1 and reflect provided regime."
    ),
)
```

---

## 7) 품질검증(백테스트) 프레임

무제약 설계에서는 과최적화 위험이 커집니다. 아래 4개는 필수입니다. 📊

1. **워크포워드 백테스트**: 시점 누수 차단
2. **거래비용/슬리피지 포함**: 실전 성능 근사
3. **샤프 + MDD + 턴오버 동시 평가**
4. **A/B 테스트**: 결정론 단독 vs 하이브리드

권장 채택 기준 예시:

- OOS 샤프 +10% 이상 개선
- MDD 악화가 5%p 이내
- 턴오버 급등(예: +30% 초과) 없을 것

---

## 8) 최종 제안

- 현재 데이터 규모(7일 3000+)에서는 **결정론+AI 보정**이 가장 현실적
- 가중치는 `신뢰성 > 최신성 > 감성 > 인기 > 문서수`로 시작
- 리밸런싱 제약 제거 요청은 반영하되, 출력 형식은 `softmax` 또는 `signed` 모드 중 선택
- pydantic.ai는 "구조화 출력 강제 + 재시도 + 폴백" 중심으로 설계

---

## 참고 URL

- PydanticAI 공식 문서: https://ai.pydantic.dev/
- PydanticAI 에이전트/출력 타입: https://ai.pydantic.dev/agent/
- OpenAI Structured Outputs 가이드: https://platform.openai.com/docs/guides/structured-outputs
- OpenAI GPT-4o mini 모델 문서: https://platform.openai.com/docs/models/gpt-4o-mini
- JSON Schema 공식 사이트: https://json-schema.org/

---

```text
hhd-research
hhd-md

주제 : 포트폴리오 리벨런싱 로직 설계

사용기술
- pydantic.ai
- gpt-4o-mini

요구사항
- 이제 HsStockDoc 이 최근7일간 기준으로도 3000개 이상 수집 되었음.
  - kr(뉴스, 증권뉴스, dart공시, 리서치), us(뉴스, sec공시, 레딧)
- 그리고 각 문서별로 sentiment_list 도 미리 계산되어 있는 상황
- 이때 이 문서들과 sentiment_list 를 모두 읽고, 포트폴리오를 리밸런싱 할려고 함
- 예를들면 포트폴리오로 삼성전자, MSFT, 금, 현금, 채권 이렇게 5종목 있다고 한다면
  - 삼성전자:0.3 MSFT:0.2 금:0.3 현금:0.1 채권:0.1
  - 이런식으로 비율을 계산하면 됨.
- 즉 이 3000여개의 문서와 그에 따른 senment_list 를 입력으로
  - 이 포트폴리오 결과 계산을 출력으로 해주는 솔루션을 만들려는 것임.
- 그런데 이 솔루션에서 pydantic.ai와 gpt api 를 사용해서 비결정적인 계산을 수행할려고 함.
- 각 정보는 아래와 같은 가중치를 기본으로 우선순위가 정해져야 함.
  - 문서의 신뢰성
    - 공시 > 리서치 > 뉴스
  - 문서의 인기
    - 좋아요수, 댓글수
  - 문서의 갯수
  - 문서의 최신성
- 가중치가 정말 중요할텐데 가중치의 순서부터 가중치의 배율까지 어떻게 하면 좋을까요?
  - 결정론적 로직이 나을지? 인공지능 로직이 나을지?
  - 가중치 우선순위 조차 의견을 주세요.
- 결정론 로직으로 수행하는 것보다 인공지능 로직으로 하는게 낫다면
  - pydantic.ai 로 어떤 설계로 구성하는게 좋을지?

리벨런싱 제약은 없애주세요
문서로 만들어 주세요
```
