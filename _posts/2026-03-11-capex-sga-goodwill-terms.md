---
layout: post
title: "260311 CAPEX SG&A Goodwill 용어정리"
comments: true
tags:
- capex
- sga
- goodwill
- terms
---

# 260311 CAPEX SG&A Goodwill 용어정리

회계나 기업 분석 문서를 읽다 보면 `CAPEX`, `SG&A`, `goodwill` 같은 약어와 함께 `자본`, `자산`, `부채`, `영업이익` 같은 기본 용어를 영어로 바로 떠올려야 할 때가 많다. 이번 메모는 이 용어들을 한 번에 연결해서 정리한 노트다.

이번 문서는 용어 정리 중심이라 삽화는 생략했다. 개념 설명보다 정의와 쓰임의 구분이 중요해서 텍스트 위주가 더 적합했다.

## 1. CAPEX는 무엇인가

`CAPEX`는 `capital expenditures` 또는 `capital expenditure`의 약자다. 한국어로는 보통 `자본적 지출`, 실무 문맥에서는 그냥 `설비투자`라고 많이 부른다.

핵심은 이렇다.

- 장기간 사용할 자산을 사거나 만들거나 크게 개선하는 데 들어가는 지출이다.
- 예: 공장 증설, 서버 구매, 생산설비 교체, 건물 취득
- 당기 비용으로 바로 털기보다 자산으로 잡고 여러 기간에 걸쳐 감가상각 또는 상각되는 경우가 많다.

실무 감각으로 보면:

- `OPEX`는 지금 운영을 위한 비용
- `CAPEX`는 미래 여러 기간에 효익을 주는 투자

주의할 점:

- 투자자들이 흔히 말하는 `CAPEX`는 현금흐름표나 PP&E 주석에서 읽는 경우가 많다.
- 손익계산서의 `expense`와 1:1로 대응한다고 생각하면 헷갈리기 쉽다.

## 2. 판매관리비는 영어로 무엇인가

`판매관리비`는 가장 자연스럽게는 `SG&A` 또는 `selling, general and administrative expenses`라고 보면 된다.

다만 완전히 기계적으로 1:1 대응하는 것은 아니다.

- `판매비`에 가까운 부분: `selling expenses`
- `관리비`에 가까운 부분: `general and administrative expenses`
- 둘을 묶은 표현: `SG&A`

보통 포함되는 항목:

- 영업/마케팅 관련 비용
- 본사 관리 인건비
- 회계, 법무, 사무실 운영비
- 제품 생산원가에 직접 넣지 않는 간접비

실무에서는 회사마다 표시 방식이 조금 다르다.

- 어떤 회사는 `SG&A` 한 줄로 합쳐 표시
- 어떤 회사는 `sales and marketing`, `general and administrative`로 분리
- 어떤 회사는 `operating expenses` 아래에 포함

그래서 한국어 `판매관리비`를 영어로 옮길 때는 문맥에 따라 아래처럼 쓰는 편이 안전하다.

- 일반 번역: `selling, general and administrative expenses (SG&A)`
- 세부 번역: `selling expenses`, `general and administrative expenses`

## 3. 자본, 자산, 부채, 영업이익 영어

가장 기본 대응은 아래와 같다.

| 한국어 | 영어 | 메모 |
|---|---|---|
| 자본 | equity | 문맥에 따라 shareholders' equity, owners' equity |
| 자산 | assets | 재무상태표의 자산 |
| 부채 | liabilities | 재무상태표의 부채 |
| 영업이익 | operating income / operating profit | 둘 다 널리 사용 |

### 자본 = equity

회계에서 `자본`은 보통 `equity`다. 법인 맥락에서는 `shareholders' equity` 또는 `stockholders' equity`가 더 구체적이다.

기본 관계식:

`assets = liabilities + equity`

### 자산 = assets

`assets`는 회사가 통제하고 있고 미래 경제적 효익이 기대되는 자원이다. 현금, 재고, 건물, 설비, 무형자산 등이 모두 들어간다.

### 부채 = liabilities

`liabilities`는 미래에 현금이나 서비스, 자산 이전 등으로 갚아야 하는 현재 의무다. 차입금, 매입채무, 충당부채 등이 대표적이다.

### 영업이익 = operating income / operating profit

둘 다 많이 쓰고, 실무상 거의 같은 뜻으로 쓰이는 경우가 많다.

- 미국 공시/재무 데이터: `operating income`
- IFRS/일반 영문 회계 문맥: `operating profit`

다만 회사별 `adjusted operating income` 같은 비GAAP 지표가 섞이면 정의가 달라질 수 있으므로, 공시 본문 기준을 항상 다시 보는 것이 안전하다.

## 4. Goodwill은 무엇인가

`goodwill`은 회계에서 보통 `영업권`이라고 번역한다.

이건 단순히 "평판이 좋다"는 일반 영어 뜻이 아니라, 기업결합에서 생기는 회계 항목이다.

핵심 구조:

- 어떤 회사를 인수할 때
- 지급한 대가가
- 식별 가능한 순자산의 공정가치보다 크면
- 그 초과분을 `goodwill`로 본다

즉 `goodwill`은 대략 이런 성격이다.

- 브랜드 힘
- 고객 관계
- 조직 역량
- 시너지 기대

하지만 중요한 점은, 이런 가치가 모두 개별 자산으로 분리 인식되지는 않는다는 것이다. 그래서 남는 잔여분이 `goodwill`로 잡힌다.

실무 포인트:

- 보통 `무형자산(intangible asset)` 계열로 보지만, 식별 가능한 개별 무형자산과는 구분된다.
- 내부에서 스스로 만든 평판은 일반적으로 `goodwill`로 자산 인식하지 않는다.
- 대체로 기업 인수합병 같은 거래가 있어야 장부에 등장한다.
- 이후에는 정기적으로 손상검사(`impairment`) 이슈가 중요하다.

## 5. 한 번에 외우는 요약

- `CAPEX` = 미래 효익이 긴 자산 관련 투자성 지출
- `판매관리비` = 보통 `SG&A`
- `자본` = `equity`
- `자산` = `assets`
- `부채` = `liabilities`
- `영업이익` = `operating income` 또는 `operating profit`
- `goodwill` = 기업결합에서 발생하는 `영업권`

## 6. 자주 헷갈리는 부분

### CAPEX와 비용은 같은가

아니다. `CAPEX`는 보통 즉시 비용 처리보다는 자산으로 인식된 뒤 여러 기간에 걸쳐 비용화된다.

### 판매관리비와 operating expenses는 같은가

완전히 같다고 단정하면 위험하다. 많은 회사에서 `SG&A`를 operating expenses의 핵심 구성으로 보지만, 회사별 표시 체계에 따라 연구개발비 등 다른 항목이 함께 들어갈 수 있다.

### operating income과 EBIT는 항상 같은가

실무에서는 비슷하게 쓰일 때가 많지만 항상 완전히 같은 것은 아니다. 공시 양식과 회사 정의를 확인해야 한다.

### goodwill은 좋은 브랜드 가치 그 자체인가

부분적으로는 맞지만, 회계상으로는 인수대가와 식별 가능한 순자산 공정가치의 차이에서 생기는 잔여 개념으로 이해하는 편이 정확하다.

## 참고 URL

- Gartner Finance Glossary, SG&A 정의  
  https://www.gartner.com/en/finance/glossary/selling-general-and-administrative-sg-a-expenses
- BDC, operating expenses / SG&A 설명  
  https://www.bdc.ca/en/articles-tools/entrepreneur-toolkit/templates-business-guides/glossary/operating-sg-and-a-expenses
- IFRS, IFRS 3 Business Combinations 개요  
  https://www.ifrs.org/issued-standards/list-of-standards/ifrs-3-business-combinations/
- IFRS Discussion Paper, goodwill 정의 인용 맥락  
  https://www.ifrs.org/content/dam/ifrs/project/goodwill-and-impairment/goodwill-and-impairment-dp-march-2020.pdf
- Montana OPI School Accounting Manual, assets/liabilities/equity 기본 관계 설명  
  https://opi.mt.gov/Portals/182/Page%20Files/School%20Finance/Accounting/Guidance%20and%20Manuals/Most%20Accessed/School%20Accouting%20Manual.pdf?ver=2023-07-12-164228-767

```text
$hhd-md
$hhd-research

think hard

- capex?
- 판매관리비?
- 자본, 자산, 부채, 영업이익 영어로?
- goodwill?
```
