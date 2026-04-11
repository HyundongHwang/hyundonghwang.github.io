---
layout: post
title: "260311 글로벌 경제뉴스 API 리서치"
comments: true
tags:
- global
- financial
- news
- api
---

# 260311 글로벌 경제뉴스 API 리서치

경제뉴스 조회와 검색 솔루션을 고를 때 가장 많이 부딪히는 요구사항은 늘 비슷하다. 무료에 가깝고, REST로 쉽게 붙고, 최근 기사만이 아니라 과거 뉴스도 길게 보고 싶고, 국가와 분야가 넓게 커버되며, 종목과 회사 단위로 분류되고, 가능하면 감정분석 점수까지 붙어 있으면 좋다는 요구다.

문제는 이 조건들을 한 번에 모두 만족하는 단일 솔루션이 거의 없다는 점이다. 특히 아래 두 조건이 가장 어렵다.

- `무료 선호`
- `최근부터 최대 20년 정도 예전 뉴스까지 자세한 원문 조회 가능`

공식 문서를 기준으로 조사해보면, 무료 또는 저가형 REST API는 대개 다음 중 하나를 포기한다.

- 원문 전체 제공
- 20년급 장기 아카이브
- 글로벌 전방위 국가/언어 커버리지
- 종목/회사 엔터티 정규화
- 감정분석 점수

즉, 이 요구는 현실적으로는 `하이브리드 구성`이 가장 맞다.

## 결론 먼저

내 판단으로는 아래처럼 정리하는 것이 가장 실용적이다.

### 1. 무료 최우선

`GDELT + 직접 기사 원문 수집/정제 + 자체 종목매핑/감정분석`

- 장점: 무료, 글로벌 범위가 매우 넓음
- 단점: 금융 종목 정규화가 약하고, 바로 쓸 수 있는 "주식 뉴스 API" 느낌은 아님
- 적합: 대규모 리서치, 국가/이슈 단위 글로벌 모니터링

### 2. 저가 + 금융 특화 + REST 편의성

`Marketaux`

- 장점: 금융 뉴스 특화, 엔터티/국가/산업/감정 필터가 강함
- 단점: 무료 플랜은 매우 제한적이고, 20년급 원문 아카이브는 아님
- 적합: 종목별 뉴스 피드, 실시간 모니터링, 대시보드

### 3. 원문 필드 + NLP + 회사/국가/주제 검색까지 균형형

`NewsCatcher`

- 장점: full content 필드, NLP, 국가/언어/주제 검색, REST 완성도 높음
- 단점: 역사 데이터는 공식 문서상 `2019년 이후`
- 적합: 기사 본문 분석, RAG, 검색엔진형 뉴스 시스템

### 4. 매우 긴 히스토리와 라이선스 품질까지 원하면

`LexisNexis Nexis Data+`

- 장점: 40년 이상, 235개 국가/영토, 75개 언어, 감정/엔터티/금융 문맥 강함
- 단점: 무료/저가 영역이 아님
- 적합: 기관형 서비스, 규제 대응, 장기 백테스트, 상업적 재배포

즉, 당신의 조건을 가장 솔직하게 해석하면:

- `무료 선호`가 가장 중요하면 `GDELT`
- `금융 종목/회사 분류와 감정분석`이 더 중요하면 `Marketaux`
- `자세한 원문 조회`가 더 중요하면 `NewsCatcher`
- `20년 가까운 장기 히스토리`가 정말 중요하면 `LexisNexis`

## 왜 단일 솔루션이 잘 안 맞는가

뉴스 API는 보통 세 갈래로 나뉜다.

### 1. 범용 뉴스 검색 API

예: NewsAPI, mediastack, NewsCatcher

- 장점: 검색이 쉽고 국가/언어 범위가 넓다
- 단점: 종목/회사 엔터티 정규화가 약할 수 있다

### 2. 금융 뉴스 특화 API

예: Marketaux, Alpha Vantage News Sentiment, EODHD News/Sentiment

- 장점: 종목, 회사, 산업, 감정 점수에 강하다
- 단점: 글로벌 일반뉴스 전체를 다 잡지는 못할 수 있다

### 3. 연구/아카이브형 글로벌 데이터셋

예: GDELT, LexisNexis

- 장점: 범위와 역사성이 강하다
- 단점: 바로 붙여서 쓰는 제품형 UX는 덜할 수 있고, 원문 라이선스나 정제는 따로 봐야 한다

## 후보별 상세 분석

## 1. GDELT

GDELT는 무료 공개 리서치 플랫폼 쪽에 가깝다. 공식 사이트는 "세계 거의 모든 국가의 뉴스"를 모니터링한다고 설명하고, 100개 이상 언어를 다룬다고 안내한다. 또한 GDELT DOC 2.0 API는 full text search API라고 소개된다.

강한 점:

- 무료
- 글로벌 범위가 매우 넓음
- 다국어 번역 기반 검색 가능
- 톤 기반 분석 가능
- REST 스타일 접근 가능

공식 문서상 확인되는 포인트:

- GDELT는 전 세계 뉴스 미디어를 100개 이상 언어로 모니터링
- 65개 언어에 대해 실시간 영어 번역 처리
- DOC 2.0 API는 full text search API
- DOC 2.0 API는 검색창구 기준 2017년 1월 1일 이후를 대상으로 설명
- 전체 이벤트 DB는 BigQuery나 다운로드 방식으로 더 깊게 접근 가능

약한 점:

- "주식 종목별 회사별 분류"가 금융 전용 API만큼 깔끔하지 않다
- 감정분석도 금융 문맥 특화 스코어라기보다 global tone 성격이 더 강하다
- 원문 전문을 그대로 라이선스 안정적으로 재배포하는 제품으로 보기엔 다소 다르다

정리:

`전세계, 무료, 장기, REST, 대규모 검색` 쪽으로는 가장 강하다.  
반면 `주식 종목별 회사별 뉴스 인덱스`로 바로 쓰기에는 후처리가 필요하다.

## 2. Marketaux

이번 조사에서 "금융뉴스 API" 관점으로 가장 균형이 좋은 후보다.

공식 pricing 페이지 기준:

- Free: 월 `0달러`, 하루 `100 requests`, 요청당 `3 articles`
- Basic: 월 `24달러`
- Standard: 월 `49달러`
- Pro: 월 `99달러`

공식 문서상 강점:

- `200,000+ entities`
- `5,000+ news sources`
- `80+ global markets`
- `30+ languages`
- 국가(`countries`), 산업(`industries`), 종목(`symbols`), 엔터티 타입(`entity_types`) 필터
- `sentiment_gte`, `sentiment_lte`, `sentiment_avg_gte` 같은 감정/스코어 필터

특히 금융 서비스로서 중요한 점은 `entity` 중심 설계다.

- `symbols=TSLA,AMZN,MSFT`
- `countries=us,ca`
- 산업, 거래소, 국가 기준 집계
- 엔터티 sentiment 기반 검색

약한 점:

- 무료 플랜이 너무 작다
- 공식 문서상 집계성 조회는 `year` 간격 기준 최대 `10년` 프레임 제한이 보인다
- 20년급 상세 기사 아카이브를 보장하는 솔루션으로 읽히지는 않는다

내 해석:

`실시간 + 금융 엔터티 + 감정분석 + 글로벌 시장` 요구에는 매우 잘 맞는다.  
하지만 `20년 가까운 기사 원문 아카이브` 요구까지 단독으로 충족한다고 보기는 어렵다.

## 3. NewsCatcher

NewsCatcher는 "검색엔진형 뉴스 API"에 가깝고, 이번 조건 중 `원문 필드`, `NLP`, `국가/언어/주제`, `REST 완성도`가 두드러진다.

공식 문서상 강점:

- `/search`, `/latest_headlines`, `/breaking_news`, `/sources`, `/aggregation_count`
- `GET`와 `POST` 모두 지원
- 고급 쿼리, NLP-enriched content, deduplication, clustering
- 국가, 언어, 소스, 날짜 기준 검색
- 기사 추가 필드에 `content` 즉 full article content 포함 가능

공식 구독 문서상 확인되는 포인트:

- `historical data since 2019`
- 감정분석을 `-1.0 ~ 1.0` 스케일로 제공
- named entities
- summaries
- 영어 번역 기반 NLP 처리
- 최대 `1,000 articles per request`

약한 점:

- 2019년 이후라서 20년 요구에는 부족
- 무료라기보다는 trial/paid 성격에 가깝다
- 금융 종목 ticker 정규화는 Marketaux보다 약할 수 있다

정리:

`기사 원문 + NLP + 검색 UX + 글로벌 뉴스` 조합은 좋다.  
하지만 `20년 아카이브`가 중요하면 단독 주력으로는 부족하다.

## 4. NewsAPI

NewsAPI는 가장 유명한 범용 REST 뉴스 API 중 하나다. 하지만 이번 요구와는 거리가 있다.

공식 문서상:

- REST API
- `Everything` 엔드포인트로 검색 가능
- 기사 검색은 최대 `5년`
- Developer 무료 플랜은 `24시간 지연`, `100 requests/day`, `한 달치 검색`
- Business 플랜부터 실시간과 5년 검색

가장 큰 한계는 공식 pricing FAQ에 명확히 나온다.

- full article content는 제공하지 않음
- URL을 주고 필요하면 별도 스크래핑하라고 안내

정리:

뉴스 검색 프로토타입에는 좋지만,  
당신이 원하는 `자세한 원문`, `종목별 회사별 분류`, `감정분석`, `20년 아카이브`에는 부적합하다.

## 5. mediastack

mediastack은 범용 글로벌 뉴스 API로는 무난하다.

공식 문서상:

- worldwide live and historical news data
- 국가, 언어, 소스, 키워드, 날짜 범위 필터
- historical news는 Standard 플랜 이상

공식 pricing 페이지 기준:

- Free: `100 calls / month`
- Standard: `10,000 calls / month`, 월 `24.99달러`
- Professional: 월 `99.99달러`
- `7,500+ News Sources`
- `50+ Countries`

약점:

- 금융 종목/회사 엔터티 정규화가 약함
- 감정분석 내장 강점이 문서상 두드러지지 않음
- 20년 장기 아카이브 근거를 찾기 어려움

정리:

범용 글로벌 뉴스 수집용으로는 괜찮지만,  
`경제뉴스 검색 솔루션`의 핵심인 종목/감정/회사 엔터티 요구에는 우선순위가 낮다.

## 6. Alpha Vantage News Sentiment

Alpha Vantage는 저비용 실험용으로 매력은 있다.

공식 문서상:

- `NEWS_SENTIMENT` 함수 제공
- live and historical market news & sentiment
- stocks, crypto, forex, fiscal policy, M&A, IPO 등 광범위 주제
- `time_from`, `time_to` 지원
- `limit=1000`
- ticker 기반 질의 가능

공식 premium 페이지에서는:

- 대부분 endpoint는 무료 사용 가능
- 무료 사용 한도는 `25 requests/day`
- 더 높은 사용량은 premium 필요

약점:

- full article content 제공이 핵심 장점으로 보이지 않는다
- 역사 깊이의 공식 명시가 이번 조사에서는 충분히 선명하지 않았다
- 글로벌 뉴스 전체를 exhaustive하게 다루는 플랫폼이라고 보기는 어렵다

내 판단:

`무료 실험`과 `종목+감정분석 시작점`으로는 좋다.  
하지만 메인 뉴스 백본으로 쓰기에는 제한이 크다.

## 7. EODHD News / Sentiment API

EODHD는 금융 데이터 패키지에 뉴스와 sentiment를 묶어 제공하는 스타일이다.

공식 문서상 확인되는 포인트:

- `Financial News Feed and Stock News Sentiment data API`
- Sentiment endpoint: `GET https://eodhd.com/api/sentiments`
- 점수 스케일: `-1 ~ 1`
- stocks, ETFs, crypto 지원
- 날짜 범위 `from`, `to` 지원
- free plan 포함 문구 존재

공식 pricing 페이지에서는 개인용 `All-in-One`이 월 `69.99달러`로 보이고, 상업용은 더 높다.

약점:

- 저가라고 보기엔 다소 애매
- 글로벌 일반뉴스 전체 범위보다 금융 데이터 묶음 성격이 강함
- 20년 원문 아카이브를 강하게 약속하는 문구는 이번 조사에서 확인하지 못했다

정리:

금융 데이터 스택을 한 군데서 묶고 싶을 때는 후보가 될 수 있다.  
하지만 `무료 선호`에는 잘 맞지 않는다.

## 8. LexisNexis Nexis Data+

이건 사실상 기관형 정답에 가깝다.

공식 문서상 확인되는 포인트:

- historical archive `45년+`
- `75 languages`
- `235 countries and territories`
- `80,000+ sources` 또는 별도 페이지에서 `100,000+ global print, broadcast, and web sources`
- stock news API 문맥에서 sentiment, historical archives, licensed content 강조

강점:

- 장기 보관
- 글로벌 범위
- 라이선스 안정성
- 감정분석/엔터티/기업 문맥

약점:

- 무료 선호와 정면 충돌
- 보통 엔터프라이즈 세일즈 대상

정리:

당신의 요구를 가장 많이 만족시키는 상용 솔루션이다.  
다만 무료/저가형이 아니라는 점이 핵심이다.

## 요구사항별 매칭표

| 요구 | GDELT | Marketaux | NewsCatcher | NewsAPI | mediastack | Alpha Vantage | EODHD | LexisNexis |
|---|---|---|---|---|---|---|---|---|
| 무료 선호 | 강함 | 보통 | 약함 | 개발용만 | 약함 | 강함 | 약함 | 매우 약함 |
| REST | 강함 | 강함 | 강함 | 강함 | 강함 | 강함 | 강함 | 보통 |
| 최근 뉴스 | 강함 | 강함 | 강함 | 강함 | 강함 | 강함 | 강함 | 강함 |
| 20년급 과거 뉴스 | 약함 | 약함 | 약함 | 약함 | 약함 | 불명확 | 약함 | 매우 강함 |
| 자세한 원문 | 약함 | 보통 | 강함 | 약함 | 약함 | 약함 | 보통 | 강함 |
| 전세계 국가/분야 | 매우 강함 | 강함 | 강함 | 강함 | 보통 | 보통 | 보통 | 매우 강함 |
| 종목/회사 분류 | 약함 | 매우 강함 | 보통 | 약함 | 약함 | 보통 | 강함 | 강함 |
| 감정분석/스코어 | 보통 | 강함 | 강함 | 약함 | 약함 | 강함 | 강함 | 강함 |

이 표의 `불명확`은 공식 문서에서 명시적으로 보장 범위를 찾지 못한 항목이다.

## 현실적인 추천 시나리오

## 시나리오 A. 무료로 최대한 시작

구성:

- `GDELT`로 글로벌 기사 후보 수집
- 기사 URL 기준으로 원문 추출기 직접 운영
- 종목 매핑은 별도 NER / ticker mapping
- 감정분석은 자체 모델 또는 별도 NLP API

장점:

- 비용이 거의 없음
- 국가/언어 범위가 매우 큼

단점:

- 구현 난이도가 높음
- 금융 종목 품질 보정이 필요

추천도:

`연구용`, `개인 프로젝트`, `프로토타입`에는 가장 적합

## 시나리오 B. 가장 균형 좋은 실무형

구성:

- `Marketaux` 메인
- 부족한 본문은 원문 URL 기반 수집
- 장기 아카이브나 특수 국가 이슈는 `GDELT` 보조

장점:

- 금융 종목/회사/산업/국가/감정 스키마가 바로 usable
- REST 사용성이 좋음

단점:

- 완전 무료는 아님
- 장기 아카이브는 한계

추천도:

`주식 종목별 뉴스 대시보드`, `알림 시스템`, `감정 기반 랭킹`에 가장 적합

## 시나리오 C. 원문 기반 검색엔진/RAG

구성:

- `NewsCatcher` 메인
- `content` 필드 활용
- 추가로 자체 벡터 인덱스 구축
- 금융 티커 분류는 별도 enrichment

장점:

- 원문 분석에 유리
- NLP, 번역, dedup, clustering이 좋음

단점:

- 2019년 이전이 부족
- 순수 금융 엔터티 중심 API는 아님

추천도:

`뉴스 검색`, `RAG`, `질의응답형 뉴스 시스템`에 적합

## 시나리오 D. 기관형 정답

구성:

- `LexisNexis Nexis Data+`

장점:

- 요구사항 충족률이 가장 높음

단점:

- 비용이 높고 도입 절차가 무거움

추천도:

`상업 제품`, `기관 고객`, `컴플라이언스`, `20년+ 백테스트`에 적합

## 내 최종 판단

당신의 요구를 있는 그대로 우선순위화하면 아마 이렇게 읽힌다.

1. 무료 또는 매우 저렴해야 함
2. REST여야 함
3. 경제뉴스를 글로벌하게 넓게 봐야 함
4. 종목/회사별 분류와 감정분석이 있으면 좋음
5. 가능하면 20년 가까운 과거도 보고 싶음

이 기준이면 단일 솔루션 1개를 고르기보다 아래처럼 가는 것이 맞다.

### 추천 1

`GDELT + Marketaux`

- GDELT: 글로벌 범위, 무료, 거시/국가/이슈 탐색
- Marketaux: 종목/회사/감정/금융 특화

이 조합이 `무료 선호`와 `금융 분석 실용성`의 균형이 가장 좋다.

### 추천 2

`NewsCatcher + Marketaux`

- NewsCatcher: 기사 원문과 NLP
- Marketaux: 금융 엔터티와 sentiment filtering

이 조합은 실제 앱/검색엔진 제품화에 더 좋다.

### 추천 3

`LexisNexis` 단독

- 예산이 충분하고 장기 아카이브가 핵심이면 가장 강함

## 한 줄 요약

- `무료` 중심이면 `GDELT`
- `금융 종목/회사/감정분석` 중심이면 `Marketaux`
- `원문 content + NLP` 중심이면 `NewsCatcher`
- `20년 이상 장기 아카이브` 중심이면 `LexisNexis`

그리고 솔직히 말하면,  
`무료 + REST + 글로벌 + 종목별 + 감정분석 + 20년 원문`을 한 번에 만족하는 단일 API는 이번 조사 기준 찾지 못했다.

## 참고 URL

- GDELT Project  
  https://www.gdeltproject.org/
- GDELT DOC 2.0 API  
  https://blog.gdeltproject.org/gdelt-doc-2-0-api-debuts/amp/
- Marketaux Pricing  
  https://www.marketaux.com/pricing
- Marketaux Documentation  
  https://www.marketaux.com/documentation
- NewsCatcher API Overview  
  https://www.newscatcherapi.com/docs/news-api/api-reference/overview
- NewsCatcher Article Fields  
  https://www.newscatcherapi.com/docs/v3/events/overview/working-with-articles
- NewsCatcher Subscription Plans  
  https://www.newscatcherapi.com/docs/news-api/get-started/subscription-plans
- NewsAPI Pricing  
  https://newsapi.org/pricing
- NewsAPI Docs  
  https://newsapi.org/docs
- mediastack Pricing  
  https://mediastack.com/pricing/
- mediastack Documentation  
  https://mediastack.com/documentation
- Alpha Vantage Documentation  
  https://www.alphavantage.co/documentation/
- Alpha Vantage Premium  
  https://www.alphavantage.co/premium/
- EODHD Financial News API  
  https://eodhd.com/financial-apis/stock-market-financial-news-api
- EODHD Pricing  
  https://eodhd.com/pricing-zorroproject
- LexisNexis Historical News API  
  https://www.lexisnexis.com/en-gb/products/nexis-daas/historical-news-api
- LexisNexis Stock News API  
  https://www.lexisnexis.com/en-gb/products/nexis-daas/stock-news-api

```text
$hhd-md
$hhd-research

think hard

경제뉴스 조회, 검색 솔루션
- 무료 선호 
- REST 솔루션 선호
- 최근부터 최대 20년 정도 예전 뉴스까지 자세한 원문 조회 가능
- 모든 국가별 모든 분야별 뉴스 선호 
- 각 주식 종목별 회사별 분류 선호
- 감정분석, 스코어 기반 분류 선호
```
