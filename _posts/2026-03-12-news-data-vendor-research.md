---
layout: post
title: "260312 뉴스 데이터 공급사 비교와 라이선스 리서치"
comments: true
tags:
- news
- data
- vendor
---

# 260312 뉴스 데이터 공급사 비교와 라이선스 리서치

> 검증 기준일: 2026-03-12 (KST)  
> 성격: 제품/데이터 아키텍처 리서치이며, 법률 자문은 아님

## 한눈에 결론 🚦

- 완전 자작은 가능합니다. 다만 MVP와 운영을 실제로 흔드는 변수는 `LLM 토큰비`보다 `뉴스 라이선스`, `종목 매핑`, `금융 문맥 정확도`, `평가셋 운영`입니다.
- 특히 외부 사용자 서비스에서 `구글검색/네이버뉴스/로이터 기사 본문을 직접 다운로드해서 저장·재배포`하는 전략은 실무 리스크가 큽니다. 공식 문서 기준으로도 `표시·재배포 권리`는 별도 계약이거나 기본 라이선스 바깥인 경우가 많습니다.
- 미국 뉴스 발견의 기본축으로 예전의 `Google Programmable Search JSON API`를 두는 것은 2026-03-12 기준으로 부적절합니다. Google 공식 문서상 신규 고객에게는 제공되지 않고, 종료일은 `2027-01-01`입니다.
- 가장 현실적인 저비용 구조는 `공식 검색/허용된 뉴스 피드 + 공시/IR + Claude 구조화 해석 + 링크아웃`입니다.
- 기사별 `좋아요/PV/UV/댓글수/댓글반응/댓글 PV`는 범용적으로 확보하기 어렵습니다. 공식 검색 API들에서는 보이지 않았고, 확보 가능하더라도 대개 `자사 서비스 1st-party 데이터` 또는 `별도 제휴 데이터` 성격입니다.

## 이번 리서치로 강화된 핵심 사실 🔍

### 1. FMP는 싸게 시작할 수 있지만, 표시/재배포 권리는 별도 문제입니다

- FMP의 `Stock News API`는 `symbols`, `from`, `to`, `page`, `limit` 기반으로 뉴스 목록을 조회할 수 있고, 응답에 `title`, `text`, `url`, `publishedDate`, `symbol`, `image`, `site` 같은 필드가 들어갑니다. 즉, `뉴스 발견 + 기초 메타데이터 + 티커 연결` 용도로는 진입장벽이 낮습니다.
- 하지만 FMP 가격 페이지는 별도로 `displaying and redistributing` 데이터에 대해 `Data Display and Licensing Agreement`가 필요하다고 명시합니다.
- 따라서 FMP는 `저가 API 구독 = 외부 서비스에서 기사 데이터 자유 재배포 가능`로 해석하면 안 됩니다.

### 2. FMP의 감성 분석은 "제공 기능"이라기보다 "직접 구축 예시"에 가깝습니다

- FMP의 교육 문서는 `FMP News API` 위에 `NLP-powered sentiment analyzer`를 얹는 예시를 설명합니다.
- 이 말은 실무적으로 `원시 뉴스 피드`는 제공하지만, `고품질 금융용 감성/이벤트 해석`은 결국 사용자가 직접 만들어야 한다는 뜻입니다.

### 3. DeepSearch는 한국 문서·기업 컨텍스트에 강합니다

- DeepSearch 회사 소개는 `국내 상장/비상장/미국 상장/해외 기업` 데이터, `문서 정보`, `뉴스, 공시, 증권사 리포트, IR, 특허, 어닝콜` 등을 폭넓게 다룬다고 소개합니다.
- DeepSearch 도움말의 데이터 가이드에서는 특히 `국내 뉴스(130개 언론사)`, `국내 공시`, `증권사 리포트`, `IR`, `특허` 등 한국 시장 중심의 문서 커버리지를 상세히 제시합니다.
- 또 `문서 검색 API`는 문서 타입별 검색을 지원하고, `FindAssociatedEntity`는 입력 문서에 대해 연관 엔티티와 점수를 반환합니다.
- 실무 추론 💡: 한국 종목 매핑과 금융 문맥 정합성은 DeepSearch 계열이 FMP보다 유리할 가능성이 높습니다. 반면 미국 뉴스 피드 자체의 실시간성·기사 본문 임베드 전략은 Benzinga/LSEG류가 더 강합니다.

### 4. Benzinga는 단순 뉴스 피드를 넘어 "서비스 탑재형"에 가깝습니다

- Benzinga 공식 `Stock News API` 페이지는 `real-time stock market news`를 제공하며, `플랫폼에 전체 기사와 이미지까지 임베드` 가능하다고 설명합니다.
- API 문서상 뉴스는 `tickers`, `ISIN`, `CUSIP`, `channels`, `authors`, `updated`, `date` 등으로 필터링할 수 있고, 응답의 `stocks` 배열에는 `ticker`, `name`, `exchange`, `ISIN`, `CUSIP`가 포함됩니다.
- `NewsQuantified`는 `sentiment`, `relevance`, 그리고 기사 이후 `1분/5분/10분` 가격 변동률 같은 정량 필드를 제공합니다.
- 실무 추론 💡: 미국 주식 대상 사용자 서비스라면, `Benzinga + Claude 후처리`는 `FMP + 직접 분석`보다 구현 속도와 품질 면에서 유리할 가능성이 큽니다.

### 5. Reuters 계열 본문은 "특히 조심"이 맞습니다

- LSEG News 서비스는 Reuters 금융 뉴스를 포함한 뉴스 제공을 설명하면서, 기본 `Standard License`는 `internal use only`이며 `distribution or redistribution`을 허용하지 않는다고 명시합니다.
- 실무 추론 💡: Reuters 본문을 직접 다운로드해 저장/재배포하는 전략은 매우 높은 라이선스 리스크로 봐야 합니다. 별도 계약 없이 `링크아웃 + 요약` 선에서 끝내는 쪽이 안전합니다.

### 6. Google 검색 API는 더 이상 장기 축이 되기 어렵습니다

- Google의 공식 문서는 `Custom Search JSON API`가 `신규 고객에게는 제공되지 않으며`, `2027-01-01`에 서비스가 중단된다고 안내합니다.
- 따라서 미국 뉴스 발견을 `Google Programmable Search JSON API`에 기대는 설계는 이제 유지보수성 측면에서 부적절합니다.

### 7. Naver Open API는 "뉴스 검색 결과 메타데이터" 용도에 가깝습니다

- Naver 뉴스 검색 API 문서는 결과 필드로 `title`, `originallink`, `link`, `description`, `pubDate` 등을 보여줍니다.
- Naver 검색 API 목록 문서는 `Search: 뉴스`의 호출 한도를 `25,000회/일`로 표기합니다.
- 2025-09-09 공지에서는 허용된 접근권한을 침해하는 자동화 접근, 봇/스크래퍼를 통한 제한 우회 같은 행위를 금지한다고 다시 못 박았습니다.
- 실무 추론 💡: 한국 뉴스 발견은 `Naver 뉴스 검색 API`로 하고, 본문 저장/대량 스크래핑은 기본 전략에서 빼는 편이 맞습니다.

## 왜 핵심 리스크는 토큰비보다 다른 곳에 있는가 💸

토큰비는 중요하지만, 대개 `선형적이고 예측 가능`합니다. 반대로 라이선스, 매핑, 금융 문맥, 평가셋은 한 번 틀리면 `서비스 자체를 막거나`, `조용히 잘못된 투자 정보`를 만들거나, `개선이 불가능한 시스템`을 만듭니다.

### Claude 토큰비는 계산 가능하고, 줄일 수 있습니다

Claude 공식 가격 문서 기준:

| 모델 | 입력 토큰 | 출력 토큰 | 비고 |
|---|---:|---:|---|
| Claude Sonnet 4.5 | $3 / MTok | $15 / MTok | 범용 고품질 |
| Claude Haiku 4.5 | $1 / MTok | $5 / MTok | 저비용 대량 처리 |

- `Prompt Caching`과 `Batch processing`도 공식적으로 제공됩니다.
- 특히 `Batch processing`은 공식 문서상 표준 가격 대비 `50% 할인`입니다.

### 예시 계산 🧮

가정:

- 월 30만 건 기사 처리
- 기사당 입력 400토큰
- 기사당 출력 120토큰
- 본문 전체가 아니라 `headline + snippet + source + 주변 공시 컨텍스트`만 사용

계산:

- 입력 총량: 1억 2천만 토큰
- 출력 총량: 3천 6백만 토큰

예상 비용:

| 모델 | 월 입력비 | 월 출력비 | 월 총비용 |
|---|---:|---:|---:|
| Sonnet 4.5 | $360 | $540 | $900 |
| Haiku 4.5 | $120 | $180 | $300 |
| Sonnet 4.5 + Batch | $180 | $270 | $450 |
| Haiku 4.5 + Batch | $60 | $90 | $150 |

해석 💡:

- 이 정도 규모면 토큰비는 `예측 가능`합니다.
- 반면 잘못된 라이선스 전략은 `출시 불가`, 잘못된 종목 매핑은 `신뢰도 붕괴`, 평가셋 부재는 `개선 불능`으로 이어집니다.
- 즉, MVP 단계에서 가장 무서운 건 `비용 폭탄`보다 `운영 불능 리스크`입니다.

## 진짜 어려운 4대 리스크 ⚠️

### 1. 뉴스 라이선스 리스크

- FMP는 표시/재배포에 별도 계약을 요구합니다.
- LSEG는 기본 라이선스로 재배포를 허용하지 않습니다.
- DeepSearch는 콘텐츠 저작권 보호를 명시하고 도입 문의형 구조입니다.
- Benzinga는 오히려 `표시/임베드 친화적`이라는 점이 장점입니다.

핵심 판단:

- `내부 리서치 툴`과 `외부 사용자 서비스`는 같은 수집 방식을 쓰면 안 됩니다.
- 외부 서비스라면 기본 저장 단위는 `title, url, source, publish_time, snippet, ticker mapping, 구조화 요약/감성/근거`까지로 제한하는 것이 안전합니다.

### 2. 종목 매핑 리스크

- 하나의 기사에 여러 회사가 등장할 수 있고, 동명이인 회사/브랜드/제품명이 섞입니다.
- 한국 시장은 `회사명`, `종목명`, `약어`, `브랜드`, `대표 제품`, `그룹명`이 자주 엇갈립니다.
- DeepSearch의 엔티티 연관 API, Benzinga의 `ticker/ISIN/CUSIP` 식별자 구조는 이 문제를 줄이는 데 유리합니다.
- FMP도 `symbol` 필드가 있지만, 문맥적으로 `주인공 종목`과 `언급 종목`을 구분하는 일은 별도 모델링이 필요합니다.

### 3. 금융 문맥 정확도 리스크

- 금융 뉴스는 일반 감성 분석보다 `이벤트 타입`과 `영향 방향`이 더 중요합니다.
- 예를 들어 `유상증자`, `전환사채`, `리콜`, `실적 서프라이즈`, `가이던스 하향`, `규제조사`, `소송`, `파트너십`, `자사주 매입`, `배당`은 같은 "긍/부정" 범주로 뭉개면 안 됩니다.
- Benzinga NewsQuantified나 RavenPack News Analytics는 이런 `금융 문맥 구조화`를 이미 제품으로 파는 쪽입니다.
- 완전 자작도 가능하지만, 이 영역이 실제 난이도 핵심입니다.

### 4. 평가셋 운영 리스크

- LLM 파이프라인은 그럴듯하게 보여도, 평가셋이 없으면 개선이 아니라 `기분`으로 튜닝하게 됩니다.
- 최소한 `KR/US 분리`, `대형주/중소형주 분리`, `업종별 분리`, `이벤트 타입 균형`, `애매한 기사/중복 기사` 버킷이 필요합니다.
- 이 부분은 공급사 API보다 내부 운영 역량 문제입니다.

## FMP vs DeepSearch vs Benzinga 실무 비교표 🧩

| 항목 | FMP | DeepSearch | Benzinga |
|---|---|---|---|
| 주력 포지션 | 저가 진입형 금융 데이터/뉴스 API | 한국 시장 중심 기업·문서·리서치 데이터 플랫폼 | 미국 주식 중심 실시간 뉴스/콘텐츠 플랫폼 |
| 뉴스/문서 범위 | Stock News API 중심. 기사 메타데이터와 URL 제공 | 국내 뉴스, 공시, 증권사 리포트, IR, 특허 등 문서층이 강함 | 실시간 시장 뉴스, 채널별 기사, 전체 기사 임베드 가능 |
| 한국 시장 적합성 | 낮음~보통 | 매우 높음 | 낮음 |
| 미국 시장 적합성 | 보통 | 보통 이하 | 매우 높음 |
| 종목 매핑 단서 | `symbol` 필드, 심볼 기반 필터 | 회사 심볼/이름 기반 조회, 연관 엔티티 API | `ticker`, `ISIN`, `CUSIP`, `exchange`, `stocks[]` 구조 |
| 감성/이벤트 분석 | 내장형보다는 DIY. 교육 문서로 예시 제공 | 공개 문서에서 강한 구조화 감성 제품은 확인 못 함 | `NewsQuantified`로 sentiment/relevance/price impact 제공 |
| 기사 본문 서비스화 | 권리 검토 필요. 표시/재배포 별도 계약 | 권리·계약 검토 필요 | 공식적으로 플랫폼 임베드 친화적 |
| 가격 투명성 | 공개 셀프서브 가격 존재 | 공개 정액 가격 확인 어려움 | 공개 정액 가격보다 도입 문의/유연 가격 |
| 장점 | 빠른 시작, 저렴한 진입, 간단한 API | 한국 금융 문맥, 기업/문서 그래프, 종목 매핑 강점 | 미국 뉴스 품질, 구조화 시그널, 서비스 탑재 용이 |
| 단점 | 재배포 권리와 금융 문맥은 별도 해결 필요 | 가격/권리 구조가 세일즈 의존적, 해외 뉴스 주력 플랫폼은 아님 | 한국 뉴스에는 약하고 비용이 FMP보다 높을 가능성 큼 |
| 추천 용도 | 미국/글로벌 메타데이터 MVP, 내부 실험 | 한국 종목 리서치/문서 연결/정확한 매핑 | 미국 사용자 서비스용 뉴스 레이어 |

### 비교표 해석 💡

- `FMP`는 `가볍게 시작`하기 좋지만, 외부 서비스에서 본문/디스플레이까지 해결해 주는 제품은 아닙니다.
- `DeepSearch`는 `한국 시장 정합성`이 강점입니다.
- `Benzinga`는 `미국 시장 서비스 탑재`에 가장 실무적입니다.

## LSEG와 RavenPack은 어떤 의미인가 🏦

- `LSEG News`는 Reuters 금융 뉴스를 포함한 고급 뉴스 인프라의 전형입니다. 다만 기본 라이선스가 `internal use` 중심이라, 일반 스타트업이 가볍게 외부 서비스에 얹기에는 무겁습니다.
- `RavenPack News Analytics`는 뉴스에서 `relevance`, `sentiment`, `novelty`, `market impact`를 구조화해 주는 대표적 벤치마크입니다.

실무 해석 💡:

- LSEG/RavenPack는 "`이걸 사면 자작을 덜 해도 되는 영역이 어디인지`"를 보여주는 상한선 벤치마크입니다.
- 하지만 예산과 계약 복잡도를 감안하면, 초기에는 `공식 메타데이터 + Claude 구조화`가 더 현실적입니다.

## 권장 아키텍처 초안 🏗️

### A안. 저예산 기본형

#### 한국 뉴스 발견

- `Naver 뉴스 검색 API`로 `종목명`, `종목코드`, `브랜드`, `제품명`, `그룹명` 질의
- 저장: `title`, `originallink`, `link`, `description`, `pubDate`
- 보강: `DART 공시`, `회사 IR/보도자료`

#### 미국 뉴스 발견

- `GDELT DOC 2.0`로 글로벌 기사 URL/제목/컨텍스트 검색
- `FMP Stock News API`를 보조 메타데이터 층으로 사용 가능
- `Google Programmable Search JSON API`는 신규 구축 기본축에서 제외

#### 해석 계층

- `Claude Haiku 4.5` 또는 `Claude Sonnet 4.5`
- `Pydantic AI output_type`으로 엄격한 구조화 출력 검증

권장 스키마 예시:

```json
{
  "primary_ticker": "string | null",
  "secondary_tickers": ["string"],
  "event_type": "earnings | guidance | lawsuit | regulation | mna | product | capital_raise | buyback | dividend | macro | other",
  "impact_direction": "positive | negative | mixed | unclear",
  "confidence": 0.0,
  "time_horizon": "intraday | days | weeks | quarters",
  "materiality": "high | medium | low",
  "why": "string",
  "counter_argument": "string",
  "evidence": ["headline/snippet/filing span"],
  "needs_human_review": true
}
```

#### 저장 원칙

- 저장: `메타데이터`, `구조화 요약`, `근거`, `종목 매핑`, `중복 제거용 해시`, `canonical URL`
- 비저장 또는 최소화: `기사 본문 전체`
- 사용자 노출: `요약 + 근거 링크 + 공시/IR 근거`

### B안. 품질 우선형

- 한국: `DeepSearch + DART + Claude`
- 미국: `Benzinga + SEC EDGAR + Claude`
- 장점: 종목 매핑과 금융 문맥 정확도가 올라감
- 단점: 비용과 계약 복잡도 상승

## Google/Naver/Reuters 본문 직접 다운로드 전략에 대한 판단 🧨

### 내 판단

- 내부 리서치 툴: 제한적 수집은 가능할 수 있으나, 여전히 약관/권리 검토 필요
- 외부 사용자 서비스: Google/Naver/Reuters 기사 본문 직접 다운로드 기반은 비추천
- 저예산 정답: `합법적 메타데이터 수집 + 공시/IR 보강 + Claude 해석`

### 이유

- Google 검색 API는 장기 축으로 부적합합니다.
- Naver는 공식 검색 API가 존재하고, 자동화 접근 제한 위반 금지를 명시했습니다.
- Reuters 계열은 엔터프라이즈 라이선스 통제가 강합니다.
- 결국 본문 크롤링은 `기술적으로 가능`과 `서비스에 안전하게 쓸 수 있음`이 다릅니다.

## 기사별 좋아요/PV/UV/댓글수/댓글반응/댓글 PV를 활용할 수 있는가 📊

짧게 답하면:

- `범용적으로는 어렵다`
- `공식 API 기반으로는 대부분 없다`
- `대체 프록시 신호`로 바꾸는 게 현실적이다

### 무엇이 공식적으로 보였나

| 지표 | 공식 API에서 확인된 정도 | 판단 |
|---|---|---|
| 제목/링크/스니펫/발행시각 | Naver, FMP, GDELT 등에서 확인 | 적극 활용 가능 |
| 티커/식별자 | FMP, Benzinga, DeepSearch 계열에서 확인 | 적극 활용 가능 |
| 기사별 좋아요/댓글수/댓글반응 | 검토한 공식 검색/뉴스 API들에서 명시적으로 확인 못 함 | 범용 수집은 어렵고 비공식 스크래핑 위험 큼 |
| 기사별 PV/UV | 검토한 공식 검색/뉴스 API들에서 확인 못 함 | 퍼블리셔 제휴나 자체 분석툴 없이는 사실상 어려움 |
| 키워드/쇼핑 관심도 | Naver DataLab에서 검색어/클릭 트렌드 제공 | 기사별이 아니라 `주제 관심도 프록시`로 활용 가능 |
| 시장 반응 프록시 | Benzinga NewsQuantified, RavenPack 류 존재 | 기사 인기도보다 `시장 영향도` 프록시로 유용 |

### 실무적으로 가능한 대안 💡

1. `자사 서비스 1st-party 지표`
   - CTR, 저장, 공유, 체류시간, 스크롤, 즐겨찾기, 알림 클릭

2. `검색 관심도 프록시`
   - Naver DataLab 검색어 트렌드
   - 종목명/브랜드명 급증 여부

3. `시장 반응 프록시`
   - Benzinga NewsQuantified의 relevance / sentiment / price impact
   - RavenPack의 relevance / novelty / market impact

4. `소셜/커뮤니티 프록시`
   - 별도 제휴 가능한 커뮤니티나 소셜 데이터가 있다면 보조 활용
   - 다만 이것도 기사 그 자체의 PV/UV와는 다름

### 추천 결론

- `기사별 PV/UV/댓글 PV`를 핵심 랭킹 신호로 두지 마세요.
- 대신 `기사 품질 + 종목 중요도 + 시장 영향 프록시 + 자사 사용자 반응` 조합이 더 현실적입니다.

예시:

```text
attention_score
  = source_weight
  + market_impact_proxy
  + search_trend_proxy
  + first_party_engagement
  - duplicate_penalty
  - stale_penalty
```

## 평가셋 운영 제안 🧪

이 부분은 공식 문서보다 실무 제안에 가깝습니다.

### 최소 권장 초기 셋

- KR 1,000건 + US 1,000건
- 이벤트 타입 균형 샘플링
- 긍정/부정/혼합/불명 기사 포함
- 중복 기사와 리라이트 기사 포함

### 라벨 항목

- `primary_ticker`
- `secondary_tickers`
- `event_type`
- `impact_direction`
- `time_horizon`
- `materiality`
- `evidence_spans`
- `abstain_flag`

### 지표

- `Ticker Precision@1`
- `Event Type F1`
- `Direction Accuracy`
- `Calibration/Brier`
- `Grounding Precision`
- `Human Review Rate`

### 운영 포인트

- 주 1회 오류 버킷 리뷰
- 월 1회 라벨셋 갱신
- 소형주/테마주/바이오/정책주처럼 어려운 버킷 별도 관리

## 최종 제안 ✅

### 지금 당장 가장 현실적인 조합

- 한국: `Naver 뉴스 검색 API + DART + 회사 IR + Claude`
- 미국: `GDELT 또는 Benzinga + SEC EDGAR + 회사 IR + Claude`
- 구조화: `Pydantic AI + 엄격한 JSON 스키마`
- 저장 정책: `메타데이터/요약/근거만 저장`, 본문은 원칙적으로 링크아웃

### 예산이 조금 더 있으면

- 한국은 `DeepSearch`를 붙여 종목 매핑과 금융 문맥을 강화
- 미국은 `Benzinga`를 붙여 품질과 서비스 탑재성을 높임

### 피해야 할 기본전략

- 구글검색/네이버뉴스/로이터 본문 대량 다운로드를 제품 핵심 파이프라인으로 두는 것
- 기사별 PV/UV/댓글수를 범용적으로 얻을 수 있다고 가정하는 것
- 평가셋 없이 감성/이벤트 모델을 운영하는 것

## 근거 URL 🌐

- FMP Pricing Plans: https://site.financialmodelingprep.com/pricing-plans
- FMP Stock News API: https://site.financialmodelingprep.com/developer/docs/stable/stock-news
- FMP NLP-powered sentiment analyzer article: https://site.financialmodelingprep.com/education/news/nlppowered-sentiment-analyzer-using-fmp-news-api
- DeepSearch company: https://company.deepsearch.com/
- DeepSearch 문서 데이터 API: https://help.deepsearch.com/dp/api/data/document
- DeepSearch 문서 검색 가이드: https://help.deepsearch.com/guide/data/document
- DeepSearch 연관 엔티티 가이드: https://help.deepsearch.com/guide/api/entity/findassociatedentity
- Benzinga NewsQuantified overview: https://docs.benzinga.com/api-reference/newsquantified-api/overview
- Benzinga News API: https://docs.benzinga.com/benzinga-apis/newsfeed/news
- Benzinga Stock News API product page: https://www.benzinga.com/apis/stock-news-api
- Benzinga pricing/contact page: https://www.benzinga.com/apis/pricing
- LSEG News overview: https://developers.lseg.com/en/product/news/overview
- RavenPack News Analytics: https://www.ravenpack.com/products/edge/data/news-analytics
- Pydantic AI output: https://ai.pydantic.dev/output/
- Claude pricing: https://platform.claude.com/docs/ko/about-claude/pricing
- Google Custom Search JSON API overview: https://developers.google.com/custom-search/v1/overview
- GDELT DOC 2.0 API: https://blog.gdeltproject.org/gdelt-doc-2-0-api-debuts/
- GDELT Context 2.0 API: https://blog.gdeltproject.org/announcing-the-gdelt-context-2-0-api/
- Naver 뉴스 검색 API: https://developers.naver.com/docs/serviceapi/search/news/news.md
- Naver 검색 API 목록/호출한도: https://api.ncloud-docs.com/docs/ai-naver-searchsearch
- Naver API 정책 공지(자동화 접근 관련): https://developers.naver.com/notice/article/1298748
- Open DART Open API: https://opendart.fss.or.kr/guide/main.do?apiGrpCd=DE001
- SEC EDGAR APIs: https://www.sec.gov/search-filings/edgar-application-programming-interfaces

## 사실검증 메모 🧾

- `Google Programmable Search JSON API` 관련 문구는 Google 공식 문서의 현재 상태 기준으로 반영했습니다. 따라서 예전 자료와 충돌할 수 있으나, 2026-03-12 시점 판단은 `신규 구축 비권장`이 맞습니다.
- `Reuters 직접 다운로드 고위험`은 Reuters 자체 약관만이 아니라 `LSEG News 기본 라이선스 구조`를 근거로 한 실무 추론입니다.
- `기사별 좋아요/PV/UV/댓글` 부재 판단은 검토한 공식 API 문서 범위에서의 결론입니다. 특정 퍼블리셔 제휴 API가 있다면 예외가 있을 수 있습니다.

## 사용자 질문 프롬프트

```text
$hhd-research

think ultra hard

아래 내용에 대해 좀더 리서치 해서 내용 보강해서 리포트를 작성

- 완전 자작은 가능하지만, 핵심 리스크는 토큰비보다 뉴스 라이선스, 종목 매핑, 금융 문맥 정확도, 평가셋 운영입니다

- 주요 근거:

  - FMP 가격/라이선스: https://site.financialmodelingprep.com/pricing-plans
  - FMP Stock News API: https://site.financialmodelingprep.com/developer/docs/stable/stock-news
  - FMP sentiment analyzer article:
    https://site.financialmodelingprep.com/education/news/nlppowered-sentiment-analyzer-using-fmp-news-api
  - DeepSearch: https://company.deepsearch.com/
  - DeepSearch 문서 데이터 API: https://help.deepsearch.com/dp/api/data/document
  - Benzinga NewsQuantified: https://docs.benzinga.com/api-reference/newsquantified-api/overview
  - LSEG News: https://developers.lseg.com/en/product/news/overview
  - RavenPack News Analytics: https://www.ravenpack.com/products/edge/data/news-analytics
  - Pydantic AI: https://ai.pydantic.dev/output/
  - Claude pricing: https://platform.claude.com/docs/ko/about-claude/pricing

- FMP vs DeepSearch vs Benzinga 실무 비교표 작성


• 짧게 결론부터 말하면, Claude API로 구글검색/네이버뉴스/로이터 기사 본문을 직접 다운로드해서 서비스에 쓰는 방식은

  현실적인 저비용 구조

  - 검색/발견은 공식 API나 허용된 피드로 처리
  - 저장은 title, url, source, publish_time, snippet, ticker mapping, Claude 요약/감성/근거만 저장
  - 원문 본문은 가능하면 저장하지 말고 링크아웃
  - 본문 추출은 허용된 소스만 제한적으로
  - Reuters 같은 고위험 매체는 본문 저장/재배포 없이 링크만 사용

  내 판단으로는, 당신 상황에서는 아래 조합이 가장 현실적입니다.

  1. 한국 뉴스 발견

  - NAVER 뉴스 검색 API 사용
  - 종목명, 종목코드, 별칭, 대표상품명으로 질의
  - 제목/링크/설명(snippet)만 수집

  2. 미국 뉴스 발견

  - Google SERP 스크래핑보다 Google Programmable Search JSON API 같은 공식 검색 API를 우선 검토
  - 또는 GDELT 같은 공개 뉴스 인덱스를 보조 신호로 사용
  - GDELT는 기사 목록, URL, 다국어 검색, tone/context 계열 분석에 쓸 수 있습니다
    출처: https://developers.google.com/custom-search/v1/overview
    출처: https://blog.gdeltproject.org/gdelt-doc-2-0-api-debuts/
    출처: https://blog.gdeltproject.org/announcing-the-gdelt-context-2.0-api/

  3. 뉘앙스 분석

  - Claude는 아주 적합합니다
  - 본문 전체가 없어도 headline + snippet + 출처 + 최근 공시/실적/IR만으로 1차 뉘앙스는 꽤 잘 뽑습니다
  - 특히 금융뉴스는 단순 긍부정보다 아래를 구조화하는 게 더 중요합니다
  - event_type, impact_direction, confidence, time_horizon, why, counter_argument

  4. 고품질 보강 데이터

  - 한국: DART 공시, 회사 IR/보도자료
  - 미국: SEC EDGAR, 회사 IR, 실적 발표자료
  - 이건 기사보다 법적 리스크가 낮고 신뢰도가 높습니다

  내 추천은 명확합니다.
  검색결과/기사 본문 대량 스크래핑을 기본전략으로 잡지 말고, 공식 검색 API + 공개 공시/IR + Claude 해석 계층으로 가는 게 맞습니다.

  실무적으로는 이렇게 시작하면 됩니다.

  - NAVER 뉴스 검색 API로 한국 종목 뉴스 수집
  - 미국은 공식 검색 API 또는 GDELT로 URL/제목/스니펫 수집
  - 원문 저장은 최소화
  - Claude로 종목별 요약/뉘앙스/이벤트 추출
  - 사용자에게는 요약 + 근거 링크 중심으로 제공
  - 본문 재게시 기능은 빼기

  한 줄 판단:

  - 내부 리서치 툴: 제한적 크롤링은 가능하나 여전히 약관 리스크 있음
  - 외부 사용자 서비스: Google/Naver/Reuters 본문 직접 다운로드 기반은 비추천
  - 저예산 정답: 합법적 메타데이터 수집 + Claude 해석

- 여기에 추가로 각 뉴스별
  - 좋아요, PV, UV, 댓글수, 댓글반응, 댓글 PV 등을 활용할 수 있는가?
```
