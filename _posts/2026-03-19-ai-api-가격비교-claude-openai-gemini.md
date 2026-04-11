---
layout: post
title: "260319 AI API 가격 비교 (2026년 3월 기준)"
comments: true
tags:
- ai
- api
- 가격비교
- claude
- openai
---

# AI API 가격 비교 (2026년 3월 기준)

> 각 회사별 주요 LLM API 모델의 토큰 단가 비교. Claude, OpenAI, Gemini, Grok, Qwen 5개사를 중심으로 정리.

---

## 1. Claude (Anthropic)

| 모델 | 입력 ($/1M 토큰) | 출력 ($/1M 토큰) | 캐시 쓰기 | 캐시 읽기 |
|------|:---:|:---:|:---:|:---:|
| **Opus 4.6** | $5.00 | $25.00 | $6.25 | $0.50 |
| **Sonnet 4.6** | $3.00 | $15.00 | $3.75 | $0.30 |
| **Haiku 4.5** | $1.00 | $5.00 | $1.25 | $0.10 |
| Opus 4.1 (레거시) | $15.00 | $75.00 | $18.75 | $1.50 |
| Sonnet 4 (레거시) | $3.00 | $15.00 | $3.75 | $0.30 |
| Haiku 3 (레거시) | $0.25 | $1.25 | $0.30 | $0.03 |

- Batch 처리 시 **50% 할인**
- 출처: https://claude.com/pricing#api

---

## 2. OpenAI

### 주력 모델 (Flagship)

| 모델 | 입력 | 캐시 입력 | 출력 |
|------|:---:|:---:|:---:|
| **gpt-5.4** | $2.50 | $0.25 | $15.00 |
| **gpt-5.4-mini** | $0.75 | $0.075 | $4.50 |
| **gpt-5.4-nano** | $0.20 | $0.02 | $1.25 |
| **gpt-5.4-pro** | $30.00 | — | $180.00 |

### 파인튜닝 가능 모델

| 모델 | 입력 | 출력 |
|------|:---:|:---:|
| gpt-4.1 | $3.00 | $12.00 |
| gpt-4.1-mini | $0.80 | $3.20 |
| gpt-4.1-nano | $0.20 | $0.80 |
| gpt-4o (레거시) | $3.75 | $15.00 |
| gpt-4o-mini (레거시) | $0.30 | $1.20 |

### 추론 / 전문 모델

| 모델 | 입력 | 출력 |
|------|:---:|:---:|
| o4-mini (deep research) | $2.00 | $8.00 |
| o3 (deep research) | $10.00 | $40.00 |

- Batch / Flex 처리 시 **50% 할인**
- 출처: https://platform.openai.com/docs/pricing

---

## 3. Gemini (Google Cloud)

### Gemini 2.5 (Standard)

| 모델 | 입력 (≤200K) | 출력 | 비고 |
|------|:---:|:---:|------|
| **Gemini 2.5 Pro** | $1.25 | $10.00 | >200K: 입력 $2.50, 출력 $15 |
| **Gemini 2.5 Flash** | $0.30 | $2.50 | — |
| **Gemini 2.5 Flash Lite** | $0.10 | $0.40 | — |

### Gemini 2.0

| 모델 | 입력 | 출력 |
|------|:---:|:---:|
| Gemini 2.0 Flash | $0.15 | $0.60 |
| Gemini 2.0 Flash Lite | $0.075 | $0.30 |

### Gemini 3 (최신 Preview)

| 모델 | 입력 (≤200K) | 출력 |
|------|:---:|:---:|
| Gemini 3.1 Pro Preview | $2.00 | $12.00 |
| Gemini 3 Flash Preview | $0.50 | $3.00 |
| Gemini 3.1 Flash-Lite Preview | $0.25 | $1.50 |

- Batch / Flex 처리 시 **~50% 할인**
- 출처: https://cloud.google.com/vertex-ai/generative-ai/pricing

---

## 4. Grok (xAI)

| 모델 | 입력 | 캐시 입력 | 출력 |
|------|:---:|:---:|:---:|
| **grok-4** | $3.00 | $0.75 | $15.00 |
| **grok-4-fast** | $0.20 | $0.05 | $0.50 |
| **grok-4-1-fast** | $0.20 | $0.05 | $0.50 |
| **grok-4.20-beta** | $2.00 | $0.20 | $6.00 |
| **grok-3** | $3.00 | $0.75 | $15.00 |
| **grok-3-mini** | $0.30 | $0.07 | $0.50 |
| **grok-code-fast-1** | $0.20 | $0.02 | $1.50 |

- Batch 처리 시 **50% 할인**
- 출처: https://docs.x.ai/docs/models

---

## 5. Qwen (Alibaba — OpenRouter 경유)

| 모델 | 입력 | 출력 | 아키텍처 |
|------|:---:|:---:|------|
| **Qwen3-235B-A22B** | $0.455 | $1.82 | MoE 235B / 22B active |
| **Qwen3 Coder 480B-A35B** | $0.22 | $1.00 | MoE 480B / 35B active |
| **Qwen3-32B** | $0.08 | $0.24 | Dense 32B |
| **Qwen3-30B-A3B** | $0.08 | $0.28 | MoE 30B / 3B active |
| **Qwen3-8B** | $0.05 | $0.40 | Dense 8B |

- Alibaba Cloud DashScope 직접 사용 시 더 저렴할 수 있음
- 출처: https://openrouter.ai (OpenRouter 경유)

---

## 한눈에 비교: 가성비 순위

### 입력 토큰 ($/1M 기준)

| 순위 | 모델 | 입력 |
|:---:|------|:---:|
| 1 | Qwen3-8B | $0.05 |
| 2 | Qwen3-32B / 30B | $0.08 |
| 3 | Gemini 2.0 Flash Lite | $0.075 |
| 4 | Gemini 2.0 Flash | $0.15 |
| 5 | grok-4-fast / 4.20-beta | $0.20 |
| 6 | gpt-5.4-nano | $0.20 |
| 7 | Gemini 2.5 Flash Lite | $0.10 |
| 8 | Claude Haiku 3 | $0.25 |
| 9 | Gemini 2.5 Flash | $0.30 |
| 10 | Gemini 3.1 Flash-Lite | $0.25 |

### 출력 토큰 ($/1M 기준)

| 순위 | 모델 | 출력 |
|:---:|------|:---:|
| 1 | Qwen3-32B | $0.24 |
| 2 | Qwen3-30B-A3B | $0.28 |
| 3 | Gemini 2.0 Flash Lite | $0.30 |
| 4 | Claude Haiku 3 | $1.25 |
| 5 | grok-4-fast | $0.50 |
| 6 | Qwen3-8B | $0.40 |
| 7 | Gemini 2.5 Flash Lite | $0.40 |
| 8 | Gemini 2.0 Flash | $0.60 |
| 9 | Gemini 2.5 Flash | $2.50 |
| 10 | Claude Haiku 4.5 | $5.00 |

---

## 종합 요약

| 구분 | 추천 모델 |
|------|----------|
| **최고 성능** | gpt-5.4-pro, grok-4, Claude Opus 4.6 |
| **가성비 최적** | Gemini 2.5 Flash Lite, grok-4-fast |
| **오픈소스 무료** | Qwen3 시리즈 (로컬 배포 가능) |
| **저렴한 입력** | Qwen3-8B ($0.05/M), Gemini 2.0 Flash Lite ($0.075/M) |
| **저렴한 출력** | Qwen3-32B ($0.24/M), Gemini 2.0 Flash Lite ($0.30/M) |

---

## 출처

- Anthropic API Pricing: https://claude.com/pricing#api
- OpenAI API Pricing: https://platform.openai.com/docs/pricing
- Google Cloud Vertex AI Pricing: https://cloud.google.com/vertex-ai/generative-ai/pricing
- xAI Grok API Models: https://docs.x.ai/docs/models
- Qwen via OpenRouter: https://openrouter.ai

---

## 프롬프트

```
api 를 사용할때 가격 비교 요청
각회사별 세부 모델도 가격 비교

claude
openai
gemini
grok
qwen
```
