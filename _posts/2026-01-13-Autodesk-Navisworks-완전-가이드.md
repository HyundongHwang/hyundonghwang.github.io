---
layout: post
title: 260113 Autodesk Navisworks 완전 가이드
comments: true
tags:
- tag0
- tag1
- tag2
---

# 🏗️ Autodesk Navisworks란 무엇인가?

Autodesk Navisworks는 건축, 엔지니어링, 건설(AEC) 산업을 위한 프로젝트 검토 및 조정 소프트웨어입니다. 다양한 소스에서 생성된 3D 모델을 하나의 통합된 환경에서 시각화하고, 분석하며, 조정할 수 있도록 지원합니다.

Navisworks는 Revit, AutoCAD, Tekla, MicroStation 등 **60개 이상의 파일 형식**을 지원하여 다양한 설계 소프트웨어에서 생성된 모델을 통합할 수 있습니다. 이를 통해 프로젝트 팀은 건설 전에 설계상의 문제를 발견하고 해결할 수 있습니다.

---

# 🎯 왜 Navisworks가 필요한가?

## 현대 건설 프로젝트의 복잡성

현대의 대규모 건설 프로젝트는 수십 개의 전문 분야가 협력해야 하는 복잡한 시스템입니다:

- **건축 설계팀**: 건물의 외관과 공간 배치
- **구조 엔지니어**: 하중 계산과 구조 시스템
- **MEP 전문가**: 기계, 전기, 배관 시스템
- **조경 설계사**: 외부 공간 및 지형
- **시공사**: 실제 시공 계획 및 일정

각 팀은 서로 다른 소프트웨어를 사용하여 작업하며, 이들의 설계가 물리적으로 충돌하지 않도록 조정하는 것은 매우 중요합니다.

## Navisworks가 해결하는 문제

### 1. 설계 충돌 조기 발견

건설 현장에서 발견되는 충돌은 막대한 비용과 시간 지연을 초래합니다. Navisworks는 **가상 환경에서 충돌을 미리 감지**하여 현장 재작업을 최소화합니다.

### 2. 다학제 협업

서로 다른 소프트웨어를 사용하는 팀들이 **하나의 통합 모델**에서 작업 결과를 확인하고 조정할 수 있습니다.

### 3. 시각적 의사소통

복잡한 기술 도면 대신 **3D 시각화**를 통해 건축주, 시공사, 설계팀 간의 의사소통이 원활해집니다.

### 4. 일정 및 비용 관리

4D 시뮬레이션(시간)과 5D 분석(비용)을 통해 프로젝트 전체 수명 주기를 관리할 수 있습니다.

---

# 💼 Navisworks의 주요 용도

## 1. 충돌 검사 (Clash Detection)

Navisworks의 가장 널리 알려진 기능입니다. **Clash Detective** 도구를 사용하여 다음과 같은 충돌을 자동으로 감지합니다:

- **Hard Clash**: 물리적으로 두 객체가 겹치는 경우 (예: 덕트가 보를 관통)
- **Soft Clash**: 필요한 간격이 확보되지 않은 경우 (예: 배관 주변 유지보수 공간 부족)
- **4D Clash**: 시간상 충돌 (예: 동시에 같은 공간에서 두 작업이 진행)

### 실무 사례

**스히폴 공항(네덜란드)**: Heijmans社는 여러 활주로와 유도로의 유지보수를 위해 4D 계획을 사용하여 작업 활동과 시간상 충돌을 시각화했습니다.

**Beatrix 수문 확장 프로젝트**: 운하 확장과 세 번째 수문 건설에서 4D 계획을 사용하여 다양한 건설 단계를 시각화하고, 특히 새로운 수문 게이트 설치와 같은 고위험 활동에 주의를 기울였습니다.

## 2. 모델 통합 및 검토

여러 파일 형식을 하나의 **연합 모델(Federated Model)**로 결합하여:

- 전체 프로젝트를 한눈에 파악
- 다양한 뷰포인트 생성 및 공유
- 마크업 및 주석 추가
- 이해관계자와의 효과적인 의사소통

## 3. 4D 시뮬레이션

프로젝트 일정을 3D 모델과 연결하여 **시간 경과에 따른 건설 진행 상황**을 시각화합니다:

- **TimeLiner** 도구를 사용하여 Microsoft Project, Primavera 등의 일정 데이터 가져오기
- 모델 요소를 일정 작업에 연결
- 건설 시퀀스 애니메이션 생성
- 장비 동선, 작업 공간, 현장 자재 보관 계획

## 4. 5D 비용 분석

**Quantification** 도구를 사용하여:

- 2D 도면이나 3D 모델에서 자재 수량 자동 추출
- 선형, 면적, 개수 측정
- Excel로 물량 데이터 내보내기
- 비용 견적 시스템과 통합

Navisworks는 완전한 비용 견적 플랫폼은 아니지만, **정확한 수량 데이터**를 제공하여 3D 모델(형상), 4D 일정(시간), 5D 수량(비용)을 연결합니다.

## 5. MEP 시스템 조정

기계, 전기, 배관(MEP) 시스템의 복잡한 조정에 특히 효과적입니다:

- HVAC 덕트와 구조 요소 간 충돌 확인
- 배관 시스템 간섭 검사
- 전기 케이블 트레이 경로 확인
- 유지보수 접근성 검증

전문 MEP BIM 조정 서비스 제공업체들은 Navisworks Manage와 BIM 360 Glue를 사용하여 MEP 3D BIM 모델의 조정 문제를 효과적으로 식별하고 해결합니다.

---

# ✅ Navisworks의 장점

## 1. 뛰어난 성능 최적화

Navisworks 모델은 **시각화에 최적화**되어 있어, 엔지니어링 및 설계 작업용 모델보다 훨씬 가볍습니다. 이는 **대용량 모델도 원활하게 처리**할 수 있음을 의미합니다.

원본 CAD 데이터 대비 **최대 80% 압축**되어 파일 크기가 크게 줄어듭니다.

## 2. 광범위한 파일 형식 지원

60개 이상의 파일 형식을 지원하여 다양한 소프트웨어 플랫폼을 통합할 수 있습니다:

- Autodesk (Revit, AutoCAD, Civil 3D)
- Bentley (MicroStation)
- Tekla Structures
- ArchiCAD
- SketchUp
- 그 외 다수

## 3. 강력한 충돌 검사 기능

업계 표준으로 인정받는 Clash Detective:

- 자동화된 충돌 검사
- 사용자 정의 가능한 충돌 규칙
- 상세한 충돌 보고서 생성
- 충돌 상태 추적 및 관리

## 4. 클라우드 협업

**Autodesk Construction Cloud**와의 통합:

- 팀원들이 동시에 모델 검토 및 업데이트
- 최신 버전에서 항상 작업
- 의사소통 개선 및 조정 지연 최소화

## 5. 4D/5D 시뮬레이션

- 일정을 모델과 직접 연결
- 시간대별 예산 영향 예측
- 정시 및 비용 효율적인 인도 지원

## 6. 2026 버전의 새로운 기능

- **향상된 Clash Detective**: 충돌 그룹화 기능이 내장되어 더욱 직관적인 워크플로우 제공
- **현대화된 인터페이스**: 피벗 포인트 잠금 기능으로 복잡한 3D 모델 탐색 혁신
- **개선된 검색**: XML 형식의 사용자 정의 쿼리 생성 및 저장
- **충돌-이슈 상태 동기화**: 충돌에서 이슈를 생성할 때 상태 자동 동기화

---

# ⚠️ Navisworks의 단점

## 1. OpenBIM 워크플로우 제한

Navisworks는 **진정한 OpenBIM 워크플로우를 지원하지 않습니다**. 충돌 데이터에 접근하려면 항상 시스템에 Navisworks가 설치되어 있어야 합니다.

## 2. 라이선스 비용

### 필수 라이선스 요구사항

충돌 검사 기능은 **Navisworks Manage**에만 제공되므로, 모든 엔지니어가 전체 버전을 보유해야 합니다.

### 가격대

- 6가지 에디션: $135 ~ $8,010
- 구독 방식: 최대 3대 컴퓨터에 설치 가능하지만 한 번에 한 명만 사용 가능

## 3. 파일 기반 방법론

**클라우드 기반 서비스**의 장점을 충분히 활용하지 못합니다. 중앙 저장소로 Navisworks를 사용할 때 파일 기반 방식의 제약이 있습니다.

## 4. 제한된 상호 운용성

**Switchback 기능**은 Revit에서만 작동합니다. 다른 모델링 도구를 사용하는 팀원들은 나란히 비교하는 방식에 의존해야 합니다.

## 5. 모델 생성 불가

Navisworks는 **모델을 처음부터 생성할 수 없습니다**. Revit처럼 설계 작업을 수행할 수 없으며, 다른 저작 소프트웨어에 의존해야 합니다.

검토, 조정, 분석 도구로는 탁월하지만, 모델링 소프트웨어는 아닙니다.

## 6. 학습 곡선

강력한 기능만큼 배우기가 복잡할 수 있습니다:

- 충돌 규칙 설정
- 검색 세트 구성
- 4D 시뮬레이션 설정
- API 사용자 정의 (개발자용)

---

# 🛠️ 주요 기능 상세

## 1. Clash Detective (충돌 검사 도구)

### 충돌 유형

**Hard Clash (경성 충돌)**

두 객체가 물리적으로 겹치는 경우입니다.

- 예시: 공조 덕트가 구조 보를 관통하는 경우
- 허용 오차(Tolerance): 객체가 설정된 값(예: 0.003m) 이상 관통해야 충돌로 보고됨

**Clearance Clash (간격 충돌)**

최소 간격 요구사항이 충족되지 않은 경우입니다.

- 예시: 배관 주변 유지보수 공간 부족
- 허용 오차: 두 요소가 설정된 간격(예: 50mm) 이하일 때 충돌로 보고됨

**4D Clash (시간 충돌)**

일정상 동일한 시간에 같은 공간에서 충돌하는 경우입니다.

### 충돌 규칙 (Clash Rules)

충돌 검사 시 **거짓 양성(False Positive)**을 줄이기 위한 규칙을 설정할 수 있습니다:

**1. Items in Same File (동일 파일 내 항목)**

같은 파일 내의 항목 간 충돌은 무시됩니다. 서로 다른 파일 간의 충돌만 검색하려는 경우에 유용합니다.

**2. Items in Same Composite Object (동일 복합 객체 내 항목)**

하나의 CAD 객체 내부의 자체 교차는 무시됩니다.

- 예시: 창틀과 유리 사이의 충돌은 무시

**3. Previously Found Pair of Composite Objects (이전에 발견된 복합 객체 쌍)**

동일한 복합 객체 쌍 간의 여러 충돌은 하나의 충돌로 처리됩니다.

**4. Items in Same Layer (동일 레이어 내 항목)**

같은 레이어의 항목 간 충돌은 무시됩니다.

- 예시: Revit 파일에서 같은 레벨의 충돌 무시

### 허용 오차 (Tolerance) 설정

**일반 권장 사항**

- 일반적으로 **25mm (또는 1인치)**가 적절한 기준입니다
- 프로젝트 유형에 따라 조정 가능

**Hard Clash에서의 허용 오차**

50mm 허용 오차를 설정하면, 두 항목이 **51mm 이상 교차**해야 충돌로 보고됩니다.

**Clearance Clash에서의 허용 오차**

50mm 허용 오차를 설정하면, 두 요소가 **50mm 이하로 가까워**야 충돌로 보고됩니다.

**중요 사항**

허용 오차는 각 충돌 테스트마다 개별적으로 설정되므로, 변경 사항은 현재 선택된 테스트에만 적용됩니다.

## 2. TimeLiner (4D 시뮬레이션)

### 작동 방식

1. **일정 가져오기**: Microsoft Project, Primavera P6 등에서 일정 데이터 가져오기
2. **요소 연결**: 3D 모델 요소를 일정 작업에 연결
3. **시뮬레이션 생성**: 시간 경과에 따른 건설 진행 상황을 시각화하는 애니메이션 생성

### 실무 적용

- **장비 동선**: 대형 장비의 이동 경로 계획
- **작업 공간 확보**: 동시 작업 간섭 최소화
- **현장 자재 보관**: 자재 적치 계획 최적화
- **주요 마일스톤 표시**: 중요 단계 시각적 강조

## 3. Quantification (수량 산출)

### 기능

- **자동 수량 추출**: 모델에서 콘크리트 입방미터, 배관 선형미터, 조명 기구 개수 등 자동 산출
- **2D/3D 측정**: 도면이나 모델에서 선, 면적, 개수 측정
- **Excel 내보내기**: 물량 내역서를 Excel로 내보내어 분석 및 견적 시스템과 통합

### 통합 솔루션

- **Sage 연동**: AMC Bridge가 개발한 Autodesk Navisworks Integrator를 통한 양방향 데이터 교환
- **eTakeoff 통합**: 견적 시각화 개선을 위한 맞춤형 플러그인

---

# 📄 파일 포맷 상세 (NWD, NWC, NWF)

Navisworks는 세 가지 주요 파일 형식을 사용하며, 각각 고유한 목적과 특성이 있습니다.

## NWC (Navisworks Cache) 파일

### 특징

- **자동 생성**: 원본 파일과 같은 디렉토리에 자동으로 생성되는 **캐시 파일**
- **용도**: CAD, Revit 등에서 내보낸 3D/2D 요소를 Navisworks가 인식할 수 있는 형식으로 변환
- **내용**: 모든 형상(geometry)과 관련 객체 속성 정보 포함
- **크기**: 원본 파일보다 작아 자주 사용하는 파일의 액세스 속도 향상
- **읽기 전용**: 사용자가 파일을 저장할 수 없는 읽기 전용 파일

### 작동 방식

원본 설계 파일이 변경되지 않았다면, Navisworks는 설계 파일을 다시 변환하는 대신 **NWC 캐시에서 로드**하여 속도를 높입니다.

### 기술적 세부사항

- 원본 소스 파일의 형상 및 속성 정보 포함
- 일반 사용을 위한 파일이 아닌 내부 캐시용
- 파일 변환 시간 단축을 위한 성능 최적화

## NWD (Navisworks Data/Document) 파일

### 특징

- **스냅샷**: 모델의 현재 상태를 캡처한 **독립 실행형 파일**
- **내용**: 모든 형상, 객체 속성, 충돌 뷰포인트, 마크업, 주석, 적선(redline) 포함
- **압축**: 원본 CAD 데이터 대비 **최대 80% 압축**으로 매우 작은 파일 크기
- **독립성**: 원본 데이터가 변경되어도 업데이트되거나 재캐시되지 않음
- **보안**: 암호 보호, 만료 날짜 설정, 읽기 전용 설정 가능

### 사용 시나리오

- 프로젝트 마일스톤 기록
- 클라이언트나 외부 이해관계자와 공유
- 검토 및 승인용
- 특정 시점의 설계 동결

### 기술적 세부사항

- NWC와 거의 동일한 포맷이지만 캐시 유효성 스트림은 포함하지 않음
- Navisworks 관련 데이터(뷰포인트, 주석 등) 저장
- 독립적으로 배포 가능

## NWF (Navisworks File Format) 파일

### 특징

- **연합 파일**: 원본 파일에 대한 **링크만 포함**하고 형상은 저장하지 않음
- **내용**: 현재 로드된 파일에 대한 포인터, 환경 설정, 현재 뷰, 충돌 결과, 저장된 뷰포인트
- **크기**: NWD 파일보다 훨씬 작음 (형상 데이터 미포함)
- **업데이트**: 원본 파일의 변경사항이 **즉시 반영**됨
- **용도**: 활성 프로젝트에 이상적

### 사용 시나리오

- 진행 중인 프로젝트 조정
- 팀 협업 (중앙 파일 역할)
- 원본 파일과의 동기화 유지 필요 시

### 기술적 세부사항

- 모델 표현 스트림은 포함하지 않음
- Navisworks에서 생성되거나 편집된 메타데이터만 저장
- 참조하는 원본 파일에 대한 경로 정보 포함

## 파일 형식 비교표

```
┌─────────────┬────────────────┬────────────────┬────────────────┐
│   항목      │      NWC       │      NWD       │      NWF       │
├─────────────┼────────────────┼────────────────┼────────────────┤
│ 형상 포함   │       ✓        │       ✓        │       ✗        │
├─────────────┼────────────────┼────────────────┼────────────────┤
│ 속성 포함   │       ✓        │       ✓        │       ✗        │
├─────────────┼────────────────┼────────────────┼────────────────┤
│ 마크업      │       ✗        │       ✓        │       ✓        │
├─────────────┼────────────────┼────────────────┼────────────────┤
│ 자동 업데이트│      ✓         │       ✗        │       ✓        │
├─────────────┼────────────────┼────────────────┼────────────────┤
│ 독립 실행   │       ✗        │       ✓        │       ✗        │
├─────────────┼────────────────┼────────────────┼────────────────┤
│ 파일 크기   │     작음       │     작음       │   매우 작음    │
├─────────────┼────────────────┼────────────────┼────────────────┤
│ 주요 용도   │    캐시        │   스냅샷       │  연합 모델     │
└─────────────┴────────────────┴────────────────┴────────────────┘
```

## 파일 형식 선택 가이드

**NWC를 사용하는 경우:**

- 자동으로 생성됨 (사용자가 선택할 필요 없음)
- 내부 캐시 용도

**NWD를 사용하는 경우:**

- 프로젝트 스냅샷 생성
- 외부 공유 (클라이언트, 하청업체 등)
- 버전 관리 및 기록
- 보안이 필요한 경우

**NWF를 사용하는 경우:**

- 진행 중인 프로젝트
- 팀 협업 (중앙 파일)
- 원본 파일 변경 시 자동 업데이트 필요
- 파일 크기 최소화

---

# 🚰 유체(Utility) 타입에 대한 기능

Navisworks에서 MEP 시스템, 특히 유체를 다루는 배관 및 HVAC 시스템의 조정은 매우 중요한 기능입니다.

## MEP 시스템 조정

### 지원되는 유체 시스템

- **HVAC**: 공조 덕트, 송풍기, 에어 핸들러
- **배관**: 급수, 배수, 소화, 냉온수 배관
- **전기**: 케이블 트레이, 전선관
- **소방**: 스프링클러 시스템

### 시스템별 특성 인식

Navisworks는 모델에서 다음과 같은 속성 정보를 인식합니다:

- 시스템 유형 (냉수, 온수, 배수 등)
- 배관 직경 및 규격
- 절연 두께
- 유지보수 간격 요구사항

## 충돌 검사 사용자 정의

### 시스템별 규칙 설정

프로젝트 관리자는 각 시스템 유형에 대해 **사용자 정의 충돌 규칙**을 설정할 수 있습니다:

**예시:**

- 소화 배관과 구조 요소 간 충돌: 허용 오차 0mm (엄격)
- 일반 배관과 전기 케이블 트레이: 허용 오차 50mm
- 덕트와 구조 보: 허용 오차 25mm

### 규칙 템플릿

Navisworks는 **기본 충돌 규칙 템플릿**과 사용자 정의 템플릿을 모두 제공합니다:

- 거래별 충돌 규칙
- 허용 오차 임계값 관리
- 특정 거래에 대한 허용 가능한 간격 지정

---

# 🔄 같은 타입의 유체는 물리적 간섭이 있어도 무시하는가?

이것은 Navisworks 사용자들이 자주 묻는 질문입니다. **답은 설정에 따라 다릅니다.**

## 기본 동작

Navisworks는 **기본적으로 모든 충돌을 감지**합니다. 같은 유형의 유체 시스템 간의 간섭도 예외가 아닙니다.

예를 들어:

- 두 개의 냉수 배관이 교차하는 경우
- 같은 시스템의 덕트가 겹치는 경우

이러한 경우도 충돌로 보고됩니다.

## 충돌 규칙을 통한 제어

그러나 **Clash Rules**를 사용하여 특정 조건의 충돌을 무시하도록 설정할 수 있습니다.

### 방법 1: Items in Same Layer

Revit에서 시스템 유형별로 레이어가 분리되어 있다면, "Items in Same Layer" 규칙을 활성화하여 같은 레이어(시스템) 내의 충돌을 무시할 수 있습니다.

### 방법 2: 사용자 정의 검색 세트

더 정교한 제어를 위해:

1. **검색 세트 생성**: 시스템 유형별로 검색 세트를 만듭니다
    - 예: "냉수 배관", "온수 배관", "배수 배관"
2. **선택적 충돌 테스트**: 충돌 테스트를 생성할 때 다음을 선택합니다:
    - Selection A: 냉수 배관
    - Selection B: 다른 모든 시스템 (냉수 배관 제외)
3. **결과**: 냉수 배관 간의 간섭은 검사되지 않습니다.

### 방법 3: 속성 기반 규칙

**Search Sets**에서 속성 필터를 사용하여:

- 시스템 이름으로 필터링
- 시스템 분류로 필터링
- 사용자 정의 속성으로 필터링

## 실무 예제

### 시나리오: 절연층과 배관

**문제**: 배관과 그 절연층 사이의 충돌을 무시하려면 어떻게 해야 하나요?

**해결책**:

1. Items in Same Composite Object 규칙 활성화
2. 또는 배관과 절연층이 같은 레이어에 있다면 Items in Same Layer 규칙 사용

### 시나리오: 65mm 벤트 파이프와 32mm 냉수 배관

**예시**: 90mm 스터드 벽 내부에 65mm 벤트 파이프가 수직으로 올라가고, 32mm 냉수 배관이 벽 공동에서 수평으로 지나가는 경우

**고려사항**:

- 물리적으로 같은 공간에 있지만 실제로는 충돌하지 않을 수 있음
- 허용 오차 설정으로 미세한 충돌 무시
- 현장에서 해결 가능한지 검토 필요

---

# ⚙️ 간섭 시 Tolerance(허용 오차) 기능

허용 오차는 Navisworks 충돌 검사에서 **가장 중요한 설정 중 하나**입니다. 이를 올바르게 설정하면 의미 없는 충돌을 자동으로 필터링하여 팀의 시간을 절약할 수 있습니다.

## 허용 오차의 작동 원리

### 절대값 기준

허용 오차는 **절대값 기준의 차단값(cutoff)**으로 작동합니다. 설정된 값보다 작은 충돌은 자동으로 무시됩니다.

### Hard Clash에서의 허용 오차

**설정**: 허용 오차 = 50mm

**동작**:

- 두 객체가 50mm 이하로 겹치면: **무시됨**
- 두 객체가 51mm 이상 겹치면: **충돌로 보고됨**

**예시**:

배관이 보를 3mm 관통하는 경우, 허용 오차가 25mm로 설정되어 있다면 이 충돌은 보고되지 않습니다. 하지만 30mm 관통하면 충돌로 보고됩니다.

### Clearance Clash에서의 허용 오차

**설정**: 허용 오차 = 50mm

**동작**:

- 두 요소가 50mm보다 멀리 떨어져 있으면: **무시됨**
- 두 요소가 50mm 이하로 가까우면: **충돌로 보고됨**

**예시**:

유지보수를 위해 배관 주변에 최소 100mm 간격이 필요한 경우, 간격 충돌 테스트를 생성하고 허용 오차를 100mm로 설정합니다.

## 권장 허용 오차 값

### 일반 건물 프로젝트

- **구조 vs MEP**: 25mm (1 inch)
- **MEP 간**: 25mm ~ 50mm
- **정밀 작업**: 10mm ~ 15mm

### 산업 시설

- **대형 배관**: 50mm ~ 100mm
- **소형 배관**: 25mm
- **케이블 트레이**: 50mm

### 프로젝트 초기 vs 후기

**초기 단계 (개념 설계)**:

- 허용 오차: 50mm ~ 100mm
- 목적: 주요 충돌에만 집중

**후기 단계 (시공 도서)**:

- 허용 오차: 10mm ~ 25mm
- 목적: 모든 충돌 포착

## 허용 오차 설정 실무 팁

### 1. 단계별 접근

**1단계**: 큰 허용 오차(50mm)로 시작하여 주요 충돌 해결

**2단계**: 허용 오차를 줄여(25mm) 중간 충돌 해결

**3단계**: 작은 허용 오차(10mm)로 미세 충돌 해결

### 2. 시스템별 차별화

다양한 시스템 조합에 대해 **별도의 충돌 테스트**를 만들고 각각 적절한 허용 오차를 설정합니다:

**테스트 1**: 구조 vs HVAC

- 허용 오차: 25mm
- 이유: 덕트는 비교적 큰 요소로 미세한 겹침은 현장 조정 가능

**테스트 2**: 배관 vs 전기

- 허용 오차: 50mm
- 이유: 유연한 조정이 가능한 시스템들

**테스트 3**: 구조 vs 구조

- 허용 오차: 0mm
- 이유: 구조 요소 간 충돌은 매우 심각

### 3. 현장 실정 고려

**질문해야 할 사항**:

- 이 충돌은 현장에서 해결 가능한가?
- 시공사가 일반적으로 허용하는 오차 범위는?
- 자재의 제작 허용 오차는 얼마인가?

**예시**:

철골 제작의 일반적인 허용 오차가 ±5mm라면, 구조 충돌 검사의 허용 오차를 10mm 이하로 설정하는 것이 합리적입니다.

### 4. 문서화

**중요**: 프로젝트 BIM 실행 계획(BEP)에 허용 오차 기준을 명확히 문서화해야 합니다:

```
예시 BEP 기준:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
충돌 유형                    허용 오차
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
구조 - 구조                   0mm
구조 - HVAC                  25mm
구조 - 배관                  25mm
구조 - 전기                  50mm
HVAC - 배관                  50mm
HVAC - 전기                  50mm
배관 - 배관                  25mm
간격 검사 (유지보수)         100mm
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 허용 오차와 거짓 양성

### 거짓 양성이란?

실제로는 문제가 되지 않지만 충돌로 보고되는 경우입니다.

**예시**:

- 창틀과 유리 사이의 의도적인 겹침
- 벽과 그 안의 전선관
- 파이프와 그 절연층

### 거짓 양성 최소화 전략

1. **적절한 허용 오차 설정**
2. **충돌 규칙 활용** (Items in Same Composite Object 등)
3. **검색 세트 정교화**
4. **단계적 충돌 테스트**

## 허용 오차 변경 시 주의사항

### 개별 테스트 적용

**중요**: 허용 오차는 **각 충돌 테스트마다 개별적으로 설정**됩니다.

현재 선택된 테스트의 허용 오차를 변경해도 다른 테스트에는 영향을 미치지 않습니다.

### 재실행 필요

허용 오차를 변경한 후에는 충돌 테스트를 **재실행**해야 새로운 설정이 적용됩니다.

---

# 💻 Navisworks API 및 자동화

Navisworks는 강력한 **.NET API**를 제공하여 사용자가 커스텀 플러그인을 작성하고 반복 작업을 자동화할 수 있습니다.

## API 기능

### 주요 기능

- **커스텀 플러그인 작성**: Navisworks 제품에 사용자 정의 기능 추가
- **외부에서 구동**: GUI 외부에서 Navisworks 제어
- **작업 자동화**: 반복적인 작업 자동화

### 자동화 가능한 작업

- 선택 세트 생성
- 충돌 테스트 생성
- 속성 조작
- 뷰포인트 자동 생성
- TimeLiner 작업 생성
- 검색 세트 생성
- 모델 변환
- 보고서 생성

## 개발 리소스

### 문서

- **Developer's Guide**: PDF 형식으로 제공되는 API 클래스 및 도구 안내
- **Reference Guide**: CHM 파일 형식의 상세 API 참조

### 학습 자료

- **Udemy**: "Basic PlugIn Development with Navisworks API C#" 강좌
- **Autodesk University**: "Make NavisWORK for You: Intro to the Navisworks API" 세션
- [**TwentyTwo.space**](http://TwentyTwo.space): 종합적인 애드인 개발 가이드
- **GitHub**: NavisAutomation 등 오픈소스 예제

## 실무 활용 사례

### Python 통합

Python을 사용하여 Navisworks 워크플로우를 자동화할 수 있습니다.

**예시**: 룸별 뷰포인트 자동 생성

프로젝트의 모든 룸에 대해 자동으로 뷰포인트를 생성하는 스크립트

### Dynamo 연동

Dynamo와 Navisworks 간의 상호작용을 탐구하여 시각적 프로그래밍으로 워크플로우를 구축할 수 있습니다.

### 외부 소프트웨어 통합

Inventor, Revit 등 다른 Autodesk 제품의 API에서 Navisworks를 실행하고 제어할 수 있습니다.

---

# 🏢 Navisworks 제품 라인

Autodesk는 다양한 사용자 요구를 충족하기 위해 세 가지 Navisworks 제품을 제공합니다.

## 1. Navisworks Manage

### 개요

**가장 포괄적인 버전**으로 AEC 팀이 충돌 검사와 고급 조정에 사용합니다.

### 주요 기능

- ✅ **Clash Detective**: 충돌 검사 및 간섭 확인
- ✅ **4D 시뮬레이션**: 일정 도구 및 TimeLiner
- ✅ **클라우드 통합**: Autodesk Construction Cloud 연동
- ✅ **Quantification**: 수량 산출 도구
- ✅ **모델 검토**: 고급 검토 기능
- ✅ **API 액세스**: 사용자 정의 플러그인 개발

### 대상 사용자

- BIM 조정자
- 프로젝트 관리자
- 설계 엔지니어
- 시공사 (조정 담당)

## 2. Navisworks Simulate

### 개요

3D 모델 검토와 시뮬레이션을 위한 버전입니다.

### 주요 기능

- ✅ **시뮬레이션 및 분석**: 4D/5D 기능
- ✅ **프로젝트 일정**: 5D 프로젝트 스케줄링
- ✅ **Quantification**: 수량 산출
- ✅ **모델 검토**: 기본 검토 기능
- ❌ **Clash Detective**: 충돌 검사 기능 없음

### 대상 사용자

- 프로젝트 관리자 (일정/비용 중심)
- 시공 계획자
- 건축주 (검토 목적)

### 중요 제한사항

**충돌 검사 기능은 Navisworks Manage에만 제공됩니다.**

## 3. Navisworks Freedom

### 개요

**무료 뷰어**로 집계된 3D 모델 데이터를 열고, 보고, 탐색할 수 있습니다.

### 주요 기능

- ✅ **무료 다운로드**: 비용 없음
- ✅ **파일 형식**: NWD 및 DWF 지원
- ✅ **기본 탐색**: 3D 모델 보기 및 탐색
- ✅ **간편한 배포**: 모델 준비, 서버 호스팅, 설정 시간 불필요
- ❌ **편집 불가**: 모델 편집이나 조정 기능 없음
- ❌ **고급 기능 없음**: 충돌 검사, 시뮬레이션 등 불가

### 대상 사용자

- 건축주 및 의사결정자
- 하청업체 (보기 목적만)
- 외부 이해관계자
- 일반 검토자

### 사용 시나리오

- 설계 검토 회의
- 클라이언트 프레젠테이션
- 현장 모델 확인
- 일반 시각화

## 제품 비교표

```
┌──────────────────┬──────────┬──────────┬──────────┐
│      기능        │  Manage  │ Simulate │ Freedom  │
├──────────────────┼──────────┼──────────┼──────────┤
│ 모델 보기        │    ✓     │    ✓     │    ✓     │
├──────────────────┼──────────┼──────────┼──────────┤
│ 충돌 검사        │    ✓     │    ✗     │    ✗     │
├──────────────────┼──────────┼──────────┼──────────┤
│ 4D 시뮬레이션    │    ✓     │    ✓     │    ✗     │
├──────────────────┼──────────┼──────────┼──────────┤
│ 수량 산출        │    ✓     │    ✓     │    ✗     │
├──────────────────┼──────────┼──────────┼──────────┤
│ 클라우드 통합    │    ✓     │    ✓     │    ✗     │
├──────────────────┼──────────┼──────────┼──────────┤
│ API 액세스       │    ✓     │    ✓     │    ✗     │
├──────────────────┼──────────┼──────────┼──────────┤
│ 마크업/주석      │    ✓     │    ✓     │    ✗     │
├──────────────────┼──────────┼──────────┼──────────┤
│ 가격            │   유료    │   유료    │   무료    │
└──────────────────┴──────────┴──────────┴──────────┘
```

## 라이선스 정보

### 구독 모델

- 연간 구독 방식
- 최대 **3대 컴퓨터**에 설치 가능
- 한 번에 **한 명의 사용자**만 사용 가능

### 가격대

- 6가지 에디션: **$135 ~ $8,010**
- (정확한 가격은 Autodesk 공식 사이트에서 확인)

---

# 📚 참고 자료 및 출처

이 가이드는 다음의 신뢰할 수 있는 출처를 기반으로 작성되었습니다:

## 공식 문서

- [Autodesk Navisworks 공식 웹사이트](https://www.autodesk.com/products/navisworks/overview)
- [Navisworks 2026 제품 업데이트](https://forums.autodesk.com/t5/navisworks-forum/product-update-navisworks-2026-what-s-new/td-p/13401440)
- [Navisworks Help | Native File Formats](https://help.autodesk.com/cloudhelp/2025/ENU/Navisworks/files/GUID-C6936A0B-1597-4880-83F9-7041894A4204.htm)
- [Navisworks API Documentation](https://aps.autodesk.com/developer/overview/navisworks)

## 기술 문서

- [File formats (NWF NWD NWC) in Navisworks](https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/NavisWorks-JetStream-file-formats-NWC-NWF-NWD-and-NWP.html)
- [Understanding clash detection setup](https://knowledge.autodesk.com/support/navisworks-products/learn-explore/caas/simplecontent/content/construction-coordination-understanding-clash-detection-types.html)
- [Set Clash Rules](https://knowledge.autodesk.com/support/navisworks-products/learn-explore/caas/CloudHelp/cloudhelp/2017/ENU/Navisworks-Manage/files/GUID-5A4502DF-75DF-4BA2-A649-35A921E3BBCD-htm.html)

## 전문 리소스

- [Novatr: Navisworks for BIM Clash Detection in 2026](https://www.novatr.com/blog/navisworks-for-clash-detection)
- [Novatr: Learn About Navisworks: Types, Features, Applications, & Benefits (2026)](https://www.novatr.com/blog/navisworks-type)
- [United-BIM: All About Clash Detection with Navisworks](https://www.united-bim.com/get-to-know-all-about-clash-detection-with-navisworks/)
- [GlobalCAD: Navisworks clash deep dive 1: configuration](https://globalcad.co.uk/navisworks-clash-deep-dive-1-configuration/)

## 실무 가이드

- [Interscale: A Complete Guide To Navisworks Clash Detection](https://interscaleedu.com/en/blog/bim/navisworks-clash-detection-guide/)
- [Modelical: Navisworks Clash Detection](https://www.modelical.com/en/gdocs/clash-detection/)
- [Autocad Everything: Navisworks Clash Detection - Ultimate Guide](https://autocadeverything.com/navisworks-clash-detection/)
- [ARKANCE UK: Navisworks File Formats (NWC, NWF, NWD) Explained](https://ukcommunity.arkance.world/hc/en-us/articles/21566098227858-Navisworks-File-Formats-NWC-NWF-NWD-Explained)

## 워크플로우 및 사례 연구

- [Autodesk University: Practical 4D Construction Simulation Using Revit and Navisworks](https://www.autodesk.com/autodesk-university/article/Practical-4D-Construction-Simulation-Using-Revit-and-Navisworks)
- [Autodesk University: 4D Planning in Construction: A Workflow in Navisworks with 3 Case Studies](https://www.autodesk.com/autodesk-university/class/4D-Planning-Construction-Workflow-Navisworks-3-Case-Studies-2019)
- [The Complete Guide to Federated Models in BIM](https://biminfo.neocities.org/the-complete-guide-to-federated-models-in-bim)

## 5D 및 수량 산출

- [ResearchGate: BIM-Based Quantity Takeoff in Autodesk Revit and Navisworks Manage](https://www.researchgate.net/publication/359854488_BIM-Based_Quantity_Takeoff_in_Autodesk_Revit_and_Navisworks_Manage)
- [Autocad Everything: Mastering Navisworks Quantification](https://autocadeverything.com/navisworks-quantification/)
- [Hi-Tech Digital: 5D BIM Services: Cost Estimation and Quantity Takeoffs](https://www.hitechdigital.com/cost-estimation-and-quantity-takeoff-solutions)

## MEP 조정

- [Advenser: MEP Coordination & Clash Detection Services](https://www.mepbim.com/mep-bim-coordination/)
- [BuiltInBIM: MEP BIM Coordination Services](https://builtinbim.com/services/mep-bim-coordination)
- [Autodesk: BIM for MEP - Key Benefits](https://www.autodesk.com/solutions/aec/bim/mep)

## API 및 자동화

- [Autodesk University: Make NavisWORK for You: Intro to the Navisworks API](https://www.autodesk.com/autodesk-university/class/Make-NavisWORK-You-Intro-Navisworks-API-2018)
- [LinkedIn: Naviswork & Python - How to automate a process](https://www.linkedin.com/pulse/naviswork-python-viewpoints-per-room-ambrosio-g%C3%B3mez-morales)
- [TwentyTwo: Navisworks Add-Ins and API Tutorials](https://twentytwo.space/navisworks-add-ins-development/)

## 제품 비교

- [Pinnacleinfotech: Navisworks vs. Revit: Key Features, Benefits & Applications](https://pinnacleinfotech.com/what-is-navisworks/)
- [Revizto: BIM 360 vs Navisworks vs Revizto](https://revizto.com/en/bim-360-vs-navisworks-vs-revizto/)
- [GlobalCAD: Navisworks on BIM projects: should you be using it?](https://globalcad.co.uk/navisworks-on-bim-projects-should-you-be-using-it/)

---

# 🎓 결론

Autodesk Navisworks는 현대 BIM 워크플로우에서 **필수적인 도구**로 자리잡았습니다. 다양한 소프트웨어 플랫폼에서 생성된 모델을 통합하고, 충돌을 조기에 발견하며, 프로젝트 일정과 비용을 시각화하는 능력은 건설 산업에 혁명적인 변화를 가져왔습니다.

## 핵심 가치

### 위험 감소

가상 환경에서 문제를 미리 발견하여 현장 재작업 비용과 시간 지연을 크게 줄입니다.

### 협업 강화

다학제 팀이 하나의 통합 모델에서 작업하여 의사소통과 조정이 원활해집니다.

### 의사결정 개선

4D/5D 시뮬레이션을 통해 프로젝트 수명 주기 전반에 걸쳐 정보에 기반한 의사결정을 내릴 수 있습니다.

## 선택 가이드

**Navisworks Manage를 선택하세요** - 충돌 검사가 필요한 경우

**Navisworks Simulate를 선택하세요** - 일정/비용 중심의 검토가 목적인 경우

**Navisworks Freedom을 사용하세요** - 단순 보기 및 프레젠테이션 목적인 경우

## 미래 전망

2026 버전의 새로운 기능들은 Navisworks가 계속해서 진화하고 있음을 보여줍니다. 클라우드 통합, AI 기반 충돌 감지, 더욱 직관적인 인터페이스 등이 향후 버전에서 기대됩니다.

건설 산업이 디지털 전환을 가속화하는 가운데, Navisworks는 BIM 조정과 프로젝트 관리의 중심에 서 있습니다.

---

**작성일**: 2026년 1월 13일  

**버전**: Navisworks 2026 기준  

**문서 유형**: 완전 가이드