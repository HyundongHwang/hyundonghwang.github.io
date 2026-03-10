---
layout: post
title: 260206 pc performance optimization
comments: true
tags:
- unity
- rendering
- performance
- optimization
- windows
- gpu
- research
- guide
- analysis
---
# 🛠️ 260206 PC 성능 최적화 가이드

## 📚 목차
1. [벤치마크 앱 관련](#벤치마크-앱-관련)
2. [PC 성능 개선](#pc-성능-개선)
3. [NVIDIA 드라이버](#nvidia-드라이버)

---

## ⚡ 벤치마크 앱 관련

### ⚡ Windows 11 주요 벤치마크 솔루션

#### ⚡ 종합 벤치마크 (CPU + GPU)

**3DMark**
- 가장 인기 있는 PC 벤치마크 도구
- 전문 컴퓨터 매거진에서 CPU/GPU 성능 테스트에 사용
- 게임 성능 측정에 특화

**Geekbench**
- CPU와 GPU 모두 테스트 가능
- AR, 머신러닝 등 최신 애플리케이션 기반 테스트 포함

#### ▫️ CPU 전문

**Cinebench 2026**
- Intel/AMD x86/64 아키텍처 지원
- Nvidia Blackwell GPU, AMD 9000 시리즈 호환
- 광범위한 하드웨어 구성 테스트

#### 🎮 GPU 전문

**Unigine Heaven**
- 오래되고 검증된 그래픽 벤치마크
- 아름다운 3D 환경으로 GPU 스트레스 테스트

**PassMark PerformanceTest**
- CPU, GPU, RAM, 스토리지 등 종합 벤치마크
- 상세한 점수 및 비교 제공

**OCCT**
- 강력한 스트레스 테스트
- 실시간 전압, 클럭 속도, 온도 추적
- CPU, GPU, RAM, PSU 안정성 테스트

### 🔹 Geekbench 6 사용 방법

#### 🛠️ 설치
1. [Geekbench 공식 사이트](https://www.geekbench.com/download/windows/)에서 다운로드
2. 다운로드한 파일 더블클릭하여 설치

#### ▫️ 실행 방법

**GUI 사용 (추천)**
- 애플리케이션 실행 후 한 번의 클릭으로 모든 테스트 실행
- 가장 간단한 방법

**커맨드 라인 (Pro 버전)**
```bash
geekbench6
```
- 새 명령 프롬프트에서 실행
- 자동화 가능

#### ⚡ 벤치마크 종류
- **CPU**: 단일 코어 / 멀티 코어 성능
- **GPU**: GPU 컴퓨팅 성능 (DirectX, OpenCL 등)

---

## ⚡ PC 성능 개선

### ⚡ 초기 벤치마크 결과 분석

#### ▫️ 테스트 시스템 사양
- **CPU**: AMD Ryzen 9 9950X3D (16코어/32스레드)
- **GPU**: NVIDIA GeForce RTX 5090
- **RAM**: 128GB DDR5 3590 MT/s
- **MB**: Gigabyte B650E AORUS ELITE X AX ICE
- **OS**: Windows 11 Pro

#### ⚡ 초기 점수 (성능 저하 상태)

| 항목 | 초기 점수 | 평균 점수 | 차이 |
|------|-----------|-----------|------|
| CPU Single-Core | 2,630 | 3,363 | ⚠️ -22% |
| CPU Multi-Core | 15,466 | 20,465 | ⚠️ -24% |
| GPU OpenCL | 290,207 | 381,225 | ⚠️ -24% |

**문제점**: CPU와 GPU 모두 제 성능의 약 76%만 사용

### ⚡ 성능 개선 조치

#### ▫️ 1. Windows 전원 계획
```
설정 → 시스템 → 전원 → 전원 모드 → "최고 성능" 선택
```

#### ▫️ 2. NVIDIA 드라이버 업데이트
- 최신 드라이버 설치
- NVIDIA 제어판 → 전원 관리 모드 → "최대 성능 우선" 설정

#### 🛠️ 3. BIOS 설정 (권장)
- **PBO (Precision Boost Overdrive)** 활성화
- **EXPO/XMP 프로파일** 활성화
  - 현재 3590 MT/s → 6000MHz+ 권장
- CPU 쓰로틀링 관련 온도 제한 확인

#### ▫️ 4. 온도 관리
- CPU/GPU 쿨링 상태 점검
- HWiNFO64 등으로 온도 모니터링
- 쓰로틀링 발생 여부 확인

#### ▫️ 5. 백그라운드 프로세스
- 벤치마크 실행 전 불필요한 프로그램 종료
- 바이러스 백신 일시 중지

### 🔹 개선 후 결과

#### ⚡ GPU 벤치마크 개선

| 항목 | 이전 | 개선 후 | 향상률 |
|------|------|---------|--------|
| GPU OpenCL | 290,207 | **359,268** | ✅ +24% |
| 평균 대비 | -24% | -6% | 18%p 개선 |

**결과**: 24% 성능 향상으로 평균 점수에 근접 (최소 점수 366,551에 거의 도달)

### 🚀 추가 최적화 권장사항

1. **CPU 벤치마크 재실행**
   - 전원 설정 변경 후 CPU 성능도 개선될 것으로 예상

2. **RAM 오버클럭**
   - EXPO 프로파일로 6000MHz+ 활성화
   - DDR5 잠재력 활용

3. **지속적인 모니터링**
   - 온도가 85°C 이상이면 쿨링 개선 필요
   - 장시간 작업 시 쓰로틀링 확인

---

## 📌 NVIDIA 드라이버

### 🔹 NVIDIA 제어판 접근 방법

#### ▫️ 방법 1: 바탕화면 우클릭 (가장 쉬움)
1. 바탕화면 빈 공간에서 우클릭
2. "NVIDIA 제어판" 클릭

#### ▫️ 방법 2: 시스템 트레이
1. 작업 표시줄 오른쪽 하단 시스템 트레이 (^) 클릭
2. NVIDIA 아이콘 (초록색 눈 모양) 우클릭
3. "NVIDIA 제어판" 선택

#### ▫️ 방법 3: 시작 메뉴 검색
1. Windows 키 누르기
2. "NVIDIA 제어판" 검색
3. 결과 클릭

### 🛠️ 전원 설정 변경

NVIDIA 제어판 경로:
```
3D 설정 관리
 → 전역 설정(Global Settings)
  → 전원 관리 모드(Power management mode)
   → "최대 성능 우선(Prefer maximum performance)" 선택
    → 적용(Apply)
```

### ⚖️ Game Ready vs Studio 드라이버

#### ⚖️ 비교표

| 항목 | Game Ready | Studio | Unity 개발 |
|------|------------|---------|-----------|
| 업데이트 빈도 | 월 2~4회 | 분기당 1~2회 | Studio ✅ |
| 안정성 | 보통 | 높음 | Studio ✅ |
| 최신 게임 최적화 | ✅ 우수 | 보통 | - |
| Unity/Unreal 최적화 | ✅ 지원 | ✅✅ 우수 | Studio ✅ |
| 장기 프로젝트 | 변수 있음 | 안정적 | Studio ✅ |
| 3D 모델링/렌더링 | 보통 | ✅ 최적화 | Studio ✅ |

#### ▫️ Unity 개발자에게 Studio 드라이버 권장 이유

1. **안정성 우선**
   - 몇 주~몇 달 걸리는 프로젝트에서 안정성 보장
   - 빌드 안정성 향상

2. **Unity/Unreal 엔진 가속**
   - 게임 엔진 최적화 포함
   - 뷰포트 성능, 라이팅 베이킹 개선

3. **적은 업데이트 빈도**
   - 작업 흐름 방해 최소화
   - 테스트된 안정적인 버전 제공

4. **3D 작업 도구 호환성**
   - Blender, Maya 등과 함께 사용 시 최적화

#### ▫️ 드라이버 전환

**GeForce Experience에서 간편 전환 가능**
- 평소: Studio 드라이버 (개발 작업)
- 최신 게임 플레이 시: Game Ready로 전환
- 클릭 몇 번으로 전환 가능

### 🛠️ Studio 드라이버 설치

1. [NVIDIA Studio 드라이버 다운로드](https://www.nvidia.com/Download/index.aspx)
2. "제품 유형" → **Studio** 선택
3. GPU 모델 선택 (RTX 5090)
4. 다운로드 후 설치
5. 설치 시 "사용자 정의" → "새로 설치" 체크

---

## 🔗 참고 자료

### ⚡ 벤치마크 관련
- [3DMark](https://benchmarks.ul.com/3dmark)
- [Geekbench](https://www.geekbench.com/)
- [Cinebench](https://www.techspot.com/downloads/7820-cinebench-r26.html)

### 🔹 NVIDIA 드라이버
- [NVIDIA 드라이버 다운로드](https://www.nvidia.com/Download/index.aspx)
- [GeForce Experience](https://www.nvidia.com/geforce/geforce-experience/)

### ⚡ 성능 모니터링
- [HWiNFO64](https://www.hwinfo.com/)
- [MSI Afterburner](https://www.msi.com/Landing/afterburner)

---

## 📌 요약

### 🔹 핵심 조치사항
1. ✅ NVIDIA 드라이버 최신 버전 설치
2. ✅ NVIDIA 제어판에서 전원 "최대 성능" 설정
3. ✅ Windows 전원 계획 "최고 성능" 설정
4. 🔄 Unity 개발자는 Studio 드라이버 사용
5. 🔄 BIOS에서 EXPO/PBO 활성화 권장
6. 🔄 CPU 벤치마크 재실행으로 개선 확인

### 🔹 달성한 성과
- GPU 성능 24% 향상 (290K → 359K OpenCL)
- 평균 점수 대비 -24% → -6%로 개선
- 최소 기준 점수에 근접

### 🔹 다음 단계
1. CPU 벤치마크 재실행
2. BIOS 설정 최적화
3. 장기적 안정성 모니터링

---

*작성일: 2026년 2월 6일*
*시스템: Ryzen 9 9950X3D + RTX 5090 + 128GB DDR5*
