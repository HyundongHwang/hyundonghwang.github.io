---
layout: post
title: 260205 unity editorwindow debug tool guide
comments: true
tags:
- unity
- rendering
- editor-tooling
- debugging
- tools
- ai
- research
- guide
---
# 🛠️ 260205 Unity EditorWindow 디버깅 도구 제작 가이드

Unity 에디터 확장을 활용한 커스텀 디버깅 도구 개발 방법을 단계별로 알아봅니다.

---

## 🧭 1. EditorWindow 소개

### 💻 EditorWindow란?

`EditorWindow`는 Unity 에디터 내에서 독립적으로 떠다니거나 탭으로 도킹될 수 있는 커스텀 창을 만드는 기반 클래스입니다.

```
┌─────────────────────────────────────────────────────────┐
│  Unity Editor                                           │
│  ┌──────────┬────────────────────┬──────────────────┐   │
│  │ Hierarchy│     Scene View     │    Inspector     │   │
│  │          │                    │                  │   │
│  │          │                    │                  │   │
│  ├──────────┴────────────────────┴──────────────────┤   │
│  │              Custom EditorWindow                 │   │
│  │         (도킹 또는 독립 창으로 사용 가능)           │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### ✨ 핵심 특징

| 특징 | 설명 |
|------|------|
| **도킹 가능** | 다른 에디터 창처럼 탭으로 도킹 |
| **위치 저장** | 창 위치가 세션 간에 유지됨 |
| **레이아웃 통합** | 커스텀 레이아웃에 저장 가능 |
| **메뉴 통합** | MenuItem으로 메뉴에서 열기 가능 |

---

## 🧪 2. 기본 예제: Hello World EditorWindow

가장 간단한 EditorWindow 구현입니다.

### 🔹 2.1 최소 구현

```csharp
using UnityEditor;
using UnityEngine;

namespace MyProject.Editor
{
    /// <summary>
    /// 가장 기본적인 EditorWindow 예제
    /// </summary>
    public class HelloWorldWindow : EditorWindow
    {
        // 메뉴에서 창 열기
        [MenuItem("Tools/Hello World")]
        public static void ShowWindow()
        {
            // 창 인스턴스 가져오기 (없으면 생성)
            var window = GetWindow<HelloWorldWindow>("Hello World");
            window.minSize = new Vector2(200, 100);
        }

        // GUI 렌더링
        private void OnGUI()
        {
            EditorGUILayout.LabelField("Hello, Unity Editor!", EditorStyles.boldLabel);

            if (GUILayout.Button("Click Me"))
            {
                Debug.Log("Button clicked!");
            }
        }
    }
}
```

### ⚠️ 2.2 파일 배치 주의사항

```
Assets/
├── Scripts/
│   ├── Runtime/          # 게임 코드
│   └── Editor/           # ★ 에디터 코드는 반드시 Editor 폴더에!
│       └── HelloWorldWindow.cs
```

> 💡 **중요**: `Editor` 폴더 내의 스크립트는 빌드에 포함되지 않습니다.

---

## 🧪 3. 중급 예제: 검색 기능이 있는 디버그 도구

게임 오브젝트를 검색하고 정보를 표시하는 도구입니다.

### 🧪 3.1 전체 코드

```csharp
using System.Collections.Generic;
using UnityEditor;
using UnityEngine;

namespace MyProject.Editor
{
    /// <summary>
    /// GameObject 검색 및 디버그 도구
    /// - 이름으로 검색
    /// - 컴포넌트로 필터링
    /// - 상세 정보 표시
    /// </summary>
    public class GameObjectDebugWindow : EditorWindow
    {
        // 검색 관련 필드
        private string _searchName = "";
        private string _componentFilter = "";
        private List<GameObject> _searchResults = new List<GameObject>();

        // 선택된 오브젝트
        private GameObject _selectedObject;

        // 스크롤 위치
        private Vector2 _listScrollPos;
        private Vector2 _detailScrollPos;

        // 창 열기
        [MenuItem("Tools/Debug/GameObject Debug")]
        public static void ShowWindow()
        {
            var window = GetWindow<GameObjectDebugWindow>("GO Debug");
            window.minSize = new Vector2(500, 400);
        }

        private void OnGUI()
        {
            // 상단: 검색 영역
            DrawSearchPanel();

            EditorGUILayout.Space(10);

            // 하단: 결과 목록 + 상세 정보 (가로 분할)
            EditorGUILayout.BeginHorizontal();

            // 왼쪽: 검색 결과 목록
            EditorGUILayout.BeginVertical(GUILayout.Width(200));
            DrawResultList();
            EditorGUILayout.EndVertical();

            // 구분선
            EditorGUILayout.Space(5);

            // 오른쪽: 상세 정보
            EditorGUILayout.BeginVertical();
            DrawDetailPanel();
            EditorGUILayout.EndVertical();

            EditorGUILayout.EndHorizontal();
        }

        /// <summary>
        /// 검색 패널 그리기
        /// </summary>
        private void DrawSearchPanel()
        {
            EditorGUILayout.LabelField("Search", EditorStyles.boldLabel);

            EditorGUILayout.BeginVertical("box");

            // 이름 검색
            EditorGUILayout.BeginHorizontal();
            EditorGUILayout.LabelField("Name:", GUILayout.Width(80));
            _searchName = EditorGUILayout.TextField(_searchName);
            EditorGUILayout.EndHorizontal();

            // 컴포넌트 필터
            EditorGUILayout.BeginHorizontal();
            EditorGUILayout.LabelField("Component:", GUILayout.Width(80));
            _componentFilter = EditorGUILayout.TextField(_componentFilter);
            EditorGUILayout.EndHorizontal();

            EditorGUILayout.Space(5);

            // 검색 버튼
            if (GUILayout.Button("Search"))
            {
                PerformSearch();
            }

            EditorGUILayout.EndVertical();
        }

        /// <summary>
        /// 검색 수행
        /// </summary>
        private void PerformSearch()
        {
            _searchResults.Clear();

            // 씬의 모든 GameObject 검색
            var allObjects = FindObjectsByType<GameObject>(FindObjectsSortMode.None);

            foreach (var obj in allObjects)
            {
                // 이름 필터
                if (!string.IsNullOrEmpty(_searchName))
                {
                    if (!obj.name.ToLower().Contains(_searchName.ToLower()))
                    {
                        continue;
                    }
                }

                // 컴포넌트 필터
                if (!string.IsNullOrEmpty(_componentFilter))
                {
                    bool hasComponent = false;
                    foreach (var comp in obj.GetComponents<Component>())
                    {
                        if (comp != null &&
                            comp.GetType().Name.ToLower().Contains(_componentFilter.ToLower()))
                        {
                            hasComponent = true;
                            break;
                        }
                    }
                    if (!hasComponent)
                    {
                        continue;
                    }
                }

                _searchResults.Add(obj);
            }
        }

        /// <summary>
        /// 검색 결과 목록 그리기
        /// </summary>
        private void DrawResultList()
        {
            EditorGUILayout.LabelField($"Results ({_searchResults.Count})", EditorStyles.boldLabel);

            _listScrollPos = EditorGUILayout.BeginScrollView(_listScrollPos, "box");

            foreach (var obj in _searchResults)
            {
                if (obj == null)
                {
                    continue;
                }

                // 선택된 항목 하이라이트
                var style = obj == _selectedObject
                    ? EditorStyles.boldLabel
                    : EditorStyles.label;

                if (GUILayout.Button(obj.name, style))
                {
                    _selectedObject = obj;
                    // Hierarchy에서도 선택
                    Selection.activeGameObject = obj;
                }
            }

            EditorGUILayout.EndScrollView();
        }

        /// <summary>
        /// 상세 정보 패널 그리기
        /// </summary>
        private void DrawDetailPanel()
        {
            EditorGUILayout.LabelField("Details", EditorStyles.boldLabel);

            _detailScrollPos = EditorGUILayout.BeginScrollView(_detailScrollPos, "box");

            if (_selectedObject == null)
            {
                EditorGUILayout.LabelField("Select an object to view details.");
            }
            else
            {
                // 기본 정보
                EditorGUILayout.LabelField("Basic Info", EditorStyles.boldLabel);
                EditorGUILayout.TextField("Name", _selectedObject.name);
                EditorGUILayout.TextField("Tag", _selectedObject.tag);
                EditorGUILayout.IntField("Layer", _selectedObject.layer);
                EditorGUILayout.Toggle("Active", _selectedObject.activeSelf);

                EditorGUILayout.Space(10);

                // Transform 정보
                EditorGUILayout.LabelField("Transform", EditorStyles.boldLabel);
                var t = _selectedObject.transform;
                EditorGUILayout.Vector3Field("Position", t.position);
                EditorGUILayout.Vector3Field("Rotation", t.eulerAngles);
                EditorGUILayout.Vector3Field("Scale", t.localScale);

                EditorGUILayout.Space(10);

                // 컴포넌트 목록
                EditorGUILayout.LabelField("Components", EditorStyles.boldLabel);
                var components = _selectedObject.GetComponents<Component>();
                foreach (var comp in components)
                {
                    if (comp != null)
                    {
                        EditorGUILayout.LabelField($"  - {comp.GetType().Name}");
                    }
                }
            }

            EditorGUILayout.EndScrollView();
        }
    }
}
```

### 🔹 3.2 주요 패턴 설명

```
┌──────────────────────────────────────────────────────────┐
│  GameObjectDebugWindow                                   │
├──────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐ │
│  │ [Search Panel]                                      │ │
│  │  Name: [___________]  Component: [___________]      │ │
│  │  [Search Button]                                    │ │
│  └─────────────────────────────────────────────────────┘ │
├────────────────────────┬─────────────────────────────────┤
│  [Result List]         │  [Detail Panel]                 │
│  ┌──────────────────┐  │  ┌────────────────────────────┐ │
│  │ ▸ Player         │  │  │ Basic Info                 │ │
│  │   Enemy_01       │  │  │ Name: Player               │ │
│  │   Enemy_02       │  │  │ Tag: Player                │ │
│  │   ...            │  │  │                            │ │
│  └──────────────────┘  │  │ Transform                  │ │
│                        │  │ Position: (0, 1, 0)        │ │
│                        │  │ ...                        │ │
│                        │  └────────────────────────────┘ │
└────────────────────────┴─────────────────────────────────┘
```

**핵심 요소:**

1. **레이아웃 분할**: `BeginHorizontal`/`EndHorizontal`로 좌우 분할
2. **스크롤 뷰**: `BeginScrollView`로 긴 목록 처리
3. **박스 스타일**: `"box"` 파라미터로 영역 구분
4. **선택 상태 관리**: 별도 필드로 선택된 항목 추적

---

## 🧪 4. 고급 예제: SelectionTreeDebugWindow 분석

실제 프로덕션 코드인 `SelectionTreeDebugWindow.cs`를 분석합니다.

### 🏗️ 4.1 전체 구조

```
SelectionTreeDebugWindow (656 lines)
├── 필드 정의 (16-43)
│   ├── 서비스 참조
│   ├── 검색 상태
│   ├── UI 상태 (스크롤, 확장 노드 등)
│   └── 스타일
├── 생명주기 메서드 (45-91)
│   ├── ShowWindow() - 창 열기
│   ├── OnEnable() - 이벤트 구독
│   ├── OnDisable() - 이벤트 해제
│   └── OnPlayModeStateChanged() - 모드 전환 처리
├── 메인 GUI (93-136)
│   └── OnGUI() - 탭 기반 렌더링
├── Search Tab (138-423)
│   ├── 검색 패널
│   ├── 노드 상세 정보
│   ├── 조상/자손 탐색
│   └── ModelItem 연결
├── Tree View Tab (426-558)
│   ├── 트리 렌더링
│   └── 확장/축소 관리
├── Statistics Tab (561-620)
│   └── 통계 정보 표시
└── Service Resolution (624-652)
    └── VContainer에서 서비스 가져오기
```

### 🔹 4.2 핵심 패턴 1: Play Mode 대응

```csharp
private void OnEnable()
{
    // Play Mode 변경 이벤트 구독
    EditorApplication.playModeStateChanged += OnPlayModeStateChanged;
}

private void OnDisable()
{
    // 이벤트 해제 (메모리 누수 방지)
    EditorApplication.playModeStateChanged -= OnPlayModeStateChanged;
}

private void OnPlayModeStateChanged(PlayModeStateChange state)
{
    // Play Mode 종료 시 상태 초기화
    if (state == PlayModeStateChange.ExitingPlayMode ||
        state == PlayModeStateChange.EnteredEditMode)
    {
        _dataLoadService = null;
        _modelItemRepository = null;
        _selectionTreeCache = null;
        _selectedNode = null;
        _linkedModelItem = null;
        _expandedNodes.Clear();
    }
}
```

```
┌─────────────────────────────────────────────────────────┐
│  Play Mode State Flow                                   │
│                                                         │
│  Edit Mode ──┬──> EnteredPlayMode ──> Playing          │
│              │                            │             │
│              │                            ▼             │
│              └───── EnteredEditMode <── ExitingPlayMode │
│                          │                              │
│                          ▼                              │
│                   [상태 초기화]                          │
│                   - 서비스 참조 null                     │
│                   - UI 상태 클리어                       │
└─────────────────────────────────────────────────────────┘
```

### 🔹 4.3 핵심 패턴 2: 탭 기반 UI

```csharp
private int _selectedTab = 0;
private readonly string[] _tabNames = { "Search", "Tree View", "Statistics" };

private void OnGUI()
{
    // 탭 바 렌더링
    _selectedTab = GUILayout.Toolbar(_selectedTab, _tabNames);

    EditorGUILayout.Space(10);

    // 탭에 따라 다른 콘텐츠 표시
    switch (_selectedTab)
    {
        case 0:
            DrawSearchTab();
            break;
        case 1:
            DrawTreeViewTab();
            break;
        case 2:
            DrawStatisticsTab();
            break;
    }
}
```

```
┌─────────────────────────────────────────────────────────┐
│  ┌─────────┬─────────────┬────────────┐                │
│  │ Search  │  Tree View  │ Statistics │  ← Toolbar     │
│  └─────────┴─────────────┴────────────┘                │
│  ═══════════════════════════════════════════════════   │
│                                                         │
│          [선택된 탭의 콘텐츠가 여기에 표시]              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 🎮 4.4 핵심 패턴 3: 재귀적 트리 렌더링

```csharp
private void DrawTreeNode(SelectionTreeNode node, int depth)
{
    // 종료 조건
    if (node == null || depth > _maxDisplayDepth)
        return;

    EditorGUILayout.BeginHorizontal();

    // 들여쓰기
    GUILayout.Space(depth * 20);

    // 확장/축소 버튼
    bool hasChildren = node.Children != null && node.Children.Count > 0;
    bool isExpanded = _expandedNodes.TryGetValue(node, out var expanded) && expanded;

    if (hasChildren)
    {
        var foldoutLabel = isExpanded ? "▼" : "►";
        if (GUILayout.Button(foldoutLabel, GUILayout.Width(20)))
        {
            _expandedNodes[node] = !isExpanded;
        }
    }
    else
    {
        GUILayout.Space(24); // 정렬을 위한 빈 공간
    }

    // 노드 라벨 (클릭 가능)
    if (GUILayout.Button(label, style))
    {
        _selectedNode = node;
        _selectedTab = 0; // Search 탭으로 전환
    }

    EditorGUILayout.EndHorizontal();

    // 자식 노드 재귀 렌더링
    if (hasChildren && isExpanded)
    {
        foreach (var child in node.Children)
        {
            DrawTreeNode(child, depth + 1);
        }
    }
}
```

```
  depth=0  ▼ Root Node
  depth=1    ▼ Child A
  depth=2      ► Grandchild A1
  depth=2      ► Grandchild A2
  depth=1    ► Child B
  depth=1      Leaf C (자식 없음)
              ↑
              빈 공간으로 정렬 유지
```

### 🔹 4.5 핵심 패턴 4: DI 컨테이너 연동 (VContainer)

```csharp
private bool TryGetServices()
{
    try
    {
        // LifetimeScope 찾기
        var lifetimeScope = FindFirstObjectByType<GameSceneLifetimeScope>();

        if (lifetimeScope?.Container == null)
        {
            return false;
        }

        // 서비스 Resolve
        _dataLoadService = lifetimeScope.Container.Resolve<IDataLoadService>();
        _modelItemRepository = lifetimeScope.Container.Resolve<IModelItemRepository>();

        // 추가 데이터 로드
        if (_dataLoadService != null)
        {
            _selectionTreeCache = _dataLoadService.GetSelectionTreeCache();
        }

        return _dataLoadService != null && _selectionTreeCache != null;
    }
    catch (System.Exception e)
    {
        Debug.LogError($"Failed to resolve services: {e.Message}");
        return false;
    }
}
```

```
┌─────────────────────────────────────────────────────────┐
│  EditorWindow                                           │
│       │                                                 │
│       ▼                                                 │
│  FindFirstObjectByType<GameSceneLifetimeScope>()       │
│       │                                                 │
│       ▼                                                 │
│  LifetimeScope.Container                               │
│       │                                                 │
│       ├──> Resolve<IDataLoadService>()                 │
│       │                                                 │
│       └──> Resolve<IModelItemRepository>()             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 🔹 4.6 핵심 패턴 5: 커스텀 GUIStyle

```csharp
private GUIStyle _nodeStyle;
private GUIStyle _selectedNodeStyle;

private void InitStyles()
{
    // null 체크로 중복 초기화 방지
    if (_nodeStyle == null)
    {
        // 기본 스타일 복사 후 커스터마이즈
        _nodeStyle = new GUIStyle(EditorStyles.label)
        {
            richText = true  // <b>, <color> 등 지원
        };

        _selectedNodeStyle = new GUIStyle(EditorStyles.label)
        {
            richText = true,
            fontStyle = FontStyle.Bold
        };
        _selectedNodeStyle.normal.textColor = new Color(0.2f, 0.6f, 1f);
    }
}
```

---

## 💻 5. EditorWindow 설계 패턴 정리

### 🔹 5.1 레이아웃 패턴

| 패턴 | 코드 | 용도 |
|------|------|------|
| 가로 분할 | `BeginHorizontal/EndHorizontal` | 좌우 패널 |
| 세로 분할 | `BeginVertical/EndVertical` | 상하 패널 |
| 스크롤 | `BeginScrollView/EndScrollView` | 긴 콘텐츠 |
| 탭 | `GUILayout.Toolbar` | 여러 뷰 전환 |
| 폴드아웃 | `EditorGUILayout.Foldout` | 접기/펼치기 |

### 🔹 5.2 입력 컨트롤

```csharp
// 텍스트 입력
string text = EditorGUILayout.TextField("Label", text);

// 정수 입력
int num = EditorGUILayout.IntField("Label", num);

// 슬라이더
float value = EditorGUILayout.Slider("Label", value, 0f, 1f);

// 토글
bool flag = EditorGUILayout.Toggle("Label", flag);

// 드롭다운
int index = EditorGUILayout.Popup("Label", index, options);

// 오브젝트 필드
Object obj = EditorGUILayout.ObjectField("Label", obj, typeof(GameObject), true);

// 버튼
if (GUILayout.Button("Click"))
{
    // 클릭 처리
}
```

### 🔹 5.3 생명주기

```
┌─────────────────────────────────────────────────────────┐
│  EditorWindow 생명주기                                   │
│                                                         │
│  [창 열기]                                               │
│      │                                                   │
│      ▼                                                   │
│  OnEnable() ──────────────────────────┐                 │
│      │                                 │                 │
│      ▼                                 │                 │
│  ┌─────────────┐                       │                 │
│  │   OnGUI()   │ ◄───────────────────┐│                 │
│  │ (매 프레임) │ ─────────────────────┘│                 │
│  └─────────────┘                       │                 │
│      │                                 │                 │
│      ▼                                 │                 │
│  OnDisable() ◄─────────────────────────┘                │
│      │                                                   │
│      ▼                                                   │
│  [창 닫기]                                               │
│                                                         │
│  ※ OnInspectorUpdate() - Inspector 업데이트 시         │
│  ※ OnFocus()/OnLostFocus() - 포커스 변경 시             │
│  ※ OnSelectionChange() - 선택 변경 시                   │
└─────────────────────────────────────────────────────────┘
```

### 🔹 5.4 유용한 콜백

```csharp
// 선택 변경 감지
private void OnSelectionChange()
{
    // Selection.activeObject 변경 시 호출
    Repaint(); // UI 갱신
}

// 정기적 업데이트 (Inspector처럼)
private void OnInspectorUpdate()
{
    // 약 10 FPS로 호출
    Repaint();
}

// 포커스 이벤트
private void OnFocus() { }
private void OnLostFocus() { }

// Hierarchy 변경
private void OnHierarchyChange() { }

// Project 변경
private void OnProjectChange() { }
```

---

## 📌 6. 자주 사용하는 EditorGUILayout 컨트롤

### 🔹 6.1 기본 컨트롤 모음

```csharp
private void DrawAllControls()
{
    // 제목
    EditorGUILayout.LabelField("Section Title", EditorStyles.boldLabel);

    // 읽기 전용 영역
    using (new EditorGUI.DisabledScope(true))
    {
        EditorGUILayout.TextField("ReadOnly", "Cannot edit");
    }

    // 박스로 감싸기
    EditorGUILayout.BeginVertical("box");
    EditorGUILayout.LabelField("Content in box");
    EditorGUILayout.EndVertical();

    // 도움말 박스
    EditorGUILayout.HelpBox("This is info message", MessageType.Info);
    EditorGUILayout.HelpBox("This is warning", MessageType.Warning);
    EditorGUILayout.HelpBox("This is error", MessageType.Error);

    // 구분선
    EditorGUILayout.Space(10);

    // 프로그레스 바
    EditorGUI.ProgressBar(
        EditorGUILayout.GetControlRect(),
        0.7f,
        "70% Complete"
    );
}
```

### 🔹 6.2 고급 컨트롤

```csharp
// 색상 피커
Color color = EditorGUILayout.ColorField("Color", color);

// 커브 에디터
AnimationCurve curve = EditorGUILayout.CurveField("Curve", curve);

// 레이어 마스크
LayerMask mask = EditorGUILayout.MaskField("Layers", mask, layerNames);

// Enum 플래그
MyFlags flags = (MyFlags)EditorGUILayout.EnumFlagsField("Flags", flags);

// 검색 필드
string search = EditorGUILayout.TextField(search, EditorStyles.toolbarSearchField);
```

---

## 📌 7. 실용적인 팁

### ⚡ 7.1 성능 최적화

```csharp
// BAD: OnGUI()에서 매번 Find 호출
private void OnGUI()
{
    var player = GameObject.Find("Player"); // 매 프레임 검색!
}

// GOOD: 캐싱 사용
private GameObject _cachedPlayer;

private void OnGUI()
{
    if (_cachedPlayer == null)
    {
        _cachedPlayer = GameObject.Find("Player");
    }
}
```

### 🔹 7.2 Repaint 호출

```csharp
// 데이터 변경 후 UI 갱신
private void RefreshData()
{
    _data = LoadNewData();
    Repaint(); // 강제로 OnGUI 재호출
}
```

### 🔹 7.3 Undo 지원

```csharp
if (GUILayout.Button("Modify Object"))
{
    Undo.RecordObject(targetObject, "Modified Object");
    targetObject.value = newValue;
    EditorUtility.SetDirty(targetObject);
}
```

---

## 🔗 8. 참고 자료

### 🔹 공식 문서
- [Unity Manual - EditorWindow](https://docs.unity3d.com/Manual/editor-EditorWindows.html)
- [Unity Scripting API - EditorWindow](https://docs.unity3d.com/ScriptReference/EditorWindow.html)
- [Unity Manual - TreeView API](https://docs.unity3d.com/Manual/TreeViewAPI.html)
- [Unity Manual - UI Toolkit EditorWindow](https://docs.unity3d.com/Manual/UIE-HowTo-CreateEditorWindow.html)

### 🔹 튜토리얼
- [Building a Custom Editor Window in Unity (Medium)](https://medium.com/xrpractices/building-a-custom-editor-window-in-unity-5b8a1378e734)
- [Custom Editor Windows in Unity (Medium)](https://medium.com/@dnwesdman/custom-editor-windows-in-unity-15a916f58ac4)
- [UI Toolkit Tutorial (Unity Discussions)](https://discussions.unity.com/t/tutorial-create-an-item-management-editor-window-with-ui-toolkit/849953)
- [Unity Learn: Introduction to Editor Scripting](https://learn.unity.com/tutorial/introduction-to-editor-scripting)
- [WeeklyHow: How To Create Custom Editor Window](https://weeklyhow.com/unity-custom-editor-window/)
- [Yarsa DevBlog: A Detailed Guide to EditorGUILayout](https://blog.yarsalabs.com/exploring-editorguilayout-by-unity-part1/)
- [Unity Editor Scripting Series (Medium)](https://medium.com/@dilaura_exp/unity-editor-scripting-series-chapter-6-guilayout-editorguilayout-c1f309fd15eb)

### 🔹 GitHub 리소스
- [Unity-IMGUI-TreeView](https://github.com/luke161/Unity-IMGUI-TreeView) - 간단한 TreeView 구현
- [Unity C# Reference](https://github.com/Unity-Technologies/UnityCsReference) - Unity 소스 코드 참조
- [RuntimeUnityEditor](https://github.com/ManlyMarco/RuntimeUnityEditor) - 런타임 디버깅 도구

---

## 📌 9. 요약

| 난이도 | 핵심 개념 | 예제 |
|--------|-----------|------|
| **기본** | MenuItem, OnGUI, 버튼 | HelloWorldWindow |
| **중급** | 레이아웃 분할, 스크롤, 검색 | GameObjectDebugWindow |
| **고급** | 탭 UI, 트리뷰, DI 연동, Play Mode 처리 | SelectionTreeDebugWindow |

EditorWindow는 단순한 디버그 출력부터 복잡한 에디터 도구까지 유연하게 확장할 수 있습니다. 핵심은 **점진적 복잡도 증가**입니다:

1. 먼저 기본 창 만들기
2. 필요한 입력 컨트롤 추가
3. 레이아웃 분할로 정보 구조화
4. 상태 관리 및 이벤트 처리 추가
5. 성능 및 UX 최적화

---

*작성일: 2026-02-05*
