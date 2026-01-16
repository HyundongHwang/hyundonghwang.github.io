---
layout: post
title: 260115 CCD Raw에서 JPG까지 고속처리 파이프라인
comments: true
tags:
- tag0
- tag1
- tag2
---


# 🎯 개요

CCD 센서의 Raw 데이터를 고속으로 JPEG로 변환하여 SSD에 저장하는 완전한 파이프라인 분석

**핵심 목표**

- 4K 30fps 실시간 처리 달성
- CPU 부하 최소화
- 메모리 대역폭 최적화

---

# 📊 전체 파이프라인 아키텍처

```
┌─────────┐   ┌─────┐   ┌──────────────┐   ┌─────┐   ┌────────┐   ┌─────┐
│   CCD   │──▶│ ISP │──▶│ JPEG Encoder │──▶│ DMA │──▶│ Memory │──▶│ SSD │
│  센서    │   └─────┘   └──────────────┘   └─────┘   └────────┘   └─────┘
└─────────┘      │              │              │          │
                 ▼              ▼              ▼          ▼
              SIMD 병렬      DCT in Chip    Zero-Copy   저장완료
               픽셀처리       하드웨어        DMA전송
```

**처리 단계**

1. CCD 센서: Raw 데이터 생성
2. ISP: 이미지 신호 처리 (SIMD 최적화)
3. JPEG Encoder: 압축 (하드웨어 DCT)
4. DMA: 직접 메모리 전송
5. SSD: 최종 저장

---

# 🚀 핵심 기술 #1: DMA (Direct Memory Access)

**역할**

- CPU 개입 없이 데이터를 메모리 간 직접 전송
- 센서 → 메모리, ISP → 메모리, 메모리 → SSD 모든 단계에서 사용

**성능 비교**

```
❌ 전통적 방식
센서 → CPU → 메모리 (5GB/s 대역폭 소비)

✅ DMA 방식  
센서 ────────→ 메모리 (2GB/s 대역폭 소비)
```

**효과**

- 메모리 대역폭 60% 절약
- CPU 부하 90% 감소
- 실시간 처리 가능

---

# ⚡ 핵심 기술 #2: Zero-Copy (No Memory Copy)

**역할**

- DMA 버퍼를 직접 공유하여 중복 복사 제거
- GPU/ISP가 같은 물리 메모리 풀 사용

**파이프라인 구현 예시 (Raspberry Pi)**

```c
// DRM PRIME 지원 - DMA 버퍼 파일 디스크립터 직접 전달

Camera Driver (DMA buf) 
       ↓ (같은 버퍼)
ISP (same buf) 
       ↓ (같은 버퍼)
JPEG Encoder (same buf)
```

**효과**

- 메모리 복사 0회 달성
- 레이턴시 감소
- 메모리 대역폭 추가 절약

---

# 🔄 핵심 기술 #3: SIMD (Single Instruction Multiple Data)

**역할**

- 픽셀 병렬 처리: 한 명령어로 8~16픽셀 동시 처리
- ISP 내부 파이프라인에서 활용

**적용 사례**

```
노이즈 제거:
[pixel1, pixel2, pixel3, ..., pixel8] ← 동시 처리 (SIMD)

색상 보정:
[R1,G1,B1, R2,G2,B2, R3,G3,B3, ...] ← 동시 변환 (ARM NEON)
```

**효과**

- ISP 처리 속도 8~16배 향상
- 전력 효율 개선
- 실시간 이미지 보정 가능

---

# 🔧 핵심 기술 #4: DCT in Chip (하드웨어 DCT)

**역할**

- JPEG 압축의 핵심인 DCT 연산을 전용 하드웨어로 처리
- 소프트웨어보다 10~100배 빠름

**하드웨어 구조**

```
┌─────────────────────────────────┐
│     JPEG Encoder Chip           │
│                                 │
│  ┌───────────────────────────┐  │
│  │  2D-DCT Pipeline          │  │  ← 전용 하드웨어
│  │  (8x8 블록 병렬 처리)        │  │
│  └───────────────────────────┘  │
│           ↓                     │
│  ┌───────────────────────────┐  │
│  │  Quantization             │  │
│  └───────────────────────────┘  │
│           ↓                     │
│  ┌───────────────────────────┐  │
│  │  Huffman Encoding         │  │
│  └───────────────────────────┘  │
│                                 │
└─────────────────────────────────┘
```

**SIMD 최적화 (libjpeg-turbo)**

- DCT/IDCT에 SIMD 명령어 적용
- ESP32-S3: SIMD로 JPEG 인코딩 가속

---

# 🏗️ 보조 기술 #1: 슈퍼스칼라 (Superscalar)

**역할**

- CPU/프로세서 아키텍처 기술
- ISP 프로세서가 슈퍼스칼라 구조일 수 있음

**특징**

- 하나의 코어에서 여러 명령어 동시 실행
- 명령어 레벨 병렬성 (ILP) 활용

**한계**

- ISP는 주로 파이프라인 + SIMD 구조
- 범용 CPU 대비 슈퍼스칼라 효과는 제한적
- 전문화된 하드웨어가 더 효율적

---

# 🔀 보조 기술 #2: MIMD (Multiple Instruction Multiple Data)

**역할**

- 여러 코어가 독립적인 작업 수행
- GPU에서 주로 사용 (ISP는 주로 SIMD)

**적용 사례**

```
┌──────────────────┐  ┌──────────────────┐
│   ISP Core 1     │  │   ISP Core 2     │  ← MIMD
│   (좌측 영역)      │  │   (우측 영역)      │  (독립 처리)
└──────────────────┘  └──────────────────┘
         ↓                      ↓
    SIMD 병렬             SIMD 병렬         ← 각 코어 내부는 SIMD
     픽셀처리              픽셀처리
```

**특징**

- 이미지 영역별 병렬 처리
- 멀티코어 ISP/GPU에서 활용

---

# 💻 실제 구현 사례 #1: ESP32-S3

**ESP32-S3 카메라 시스템**

```
┌─────────┐
│CCD/CMOS │
└────┬────┘
     │ DMA
     ▼
┌─────────┐ Zero-Copy
│   RAM   │◄─────────┐
└────┬────┘          │
     │               │
     ▼               │
┌─────────┐          │
│   ISP   │          │
│ (SIMD)  │          │
└────┬────┘          │
     │               │
     ▼               │
┌─────────┐          │
│  JPEG   │          │
│ Encoder │──────────┘
│(HW DCT) │
└────┬────┘
     │ DMA
     ▼
┌─────────┐
│   SSD   │
└─────────┘
```

**처리 단계**

1. CCD/CMOS → DMA → RAM (Zero-Copy)
2. ISP (SIMD 최적화) 처리
    - 노이즈 제거, 색보정, 샤프닝
3. JPEG Encoder (하드웨어 DCT)
    - 2D-DCT → Quantization → Huffman
4. DMA → SSD (Zero-Copy)

---

# 💻 실제 구현 사례 #2: NVIDIA Jetson

**Jetson 통합 메모리 아키텍처**

```
Camera Driver (DMA) → ISP → GPU (Zero-Copy) → JPEG Encoder
        ↓              ↓         ↓                  ↓
    DMA 버퍼 공유   SIMD/CUDA  통합 메모리          하드웨어 DCT
```

**핵심 특징**

- 통합 메모리: CPU와 GPU가 같은 물리 메모리 공유
- Zero-Copy: 전 과정에서 메모리 복사 없음
- CUDA: GPU 가속 ISP 처리
- 하드웨어 NVENC: 고속 JPEG/비디오 인코딩

**장점**

- 초저지연 처리
- 고해상도 실시간 처리
- AI 후처리 통합 가능

---

# 📈 성능 지표 총정리

| 최적화 기술 | 성능 효과 | 적용 단계 |
| --- | --- | --- |
| DMA | CPU 부하 90% 감소 | 센서 → 메모리 → SSD |
| Zero-Copy | 메모리 대역폭 60% 절약 | 전체 파이프라인 |
| SIMD | ISP 처리 8~16배 가속 | ISP 픽셀 처리 |
| 하드웨어 DCT | JPEG 압축 10~100배 가속 | JPEG Encoder |
| 파이프라인 | 레이턴시 숨김 (overlapping) | 전체 흐름 |

**최종 결과**

4K (3840×2160) @ 30fps 실시간 처리 가능

---

# 🎯 기술 분류 및 중요도

**필수 사용 기술 (Must Have)**

1. DMA - 센서 → 메모리 → SSD 전 과정
2. Zero-Copy - 중복 복사 제거
3. SIMD - ISP 픽셀 병렬 처리
4. 하드웨어 DCT - JPEG 압축 가속

**보조 기술 (Nice to Have)**

1. 슈퍼스칼라 - ISP 프로세서 내부 구조
2. MIMD - 멀티코어 ISP/GPU

**기술 조합**

- 필수 4가지 기술 모두 적용 시: 4K 30fps 달성
- 일부만 적용 시: 성능 저하 (1080p 또는 낮은 fps)

---

# 📚 참고 자료

**공식 문서 및 기술 논문**

- [Jetson Zero Copy for Embedded applications](https://www.ximea.com/support/wiki/apis/Jetson_Zero_Copy_for_Embedded_applications)
- [ESP_NEW_JPEG with SIMD Introduction](https://developer.espressif.com/blog/2025/09/esp-new-jpeg-introduction/)
- [SIMD IDCT in libjpeg-turbo](https://ssvb.github.io/2011/08/22/simd-idct-libjpeg-turbo-bitexactness.html)

**ISP 및 카메라 파이프라인**

- [Camera ISP Pipeline Inside Look](https://www.einfochips.com/blog/a-peek-inside-your-camera-i-image-signal-processing-isp-pipeline/)
- [ROS-DMA Double Buffering (IEEE)](https://ieeexplore.ieee.org/document/1613350/)

**위키백과**

- [디지털 카메라 - 위키백과](https://ent.churchrez.org/wiki-https-ko.wikipedia.org/wiki/디지털_카메라)

---

# ✅ 핵심 요약

**파이프라인 흐름**

CCD 센서 → (DMA) → ISP (SIMD) → (Zero-Copy) → JPEG Encoder (DCT) → (DMA) → SSD

**성능 달성 요소**

- DMA: CPU 우회, 직접 전송
- Zero-Copy: 메모리 복사 제거
- SIMD: 병렬 픽셀 처리
- 하드웨어 DCT: 고속 압축

**최종 성능**

4K 30fps 실시간 처리 가능