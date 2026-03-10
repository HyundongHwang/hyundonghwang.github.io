---
layout: post
title: 260223 bim-api-research
comments: true
tags:
- autodesk
- ifc
- revit
- bim
- performance
- optimization
- api
- ai
- research
- guide
---
# 🔍 BIM 모델링 도구 API 심층 리서치

> 📝 **작성일**: 2026-02-23
> 💡 **리서치 범위**: Revit API, RhinoCommon API, Dynamo API, ArchiCAD API, IFC 오픈소스 라이브러리
> 💡 **대상 독자**: BIM 자동화/플러그인 개발자, 기술 기획자

---

## 📚 목차

1. [Revit API (Autodesk Revit)](#1-revit-api-autodesk-revit)
2. [RhinoCommon API (Rhino / Grasshopper)](#2-rhinocommon-api-rhino--grasshopper)
3. [Dynamo API (Dynamo BIM / DesignScript)](#3-dynamo-api-dynamo-bim--designscript)
4. [ArchiCAD API (Graphisoft ArchiCAD)](#4-archicad-api-graphisoft-archicad)
5. [기타 BIM 도구 API (IFC 라이브러리 및 오픈소스)](#5-기타-bim-도구-api)
6. [API 종합 비교](#6-api-종합-비교)
7. [참고 문헌](#7-참고-문헌)

---

## 💻 1. Revit API (Autodesk Revit)

### 🧭 1.1 전반적인 소개

#### 🧭 API 개요 및 아키텍처

Revit API는 Autodesk Revit의 내부 데이터와 기능에 프로그래밍 방식으로 접근할 수 있게 하는 **.NET 기반 API**다. 개발자는 IExternalCommand, IExternalApplication 등의 인터페이스를 구현하여 Revit의 거의 모든 기능을 자동화하거나 확장할 수 있다.

**핵심 네임스페이스 구조**:

| 네임스페이스 | 역할 |
|---|---|
| `Autodesk.Revit.DB` | 모델 요소, 파라미터, 지오메트리 등 핵심 데이터베이스 |
| `Autodesk.Revit.UI` | UI 확장, 리본 패널, 대화상자 |
| `Autodesk.Revit.ApplicationServices` | 애플리케이션 레벨 서비스 |
| `Autodesk.Revit.Creation` | 요소 생성 팩토리 메서드 |
| `Autodesk.Revit.Attributes` | 트랜잭션 모드 등 어트리뷰트 |

**아키텍처 흐름**: ExternalCommand 실행 -> ExternalCommandData 수신 -> Application/Document 접근 -> Transaction 내에서 모델 조작 -> 결과 반환

#### ▫️ 지원 언어

- **C# (공식 지원)**: .NET SDK를 통한 풀 액세스. 모든 공식 예제와 SDK가 C# 기반
- **Python (비공식/서드파티)**: [pyRevit](https://docs.pyrevitlabs.io/)를 통한 IronPython/CPython 지원. [RevitPythonShell](https://github.com/architecture-building-systems/revitpythonshell)도 활용 가능
- **VB.NET**: 지원되나 커뮤니티에서 거의 사용하지 않음

#### 💻 API 버전 및 최신 동향 (2025-2026)

- **Revit 2025**: .NET 8 기반으로 전환. 기존 .NET Framework 기반 애드인은 재빌드 필요 ([revitapidocs.com/2025](https://www.revitapidocs.com/2025/))
- **Revit 2026**: 애드인을 별도의 Assembly Load Context에서 실행하는 옵션 도입으로 의존성 충돌 감소. CefSharp 의존성 제거 ([rvtdocs.com/2026/whatsnew](https://rvtdocs.com/2026/whatsnew))
- **pyRevit 5.0**: Revit 2025 API 최적화, 디지털 서명된 설치 파일 제공 ([archilabs.ai](https://archilabs.ai/posts/pyrevit-2025))

#### 🔗 공식 문서 URL

- Revit API Docs (비공식, 검색 가능): [revitapidocs.com](https://www.revitapidocs.com/)
- Revit API 2026 온라인 문서: [rvtdocs.com/2026](https://rvtdocs.com/2026/whatsnew)
- Autodesk 공식 개발자 센터: [aps.autodesk.com/developer/overview/revit](https://aps.autodesk.com/developer/overview/revit)
- Revit API 개발자 가이드: [Autodesk Help](https://help.autodesk.com/view/RVT/2025/ENU/?guid=Revit_API_Revit_API_Developers_Guide_html)

### ⚠️ 1.2 장단점

**강점**:
- BIM 업계 표준 도구의 거의 모든 기능에 대한 프로그래밍적 접근 가능
- .NET 8 기반으로 최신 성능 및 보안 이점 활용
- FilteredElementCollector를 통한 강력한 요소 검색/필터링 시스템
- 대규모 커뮤니티 (The Building Coder 블로그, Autodesk 포럼, Stack Overflow revit-api 태그)
- NuGet 패키지를 통한 간편한 SDK 배포 ([NuGet Revit SDK](https://www.nuget.org/packages/Autodesk.Revit.SDK))

**약점**:
- 공식적으로 C#/.NET만 지원하여 언어 선택의 제약이 큼
- 매 버전마다 API 변경이 발생하여 유지보수 부담 (특히 .NET Framework -> .NET 8 마이그레이션)
- 모든 모델 변경은 Transaction 내에서 수행해야 하는 구조적 제약
- 공식 API 문서(CHM 파일)의 검색성이 떨어지며, 비공식 사이트(revitapidocs.com)에 의존하는 경우가 많음
- Revit 실행 환경 내에서만 API 호출 가능 (외부 독립 실행 불가, DA4R 제외)

### 🧪 1.3 간단 개념 예제

#### ▫️ C# - Hello World (ExternalCommand)

```csharp
using Autodesk.Revit.Attributes;
using Autodesk.Revit.DB;
using Autodesk.Revit.UI;

[Transaction(TransactionMode.Manual)]
public class HelloRevit : IExternalCommand
{
    public Result Execute(
        ExternalCommandData commandData,
        ref string message,
        ElementSet elements)
    {
        // 현재 활성 문서 가져오기
        UIDocument uidoc = commandData.Application.ActiveUIDocument;
        Document doc = uidoc.Document;

        // 프로젝트 정보 표시
        TaskDialog.Show("Hello Revit",
            $"프로젝트: {doc.Title}\n요소 수: {new FilteredElementCollector(doc).WhereElementIsNotElementType().GetElementCount()}");

        return Result.Succeeded;
    }
}
```

#### ▫️ C# - 벽 생성

```csharp
using Autodesk.Revit.DB;

// Transaction 내에서 실행 필요
using (Transaction tx = new Transaction(doc, "벽 생성"))
{
    tx.Start();

    // 시작점과 끝점으로 라인 생성
    XYZ start = new XYZ(0, 0, 0);
    XYZ end = new XYZ(20, 0, 0);   // 단위: feet
    Line wallLine = Line.CreateBound(start, end);

    // 레벨 가져오기
    Level level = new FilteredElementCollector(doc)
        .OfClass(typeof(Level))
        .First() as Level;

    // 벽 생성 (라인, 레벨ID, 구조여부)
    Wall wall = Wall.Create(doc, wallLine, level.Id, false);

    tx.Commit();
}
```

#### ▫️ Python (pyRevit) - 모든 벽 목록 출력

```python
# pyRevit 스크립트 예제
from pyrevit import revit, DB

# 현재 문서에서 모든 벽 수집
doc = revit.doc
walls = DB.FilteredElementCollector(doc)\
    .OfClass(DB.Wall)\
    .ToElements()

# 벽 정보 출력
for wall in walls:
    wall_type = wall.WallType.Name
    print("벽 ID: {} | 타입: {} | 길이: {:.2f}m".format(
        wall.Id.IntegerValue,
        wall_type,
        wall.get_Parameter(DB.BuiltInParameter.CURVE_ELEM_LENGTH).AsDouble() * 0.3048
    ))
```

### 🧪 1.4 실무 예제

#### 🏢 C# - FilteredElementCollector를 활용한 데이터 추출 및 Excel 내보내기

```csharp
using Autodesk.Revit.DB;
using System.IO;
using System.Linq;
using System.Text;

// 모든 문(Door) 요소의 파라미터를 CSV로 내보내기
public void ExportDoorDataToCsv(Document doc, string filePath)
{
    // 문 카테고리 요소 수집
    var doors = new FilteredElementCollector(doc)
        .OfCategory(BuiltInCategory.OST_Doors)
        .WhereElementIsNotElementType()
        .ToElements();

    var sb = new StringBuilder();
    sb.AppendLine("ID,이름,너비(mm),높이(mm),레벨,방화등급");

    foreach (Element door in doors)
    {
        string id = door.Id.IntegerValue.ToString();
        string name = door.Name;

        // 파라미터 값 읽기 (Built-in 및 공유 파라미터)
        double width = door.get_Parameter(BuiltInParameter.DOOR_WIDTH)?.AsDouble() ?? 0;
        double height = door.get_Parameter(BuiltInParameter.DOOR_HEIGHT)?.AsDouble() ?? 0;

        // feet -> mm 변환
        width = UnitUtils.ConvertFromInternalUnits(width, UnitTypeId.Millimeters);
        height = UnitUtils.ConvertFromInternalUnits(height, UnitTypeId.Millimeters);

        string level = door.get_Parameter(BuiltInParameter.FAMILY_LEVEL_PARAM)?.AsValueString() ?? "N/A";
        string fireRating = door.LookupParameter("방화등급")?.AsString() ?? "미지정";

        sb.AppendLine($"{id},{name},{width:F0},{height:F0},{level},{fireRating}");
    }

    File.WriteAllText(filePath, sb.ToString(), Encoding.UTF8);
}
```

#### ▫️ Python (pyRevit) - 파라미터 일괄 수정

```python
# pyRevit: 선택한 요소들의 마크(Mark) 파라미터를 일괄 수정
from pyrevit import revit, DB, forms

doc = revit.doc

# 카테고리 선택 대화상자
selected_cat = forms.SelectFromList.show(
    ['Walls', 'Doors', 'Windows'],
    title='파라미터 수정 대상 카테고리'
)

# 카테고리별 BuiltInCategory 매핑
cat_map = {
    'Walls': DB.BuiltInCategory.OST_Walls,
    'Doors': DB.BuiltInCategory.OST_Doors,
    'Windows': DB.BuiltInCategory.OST_Windows
}

if selected_cat:
    elements = DB.FilteredElementCollector(doc)\
        .OfCategory(cat_map[selected_cat])\
        .WhereElementIsNotElementType()\
        .ToElements()

    # 접두사 입력
    prefix = forms.ask_for_string(
        prompt='마크 접두사를 입력하세요',
        title='일괄 마크 수정'
    )

    if prefix:
        with revit.Transaction("마크 일괄 수정"):
            count = 0
            for i, elem in enumerate(elements):
                mark_param = elem.get_Parameter(DB.BuiltInParameter.ALL_MODEL_MARK)
                if mark_param and not mark_param.IsReadOnly:
                    mark_param.Set("{}-{:04d}".format(prefix, i + 1))
                    count += 1

        print("{}개 요소의 마크를 수정했습니다.".format(count))
```

---

## 💻 2. RhinoCommon API (Rhino / Grasshopper)

### 🧭 2.1 전반적인 소개

#### 🧭 API 개요 및 아키텍처

RhinoCommon은 Rhino의 핵심 **.NET SDK**로, Rhino와 Grasshopper의 모든 지오메트리 타입과 기능에 프로그래밍적으로 접근할 수 있는 크로스 플랫폼 라이브러리다. Grasshopper 자체가 RhinoCommon 플러그인이므로, 모든 Grasshopper 컴포넌트는 RhinoCommon에 기반한다.

**주요 네임스페이스**:

| 네임스페이스 | 역할 |
|---|---|
| `Rhino.Geometry` | Point, Curve, Surface, Brep, Mesh 등 핵심 지오메트리 |
| `Rhino.DocObjects` | 문서 내 객체 관리 |
| `Rhino.Commands` | 커맨드 인터페이스 |
| `Rhino.Input` | 사용자 입력 처리 |
| `Rhino.Display` | 뷰포트 표시 및 디스플레이 파이프라인 |
| `Grasshopper.Kernel` | Grasshopper 컴포넌트 프레임워크 |

#### ▫️ 지원 언어

- **C#**: 풀 기능 플러그인 개발. Visual Studio 템플릿 제공 ([developer.rhino3d.com](https://developer.rhino3d.com/guides/grasshopper/csharp-essentials/))
- **Python**: Rhino 8 기준 IronPython 2.7 + CPython 3.9 지원. `rhinoscriptsyntax`(래퍼)와 `Rhino.Geometry`(네이티브) 두 가지 접근 방식
- **VB.NET**: Grasshopper 스크립팅 컴포넌트에서 지원

#### 💻 API 버전 및 최신 동향

- **Rhino 8**: .NET Core 지원 추가. CPython 3.9 네이티브 지원으로 NumPy, SciPy 등 과학 라이브러리 활용 가능
- **Rhino.Inside.Revit**: Revit 2025/2026과 호환 (Rhino 8 필수, .NET Core 8 기반). 300개 이상의 Revit 전용 Grasshopper 컴포넌트 제공 ([rhino3d.com/inside/revit](https://www.rhino3d.com/inside/revit/1.0/reference/rir-api))

#### 🔗 공식 문서 URL

- 개발자 문서: [developer.rhino3d.com](https://developer.rhino3d.com/)
- API 레퍼런스: [developer.rhino3d.com/api](https://developer.rhino3d.com/api/)
- 샘플 코드: [github.com/mcneel/rhino-developer-samples](https://github.com/mcneel/rhino-developer-samples)

### ⚠️ 2.2 장단점

**강점**:
- NURBS 기반 자유 곡면 모델링에 최적화된 강력한 지오메트리 엔진
- Grasshopper를 통한 비주얼 프로그래밍과 C#/Python 스크립팅의 자연스러운 통합
- 크로스 플랫폼 지원 (Windows, macOS)
- Rhino.Inside 기술로 Revit, AutoCAD 등 타 BIM 도구와 연동 가능
- 풍부한 공식 문서와 활발한 McNeel 포럼 커뮤니티

**약점**:
- BIM 전용 도구가 아니므로 건축 요소(벽, 문, 창) 등의 BIM 의미론이 내장되지 않음
- Grasshopper 정의 파일의 버전 관리가 어렵고, 대규모 프로젝트에서 복잡도가 급격히 증가
- IronPython 2.7의 한계 (Python 3 문법 미지원, Rhino 8에서 CPython으로 대체 중)
- 라이선스 비용이 발생 (상용 소프트웨어)

### 🧪 2.3 간단 개념 예제

#### ▫️ C# - Grasshopper 컴포넌트에서 점/선/면 생성

```csharp
// Grasshopper C# 스크립팅 컴포넌트 내부
using Rhino.Geometry;

// 점 생성
Point3d pt1 = new Point3d(0, 0, 0);
Point3d pt2 = new Point3d(10, 0, 0);
Point3d pt3 = new Point3d(10, 10, 0);
Point3d pt4 = new Point3d(0, 10, 0);

// 선 생성
Line line = new Line(pt1, pt2);

// 폴리라인으로 사각형 생성
Polyline rect = new Polyline(new Point3d[] { pt1, pt2, pt3, pt4, pt1 });

// NURBS 곡면 생성 (평면)
PlaneSurface planeSrf = new PlaneSurface(
    Plane.WorldXY,
    new Interval(0, 10),  // X 범위
    new Interval(0, 10)   // Y 범위
);

// 출력 할당 (Grasshopper 컴포넌트)
A = line;
B = rect;
C = planeSrf;
```

#### 🏢 Python - rhinoscriptsyntax를 활용한 기본 지오메트리

```python
import rhinoscriptsyntax as rs
import Rhino.Geometry as rg

# rhinoscriptsyntax 방식 - 간결하고 직관적
line_id = rs.AddLine((0, 0, 0), (10, 0, 0))
circle_id = rs.AddCircle((5, 5, 0), 3)

# RhinoCommon 직접 사용 - 더 강력하고 세밀한 제어
pt1 = rg.Point3d(0, 0, 0)
pt2 = rg.Point3d(10, 0, 0)
pt3 = rg.Point3d(10, 10, 5)

# 3개 점을 지나는 NURBS 커브
curve = rg.NurbsCurve.Create(False, 3, [pt1, pt2, pt3])

# 커브를 회전하여 곡면 생성 (Revolution Surface)
axis = rg.Line(rg.Point3d(0, 0, 0), rg.Point3d(0, 0, 1))
rev_srf = rg.RevSurface.Create(curve, axis)
brep = rev_srf.ToBrep()

print("생성된 Brep 면 수: {}".format(brep.Faces.Count))
```

### 🧪 2.4 실무 예제

#### 🔍 C# - 메시 분석 및 데이터 추출

```csharp
using Rhino.Geometry;
using System.Collections.Generic;
using System.Linq;

// Grasshopper 컴포넌트: 메시 품질 분석
public void AnalyzeMesh(Mesh mesh)
{
    // 메시 유효성 검증
    mesh.Vertices.CombineIdentical(true, true);
    mesh.FaceNormals.ComputeFaceNormals();
    mesh.Normals.ComputeNormals();

    // 면적/체적 계산
    var areaProps = AreaMassProperties.Compute(mesh);
    var volProps = VolumeMassProperties.Compute(mesh);

    double totalArea = areaProps.Area;
    double volume = volProps.Volume;

    // 면별 각도 분석 (수직면, 수평면 분류)
    List<int> verticalFaces = new List<int>();
    List<int> horizontalFaces = new List<int>();

    for (int i = 0; i < mesh.Faces.Count; i++)
    {
        Vector3d normal = mesh.FaceNormals[i];
        double angle = Vector3d.VectorAngle(normal, Vector3d.ZAxis);

        // 수평: Z축과의 각도가 10도 이내
        if (angle < Rhino.RhinoMath.ToRadians(10))
            horizontalFaces.Add(i);
        // 수직: Z축과의 각도가 80~100도
        else if (Math.Abs(angle - Math.PI / 2) < Rhino.RhinoMath.ToRadians(10))
            verticalFaces.Add(i);
    }

    // 결과 출력
    A = totalArea;          // 전체 면적
    B = volume;             // 체적
    C = verticalFaces;      // 수직면 인덱스
    D = horizontalFaces;    // 수평면 인덱스
}
```

#### ▫️ Python - 외부 데이터(CSV) 기반 3D 형상 생성

```python
import Rhino.Geometry as rg
import csv
import os

# CSV 파일에서 좌표 데이터 읽기
csv_path = r"C:\data\building_footprints.csv"
buildings = []

with open(csv_path, 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        # CSV 컬럼: x, y, width, depth, height
        x = float(row['x'])
        y = float(row['y'])
        w = float(row['width'])
        d = float(row['depth'])
        h = float(row['height'])

        # 바닥 사각형 생성
        corners = [
            rg.Point3d(x, y, 0),
            rg.Point3d(x + w, y, 0),
            rg.Point3d(x + w, y + d, 0),
            rg.Point3d(x, y + d, 0)
        ]

        # 폴리라인 -> 커브 -> 돌출(Extrusion)
        polyline = rg.Polyline(corners + [corners[0]])
        curve = polyline.ToNurbsCurve()
        extrusion = rg.Extrusion.Create(
            curve, h, True  # 높이, 캡 여부
        )

        if extrusion:
            buildings.append(extrusion.ToBrep())

print("{}개 건물 형상 생성 완료".format(len(buildings)))

# Grasshopper 출력
a = buildings
```

---

## 🏢 3. Dynamo API (Dynamo BIM / DesignScript)

### 🧭 3.1 전반적인 소개

#### 🧭 API 개요 및 아키텍처

Dynamo는 Autodesk 제품군(특히 Revit, Civil 3D)과 통합되는 **비주얼 프로그래밍 환경**이다. 핵심 구성요소는 다음과 같다:

- **Dynamo Core**: 그래프 인터페이스, 컴퓨트 엔진, DesignScript 언어, 기본 노드
- **DesignScript**: Dynamo 네이티브 텍스트 기반 스크립팅 언어 (C#과 유사한 문법)
- **Python 노드**: CPython/PythonNet을 통한 Python 스크립팅
- **Zero Touch**: C# 클래스 라이브러리를 Dynamo 노드로 자동 변환하는 기술

**실행 흐름**: 노드 그래프 구성 -> DesignScript/Python으로 변환 -> Compute Engine 실행 -> Revit API 호출 (Revit 통합 시) -> 결과 시각화

#### ▫️ 지원 언어

- **DesignScript**: Dynamo 네이티브 언어. 코드 블록에서 직접 사용
- **Python**: PythonNet3 (Dynamo 4.0 이후 기본 엔진). `revit` 라이브러리를 통한 Revit API 접근 가능
- **C#**: Zero Touch Import를 통해 C# 클래스 라이브러리를 노드로 변환

#### 💻 API 버전 및 최신 동향

- **Dynamo Core 4.0** (2025-2026): .NET 10 마이그레이션, PythonNet3 기본 엔진 전환, CPython에서 자동 마이그레이션, PolySurface/PolyCurve/Boolean 노드 성능 향상 ([dynamobim.org/dynamo-core-4-0-release](https://dynamobim.org/dynamo-core-4-0-release/))
- **Dynamo Core 3.6**: Revit 2025/2026용. 파일 로딩 속도 2배 향상 ([dynamobim.org](https://dynamobim.org/dynamo-core-3-6-release/))
- Autodesk 2026 제품군과 호환

#### 🔗 공식 문서 URL

- Dynamo BIM 공식: [dynamobim.org](https://dynamobim.org/)
- Dynamo Primer (학습서): [primer.dynamobim.org](https://primer.dynamobim.org/)
- GitHub 위키 (릴리스 노트): [github.com/DynamoDS/Dynamo/wiki](https://github.com/DynamoDS/Dynamo/wiki/Release-Notes)
- Python Primer: [dynamopythonprimer.gitbook.io](https://dynamopythonprimer.gitbook.io/dynamo-python-primer/)

### ⚠️ 3.2 장단점

**강점**:
- 비주얼 프로그래밍으로 비개발자도 접근 가능한 낮은 진입 장벽
- Revit과의 네이티브 통합 (Revit 내장)
- DesignScript + Python + C# (Zero Touch) 세 가지 스크립팅 방식 지원
- 활발한 패키지 생태계 (Dynamo Package Manager)
- 오픈소스로 커뮤니티 기여 활발

**약점**:
- 대규모/복잡한 그래프의 관리와 디버깅이 어려움
- 순수 Revit API 대비 성능 오버헤드 존재
- DesignScript의 학습 자료가 Python/C#에 비해 부족
- 복잡한 로직(조건 분기, 루프 등)의 비주얼 프로그래밍 표현이 비효율적
- Revit 외 다른 BIM 도구와의 통합이 제한적 (Grasshopper 대비)

### 🧪 3.3 간단 개념 예제

#### 🧪 DesignScript - 코드 블록 기본

```
// DesignScript 코드 블록 예제

// 변수 선언
x = 0..10..1;          // 0부터 10까지 1 간격의 시퀀스
y = Math.Sin(x) * 5;   // 사인 함수 적용

// 점 생성
points = Point.ByCoordinates(x, y, 0);

// 점들을 잇는 NURBS 커브
curve = NurbsCurve.ByPoints(points, 3);

// 커브를 돌출하여 면 생성
surface = curve.Extrude(Vector.ByCoordinates(0, 0, 3));
```

#### ▫️ Python 노드 - Revit 요소 접근

```python
# Dynamo Python 노드 (PythonNet3)
import clr

# Revit API 라이브러리 로드
clr.AddReference('RevitAPI')
clr.AddReference('RevitServices')

from Autodesk.Revit.DB import *
from RevitServices.Persistence import DocumentManager

# 현재 Revit 문서 가져오기
doc = DocumentManager.Instance.CurrentDBDocument

# 모든 레벨 수집
collector = FilteredElementCollector(doc)
levels = collector.OfClass(Level).ToElements()

# 레벨 이름과 높이 출력
output = []
for level in levels:
    name = level.Name
    elevation = level.Elevation * 304.8  # feet -> mm
    output.append("{}: {:.0f}mm".format(name, elevation))

# Dynamo 출력 (OUT 변수에 할당)
OUT = output
```

### 🧪 3.4 실무 예제

#### ▫️ Python 노드 - Revit 파라미터 일괄 수정 및 데이터 내보내기

```python
# Dynamo Python 노드: 방(Room) 데이터를 CSV로 내보내기
import clr
import csv

clr.AddReference('RevitAPI')
clr.AddReference('RevitServices')

from Autodesk.Revit.DB import *
from Autodesk.Revit.DB.Architecture import Room
from RevitServices.Persistence import DocumentManager

doc = DocumentManager.Instance.CurrentDBDocument

# 모든 Room 수집
rooms = FilteredElementCollector(doc)\
    .OfCategory(BuiltInCategory.OST_Rooms)\
    .WhereElementIsNotElementType()\
    .ToElements()

# 데이터 수집
room_data = []
for room in rooms:
    if room.Area > 0:  # 배치된 방만
        data = {
            'Number': room.Number,
            'Name': room.get_Parameter(BuiltInParameter.ROOM_NAME).AsString(),
            'Area_sqm': round(room.Area * 0.092903, 2),  # sqft -> sqm
            'Level': room.Level.Name,
            'Department': room.get_Parameter(
                BuiltInParameter.ROOM_DEPARTMENT
            ).AsString() or 'N/A'
        }
        room_data.append(data)

# CSV 내보내기
output_path = r"C:\temp\room_schedule.csv"
with open(output_path, 'w', newline='', encoding='utf-8-sig') as f:
    writer = csv.DictWriter(f, fieldnames=room_data[0].keys())
    writer.writeheader()
    writer.writerows(room_data)

OUT = "{}개 방 데이터를 {}에 저장했습니다.".format(len(room_data), output_path)
```

#### ▫️ C# Zero Touch 노드 - 커스텀 지오메트리 유틸리티

```csharp
// Zero Touch Import용 C# 클래스
// 빌드 후 DLL을 Dynamo에서 Import하면 자동으로 노드화됨
using Autodesk.DesignScript.Geometry;
using System.Collections.Generic;

namespace BimToolkit
{
    /// <summary>
    /// BIM 지오메트리 유틸리티 노드 모음
    /// </summary>
    public static class GeometryUtils
    {
        /// <summary>
        /// 격자 패턴의 점 배열 생성
        /// </summary>
        /// <param name="origin">시작점</param>
        /// <param name="xCount">X방향 개수</param>
        /// <param name="yCount">Y방향 개수</param>
        /// <param name="spacing">간격</param>
        /// <returns>2D 점 리스트</returns>
        public static List<List<Point>> CreateGrid(
            Point origin, int xCount, int yCount, double spacing)
        {
            var grid = new List<List<Point>>();
            for (int i = 0; i < xCount; i++)
            {
                var row = new List<Point>();
                for (int j = 0; j < yCount; j++)
                {
                    row.Add(Point.ByCoordinates(
                        origin.X + i * spacing,
                        origin.Y + j * spacing,
                        origin.Z));
                }
                grid.Add(row);
            }
            return grid;
        }

        /// <summary>
        /// 폴리곤을 지정 높이로 돌출하여 솔리드 생성
        /// </summary>
        public static Solid ExtrudePolygon(
            List<Point> vertices, double height)
        {
            // 점 목록으로 폴리커브 생성
            var pts = new List<Point>(vertices);
            pts.Add(vertices[0]);  // 닫힌 커브

            var curve = PolyCurve.ByPoints(pts, true);
            var surface = Surface.ByPatch(curve);
            var solid = surface.Thicken(height);

            return solid;
        }
    }
}
```

---

## 💻 4. ArchiCAD API (Graphisoft ArchiCAD)

### 🧭 4.1 전반적인 소개

#### 🧭 API 개요 및 아키텍처

ArchiCAD API는 두 가지 주요 인터페이스를 제공한다:

1. **C++ API (Add-On Development Kit)**: 네이티브 플러그인(Add-On, .apx 확장자) 개발용. Archicad의 전체 기능에 접근 가능
2. **Python API (JSON Interface)**: HTTP 기반 JSON 명령 인터페이스의 Python 래퍼. 외부 스크립트에서 ArchiCAD를 원격 제어

**C++ API 아키텍처**: Add-On은 컴파일된 DLL로, ArchiCAD가 로드하여 실행. API_Element, API_ElementMemo 구조체를 채워 요소를 생성/수정하는 방식

**Python API 아키텍처**: ArchiCAD 내장 HTTP 서버 <-> JSON 메시지 교환 <-> Python `archicad` 패키지가 JSON 레이어를 추상화

#### ▫️ 지원 언어

- **C++**: 공식 Add-On 개발 언어. API Development Kit 필요 ([archicadapi.graphisoft.com](https://archicadapi.graphisoft.com/))
- **Python**: `archicad` PyPI 패키지를 통한 JSON 인터페이스 접근 (Python 3.7+) ([pypi.org/project/archicad](https://pypi.org/project/archicad/))

#### 💻 API 버전 및 최신 동향

- **ArchiCAD 29**: C++ API DevKit v29.0.1 (build 3100) 출시 ([github.com/GRAPHISOFT/archicad-api-devkit](https://github.com/GRAPHISOFT/archicad-api-devkit))
- ArchiCAD 26 이상에서 Add-On 하위 호환성 유지
- Python 패키지는 ArchiCAD 24 이상 지원

#### 🔗 공식 문서 URL

- ArchiCAD API 포털: [archicadapi.graphisoft.com](https://archicadapi.graphisoft.com/)
- C++ API 레퍼런스 (v29): [graphisoft.github.io/archicad-api-devkit](https://graphisoft.github.io/archicad-api-devkit/)
- Python 패키지 문서: [archicadapi.graphisoft.com/archicadPythonPackage](https://archicadapi.graphisoft.com/archicadPythonPackage/archicad.html)
- JSON Interface 문서: [archicadapi.graphisoft.com/JSONInterfaceDocumentation](https://archicadapi.graphisoft.com/JSONInterfaceDocumentation/)

### ⚠️ 4.2 장단점

**강점**:
- C++ API를 통한 ArchiCAD 전체 기능의 네이티브 레벨 접근
- Python JSON 인터페이스로 외부 프로세스에서 ArchiCAD 제어 가능 (Revit API와 차별화)
- GitHub 템플릿으로 Add-On 개발 시작이 비교적 용이
- 요소 생성이 API_Element 구조체 채우기 방식으로 직관적

**약점**:
- C++ 개발이 필수여서 진입 장벽이 높음 (Python API는 기능이 제한적)
- Revit/Rhino 대비 개발자 커뮤니티 규모가 현저히 작음
- Python API는 Add-On이 아니며, C++ Add-On으로 변환 불가능
- 문서화 수준이 Revit API 대비 부족하며, 예제 코드가 적음
- 시장 점유율이 상대적으로 낮아 서드파티 도구 생태계가 빈약

### 🧪 4.3 간단 개념 예제

#### ▫️ Python - ArchiCAD 연결 및 벽 목록 조회

```python
# ArchiCAD Python API: 벽 목록 조회
from archicad import ACConnection

# ArchiCAD에 연결 (ArchiCAD가 실행 중이어야 함)
conn = ACConnection.connect()
assert conn, "ArchiCAD 연결 실패"

acc = conn.commands   # 명령 인터페이스
act = conn.types      # 타입 정의
acu = conn.utilities  # 유틸리티 함수

# 모든 벽 요소 가져오기
walls = acc.GetElementsByType('Wall')
print("벽 개수: {}".format(len(walls)))

# 벽의 속성값 조회
property_ids = [
    acu.GetBuiltInPropertyId('General_ElementID'),
    acu.GetBuiltInPropertyId('General_Height'),
    acu.GetBuiltInPropertyId('General_Thickness')
]

# 각 벽의 속성 출력
element_ids = [wall.elementId for wall in walls]
property_values = acc.GetPropertyValuesOfElements(element_ids, property_ids)

for i, wall_props in enumerate(property_values):
    print("벽 {}: ID={}, 높이={}, 두께={}".format(
        i + 1,
        wall_props[0].propertyValue.value if wall_props[0].propertyValue else 'N/A',
        wall_props[1].propertyValue.value if wall_props[1].propertyValue else 'N/A',
        wall_props[2].propertyValue.value if wall_props[2].propertyValue else 'N/A'
    ))
```

#### 🏗️ C++ - Add-On 기본 구조 (요소 생성)

```cpp
// ArchiCAD C++ Add-On: 벽 요소 생성
#include "ACAPinc.h"  // ArchiCAD API 헤더

GSErrCode CreateWall()
{
    API_Element element = {};
    API_ElementMemo memo = {};

    // 요소 타입 설정
    element.header.type = API_WallID;

    // 기본값으로 초기화
    GSErrCode err = ACAPI_Element_GetDefaults(&element, &memo);
    if (err != NoError) return err;

    // 벽 파라미터 설정
    element.wall.begC = { 0.0, 0.0 };       // 시작점 (m 단위)
    element.wall.endC = { 5.0, 0.0 };       // 끝점
    element.wall.height = 3.0;               // 높이 3m
    element.wall.thickness = 0.2;            // 두께 200mm
    element.wall.bottomOffset = 0.0;         // 바닥 오프셋

    // 요소 생성
    err = ACAPI_Element_Create(&element, &memo);

    // 메모리 해제
    ACAPI_DisposeElemMemoHdls(&memo);

    return err;
}
```

### 🧪 4.4 실무 예제

#### ▫️ Python - 요소 속성 일괄 조회 및 JSON 내보내기

```python
# ArchiCAD Python API: 프로젝트 데이터를 JSON으로 내보내기
import json
from archicad import ACConnection

conn = ACConnection.connect()
assert conn

acc = conn.commands
acu = conn.utilities

# 여러 요소 타입 조회
element_types = ['Wall', 'Column', 'Slab', 'Door', 'Window']
project_data = {}

for elem_type in element_types:
    elements = acc.GetElementsByType(elem_type)

    if not elements:
        continue

    # 속성 ID 준비
    prop_ids = [
        acu.GetBuiltInPropertyId('General_ElementID'),
        acu.GetBuiltInPropertyId('General_Area'),
        acu.GetBuiltInPropertyId('General_Volume')
    ]

    elem_ids = [e.elementId for e in elements]
    props = acc.GetPropertyValuesOfElements(elem_ids, prop_ids)

    type_data = []
    for i, elem_props in enumerate(props):
        entry = {
            'guid': str(elem_ids[i].guid),
            'element_id': str(elem_props[0].propertyValue.value) if elem_props[0].propertyValue else None,
            'area': elem_props[1].propertyValue.value if elem_props[1].propertyValue else None,
            'volume': elem_props[2].propertyValue.value if elem_props[2].propertyValue else None
        }
        type_data.append(entry)

    project_data[elem_type] = {
        'count': len(type_data),
        'elements': type_data
    }

# JSON 파일로 내보내기
output_path = r"C:\temp\archicad_export.json"
with open(output_path, 'w', encoding='utf-8') as f:
    json.dump(project_data, f, indent=2, ensure_ascii=False)

print("내보내기 완료: {}".format(output_path))
for et, data in project_data.items():
    print("  {}: {}개".format(et, data['count']))
```

---

## 🏢 5. 기타 BIM 도구 API

### 🔹 5.1 IfcOpenShell (IFC 오픈소스 라이브러리)

#### 🧭 개요

[IfcOpenShell](https://ifcopenshell.org/)은 IFC(Industry Foundation Classes) 파일을 읽고, 쓰고, 조작하기 위한 **오픈소스(LGPL) 라이브러리**다. C++ 지오메트리 엔진과 풍부한 Python API를 제공하며, BIM 데이터의 도구 독립적인 처리가 가능하다.

**지원 포맷**: IFC2X3, IFC4, IFC4X3 / IFC-SPF, IFCJSON, IFCXML, IFCHDF5, IFCSQL

**공식 문서**: [docs.ifcopenshell.org](https://docs.ifcopenshell.org/ifcopenshell-python/code_examples.html)

#### 🗂️ Python 예제 - IFC 파일 읽기 및 벽 데이터 추출

```python
import ifcopenshell
import ifcopenshell.util.element
import ifcopenshell.geom

# IFC 파일 열기
model = ifcopenshell.open("building.ifc")

# 프로젝트 정보
project = model.by_type("IfcProject")[0]
print("프로젝트: {}".format(project.Name))

# 모든 벽 조회
walls = model.by_type("IfcWall")
print("벽 개수: {}".format(len(walls)))

# 벽 상세 정보 추출
for wall in walls[:5]:  # 처음 5개만
    # 기본 속성
    info = wall.get_info()
    print("\n벽: {} (GlobalId: {})".format(wall.Name, wall.GlobalId))

    # 프로퍼티 셋 조회
    psets = ifcopenshell.util.element.get_psets(wall)
    for pset_name, props in psets.items():
        print("  [{}]".format(pset_name))
        for key, value in props.items():
            if key != 'id':
                print("    {}: {}".format(key, value))

# 지오메트리 추출
settings = ifcopenshell.geom.settings()
wall = walls[0]
shape = ifcopenshell.geom.create_shape(settings, wall)

# 삼각형 메시 데이터
verts = shape.geometry.verts     # [x1,y1,z1, x2,y2,z2, ...]
faces = shape.geometry.faces     # [v1,v2,v3, v1,v2,v3, ...]
print("\n지오메트리 - 정점: {}, 면: {}".format(
    len(verts) // 3, len(faces) // 3
))
```

#### 🗂️ Python 예제 - IFC 파일 생성

```python
import ifcopenshell

# 새 IFC4 파일 생성
ifc = ifcopenshell.file(schema="IFC4")

# 프로젝트 구조 생성
project = ifc.create_entity("IfcProject", Name="테스트 프로젝트")
context = ifc.create_entity("IfcGeometricRepresentationContext",
    ContextType="Model", CoordinateSpaceDimension=3)

site = ifc.create_entity("IfcSite", Name="현장")
building = ifc.create_entity("IfcBuilding", Name="건물")
storey = ifc.create_entity("IfcBuildingStorey", Name="1층")

# 관계 설정
ifc.create_entity("IfcRelAggregates",
    RelatingObject=project, RelatedObjects=[site])
ifc.create_entity("IfcRelAggregates",
    RelatingObject=site, RelatedObjects=[building])
ifc.create_entity("IfcRelAggregates",
    RelatingObject=building, RelatedObjects=[storey])

# 파일 저장
ifc.write("output.ifc")
print("IFC 파일 생성 완료")
```

### 🏢 5.2 xBIM Toolkit

#### 🧭 개요

[xBIM](https://docs.xbim.net/)은 .NET 기반 오픈소스 BIM SDK로, IFC2x3/IFC4 파일의 읽기/쓰기/검증/시각화를 지원한다.

**주요 특징**:
- XbimXplorer: WPF 기반 IFC 뷰어
- XbimWebUI: WebGL/Three.js 기반 웹 뷰어
- C++ 지오메트리 엔진으로 IFC 데이터의 3D 형상 생성

#### 🗂️ C# 예제 - xBIM으로 IFC 파일 읽기

```csharp
using Xbim.Ifc;
using Xbim.Ifc4.Interfaces;
using System.Linq;

// IFC 파일 열기
using (var model = IfcStore.Open("building.ifc"))
{
    // 모든 벽 조회
    var walls = model.Instances.OfType<IIfcWall>().ToList();
    Console.WriteLine($"벽 개수: {walls.Count}");

    foreach (var wall in walls)
    {
        Console.WriteLine($"벽: {wall.Name} (GUID: {wall.GlobalId})");

        // 프로퍼티 셋 조회
        var psets = wall.IsDefinedBy
            .SelectMany(r => r.RelatingPropertyDefinition
                .PropertySetDefinitions)
            .OfType<IIfcPropertySet>();

        foreach (var pset in psets)
        {
            Console.WriteLine($"  [{pset.Name}]");
            foreach (var prop in pset.HasProperties.OfType<IIfcPropertySingleValue>())
            {
                Console.WriteLine($"    {prop.Name}: {prop.NominalValue}");
            }
        }
    }
}
```

### 🔹 5.3 Speckle

#### 🧭 개요

[Speckle](https://speckle.systems/)은 AEC 데이터 플랫폼(오픈소스)으로, "BIM의 GitHub"를 지향한다. Revit, Rhino, Grasshopper, Blender, QGIS 등 12개 이상의 도구 간 데이터를 버전 관리하며 공유할 수 있다.

**핵심 기능**: 버전 컨트롤, GraphQL API, 웹 3D 뷰어, 자동화 파이프라인(CI/CD), 웹훅

---

## ⚖️ 6. API 종합 비교

### ⚖️ 6.1 기능 비교표

| 항목 | Revit API | RhinoCommon | Dynamo | ArchiCAD API | IfcOpenShell |
|---|---|---|---|---|---|
| **주요 언어** | C# (.NET 8) | C#, Python | DesignScript, Python, C# | C++, Python | C++, Python |
| **API 유형** | 내장 .NET SDK | 내장 .NET SDK | 비주얼+텍스트 | C++ SDK + JSON RPC | 오픈소스 라이브러리 |
| **진입 장벽** | 중간 | 중간 | 낮음 | 높음 (C++), 낮음 (Python) | 낮음 |
| **실행 환경** | Revit 프로세스 내부 | Rhino 프로세스 내부 | Revit/Sandbox | ArchiCAD 내부(C++), 외부(Python) | 독립 실행 |
| **BIM 의미론** | 완전 지원 (벽, 문, 방 등) | 미지원 (순수 지오메트리) | Revit 통합 시 완전 지원 | 완전 지원 | IFC 스키마 기반 완전 지원 |
| **자유형 모델링** | 제한적 | 최고 수준 (NURBS) | 중간 (Revit 종속) | 제한적 | 읽기/쓰기만 |
| **커뮤니티 규모** | 매우 큼 | 큼 | 큼 | 작음 | 중간 |
| **문서화 수준** | 양호 (비공식 포함) | 우수 | 양호 | 보통 | 양호 |
| **크로스 플랫폼** | Windows만 | Windows, macOS | Windows만 | Windows, macOS | Windows, macOS, Linux |
| **라이선스** | 상용 (Revit 필요) | 상용 (Rhino 필요) | 무료 (Revit 필요) | 상용 (ArchiCAD 필요) | LGPL 오픈소스 |
| **.NET 버전** | .NET 8 (2025+) | .NET Core (Rhino 8) | .NET 10 (4.0+) | 해당 없음 | 해당 없음 |

### 💻 6.2 작업별 추천 API

| 작업 유형 | 1순위 추천 | 2순위 추천 | 비고 |
|---|---|---|---|
| **Revit 자동화 (단순)** | Dynamo | pyRevit | 비개발자도 접근 가능 |
| **Revit 자동화 (고급)** | Revit API (C#) | pyRevit | 성능과 안정성 우선 |
| **자유형 지오메트리** | RhinoCommon | Dynamo | NURBS 기반 설계 |
| **Revit + Rhino 통합** | Rhino.Inside.Revit | Speckle | 양방향 데이터 교환 |
| **IFC 데이터 처리** | IfcOpenShell | xBIM | 도구 독립적 |
| **파라메트릭 설계** | Grasshopper + RhinoCommon | Dynamo | 복잡한 형태 최적화 |
| **배치 작업** | Revit API + RevitBatchProcessor | pyRevit | 대량 파일 처리 |
| **웹 기반 BIM** | Speckle + IfcOpenShell | xBIM WebUI | 서버 사이드 처리 |
| **ArchiCAD 자동화** | ArchiCAD Python API | C++ Add-On | Python은 간단한 작업에 적합 |
| **다중 도구 연동** | Speckle | Rhino.Inside | 플랫폼 간 데이터 허브 |

### 🔹 6.3 상호 연동 가능성

#### 💻 Rhino.Inside.Revit

Rhino.Inside.Revit은 Revit 프로세스 내에서 Rhino와 Grasshopper를 실행하는 기술이다. Revit 2025/2026에서는 Rhino 8이 필수이며(.NET Core 8 호환), 300개 이상의 Revit 전용 Grasshopper 컴포넌트를 제공한다. Grasshopper의 Python/C# 스크립팅 컴포넌트에서 Revit API와 RhinoCommon을 동시에 사용할 수 있다.

- 공식 사이트: [rhino3d.com/inside/revit](https://www.rhino3d.com/inside/revit/1.0/reference/rir-api)
- 2026년 3월 온라인 워크숍 예정: [McNeel Europe](https://blog.rhino3d.com/2026/01/rhinoinsiderevit-online-workshop-march.html)

#### ▫️ Speckle을 통한 다중 도구 연동

Speckle은 Revit, Rhino, Grasshopper, ArchiCAD, Blender, QGIS 등 12개 이상의 도구에 커넥터를 제공하며, GraphQL API를 통해 커스텀 통합이 가능하다. 버전 관리, 변경 이력 추적, 자동화 파이프라인 기능으로 BIM 협업의 중심 허브 역할을 한다.

#### ▫️ IFC를 통한 범용 데이터 교환

IFC(Industry Foundation Classes)는 buildingSMART 표준으로, 모든 주요 BIM 도구에서 내보내기/가져오기를 지원한다. IfcOpenShell을 사용하면 특정 도구에 의존하지 않고 BIM 데이터를 처리할 수 있으며, 도구 간 데이터 무결성 검증이나 모델 비교에 활용된다.

```
[Revit] --IFC--> [IfcOpenShell] --분석/변환--> [ArchiCAD / 다른 BIM 도구]
   |                    |
   +----Speckle---------+------> [Rhino/Grasshopper]
   |                    |
   +--Rhino.Inside------+------> [웹 뷰어 / 데이터베이스]
```

---

## 🔗 7. 참고 문헌

### 💻 Revit API
- Revit API Docs (비공식, 검색 가능): [revitapidocs.com](https://www.revitapidocs.com/)
- Revit API 2026 What's New: [rvtdocs.com/2026/whatsnew](https://rvtdocs.com/2026/whatsnew)
- Autodesk 개발자 센터: [aps.autodesk.com/developer/overview/revit](https://aps.autodesk.com/developer/overview/revit)
- Revit API 개발자 가이드: [Autodesk Help](https://help.autodesk.com/view/RVT/2025/ENU/?guid=Revit_API_Revit_API_Developers_Guide_html)
- The Building Coder (Jeremy Tammik): [thebuildingcoder.typepad.com](https://thebuildingcoder.typepad.com/)
- pyRevit 공식 문서: [docs.pyrevitlabs.io](https://docs.pyrevitlabs.io/)
- pyRevit 2025 안내: [archilabs.ai/posts/pyrevit-2025](https://archilabs.ai/posts/pyrevit-2025)
- Revit API NuGet: [nuget.org/packages/Autodesk.Revit.SDK](https://www.nuget.org/packages/Autodesk.Revit.SDK)
- AYDrafting Revit 2026 플러그인 가이드: [aydrafting.com](https://www.aydrafting.com/case/starting-a-project-in-revit-2026-net8-api)
- Revit API 기초 아키텍처: [shiftasia.com](https://shiftasia.com/community/revit-api-basics-architecture-and-how-to-get-started-2/)

### 🔹 RhinoCommon / Grasshopper
- Rhino 개발자 문서: [developer.rhino3d.com](https://developer.rhino3d.com/)
- RhinoCommon API 레퍼런스: [developer.rhino3d.com/api](https://developer.rhino3d.com/api/)
- C# Essentials for Grasshopper: [developer.rhino3d.com/guides/grasshopper/csharp-essentials](https://developer.rhino3d.com/guides/grasshopper/csharp-essentials/)
- RhinoCommon 지오메트리 가이드: [developer.rhino3d.com/guides/grasshopper/csharp-essentials/3-rhinocommon-geometry](https://developer.rhino3d.com/guides/grasshopper/csharp-essentials/3-rhinocommon-geometry/)
- Rhino Developer Samples: [github.com/mcneel/rhino-developer-samples](https://github.com/mcneel/rhino-developer-samples)
- Rhino Python 가이드: [developer.rhino3d.com/guides/rhinopython](https://developer.rhino3d.com/guides/rhinopython/what-is-rhinopython/)
- Rhino.Inside.Revit: [rhino3d.com/inside/revit](https://www.rhino3d.com/inside/revit/1.0/reference/rir-api)

### 🔹 Dynamo
- Dynamo BIM 공식: [dynamobim.org](https://dynamobim.org/)
- Dynamo Core 4.0 릴리스: [dynamobim.org/dynamo-core-4-0-release](https://dynamobim.org/dynamo-core-4-0-release/)
- Dynamo Primer: [primer.dynamobim.org](https://primer.dynamobim.org/)
- Dynamo Python Primer: [dynamopythonprimer.gitbook.io](https://dynamopythonprimer.gitbook.io/dynamo-python-primer/)
- PythonNet3 안내: [dynamobim.org/pythonnet3](https://dynamobim.org/pythonnet3-a-new-dynamo-python-to-fix-everything/)
- Zero Touch 개발 가이드: [github.com/DynamoDS/Dynamo/wiki/Zero-Touch-Plugin-Development](https://github.com/DynamoDS/Dynamo/wiki/Zero-Touch-Plugin-Development)
- Dynamo vs Revit API 비교: [digitalbbq.au](https://digitalbbq.au/index.php/2025/07/31/dynamo-vs-revit-api-choosing-the-right-tool-for-your-bim-automation-needs/)

### 💻 ArchiCAD API
- ArchiCAD API 포털: [archicadapi.graphisoft.com](https://archicadapi.graphisoft.com/)
- ArchiCAD 29 C++ API: [graphisoft.github.io/archicad-api-devkit](https://graphisoft.github.io/archicad-api-devkit/)
- ArchiCAD Python 패키지: [pypi.org/project/archicad](https://pypi.org/project/archicad/)
- ArchiCAD API DevKit GitHub: [github.com/GRAPHISOFT/archicad-api-devkit](https://github.com/GRAPHISOFT/archicad-api-devkit)
- ArchiCAD Python 연결 가이드: [archicadapi.graphisoft.com/getting-started-with-archicad-python-connection](https://archicadapi.graphisoft.com/getting-started-with-archicad-python-connection)

### 🔹 IFC 및 오픈소스 도구
- IfcOpenShell 공식: [ifcopenshell.org](https://ifcopenshell.org/)
- IfcOpenShell GitHub: [github.com/IfcOpenShell/IfcOpenShell](https://github.com/IfcOpenShell/IfcOpenShell)
- IfcOpenShell Python 문서: [docs.ifcopenshell.org](https://docs.ifcopenshell.org/ifcopenshell-python/code_examples.html)
- IfcOpenShell 코드 예제: [wiki.osarch.org](https://wiki.osarch.org/index.php?title=IfcOpenShell_code_examples)
- IfcOpenShell Academy: [academy.ifcopenshell.org](https://academy.ifcopenshell.org/)
- xBIM Toolkit: [docs.xbim.net](https://docs.xbim.net/)
- Speckle: [speckle.systems](https://speckle.systems/)
- Speckle GitHub: [github.com/specklesystems](https://github.com/specklesystems)

### ⚖️ 비교 및 일반
- Revit 자동화 비교 (Macros vs Dynamo vs Python vs AI): [archilabs.ai](https://archilabs.ai/posts/revit-automation-macros-vs-dynamo-vs-python-vs-ai)
- Dynamo vs Revit API 비교: [elogictech.com](https://www.elogictech.com/blog/dynamo-vs-revit-api/)
- 오픈소스 BIM 도구 2025: [clovetech.com](https://clovetech.com/blog/bim-software-tools/)

---

> 💡 **면책 조항**: 이 문서는 2026년 2월 기준 공개된 정보를 바탕으로 작성되었다. API 버전과 기능은 각 벤더의 업데이트에 따라 변경될 수 있으므로, 실무 적용 시 반드시 해당 시점의 공식 문서를 확인해야 한다. 코드 예제는 개념 설명 목적이며, 실제 프로젝트에서는 에러 처리, 성능 최적화 등을 추가로 고려해야 한다.
