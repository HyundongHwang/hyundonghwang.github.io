---
layout: post
title: 260203 navisworks addin development guide
comments: true
tags:
- autodesk
- bim
- navisworks
- optimization
- windows
- api
- research
- guide
- analysis
---
# 🛠️ 260203 Navisworks Addin 개발 가이드

## 📚 목차

1. [소개](#소개)
2. [필요성](#필요성)
3. [기능](#기능)
4. [개발환경](#개발환경)
5. [개발 프로세스](#개발-프로세스)
6. [배포](#배포)
7. [테스트](#테스트)
8. [예시 코드](#예시-코드)

---

## 🧭 소개

### 🔹 Navisworks Addin이란?

Autodesk Navisworks는 AEC(Architecture, Engineering, Construction) 산업에서 3D 모델 검토, 조정, 시뮬레이션을 위한 프로젝트 검토 소프트웨어입니다. **Navisworks Addin(Add-in 또는 Plugin)**은 Navisworks의 기본 기능을 확장하고 커스터마이징하기 위해 개발자가 작성하는 확장 프로그램입니다.

Navisworks는 세 가지 주요 API를 제공합니다:

1. **.NET API** - C#, VB.NET 등을 사용하여 커스텀 플러그인을 작성하고, GUI 외부에서 Navisworks를 제어하며, 특정 작업을 자동화할 수 있습니다.
2. **NWCreate API** - 지오메트리를 생성하고 NWC 파일로 내보낼 수 있습니다.
3. **COM API** - 레거시 API 옵션으로, .NET API에서 노출되지 않은 일부 기능(커스텀 속성 추가, 하이퍼링크 조작, 빠른 속성, 단면 및 줌 등)에 접근할 수 있습니다.

### 🧭 API 구조 개요

```
┌─────────────────────────────────────────┐
│       Navisworks Application            │
│         (Roamer.exe)                    │
└─────────────────┬───────────────────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
┌───────▼────────┐  ┌──────▼─────────┐
│   .NET API     │  │   COM API      │
│  (Primary)     │  │   (Legacy)     │
└───────┬────────┘  └──────┬─────────┘
        │                  │
┌───────▼──────────────────▼─────────┐
│      Your Custom Plugin            │
│  - AddInPlugin                     │
│  - DockPanePlugin                  │
│  - CommandHandlerPlugin            │
└────────────────────────────────────┘
```

SDK는 기본적으로 Autodesk Navisworks Manage 및 Simulate의 `\api\` 폴더에 설치되며, Developer's Guide(PDF 형식)와 Reference Guide(CHM 파일)를 포함합니다.

---

## 🎯 필요성

### 🎯 왜 커스텀 Addin이 필요한가?

Navisworks의 기본 기능은 강력하지만, 프로젝트나 조직의 특정 요구사항을 충족하기 위해서는 커스터마이징이 필요합니다:

1. **자동화**: 반복적인 작업을 자동화하여 시간과 비용을 절감합니다.
   - 예: 수백 개의 NWC 파일을 자동으로 로드하고 충돌 검사 실행
   - 예: 속성 데이터를 데이터베이스로 자동 추출

2. **데이터 통합**: Navisworks 모델 데이터를 외부 시스템(ERP, BIM 360, 데이터베이스 등)과 통합합니다.
   - 모델 속성을 외부 데이터베이스와 동기화
   - 클라우드 기반 협업 플랫폼과 연동

3. **커스텀 워크플로우**: 조직의 고유한 프로세스에 맞는 도구를 개발합니다.
   - 품질 관리 체크리스트 자동화
   - 커스텀 리포트 생성

4. **고급 분석**: 기본 제공되지 않는 고급 분석 기능을 구현합니다.
   - 지오메트리 충돌 분석
   - 공간 활용도 분석
   - 시공 순서 시뮬레이션

5. **사용자 경험 개선**: 특정 사용자 그룹에 최적화된 인터페이스를 제공합니다.
   - 커스텀 도킹 패널로 빠른 액세스
   - 자주 사용하는 기능을 리본에 통합

### 🔹 실무 사례

- **시공사**: 일일 진척도 추적 및 4D 시뮬레이션 자동화
- **설계 회사**: 설계 검증 및 규정 준수 체크 자동화
- **시설 관리**: 유지보수 일정 및 자산 추적 통합
- **품질 관리**: 자동화된 모델 검증 및 리포트 생성

---

## 📌 기능

### 🔹 Addin으로 할 수 있는 작업들

Navisworks .NET API를 사용하여 다음과 같은 작업을 수행할 수 있습니다:

#### ▫️ 1. 모델 데이터 액세스

- **ModelItem 탐색**: 모델의 계층 구조를 탐색하고 개별 요소에 액세스
- **속성 읽기/쓰기**: 요소의 속성(Properties) 데이터 읽기 및 커스텀 속성 추가
- **지오메트리 정보**: 위치, 회전, 바운딩 박스, 메시 데이터 추출
- **검색(Search)**: GUI의 'Find' 기능과 동일한 검색 기능 구현

#### ▫️ 2. 뷰포트 제어

- **카메라 조작**: 뷰포인트(Viewpoint) 생성, 수정, 저장
- **시각화 제어**: 색상 오버라이드, 투명도 설정, 표시/숨김
- **섹션 박스**: 프로그래밍 방식으로 단면 생성

#### 🧠 3. 충돌 검사(Clash Detection)

- **자동화된 충돌 테스트**: 충돌 검사 자동 실행 및 결과 추출
- **커스텀 충돌 규칙**: 특정 조건에 따른 충돌 검사 설정
- **리포트 생성**: 충돌 결과를 자동으로 리포트로 생성

#### ▫️ 4. 타임라인 및 시뮬레이션

- **4D 시뮬레이션**: 타임라인 애니메이션 자동화
- **작업 일정 연동**: 프로젝트 일정과 모델 연동

#### 🗂️ 5. 파일 작업

- **파일 로드/저장**: NWD, NWC, NWF 파일 프로그래밍 방식으로 열기/저장
- **일괄 처리**: 다수의 파일에 대한 배치 작업
- **내보내기**: 스크린샷, 리포트, 데이터 추출

#### ▫️ 6. 사용자 인터페이스 확장

- **커스텀 리본**: 리본에 사용자 정의 탭 및 버튼 추가
- **도킹 패널**: 사이드바에 커스텀 패널 추가
- **다이얼로그**: WPF 또는 WinForms 기반 사용자 인터페이스 생성

#### ▫️ 7. 외부 시스템 통합

- **데이터베이스 연결**: SQL Server, Oracle 등과 데이터 동기화
- **웹 서비스**: REST API, SOAP 서비스 호출
- **파일 시스템**: Excel, PDF, XML 등 외부 파일 읽기/쓰기

---

## 📌 개발환경

### 🛠️ Visual Studio 설정

#### ▫️ 필수 요구사항

| Navisworks 버전 | .NET Framework | Visual Studio | 폴더 버전 |
|----------------|----------------|---------------|----------|
| 2021 | 4.7 | 2019 이상 | v20 |
| 2024 | 4.8 | 2022 이상 | v22 |
| 2026 | 4.8 | 2022 이상 | v23 |

#### ▫️ 프로젝트 유형

- **Class Library (.NET Framework)** 프로젝트로 시작
- 출력 형식: DLL (Dynamic Link Library)
- 플랫폼 타겟: x64 (64비트)

#### 🛠️ Visual Studio 설치 구성 요소

Visual Studio Installer에서 다음 워크로드를 설치해야 합니다:

- **.NET desktop development** (필수)
- **Desktop development with C++** (일부 고급 기능에 필요)

### 💻 Navisworks API/SDK

#### ▫️ SDK 위치

SDK는 Navisworks 설치 시 자동으로 포함됩니다:

```
C:\Program Files\Autodesk\Navisworks Manage 20##\api\
```

SDK에 포함된 내용:

- **Developer's Guide (PDF)**: API 개요, 클래스, 도구 설명
- **Reference Guide (CHM)**: 상세한 API 참조 문서
- **샘플 코드**: 다양한 플러그인 예제

#### ▫️ 필수 참조 DLL

프로젝트에 다음 DLL을 참조로 추가해야 합니다:

**기본 API:**
```
C:\Program Files\Autodesk\Navisworks Manage 20##\Autodesk.Navisworks.Api.dll
```

**COM API (필요시):**
```
C:\Program Files\Autodesk\Navisworks Manage 20##\Autodesk.Navisworks.Interop.ComApi.dll
```

**UI 확장 (필요시):**
```
C:\Program Files\Autodesk\Navisworks Manage 20##\AdWindows.dll
```

#### 🛠️ 참조 속성 설정

각 Navisworks API DLL에 대해:

1. Solution Explorer에서 참조 우클릭 > Properties
2. **Copy Local**: `False` (매우 중요!)
   - Navisworks가 설치된 DLL을 사용해야 하므로 복사하지 않음

### 🔹 필요한 NuGet 패키지

#### ▫️ 공식 버전별 패키지

NuGet Package Manager를 통해 설치 가능:

**Navisworks 2020:**
```
Install-Package Navisworks-2020-x64-API -Version 1.0.0
```

**Navisworks 2024:**
```
Install-Package Chuongmep.Navis.Api.Autodesk.Navisworks.Api -Version 2024.0.0
```

**Navisworks 2025:**
```
Install-Package Chuongmep.Navis.Api.Autodesk.Navisworks.Api -Version 2025.0.0
```

#### ▫️ .NET CLI 사용:

```bash
dotnet add package Navisworks-2020-x64-API --version 1.0.0
```

#### ⚠️ 주의사항

- NuGet 패키지를 사용하는 경우, 로컬 DLL 참조와 충돌하지 않도록 주의
- 팀 프로젝트의 경우, 모든 팀원이 동일한 Navisworks 버전을 사용하는지 확인

### 🔹 라이선스 요구사항

#### ▫️ 개발자 라이선스

**중요:** Navisworks API 개발 및 플러그인 실행을 위해서는 최소한 **Navisworks Simulate 라이선스**가 필요합니다.

- **Navisworks Freedom**: API 자동화 불가 (무료 뷰어)
- **Navisworks Simulate**: API 사용 가능 (최소 요구사항)
- **Navisworks Manage**: API 사용 가능 + 추가 기능

#### ▫️ 라이선스 우선순위

라이선스 캐스케이딩(license cascading) 메커니즘:

1. Manage 라이선스가 우선순위가 높음
2. Simulate 라이선스가 없으면 Manage 라이선스가 사용될 수 있음
3. .NET 플러그인은 Simulate 또는 Manage 애플리케이션에 의해 로드됨

#### ▫️ 배포 시 고려사항

- **최종 사용자도 Simulate 또는 Manage 라이선스 필요**
- Standalone RealDWG와 같은 별도 애플리케이션 없음
- 레거시 경량 ActiveX 객체를 제외하고는 Simulate/Manage 필수

---

## 📌 개발 프로세스

### 🏗️ 프로젝트 구조

#### 🏗️ 기본 프로젝트 구조

```
MyNavisworksPlugin/
│
├── Properties/
│   └── AssemblyInfo.cs           # 어셈블리 정보
│
├── Resources/                    # 리소스 파일 (선택적)
│   ├── Icons/
│   └── en-US/                   # 지역화 파일
│       └── MyPlugin.name
│
├── UI/                          # UI 관련 (선택적)
│   ├── MyDockPane.xaml
│   └── MyDockPane.xaml.cs
│
├── Commands/                    # 명령 클래스들
│   ├── Command1.cs
│   └── Command2.cs
│
├── Utilities/                   # 유틸리티 클래스들
│   ├── ModelHelper.cs
│   └── Logger.cs
│
└── MainPlugin.cs                # 메인 플러그인 클래스
```

#### 🛠️ AssemblyInfo.cs 설정 예시

```csharp
using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

// 어셈블리 정보
[assembly: AssemblyTitle("MyNavisworksPlugin")]
[assembly: AssemblyDescription("Custom Navisworks Plugin")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("My Company")]
[assembly: AssemblyProduct("MyNavisworksPlugin")]
[assembly: AssemblyCopyright("Copyright © 2026")]
[assembly: AssemblyTrademark("")]
[assembly: AssemblyCulture("")]

// COM 가시성
[assembly: ComVisible(false)]

// 버전 정보
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]
```

### 🏗️ 기본 클래스 구조

Navisworks API는 세 가지 주요 플러그인 타입을 제공합니다:

#### ▫️ 1. AddInPlugin (기본 플러그인)

가장 기본적인 플러그인 타입으로, 메뉴 항목을 추가합니다.

```csharp
using System;
using Autodesk.Navisworks.Api.Plugins;

namespace MyNavisworksPlugin
{
    /// <summary>
    /// 기본 AddIn 플러그인 클래스
    /// Navisworks 메뉴에 간단한 명령을 추가합니다.
    /// </summary>
    [Plugin(
        "MyBasicPlugin",                          // Plugin ID (고유해야 함)
        "MyCompany",                              // Developer ID
        DisplayName = "My Basic Plugin",          // 메뉴에 표시될 이름
        ToolTip = "This is my basic plugin"       // 툴팁 텍스트
    )]
    public class MyBasicPlugin : AddInPlugin
    {
        /// <summary>
        /// 플러그인이 실행될 때 호출되는 메서드
        /// 사용자가 메뉴에서 플러그인을 클릭하면 실행됩니다.
        /// </summary>
        /// <param name="parameters">실행 매개변수</param>
        /// <returns>실행 성공 여부 (0: 성공)</returns>
        public override int Execute(params string[] parameters)
        {
            try
            {
                // 현재 활성 문서 가져오기
                var doc = Autodesk.Navisworks.Api.Application.ActiveDocument;

                // 문서가 열려있는지 확인
                if (doc == null)
                {
                    System.Windows.MessageBox.Show(
                        "No document is currently open.",
                        "Error",
                        System.Windows.MessageBoxButton.OK,
                        System.Windows.MessageBoxImage.Error
                    );
                    return 1; // 실패
                }

                // 플러그인 로직 구현
                System.Windows.MessageBox.Show(
                    "Plugin executed successfully!",
                    "Success",
                    System.Windows.MessageBoxButton.OK,
                    System.Windows.MessageBoxImage.Information
                );

                return 0; // 성공
            }
            catch (Exception ex)
            {
                // 에러 처리
                System.Windows.MessageBox.Show(
                    $"Error: {ex.Message}",
                    "Error",
                    System.Windows.MessageBoxButton.OK,
                    System.Windows.MessageBoxImage.Error
                );
                return 1; // 실패
            }
        }
    }
}
```

#### ▫️ 2. DockPanePlugin (도킹 패널)

Navisworks GUI에 커스텀 도킹 패널을 추가합니다.

```csharp
using System;
using System.Windows.Controls;
using Autodesk.Navisworks.Api.Plugins;

namespace MyNavisworksPlugin
{
    /// <summary>
    /// 도킹 패널 플러그인
    /// Navisworks 인터페이스에 도킹 가능한 커스텀 패널을 추가합니다.
    /// </summary>
    [Plugin(
        "MyDockPanePlugin",                       // Plugin ID
        "MyCompany",                              // Developer ID
        DisplayName = "My Dock Pane",             // 패널 제목
        ToolTip = "Custom dockable panel"         // 툴팁
    )]
    [DockPanePlugin(
        800,                                      // 기본 너비 (픽셀)
        600                                       // 기본 높이 (픽셀)
    )]
    public class MyDockPanePlugin : DockPanePlugin
    {
        /// <summary>
        /// 도킹 패널의 WPF 컨트롤을 생성하고 반환합니다.
        /// 이 메서드는 패널이 처음 표시될 때 한 번 호출됩니다.
        /// </summary>
        /// <returns>WPF UserControl 또는 다른 WPF 컨트롤</returns>
        public override System.Windows.Controls.Control CreateControlPane()
        {
            // WPF UserControl 또는 간단한 컨트롤 반환
            var stackPanel = new StackPanel();

            // 라벨 추가
            var label = new Label
            {
                Content = "This is a custom dock pane",
                FontSize = 14,
                Margin = new System.Windows.Thickness(10)
            };
            stackPanel.Children.Add(label);

            // 버튼 추가
            var button = new Button
            {
                Content = "Click Me",
                Margin = new System.Windows.Thickness(10),
                Padding = new System.Windows.Thickness(10, 5, 10, 5)
            };
            button.Click += Button_Click;
            stackPanel.Children.Add(button);

            return stackPanel;
        }

        /// <summary>
        /// 버튼 클릭 이벤트 핸들러
        /// </summary>
        private void Button_Click(object sender, System.Windows.RoutedEventArgs e)
        {
            // 버튼 클릭 시 실행할 로직
            System.Windows.MessageBox.Show(
                "Button clicked from dock pane!",
                "Info",
                System.Windows.MessageBoxButton.OK,
                System.Windows.MessageBoxImage.Information
            );
        }

        /// <summary>
        /// 도킹 패널이 삭제될 때 호출됩니다.
        /// 리소스 정리에 사용합니다.
        /// </summary>
        public override void DestroyControlPane(Control pane)
        {
            // 이벤트 핸들러 해제 등 정리 작업
            if (pane is StackPanel stackPanel)
            {
                foreach (var child in stackPanel.Children)
                {
                    if (child is Button btn)
                    {
                        btn.Click -= Button_Click;
                    }
                }
            }

            base.DestroyControlPane(pane);
        }
    }
}
```

#### ▫️ 3. CommandHandlerPlugin (커스텀 리본)

Navisworks 2012부터 도입된 타입으로, 리본에 커스텀 탭과 여러 명령을 추가할 수 있습니다.

**XAML 리본 레이아웃 (RibbonLayout.xaml):**

```xml
<?xml version="1.0" encoding="utf-8"?>
<RibbonDefinition xmlns="clr-namespace:Autodesk.Windows;assembly=AdWindows">
  <!-- 리본 탭 정의 -->
  <RibbonTab
    Id="MyCustomTab"
    Title="My Tools"
    UID="MyCompany_MyCustomTab">

    <!-- 리본 패널 -->
    <RibbonPanel
      Id="MyPanel1"
      Title="Basic Commands"
      UID="MyCompany_MyPanel1">

      <!-- 버튼들 -->
      <RibbonButton
        Id="Command1"
        Text="Command 1"
        ToolTip="Execute Command 1"
        Image="pack://application:,,,/MyNavisworksPlugin;component/Resources/Icons/cmd1.png"
        LargeImage="pack://application:,,,/MyNavisworksPlugin;component/Resources/Icons/cmd1_large.png" />

      <RibbonButton
        Id="Command2"
        Text="Command 2"
        ToolTip="Execute Command 2"
        Image="pack://application:,,,/MyNavisworksPlugin;component/Resources/Icons/cmd2.png"
        LargeImage="pack://application:,,,/MyNavisworksPlugin;component/Resources/Icons/cmd2_large.png" />

    </RibbonPanel>
  </RibbonTab>
</RibbonDefinition>
```

**C# CommandHandlerPlugin 클래스:**

```csharp
using System;
using System.Windows.Forms;
using Autodesk.Navisworks.Api.Plugins;

namespace MyNavisworksPlugin
{
    /// <summary>
    /// 커스텀 리본 탭과 명령을 추가하는 플러그인
    /// 여러 개의 명령을 하나의 플러그인에서 관리할 수 있습니다.
    /// </summary>
    [Plugin(
        "MyCommandHandler",                       // Plugin ID
        "MyCompany",                              // Developer ID
        DisplayName = "My Command Handler",       // 표시 이름
        ToolTip = "Custom ribbon commands"        // 툴팁
    )]
    [RibbonLayout("RibbonLayout.xaml")]           // XAML 파일 경로
    [RibbonTab("MyCustomTab")]                    // XAML에서 정의한 탭 ID
    [Command("Command1", DisplayName = "Command 1", ToolTip = "First command")]
    [Command("Command2", DisplayName = "Command 2", ToolTip = "Second command")]
    public class MyCommandHandler : CommandHandlerPlugin
    {
        /// <summary>
        /// 리본 버튼이 클릭되었을 때 호출됩니다.
        /// </summary>
        /// <param name="commandId">클릭된 버튼의 Command ID</param>
        /// <param name="parameters">명령 매개변수</param>
        /// <returns>실행 성공 여부 (0: 성공)</returns>
        public override int ExecuteCommand(string commandId, params string[] parameters)
        {
            try
            {
                // 현재 활성 문서 가져오기
                var doc = Autodesk.Navisworks.Api.Application.ActiveDocument;

                // commandId에 따라 다른 동작 수행
                switch (commandId)
                {
                    case "Command1":
                        ExecuteCommand1(doc);
                        break;

                    case "Command2":
                        ExecuteCommand2(doc);
                        break;

                    default:
                        MessageBox.Show(
                            $"Unknown command: {commandId}",
                            "Error",
                            MessageBoxButtons.OK,
                            MessageBoxIcon.Error
                        );
                        return 1; // 실패
                }

                return 0; // 성공
            }
            catch (Exception ex)
            {
                MessageBox.Show(
                    $"Error in {commandId}: {ex.Message}",
                    "Error",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error
                );
                return 1; // 실패
            }
        }

        /// <summary>
        /// Command1 실행 로직
        /// </summary>
        private void ExecuteCommand1(Autodesk.Navisworks.Api.Document doc)
        {
            // Command1 로직 구현
            MessageBox.Show(
                "Command 1 executed!",
                "Info",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information
            );
        }

        /// <summary>
        /// Command2 실행 로직
        /// </summary>
        private void ExecuteCommand2(Autodesk.Navisworks.Api.Document doc)
        {
            // Command2 로직 구현
            MessageBox.Show(
                "Command 2 executed!",
                "Info",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information
            );
        }
    }
}
```

**XAML 파일 위치:**

XAML 파일과 지역화 파일은 플러그인 DLL의 하위 폴더에 위치해야 합니다:

```
Plugins/
└── MyNavisworksPlugin/
    ├── MyNavisworksPlugin.dll
    ├── RibbonLayout.xaml         # 리본 레이아웃
    └── en-US/                    # 지역화 (선택적)
        └── MyPlugin.name         # 표시 이름 및 툴팁
```

### 🛠️ API 사용법

#### ▫️ 주요 네임스페이스

```csharp
using Autodesk.Navisworks.Api;                    // 핵심 API
using Autodesk.Navisworks.Api.Plugins;            // 플러그인 관련
using Autodesk.Navisworks.Api.DocumentParts;      // 문서 부분 (Selection, CurrentSelection 등)
using Autodesk.Navisworks.Api.Interop.ComApi;     // COM API (고급 기능)
```

#### ▫️ 문서(Document) 액세스

```csharp
// 활성 문서 가져오기
var doc = Autodesk.Navisworks.Api.Application.ActiveDocument;

// 문서가 열려있는지 확인
if (doc == null)
{
    // 문서가 없음
    return;
}

// 파일 경로
var filePath = doc.FileName;
Console.WriteLine($"DOCUMENT_PATH:{filePath}");

// 문서의 모델 루트
var models = doc.Models;
Console.WriteLine($"MODEL_COUNT:{models.Count}");
```

#### ▫️ ModelItem 탐색

```csharp
// 모든 ModelItem을 재귀적으로 탐색하는 헬퍼 메서드
private void TraverseModelItems(ModelItem item, Action<ModelItem> action)
{
    // 현재 아이템에 대해 액션 수행
    action(item);

    // 자식 아이템들을 재귀적으로 탐색
    foreach (var child in item.Children)
    {
        TraverseModelItems(child, action);
    }
}

// 사용 예시
var rootItems = doc.Models.RootItems;
foreach (var rootItem in rootItems)
{
    TraverseModelItems(rootItem, (item) =>
    {
        // 각 ModelItem에 대한 처리
        var displayName = item.DisplayName;
        Console.WriteLine($"ITEM:{displayName}");
    });
}
```

#### 💻 검색(Search) API

```csharp
// 검색 조건 생성: 이름에 "Wall"이 포함된 모든 아이템 찾기
var searchCondition = SearchCondition.HasPropertyByDisplayName(
    "Item",         // PropertyCategory 이름
    "Name"          // Property 이름
).Contains("Wall"); // 조건

// 검색 실행
var search = new Search();
search.Selection.SelectAll();  // 모든 아이템을 대상으로 검색
search.SearchConditions.Add(searchCondition);

// 검색 결과
var results = search.FindAll(doc, false); // false = 첫 번째 발견 시 중단하지 않음
Console.WriteLine($"SEARCH_RESULTS:{results.Count}");

// 결과 처리
foreach (var item in results)
{
    Console.WriteLine($"FOUND:{item.DisplayName}");
}
```

#### ▫️ 속성(Properties) 읽기

```csharp
// ModelItem의 모든 속성 읽기
private void ReadProperties(ModelItem item)
{
    // 모든 PropertyCategory 순회
    foreach (var category in item.PropertyCategories)
    {
        var categoryName = category.DisplayName;
        Console.WriteLine($"CATEGORY:{categoryName}");

        // 카테고리 내의 모든 속성 순회
        foreach (var property in category.Properties)
        {
            var propertyName = property.DisplayName;
            var propertyValue = property.Value.ToDisplayString();

            Console.WriteLine($"PROPERTY:{propertyName}={propertyValue}");
        }
    }
}

// 특정 속성 찾기
private string GetPropertyValue(
    ModelItem item,
    string categoryName,
    string propertyName)
{
    // 카테고리 찾기
    var category = item.PropertyCategories.FindCategoryByDisplayName(categoryName);
    if (category == null)
    {
        Console.WriteLine($"CATEGORY_NOT_FOUND category:{categoryName}");
        return null;
    }

    // 속성 찾기
    var property = category.Properties.FindPropertyByDisplayName(propertyName);
    if (property == null)
    {
        Console.WriteLine($"PROPERTY_NOT_FOUND property:{propertyName}");
        return null;
    }

    // 값 반환
    return property.Value.ToDisplayString();
}
```

#### ▫️ 선택(Selection) 조작

```csharp
// 현재 선택된 아이템 가져오기
var selection = doc.CurrentSelection.SelectedItems;
Console.WriteLine($"SELECTED_COUNT:{selection.Count}");

// 선택된 아이템 순회
foreach (var item in selection)
{
    Console.WriteLine($"SELECTED_ITEM:{item.DisplayName}");
}

// 프로그래밍 방식으로 선택 설정
var itemsToSelect = new ModelItemCollection();
// ... itemsToSelect에 아이템 추가 ...
doc.CurrentSelection.CopyFrom(itemsToSelect);
```

#### ▫️ 뷰포인트(Viewpoint) 조작

```csharp
// 현재 뷰포인트 가져오기
var currentViewpoint = doc.CurrentViewpoint.CreateCopy();

// 카메라 위치 가져오기
var position = currentViewpoint.Position;
Console.WriteLine($"CAMERA_POSITION:X={position.X}, Y={position.Y}, Z={position.Z}");

// 카메라 회전 가져오기 (Quaternion)
var rotation = currentViewpoint.Rotation;

// 회전을 Axis-Angle 형식으로 변환
var axisAngle = rotation.ToAxisAndAngle();
var axis = axisAngle.Axis;
var angle = axisAngle.Angle; // 라디안

Console.WriteLine($"CAMERA_ROTATION:Axis=({axis.X},{axis.Y},{axis.Z}), Angle={angle}");

// 새로운 뷰포인트 생성 및 적용
var newViewpoint = new Viewpoint();
newViewpoint.Position = new Point3D(10, 20, 30);
newViewpoint.FocalDistance = 100;
doc.CurrentViewpoint.CopyFrom(newViewpoint);
```

#### ▫️ 지오메트리(Geometry) 접근

**주의:** 지오메트리 추출은 성능 이슈가 있으며, 일부 기능은 COM API를 사용해야 합니다.

```csharp
// ModelItem의 지오메트리 확인
if (item.HasGeometry)
{
    var geometry = item.Geometry;

    // 바운딩 박스 (BoundingBox)
    var boundingBox = geometry.BoundingBox();
    var min = boundingBox.Min;
    var max = boundingBox.Max;

    Console.WriteLine($"BOUNDING_BOX:Min=({min.X},{min.Y},{min.Z}), Max=({max.X},{max.Y},{max.Z})");

    // Transform 정보
    var activeTransform = geometry.ActiveTransform;
    Console.WriteLine($"ACTIVE_TRANSFORM:{activeTransform}");
}
```

---

## 📌 배포

### 🔹 빌드 및 패키징

#### 🛠️ 빌드 설정

**프로젝트 속성 > Build:**

- **Configuration**: Release
- **Platform target**: x64
- **Optimize code**: 체크 (성능 최적화)

**Post-build Event 설정:**

플러그인 DLL을 자동으로 Navisworks Plugins 폴더로 복사:

```batch
REM DLL과 의존성 파일을 Navisworks Plugins 폴더로 복사
xcopy /Y "$(TargetDir)*.*" "C:\Program Files\Autodesk\Navisworks Manage 2024\Plugins\$(TargetName)\"

REM XAML 파일이 있는 경우
xcopy /Y "$(ProjectDir)RibbonLayout.xaml" "C:\Program Files\Autodesk\Navisworks Manage 2024\Plugins\$(TargetName)\"
```

#### ▫️ 의존성 처리

**Bundle 형식 (Navisworks 2015+):**

Navisworks 2015부터는 Bundle 형식을 사용하여 의존 DLL을 플러그인 DLL의 상대 경로에 위치시킬 수 있습니다.

```
Plugins/
└── MyNavisworksPlugin/
    ├── MyNavisworksPlugin.dll      # 메인 플러그인
    ├── Dependencies/               # 의존성 폴더
    │   ├── Newtonsoft.Json.dll
    │   └── Other.dll
    └── RibbonLayout.xaml           # XAML (필요시)
```

**Dependencies 폴더:**

또는 Navisworks 설치 폴더의 `dependencies` 폴더 사용:

```
C:\Program Files\Autodesk\Navisworks Manage 2024\dependencies\
```

### 🛠️ 설치 방법

#### 🛠️ 수동 설치 (Side Loading)

가장 간단한 방법으로, 플러그인 DLL을 직접 Plugins 폴더에 복사합니다.

**단계:**

1. **폴더 구조 생성:**
   ```
   C:\Program Files\Autodesk\Navisworks Manage 2024\Plugins\MyNavisworksPlugin\
   ```

   **중요:** 폴더 이름은 DLL 파일명과 동일해야 합니다 (확장자 제외).

2. **DLL 복사:**
   ```
   MyNavisworksPlugin.dll
   ```

3. **추가 파일 복사 (필요시):**
   - XAML 파일
   - 지역화 파일 (en-US 폴더 등)
   - 의존성 DLL

4. **Navisworks 재시작**

#### ▫️ 사용자 AppData 경로 (Navisworks 2014+)

관리자 권한이 필요 없는 설치 경로:

```
%APPDATA%\Autodesk\Navisworks Manage 2024\Plugins\MyNavisworksPlugin\
```

예:
```
C:\Users\YourUsername\AppData\Roaming\Autodesk\Navisworks Manage 2024\Plugins\MyNavisworksPlugin\
```

#### 🛠️ 설치 프로그램 사용

배포 규모가 큰 경우 설치 프로그램(Installer)을 만들어 배포할 수 있습니다.

**도구 옵션:**

- **WiX Toolset**: XML 기반 Windows Installer 생성
- **Inno Setup**: 스크립트 기반 설치 프로그램
- **NSIS (Nullsoft Scriptable Install System)**: 가벼운 설치 프로그램

**설치 프로그램이 수행할 작업:**

1. Navisworks 설치 여부 및 버전 확인
2. 적절한 Plugins 폴더에 파일 복사
3. 레지스트리 설정 (필요시)
4. 바탕화면 문서 또는 가이드 추가 (선택적)

### 🗂️ Manifest 파일 작성

**중요:** Navisworks는 Revit과 달리 별도의 manifest XAML 파일을 사용하지 않습니다. 대신 `PluginAttribute`를 코드에서 직접 사용합니다.

#### ▫️ PluginAttribute 속성

```csharp
[Plugin(
    "UniquePluginID",                     // 고유 ID (필수)
    "DeveloperID",                        // 개발자 ID (필수)
    DisplayName = "Display Name",         // GUI 표시 이름
    ToolTip = "Tooltip text"              // 툴팁
)]
```

#### ▫️ DockPanePluginAttribute 속성

```csharp
[DockPanePlugin(
    800,                                  // 기본 너비
    600,                                  // 기본 높이
    MinimumWidth = 400,                   // 최소 너비 (선택적)
    MinimumHeight = 300                   // 최소 높이 (선택적)
)]
```

#### ▫️ CommandHandlerPlugin 속성

```csharp
[RibbonLayout("RibbonLayout.xaml")]       // XAML 파일 경로
[RibbonTab("MyCustomTab")]                // XAML의 탭 ID
[Command("Command1",
    DisplayName = "Command 1",
    ToolTip = "Command 1 tooltip",
    Icon = "icon16.png",                  // 16x16 아이콘
    LargeIcon = "icon32.png")]            // 32x32 아이콘
```

#### 🗂️ 지역화 (.name 파일)

XAML 및 .name 파일을 사용하여 다국어 지원:

**폴더 구조:**
```
Plugins/MyNavisworksPlugin/
├── MyNavisworksPlugin.dll
├── en-US/
│   └── MyPlugin.name
└── zh-CN/
    └── MyPlugin.name
```

**MyPlugin.name 예시 (텍스트 파일):**
```
Command1.DisplayName=Command 1
Command1.ToolTip=This is command 1
Command2.DisplayName=Command 2
Command2.ToolTip=This is command 2
```

**로드 우선순위:**

1. XAML 파일의 문자열 확인
2. 지역화 폴더의 .name 파일 확인
3. 플러그인 클래스의 Attribute에서 정의한 이름 사용

---

## 📌 테스트

### 🔹 디버깅 방법

#### 🛠️ Visual Studio 디버깅 설정

**프로젝트 속성 > Debug 탭:**

1. **Start Action** 섹션에서:
   - **Start external program** 선택
   - 경로 입력:
     ```
     C:\Program Files\Autodesk\Navisworks Manage 2024\Roamer.exe
     ```

2. **Command line arguments** (선택적):
   - 특정 파일을 열면서 시작하려면:
     ```
     "C:\Path\To\YourModel.nwd"
     ```

3. **Working directory** (선택적):
   - 작업 디렉토리 설정

#### ▫️ 디버깅 실행

1. Visual Studio에서 **F5** 또는 **Debug > Start Debugging** 클릭
2. Navisworks가 실행됩니다.
3. 플러그인이 자동으로 로드됩니다.
4. 중단점(Breakpoint)을 설정하여 코드를 단계별로 실행할 수 있습니다.

#### ▫️ Visual Studio Express 사용자

Visual Studio Express는 GUI에서 "Start external program" 설정을 지원하지 않습니다. 대안:

**.csproj 파일 직접 수정:**

```xml
<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|x64' ">
  <StartAction>Program</StartAction>
  <StartProgram>C:\Program Files\Autodesk\Navisworks Manage 2024\Roamer.exe</StartProgram>
</PropertyGroup>
```

#### ▫️ NavisAddinManager 사용

**NavisAddinManager**는 Navisworks를 재시작하지 않고 플러그인을 다시 로드할 수 있는 도구입니다.

**GitHub:**
- https://github.com/chuongmep/NavisAddinManager

**장점:**

- Navisworks 재시작 없이 개발 가능
- 빠른 반복 개발
- 디버깅 효율성 향상

### 🔹 테스트 전략

#### ▫️ 1. 단위 테스트 (Unit Testing)

비즈니스 로직을 별도 클래스로 분리하여 단위 테스트 가능하게 만듭니다.

**예시:**

```csharp
// Utilities/GeometryHelper.cs
public class GeometryHelper
{
    /// <summary>
    /// 두 점 사이의 거리 계산
    /// </summary>
    public static double Distance(Point3D p1, Point3D p2)
    {
        var dx = p2.X - p1.X;
        var dy = p2.Y - p1.Y;
        var dz = p2.Z - p1.Z;
        return Math.Sqrt(dx * dx + dy * dy + dz * dz);
    }
}

// Tests/GeometryHelperTests.cs (별도 테스트 프로젝트)
[TestClass]
public class GeometryHelperTests
{
    [TestMethod]
    public void Distance_TwoPoints_ReturnsCorrectDistance()
    {
        var p1 = new Point3D(0, 0, 0);
        var p2 = new Point3D(3, 4, 0);

        var distance = GeometryHelper.Distance(p1, p2);

        Assert.AreEqual(5.0, distance, 0.001);
    }
}
```

#### ▫️ 2. 통합 테스트 (Integration Testing)

실제 Navisworks 모델을 사용하여 플러그인 전체 기능을 테스트합니다.

**테스트 시나리오 예시:**

1. **파일 로드 테스트**
   - 다양한 형식의 파일 (NWD, NWC, NWF) 로드
   - 대용량 파일 처리
   - 손상된 파일 처리

2. **ModelItem 탐색 테스트**
   - 빈 모델
   - 단일 객체 모델
   - 복잡한 계층 구조 모델

3. **속성 읽기 테스트**
   - 표준 속성
   - 커스텀 속성
   - 누락된 속성

4. **성능 테스트**
   - 수천 개의 객체 처리 시간 측정
   - 메모리 사용량 모니터링

#### ▫️ 3. 사용자 수용 테스트 (UAT)

실제 사용자 환경에서 플러그인을 테스트합니다.

**체크리스트:**

- [ ] 플러그인이 예상대로 로드되는가?
- [ ] UI 요소가 올바르게 표시되는가?
- [ ] 모든 명령이 작동하는가?
- [ ] 에러 메시지가 명확한가?
- [ ] 성능이 허용 가능한 수준인가?
- [ ] 다른 플러그인과 충돌하지 않는가?

#### ▫️ 4. 로깅 및 모니터링

**로깅 구현:**

```csharp
using System;
using System.IO;

namespace MyNavisworksPlugin.Utilities
{
    /// <summary>
    /// 간단한 파일 로거
    /// </summary>
    public static class Logger
    {
        private static readonly string _logFilePath;

        static Logger()
        {
            // 로그 파일 경로 설정
            var appData = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
            var logFolder = Path.Combine(appData, "MyNavisworksPlugin");

            // 폴더 생성 (없으면)
            if (!Directory.Exists(logFolder))
            {
                Directory.CreateDirectory(logFolder);
            }

            // 로그 파일명 (날짜 포함)
            var fileName = $"log_{DateTime.Now:yyyyMMdd}.txt";
            _logFilePath = Path.Combine(logFolder, fileName);
        }

        /// <summary>
        /// 디버그 로그 기록
        /// </summary>
        public static void Log(string message)
        {
            try
            {
                var timestamp = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss.fff");
                var logEntry = $"[{timestamp}] {message}";

                File.AppendAllText(_logFilePath, logEntry + Environment.NewLine);
            }
            catch (Exception ex)
            {
                // 로깅 실패 시 조용히 무시 (크래시 방지)
                System.Diagnostics.Debug.WriteLine($"LOGGING_FAILED:{ex.Message}");
            }
        }

        /// <summary>
        /// 에러 로그 기록
        /// </summary>
        public static void LogError(string message, Exception ex = null)
        {
            var errorMessage = $"ERROR:{message}";
            if (ex != null)
            {
                errorMessage += $" Exception:{ex.Message}, StackTrace:{ex.StackTrace}";
            }

            Log(errorMessage);
        }
    }
}
```

**사용 예시:**

```csharp
try
{
    Logger.Log($"PLUGIN_STARTED");

    // 플러그인 로직
    var doc = Autodesk.Navisworks.Api.Application.ActiveDocument;
    Logger.Log($"DOCUMENT_LOADED:Path={doc.FileName}");

    // ...
}
catch (Exception ex)
{
    Logger.LogError("PLUGIN_EXECUTION_FAILED", ex);
    throw; // 크래시로 명확한 디버깅
}
```

#### ▫️ 5. 자동화된 테스트

**COM API를 사용한 자동화:**

```csharp
// 외부 애플리케이션에서 Navisworks 제어
var navisworks = new Autodesk.Navisworks.Api.Automation.NavisworksApplication();
navisworks.OpenFile(@"C:\Path\To\Model.nwd");

// 플러그인 실행
// ... 테스트 로직 ...

navisworks.Dispose();
```

---

## 🧪 예시 코드

### 🗂️ 현재 로딩된 nwc, nwd 파일에서 모든 오브젝트의 정보 추출

이 예시는 현재 열린 Navisworks 문서에서 모든 ModelItem을 순회하며 다음 정보를 추출합니다:

- World Position (월드 좌표)
- Local Position (로컬 좌표)
- Rotation (회전)
- Mesh 정보
- 속성 (Properties)

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Text;
using System.Windows.Forms;
using Autodesk.Navisworks.Api;
using Autodesk.Navisworks.Api.Plugins;
using Autodesk.Navisworks.Api.Interop.ComApi;

namespace NavisworksDataExtractor
{
    /// <summary>
    /// Navisworks 모델에서 모든 오브젝트의 상세 정보를 추출하는 플러그인
    /// </summary>
    [Plugin(
        "DataExtractorPlugin",
        "MyCompany",
        DisplayName = "Extract Object Data",
        ToolTip = "Extract position, rotation, mesh, and properties from all objects"
    )]
    public class DataExtractorPlugin : AddInPlugin
    {
        /// <summary>
        /// 플러그인 실행
        /// </summary>
        public override int Execute(params string[] parameters)
        {
            try
            {
                // 현재 활성 문서 가져오기
                var doc = Autodesk.Navisworks.Api.Application.ActiveDocument;

                // 문서 확인
                if (doc == null)
                {
                    MessageBox.Show(
                        "No document is currently open.",
                        "Error",
                        MessageBoxButtons.OK,
                        MessageBoxIcon.Error
                    );
                    return 1;
                }

                Logger.Log($"EXTRACTION_STARTED:File={doc.FileName}");

                // 데이터 추출
                var objectDataList = ExtractAllObjectData(doc);

                Logger.Log($"EXTRACTION_COMPLETED:Count={objectDataList.Count}");

                // 결과를 CSV로 저장
                SaveToCSV(objectDataList);

                MessageBox.Show(
                    $"Extracted data from {objectDataList.Count} objects.\nSaved to CSV file.",
                    "Success",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Information
                );

                return 0; // 성공
            }
            catch (Exception ex)
            {
                Logger.LogError("EXTRACTION_FAILED", ex);
                MessageBox.Show(
                    $"Error: {ex.Message}",
                    "Error",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error
                );
                return 1; // 실패
            }
        }

        /// <summary>
        /// 모든 객체의 데이터를 추출합니다.
        /// </summary>
        private List<ObjectData> ExtractAllObjectData(Document doc)
        {
            var objectDataList = new List<ObjectData>();

            // COM API 초기화 (일부 고급 기능에 필요)
            InwOpState10 comState = ComApiBridge.State;

            // 모든 루트 아이템 순회
            var rootItems = doc.Models.RootItems;
            Logger.Log($"ROOT_ITEMS_COUNT:{rootItems.Count}");

            foreach (var rootItem in rootItems)
            {
                // 재귀적으로 모든 자식 아이템 추출
                TraverseAndExtract(rootItem, comState, objectDataList);
            }

            return objectDataList;
        }

        /// <summary>
        /// ModelItem을 재귀적으로 순회하며 데이터를 추출합니다.
        /// </summary>
        private void TraverseAndExtract(
            ModelItem item,
            InwOpState10 comState,
            List<ObjectData> dataList)
        {
            try
            {
                // ObjectData 생성 및 추출
                var objectData = new ObjectData();

                // 기본 정보
                objectData.DisplayName = item.DisplayName ?? "Unnamed";
                objectData.InstanceGuid = item.InstanceGuid.ToString();
                objectData.ClassDisplayName = item.ClassDisplayName ?? "Unknown";

                Logger.Log($"PROCESSING_ITEM:Name={objectData.DisplayName}");

                // 지오메트리가 있는 경우에만 추가 정보 추출
                if (item.HasGeometry)
                {
                    ExtractGeometryData(item, comState, objectData);
                }

                // 속성 추출
                ExtractProperties(item, objectData);

                // 리스트에 추가
                dataList.Add(objectData);
            }
            catch (Exception ex)
            {
                Logger.LogError($"ITEM_EXTRACTION_FAILED:Item={item.DisplayName}", ex);
                // 에러가 발생해도 계속 진행
            }

            // 자식 아이템들을 재귀적으로 처리
            foreach (var child in item.Children)
            {
                TraverseAndExtract(child, comState, dataList);
            }
        }

        /// <summary>
        /// 지오메트리 데이터를 추출합니다 (위치, 회전, 메시).
        /// </summary>
        private void ExtractGeometryData(
            ModelItem item,
            InwOpState10 comState,
            ObjectData objectData)
        {
            try
            {
                var geometry = item.Geometry;

                // 바운딩 박스 (World 좌표계)
                var boundingBox = geometry.BoundingBox();
                var center = new Point3D(
                    (boundingBox.Min.X + boundingBox.Max.X) / 2.0,
                    (boundingBox.Min.Y + boundingBox.Max.Y) / 2.0,
                    (boundingBox.Min.Z + boundingBox.Max.Z) / 2.0
                );

                objectData.WorldPositionX = center.X;
                objectData.WorldPositionY = center.Y;
                objectData.WorldPositionZ = center.Z;

                Logger.Log($"WORLD_POSITION:X={center.X}, Y={center.Y}, Z={center.Z}");

                // Transform 정보
                var activeTransform = geometry.ActiveTransform;

                // Transform을 Matrix로 변환하여 로컬 좌표 추출
                // Transform3D는 4x4 매트릭스를 나타냅니다.
                // Translation (로컬 위치)는 매트릭스의 마지막 행에 있습니다.
                var translation = activeTransform.Translation;
                objectData.LocalPositionX = translation.X;
                objectData.LocalPositionY = translation.Y;
                objectData.LocalPositionZ = translation.Z;

                Logger.Log($"LOCAL_POSITION:X={translation.X}, Y={translation.Y}, Z={translation.Z}");

                // 회전 정보 추출 (Rotation3D를 Euler Angles로 변환)
                // Navisworks는 Quaternion을 사용하므로 Euler Angles로 변환
                var rotation = activeTransform.Rotation;
                var eulerAngles = QuaternionToEulerAngles(rotation);

                objectData.RotationX = eulerAngles.X;
                objectData.RotationY = eulerAngles.Y;
                objectData.RotationZ = eulerAngles.Z;

                Logger.Log($"ROTATION:X={eulerAngles.X}, Y={eulerAngles.Y}, Z={eulerAngles.Z}");

                // 메시 정보 추출 (COM API 사용)
                ExtractMeshData(item, comState, objectData);
            }
            catch (Exception ex)
            {
                Logger.LogError($"GEOMETRY_EXTRACTION_FAILED:Item={objectData.DisplayName}", ex);
            }
        }

        /// <summary>
        /// COM API를 사용하여 메시 데이터를 추출합니다.
        /// </summary>
        private void ExtractMeshData(
            ModelItem item,
            InwOpState10 comState,
            ObjectData objectData)
        {
            try
            {
                // ModelItem을 COM 객체로 변환
                InwOaPath comPath = ComApiBridge.ToInwOaPath(item);

                // Fragment 가져오기 (지오메트리의 실제 데이터)
                var fragments = comState.ObjectFactory.CreateFragmentsColl();
                fragments.AddFragsForPath(comPath);

                // Fragment가 있는 경우
                if (fragments.Count > 0)
                {
                    var fragment = fragments.Item(1); // 첫 번째 Fragment

                    // Vertex 개수 및 Triangle 개수 추출
                    // 주의: GenerateSimplePrimitives()는 성능 이슈가 있을 수 있음
                    var primitives = fragment.GenerateSimplePrimitives(
                        nwEVertexProperty.eNORMAL,  // Normal 포함
                        null
                    );

                    // Primitives 정보 추출
                    var vertexCount = 0;
                    var triangleCount = 0;

                    foreach (InwSimplePrimitive primitive in primitives)
                    {
                        vertexCount += primitive.VerticesCount;
                        triangleCount += primitive.TrianglesCount;
                    }

                    objectData.VertexCount = vertexCount;
                    objectData.TriangleCount = triangleCount;

                    Logger.Log($"MESH_DATA:Vertices={vertexCount}, Triangles={triangleCount}");
                }
            }
            catch (Exception ex)
            {
                Logger.LogError($"MESH_EXTRACTION_FAILED:Item={objectData.DisplayName}", ex);
                // 메시 추출 실패 시에도 계속 진행
            }
        }

        /// <summary>
        /// Quaternion을 Euler Angles(도)로 변환합니다.
        /// </summary>
        private Point3D QuaternionToEulerAngles(Rotation3D rotation)
        {
            // Quaternion 성분 추출
            var axisAngle = rotation.ToAxisAndAngle();
            var axis = axisAngle.Axis;
            var angle = axisAngle.Angle; // 라디안

            // 간단한 변환 (정확한 변환은 더 복잡함)
            // 여기서는 근사치를 사용
            var x = axis.X * angle * (180.0 / Math.PI);
            var y = axis.Y * angle * (180.0 / Math.PI);
            var z = axis.Z * angle * (180.0 / Math.PI);

            return new Point3D(x, y, z);
        }

        /// <summary>
        /// ModelItem의 속성을 추출합니다.
        /// </summary>
        private void ExtractProperties(ModelItem item, ObjectData objectData)
        {
            try
            {
                var propertiesDict = new Dictionary<string, string>();

                // 모든 PropertyCategory 순회
                foreach (var category in item.PropertyCategories)
                {
                    var categoryName = category.DisplayName;

                    // 카테고리 내의 모든 속성 순회
                    foreach (var property in category.Properties)
                    {
                        var propertyName = property.DisplayName;
                        var propertyValue = property.Value.ToDisplayString();

                        // 키 생성: "Category::Property"
                        var key = $"{categoryName}::{propertyName}";
                        propertiesDict[key] = propertyValue;
                    }
                }

                // Dictionary를 JSON 형식 문자열로 변환 (간단하게)
                var sb = new StringBuilder();
                sb.Append("{");
                var first = true;
                foreach (var kvp in propertiesDict)
                {
                    if (!first) sb.Append(", ");
                    sb.Append($"\"{kvp.Key}\":\"{kvp.Value}\"");
                    first = false;
                }
                sb.Append("}");

                objectData.Properties = sb.ToString();

                Logger.Log($"PROPERTIES_EXTRACTED:Count={propertiesDict.Count}");
            }
            catch (Exception ex)
            {
                Logger.LogError($"PROPERTIES_EXTRACTION_FAILED:Item={objectData.DisplayName}", ex);
            }
        }

        /// <summary>
        /// 추출된 데이터를 CSV 파일로 저장합니다.
        /// </summary>
        private void SaveToCSV(List<ObjectData> dataList)
        {
            try
            {
                // 저장 경로 설정
                var appData = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
                var fileName = $"NavisworksExtraction_{DateTime.Now:yyyyMMdd_HHmmss}.csv";
                var filePath = Path.Combine(appData, fileName);

                Logger.Log($"SAVING_CSV:Path={filePath}");

                using (var writer = new StreamWriter(filePath, false, Encoding.UTF8))
                {
                    // CSV 헤더
                    writer.WriteLine("DisplayName,InstanceGuid,ClassDisplayName," +
                        "WorldX,WorldY,WorldZ," +
                        "LocalX,LocalY,LocalZ," +
                        "RotationX,RotationY,RotationZ," +
                        "VertexCount,TriangleCount," +
                        "Properties");

                    // 데이터 행
                    foreach (var data in dataList)
                    {
                        writer.WriteLine(
                            $"\"{data.DisplayName}\"," +
                            $"\"{data.InstanceGuid}\"," +
                            $"\"{data.ClassDisplayName}\"," +
                            $"{data.WorldPositionX}," +
                            $"{data.WorldPositionY}," +
                            $"{data.WorldPositionZ}," +
                            $"{data.LocalPositionX}," +
                            $"{data.LocalPositionY}," +
                            $"{data.LocalPositionZ}," +
                            $"{data.RotationX}," +
                            $"{data.RotationY}," +
                            $"{data.RotationZ}," +
                            $"{data.VertexCount}," +
                            $"{data.TriangleCount}," +
                            $"\"{EscapeCSV(data.Properties)}\""
                        );
                    }
                }

                Logger.Log($"CSV_SAVED_SUCCESSFULLY:Path={filePath}");

                // 파일 열기
                System.Diagnostics.Process.Start(filePath);
            }
            catch (Exception ex)
            {
                Logger.LogError("CSV_SAVE_FAILED", ex);
                throw;
            }
        }

        /// <summary>
        /// CSV 문자열 이스케이프 처리
        /// </summary>
        private string EscapeCSV(string value)
        {
            if (string.IsNullOrEmpty(value))
                return string.Empty;

            // 따옴표를 이중 따옴표로 변환
            return value.Replace("\"", "\"\"");
        }
    }

    /// <summary>
    /// 추출된 객체 데이터를 담는 클래스
    /// </summary>
    public class ObjectData
    {
        // 기본 정보
        public string DisplayName { get; set; }
        public string InstanceGuid { get; set; }
        public string ClassDisplayName { get; set; }

        // World Position
        public double WorldPositionX { get; set; }
        public double WorldPositionY { get; set; }
        public double WorldPositionZ { get; set; }

        // Local Position
        public double LocalPositionX { get; set; }
        public double LocalPositionY { get; set; }
        public double LocalPositionZ { get; set; }

        // Rotation (Euler Angles in degrees)
        public double RotationX { get; set; }
        public double RotationY { get; set; }
        public double RotationZ { get; set; }

        // Mesh 정보
        public int VertexCount { get; set; }
        public int TriangleCount { get; set; }

        // 속성 (JSON 형식 문자열)
        public string Properties { get; set; }
    }

    /// <summary>
    /// 간단한 로거 클래스
    /// </summary>
    public static class Logger
    {
        private static readonly string _logFilePath;

        static Logger()
        {
            var appData = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);
            var logFolder = Path.Combine(appData, "NavisworksDataExtractor");

            if (!Directory.Exists(logFolder))
            {
                Directory.CreateDirectory(logFolder);
            }

            var fileName = $"log_{DateTime.Now:yyyyMMdd}.txt";
            _logFilePath = Path.Combine(logFolder, fileName);
        }

        public static void Log(string message)
        {
            try
            {
                var timestamp = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss.fff");
                var logEntry = $"[{timestamp}] {message}";

                File.AppendAllText(_logFilePath, logEntry + Environment.NewLine);
            }
            catch
            {
                // 로깅 실패 시 조용히 무시
            }
        }

        public static void LogError(string message, Exception ex = null)
        {
            var errorMessage = $"ERROR:{message}";
            if (ex != null)
            {
                errorMessage += $" Exception:{ex.Message}, StackTrace:{ex.StackTrace}";
            }

            Log(errorMessage);
        }
    }
}
```

### 🔹 사용 방법

1. **빌드**: 프로젝트를 Release 모드로 빌드합니다.
2. **배포**: DLL을 Navisworks Plugins 폴더에 복사합니다.
3. **실행**:
   - Navisworks에서 모델 파일(NWC, NWD)을 엽니다.
   - 플러그인 메뉴에서 "Extract Object Data" 선택
   - 추출이 완료되면 CSV 파일이 자동으로 열립니다.
4. **결과**: CSV 파일에는 모든 객체의 위치, 회전, 메시, 속성 정보가 포함됩니다.

### 🔹 주요 기능 설명

#### ⚖️ World Position vs Local Position

- **World Position**: 전체 모델의 글로벌 좌표계에서의 위치
  - 바운딩 박스의 중심점을 사용하여 계산

- **Local Position**: 객체의 로컬 좌표계에서의 위치
  - Transform의 Translation 성분을 사용

#### ▫️ Rotation 추출

- Navisworks는 내부적으로 Quaternion을 사용합니다.
- Euler Angles(X, Y, Z 회전각)로 변환하여 더 이해하기 쉬운 형태로 제공합니다.
- 각도는 도(degree) 단위로 표시됩니다.

#### 🎮 Mesh 정보

- **Vertex Count**: 메시를 구성하는 정점의 개수
- **Triangle Count**: 삼각형의 개수
- COM API의 `GenerateSimplePrimitives()` 메서드를 사용하여 추출
- **주의**: 이 작업은 성능 부하가 있을 수 있습니다.

#### ▫️ Properties

- 모든 PropertyCategory와 Property를 순회하여 수집
- JSON 형식 문자열로 저장 (간단한 구현)
- 실제 프로젝트에서는 Newtonsoft.Json 등의 라이브러리 사용 권장

### 🔹 로그 출력

로그는 다음 위치에 저장됩니다:

```
C:\Users\[Username]\AppData\Roaming\NavisworksDataExtractor\log_yyyyMMdd.txt
```

로그 메시지 형식:

```
[2026-02-03 14:23:45.123] EXTRACTION_STARTED:File=C:\Models\MyModel.nwd
[2026-02-03 14:23:45.456] ROOT_ITEMS_COUNT:5
[2026-02-03 14:23:45.789] PROCESSING_ITEM:Name=Wall-001
[2026-02-03 14:23:45.890] WORLD_POSITION:X=1000.5, Y=2000.3, Z=3000.1
[2026-02-03 14:23:45.991] LOCAL_POSITION:X=0.0, Y=0.0, Z=0.0
[2026-02-03 14:23:46.092] ROTATION:X=0.0, Y=90.0, Z=0.0
[2026-02-03 14:23:46.193] MESH_DATA:Vertices=240, Triangles=120
[2026-02-03 14:23:46.294] PROPERTIES_EXTRACTED:Count=15
[2026-02-03 14:23:50.000] EXTRACTION_COMPLETED:Count=1250
[2026-02-03 14:23:50.100] SAVING_CSV:Path=C:\Users\...\NavisworksExtraction_20260203_142350.csv
[2026-02-03 14:23:50.200] CSV_SAVED_SUCCESSFULLY:Path=C:\Users\...\NavisworksExtraction_20260203_142350.csv
```

---

## 📌 추가 리소스

### 🛠️ 공식 문서 및 가이드

1. **Autodesk Platform Services (APS)**
   - https://aps.autodesk.com/developer/overview/navisworks
   - 공식 개발자 포털, API 문서 및 샘플 코드

2. **ApiDocs.co - Navisworks Developer Guide**
   - https://apidocs.co/apps/navisworks/2018/87317537-2911-4c08-b492-6496c82b3ed0.htm
   - API 참조 문서 온라인 버전

3. **Autodesk University - Navisworks API**
   - https://www.autodesk.com/autodesk-university/class/Make-NavisWORK-You-Intro-Navisworks-API-2018
   - 무료 온라인 강의 및 클래스 자료

### 🔹 커뮤니티 및 튜토리얼

4. **TwentyTwo Blog - Navisworks Add-Ins Development**
   - https://twentytwo.space/navisworks-add-ins-development/
   - 단계별 튜토리얼 및 실용적인 예제

5. **AEC DevBlog - Navisworks**
   - https://adndevblog.typepad.com/aec/navisworks/
   - Autodesk 공식 개발 블로그, SDK 업데이트 및 기술 문서

6. **Navisworks API 포럼**
   - https://forums.autodesk.com/t5/navisworks-api-forum/bd-p/80
   - 개발자 커뮤니티 Q&A

### 🔹 GitHub 리포지토리

7. **NavisAddinManager**
   - https://github.com/chuongmep/NavisAddinManager
   - Navisworks 재시작 없이 플러그인 업데이트

8. **navisworks-api Topic**
   - https://github.com/topics/navisworks-api
   - GitHub의 Navisworks API 관련 프로젝트 모음

### 🔹 NuGet 패키지

9. **Navisworks-2020-x64-API**
   - https://www.nuget.org/packages/Navisworks-2020-x64-API

10. **Chuongmep.Navis.Api.Autodesk.Navisworks.Api**
    - https://www.nuget.org/packages/Chuongmep.Navis.Api.Autodesk.Navisworks.Api/2025.0.0
    - 커뮤니티 유지보수 NuGet 패키지

### 🔹 학습 자료

11. **Udemy - Basic PlugIn Development with Navisworks API C#**
    - https://www.udemy.com/course/nwapicsh/
    - 유료 온라인 강의

12. **Introduction to the Navisworks .NET API (PDF)**
    - http://www.fig-tech.com/wp-content/uploads/2015/11/3.-DotNETAPIIntroduction.pdf
    - 무료 PDF 가이드

### 🔹 디버깅 및 개발 도구

13. **NavisLookup**
    - https://github.com/chuongmep/NavisLookup
    - Navisworks 데이터베이스 탐색 도구

14. **Navisworks .NET Addin Wizard Pro**
    - https://marketplace.visualstudio.com/items?itemName=VisualSmarterPro.NavisworksNETAddinWizardPro
    - Visual Studio 플러그인 템플릿

---

## 📌 결론

Navisworks Addin 개발은 AEC 산업의 BIM 워크플로우를 크게 향상시킬 수 있는 강력한 도구입니다. 이 가이드에서 다룬 내용을 요약하면:

### 🔹 핵심 포인트

1. **개발 환경**: Visual Studio 2022 + .NET Framework 4.8 + Navisworks Simulate 라이선스
2. **플러그인 타입**: AddInPlugin, DockPanePlugin, CommandHandlerPlugin
3. **주요 API**: .NET API (주), COM API (고급 기능)
4. **배포**: Plugins 폴더에 DLL 복사 (Bundle 형식 권장)
5. **테스트**: Visual Studio 디버깅 + NavisAddinManager

### ⚠️ 개발 시 주의사항

- **Copy Local = False**: Navisworks API DLL 참조 시 필수 설정
- **성능**: 지오메트리 추출(`GenerateSimplePrimitives()`)은 성능 부하가 있음
- **COM API**: .NET API에 없는 기능은 COM API 사용
- **에러 처리**: 복잡한 처리보다 크래시로 명확한 디버깅 우선
- **로깅**: 상세한 로그로 문제 추적 용이성 확보

### 🔹 다음 단계

1. **간단한 플러그인으로 시작**: AddInPlugin으로 "Hello World" 수준의 플러그인 작성
2. **API 탐색**: SDK의 Developer's Guide와 Reference Guide 숙독
3. **샘플 코드 실행**: 공식 샘플 코드를 실행하고 수정하며 학습
4. **커뮤니티 참여**: 포럼과 GitHub에서 다른 개발자들과 교류
5. **실무 프로젝트**: 실제 업무 문제를 해결하는 플러그인 개발

Navisworks API는 지속적으로 발전하고 있으며, 새로운 버전마다 개선된 기능이 추가됩니다. 최신 SDK 문서를 참고하고, 커뮤니티의 지식을 활용하여 효과적인 플러그인을 개발하시기 바랍니다.

---

**문서 작성일**: 2026-02-03
**작성자**: Claude Code
**버전**: 1.0
**대상 독자**: C# 개발자, BIM 소프트웨어 개발자, Navisworks 사용자

---

## 🔗 참고 자료 (Sources)

- [Navisworks API | Autodesk Platform Services (APS)](https://aps.autodesk.com/developer/overview/navisworks)
- [Navisworks Add-Ins and API Tutorials – TwentyTwo](https://twentytwo.space/navisworks-add-ins-development/)
- [Navisworks API : Creating Navisworks Add-Ins – TwentyTwo](https://twentytwo.space/2020/04/08/creating-navisworks-add-ins/)
- [ApiDocs.co · Navisworks · Developer Guide](https://apidocs.co/apps/navisworks/2018/87317537-2911-4c08-b492-6496c82b3ed0.htm)
- [AEC DevBlog: Navisworks](https://adndevblog.typepad.com/aec/navisworks/)
- [GitHub - chuongmep/NavisAddinManager](https://github.com/chuongmep/NavisAddinManager)
- [Make NavisWORK for You: Intro to the Navisworks API | Autodesk University](https://www.autodesk.com/autodesk-university/class/Make-NavisWORK-You-Intro-Navisworks-API-2018)
- [Basic PlugIn Development with Navisworks API C# | Udemy](https://www.udemy.com/course/nwapicsh/)
- [Navisworks API Documentation Forums](https://forums.autodesk.com/t5/navisworks-api-forum/navisworks-api-documentation-2020-or-later/td-p/9841482)
- [Navisworks API : Viewpoint (Part-1) – TwentyTwo](https://twentytwo.space/2020/12/06/navisworks-api-viewpoint-part-1/)
- [Navisworks API : Search ModelItem and Collect Properties – TwentyTwo](https://twentytwo.space/2020/05/28/navisworks-api-search-modelitem-and-collect-properties/)
- [Retrieve geometry from Navisworks model - Forums](https://forums.autodesk.com/t5/navisworks-api-forum/retrieve-geometry-from-navisworks-model/td-p/11372887)
- [Debugging Navisworks Plugins in Visual Studio Express](https://teocomi.com/debugging-navisworks-plugins-in-visual-studio-express/)
- [Side Loading - Manually Installing a Navisworks Plugin | House of BIM](https://www.houseofbim.com/posts/side-loadingmanually-installing-a-navisworks-plugin/)
- [Navisworks API : CommandHandlerPlugin – TwentyTwo](https://twentytwo.space/2022/02/02/navisworks-api-commandhandlerplugin/)
- [NuGet Gallery | Navisworks-2020-x64-API](https://www.nuget.org/packages/Navisworks-2020-x64-API)
- [NuGet Gallery | Chuongmep.Navis.Api.Autodesk.Navisworks.Api](https://www.nuget.org/packages/Chuongmep.Navis.Api.Autodesk.Navisworks.Api)
- [Required Licenses - Navisworks API - Forums](https://forums.autodesk.com/t5/navisworks-forum/required-licenses-navisworks-api/td-p/6508741)
- [Navisworks 2014 API Override Transformation - AEC DevBlog](https://adndevblog.typepad.com/aec/2013/05/navisworks-2014-api-new-feature-override-transformation.html)
- [Convert LCS to WCS with COM api - Forums](https://forums.autodesk.com/t5/navisworks-api-forum/convert-lcs-to-wcs-with-com-api/td-p/4840957)
- [Navisworks API plugin locations - Forums](https://forums.autodesk.com/t5/navisworks-api-forum/navisworks-api-plugin-locations/td-p/5358803)
