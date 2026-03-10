---
layout: post
title: 260202 unity position transforms
comments: true
tags:
- unity
- performance
- transform
- debugging
- ai
- research
- guide
---
# 🛠️ Unity Transform Position 완전 가이드

Unity에서 게임 오브젝트의 위치를 다루는 것은 게임 개발의 가장 기본적이면서도 중요한 개념입니다. 이 문서에서는 Unity의 세 가지 주요 Position 개념인 **localPosition**, **worldPosition (position)**, 그리고 **viewPosition (Screen/Viewport Space)**에 대해 상세히 알아보고, 실제 사용 예제를 통해 이들 간의 변환 방법을 살펴보겠습니다.

## 📚 목차

1. [Position의 세 가지 개념](#position의-세-가지-개념)
2. [Local Space와 World Space](#local-space와-world-space)
3. [Transform 변환 메서드](#transform-변환-메서드)
4. [Camera Space와 Screen Space](#camera-space와-screen-space)
5. [실전 예제: 계층 구조에서의 좌표 변환](#실전-예제-계층-구조에서의-좌표-변환)
6. [성능 고려사항](#성능-고려사항)
7. [참고 자료](#참고-자료)

---

## 📌 Position의 세 가지 개념

Unity에서 위치를 표현하는 방법은 크게 세 가지가 있습니다.

### 🔹 1. World Position (전역 좌표)

**World Position**은 Unity Scene의 절대적인 원점 (0, 0, 0)을 기준으로 한 좌표입니다. Scene 뷰에서 보이는 월드 그리드가 바로 이 World Space를 나타냅니다.

```csharp
// World Position 접근
Vector3 worldPos = transform.position;

// World Position 설정
transform.position = new Vector3(10f, 5f, 0f);
```

**특징:**
- Scene의 절대 좌표계
- 부모 오브젝트와 무관하게 항상 동일한 기준점
- 다른 오브젝트와의 절대적 거리 계산에 사용

### 🔹 2. Local Position (지역 좌표)

**Local Position**은 부모 Transform을 기준으로 한 상대적 좌표입니다. 부모가 없는 경우, Local Position과 World Position은 동일합니다.

```csharp
// Local Position 접근
Vector3 localPos = transform.localPosition;

// Local Position 설정
transform.localPosition = new Vector3(5f, 2f, 0f);
```

**특징:**
- 부모 Transform 기준의 상대 좌표
- 부모가 이동/회전/크기 변경되면 자식의 World Position도 함께 변경됨
- 부모-자식 관계에서 상대적 배치에 사용

### 🔹 3. View Position (화면 좌표)

**View Position**은 카메라를 기준으로 한 좌표로, Screen Space와 Viewport Space로 나뉩니다.

**Screen Space** (픽셀 단위):
```csharp
// World → Screen
Vector3 screenPos = Camera.main.WorldToScreenPoint(worldPosition);

// Screen → World
Vector3 worldPos = Camera.main.ScreenToWorldPoint(screenPosition);
```

**Viewport Space** (정규화된 0~1 범위):
```csharp
// World → Viewport
Vector3 viewportPos = Camera.main.WorldToViewportPoint(worldPosition);

// Viewport → World
Vector3 worldPos = Camera.main.ViewportToWorldPoint(viewportPosition);
```

**특징:**
- Screen Space: 해상도에 따라 변하는 픽셀 좌표 (0, 0) ~ (Screen.width, Screen.height)
- Viewport Space: 해상도 독립적인 정규화 좌표 (0, 0) ~ (1, 1)
- UI 배치, 마우스 입력 처리, 화면 효과에 주로 사용

---

## 📌 Local Space와 World Space

Unity의 계층 구조에서 Local Space와 World Space의 관계를 시각적으로 이해해봅시다.

![Unity World Space와 Local Space](260202_unity_position_transforms/img_1.svg)

### 🏗️ 계층 구조의 이해

게임 오브젝트가 부모-자식 관계로 연결되어 있을 때:

```
Aobj (Parent)
 └─ Bobj (Child)
     └─ Cobj (GrandChild)
```

각 오브젝트는 두 가지 좌표를 가집니다:
- **World Position**: Scene의 (0, 0, 0)을 기준으로 한 절대 좌표
- **Local Position**: 부모 오브젝트를 기준으로 한 상대 좌표

### 🔹 좌표 계산 규칙

자식 오브젝트의 World Position은 다음과 같이 계산됩니다:

```
Child World Position = Parent World Position + (Parent Rotation * Parent Scale * Child Local Position)
```

이는 부모의 Transform이 자식에게 다음과 같이 영향을 미친다는 의미입니다:
1. **Translation (이동)**: 부모가 이동하면 자식도 함께 이동
2. **Rotation (회전)**: 부모가 회전하면 자식의 Local Position이 회전축을 중심으로 공전
3. **Scale (크기)**: 부모의 Scale이 2배가 되면, 자식의 Local Position도 2배 멀어짐

### 🧪 코드 예제

```csharp
// 계층 구조 생성
GameObject aobj = new GameObject("Aobj");
GameObject bobj = new GameObject("Bobj");
GameObject cobj = new GameObject("Cobj");

// 부모-자식 관계 설정
bobj.transform.SetParent(aobj.transform);
cobj.transform.SetParent(bobj.transform);

// Aobj 위치 설정 (World 기준)
aobj.transform.position = new Vector3(50f, -50f, 0f);

// Bobj 위치 설정 (Aobj 기준 Local)
bobj.transform.localPosition = new Vector3(100f, -50f, 0f);
// Bobj의 World Position은 자동으로 (150, -100, 0)이 됨

// Cobj 위치 설정 (Bobj 기준 Local)
cobj.transform.localPosition = new Vector3(70f, -20f, 0f);
// Cobj의 World Position은 자동으로 (220, -120, 0)이 됨

// 확인
Debug.Log($"Bobj World Position: {bobj.transform.position}");
Debug.Log($"Cobj World Position: {cobj.transform.position}");
```

---

## 📌 Transform 변환 메서드

Unity는 Local Space와 World Space 간 변환을 위한 여러 메서드를 제공합니다.

![Unity Transform 변환 메서드](260202_unity_position_transforms/img_2.svg)

### 🔹 1. TransformPoint / InverseTransformPoint

**위치(Position)를 변환**할 때 사용합니다. Translation, Rotation, Scale이 모두 적용됩니다.

```csharp
// Local → World
Vector3 localPoint = new Vector3(1f, 0f, 0f);
Vector3 worldPoint = transform.TransformPoint(localPoint);
Debug.Log($"LOCAL_TO_WORLD localPoint:{localPoint}, worldPoint:{worldPoint}");

// World → Local
Vector3 worldPoint = new Vector3(10f, 5f, 0f);
Vector3 localPoint = transform.InverseTransformPoint(worldPoint);
Debug.Log($"WORLD_TO_LOCAL worldPoint:{worldPoint}, localPoint:{localPoint}");
```

**사용 사례:**
- 자식 오브젝트의 Local Position을 World Position으로 변환
- 월드 좌표를 특정 오브젝트 기준 상대 좌표로 변환
- 충돌 지점을 오브젝트 로컬 좌표로 변환

### 🔹 2. TransformDirection / InverseTransformDirection

**방향(Direction) 벡터를 변환**할 때 사용합니다. **Rotation만 적용**되고, Scale과 Translation은 무시됩니다.

```csharp
// Local → World
Vector3 localDirection = Vector3.forward; // (0, 0, 1)
Vector3 worldDirection = transform.TransformDirection(localDirection);
Debug.Log($"LOCAL_DIR_TO_WORLD localDir:{localDirection}, worldDir:{worldDirection}");

// World → Local
Vector3 worldDirection = Vector3.up; // (0, 1, 0)
Vector3 localDirection = transform.InverseTransformDirection(worldDirection);
Debug.Log($"WORLD_DIR_TO_LOCAL worldDir:{worldDirection}, localDir:{localDirection}");
```

**사용 사례:**
- 오브젝트의 전방 방향(forward) 구하기
- 이동 방향을 오브젝트 로컬 축 기준으로 변환
- 카메라가 바라보는 방향 계산

### 🔹 3. TransformVector / InverseTransformVector

**벡터(Vector)를 변환**할 때 사용합니다. **Rotation과 Scale이 적용**되지만, Translation은 무시됩니다.

```csharp
// Local → World
Vector3 localVector = new Vector3(2f, 1f, 0f);
Vector3 worldVector = transform.TransformVector(localVector);
Debug.Log($"LOCAL_VEC_TO_WORLD localVec:{localVector}, worldVec:{worldVector}");

// World → Local
Vector3 worldVector = new Vector3(5f, 3f, 0f);
Vector3 localVector = transform.InverseTransformVector(worldVector);
Debug.Log($"WORLD_VEC_TO_LOCAL worldVec:{worldVector}, localVec:{localVector}");
```

**사용 사례:**
- Scale이 적용된 크기를 고려해야 하는 벡터 변환
- 물리 계산에서 힘의 방향과 크기 변환
- 커스텀 바운딩 박스 계산

### ⚖️ 메서드 비교표

| 메서드 | Translation | Rotation | Scale | 주 용도 |
|--------|-------------|----------|-------|---------|
| TransformPoint | ✓ | ✓ | ✓ | 위치 변환 |
| TransformDirection | ✗ | ✓ | ✗ | 방향 변환 |
| TransformVector | ✗ | ✓ | ✓ | 크기가 있는 벡터 변환 |

### 🧪 실전 예제: 각 메서드의 차이

```csharp
// 테스트용 Transform 생성
GameObject parent = new GameObject("Parent");
parent.transform.position = new Vector3(10f, 5f, 0f);    // Translation
parent.transform.rotation = Quaternion.Euler(0f, 45f, 0f); // Rotation
parent.transform.localScale = new Vector3(2f, 2f, 2f);    // Scale

Vector3 testInput = new Vector3(1f, 0f, 0f);

// 1. TransformPoint: 모든 변환 적용
Vector3 resultPoint = parent.transform.TransformPoint(testInput);
// 결과: Translation + Rotation + Scale 모두 적용됨

// 2. TransformDirection: Rotation만 적용
Vector3 resultDirection = parent.transform.TransformDirection(testInput);
// 결과: Rotation만 적용, 크기는 1 유지

// 3. TransformVector: Rotation + Scale 적용
Vector3 resultVector = parent.transform.TransformVector(testInput);
// 결과: Rotation + Scale 적용, Translation 무시

Debug.Log($"TRANSFORM_COMPARISON input:{testInput}");
Debug.Log($"  TransformPoint:{resultPoint}");
Debug.Log($"  TransformDirection:{resultDirection}");
Debug.Log($"  TransformVector:{resultVector}");
```

---

## 📌 Camera Space와 Screen Space

카메라와 관련된 좌표 변환은 UI, 마우스 입력, 화면 효과 등에서 필수적입니다.

![Unity Camera Space 변환](260202_unity_position_transforms/img_3.svg)

### 🔹 Screen Space (화면 픽셀 좌표)

Screen Space는 **픽셀 단위**의 좌표계입니다.

**좌표 범위:**
- X: 0 (왼쪽) ~ Screen.width (오른쪽)
- Y: 0 (아래) ~ Screen.height (위)
- Z: 카메라로부터의 거리

**주요 메서드:**

```csharp
// World → Screen
Vector3 worldPos = targetObject.transform.position;
Vector3 screenPos = Camera.main.WorldToScreenPoint(worldPos);
Debug.Log($"WORLD_TO_SCREEN worldPos:{worldPos}, screenPos:{screenPos}");

// Screen → World
Vector3 mouseScreenPos = Input.mousePosition;
mouseScreenPos.z = 10f; // 카메라로부터의 거리 설정 필수!
Vector3 worldPos = Camera.main.ScreenToWorldPoint(mouseScreenPos);
Debug.Log($"SCREEN_TO_WORLD mouseScreenPos:{mouseScreenPos}, worldPos:{worldPos}");
```

**사용 사례:**

```csharp
// 예제 1: 마우스 클릭 위치에 오브젝트 생성
void SpawnObjectAtMousePosition()
{
    // 마우스 위치 가져오기
    Vector3 mousePos = Input.mousePosition;

    // Z축 거리 설정 (카메라로부터 10 유닛 떨어진 곳)
    mousePos.z = 10f;

    // Screen → World 변환
    Vector3 worldPos = Camera.main.ScreenToWorldPoint(mousePos);

    // 오브젝트 생성
    Instantiate(prefab, worldPos, Quaternion.identity);
    Debug.Log($"SPAWN_AT_MOUSE worldPos:{worldPos}");
}

// 예제 2: 오브젝트가 화면 밖으로 나갔는지 확인
bool IsObjectOffScreen(GameObject obj)
{
    Vector3 screenPos = Camera.main.WorldToScreenPoint(obj.transform.position);

    // 화면 밖으로 나간 경우
    var isOffScreen = screenPos.x < 0 || screenPos.x > Screen.width ||
           screenPos.y < 0 || screenPos.y > Screen.height ||
           screenPos.z < 0; // 카메라 뒤에 있는 경우

    if (isOffScreen)
    {
        Debug.Log($"OBJECT_OFF_SCREEN objName:{obj.name}, screenPos:{screenPos}");
    }

    return isOffScreen;
}
```

### 🔹 Viewport Space (정규화 좌표)

Viewport Space는 **0~1 범위로 정규화**된 좌표계로, 해상도에 독립적입니다.

**좌표 범위:**
- X: 0 (왼쪽) ~ 1 (오른쪽)
- Y: 0 (아래) ~ 1 (위)
- Z: 카메라로부터의 거리
- 화면 중앙: (0.5, 0.5)

**주요 메서드:**

```csharp
// World → Viewport
Vector3 worldPos = targetObject.transform.position;
Vector3 viewportPos = Camera.main.WorldToViewportPoint(worldPos);
Debug.Log($"WORLD_TO_VIEWPORT worldPos:{worldPos}, viewportPos:{viewportPos}");

// Viewport → World
Vector3 viewportPos = new Vector3(0.5f, 0.5f, 10f); // 화면 중앙, 거리 10
Vector3 worldPos = Camera.main.ViewportToWorldPoint(viewportPos);
Debug.Log($"VIEWPORT_TO_WORLD viewportPos:{viewportPos}, worldPos:{worldPos}");

// Screen ↔ Viewport 변환
Vector3 screenPos = Input.mousePosition;
Vector3 viewportPos = Camera.main.ScreenToViewportPoint(screenPos);
Vector3 backToScreen = Camera.main.ViewportToScreenPoint(viewportPos);
Debug.Log($"SCREEN_VIEWPORT_CONVERSION screenPos:{screenPos}, viewportPos:{viewportPos}");
```

**사용 사례:**

```csharp
// 예제 1: 화면의 특정 비율 위치에 UI 배치
void PlaceUIAtScreenRatio(float ratioX, float ratioY)
{
    // Viewport 좌표 (0~1 범위)
    Vector3 viewportPos = new Vector3(ratioX, ratioY, 10f);

    // World 좌표로 변환
    Vector3 worldPos = Camera.main.ViewportToWorldPoint(viewportPos);

    // UI 오브젝트 배치
    uiObject.transform.position = worldPos;
    Debug.Log($"PLACE_UI_AT_RATIO ratio:({ratioX},{ratioY}), worldPos:{worldPos}");
}

// 예제 2: 오브젝트가 화면 중앙 근처에 있는지 확인
bool IsObjectNearCenter(GameObject obj, float threshold = 0.2f)
{
    Vector3 viewportPos = Camera.main.WorldToViewportPoint(obj.transform.position);

    // 중앙 (0.5, 0.5)으로부터의 거리
    var distanceFromCenter = Vector2.Distance(
        new Vector2(viewportPos.x, viewportPos.y),
        new Vector2(0.5f, 0.5f)
    );

    var isNearCenter = distanceFromCenter < threshold;

    if (isNearCenter)
    {
        Debug.Log($"OBJECT_NEAR_CENTER objName:{obj.name}, viewportPos:{viewportPos}, distance:{distanceFromCenter}");
    }

    return isNearCenter;
}

// 예제 3: 화면 가장자리 여백 확인
bool IsInSafeArea(GameObject obj, float margin = 0.1f)
{
    Vector3 viewportPos = Camera.main.WorldToViewportPoint(obj.transform.position);

    // Safe Area: 화면 가장자리로부터 margin만큼 안쪽
    var isInSafeArea = viewportPos.x > margin && viewportPos.x < (1f - margin) &&
           viewportPos.y > margin && viewportPos.y < (1f - margin) &&
           viewportPos.z > 0;

    if (!isInSafeArea)
    {
        Debug.Log($"OBJECT_OUT_OF_SAFE_AREA objName:{obj.name}, viewportPos:{viewportPos}");
    }

    return isInSafeArea;
}
```

### ⚖️ Screen Space vs Viewport Space 비교

| 특성 | Screen Space | Viewport Space |
|------|--------------|----------------|
| 좌표 범위 | (0, 0) ~ (width, height) | (0, 0) ~ (1, 1) |
| 단위 | 픽셀 | 정규화 (비율) |
| 해상도 의존성 | 해상도에 따라 변함 | 해상도 독립적 |
| 주 사용처 | 마우스 입력, UI 픽셀 위치 | 상대적 화면 배치, 비율 계산 |

---

## 🏗️ 실전 예제: 계층 구조에서의 좌표 변환

이제 실제 시나리오에서 좌표 변환을 어떻게 사용하는지 알아봅시다.

### 🛠️ 시나리오 설정

```
Aobj (부모)
 └─ Bobj (자식)
     └─ Cobj (손자)
```

```csharp
// 계층 구조 생성
GameObject aobj = new GameObject("Aobj");
GameObject bobj = new GameObject("Bobj");
GameObject cobj = new GameObject("Cobj");

bobj.transform.SetParent(aobj.transform);
cobj.transform.SetParent(bobj.transform);

// 위치 설정
aobj.transform.position = new Vector3(10f, 5f, 0f);
bobj.transform.localPosition = new Vector3(3f, 2f, 0f);
cobj.transform.localPosition = new Vector3(1f, 1f, 0f);

// 회전 및 스케일 설정 (변환 테스트용)
aobj.transform.rotation = Quaternion.Euler(0f, 45f, 0f);
bobj.transform.localScale = new Vector3(2f, 2f, 2f);

Debug.Log($"HIERARCHY_SETUP aobj.position:{aobj.transform.position}");
Debug.Log($"HIERARCHY_SETUP bobj.localPosition:{bobj.transform.localPosition}, bobj.position:{bobj.transform.position}");
Debug.Log($"HIERARCHY_SETUP cobj.localPosition:{cobj.transform.localPosition}, cobj.position:{cobj.transform.position}");
```

### 🧪 예제 1: Cobj의 Local Position을 Bobj 기준으로 얻기

Cobj는 Bobj의 직속 자식이므로, 이미 Bobj 기준 Local Position을 가지고 있습니다.

```csharp
// Cobj의 Bobj 기준 Local Position
Vector3 cobjLocalToBobj = cobj.transform.localPosition;
Debug.Log($"COBJ_LOCAL_TO_BOBJ localPosition:{cobjLocalToBobj}");
// 결과: (1, 1, 0)
```

### 🧪 예제 2: Cobj의 Local Position을 Aobj 기준으로 변환

Cobj는 Aobj의 손자이므로, Aobj 기준 상대 좌표를 구하려면 변환이 필요합니다.

```csharp
// 방법 1: InverseTransformPoint 사용
Vector3 cobjWorldPos = cobj.transform.position;
Vector3 cobjLocalToAobj = aobj.transform.InverseTransformPoint(cobjWorldPos);
Debug.Log($"COBJ_LOCAL_TO_AOBJ method1 worldPos:{cobjWorldPos}, localToAobj:{cobjLocalToAobj}");

// 방법 2: 중간 단계 계산 (이해를 돕기 위한 상세 버전)
// Cobj World = Aobj World + Aobj.TransformPoint(Bobj Local + Bobj.TransformPoint(Cobj Local))
// 따라서 역산하면:
Vector3 cobjLocalToAobj2 = aobj.transform.InverseTransformPoint(
    bobj.transform.TransformPoint(cobj.transform.localPosition)
);
Debug.Log($"COBJ_LOCAL_TO_AOBJ method2 localToAobj:{cobjLocalToAobj2}");
```

**결과 해석:**
- Cobj의 World Position에서 Aobj의 Transform을 역으로 적용
- Aobj의 회전, 스케일, 위치가 모두 역산됨
- Bobj의 Scale이 Cobj의 상대 좌표에 영향을 미침

### 🧪 예제 3: Cobj의 Local Position을 World 기준으로 변환

Cobj의 World Position은 `transform.position`으로 바로 얻을 수 있지만, Local Position에서 수동으로 계산하는 방법도 있습니다.

```csharp
// 방법 1: 직접 접근
Vector3 cobjWorldPos = cobj.transform.position;
Debug.Log($"COBJ_TO_WORLD method1 worldPos:{cobjWorldPos}");

// 방법 2: TransformPoint 사용 (단계별 계산)
// Cobj Local → Bobj World
Vector3 cobjInBobj = bobj.transform.TransformPoint(cobj.transform.localPosition);
Debug.Log($"COBJ_TO_WORLD method2_step1 cobjInBobj:{cobjInBobj}");

// 실제로는 부모 체인을 따라 올라가며 자동 계산됨
// 최종 결과는 동일
Debug.Log($"COBJ_TO_WORLD method2_final worldPos:{cobjWorldPos}");

// 방법 3: 수동 계산 (이론적 이해용)
// Cobj World = Aobj.TransformPoint(Bobj.localPosition + Bobj.TransformPoint(Cobj.localPosition))
// 하지만 실제로는 transform.position 사용 권장
```

### 🧪 예제 4: 역변환 - World Position을 각 부모 기준으로 변환

월드 좌표의 한 점을 각 오브젝트 기준 Local Position으로 변환합니다.

```csharp
// 테스트할 World Position
Vector3 testWorldPos = new Vector3(20f, 10f, 5f);
Debug.Log($"WORLD_TO_LOCAL_CONVERSIONS testWorldPos:{testWorldPos}");

// 1. World → Aobj Local
Vector3 localToAobj = aobj.transform.InverseTransformPoint(testWorldPos);
Debug.Log($"WORLD_TO_LOCAL world_to_aobj:{localToAobj}");

// 2. World → Bobj Local
Vector3 localToBobj = bobj.transform.InverseTransformPoint(testWorldPos);
Debug.Log($"WORLD_TO_LOCAL world_to_bobj:{localToBobj}");

// 3. World → Cobj Local
Vector3 localToCobj = cobj.transform.InverseTransformPoint(testWorldPos);
Debug.Log($"WORLD_TO_LOCAL world_to_cobj:{localToCobj}");

// 검증: Local → World로 다시 변환하면 원래 값이 나와야 함
Vector3 verifyFromAobj = aobj.transform.TransformPoint(localToAobj);
Vector3 verifyFromBobj = bobj.transform.TransformPoint(localToBobj);
Vector3 verifyFromCobj = cobj.transform.TransformPoint(localToCobj);

Debug.Log($"VERIFICATION fromAobj:{verifyFromAobj}");
Debug.Log($"VERIFICATION fromBobj:{verifyFromBobj}");
Debug.Log($"VERIFICATION fromCobj:{verifyFromCobj}");
// 모든 결과가 testWorldPos와 동일해야 함
```

### 🧪 예제 5: 부모 간 Local Position 변환

Cobj의 Local Position을 Bobj 기준에서 Aobj 기준으로 직접 변환합니다.

```csharp
// Cobj의 Bobj 기준 Local Position
Vector3 cobjLocalToBobj = cobj.transform.localPosition;

// Bobj 기준 → World → Aobj 기준
Vector3 cobjInWorld = bobj.transform.TransformPoint(cobjLocalToBobj);
Vector3 cobjLocalToAobj = aobj.transform.InverseTransformPoint(cobjInWorld);

Debug.Log($"PARENT_CONVERSION bobj_local:{cobjLocalToBobj}");
Debug.Log($"PARENT_CONVERSION world:{cobjInWorld}");
Debug.Log($"PARENT_CONVERSION aobj_local:{cobjLocalToAobj}");

// 또는 한 줄로
Vector3 cobjLocalToAobjDirect = aobj.transform.InverseTransformPoint(
    bobj.transform.TransformPoint(cobjLocalToBobj)
);
Debug.Log($"PARENT_CONVERSION direct:{cobjLocalToAobjDirect}");
```

### 🧪 예제 6: 실용적인 사용 사례 - 캐릭터 무기 장착

```csharp
// 캐릭터 손에 무기를 장착하는 예제
public class WeaponAttachment : MonoBehaviour
{
    public Transform characterRoot;    // Aobj: 캐릭터 루트
    public Transform rightHand;        // Bobj: 오른손 본
    public GameObject weaponPrefab;    // Cobj: 장착할 무기

    private GameObject currentWeapon;

    // 무기를 손에 장착
    public void AttachWeapon()
    {
        // 무기 인스턴스 생성
        currentWeapon = Instantiate(weaponPrefab);

        // 부모 설정 (손에 부착)
        currentWeapon.transform.SetParent(rightHand);

        // 손 기준 Local Position 설정 (미리 정의된 장착 위치)
        currentWeapon.transform.localPosition = new Vector3(0f, 0.1f, 0f);
        currentWeapon.transform.localRotation = Quaternion.Euler(0f, 90f, 0f);

        Debug.Log($"WEAPON_ATTACHED name:{currentWeapon.name}, localPos:{currentWeapon.transform.localPosition}");
    }

    // 무기의 끝부분 World Position 구하기 (검의 끝 등)
    public Vector3 GetWeaponTipWorldPosition()
    {
        if (currentWeapon == null) return Vector3.zero;

        // 무기 Local Space에서 끝부분 좌표 (예: 검 끝)
        Vector3 tipLocalPos = new Vector3(0f, 0f, 1f);

        // Local → World 변환
        var tipWorldPos = currentWeapon.transform.TransformPoint(tipLocalPos);

        Debug.Log($"WEAPON_TIP_POS localPos:{tipLocalPos}, worldPos:{tipWorldPos}");

        return tipWorldPos;
    }

    // 특정 World Position이 무기의 타격 범위 내인지 확인
    public bool IsInWeaponRange(Vector3 targetWorldPos, float range = 1.5f)
    {
        Vector3 weaponTip = GetWeaponTipWorldPosition();

        var distance = Vector3.Distance(weaponTip, targetWorldPos);
        var isInRange = distance < range;

        Debug.Log($"WEAPON_RANGE_CHECK targetPos:{targetWorldPos}, distance:{distance}, inRange:{isInRange}");

        return isInRange;
    }

    // 타격 지점을 캐릭터 기준 좌표로 변환 (UI 표시 등에 사용)
    public Vector3 ConvertHitPointToCharacterSpace(Vector3 hitWorldPos)
    {
        // World → Character Root Local
        var hitLocalToCharacter = characterRoot.InverseTransformPoint(hitWorldPos);

        Debug.Log($"HIT_POINT_CONVERSION worldPos:{hitWorldPos}, characterLocal:{hitLocalToCharacter}");

        // "앞/뒤", "좌/우" 판단 가능
        if (hitLocalToCharacter.z > 0)
        {
            Debug.Log($"HIT_FROM_FRONT");
        }
        else
        {
            Debug.Log($"HIT_FROM_BACK");
        }

        return hitLocalToCharacter;
    }
}
```

### 🏗️ 예제 7: Screen Space를 계층 구조에 적용

마우스 클릭 위치를 계층 구조의 특정 오브젝트 Local 좌표로 변환합니다.

```csharp
// 마우스 클릭 위치를 Bobj의 Local 좌표로 변환
void HandleMouseClick()
{
    if (Input.GetMouseButtonDown(0))
    {
        // Screen Space (마우스 위치)
        Vector3 mouseScreenPos = Input.mousePosition;
        mouseScreenPos.z = 10f; // 카메라로부터의 거리

        // Screen → World
        Vector3 mouseWorldPos = Camera.main.ScreenToWorldPoint(mouseScreenPos);

        // World → Bobj Local
        Vector3 mouseLocalToBobj = bobj.transform.InverseTransformPoint(mouseWorldPos);

        Debug.Log($"MOUSE_CLICK screenPos:{mouseScreenPos}");
        Debug.Log($"MOUSE_CLICK worldPos:{mouseWorldPos}");
        Debug.Log($"MOUSE_CLICK bobjLocal:{mouseLocalToBobj}");

        // Bobj 기준으로 오브젝트 생성
        GameObject marker = GameObject.CreatePrimitive(PrimitiveType.Sphere);
        marker.transform.SetParent(bobj.transform);
        marker.transform.localPosition = mouseLocalToBobj;
        marker.transform.localScale = Vector3.one * 0.2f;
    }
}
```

---

## ⚡ 성능 고려사항

Transform 연산은 자주 사용되므로 성능에 주의해야 합니다.

### ⚡ 1. Position vs LocalPosition 성능

```csharp
// LocalPosition이 더 빠름
// Position 수정 시 자식 오브젝트들에게 업데이트 전파됨
transform.localPosition = newPos;  // 빠름
transform.position = newPos;       // 상대적으로 느림 (자식 업데이트 필요)
```

**권장 사항:**
- 부모-자식 관계가 있을 때는 가능한 `localPosition` 사용
- 매 프레임 업데이트가 필요한 경우 더욱 중요
- 자식이 없는 오브젝트는 성능 차이가 미미함

### 🔹 2. 변환 메서드 캐싱

```csharp
// 나쁜 예: 매번 변환 계산
void Update()
{
    for (var i = 0; i < 100; i++)
    {
        // 매 프레임 100번 변환 계산
        Vector3 worldPos = transform.TransformPoint(localPoints[i]);
        ProcessPoint(worldPos);
    }
}

// 좋은 예: 필요할 때만 계산
void Update()
{
    // Transform이 변경되었을 때만 재계산
    if (transform.hasChanged)
    {
        for (var i = 0; i < 100; i++)
        {
            worldPoints[i] = transform.TransformPoint(localPoints[i]);
        }
        transform.hasChanged = false;
    }

    // 캐시된 값 사용
    for (var i = 0; i < 100; i++)
    {
        ProcessPoint(worldPoints[i]);
    }
}
```

### 🚀 3. 카메라 변환 최적화

```csharp
// 나쁜 예: 매번 Camera.main 호출
void Update()
{
    for (var i = 0; i < enemies.Length; i++)
    {
        // Camera.main은 FindObjectOfType을 내부적으로 호출
        Vector3 screenPos = Camera.main.WorldToScreenPoint(enemies[i].position);
    }
}

// 좋은 예: Camera 참조 캐싱
private Camera mainCamera;

void Start()
{
    mainCamera = Camera.main;
}

void Update()
{
    for (var i = 0; i < enemies.Length; i++)
    {
        // 캐시된 참조 사용
        Vector3 screenPos = mainCamera.WorldToScreenPoint(enemies[i].position);
    }
}
```

### 🚀 4. 벡터 계산 최적화

```csharp
// 거리 비교 시 제곱근 연산 피하기
// 나쁜 예
var distance = Vector3.Distance(a, b);
if (distance < 10f)
{
    // 처리
}

// 좋은 예 (제곱근 연산 불필요)
var sqrDistance = (a - b).sqrMagnitude;
if (sqrDistance < 100f)  // 10^2 = 100
{
    // 처리
}
```

### 🔹 5. Transform 접근 최소화

```csharp
// 나쁜 예: 반복적인 transform 접근
void Update()
{
    transform.position += Vector3.forward * speed * Time.deltaTime;
    transform.rotation *= Quaternion.Euler(0f, rotSpeed * Time.deltaTime, 0f);
    transform.localScale = Vector3.one * scale;
}

// 좋은 예: 지역 변수로 캐싱
void Update()
{
    var t = transform;
    t.position += Vector3.forward * speed * Time.deltaTime;
    t.rotation *= Quaternion.Euler(0f, rotSpeed * Time.deltaTime, 0f);
    t.localScale = Vector3.one * scale;
}

// 더 좋은 예: 한 번에 처리
void Update()
{
    var t = transform;
    var newPos = t.position + Vector3.forward * speed * Time.deltaTime;
    var newRot = t.rotation * Quaternion.Euler(0f, rotSpeed * Time.deltaTime, 0f);
    var newScale = Vector3.one * scale;

    t.SetPositionAndRotation(newPos, newRot);
    t.localScale = newScale;
}
```

### ⚡ 성능 팁 요약

| 항목 | 권장 사항 |
|------|-----------|
| Position 수정 | 부모-자식 관계가 있으면 `localPosition` 사용 |
| 변환 결과 | 자주 사용되면 캐싱 |
| Camera 참조 | `Camera.main` 반복 호출 피하기 |
| 거리 비교 | `sqrMagnitude` 사용 (제곱근 연산 회피) |
| Transform 접근 | 지역 변수로 캐싱 |
| 일괄 업데이트 | `SetPositionAndRotation` 등 사용 |

---

## 🔗 참고 자료

이 문서를 작성하는 데 참고한 공식 문서 및 커뮤니티 자료입니다.

### 🔹 Unity 공식 문서

- [Unity Scripting API: Transform.localPosition](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/Transform-localPosition.html)
- [Unity Scripting API: Transform.TransformPoint](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/Transform.TransformPoint.html)
- [Unity Manual: Transforms](https://docs.unity3d.com/6000.3/Documentation/Manual/class-Transform.html)
- [Unity Scripting API: Camera.ScreenToWorldPoint](https://docs.unity3d.com/ScriptReference/Camera.ScreenToWorldPoint.html)
- [Unity Scripting API: Camera.WorldToScreenPoint](https://docs.unity3d.com/ScriptReference/Camera.WorldToScreenPoint.html)
- [Unity Scripting API: Space](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/Space.html)

### 🔹 커뮤니티 자료

- [Local Space and World Space in Unity - Medium](https://m-ansley.medium.com/local-space-and-world-space-in-unity-970e0bfb7694)
- [Convert from local position to world position - Unity Discussions](https://discussions.unity.com/t/convert-from-local-position-to-world-position/25406)
- [Unity3d Tutorial: Parenting and Children](https://riptutorial.com/unity3d/example/7478/parenting-and-children)
- [Converting Screen Space to World Space in Unity - Medium](https://medium.com/@austinjy13/converting-screen-space-to-world-space-in-unity-d32478eaf325)
- [A Guide to Unity's Coordinate System - TechArtHub](https://techarthub.com/a-guide-to-unitys-coordinate-system-with-practical-examples/)

### 🔹 추가 학습 자료

- [Transform concepts - Unity Entities](https://docs.unity3d.com/Packages/com.unity.entities@1.0/manual/transforms-concepts.html)
- [Unity Transforms and Object Parenting - Tutorialspoint](https://www.tutorialspoint.com/unity/unity_transforms_and_object_parenting.htm)
- [Understanding Transform position and rotation - Tee's Blog](https://killertee.wordpress.com/2017/12/11/unity-understanding-transform-position-and-rotation-in-difference-coordinate-systems/)

---

## 📌 마치며

Unity의 Position Transform은 게임 개발의 기초이자 핵심입니다. Local Space, World Space, 그리고 Screen/Viewport Space의 개념을 정확히 이해하고, 상황에 맞는 변환 메서드를 선택하는 것이 중요합니다.

**핵심 요약:**

1. **Local Position**: 부모 기준 상대 좌표 (`transform.localPosition`)
2. **World Position**: Scene 절대 좌표 (`transform.position`)
3. **Screen/Viewport Space**: 카메라 기준 화면 좌표

4. **변환 메서드 선택**:
   - 위치 변환: `TransformPoint` / `InverseTransformPoint`
   - 방향 변환: `TransformDirection` / `InverseTransformDirection`
   - 벡터 변환: `TransformVector` / `InverseTransformVector`

5. **성능 최적화**:
   - `localPosition` 우선 사용
   - Camera 및 Transform 참조 캐싱
   - 불필요한 변환 계산 최소화

이 지식을 바탕으로 복잡한 계층 구조, UI 배치, 물리 계산, 카메라 효과 등을 자유자재로 다룰 수 있을 것입니다.

---

**작성일**: 2026-02-02
**문서 버전**: 1.0
