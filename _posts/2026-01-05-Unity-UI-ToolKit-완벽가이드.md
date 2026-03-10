---
layout: post
title: 260115 Unity UI Toolkit 완벽 가이드
comments: true
tags:
- tag0
- tag1
- tag2
---

# 🎨 Unity UI Toolkit 완벽 가이드

> 웹 기술에서 영감을 받은 Unity의 차세대 UI 시스템 완벽 정복
> 

# 📖 목차

- 개요 및 역사
- UI Toolkit vs uGUI 비교
- UXML: UI 구조 정의
- USS: 스타일시트 시스템
- Flexbox 레이아웃
- Visual Element 계층구조
- UI Builder 사용법
- C# 스크립트 제어
- 데이터 바인딩
- 커스텀 컨트롤
- 이벤트 시스템
- 런타임 vs 에디터 UI
- 성능 최적화
- 반응형 UI 디자인
- 애니메이션 및 트랜지션
- 실전 예제
- 에디터 확장
- 트러블슈팅

---

# 🌟 UI Toolkit 개요 및 역사

## 📜 진화의 역사

Unity UI Toolkit은 웹 기술(HTML, CSS, JavaScript)에서 영감을 받아 개발된 **차세대 UI 프레임워크**입니다.

### 타임라인

```
2019 → UIElements (초기 버전, 에디터 전용)
2020 → UI Toolkit으로 브랜딩 변경
2021 → 런타임 UI 지원 시작 (실험적)
2023 → 프로덕션 레디 상태 도달
2025 → Unity 6에서 안정화, 대규모 프로젝트 권장
```

## 🎯 핵심 철학

UI Toolkit은 **3가지 핵심 기술**로 구성됩니다:

1. **UXML** (Unity XML) - HTML과 유사한 UI 구조 정의
2. **USS** (Unity Style Sheets) - CSS 기반 스타일링
3. **C# Scripts** - JavaScript처럼 동작하는 로직 제어

```
┌─────────────────────────────────┐
│     Web Development Pattern     │
├─────────────┬──────────┬────────┤
│    HTML     │   CSS    │   JS   │
└─────────────┴──────────┴────────┘
        ↓           ↓         ↓
┌─────────────┬──────────┬────────┐
│    UXML     │   USS    │   C#   │
│  Structure  │  Style   │ Logic  │
└─────────────┴──────────┴────────┘
```

## 💡 왜 UI Toolkit인가?

### 장점 ✅

- **성능**: GameObject 오버헤드 없음, 배칭된 렌더링
- **생산성**: UI Builder로 시각적 편집 가능
- **유지보수**: 구조와 스타일 분리로 관리 용이
- **웹 개발자 친화적**: CSS Flexbox 사용
- **크로스 플랫폼**: 에디터와 런타임 모두 지원

### 단점 ⚠️

- **3D 월드 공간 미지원**: Canvas World Space 불가
- **학습 곡선**: 웹 기술에 익숙하지 않으면 초기 진입장벽
- **애니메이션 제한**: Animator/Timeline 미지원
- **레거시 프로젝트**: 기존 uGUI 마이그레이션 비용

---

# ⚖️ UI Toolkit vs uGUI 비교

## 🔄 아키텍처 차이

### uGUI (Unity UI)

```csharp
// GameObject 기반 계층 구조
Canvas (GameObject)
  └─ Panel (GameObject + RectTransform)
       └─ Button (GameObject + Button Component)
            └─ Text (GameObject + Text Component)

// 모든 UI 요소 = GameObject
// RectTransform으로 위치/크기 제어
// MonoBehaviour 컴포넌트로 로직 추가
```

### UI Toolkit

```csharp
// Visual Element 기반 가상 트리
UIDocument (MonoBehaviour)
  └─ VisualElement (Virtual)
       └─ Button (Virtual)
            └─ Label (Virtual)

// GameObject 없음 → 메모리/CPU 절약
// USS로 스타일 제어
// C# 스크립트로 이벤트 처리
```

## 📊 비교표

| 항목 | uGUI | UI Toolkit |
| --- | --- | --- |
| **기반 구조** | GameObject + Component | Visual Element (가상) |
| **레이아웃** | RectTransform, Anchors | Flexbox (CSS) |
| **스타일링** | Inspector 직접 설정 | USS 파일 (재사용 가능) |
| **UI 편집** | Scene/Game View | UI Builder |
| **성능** | GameObject 오버헤드 | 배칭된 렌더링, 낮은 오버헤드 |
| **드로우 콜** | Canvas별 Batch | 전체 UIDocument 단일 배치 |
| **애니메이션** | Animator, DOTween | USS Transition (제한적) |
| **3D 월드 공간** | ✅ World Space Canvas | ❌ 미지원 |
| **데이터 바인딩** | 수동 구현 필요 | SerializedObject 자동 바인딩 |
| **에디터 확장** | IMGUI | UI Toolkit (권장) |
| **학습 곡선** | Unity 전통 방식 | 웹 개발 패턴 |

## 🎯 사용 케이스별 권장

### ✅ UI Toolkit 권장

- 📊 **데이터 중심 인터페이스**: 인벤토리, 스킬 트리, 퀘스트 로그, 리더보드
- 📱 **모바일 게임**: 많은 UI 요소, 배터리/성능 중요
- 🛠️ **에디터 확장**: Custom Inspector, EditorWindow
- 🆕 **신규 프로젝트**: Unity 6 이상 사용 시
- 📈 **대규모 UI**: 수백 개 이상의 UI 요소

### ✅ uGUI 권장

- 🌍 **3D 월드 공간 UI**: 캐릭터 위 HP 바, 월드 공간 메뉴
- 🎬 **복잡한 애니메이션**: Animator, Timeline 통합 필요
- 🔄 **레거시 프로젝트**: 기존 uGUI 자산 재사용
- 🎮 **게임 같은 UI**: 다이나믹한 애니메이션, 파티클 효과

### 🔀 하이브리드 접근

동일 프로젝트에서 **uGUI + UI Toolkit 혼용** 가능:

```csharp
// 예시 구조
Game Scene
├─ UI Toolkit (UIDocument)
│   ├─ HUD (체력, 스태미나 바)
│   ├─ 인벤토리 시스템
│   └─ 메뉴 시스템
│
└─ uGUI (Canvas - World Space)
    ├─ NPC 위 이름표
    ├─ 데미지 숫자 (파티클)
    └─ 월드 공간 인터랙션 UI
```

## 🚀 마이그레이션 전략

### Unity 공식 가이드라인

1. **프로파일링 먼저**: 성능 문제 확인 후 마이그레이션
2. **점진적 전환**: 문제 화면부터 하나씩
3. **하이브리드 유지**: 잘 작동하는 uGUI는 그대로 유지
4. **신규 기능**: UI Toolkit으로 개발

### 컴포넌트 매핑표

| uGUI | UI Toolkit |
| --- | --- |
| Canvas | UIDocument |
| RectTransform | VisualElement + USS |
| Button | Button |
| Text / TextMeshPro | Label |
| Image | VisualElement (background-image) |
| ScrollRect | ScrollView |
| Slider | Slider |
| Toggle | Toggle |
| InputField | TextField |
| Dropdown | DropdownField |

---

# 🏗️ UXML: UI 구조 정의 완벽 가이드

## 📝 UXML이란?

**UXML (Unity XML)**은 UI의 구조와 콘텐츠를 정의하는 마크업 언어입니다. HTML과 매우 유사하며, XML 형식을 따릅니다.

## 🔍 기본 문법

### 최소 구조

```xml
<ui:UXML xmlns:ui="UnityEngine.UIElements" xmlns:uie="UnityEditor.UIElements">
    <ui:VisualElement name="root">
        <ui:Label text="Hello, UI Toolkit!" />
    </ui:VisualElement>
</ui:UXML>
```

### 네임스페이스 설명

```xml
<!-- 런타임 + 에디터 공통 요소 -->
xmlns:ui="UnityEngine.UIElements"

<!-- 에디터 전용 요소 (Inspector 등) -->
xmlns:uie="UnityEditor.UIElements"
```

## 🎨 주요 UI 요소

### 기본 컨트롤

```xml
<ui:UXML xmlns:ui="UnityEngine.UIElements">
    <!-- 텍스트 표시 -->
    <ui:Label text="게임 타이틀" />
    
    <!-- 버튼 -->
    <ui:Button text="시작하기" name="start-button" />
    
    <!-- 텍스트 입력 -->
    <ui:TextField label="플레이어 이름" name="player-name" />
    
    <!-- 슬라이더 -->
    <ui:Slider label="볼륨" low-value="0" high-value="100" value="50" />
    
    <!-- 토글 -->
    <ui:Toggle label="음소거" name="mute-toggle" />
    
    <!-- 드롭다운 -->
    <ui:DropdownField label="난이도" />
    
    <!-- 이미지 -->
    <ui:VisualElement name="logo" class="logo-image" />
</ui:UXML>
```

### 레이아웃 컨테이너

```xml
<!-- 수직 레이아웃 -->
<ui:VisualElement class="vertical-container">
    <ui:Label text="항목 1" />
    <ui:Label text="항목 2" />
    <ui:Label text="항목 3" />
</ui:VisualElement>

<!-- 수평 레이아웃 -->
<ui:VisualElement class="horizontal-container">
    <ui:Button text="예" />
    <ui:Button text="아니오" />
</ui:VisualElement>

<!-- 스크롤 뷰 -->
<ui:ScrollView>
    <ui:VisualElement style="height: 2000px;">
        <!-- 긴 콘텐츠 -->
    </ui:VisualElement>
</ui:ScrollView>
```

## 🏷️ 속성 시스템

### name vs class

```xml
<!-- name: 고유 식별자 (C#에서 접근) -->
<ui:Button name="submit-button" text="제출" />

<!-- class: 스타일 그룹 (USS로 스타일링) -->
<ui:Button class="primary-button" text="확인" />
<ui:Button class="primary-button" text="적용" />

<!-- 여러 클래스 적용 -->
<ui:Button class="primary-button large-button" text="시작" />
```

### 인라인 스타일

```xml
<!-- 비권장: 재사용 불가 -->
<ui:Label text="경고" style="color: red; font-size: 20px;" />

<!-- 권장: USS 사용 -->
<ui:Label text="경고" class="warning-text" />
```

## 🔗 USS 연결

```xml
<ui:UXML xmlns:ui="UnityEngine.UIElements">
    <!-- USS 파일 참조 -->
    <Style src="/Assets/UI/Styles/MainMenu.uss" />
    
    <ui:VisualElement class="menu-container">
        <ui:Button class="menu-button" text="게임 시작" />
    </ui:VisualElement>
</ui:UXML>
```

## 🧩 템플릿 및 재사용

### 템플릿 정의

```xml
<!-- ItemSlot.uxml -->
<ui:UXML xmlns:ui="UnityEngine.UIElements">
    <ui:Template name="ItemSlot" src="ItemSlot.uxml" />
    
    <ui:VisualElement class="item-slot">
        <ui:VisualElement name="item-icon" class="icon" />
        <ui:Label name="item-name" text="아이템" />
        <ui:Label name="item-count" text="x1" />
    </ui:VisualElement>
</ui:UXML>
```

### 템플릿 인스턴스화

```xml
<!-- Inventory.uxml -->
<ui:UXML xmlns:ui="UnityEngine.UIElements">
    <ui:Template name="ItemSlot" src="ItemSlot.uxml" />
    
    <ui:VisualElement class="inventory-grid">
        <!-- 템플릿 10개 인스턴스 -->
        <ui:Instance template="ItemSlot" />
        <ui:Instance template="ItemSlot" />
        <ui:Instance template="ItemSlot" />
        <!-- ... -->
    </ui:VisualElement>
</ui:UXML>
```

## 🎯 실전 예제: 로그인 화면

```xml
<ui:UXML xmlns:ui="UnityEngine.UIElements">
    <Style src="/Assets/UI/LoginScreen.uss" />
    
    <ui:VisualElement class="login-container">
        <!-- 로고 -->
        <ui:VisualElement name="logo" class="game-logo" />
        
        <!-- 타이틀 -->
        <ui:Label text="어드벤처 게임" class="title" />
        
        <!-- 입력 필드 -->
        <ui:VisualElement class="input-group">
            <ui:TextField 
                label="아이디" 
                name="username-field" 
                placeholder-text="이메일을 입력하세요"
                class="input-field" />
            
            <ui:TextField 
                label="비밀번호" 
                name="password-field" 
                password="true"
                placeholder-text="비밀번호를 입력하세요"
                class="input-field" />
        </ui:VisualElement>
        
        <!-- 체크박스 -->
        <ui:Toggle label="로그인 상태 유지" name="remember-toggle" />
        
        <!-- 버튼 그룹 -->
        <ui:VisualElement class="button-group">
            <ui:Button text="로그인" name="login-button" class="primary-button" />
            <ui:Button text="회원가입" name="signup-button" class="secondary-button" />
        </ui:VisualElement>
        
        <!-- 에러 메시지 -->
        <ui:Label name="error-message" class="error-text hidden" />
    </ui:VisualElement>
</ui:UXML>
```

## 🔧 UXML 베스트 프랙티스

### ✅ Do

```xml
<!-- 의미 있는 name 사용 -->
<ui:Button name="submit-button" />

<!-- class로 스타일 그룹화 -->
<ui:Label class="header-text primary-color" />

<!-- 템플릿으로 재사용 -->
<ui:Template name="Card" src="Card.uxml" />
```

### ❌ Don't

```xml
<!-- 인라인 스타일 남발 -->
<ui:Label style="color: red; font-size: 20px; margin: 10px;" />

<!-- 의미 없는 name -->
<ui:Button name="btn1" />

<!-- 중복 코드 -->
<ui:VisualElement class="item">
    <ui:Label text="아이템 1" />
</ui:VisualElement>
<ui:VisualElement class="item">
    <ui:Label text="아이템 2" />
</ui:VisualElement>
<!-- 템플릿 사용 권장 -->
```

---

# 🎨 USS: 스타일시트 시스템 완벽 가이드

## 📝 USS란?

**USS (Unity Style Sheets)**는 CSS에서 영감을 받은 Unity의 스타일시트 언어입니다. UI 요소의 시각적 표현(색상, 크기, 레이아웃 등)을 정의합니다.

## 🎯 기본 문법

### 선택자 (Selectors)

```css
/* 타입 선택자: 모든 Label에 적용 */
Label {
    color: white;
    font-size: 16px;
}

/* 클래스 선택자: .primary-button 클래스 */
.primary-button {
    background-color: rgb(0, 120, 215);
    border-radius: 5px;
}

/* name 선택자: 특정 name */
#submit-button {
    width: 200px;
    height: 50px;
}

/* 자식 선택자 */
.menu-container > Button {
    margin: 5px;
}

/* 후손 선택자 */
.inventory Label {
    text-align: center;
}

/* 의사 클래스 */
Button:hover {
    background-color: rgb(0, 150, 255);
}

Button:active {
    background-color: rgb(0, 100, 200);
}

/* 여러 선택자 */
.primary-button, .secondary-button {
    border-width: 2px;
}
```

## 🎨 색상 및 배경

```css
/* 색상 포맷 */
.element {
    /* RGB */
    color: rgb(255, 0, 0);
    
    /* RGBA */
    background-color: rgba(0, 0, 0, 0.5);
    
    /* 16진수 */
    border-color: #FF6600;
    
    /* 이름 */
    color: red;
}

/* 배경 이미지 */
.logo {
    background-image: url('/Assets/Textures/logo.png');
    width: 200px;
    height: 100px;
}

/* 배경 이미지 옵션 */
.panel-background {
    background-image: url('/Assets/UI/panel-bg.png');
    -unity-background-scale-mode: stretch-to-fill; /* scale-and-crop, scale-to-fit */
    -unity-background-image-tint-color: rgba(255, 255, 255, 0.8);
}
```

## 📐 크기 및 위치

```css
/* 크기 단위 */
.element {
    /* 픽셀 */
    width: 200px;
    height: 100px;
    
    /* 퍼센트 (부모 기준) */
    width: 50%;
    height: 100%;
    
    /* auto (콘텐츠에 맞춤) */
    width: auto;
}

/* 최소/최대 크기 */
.resizable {
    min-width: 100px;
    max-width: 500px;
    min-height: 50px;
    max-height: 300px;
}

/* 여백 (Margin) */
.spaced {
    margin: 10px; /* 모든 방향 */
    margin-top: 5px;
    margin-bottom: 5px;
    margin-left: 10px;
    margin-right: 10px;
}

/* 패딩 (Padding) */
.padded {
    padding: 15px;
    padding-top: 10px;
    padding-bottom: 10px;
    padding-left: 20px;
    padding-right: 20px;
}
```

## 🖼️ 테두리 및 모서리

```css
/* 테두리 */
.bordered {
    border-width: 2px;
    border-color: white;
    border-radius: 10px;
}

/* 개별 테두리 */
.custom-border {
    border-top-width: 1px;
    border-bottom-width: 2px;
    border-left-width: 1px;
    border-right-width: 1px;
    
    border-top-color: red;
    border-bottom-color: blue;
}

/* 모서리 반경 */
.rounded {
    border-radius: 20px; /* 모든 모서리 */
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    border-bottom-left-radius: 5px;
    border-bottom-right-radius: 5px;
}
```

## ✍️ 텍스트 스타일링

```css
/* 폰트 */
.text {
    font-size: 18px;
    -unity-font-style: bold; /* normal, italic, bold-and-italic */
    -unity-text-align: middle-center; /* upper-left, middle-center, etc */
    color: white;
}

/* 커스텀 폰트 */
.custom-font {
    -unity-font: url('/Assets/Fonts/MyFont.ttf');
    -unity-font-definition: url('/Assets/Fonts/MyFont SDF.asset');
}

/* 텍스트 그림자 (미지원, workaround 필요) */
.text-with-shadow {
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5); /* CSS에는 있지만 USS 미지원 */
}

/* 텍스트 줄바꿈 */
.wrapped-text {
    white-space: normal; /* nowrap, pre, pre-wrap */
    -unity-text-overflow-position: end; /* start, middle, end */
}
```

## 🔲 위치 및 배치

```css
/* Position */
.absolute-positioned {
    position: absolute;
    top: 10px;
    left: 10px;
    right: 10px;
    bottom: 10px;
}

.relative-positioned {
    position: relative; /* 기본값 */
}

/* Display */
.hidden {
    display: none; /* 렌더링 안 함, 레이아웃에서 제외 */
}

.visible {
    display: flex; /* 기본값 */
}

/* Visibility */
.invisible {
    visibility: hidden; /* 렌더링 안 하지만 레이아웃 공간 차지 */
}

/* Overflow */
.scrollable {
    overflow: scroll; /* visible, hidden, scroll */
}
```

## 🎯 실전 예제: 버튼 스타일

```css
/* 기본 버튼 */
.menu-button {
    width: 250px;
    height: 60px;
    background-color: rgba(30, 30, 30, 0.9);
    border-width: 2px;
    border-color: rgba(255, 255, 255, 0.3);
    border-radius: 10px;
    color: white;
    font-size: 20px;
    -unity-font-style: bold;
    margin: 10px;
    
    /* 트랜지션 */
    transition: background-color 0.2s, border-color 0.2s, scale 0.1s;
}

/* 호버 상태 */
.menu-button:hover {
    background-color: rgba(50, 50, 50, 0.95);
    border-color: rgba(255, 255, 255, 0.6);
    scale: 1.05;
}

/* 클릭 상태 */
.menu-button:active {
    background-color: rgba(20, 20, 20, 1);
    scale: 0.98;
}

/* Primary 변형 */
.menu-button.primary {
    background-color: rgb(0, 120, 215);
    border-color: rgb(0, 150, 255);
}

.menu-button.primary:hover {
    background-color: rgb(0, 150, 255);
}

/* Disabled 상태 */
.menu-button:disabled {
    background-color: rgba(50, 50, 50, 0.5);
    border-color: rgba(100, 100, 100, 0.5);
    color: rgba(150, 150, 150, 0.8);
}
```

## 🎨 실전 예제: 카드 UI

```css
/* 카드 컨테이너 */
.card {
    width: 300px;
    background-color: rgba(20, 20, 20, 0.95);
    border-width: 1px;
    border-color: rgba(255, 255, 255, 0.2);
    border-radius: 15px;
    padding: 20px;
    margin: 15px;
    
    /* 그림자 효과 (workaround) */
    -unity-background-image-tint-color: rgba(0, 0, 0, 0.3);
}

/* 카드 헤더 */
.card-header {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
    border-bottom-width: 1px;
    border-bottom-color: rgba(255, 255, 255, 0.1);
    padding-bottom: 10px;
}

.card-title {
    font-size: 22px;
    -unity-font-style: bold;
    color: white;
}

.card-icon {
    width: 40px;
    height: 40px;
    background-image: url('/Assets/UI/Icons/card-icon.png');
}

/* 카드 본문 */
.card-body {
    margin-top: 10px;
    margin-bottom: 10px;
}

.card-description {
    font-size: 14px;
    color: rgba(255, 255, 255, 0.8);
    white-space: normal;
}

/* 카드 푸터 */
.card-footer {
    flex-direction: row;
    justify-content: flex-end;
    margin-top: 15px;
}

.card-button {
    width: 80px;
    height: 35px;
    background-color: rgb(0, 120, 215);
    border-radius: 5px;
    color: white;
    font-size: 14px;
    margin-left: 10px;
}

.card-button:hover {
    background-color: rgb(0, 150, 255);
}
```

## 🧩 USS 변수 (Unity 2023+)

```css
/* 변수 정의 */
:root {
    --primary-color: rgb(0, 120, 215);
    --secondary-color: rgb(108, 117, 125);
    --text-color: white;
    --background-color: rgba(20, 20, 20, 0.95);
    --border-radius: 10px;
    --spacing: 15px;
}

/* 변수 사용 */
.themed-button {
    background-color: var(--primary-color);
    border-radius: var(--border-radius);
    color: var(--text-color);
    padding: var(--spacing);
}

.themed-panel {
    background-color: var(--background-color);
    border-radius: var(--border-radius);
}
```

## 🔧 USS 베스트 프랙티스

### ✅ Do

```css
/* 재사용 가능한 클래스 */
.flex-row {
    flex-direction: row;
}

.flex-column {
    flex-direction: column;
}

.center {
    justify-content: center;
    align-items: center;
}

/* 의미 있는 클래스명 */
.primary-button { }
.warning-text { }
.success-message { }
```

### ❌ Don't

```css
/* 너무 구체적인 선택자 (유지보수 어려움) */
#root > .container > .panel > .button { }

/* 인라인 스타일 대신 USS 사용 */
/* UXML: style="color: red" (X) */
/* USS: .error-text { color: red; } (O) */
```

---

# 🧱 Flexbox 레이아웃 시스템

## 📝 Flexbox란?

UI Toolkit의 레이아웃 엔진은 **Yoga**를 기반으로 하며, CSS Flexbox의 서브셋을 구현합니다. 웹 개발에 익숙한 개발자에게 친숙한 방식입니다.

## 🎯 Flex Container 기본

### Flex Direction

```css
/* 세로 배치 (기본값) */
.vertical-container {
    flex-direction: column;
}

/* 가로 배치 */
.horizontal-container {
    flex-direction: row;
}

/* 역순 세로 */
.vertical-reverse {
    flex-direction: column-reverse;
}

/* 역순 가로 */
.horizontal-reverse {
    flex-direction: row-reverse;
}
```

```xml
<!-- UXML 예시 -->
<ui:VisualElement class="horizontal-container">
    <ui:Label text="항목 1" />
    <ui:Label text="항목 2" />
    <ui:Label text="항목 3" />
</ui:VisualElement>
<!-- 결과: [항목 1] [항목 2] [항목 3] -->
```

### Justify Content (주축 정렬)

```css
/* 시작점 정렬 (기본값) */
.justify-start {
    justify-content: flex-start;
}

/* 끝점 정렬 */
.justify-end {
    justify-content: flex-end;
}

/* 중앙 정렬 */
.justify-center {
    justify-content: center;
}

/* 양끝 정렬 */
.justify-space-between {
    justify-content: space-between;
}

/* 균등 분배 */
.justify-space-around {
    justify-content: space-around;
}
```

### Align Items (교차축 정렬)

```css
/* 시작점 정렬 */
.align-start {
    align-items: flex-start;
}

/* 끝점 정렬 */
.align-end {
    align-items: flex-end;
}

/* 중앙 정렬 */
.align-center {
    align-items: center;
}

/* 늘림 (기본값) */
.align-stretch {
    align-items: stretch;
}
```

## 🎨 Flex Item 속성

### Flex Grow (공간 채우기)

```css
/* Flex Grow */
.flex-1 {
    flex-grow: 1; /* 남은 공간의 1배 차지 */
}

.flex-2 {
    flex-grow: 2; /* 남은 공간의 2배 차지 */
}

.flex-0 {
    flex-grow: 0; /* 고정 크기 (기본값) */
}
```

```xml
<!-- 예시: 2:1 비율 레이아웃 -->
<ui:VisualElement class="horizontal-container">
    <ui:VisualElement class="flex-2" style="background-color: red;" />
    <ui:VisualElement class="flex-1" style="background-color: blue;" />
</ui:VisualElement>
<!-- 빨강이 파랑보다 2배 넓음 -->
```

### Flex Shrink (공간 축소)

```css
/* Flex Shrink */
.no-shrink {
    flex-shrink: 0; /* 축소 안 함 */
}

.shrinkable {
    flex-shrink: 1; /* 축소 가능 (기본값) */
}
```

### Flex Basis (기본 크기)

```css
.fixed-width {
    flex-basis: 200px; /* 초기 크기 200px */
}

.auto-width {
    flex-basis: auto; /* 콘텐츠 크기 (기본값) */
}
```

## 🎯 실전 예제: 헤더 레이아웃

```xml
<ui:VisualElement class="header" />
```

```css
.header {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    height: 60px;
    background-color: rgba(20, 20, 20, 0.95);
    padding-left: 20px;
    padding-right: 20px;
}

.header-logo {
    width: 150px;
    height: 40px;
    flex-shrink: 0; /* 로고는 축소 안 함 */
}

.header-menu {
    flex-direction: row;
    flex-grow: 1; /* 남은 공간 차지 */
    justify-content: center;
}

.header-menu Button {
    margin-left: 10px;
    margin-right: 10px;
}

.header-profile {
    flex-direction: row;
    align-items: center;
    flex-shrink: 0;
}
```

## 🎨 실전 예제: 그리드 레이아웃

```xml
<ui:VisualElement class="grid-container">
    <ui:VisualElement class="grid-row">
        <ui:VisualElement class="grid-item" />
        <ui:VisualElement class="grid-item" />
        <ui:VisualElement class="grid-item" />
    </ui:VisualElement>
    <ui:VisualElement class="grid-row">
        <ui:VisualElement class="grid-item" />
        <ui:VisualElement class="grid-item" />
        <ui:VisualElement class="grid-item" />
    </ui:VisualElement>
</ui:VisualElement>
```

```css
.grid-container {
    flex-direction: column;
    width: 100%;
}

.grid-row {
    flex-direction: row;
    justify-content: space-between;
    margin-bottom: 10px;
}

.grid-item {
    flex-grow: 1;
    flex-basis: 0; /* 균등 분배 */
    height: 100px;
    margin-left: 5px;
    margin-right: 5px;
    background-color: rgba(50, 50, 50, 0.9);
    border-radius: 10px;
}
```

## 🧩 반응형 레이아웃 패턴

### 중앙 정렬 패널

```css
.fullscreen-center {
    width: 100%;
    height: 100%;
    justify-content: center;
    align-items: center;
}

.center-panel {
    width: 400px;
    height: 300px;
    background-color: rgba(20, 20, 20, 0.95);
    border-radius: 15px;
}
```

### 스크롤 가능한 콘텐츠

```css
.scrollable-container {
    flex-grow: 1;
    overflow: scroll;
}

.content {
    flex-direction: column;
    padding: 20px;
}
```

### 고정 헤더 + 스크롤 본문

```css
.main-layout {
    flex-direction: column;
    width: 100%;
    height: 100%;
}

.fixed-header {
    flex-shrink: 0; /* 축소 안 함 */
    height: 60px;
}

.scrollable-body {
    flex-grow: 1; /* 남은 공간 차지 */
    overflow: scroll;
}

.fixed-footer {
    flex-shrink: 0;
    height: 40px;
}
```

---

# 🌳 Visual Element 계층구조

## 📝 Visual Element란?

**VisualElement**는 UI Toolkit의 가장 기본적인 빌딩 블록입니다. GameObject가 아닌 **가상 객체**이며, **Visual Tree**라는 계층 구조를 형성합니다.

## 🏗️ Visual Tree 구조

```
UIDocument (MonoBehaviour)
  └─ rootVisualElement (VisualElement)
       ├─ Panel (VisualElement)
       │   ├─ Header (VisualElement)
       │   │   ├─ Title (Label)
       │   │   └─ CloseButton (Button)
       │   ├─ Body (ScrollView)
       │   │   ├─ ContentItem1 (VisualElement)
       │   │   └─ ContentItem2 (VisualElement)
       │   └─ Footer (VisualElement)
       └─ Overlay (VisualElement)
```

## 🎯 VisualElement 생성 방법

### 1. UI Builder (시각적 편집)

```
1. Window > UI Toolkit > UI Builder
2. Hierarchy > 우클릭 > Add Element > VisualElement
3. Inspector에서 속성 설정
4. USS로 스타일링
```

### 2. UXML (마크업)

```xml
<ui:VisualElement name="my-container" class="container">
    <ui:Label text="자식 요소" />
</ui:VisualElement>
```

### 3. C# (프로그래매틱)

```csharp
using UnityEngine.UIElements;

public class MyUI : MonoBehaviour
{
    private void Start()
    {
        var root: VisualElement = GetComponent<UIDocument>().rootVisualElement;
        
        // VisualElement 생성
        var container: VisualElement = new VisualElement();
        [container.name](http://container.name) = "my-container";
        container.AddToClassList("container");
        
        // 자식 추가
        var label: Label = new Label("자식 요소");
        container.Add(label);
        
        // 루트에 추가
        root.Add(container);
    }
}
```

## 🧩 주요 VisualElement 타입

### 기본 요소

```csharp
// 텍스트
Label label = new Label("텍스트");

// 버튼
Button button = new Button(() => Debug.Log("클릭!"));
button.text = "클릭하세요";

// 텍스트 입력
TextField textField = new TextField("라벨");
textField.value = "초기값";

// 이미지 (VisualElement + background-image)
VisualElement image = new VisualElement();
[image.style](http://image.style).backgroundImage = new StyleBackground(myTexture);

// 토글
Toggle toggle = new Toggle("체크박스");
toggle.value = true;

// 슬라이더
Slider slider = new Slider("볼륨", 0, 100);
slider.value = 50;

// 드롭다운
DropdownField dropdown = new DropdownField("옵션", new List<string> { "A", "B", "C" }, 0);
```

### 레이아웃 요소

```csharp
// 스크롤 뷰
ScrollView scrollView = new ScrollView();
scrollView.Add(new Label("스크롤 콘텐츠"));

// Foldout (접을 수 있는 영역)
Foldout foldout = new Foldout();
foldout.text = "펼치기/접기";
foldout.Add(new Label("숨겨진 콘텐츠"));

// GroupBox (그룹화)
GroupBox groupBox = new GroupBox("그룹");
groupBox.Add(new Label("그룹 내용"));
```

## 🔍 요소 탐색 및 접근

### C#에서 요소 찾기

```csharp
using UnityEngine.UIElements;

public class UIController : MonoBehaviour
{
    [SerializeField] private UIDocument _uiDocument;
    
    private void Start()
    {
        var root: VisualElement = _uiDocument.rootVisualElement;
        
        // name으로 찾기 (권장)
        Button submitButton = root.Q<Button>("submit-button");
        submitButton.clicked += OnSubmitClicked;
        
        // class로 찾기
        var primaryButtons: UQueryBuilder<Button> = root.Query<Button>(className: "primary-button");
        primaryButtons.ForEach(btn => [btn.style](http://btn.style).backgroundColor = [Color.blue](http://Color.blue));
        
        // 타입으로 모든 요소 찾기
        List<Label> allLabels = root.Query<Label>().ToList();
        
        // 자식 직접 접근
        VisualElement firstChild = root[0]; // 첫 번째 자식
        int childCount = root.childCount;
    }
    
    private void OnSubmitClicked()
    {
        Debug.Log($"제출 버튼 클릭!");
    }
}
```

### 계층 구조 탐색

```csharp
// 부모 접근
VisualElement parent = myElement.parent;

// 자식 순회
foreach (VisualElement child in myElement.Children())
{
    Debug.Log($"자식: {[child.name](http://child.name)}");
}

// 형제 접근
int index = myElement.parent.IndexOf(myElement);
VisualElement nextSibling = myElement.parent[index + 1];

// 조상 찾기
VisualElement ancestor = myElement.GetFirstAncestorOfType<ScrollView>();
```

## 🎨 동적 UI 생성 예제

### 리스트 아이템 생성

```csharp
public class InventoryUI : MonoBehaviour
{
    [SerializeField] private UIDocument _uiDocument;
    private VisualElement _itemContainer;
    
    private void Start()
    {
        var root: VisualElement = _uiDocument.rootVisualElement;
        _itemContainer = root.Q<VisualElement>("item-container");
        
        // 아이템 10개 생성
        for (int i = 0; i < 10; i++)
        {
            CreateItemSlot(i);
        }
    }
    
    private void CreateItemSlot(int index: int)
    {
        // 슬롯 컨테이너
        var slot: VisualElement = new VisualElement();
        [slot.name](http://slot.name) = $"item-slot-{index}";
        slot.AddToClassList("item-slot");
        
        // 아이콘
        var icon: VisualElement = new VisualElement();
        [icon.name](http://icon.name) = "icon";
        icon.AddToClassList("item-icon");
        slot.Add(icon);
        
        // 이름
        var itemName: Label = new Label($"아이템 {index + 1}");
        itemName.AddToClassList("item-name");
        slot.Add(itemName);
        
        // 수량
        var count: Label = new Label("x1");
        count.AddToClassList("item-count");
        slot.Add(count);
        
        // 클릭 이벤트
        slot.RegisterCallback<ClickEvent>(evt => OnItemClicked(index));
        
        _itemContainer.Add(slot);
    }
    
    private void OnItemClicked(int index: int)
    {
        Debug.Log($"아이템 {index} 클릭!");
    }
}
```

## 🔧 요소 조작

### 추가/제거

```csharp
// 추가
parent.Add(child);
parent.Insert(0, child); // 특정 인덱스에 추가

// 제거
parent.Remove(child);
parent.RemoveAt(0); // 인덱스로 제거
parent.Clear(); // 모든 자식 제거

// 교체
parent.Remove(oldChild);
parent.Add(newChild);
```

### 표시/숨김

```csharp
// Display (렌더링 + 레이아웃 제외)
[element.style](http://element.style).display = DisplayStyle.None; // 숨김
[element.style](http://element.style).display = DisplayStyle.Flex; // 표시

// Visibility (렌더링만 제외, 레이아웃 유지)
[element.style](http://element.style).visibility = Visibility.Hidden;
[element.style](http://element.style).visibility = Visibility.Visible;

// USS 클래스 토글
element.AddToClassList("hidden"); // .hidden { display: none; }
element.RemoveFromClassList("hidden");
element.ToggleInClassList("hidden");
```

---

# 🛠️ UI Builder 사용법

## 📝 UI Builder란?

UI Builder는 Unity 에디터에 통합된 **시각적 UI 편집 도구**로, UXML과 USS 파일을 직접 코딩하지 않고도 UI를 디자인할 수 있습니다.

## 🚀 시작하기

### UI Builder 열기

```
Window > UI Toolkit > UI Builder
```

### 새 UXML 문서 생성

```
1. UI Builder > File > New
2. Save As... > Assets/UI/MyUI.uxml
```

## 🖼️ 인터페이스 구성

```jsx
┌─────────────────────────────────────────────┐
│  Toolbar (New, Save, Preview)               │
├──────────┬──────────────────┬───────────────┤
│          │                  │               │
│ Library  │   Viewport       │  Inspector    │

│          │                  │               │
│ - Containers │                  │ - Name        │
│ - Controls   │                  │ - StyleSheets │
│ - Templates  │                  │ - Size        │
│          │                  │ - Position    │
├──────────┴──────────────────┴───────────────┤
│  Hierarchy                                   │
│  - Document Structure                       │
│  - Element Tree                             │
└─────────────────────────────────────────────┘
```

### 패널 설명

- **Library**: 사용 가능한 UI 요소 목록 (드래그&드롭)
- **Viewport**: 실시간 미리보기 캔버스
- **Hierarchy**: UI 요소 트리 구조
- **Inspector**: 선택한 요소의 속성/스타일 편집

## 🎯 UI 요소 추가하기

### 방법 1: 드래그 & 드롭

```
1. Library 패널에서 요소 선택 (ex: Button)
2. Viewport 또는 Hierarchy로 드래그
3. 위치 조정
```

### 방법 2: 우클릭 메뉴

```
1. Hierarchy에서 부모 요소 우클릭
2. Add Element > 요소 선택
```

## 🎨 스타일 편집

### 인라인 스타일 (Inspector)

```
1. Hierarchy에서 요소 선택
2. Inspector > Style Class List > 클래스 추가
3. Inspector > Style > 속성 조정
   - Size (width, height)
   - Position (margin, padding)
   - Background (color, image)
   - Border
   - Text (font, size, color)
   - Flex (direction, grow, shrink)
```

### USS 파일 생성 및 연결

```
1. Project 패널 > 우클릭 > Create > UI Toolkit > Style Sheet
2. UI Builder > StyleSheets 패널 > + 버튼 > USS 파일 선택
3. Hierarchy에서 요소 선택
4. Inspector > Style Class List > 클래스명 입력
5. USS 파일에서 해당 클래스 스타일 정의
```

## 🧩 템플릿 사용

### 템플릿 등록

```
1. UI Builder에서 UXML 파일 저장 (ex: ItemSlot.uxml)
2. 다른 UXML에서 템플릿 사용:
   - Library > Project 탭
   - 해당 UXML 파일을 Viewport에 드래그
```

## 🎯 실습: 로그인 화면 만들기

### Step 1: 컨테이너 구조

```
1. Library > Containers > VisualElement 드래그
2. Inspector > Name: "login-container"
3. Inspector > Style Class List: "login-container"
4. Inspector > Style > Flex:
   - Grow: 1
   - Justify Content: Center
   - Align Items: Center
```

### Step 2: 타이틀

```
1. login-container 선택 상태에서 Library > Controls > Label 드래그
2. Inspector > Text: "어드벤처 게임"
3. Inspector > Style Class List: "title"
```

### Step 3: 입력 필드

```
1. Library > Controls > TextField 드래그
2. Inspector > Label: "아이디"
3. Inspector > Name: "username-field"
4. Inspector > Placeholder Text: "이메일을 입력하세요"

5. TextField 복사/붙여넣기
6. Inspector > Label: "비밀번호"
7. Inspector > Name: "password-field"
8. Inspector > Password: 체크
```

### Step 4: 버튼

```
1. Library > Controls > Button 드래그
2. Inspector > Text: "로그인"
3. Inspector > Name: "login-button"
4. Inspector > Style Class List: "primary-button"
```

### Step 5: USS 스타일링

**LoginScreen.uss**

```css
.login-container {
    background-color: rgba(0, 0, 0, 0.8);
    padding: 40px;
    border-radius: 20px;
    width: 400px;
}

.title {
    font-size: 32px;
    -unity-font-style: bold;
    color: white;
    -unity-text-align: middle-center;
    margin-bottom: 30px;
}

#username-field, #password-field {
    margin-bottom: 15px;
}

.primary-button {
    height: 50px;
    background-color: rgb(0, 120, 215);
    border-radius: 10px;
    color: white;
    font-size: 18px;
    margin-top: 20px;
}

.primary-button:hover {
    background-color: rgb(0, 150, 255);
}
```

## 🔧 UI Builder 팁

### ✅ 베스트 프랙티스

```
1. name 사용: C#에서 접근할 요소는 반드시 name 설정
2. class 사용: 스타일링은 클래스로 관리
3. USS 분리: 인라인 스타일 대신 USS 파일 사용
4. 템플릿 활용: 반복되는 UI 요소는 템플릿으로 분리
5. 프리뷰 테스트: Viewport의 Preview 버튼으로 테스트
```

### ⚡ 키보드 단축키

```
Ctrl/Cmd + S: 저장
Ctrl/Cmd + D: 복제
Ctrl/Cmd + Z: 실행 취소
Delete: 선택한 요소 삭제
F2: 요소 이름 변경
```

## 🚀 Runtime UI 테스트

### UIDocument 컴포넌트 추가

```
1. Hierarchy > 우클릭 > UI Toolkit > UI Document
2. Inspector > Source Asset: UXML 파일 할당
3. Inspector > Panel Settings: 새로 생성
   - Assets > Create > UI Toolkit > Panel Settings Asset
   - Scale Mode: Scale With Screen Size
   - Reference Resolution: 1920x1080
4. Play 버튼으로 테스트
```

---

# 💻 C# 스크립트로 UI 제어

## 📝 기본 설정

### UIDocument 참조

```csharp
using UnityEngine;
using UnityEngine.UIElements;

public class UIController : MonoBehaviour
{
    [SerializeField] private UIDocument _uiDocument;
    private VisualElement _root;
    
    private void Awake()
    {
        // rootVisualElement 가져오기
        _root = _uiDocument.rootVisualElement;
    }
}
```

## 🔍 요소 찾기 및 접근

### Q() 메서드 (권장)

```csharp
using UnityEngine.UIElements;

public class LoginController : MonoBehaviour
{
    private TextField _usernameField;
    private TextField _passwordField;
    private Button _loginButton;
    private Label _errorMessage;
    
    private void Start()
    {
        var root: VisualElement = GetComponent<UIDocument>().rootVisualElement;
        
        // name으로 요소 찾기
        _usernameField = root.Q<TextField>("username-field");
        _passwordField = root.Q<TextField>("password-field");
        _loginButton = root.Q<Button>("login-button");
        _errorMessage = root.Q<Label>("error-message");
        
        // null 체크
        if (_loginButton == null)
        {
            Debug.LogError($"login-button을 찾을 수 없습니다!");
            return;
        }
    }
}
```

### Query() 메서드 (여러 요소)

```csharp
// class로 모든 버튼 찾기
var menuButtons: UQueryBuilder<Button> = root.Query<Button>(className: "menu-button");

// ForEach로 순회
menuButtons.ForEach(button =>
{
    button.clicked += () => OnMenuButtonClicked(button.text);
});

// 리스트로 변환
List<Button> buttonList = menuButtons.ToList();

// 개수 확인
int count = menuButtons.ToList().Count;
```

## 🎯 이벤트 처리

### Button 클릭

```csharp
private void Start()
{
    var root: VisualElement = GetComponent<UIDocument>().rootVisualElement;
    
    // 방법 1: clicked 이벤트
    Button submitButton = root.Q<Button>("submit-button");
    submitButton.clicked += OnSubmitClicked;
    
    // 방법 2: 람다
    root.Q<Button>("cancel-button").clicked += () =>
    {
        Debug.Log($"취소 클릭!");
    };
}

private void OnSubmitClicked()
{
    Debug.Log($"제출 버튼 클릭!");
}

private void OnDestroy()
{
    // 이벤트 해제 (메모리 누수 방지)
    Button submitButton = GetComponent<UIDocument>().rootVisualElement.Q<Button>("submit-button");
    if (submitButton != null)
    {
        submitButton.clicked -= OnSubmitClicked;
    }
}
```

### TextField 값 변경

```csharp
private void Start()
{
    var root: VisualElement = GetComponent<UIDocument>().rootVisualElement;
    
    TextField nameField = root.Q<TextField>("name-field");
    
    // 방법 1: RegisterValueChangedCallback
    nameField.RegisterValueChangedCallback(evt =>
    {
        string oldValue = evt.previousValue;
        string newValue = evt.newValue;
        Debug.Log($"이름 변경: {oldValue} -> {newValue}");
    });
    
    // 방법 2: RegisterCallback (더 저수준)
    nameField.RegisterCallback<ChangeEvent<string>>(evt =>
    {
        Debug.Log($"새 값: {evt.newValue}");
    });
}
```

### Toggle/Slider/Dropdown

```csharp
private void Start()
{
    var root: VisualElement = GetComponent<UIDocument>().rootVisualElement;
    
    // Toggle
    Toggle muteToggle = root.Q<Toggle>("mute-toggle");
    muteToggle.RegisterValueChangedCallback(evt =>
    {
        bool isMuted = evt.newValue;
        AudioListener.volume = isMuted ? 0f : 1f;
    });
    
    // Slider
    Slider volumeSlider = root.Q<Slider>("volume-slider");
    volumeSlider.RegisterValueChangedCallback(evt =>
    {
        float volume = evt.newValue;
        AudioListener.volume = volume / 100f;
    });
    
    // DropdownField
    DropdownField difficultyDropdown = root.Q<DropdownField>("difficulty-dropdown");
    difficultyDropdown.RegisterValueChangedCallback(evt =>
    {
        string difficulty = evt.newValue;
        Debug.Log($"난이도: {difficulty}");
    });
}
```

## 🎨 스타일 동적 변경

### IStyle 사용

```csharp
// 색상 변경
[element.style](http://element.style).backgroundColor = new StyleColor([Color.red](http://Color.red));
[element.style](http://element.style).color = new StyleColor(Color.white);

// 크기 변경
[element.style](http://element.style).width = new StyleLength(200f); // px
[element.style](http://element.style).height = new StyleLength(Length.Percent(50)); // %

// 여백
[element.style](http://element.style).marginTop = new StyleLength(10f);
[element.style](http://element.style).paddingLeft = new StyleLength(20f);

// 테두리
[element.style](http://element.style).borderTopWidth = new StyleFloat(2f);
[element.style](http://element.style).borderTopColor = new StyleColor(Color.white);
[element.style](http://element.style).borderRadius = new StyleLength(10f);

// 표시/숨김
[element.style](http://element.style).display = DisplayStyle.None;
[element.style](http://element.style).visibility = Visibility.Hidden;

// 배경 이미지
Texture2D texture = Resources.Load<Texture2D>("Textures/background");
[element.style](http://element.style).backgroundImage = new StyleBackground(texture);

// Transform
[element.style](http://element.style).scale = new StyleScale(new Scale(new Vector3(1.5f, 1.5f, 1f)));
[element.style](http://element.style).rotate = new StyleRotate(new Rotate(45f));
```

### USS 클래스 토글 (권장)

```csharp
// 클래스 추가
element.AddToClassList("highlighted");

// 클래스 제거
element.RemoveFromClassList("highlighted");

// 클래스 토글
element.ToggleInClassList("highlighted");

// 클래스 존재 확인
bool hasClass = element.ClassListContains("highlighted");

// 여러 클래스 동시 적용
element.AddToClassList("card");
element.AddToClassList("active");
element.AddToClassList("highlighted");
```

## 🔧 실전 예제: 로그인 시스템

```csharp
using UnityEngine;
using UnityEngine.UIElements;
using System.Collections;

public class LoginController : MonoBehaviour
{
    [SerializeField] private UIDocument _uiDocument;
    
    private TextField _usernameField;
    private TextField _passwordField;
    private Button _loginButton;
    private Label _errorMessage;
    
    private void Start()
    {
        var root: VisualElement = _uiDocument.rootVisualElement;
        
        // 요소 찾기
        _usernameField = root.Q<TextField>("username-field");
        _passwordField = root.Q<TextField>("password-field");
        _loginButton = root.Q<Button>("login-button");
        _errorMessage = root.Q<Label>("error-message");
        
        // 이벤트 등록
        _loginButton.clicked += OnLoginClicked;
        
        // Enter 키 로그인
        _passwordField.RegisterCallback<KeyDownEvent>(evt =>
        {
            if (evt.keyCode == KeyCode.Return)
            {
                OnLoginClicked();
            }
        });
        
        // 초기 에러 메시지 숨김
        _[errorMessage.style](http://errorMessage.style).display = DisplayStyle.None;
    }
    
    private void OnLoginClicked()
    {
        string username = _usernameField.value;
        string password = _passwordField.value;
        
        // 유효성 검사
        if (string.IsNullOrEmpty(username))
        {
            ShowError("아이디를 입력하세요");
            return;
        }
        
        if (string.IsNullOrEmpty(password))
        {
            ShowError("비밀번호를 입력하세요");
            return;
        }
        
        // 로그인 처리
        StartCoroutine(LoginCoroutine(username, password));
    }
    
    private IEnumerator LoginCoroutine(string username: string, string password: string)
    {
        // 버튼 비활성화
        _loginButton.SetEnabled(false);
        _loginButton.text = "로그인 중...";
        
        // 가상의 API 호출
        yield return new WaitForSeconds(2f);
        
        // 간단한 인증 로직
        if (username == "admin" && password == "1234")
        {
            Debug.Log($"로그인 성공!");
            // 메인 화면으로 이동
        }
        else
        {
            ShowError("아이디 또는 비밀번호가 잘못되었습니다");
            
            // 버튼 재활성화
            _loginButton.SetEnabled(true);
            _loginButton.text = "로그인";
        }
    }
    
    private void ShowError(string message: string)
    {
        _errorMessage.text = message;
        _[errorMessage.style](http://errorMessage.style).display = DisplayStyle.Flex;
        
        // 3초 후 자동 숨김
        StartCoroutine(HideErrorAfterDelay(3f));
    }
    
    private IEnumerator HideErrorAfterDelay(float delay: float)
    {
        yield return new WaitForSeconds(delay);
        _[errorMessage.style](http://errorMessage.style).display = DisplayStyle.None;
    }
    
    private void OnDestroy()
    {
        // 이벤트 해제
        if (_loginButton != null)
        {
            _loginButton.clicked -= OnLoginClicked;
        }
    }
}
```

---

# 🔗 데이터 바인딩

## 📝 데이터 바인딩이란?

UI Toolkit의 **데이터 바인딩**은 UI 요소와 데이터 소스를 자동으로 동기화하는 기능입니다. **SerializedObject**를 기반으로 하며, 주로 **에디터 환경**에서 사용됩니다.

## 🎯 SerializedObject 바인딩 (에디터)

### 기본 바인딩

```csharp
using UnityEngine;

public class PlayerData : MonoBehaviour
{
    public string playerName = "Player";
    public int level = 1;
    public float health = 100f;
    public bool isAlive = true;
}
```

```csharp
using UnityEditor;
using UnityEngine.UIElements;

[CustomEditor(typeof(PlayerData))]
public class PlayerDataEditor : Editor
{
    public override VisualElement CreateInspectorGUI()
    {
        // 루트 컨테이너
        var root: VisualElement = new VisualElement();
        
        // TextField 생성 및 바인딩
        var nameField: TextField = new TextField("Player Name");
        nameField.bindingPath = "playerName"; // 프로퍼티명과 동일
        root.Add(nameField);
        
        // IntegerField 바인딩
        var levelField: IntegerField = new IntegerField("Level");
        levelField.bindingPath = "level";
        root.Add(levelField);
        
        // FloatField 바인딩
        var healthField: FloatField = new FloatField("Health");
        healthField.bindingPath = "health";
        root.Add(healthField);
        
        // Toggle 바인딩
        var aliveToggle: Toggle = new Toggle("Is Alive");
        aliveToggle.bindingPath = "isAlive";
        root.Add(aliveToggle);
        
        return root;
    }
}
```

### UXML에서 바인딩

```xml
<ui:UXML xmlns:ui="UnityEngine.UIElements" xmlns:uie="UnityEditor.UIElements">
    <ui:TextField label="Player Name" binding-path="playerName" />
    <ui:IntegerField label="Level" binding-path="level" />
    <ui:FloatField label="Health" binding-path="health" />
    <ui:Toggle label="Is Alive" binding-path="isAlive" />
</ui:UXML>
```

```csharp
using UnityEditor;
using UnityEngine.UIElements;

[CustomEditor(typeof(PlayerData))]
public class PlayerDataEditor : Editor
{
    [SerializeField] private VisualTreeAsset _uxml;
    
    public override VisualElement CreateInspectorGUI()
    {
        var root: VisualElement = new VisualElement();
        
        // UXML 로드
        _uxml.CloneTree(root);
        
        // 자동 바인딩 (반드시 호출!)
        return root;
    }
}
```

## 🧩 커스텀 컨트롤 바인딩

### 커스텀 컨트롤 정의

```csharp
using UnityEngine.UIElements;

public class HealthBar : VisualElement
{
    public new class UxmlFactory : UxmlFactory<HealthBar, UxmlTraits> { }
    
    public new class UxmlTraits : VisualElement.UxmlTraits
    {
        UxmlFloatAttributeDescription _maxHealth = new UxmlFloatAttributeDescription 
        { 
            name = "max-health", 
            defaultValue = 100f 
        };
        
        UxmlFloatAttributeDescription _currentHealth = new UxmlFloatAttributeDescription 
        { 
            name = "current-health", 
            defaultValue = 100f 
        };
        
        public override void Init(VisualElement ve: VisualElement, IUxmlAttributes bag: IUxmlAttributes, CreationContext cc: CreationContext)
        {
            base.Init(ve, bag, cc);
            var healthBar: HealthBar = ve as HealthBar;
            
            healthBar.MaxHealth = _maxHealth.GetValueFromBag(bag, cc);
            healthBar.CurrentHealth = _currentHealth.GetValueFromBag(bag, cc);
        }
    }
    
    private VisualElement _fillBar;
    private Label _label;
    
    private float _maxHealth;
    private float _currentHealth;
    
    public float MaxHealth
    {
        get => _maxHealth;
        set
        {
            _maxHealth = value;
            UpdateDisplay();
        }
    }
    
    public float CurrentHealth
    {
        get => _currentHealth;
        set
        {
            _currentHealth = value;
            UpdateDisplay();
        }
    }
    
    public HealthBar()
    {
        // 컨테이너
        AddToClassList("health-bar");
        
        // 배경
        var background: VisualElement = new VisualElement();
        background.AddToClassList("health-bar-background");
        Add(background);
        
        // 채우기 바
        _fillBar = new VisualElement();
        _fillBar.AddToClassList("health-bar-fill");
        background.Add(_fillBar);
        
        // 라벨
        _label = new Label();
        _label.AddToClassList("health-bar-label");
        Add(_label);
    }
    
    private void UpdateDisplay()
    {
        float percentage = _currentHealth / _maxHealth;
        _[fillBar.style](http://fillBar.style).width = Length.Percent(percentage * 100);
        _label.text = $"{_currentHealth:F0} / {_maxHealth:F0}";
        
        // 색상 변경
        if (percentage > 0.5f)
        {
            _[fillBar.style](http://fillBar.style).backgroundColor = [Color.green](http://Color.green);
        }
        else if (percentage > 0.2f)
        {
            _[fillBar.style](http://fillBar.style).backgroundColor = Color.yellow;
        }
        else
        {
            _[fillBar.style](http://fillBar.style).backgroundColor = [Color.red](http://Color.red);
        }
    }
}
```

### UXML에서 사용

```xml
<ui:UXML xmlns:ui="UnityEngine.UIElements">
    <HealthBar max-health="100" current-health="75" />
</ui:UXML>
```

### C#에서 사용

```csharp
var healthBar: HealthBar = new HealthBar();
healthBar.MaxHealth = 100f;
healthBar.CurrentHealth = 75f;
root.Add(healthBar);

// 체력 감소
healthBar.CurrentHealth -= 25f;
```

## 🚀 런타임 바인딩 (Unity 6+)

### DataBinding (Experimental)

```csharp
using [Unity.Properties](http://Unity.Properties); // Package Manager에서 설치 필요
using UnityEngine;
using UnityEngine.UIElements;

[System.Serializable]
public class PlayerStats
{
    [CreateProperty]
    public string Name { get; set; } = "Player";
    
    [CreateProperty]
    public int Level { get; set; } = 1;
    
    [CreateProperty]
    public float Health { get; set; } = 100f;
}

public class PlayerUI : MonoBehaviour
{
    [SerializeField] private UIDocument _uiDocument;
    private PlayerStats _playerStats = new PlayerStats();
    
    private void Start()
    {
        var root: VisualElement = _uiDocument.rootVisualElement;
        
        // 바인딩 설정
        var nameLabel: Label = root.Q<Label>("player-name");
        nameLabel.dataSource = _playerStats;
        nameLabel.SetBinding("text", new DataBinding { dataSourcePath = new PropertyPath("Name") });
        
        var levelLabel: Label = root.Q<Label>("player-level");
        levelLabel.dataSource = _playerStats;
        levelLabel.SetBinding("text", new DataBinding { dataSourcePath = new PropertyPath("Level") });
        
        // 데이터 변경 시 자동 업데이트
        StartCoroutine(UpdateStatsRoutine());
    }
    
    private IEnumerator UpdateStatsRoutine()
    {
        while (true)
        {
            yield return new WaitForSeconds(1f);
            _playerStats.Level++;
            _[playerStats.Health](http://playerStats.Health) -= 5f;
        }
    }
}
```

---

# 🧩 커스텀 컨트롤 제작

## 📝 커스텀 컨트롤이란?

**커스텀 컨트롤**은 VisualElement를 상속하여 재사용 가능한 UI 컴포넌트를 만드는 기능입니다.

## 🎯 기본 구조

### 간단한 커스텀 버튼

```csharp
using UnityEngine;
using UnityEngine.UIElements;

public class CustomButton : VisualElement
{
    // UI Builder에서 사용 가능하게 하려면 필수
    public new class UxmlFactory : UxmlFactory<CustomButton, UxmlTraits> { }
    
    // UXML 속성 정의
    public new class UxmlTraits : VisualElement.UxmlTraits
    {
        UxmlStringAttributeDescription _text = new UxmlStringAttributeDescription 
        { 
            name = "text", 
            defaultValue = "Button" 
        };
        
        public override void Init(VisualElement ve: VisualElement, IUxmlAttributes bag: IUxmlAttributes, CreationContext cc: CreationContext)
        {
            base.Init(ve, bag, cc);
            var button: CustomButton = ve as CustomButton;
            button.Text = _text.GetValueFromBag(bag, cc);
        }
    }
    
    // 내부 요소
    private Label _label;
    private VisualElement _icon;
    
    // 프로퍼티
    private string _text;
    public string Text
    {
        get => _text;
        set
        {
            _text = value;
            _label.text = value;
        }
    }
    
    // 이벤트
    public event System.Action Clicked;
    
    public CustomButton()
    {
        // 스타일 클래스
        AddToClassList("custom-button");
        
        // 아이콘
        _icon = new VisualElement();
        _icon.AddToClassList("custom-button-icon");
        Add(_icon);
        
        // 라벨
        _label = new Label();
        _label.AddToClassList("custom-button-label");
        Add(_label);
        
        // 클릭 이벤트 등록
        RegisterCallback<ClickEvent>(OnClick);
    }
    
    private void OnClick(ClickEvent evt: ClickEvent)
    {
        Clicked?.Invoke();
    }
    
    // 아이콘 설정
    public void SetIcon(Texture2D texture: Texture2D)
    {
        _[icon.style](http://icon.style).backgroundImage = new StyleBackground(texture);
    }
}
```

### USS 스타일

```css
.custom-button {
    flex-direction: row;
    align-items: center;
    background-color: rgb(0, 120, 215);
    border-radius: 10px;
    padding: 10px 20px;
    cursor: link;
}

.custom-button:hover {
    background-color: rgb(0, 150, 255);
}

.custom-button:active {
    background-color: rgb(0, 100, 200);
}

.custom-button-icon {
    width: 24px;
    height: 24px;
    margin-right: 10px;
}

.custom-button-label {
    color: white;
    font-size: 16px;
    -unity-font-style: bold;
}
```

### UXML에서 사용

```xml
<ui:UXML xmlns:ui="UnityEngine.UIElements">
    <CustomButton text="커스텀 버튼" />
</ui:UXML>
```

### C#에서 사용

```csharp
var customButton: CustomButton = new CustomButton();
customButton.Text = "클릭하세요";
customButton.SetIcon(myIconTexture);
customButton.Clicked += () =>
{
    Debug.Log($"커스텀 버튼 클릭!");
};
root.Add(customButton);
```

## 🎨 고급 예제: Progress Bar

```csharp
using UnityEngine;
using UnityEngine.UIElements;

public class ProgressBar : VisualElement
{
    public new class UxmlFactory : UxmlFactory<ProgressBar, UxmlTraits> { }
    
    public new class UxmlTraits : VisualElement.UxmlTraits
    {
        UxmlFloatAttributeDescription _min = new UxmlFloatAttributeDescription 
        { 
            name = "min-value", 
            defaultValue = 0f 
        };
        
        UxmlFloatAttributeDescription _max = new UxmlFloatAttributeDescription 
        { 
            name = "max-value", 
            defaultValue = 100f 
        };
        
        UxmlFloatAttributeDescription _value = new UxmlFloatAttributeDescription 
        { 
            name = "value", 
            defaultValue = 0f 
        };
        
        UxmlStringAttributeDescription _label = new UxmlStringAttributeDescription 
        { 
            name = "label", 
            defaultValue = "" 
        };
        
        public override void Init(VisualElement ve: VisualElement, IUxmlAttributes bag: IUxmlAttributes, CreationContext cc: CreationContext)
        {
            base.Init(ve, bag, cc);
            var bar: ProgressBar = ve as ProgressBar;
            
            bar.MinValue = _min.GetValueFromBag(bag, cc);
            bar.MaxValue = _max.GetValueFromBag(bag, cc);
            bar.Value = _value.GetValueFromBag(bag, cc);
            bar.Label = _label.GetValueFromBag(bag, cc);
        }
    }
    
    private VisualElement _background;
    private VisualElement _fill;
    private Label _labelElement;
    private Label _percentElement;
    
    private float _minValue = 0f;
    private float _maxValue = 100f;
    private float _value = 0f;
    private string _label = "";
    
    public float MinValue
    {
        get => _minValue;
        set
        {
            _minValue = value;
            UpdateDisplay();
        }
    }
    
    public float MaxValue
    {
        get => _maxValue;
        set
        {
            _maxValue = value;
            UpdateDisplay();
        }
    }
    
    public float Value
    {
        get => _value;
        set
        {
            _value = Mathf.Clamp(value, _minValue, _maxValue);
            UpdateDisplay();
        }
    }
    
    public string Label
    {
        get => _label;
        set
        {
            _label = value;
            _labelElement.text = value;
            _[labelElement.style](http://labelElement.style).display = string.IsNullOrEmpty(value) ? DisplayStyle.None : DisplayStyle.Flex;
        }
    }
    
    public ProgressBar()
    {
        AddToClassList("progress-bar");
        
        // 라벨
        _labelElement = new Label();
        _labelElement.AddToClassList("progress-bar-label");
        Add(_labelElement);
        
        // 배경
        _background = new VisualElement();
        _background.AddToClassList("progress-bar-background");
        Add(_background);
        
        // 채우기
        _fill = new VisualElement();
        _fill.AddToClassList("progress-bar-fill");
        _background.Add(_fill);
        
        // 퍼센트
        _percentElement = new Label();
        _percentElement.AddToClassList("progress-bar-percent");
        _background.Add(_percentElement);
        
        UpdateDisplay();
    }
    
    private void UpdateDisplay()
    {
        float range = _maxValue - _minValue;
        float normalized = (range > 0) ? (_value - _minValue) / range : 0f;
        float percent = normalized * 100f;
        
        // 채우기 너비
        _[fill.style](http://fill.style).width = Length.Percent(percent);
        
        // 퍼센트 표시
        _percentElement.text = $"{percent:F0}%";
        
        // 색상 그라데이션
        Color fillColor = Color.Lerp([Color.red](http://Color.red), [Color.green](http://Color.green), normalized);
        _[fill.style](http://fill.style).backgroundColor = fillColor;
    }
    
    // 부드럽게 증가
    public void AnimateToValue(float targetValue: float, float duration: float)
    {
        VisualElement ve = this;
        ve.schedule.Execute(() =>
        {
            float current = Value;
            float target = Mathf.Clamp(targetValue, _minValue, _maxValue);
            float step = (target - current) * Time.deltaTime / duration;
            
            Value = current + step;
            
            if (Mathf.Abs(target - current) < 0.1f)
            {
                Value = target;
            }
        }).Every(16); // ~60 FPS
    }
}
```

### USS

```css
.progress-bar {
    width: 300px;
}

.progress-bar-label {
    font-size: 14px;
    color: white;
    margin-bottom: 5px;
}

.progress-bar-background {
    height: 30px;
    background-color: rgba(50, 50, 50, 0.9);
    border-radius: 15px;
    overflow: hidden;
    position: relative;
}

.progress-bar-fill {
    height: 100%;
    background-color: rgb(0, 200, 0);
    transition: width 0.3s, background-color 0.3s;
}

.progress-bar-percent {
    position: absolute;
    width: 100%;
    height: 100%;
    justify-content: center;
    align-items: center;
    color: white;
    font-size: 14px;
    -unity-font-style: bold;
}
```

### 사용 예제

```csharp
// UXML
<ProgressBar label="로딩" min-value="0" max-value="100" value="0" />

// C#
var progressBar: ProgressBar = new ProgressBar();
progressBar.Label = "로딩";
progressBar.MinValue = 0f;
progressBar.MaxValue = 100f;
progressBar.Value = 0f;
root.Add(progressBar);

// 진행률 업데이트
progressBar.AnimateToValue(100f, 3f); // 3초동안 100%까지
```

---

# 💚 반응형 UI 디자인

## 📝 반응형 디자인 기초

UI Toolkit은 **퍼센트 기반 크기**와 **Flexbox**를 통해 다양한 화면 크기에 대응하는 반응형 UI를 만들 수 있습니다.

## 🎯 Panel Settings 구성

```
Panel Settings Asset 설정:

- Scale Mode: Scale With Screen Size
- Reference Resolution: 1920 x 1080
- Screen Match Mode: Match Width Or Height
- Match: 0.5 (Width와 Height 균형)
```

### CSS로 반응형 크기

```css
/* 퍼센트 기반 */
.responsive-panel {
    width: 80%; /* 부모의 80% */
    max-width: 1200px; /* 최대 크기 제한 */
    min-width: 320px; /* 최소 크기 보장 */
}

/* Viewport 단위 */
.fullscreen {
    width: 100%;
    height: 100%;
}
```

## 📱 모바일/태블릿/PC 대응

```css
/* 기본 레이아웃 (모바일) */
.container {
    flex-direction: column;
    padding: 10px;
}

/* 태블릿 (중간 크기) */
.container-tablet {
    flex-direction: row;
    padding: 20px;
}

/* 데스크톱 (크 크기) */
.container-desktop {
    flex-direction: row;
    padding: 30px;
    max-width: 1200px;
}
```

### C#에서 화면 크기 감지

```csharp
using UnityEngine;
using UnityEngine.UIElements;

public class ResponsiveUI : MonoBehaviour
{
    [SerializeField] private UIDocument _uiDocument;
    private VisualElement _container;
    
    private void Start()
    {
        var root = _uiDocument.rootVisualElement;
        _container = root.Q<VisualElement>("container");
        
        UpdateLayout();
    }
    
    private void Update()
    {
        // 화면 크기 변경 감지
        if (Screen.width != _lastWidth || Screen.height != _lastHeight)
        {
            UpdateLayout();
            _lastWidth = Screen.width;
            _lastHeight = Screen.height;
        }
    }
    
    private int _lastWidth;
    private int _lastHeight;
    
    private void UpdateLayout()
    {
        _container.RemoveFromClassList("container-mobile");
        _container.RemoveFromClassList("container-tablet");
        _container.RemoveFromClassList("container-desktop");
        
        if (Screen.width < 768)
        {
            _container.AddToClassList("container-mobile");
        }
        else if (Screen.width < 1024)
        {
            _container.AddToClassList("container-tablet");
        }
        else
        {
            _container.AddToClassList("container-desktop");
        }
    }
}
```

---

# 🎥 애니메이션 및 트랜지션

## 📝 USS Transition

**USS Transition**은 CSS transition을 기반으로 한 간단한 애니메이션 기능입니다.

### 기본 Transition

```css
.animated-button {
    background-color: rgb(0, 120, 215);
    scale: 1;
    
    /* transition: property duration ease */
    transition: background-color 0.3s, scale 0.2s;
}

.animated-button:hover {
    background-color: rgb(0, 150, 255);
    scale: 1.1;
}
```

### 여러 속성 Transition

```css
.card {
    opacity: 1;
    translate: 0 0;
    rotate: 0deg;
    scale: 1;
    
    transition: 
        opacity 0.3s ease-in-out,
        translate 0.5s ease-out,
        rotate 0.4s ease-in-out,
        scale 0.2s ease-out;
}

.card.hidden {
    opacity: 0;
    translate: 0 -50px;
}

.card.rotated {
    rotate: 180deg;
}
```

### Easing 함수

```css
.element {
    transition: all 0.3s ease-in-out;
    /* ease, ease-in, ease-out, ease-in-out, linear */
}
```

## 🎯 Transition 이벤트

```csharp
using UnityEngine.UIElements;

public class TransitionController : MonoBehaviour
{
    private void Start()
    {
        var root = GetComponent<UIDocument>().rootVisualElement;
        var element = root.Q<VisualElement>("animated-element");
        
        // Transition 시작 이벤트
        element.RegisterCallback<TransitionStartEvent>(evt =>
        {
            Debug.Log($"Transition 시작: {evt.stylePropertyNames}");
        });
        
        // Transition 종료 이벤트
        element.RegisterCallback<TransitionEndEvent>(evt =>
        {
            Debug.Log($"Transition 종료: {evt.stylePropertyNames}");
        });
    }
}
```

## 🚀 C#로 애니메이션 제어

### Schedule 사용 (권장)

```csharp
// 페이드 인 애니메이션
public void FadeIn(VisualElement element, float duration)
{
    [element.style](http://element.style).opacity = 0;
    
    float elapsed = 0f;
    element.schedule.Execute(() =>
    {
        elapsed += 0.016f; // ~60 FPS
        float progress = Mathf.Clamp01(elapsed / duration);
        [element.style](http://element.style).opacity = progress;
        
        if (progress >= 1f)
        {
            return; // 종료
        }
    }).Every(16); // 16ms = 60 FPS
}

// 슬라이드 인 애니메이션
public void SlideIn(VisualElement element, float duration)
{
    [element.style](http://element.style).translate = new Translate(0, -100, 0);
    
    float elapsed = 0f;
    element.schedule.Execute(() =>
    {
        elapsed += 0.016f;
        float progress = Mathf.Clamp01(elapsed / duration);
        float y = Mathf.Lerp(-100f, 0f, progress);
        [element.style](http://element.style).translate = new Translate(0, y, 0);
        
        if (progress >= 1f)
        {
            return;
        }
    }).Every(16);
}
```

### Coroutine 사용

```csharp
using System.Collections;

public IEnumerator PulseAnimation(VisualElement element)
{
    while (true)
    {
        // 확대
        float elapsed = 0f;
        while (elapsed < 0.5f)
        {
            elapsed += Time.deltaTime;
            float scale = Mathf.Lerp(1f, 1.2f, elapsed / 0.5f);
            [element.style](http://element.style).scale = new Scale(new Vector3(scale, scale, 1f));
            yield return null;
        }
        
        // 축소
        elapsed = 0f;
        while (elapsed < 0.5f)
        {
            elapsed += Time.deltaTime;
            float scale = Mathf.Lerp(1.2f, 1f, elapsed / 0.5f);
            [element.style](http://element.style).scale = new Scale(new Vector3(scale, scale, 1f));
            yield return null;
        }
    }
}
```

---

# 🎮 실전 예제

## 🎮 예제 1: 게임 HUD

```csharp
using UnityEngine;
using UnityEngine.UIElements;

public class GameHUD : MonoBehaviour
{
    [SerializeField] private UIDocument _uiDocument;
    
    private Label _scoreLabel;
    private Label _timerLabel;
    private VisualElement _healthBar;
    private VisualElement _healthFill;
    
    private int _score = 0;
    private float _health = 100f;
    private float _gameTime = 0f;
    
    private void Start()
    {
        var root = _uiDocument.rootVisualElement;
        
        _scoreLabel = root.Q<Label>("score-label");
        _timerLabel = root.Q<Label>("timer-label");
        _healthBar = root.Q<VisualElement>("health-bar");
        _healthFill = root.Q<VisualElement>("health-fill");
        
        UpdateUI();
    }
    
    private void Update()
    {
        _gameTime += Time.deltaTime;
        UpdateTimer();
    }
    
    public void AddScore(int points)
    {
        _score += points;
        _scoreLabel.text = $"점수: {_score}";
        
        // 점수 증가 효과
        StartCoroutine(ScorePopAnimation());
    }
    
    public void TakeDamage(float damage)
    {
        _health = Mathf.Max(0, _health - damage);
        UpdateHealth();
        
        if (_health <= 0)
        {
            OnGameOver();
        }
    }
    
    private void UpdateHealth()
    {
        float percentage = _health / 100f;
        _[healthFill.style](http://healthFill.style).width = Length.Percent(percentage * 100);
        
        // 색상 변경
        if (percentage > 0.5f)
            _[healthFill.style](http://healthFill.style).backgroundColor = [Color.green](http://Color.green);
        else if (percentage > 0.2f)
            _[healthFill.style](http://healthFill.style).backgroundColor = Color.yellow;
        else
            _[healthFill.style](http://healthFill.style).backgroundColor = [Color.red](http://Color.red);
    }
    
    private void UpdateTimer()
    {
        int minutes = Mathf.FloorToInt(_gameTime / 60f);
        int seconds = Mathf.FloorToInt(_gameTime % 60f);
        _timerLabel.text = $"{minutes:00}:{seconds:00}";
    }
    
    private IEnumerator ScorePopAnimation()
    {
        _[scoreLabel.style](http://scoreLabel.style).scale = new Scale([Vector3.one](http://Vector3.one) * 1.3f);
        yield return new WaitForSeconds(0.2f);
        _[scoreLabel.style](http://scoreLabel.style).scale = new Scale([Vector3.one](http://Vector3.one));
    }
    
    private void OnGameOver()
    {
        Debug.Log($"게임 오버! 최종 점수: {_score}");
    }
    
    private void UpdateUI()
    {
        _scoreLabel.text = $"점수: {_score}";
        UpdateHealth();
        UpdateTimer();
    }
}
```

## 🎮 예제 2: 인벤토리 시스템

```csharp
using UnityEngine;
using UnityEngine.UIElements;
using System.Collections.Generic;

public class InventorySystem : MonoBehaviour
{
    [SerializeField] private UIDocument _uiDocument;
    [SerializeField] private VisualTreeAsset _itemSlotTemplate;
    
    private VisualElement _inventoryGrid;
    private List<ItemData> _items = new List<ItemData>();
    
    private void Start()
    {
        var root = _uiDocument.rootVisualElement;
        _inventoryGrid = root.Q<VisualElement>("inventory-grid");
        
        // 30개 슬롯 생성
        for (int i = 0; i < 30; i++)
        {
            CreateItemSlot(i);
        }
        
        // 테스트 아이템 추가
        AddItem(new ItemData { name = "검", icon = null, count = 1 });
        AddItem(new ItemData { name = "포션", icon = null, count = 5 });
    }
    
    private void CreateItemSlot(int index)
    {
        var slot = _itemSlotTemplate.CloneTree();
        slot.AddToClassList("item-slot");
        [slot.name](http://slot.name) = $"slot-{index}";
        
        // 클릭 이벤트
        slot.RegisterCallback<ClickEvent>(evt =>
        {
            OnSlotClicked(index);
        });
        
        _inventoryGrid.Add(slot);
    }
    
    public void AddItem(ItemData item)
    {
        _items.Add(item);
        UpdateSlot(_items.Count - 1, item);
    }
    
    private void UpdateSlot(int index, ItemData item)
    {
        var slot = _inventoryGrid.Q<VisualElement>($"slot-{index}");
        
        var icon = slot.Q<VisualElement>("item-icon");
        var nameLabel = slot.Q<Label>("item-name");
        var countLabel = slot.Q<Label>("item-count");
        
        if (item.icon != null)
            [icon.style](http://icon.style).backgroundImage = new StyleBackground(item.icon);
        
        nameLabel.text = [item.name](http://item.name);
        countLabel.text = $"x{item.count}";
        
        slot.RemoveFromClassList("empty");
        slot.AddToClassList("filled");
    }
    
    private void OnSlotClicked(int index)
    {
        if (index < _items.Count)
        {
            Debug.Log($"아이템 사용: {_items[index].name}");
        }
    }
}

[System.Serializable]
public class ItemData
{
    public string name;
    public Texture2D icon;
    public int count;
}
```

---

# 🔧 트러블슈팅

## ⚠️ 일반적인 문제

### 1. 요소를 찾을 수 없음

```csharp
// 문제
var button = root.Q<Button>("my-button");
if (button == null) // null!

// 해결방법
1. UXML의 name 속성 확인
2. UI Builder에서 name 설정 확인
3. UIDocument가 제대로 로드되었는지 확인
4. 타이밍 문제 (너무 빨리 접근) - Awake 대신 Start 사용
```

### 2. USS 스타일이 적용되지 않음

```css
/* 문제 */
.my-button { color: red; } /* 적용 안됨 */

/* 해결방법 */
1. USS 파일이 UXML에 연결되었는지 확인
   <Style src="/Assets/UI/MyStyles.uss" />

2. 클래스명이 정확한지 확인
   element.AddToClassList("my-button");

3. 선택자 우선순위 확인
   #id > .class > Type

4. UI Toolkit Debugger로 스타일 확인
   Window > UI Toolkit > Debugger
```

### 3. 이벤트가 발생하지 않음

```csharp
// 문제
button.clicked += OnButtonClicked; // 호출 안됨

// 해결방법
1. 이벤트 등록 타이밍 확인
2. StopPropagation()으로 이벤트가 차단되었는지 확인
3. SetEnabled(false) 상태 확인
4. 다른 요소가 마우스 이벤트를 가로채는지 확인
```

### 4. 메모리 누수

```csharp
// 문제
_button.clicked += OnButtonClicked;
// OnDestroy에서 해제 안함

// 해결방법
private void OnDestroy()
{
    if (_button != null)
    {
        _button.clicked -= OnButtonClicked;
    }
}
```

## 🔍 디버깅 도구

### UI Toolkit Debugger

```
Window > UI Toolkit > Debugger

- Pick Element: 요소 선택 및 검사
- Styles: 적용된 모든 USS 스타일 확인
- Layout: Flexbox 레이아웃 박스 모델 확인
- Events: 이벤트 흐름 추적
```

### Console 로그

```csharp
// 요소 정보 출력
Debug.Log($"Element: {[element.name](http://element.name)}, Class: {string.Join(", ", element.GetClasses())}");

// 계층 구조 출력
void PrintHierarchy(VisualElement element, int depth = 0)
{
    string indent = new string(' ', depth * 2);
    Debug.Log($"{indent}{element.GetType().Name} [{[element.name](http://element.name)}]");
    
    foreach (var child in element.Children())
    {
        PrintHierarchy(child, depth + 1);
    }
}
```

---

# 📚 참고 자료

## 공식 문서

- [Unity UI Toolkit Manual](https://docs.unity3d.com/Manual/UIElements.html)
- [UI Toolkit Runtime Guide](https://blog.unity.com/engine-platform/ui-toolkit-at-runtime-get-the-breakdown)
- [Unity Learn: Getting Started with UI Toolkit](https://learn.unity.com/tutorial/getting-started-with-ui-toolkit)
- [UI Toolkit Samples](https://github.com/Unity-Technologies/ui-toolkit-manual-code-examples)

## 커뮤니티 리소스

- [Unity Forums - UI Toolkit](https://discussions.unity.com/c/unity-engine/ui-toolkit/)
- [GitHub - Unity UI Toolkit Examples](https://github.com/unity-e-book/UIToolkit)
- [Medium - UI Toolkit Articles](https://medium.com/search?q=unity%20ui%20toolkit)

## 비교 가이드

- [Unity UI Toolkit vs UGUI: 2025 Guide](https://www.angry-shark-studio.com/blog/unity-ui-toolkit-vs-ugui-2025-guide/)
- [UI Systems Comparison](https://docs.unity3d.com/Manual/UI-system-compare.html)
- [Migration Guide from uGUI](https://docs.unity3d.com/Manual/UIE-Transitioning-From-UGUI.html)

## 고급 토픽

- [Flexbox Layout Guide](https://discussions.unity.com/t/ui-toolkit-introduction-and-flexbox-layout/316856)
- [Custom Controls Guide](https://learn.unity.com/tutorial/ui-toolkit-in-unity-6-crafting-custom-controls-with-data-bindings)
- [USS Transitions](https://docs.unity3d.com/Manual/UIE-Transitions.html)
- [Data Binding Manual](https://docs.unity3d.com/Manual/UIE-Binding.html)

---

# 🎉 결론

Unity UI Toolkit은 **웹 기술의 유연성**과 **Unity의 강력함**을 결합한 차세대 UI 프레임워크입니다.

## 핵심 요약

### 주요 장점

- ⚡ **뛰어난 성능**: GameObject 오버헤드 제거, 배칭된 렌더링
- 🛠️ **생산성 향상**: UI Builder로 시각적 편집, UXML/USS 분리
- 🌍 **웹 개발자 친화적**: CSS Flexbox, HTML 유사 문법
- 🔄**크로스 플랫폼**: 에디터와 런타임 통합

### 고려 사항

- ⚠️ 3D 월드 공간 UI 미지원
- ⚠️ 복잡한 애니메이션 제한적
- ⚠️ 웹 기술 학습 필요

### 권장 사항

🎓 **신규 프로젝트**는 UI Toolkit으로 시작하세요!

🔄 **기존 프로젝트**는 하이브리드 접근을 고려하세요.

📊 **데이터 중심 UI**는 UI Toolkit이 최적입니다.

---

> 🚀 **Generated with [Claude Code](https://claude.com/claude-code)**
> 

> 📌 **마지막 업데이트**: 2026-01-15
>