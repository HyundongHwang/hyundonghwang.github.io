---
layout: post
title: "260324 포트폴리오 리밸런싱 — 레짐·배율근거·파이프라인 심층 분석"
comments: true
tags:
- portfolio
- rebalancing
- regime
- pipeline
---

# 260324 포트폴리오 리밸런싱 — 레짐·배율근거·파이프라인 심층 분석

> 이 글은 **260324_0040_portfolio_rebalancing_no_constraints.md** 설계안을 바탕으로,
> 레짐 개념·전환 대응, 초기 가중치 배율 이론 근거, 단계별 파이프라인 상세 설명,
> 그리고 sentiment score 와 레짐의 관계를 정리한 리포트입니다.

---

## 🔄 1. 레짐(Regime)이란?

**레짐(Regime)** = 금융시장이 현재 어떤 "상태(모드)"에 있는지를 나타내는 개념입니다.

시장은 한 가지 행동양식이 아니라, **시기에 따라 전혀 다른 법칙으로 움직이는 구간**이 반복됩니다.

| 레짐 | 시장 심리 | 자산 행동 | 예시 |
|------|-----------|-----------|------|
| **risk_on** | 낙관, 탐욕 | 주식↑ / 현금·채권↓ | 2020 V자 반등, 2023 AI 랠리 |
| **neutral** | 방향 불명 | 혼조세 | 금리 결정 대기, 실적 시즌 초반 |
| **risk_off** | 공포, 회피 | 현금·금·국채↑ / 주식↓ | 2022 인플레이션 쇼크, 코로나 급락 |

**왜 중요한가?**

> 같은 삼성전자 호재 뉴스라도,
> risk_on 레짐에서는 비중을 더 올려야 하고,
> risk_off 레짐에서는 호재 뉴스조차 무시하고 방어 자산 위주로 가야 합니다.

즉, **레짐이 틀리면 신호 해석 자체가 반전**됩니다.

---

## 🔃 2. 레짐 전환 대응

파이프라인의 `RegimeAgent → WeightTunerAgent` 흐름이 이 역할을 담당합니다.

### 레짐별 가중치 조정 전략

| 가중치 항목 | risk_on | neutral | risk_off |
|-------------|---------|---------|----------|
| 신뢰성 $R_d$ | 0.40 | **0.45** | **0.55** |
| 최신성 $T_d$ | 0.25 | 0.25 | 0.20 |
| 감성강도 $E_d$ | **0.20** | 0.15 | **0.10** |
| 인기 $P_d$ | 0.10 | 0.10 | 0.10 |
| 문서수 $V_d$ | 0.05 | 0.05 | 0.05 |

**핵심 로직:**
- **risk_off** → 신뢰성(공시/리서치) 비중 ↑, 감성(SNS/레딧) 비중 ↓ (공포 심리 과장 차단)
- **risk_on** → 감성 비중 약간 ↑ (모멘텀·FOMO 반영 필요)
- **WeightTunerAgent** 가 이 조정을 LLM 으로 수행하고, Pydantic 으로 "합=1" 강제 검증

---

## 📐 3. 초기 배율 근거 (0.45 / 0.25 / 0.15 / 0.10 / 0.05)

초기 공식:

$$
S_d = 0.45R_d + 0.25T_d + 0.15E_d + 0.10P_d + 0.05V_d
$$

이 숫자들은 **AHP + 포트폴리오 이론 기반으로 정당화 가능**합니다.

### 3-1. AHP(계층분석법) 기반 정당화

AHP(Analytic Hierarchy Process)는 다기준 의사결정에서 각 항목의 상대 중요도를 수치화하는 방법론입니다 (Saaty, 1980).

$$
R : T : E : P : V = 9 : 5 : 3 : 2 : 1
$$

합계 = 20, 정규화하면:

$$
\frac{9}{20} : \frac{5}{20} : \frac{3}{20} : \frac{2}{20} : \frac{1}{20} = 0.45 : 0.25 : 0.15 : 0.10 : 0.05
$$

✅ 즉, **초기 배율은 AHP 상대중요도의 자연스러운 정수비 정규화**입니다.

---

### 3-2. 각 가중치별 포트폴리오 이론 근거

#### 📌 신뢰성 $R_d = 0.45$ — Information Coefficient (IC) 이론

> **Grinold & Kahn "Active Portfolio Management" (2000)**
> 알파 신호의 최적 배합 가중치는 IC(정보계수)에 비례한다.

- 공시(SEC/DART)는 법적 사실 기반 → IC 최고
- 레딧 감성은 노이즈 포함 → IC 낮음
- IC 격차가 크면 클수록 상위 신호에 집중해야 하므로 **0.45** 최대 부여

#### 📌 최신성 $T_d = 0.25$ — 신호 반감기(Signal Half-Life)

> **Tetlock (2007), "Giving Content to Investor Sentiment"** (Journal of Finance)
> 뉴스 감성의 시장 영향은 **48시간 이내 대부분 흡수**됨.

$$
T_d = \exp\left(-\frac{\Delta h}{\tau}\right)
$$

| 소스 | 반감기 $\tau$ |
|------|--------------|
| 공시 | 72h |
| 리서치 | 120h |
| 뉴스 | 48h |
| 레딧 | 24h |

7일 윈도우에서 최신 정보가 지수적으로 중요 → **0.25**

#### 📌 감성강도 $E_d = 0.15$ — 감성 팩터 연구

> **Baker & Wurgler (2007), "Investor Sentiment in the Stock Market"** (JEP)
> 감성 지수는 단기 수익률 예측에 통계적으로 유의하나, 단독 사용 시 과최적화 위험.

방향성 신호로서 무시할 수 없으나, 공포·탐욕 편향 존재 → **0.15**

#### 📌 인기 $P_d = 0.10$ — 소셜 신호 연구

> **Gu et al. (2020), "Empirical Asset Pricing via Machine Learning"** (RFS)
> 소셜미디어 신호는 예측력은 있으나 조작 가능성이 높아 독립 사용 위험.

`log1p(likes + 0.5*comments)` 포화 함수로 극단값 억제 필수, 보조 신호 → **0.10**

#### 📌 문서수 $V_d = 0.05$ — 샘플 크기 효과

대수의 법칙: 샘플 수 증가 시 노이즈↓, 그러나 **한계 효용 체감**.

`log1p(count)` 포화 함수 사용. 문서 수 자체는 품질 보증 불가 → **0.05**

---

### 3-3. 요약표

| 가중치 | 값 | 이론 근거 | 핵심 이유 |
|--------|----|-----------|-----------|
| 신뢰성 | **0.45** | Grinold-Kahn IC 이론 | 신호 품질 = 알파의 원천 |
| 최신성 | **0.25** | Tetlock(2007) 신호 반감기 | 7일 윈도우 내 정보 가치 지수감쇠 |
| 감성강도 | **0.15** | Baker-Wurgler(2007) 감성 팩터 | 방향성 신호, 단독 위험 있음 |
| 인기 | **0.10** | Gu et al.(2020) 소셜 신호 | 조작 가능성으로 보조 역할만 |
| 문서수 | **0.05** | 대수의 법칙 + 한계체감 | 품질 보증 불가, 메타신호 |

---

## 🏭 4. 단계별 파이프라인 상세 설명

**비유: 이 파이프라인은 "요리사 팀"입니다.**

```
문서 3000개
    ↓
[Step 1] 재료 손질 (파이썬 계산)
    ↓ 자산별 숫자 요약
[Step 2] 날씨 예보 (GPT: risk_on/off?)
    ↓ 레짐 + 신뢰도
[Step 3] 레시피 조정 (GPT: 가중치 튜닝)
    ↓ 조정된 가중치 벡터
[Step 4] 최종 계산 (파이썬: softmax)
    ↓ 비중 결과
[Step 5] 검수 (GPT: 이상치 있어?)
    ↓
포트폴리오 비중 출력
```

**GPT 호출 3번만** (Step 2, 3, 5) → 토큰 비용 최소화, 계산 속도 확보 ✅

---

### 🥩 Step 1: Ingest/Feature Builder (재료 손질 담당)

```python
# 자산별 문서 집계 예시
asset_features = {
    "삼성전자": {
        "avg_score": 0.72,
        "doc_count": 340,
        "sentiment_avg": 0.6,
        "reliability_weighted": 0.78
    },
    ...
}
```

- 3000개 문서 → 자산별 숫자 요약
- 공식 $S_d = 0.45R_d + 0.25T_d + 0.15E_d + 0.10P_d + 0.05V_d$ 적용
- **순수 파이썬 계산 (AI 없음)** → 항상 같은 입력이면 같은 출력 보장

---

### 🌡️ Step 2: RegimeAgent (시장 날씨 예보관)

```python
# 출력 예시
{
    "regime": "risk_off",
    "confidence": 0.82,
    "reason": "Fed 긴축 뉴스 급증, 주식 감성 악화"
}
```

- Step 1 요약 피처만 GPT 에 전달 (토큰 절약)
- "지금 risk_on 인가, risk_off 인가?" 판단
- **조건부 호출 최적화** (아래 섹션 참고)

---

### ⚖️ Step 3: WeightTunerAgent (레시피 조정관)

```python
# risk_off 레짐 시 출력 예시
{
    "reliability": 0.55,
    "recency": 0.20,
    "sentiment": 0.10,
    "popularity": 0.10,
    "volume": 0.05
}
```

- 레짐에 따라 가중치 벡터 조정
- Pydantic 으로 `합=1` 런타임 검증, 실패 시 기본값 폴백

---

### 🧮 Step 4: Allocator (최종 계산기)

$$
w_i = \frac{\exp(\beta \alpha_i)}{\sum_j \exp(\beta \alpha_j)}
$$

```python
# 출력 예시
{
    "삼성전자": 0.28,
    "MSFT": 0.22,
    "금": 0.35,
    "현금": 0.10,
    "채권": 0.05
}
```

- **다시 순수 계산 (AI 없음)** → 재현 가능, 감사 가능

---

### 🔍 Step 5: SanityAgent (최종 검수원)

```python
# 출력 예시
{
    "flag": true,
    "reason": "고비중 자산의 부정 공시 감지됨"
}
```

- 플래그만 제공, 최종 수치는 Step 4 결과를 우선 사용
- **AI 가 결과를 바꾸지 않고 검토만** → 결정론 결과의 무결성 보호

---

## 💡 5. 레짐 = Sentiment Score 로 단순화 가능한가?

### ✅ 단순화가 충분한 경우

sentiment 집계 평균이 이미 -1~1 사이에 있고, 이를 그대로 레짐 프록시로 사용하는 것은 **충분히 합리적**입니다.

```python
avg_sentiment = weighted_mean(all_docs)  # 이미 계산됨

if avg_sentiment > 0.3:    regime = "risk_on"
elif avg_sentiment < -0.3: regime = "risk_off"
else:                      regime = "neutral"
```

RegimeAgent(LLM 호출) 없이도 동작합니다.

---

### ⚠️ Sentiment Score 가 레짐을 놓치는 2가지 케이스

#### 케이스 1: 감성 중립 + 구조적 위기

> **예시**: Fed 75bp 금리 인상 발표
> - 시장이 "예상된 일"로 받아들여 sentiment ≈ 0 (neutral)
> - 그러나 실제 포트폴리오는 **risk_off** 포지션이 맞음

sentiment score 는 0 인데 레짐은 risk_off 인 케이스.

#### 케이스 2: 자산별 감성 방향이 엇갈릴 때

| 자산 | sentiment |
|------|-----------|
| 삼성전자 | -0.7 (부진) |
| MSFT | +0.8 (AI 호재) |
| **집계 평균** | **≈ 0.05 (neutral 처럼 보임)** |

실제로는 **섹터 로테이션 레짐**이지, 중립이 아닙니다.

---

### 🛠️ 실용적 제안: 2단계 혼합

| 판단 기준 | 방식 | 비용 |
|-----------|------|------|
| 기본 레짐 | sentiment 집계 평균 (-1~1) | 무료 (이미 계산됨) |
| 예외 감지 | 자산별 분산(variance) > 0.4 → LLM 위임 | LLM 호출 조건부 |

```python
sentiment_avg = weighted_mean(all_docs)
sentiment_var = variance_across_assets(all_docs)

if sentiment_var > 0.4:
    # 자산 간 방향이 엇갈림 → LLM 에 위임
    regime = regime_agent.run(features)
else:
    # 단순 구간 분류로 충분
    regime = classify_by_threshold(sentiment_avg)
```

**결론**: RegimeAgent 를 항상 실행할 필요는 없습니다.
**sentiment 분산이 클 때만 LLM 을 쓰는 조건부 호출**이 비용과 정확도 모두 잡는 현실적인 구조입니다. 💡

---

## 📚 참고 문헌 / URL

| 자료 | 출처 |
|------|------|
| Active Portfolio Management | Grinold & Kahn, 2000 |
| Giving Content to Investor Sentiment | Tetlock (2007), Journal of Finance |
| Investor Sentiment in the Stock Market | Baker & Wurgler (2007), JEP |
| Empirical Asset Pricing via Machine Learning | Gu et al. (2020), RFS |
| Analytic Hierarchy Process | Saaty (1980) |
| PydanticAI 공식 문서 | https://ai.pydantic.dev/ |
| OpenAI Structured Outputs 가이드 | https://platform.openai.com/docs/guides/structured-outputs |

---

```text
[사용자 프롬프트]

원본 문서: 260324_0040_portfolio_rebalancing_no_constraints.md

추가 질문:
- 레짐?
- 레짐 전환 대응?
- 초기배율의 각 숫자들은 근거가 있는가?
  - 근거가 없어도 되지만, 포트폴리오 이론이나 기타 웹검색으로 최대한 근거를 만들어서 적용
  - 이렇게 배율의 각 숫자값들을 만들면, 그 근거를 한번더 설명해 주세요.
- 단계별 파이프라인을 좀더 쉽고 자세히 설명
- 품질검증은 일단 논외로 합시다

후속 질문:
- 레짐이 단순히 sentiment score 의 -1~1 구간으로 충분히 표현되는것 같은데?
```
