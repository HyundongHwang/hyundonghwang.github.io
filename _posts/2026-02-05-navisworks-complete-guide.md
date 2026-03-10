---
layout: post
title: 260205 navisworks complete guide
comments: true
tags:
- autodesk
- revit
- bim
- mep
- navisworks
- rendering
- routing
- research
- guide
---
# 🛠️ 260205 Navisworks 완벽 사용 가이드

> 💡 BIM 프로젝트 검토와 충돌 검사를 위한 필수 도구

---

## 📚 목차
1. [Navisworks 소개](#navisworks-소개)
2. [파일 형식 이해 (NWC, NWF, NWD)](#파일-형식-이해)
3. [모델 로드 및 기본 워크플로우](#모델-로드-및-기본-워크플로우)
4. [주요 기능 소개](#주요-기능-소개)
5. [주요 윈도우 및 UI](#주요-윈도우-및-ui)
6. [필수 단축키](#필수-단축키)

---

## 🧭 Navisworks 소개

**Navisworks**는 Autodesk에서 제공하는 3D 디자인 리뷰 및 조정 소프트웨어입니다. 주로 건축, 엔지니어링, 건설(AEC) 산업에서 Revit, AutoCAD, MicroStation 등의 설계 도구와 함께 사용됩니다.

### ✨ 주요 특징
- 다양한 형식의 3D 모델을 하나로 통합하여 검토
- **충돌 검사(Clash Detection)** 를 통한 설계 오류 사전 발견
- **4D 시뮬레이션** 으로 시공 일정 시각화
- 실시간 3D 탐색 및 협업 지원

### 🔹 Navisworks 제품군

```
┌─────────────────────────────────────────────────────────────────┐
│                    Navisworks 제품 비교                          │
├──────────────────┬──────────────────┬───────────────────────────┤
│   Freedom (무료)  │    Simulate      │        Manage             │
├──────────────────┼──────────────────┼───────────────────────────┤
│ • NWD 뷰어        │ • 모델 통합       │ • 모델 통합               │
│ • 3D 탐색        │ • 4D 시뮬레이션   │ • 4D 시뮬레이션           │
│ • 뷰포인트 검토   │ • 애니메이션      │ • 애니메이션              │
│                  │ • 렌더링          │ • 렌더링                  │
│                  │                  │ • ✅ Clash Detection      │
└──────────────────┴──────────────────┴───────────────────────────┘
```

> 📌 **Clash Detection**은 **Navisworks Manage** 에서만 사용 가능합니다.

---

## 🗂️ 파일 형식 이해

Navisworks의 세 가지 핵심 파일 형식을 이해하는 것이 중요합니다.

### 🗂️ 파일 형식 비교표

| 형식 | 이름 | 용도 | 특징 |
|:---:|:---|:---|:---|
| **NWC** | Cache File | 캐시/빠른 로딩 | 자동 생성, 읽기 전용 |
| **NWF** | Federated File | 협업/조정 작업 | 원본 파일 참조, 수정 가능 |
| **NWD** | Document File | 최종 배포/아카이빙 | 독립 실행, 읽기 전용 |

### 🗂️ 각 파일 형식 상세 설명

```
                     Navisworks 파일 워크플로우

    ┌─────────────┐
    │ Revit/CAD   │ ──────┐
    │ 원본 파일    │       │
    └─────────────┘       │     자동 변환
                          ▼
    ┌─────────────┐    ┌─────────────┐
    │  .rvt       │───▶│    .NWC     │ ← 캐시 파일 (자동 생성)
    │  .dwg       │    │  Cache File │
    └─────────────┘    └──────┬──────┘
                              │
                              │ 참조
                              ▼
                       ┌─────────────┐
                       │    .NWF     │ ← 작업 파일 (협업용)
                       │ Federated   │   • 충돌 검사 결과
                       │    File     │   • 뷰포인트
                       └──────┬──────┘   • 코멘트/마크업
                              │
                              │ 패키징
                              ▼
                       ┌─────────────┐
                       │    .NWD     │ ← 배포 파일 (최종본)
                       │  Document   │   • 독립 실행 가능
                       │    File     │   • 비밀번호 보호 가능
                       └─────────────┘
```

#### ▫️ NWC (Cache File)
- Revit, AutoCAD 파일을 열 때 **자동 생성**
- 다음 로드 시 빠른 로딩 지원
- 사용자가 직접 저장 불가

#### ▫️ NWF (Federated File)
- **협업 작업의 핵심 파일**
- 원본 CAD 파일의 **참조(포인터)** 만 저장
- 충돌 검사, 뷰포인트, 코멘트 등 모든 조정 데이터 포함
- 파일 크기가 매우 작음 (수 MB)
- 열 때마다 최신 NWC를 자동 갱신

#### ▫️ NWD (Document File)
- 모든 데이터를 하나의 파일로 **패키징**
- 원본 파일 없이 독립 실행 가능
- **Navisworks Freedom** 에서 열기 가능
- 비밀번호 보호, 만료일 설정 가능

---

## 📌 모델 로드 및 기본 워크플로우

### 🔹 모델 로드 방법

```
┌────────────────────────────────────────────────────────────────┐
│                    모델 로드 옵션                                │
├────────────────┬───────────────────────────────────────────────┤
│   Open         │ 새 세션에서 파일 열기                          │
├────────────────┼───────────────────────────────────────────────┤
│   Append       │ 현재 모델에 파일 추가 (★ 가장 많이 사용)        │
├────────────────┼───────────────────────────────────────────────┤
│   Merge        │ 파일 병합 (좌표계 조정 가능)                    │
├────────────────┼───────────────────────────────────────────────┤
│   Insert       │ 위치 지정하여 삽입                             │
└────────────────┴───────────────────────────────────────────────┘
```

### 🧠 충돌 검사(Clash Detection) 워크플로우

BIM 프로젝트에서 가장 중요한 워크플로우입니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                 Clash Detection 프로세스                         │
└─────────────────────────────────────────────────────────────────┘

     ① 모델 수집          ② 모델 통합         ③ Selection Set
    ┌─────────┐         ┌─────────┐         ┌─────────┐
    │ 건축    │         │         │         │ 구조 Set │
    │ 구조    │ ──────▶ │   NWF   │ ──────▶ │ 기계 Set │
    │ MEP    │         │  통합    │         │ 전기 Set │
    └─────────┘         └─────────┘         └─────────┘
                                                  │
     ┌────────────────────────────────────────────┘
     │
     ▼
     ④ 충돌 테스트 설정    ⑤ 충돌 검사 실행    ⑥ 결과 검토/보고서
    ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
    │ Selection A │     │             │     │ • 충돌 목록  │
    │     vs      │────▶│  Run Test   │────▶│ • 심각도    │
    │ Selection B │     │             │     │ • 리포트    │
    └─────────────┘     └─────────────┘     └─────────────┘
```

### 🛠️ 충돌 검사 단계별 가이드

1. **Selection Set 생성**
   - Selection Tree에서 검사할 요소 선택
   - Sets 탭에서 Selection Set으로 저장

2. **Clash Detective 실행** (`Ctrl+F2`)
   - Selection A: 첫 번째 검사 대상 (예: 구조)
   - Selection B: 두 번째 검사 대상 (예: 기계설비)

3. **충돌 유형 선택**
   - **Hard Clash**: 물리적 간섭
   - **Clearance**: 최소 이격거리 위반
   - **Duplicates**: 중복 객체

4. **테스트 실행 및 결과 검토**
   - 충돌 항목별 상태 관리 (New, Active, Resolved 등)
   - 보고서 생성 및 공유

---

## 🧭 주요 기능 소개

### 🧠 1. Clash Detective (충돌 검출)

> 💡 **가장 중요한 기능** - 설계 단계에서 충돌을 미리 발견

```
┌──────────────────────────────────────────────────────────────┐
│                    Clash Detective                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   충돌 유형:                                                  │
│   ├── Hard Clash      : 객체 간 물리적 간섭                   │
│   ├── Clearance       : 최소 이격거리 미확보                  │
│   └── Duplicates      : 동일 위치 중복 객체                   │
│                                                              │
│   충돌 상태:                                                  │
│   ├── New             : 새로 발견된 충돌                      │
│   ├── Active          : 검토 중인 충돌                        │
│   ├── Reviewed        : 검토 완료                             │
│   ├── Approved        : 승인됨 (허용 가능한 충돌)             │
│   └── Resolved        : 해결됨                                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 🔹 2. TimeLiner (4D 시뮬레이션)

프로젝트 일정과 3D 모델을 연결하여 **시공 시뮬레이션**을 생성합니다.

```
┌──────────────────────────────────────────────────────────────┐
│                      TimeLiner                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   주요 기능:                                                  │
│   • 일정 데이터 가져오기 (MS Project, Primavera)              │
│   • 작업별 3D 객체 연결                                       │
│   • 시공 순서 시각화                                          │
│   • 진척도 표시 (실제 vs 계획)                                │
│                                                              │
│   작업 유형 (Task Type):                                      │
│   ├── Construct       : 시공 (나타남)                         │
│   ├── Demolish        : 철거 (사라짐)                         │
│   └── Temporary       : 임시 (나타났다 사라짐)                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 🔹 3. Animator (애니메이션)

객체의 움직임을 애니메이션으로 표현합니다.

- 문 열림/닫힘 동작
- 크레인 회전
- 장비 이동 경로
- TimeLiner와 연동하여 4D 시뮬레이션 강화

### 🔹 4. Presenter (프레젠테이션)

- 재질(Material) 적용
- 조명(Lighting) 설정
- 렌더링 이미지/영상 생성

### 🔹 5. Quantification (물량 산출)

- 모델 기반 물량 산출
- Excel 내보내기 지원
- 비용 추정 데이터 생성

---

## 📌 주요 윈도우 및 UI

### 🏗️ Navisworks 인터페이스 구조

```
┌────────────────────────────────────────────────────────────────┐
│  Quick Access Toolbar                              [- □ X]    │
├────────────────────────────────────────────────────────────────┤
│  Home │ Viewpoint │ Review │ Animation │ View │ Output │ ...  │  ← Ribbon
├────────────────────────────────────────────────────────────────┤
│                    │                                          │
│   Selection Tree   │                                          │
│   ┌────────────┐  │           Scene View                      │
│   │ ▼ Model    │  │         (3D Viewport)                     │
│   │   ├ Arch   │  │                                          │
│   │   ├ Struct │  │              ┌─────────┐                 │
│   │   └ MEP    │  │              │ 3D Model│                 │
│   └────────────┘  │              └─────────┘                 │
│                    │                                          │
│   Properties      │                                          │
│   ┌────────────┐  │                                          │
│   │ Category   │  │                                          │
│   │ ├ Name     │  │                                          │
│   │ └ Value    │  │                                          │
│   └────────────┘  │                                          │
├────────────────────┴──────────────────────────────────────────┤
│                    TimeLiner / Clash Detective                │
└────────────────────────────────────────────────────────────────┘
```

### 🔹 주요 윈도우 설명

| 윈도우 | 단축키 | 설명 |
|:---|:---:|:---|
| **Selection Tree** | F12 | 모델 계층 구조 탐색 및 객체 선택 |
| **Properties** | Shift+F7 | 선택된 객체의 속성 정보 표시 |
| **Clash Detective** | Ctrl+F2 | 충돌 검사 설정 및 결과 |
| **TimeLiner** | Ctrl+F3 | 4D 시뮬레이션 설정 |
| **Presenter** | Ctrl+F4 | 재질 및 렌더링 설정 |
| **Animator** | Ctrl+F5 | 애니메이션 설정 |
| **Find Items** | Ctrl+F | 객체 검색 |
| **Saved Viewpoints** | - | 저장된 뷰포인트 관리 |

### 🔹 Selection Tree 표시 모드

```
┌─────────────────────────────────────────────────────────────────┐
│  Selection Tree 표시 옵션                                       │
├─────────────────┬───────────────────────────────────────────────┤
│  Standard       │  기본 계층 구조 (인스턴스 포함)                │
│  Compact        │  단순화된 계층 구조                           │
│  Properties     │  속성 기반 계층 구조                          │
│  Sets           │  Selection/Search Set 목록                   │
└─────────────────┴───────────────────────────────────────────────┘
```

### 🔹 Quick Properties 활성화

마우스 오버 시 객체 정보를 빠르게 확인하려면:

`Options > Interface > Quick Properties > Show Quick Properties` 활성화

---

## 📌 필수 단축키

### 🔹 탐색 모드 전환

| 단축키 | 기능 |
|:---:|:---|
| `Ctrl+0` | Turntable 모드 |
| `Ctrl+1` | Select 모드 (선택) |
| `Ctrl+2` | Walk 모드 (걷기) |
| `Ctrl+3` | Look Around 모드 (둘러보기) |
| `Ctrl+4` | Zoom 모드 |
| `Ctrl+5` | Zoom Box 모드 |
| `Ctrl+6` | Pan 모드 |
| `Ctrl+7` | Orbit 모드 |
| `Ctrl+8` | Examine 모드 (자유 회전) |
| `Ctrl+9` | Fly 모드 |

### 🗂️ 파일 및 모델 조작

| 단축키 | 기능 |
|:---:|:---|
| `Ctrl+A` | Append 대화상자 (파일 추가) |
| `Ctrl+M` | Merge 대화상자 (파일 병합) |
| `Ctrl+I` | Insert 대화상자 (파일 삽입) |
| `Ctrl+F` | Quick Find (빠른 검색) |
| `Ctrl+H` | 선택 항목 숨기기 토글 |
| `Ctrl+R` | Required 모드 토글 |

### 🔹 뷰 조작

| 단축키 | 기능 |
|:---:|:---|
| `Home` | Home View로 이동 |
| `PgUp/PgDn` | 확대/축소 |
| `F` | Fit All (전체 보기) |
| `Z` | Zoom Selected (선택 항목 확대) |
| `Esc` | 선택 해제 |

### 🧠 충돌 및 시뮬레이션

| 단축키 | 기능 |
|:---:|:---|
| `Ctrl+D` | Collision 모드 토글 |
| `Ctrl+G` | Gravity 모드 토글 |
| `Ctrl+T` | Third Person 모드 토글 |

### 🔹 윈도우 열기

| 단축키 | 기능 |
|:---:|:---|
| `Ctrl+F1` | 도움말 |
| `Ctrl+F2` | **Clash Detective** |
| `Ctrl+F3` | **TimeLiner** |
| `Ctrl+F4` | Presenter |
| `Ctrl+F5` | Animator |
| `Shift+F7` | Properties |
| `F12` | Selection Tree |

### 🔹 실무자 추천 TOP 10 단축키

```
┌────────────────────────────────────────────────────────────────┐
│                   ⭐ 실무 필수 단축키 TOP 10                    │
├─────────────┬──────────────────────────────────────────────────┤
│  Ctrl+F2    │  Clash Detective 열기 (충돌 검사)                │
│  Ctrl+F3    │  TimeLiner 열기 (4D 시뮬레이션)                  │
│  Ctrl+1     │  Select 모드 (가장 많이 사용)                    │
│  Ctrl+7     │  Orbit 모드 (모델 회전)                          │
│  Ctrl+A     │  Append (모델 추가)                              │
│  Ctrl+H     │  Hide/Unhide (숨기기 토글)                       │
│  Home       │  Home View (초기 뷰로 복귀)                      │
│  F          │  Fit All (전체 화면에 맞추기)                    │
│  Z          │  Zoom to Selected (선택 항목 확대)               │
│  Esc        │  Deselect (선택 해제)                            │
└─────────────┴──────────────────────────────────────────────────┘
```

---

## 🔗 참고 자료

### 🔹 공식 문서
- [Navisworks 2024 도움말 (한국어)](https://help.autodesk.com/view/NAV/2024/KOR/)
- [Navisworks 2025 도움말](https://help.autodesk.com/view/NAV/2025/KOR/)
- [Autodesk Navisworks PDF 가이드 (한국어)](https://damassets.autodesk.net/content/dam/autodesk/docs/pdfs/final-naviswork-eguide-ko.pdf)
- [Default Keyboard Shortcuts (2024)](https://help.autodesk.com/view/NAV/2024/ENU/?guid=GUID-C8B9A9A2-31A9-4D29-A578-C927435D13BF)

### 🛠️ 튜토리얼 및 가이드
- [나비스웍스 완벽 가이드 시리즈](https://dotoryworld.com/19)
- [Navisworks 파일 형식 완벽 정리](https://dotoryworld.com/53)
- [Clash Detection in Navisworks - oceanBIM](https://blog.oceanbim.com/clash-detection-in-navisworks/)
- [A Complete Guide To Navisworks Clash Detection](https://interscaleedu.com/en/blog/bim/navisworks-clash-detection-guide/)
- [Navisworks Keyboard Shortcuts - Symetri](https://www.symetri.ie/discover/blog/navisworks-keyboard-shortcuts/)

### 🔹 추가 학습
- [Why you should be Using TimeLiner in Navisworks](https://ukcommunity.arkance.world/hc/en-us/articles/24873267030546-Why-you-should-be-Using-Timeliner-in-Navisworks)
- [30 Navisworks Tips & Shortcuts - Viatechnik](https://www.viatechnik.com/insights/blog/30-navisworks-tips-shortcuts/)
- [Selection Tree Window 도움말](https://help.autodesk.com/view/NAV/2025/ENU/?guid=GUID-AF4CFA5C-1455-4444-982A-34FBA2AE4608)

---

> 📝 **작성일**: 2026-02-05
> 💡 **출처**: Autodesk 공식 문서 및 웹 리서치 기반
