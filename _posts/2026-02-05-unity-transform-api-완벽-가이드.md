---
layout: post
title: 260205 Unity Transform API 완벽 가이드
comments: true
tags:
- unity
- transform
- debugging
- bullet-physics
- api
- research
- guide
---
# 🛠️ 260205 Unity Transform API 완벽 가이드

Unity의 `GameObject.transform`은 모든 게임 오브젝트에 기본으로 포함되는 핵심 컴포넌트입니다. 이 가이드에서는 Transform의 모든 공개 속성과 메서드를 상세히 다룹니다.

---

## 📚 목차
1. [개요](#개요)
2. [Properties (속성)](#properties-속성)
3. [Methods (메서드)](#methods-메서드)
4. [좌표 변환 메서드 비교표](#좌표-변환-메서드-비교표)
5. [참고 자료](#참고-자료)

---

## 🧭 개요

Transform 컴포넌트는 게임 오브젝트의 **위치(Position)**, **회전(Rotation)**, **크기(Scale)**를 관리합니다. 또한 부모-자식 계층 구조를 통해 오브젝트 간의 관계를 정의합니다.

```
┌─────────────────────────────────────────────────────────┐
│                     Transform                            │
│  ┌─────────────┬─────────────┬─────────────┐            │
│  │  Position   │  Rotation   │   Scale     │            │
│  │  (Vector3)  │ (Quaternion)│  (Vector3)  │            │
│  └─────────────┴─────────────┴─────────────┘            │
│                        │                                 │
│            ┌───────────┴───────────┐                    │
│            ▼                       ▼                    │
│     Local Space              World Space                │
│  (부모 기준 상대값)        (월드 기준 절대값)           │
└─────────────────────────────────────────────────────────┘
```

---

## 📌 Properties (속성)

### 🔹 위치 관련 속성

#### ▫️ `position` (Vector3)
월드 좌표계에서의 절대 위치입니다.

```csharp
// 월드 좌표 (10, 5, 3)으로 이동
transform.position = new Vector3(10f, 5f, 3f);

// 현재 위치에서 위로 2만큼 이동
transform.position += Vector3.up * 2f;

// 현재 월드 위치 읽기
Vector3 worldPos = transform.position;
Debug.Log($"WORLD_POSITION pos:{worldPos}");
```

#### ▫️ `localPosition` (Vector3)
부모 Transform 기준 상대 위치입니다. 부모가 없으면 `position`과 동일합니다.

```csharp
// 부모로부터 오른쪽으로 5만큼 떨어진 위치
transform.localPosition = new Vector3(5f, 0f, 0f);

// 부모 기준 상대 위치 읽기
Vector3 localPos = transform.localPosition;
Debug.Log($"LOCAL_POSITION pos:{localPos}");
```

---

### 🔹 회전 관련 속성

#### ▫️ `rotation` (Quaternion)
월드 좌표계에서의 회전값입니다. Quaternion으로 저장됩니다.

```csharp
// 월드 기준 Y축으로 45도 회전
transform.rotation = Quaternion.Euler(0f, 45f, 0f);

// 특정 방향을 바라보는 회전값 생성
transform.rotation = Quaternion.LookRotation(Vector3.forward);

// 현재 회전값 읽기
Quaternion rot = transform.rotation;
```

#### ▫️ `localRotation` (Quaternion)
부모 Transform 기준 상대 회전값입니다.

```csharp
// 부모 기준 X축으로 30도 회전
transform.localRotation = Quaternion.Euler(30f, 0f, 0f);

// 로컬 회전 초기화
transform.localRotation = Quaternion.identity;
```

#### ▫️ `eulerAngles` (Vector3)
월드 기준 회전을 오일러 각도(도 단위)로 표현합니다.

```csharp
// Y축으로 90도 회전
transform.eulerAngles = new Vector3(0f, 90f, 0f);

// 현재 Y축 회전값만 읽기
float yRotation = transform.eulerAngles.y;
Debug.Log($"Y_ROTATION angle:{yRotation}");
```

#### ▫️ `localEulerAngles` (Vector3)
부모 기준 회전을 오일러 각도(도 단위)로 표현합니다.

```csharp
// 부모 기준 Z축으로 45도 회전
transform.localEulerAngles = new Vector3(0f, 0f, 45f);
```

---

### 🔹 크기 관련 속성

#### ▫️ `localScale` (Vector3)
부모 기준 상대 크기입니다.

```csharp
// 2배 크기로 설정
transform.localScale = new Vector3(2f, 2f, 2f);

// 균일하게 1.5배 확대
transform.localScale = Vector3.one * 1.5f;

// X축만 2배로 늘리기
transform.localScale = new Vector3(2f, 1f, 1f);
```

#### ▫️ `lossyScale` (Vector3) - **Read Only**
월드 좌표계에서의 전역 크기입니다. 부모의 크기가 모두 반영된 값입니다.

```csharp
// 실제 월드 스케일 확인 (읽기 전용)
Vector3 worldScale = transform.lossyScale;
Debug.Log($"WORLD_SCALE scale:{worldScale}");
```

---

### 🔹 방향 벡터 속성

```
        Y (up/green)
        │
        │    Z (forward/blue)
        │   /
        │  /
        │ /
        └───────── X (right/red)
```

#### ▫️ `forward` (Vector3)
Transform의 앞 방향(Z축, 파란색)을 나타내는 정규화된 벡터입니다.

```csharp
// 앞으로 이동
transform.position += transform.forward * speed * Time.deltaTime;

// 총알 발사 방향 설정
var bullet = Instantiate(bulletPrefab, firePoint.position, Quaternion.identity);
bullet.velocity = transform.forward * bulletSpeed;
```

#### ▫️ `right` (Vector3)
Transform의 오른쪽 방향(X축, 빨간색)을 나타내는 벡터입니다.

```csharp
// 오른쪽으로 이동
transform.position += transform.right * speed * Time.deltaTime;

// 횡이동 (Strafe)
float horizontal = Input.GetAxis("Horizontal");
transform.position += transform.right * horizontal * speed * Time.deltaTime;
```

#### ▫️ `up` (Vector3)
Transform의 위쪽 방향(Y축, 녹색)을 나타내는 벡터입니다.

```csharp
// 위로 점프
transform.position += transform.up * jumpForce * Time.deltaTime;

// 지면 법선 방향으로 정렬
transform.up = groundNormal;
```

---

### 🏗️ 계층 구조 관련 속성

#### ▫️ `parent` (Transform)
부모 Transform입니다. 최상위 오브젝트는 `null`입니다.

```csharp
// 부모 설정
transform.parent = newParent;

// 부모 제거 (루트로 이동)
transform.parent = null;

// 부모가 있는지 확인
if (transform.parent != null)
{
    Debug.Log($"PARENT_NAME name:{transform.parent.name}");
}
```

#### ▫️ `root` (Transform) - **Read Only**
계층 구조의 최상위 Transform을 반환합니다.

```csharp
// 최상위 부모 찾기
Transform rootTransform = transform.root;
Debug.Log($"ROOT_OBJECT name:{rootTransform.name}");

// 같은 계층에 속해 있는지 확인
if (transform.root == otherTransform.root)
{
    Debug.Log("SAME_HIERARCHY");
}
```

#### ▫️ `childCount` (int) - **Read Only**
직속 자식의 수를 반환합니다.

```csharp
int count = transform.childCount;
Debug.Log($"CHILD_COUNT count:{count}");

// 모든 자식 순회
for (int i = 0; i < transform.childCount; i++)
{
    Transform child = transform.GetChild(i);
    Debug.Log($"CHILD_FOUND index:{i} name:{child.name}");
}
```

---

### 🔹 매트릭스 속성

#### ▫️ `localToWorldMatrix` (Matrix4x4) - **Read Only**
로컬 좌표를 월드 좌표로 변환하는 4x4 행렬입니다.

```csharp
// 로컬 포인트를 월드로 변환 (수동 계산)
Matrix4x4 matrix = transform.localToWorldMatrix;
Vector3 worldPoint = matrix.MultiplyPoint3x4(localPoint);
```

#### ▫️ `worldToLocalMatrix` (Matrix4x4) - **Read Only**
월드 좌표를 로컬 좌표로 변환하는 4x4 행렬입니다.

```csharp
// 월드 포인트를 로컬로 변환 (수동 계산)
Matrix4x4 matrix = transform.worldToLocalMatrix;
Vector3 localPoint = matrix.MultiplyPoint3x4(worldPoint);
```

---

### 🔹 기타 속성

#### ▫️ `hasChanged` (bool)
마지막으로 `false`로 설정된 이후 Transform이 변경되었는지 여부입니다.

```csharp
void Update()
{
    if (transform.hasChanged)
    {
        Debug.Log("TRANSFORM_CHANGED");
        // 변경 감지 후 처리
        UpdateBoundingBox();
        // 플래그 리셋
        transform.hasChanged = false;
    }
}
```

#### ▫️ `hierarchyCapacity` (int)
계층 구조 데이터의 현재 용량입니다.

#### ▫️ `hierarchyCount` (int) - **Read Only**
계층 구조에 포함된 Transform의 총 개수입니다.

```csharp
Debug.Log($"HIERARCHY_COUNT count:{transform.hierarchyCount}");
```

---

## 📌 Methods (메서드)

### 🔹 이동 메서드

#### ▫️ `Translate(Vector3 translation, Space relativeTo = Space.Self)`
Transform을 지정된 방향과 거리만큼 이동시킵니다.

```csharp
// 로컬 좌표 기준 앞으로 이동
transform.Translate(Vector3.forward * speed * Time.deltaTime);

// 월드 좌표 기준 이동
transform.Translate(Vector3.up * speed * Time.deltaTime, Space.World);

// 다른 Transform 기준 이동
transform.Translate(Vector3.forward * speed * Time.deltaTime, cameraTransform);
```

**오버로드:**
```csharp
// Vector3 버전
void Translate(Vector3 translation);
void Translate(Vector3 translation, Space relativeTo);
void Translate(Vector3 translation, Transform relativeTo);

// x, y, z 분리 버전
void Translate(float x, float y, float z);
void Translate(float x, float y, float z, Space relativeTo);
void Translate(float x, float y, float z, Transform relativeTo);
```

---

### 🔹 회전 메서드

#### ▫️ `Rotate(Vector3 eulers, Space relativeTo = Space.Self)`
Transform을 오일러 각도만큼 회전시킵니다.

```csharp
// 로컬 Y축 기준 회전
transform.Rotate(Vector3.up * rotationSpeed * Time.deltaTime);

// 월드 Y축 기준 회전
transform.Rotate(Vector3.up * rotationSpeed * Time.deltaTime, Space.World);

// 각 축별 회전
transform.Rotate(0f, 45f, 0f); // Y축으로 45도

// 특정 축을 기준으로 회전
transform.Rotate(Vector3.right, 30f); // X축 기준 30도
```

**오버로드:**
```csharp
void Rotate(Vector3 eulers);
void Rotate(Vector3 eulers, Space relativeTo);
void Rotate(float xAngle, float yAngle, float zAngle);
void Rotate(float xAngle, float yAngle, float zAngle, Space relativeTo);
void Rotate(Vector3 axis, float angle);
void Rotate(Vector3 axis, float angle, Space relativeTo);
```

#### ▫️ `RotateAround(Vector3 point, Vector3 axis, float angle)`
월드 좌표의 특정 점을 중심으로 회전합니다.

```csharp
// 타겟 주위를 공전
transform.RotateAround(target.position, Vector3.up, orbitSpeed * Time.deltaTime);

// 원점을 중심으로 회전
transform.RotateAround(Vector3.zero, Vector3.up, 20f * Time.deltaTime);

// 행성 공전 예제
void Update()
{
    // 태양 주위를 Y축 기준으로 공전
    transform.RotateAround(sun.position, Vector3.up, orbitSpeed * Time.deltaTime);
    // 자전
    transform.Rotate(Vector3.up, spinSpeed * Time.deltaTime);
}
```

#### ▫️ `LookAt(Transform target, Vector3 worldUp = Vector3.up)`
타겟을 바라보도록 회전합니다.

```csharp
// 타겟을 바라보기
transform.LookAt(target);

// 월드 좌표를 바라보기
transform.LookAt(Vector3.zero);

// 커스텀 업 벡터 사용
transform.LookAt(target, Vector3.forward);

// 카메라가 플레이어를 추적
void LateUpdate()
{
    transform.LookAt(player);
}

// Y축 회전만 적용 (수평 회전)
void LookAtHorizontal(Transform target)
{
    Vector3 direction = target.position - transform.position;
    direction.y = 0; // Y 성분 제거
    if (direction != Vector3.zero)
    {
        transform.rotation = Quaternion.LookRotation(direction);
    }
}
```

**오버로드:**
```csharp
void LookAt(Transform target);
void LookAt(Transform target, Vector3 worldUp);
void LookAt(Vector3 worldPosition);
void LookAt(Vector3 worldPosition, Vector3 worldUp);
```

---

### 🔹 좌표 변환 메서드

```
┌─────────────────────────────────────────────────────────────────┐
│                    좌표 변환 메서드 흐름도                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Local Space                              World Space           │
│   (로컬 좌표)                               (월드 좌표)           │
│                                                                  │
│   ┌─────────────┐    TransformPoint()    ┌─────────────┐        │
│   │   Point     │ ────────────────────▶  │   Point     │        │
│   └─────────────┘    (위치+회전+크기)     └─────────────┘        │
│                  ◀────────────────────                           │
│                   InverseTransformPoint()                        │
│                                                                  │
│   ┌─────────────┐   TransformDirection() ┌─────────────┐        │
│   │  Direction  │ ────────────────────▶  │  Direction  │        │
│   └─────────────┘     (회전만)           └─────────────┘        │
│                  ◀────────────────────                           │
│                  InverseTransformDirection()                     │
│                                                                  │
│   ┌─────────────┐    TransformVector()   ┌─────────────┐        │
│   │   Vector    │ ────────────────────▶  │   Vector    │        │
│   └─────────────┘    (회전+크기)         └─────────────┘        │
│                  ◀────────────────────                           │
│                   InverseTransformVector()                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### ▫️ `TransformPoint(Vector3 position)` - Local → World
로컬 좌표를 월드 좌표로 변환합니다. **위치, 회전, 크기** 모두 영향을 받습니다.

```csharp
// 로컬 좌표 (1, 0, 0)을 월드 좌표로 변환
Vector3 worldPos = transform.TransformPoint(new Vector3(1f, 0f, 0f));

// 오브젝트 오른쪽에 아이템 생성
Vector3 spawnPos = transform.TransformPoint(Vector3.right * 2f);
Instantiate(itemPrefab, spawnPos, Quaternion.identity);

// 바운딩 박스 코너 계산
Vector3[] localCorners = new Vector3[]
{
    new Vector3(-0.5f, -0.5f, -0.5f),
    new Vector3(0.5f, -0.5f, -0.5f),
    new Vector3(-0.5f, 0.5f, -0.5f),
    new Vector3(0.5f, 0.5f, -0.5f),
    // ... 나머지 코너들
};

for (int i = 0; i < localCorners.Length; i++)
{
    Vector3 worldCorner = transform.TransformPoint(localCorners[i]);
}
```

#### ▫️ `InverseTransformPoint(Vector3 position)` - World → Local
월드 좌표를 로컬 좌표로 변환합니다.

```csharp
// 월드 좌표를 로컬 좌표로 변환
Vector3 localPos = transform.InverseTransformPoint(worldPosition);

// 타겟이 내 앞에 있는지 뒤에 있는지 확인
Vector3 localTargetPos = transform.InverseTransformPoint(target.position);
if (localTargetPos.z > 0)
{
    Debug.Log("TARGET_IN_FRONT");
}
else
{
    Debug.Log("TARGET_BEHIND");
}

// 카메라 기준 오브젝트 위치 계산
Vector3 cameraRelative = Camera.main.transform.InverseTransformPoint(transform.position);
if (cameraRelative.z > 0)
{
    Debug.Log("OBJECT_VISIBLE");
}
```

#### ▫️ `TransformDirection(Vector3 direction)` - Local → World
로컬 방향을 월드 방향으로 변환합니다. **회전만** 영향을 받습니다 (크기, 위치 무시).

```csharp
// 로컬 forward를 월드 방향으로 변환
Vector3 worldForward = transform.TransformDirection(Vector3.forward);

// 레이캐스트 방향 설정
Vector3 shootDirection = transform.TransformDirection(Vector3.forward);
if (Physics.Raycast(transform.position, shootDirection, out RaycastHit hit, 100f))
{
    Debug.Log($"HIT_DETECTED name:{hit.collider.name}");
}
```

#### ▫️ `InverseTransformDirection(Vector3 direction)` - World → Local
월드 방향을 로컬 방향으로 변환합니다.

```csharp
// 월드 forward를 로컬 방향으로 변환
Vector3 localDir = transform.InverseTransformDirection(Vector3.forward);

// 중력 방향을 로컬 좌표로 변환
Vector3 localGravity = transform.InverseTransformDirection(Physics.gravity);
```

#### ▫️ `TransformVector(Vector3 vector)` - Local → World
로컬 벡터를 월드 벡터로 변환합니다. **회전과 크기**가 영향을 받습니다 (위치 무시).

```csharp
// 로컬 벡터를 월드로 변환
Vector3 worldVector = transform.TransformVector(localVector);

// 스케일이 적용된 속도 계산
Vector3 localVelocity = new Vector3(1f, 0f, 2f);
Vector3 worldVelocity = transform.TransformVector(localVelocity);
```

#### ▫️ `InverseTransformVector(Vector3 vector)` - World → Local
월드 벡터를 로컬 벡터로 변환합니다.

```csharp
// 월드 벡터를 로컬로 변환
Vector3 localVector = transform.InverseTransformVector(worldVector);
```

**배치 처리 메서드:**
```csharp
// 여러 포인트를 한 번에 변환 (성능 최적화)
void TransformPoints(Span<Vector3> positions);
void InverseTransformPoints(Span<Vector3> positions);
void TransformDirections(Span<Vector3> directions);
void InverseTransformDirections(Span<Vector3> directions);
void TransformVectors(Span<Vector3> vectors);
void InverseTransformVectors(Span<Vector3> vectors);
```

---

### 🏗️ 계층 구조 메서드

#### ▫️ `SetParent(Transform parent, bool worldPositionStays = true)`
부모를 설정합니다.

```csharp
// 새 부모 설정 (월드 위치 유지)
transform.SetParent(newParent);

// 새 부모 설정 (로컬 위치/회전/크기 유지)
transform.SetParent(newParent, false);

// 부모에서 분리
transform.SetParent(null);

// UI 요소의 경우 worldPositionStays = false 권장
uiElement.transform.SetParent(canvas.transform, false);
```

#### ▫️ `DetachChildren()`
모든 자식의 부모를 해제합니다.

```csharp
// 모든 자식을 루트로 이동
transform.DetachChildren();

// 파괴 전 자식 분리
transform.DetachChildren();
Destroy(gameObject);
```

#### ▫️ `GetChild(int index)`
인덱스로 자식을 가져옵니다.

```csharp
// 첫 번째 자식 가져오기
Transform firstChild = transform.GetChild(0);

// 마지막 자식 가져오기
Transform lastChild = transform.GetChild(transform.childCount - 1);

// 모든 자식 순회
for (int i = 0; i < transform.childCount; i++)
{
    Transform child = transform.GetChild(i);
    Debug.Log($"CHILD name:{child.name} index:{i}");
}

// foreach로 순회 (IEnumerable 구현)
foreach (Transform child in transform)
{
    Debug.Log($"CHILD name:{child.name}");
}
```

#### ▫️ `Find(string name)`
이름으로 자식을 찾습니다. **직속 자식만** 검색합니다.

```csharp
// 직속 자식 찾기
Transform child = transform.Find("ChildName");

// 경로로 깊은 자식 찾기
Transform deepChild = transform.Find("Child/GrandChild/GreatGrandChild");

// null 체크 필수
Transform found = transform.Find("SomeChild");
if (found != null)
{
    Debug.Log($"CHILD_FOUND name:{found.name}");
}
else
{
    Debug.Log("CHILD_NOT_FOUND");
}
```

#### ▫️ `IsChildOf(Transform parent)`
이 Transform이 특정 부모의 자식인지 확인합니다.

```csharp
// 자식 여부 확인
if (transform.IsChildOf(targetParent))
{
    Debug.Log("IS_CHILD_OF_TARGET");
}

// 같은 계층에 속해 있는지 확인
if (transform.IsChildOf(otherTransform.root))
{
    Debug.Log("SAME_HIERARCHY");
}
```

---

### 🔹 형제(Sibling) 순서 메서드

```
┌─────────────────────────────────────────┐
│             Parent                       │
│  ┌─────┬─────┬─────┬─────┬─────┐       │
│  │  0  │  1  │  2  │  3  │  4  │       │
│  │ A   │ B   │ C   │ D   │ E   │       │
│  └─────┴─────┴─────┴─────┴─────┘       │
│     ▲                          ▲        │
│     │                          │        │
│  First                      Last        │
│  Sibling                   Sibling      │
└─────────────────────────────────────────┘
```

#### ▫️ `GetSiblingIndex()`
형제들 사이에서 자신의 인덱스를 반환합니다.

```csharp
int index = transform.GetSiblingIndex();
Debug.Log($"SIBLING_INDEX index:{index}");
```

#### ▫️ `SetSiblingIndex(int index)`
형제 순서를 설정합니다.

```csharp
// 두 번째 위치로 이동
transform.SetSiblingIndex(1);

// 인벤토리 슬롯 스왑
int myIndex = transform.GetSiblingIndex();
int otherIndex = other.GetSiblingIndex();
transform.SetSiblingIndex(otherIndex);
other.SetSiblingIndex(myIndex);
```

#### ▫️ `SetAsFirstSibling()`
형제 중 첫 번째로 이동합니다.

```csharp
// 맨 앞으로 이동
transform.SetAsFirstSibling();

// UI에서 가장 아래로 렌더링됨 (다른 UI 뒤에)
```

#### ▫️ `SetAsLastSibling()`
형제 중 마지막으로 이동합니다.

```csharp
// 맨 뒤로 이동
transform.SetAsLastSibling();

// UI에서 가장 위에 렌더링됨 (다른 UI 앞에)
// 팝업이나 모달에 유용
popupWindow.transform.SetAsLastSibling();
```

---

### 🛠️ 위치/회전 동시 설정 메서드

#### ▫️ `SetPositionAndRotation(Vector3 position, Quaternion rotation)`
월드 위치와 회전을 한 번에 설정합니다. 개별 설정보다 효율적입니다.

```csharp
// 한 번에 위치와 회전 설정 (최적화됨)
transform.SetPositionAndRotation(targetPosition, targetRotation);

// 텔레포트 구현
void Teleport(Vector3 destination, Quaternion rotation)
{
    transform.SetPositionAndRotation(destination, rotation);
}

// 스폰 포인트에서 생성
transform.SetPositionAndRotation(spawnPoint.position, spawnPoint.rotation);
```

#### ▫️ `SetLocalPositionAndRotation(Vector3 localPosition, Quaternion localRotation)`
로컬 위치와 회전을 한 번에 설정합니다.

```csharp
// 로컬 좌표로 한 번에 설정
transform.SetLocalPositionAndRotation(localPos, localRot);

// 오프셋 위치에 부착
void AttachToSocket(Transform socket, Vector3 offset, Quaternion rotOffset)
{
    transform.SetParent(socket);
    transform.SetLocalPositionAndRotation(offset, rotOffset);
}
```

#### ▫️ `GetPositionAndRotation(out Vector3 position, out Quaternion rotation)`
월드 위치와 회전을 한 번에 가져옵니다.

```csharp
// 한 번에 위치와 회전 읽기
transform.GetPositionAndRotation(out Vector3 pos, out Quaternion rot);

// 상태 저장
void SaveTransformState()
{
    transform.GetPositionAndRotation(out _savedPosition, out _savedRotation);
}
```

#### ▫️ `GetLocalPositionAndRotation(out Vector3 localPosition, out Quaternion localRotation)`
로컬 위치와 회전을 한 번에 가져옵니다.

```csharp
// 로컬 좌표로 한 번에 읽기
transform.GetLocalPositionAndRotation(out Vector3 localPos, out Quaternion localRot);
```

---

## ⚖️ 좌표 변환 메서드 비교표

| 메서드 | 변환 방향 | 영향 받는 요소 | 용도 |
|--------|----------|---------------|------|
| `TransformPoint` | Local → World | 위치, 회전, 크기 | 로컬 좌표의 점을 월드로 |
| `InverseTransformPoint` | World → Local | 위치, 회전, 크기 | 월드 좌표의 점을 로컬로 |
| `TransformDirection` | Local → World | 회전만 | 방향 벡터 변환 (길이 유지) |
| `InverseTransformDirection` | World → Local | 회전만 | 방향 벡터 변환 (길이 유지) |
| `TransformVector` | Local → World | 회전, 크기 | 벡터 변환 (스케일 반영) |
| `InverseTransformVector` | World → Local | 회전, 크기 | 벡터 변환 (스케일 반영) |

---

## 🧪 실용 예제 모음

### 🔹 1. 부드러운 추적 카메라

```csharp
public class SmoothFollowCamera : MonoBehaviour
{
    public Transform _target;
    public Vector3 _offset = new Vector3(0f, 5f, -10f);
    public float _smoothSpeed = 5f;

    void LateUpdate()
    {
        // 타겟의 로컬 오프셋을 월드 좌표로 변환
        Vector3 desiredPosition = _target.TransformPoint(_offset);

        // 부드럽게 이동
        transform.position = Vector3.Lerp(transform.position, desiredPosition, _smoothSpeed * Time.deltaTime);

        // 타겟 바라보기
        transform.LookAt(_target);
    }
}
```

### 🔹 2. 오브젝트 공전 시스템

```csharp
public class OrbitSystem : MonoBehaviour
{
    public Transform _center;
    public float _orbitSpeed = 30f;
    public float _spinSpeed = 100f;

    void Update()
    {
        // 중심점 주위 공전
        transform.RotateAround(_center.position, Vector3.up, _orbitSpeed * Time.deltaTime);

        // 자전
        transform.Rotate(Vector3.up, _spinSpeed * Time.deltaTime, Space.Self);
    }
}
```

### 🔹 3. 시야 내 적 감지

```csharp
public class EnemyDetector : MonoBehaviour
{
    public float _viewAngle = 60f;
    public float _viewDistance = 10f;

    public bool IsInView(Transform target)
    {
        // 월드 좌표를 로컬로 변환
        Vector3 localPos = transform.InverseTransformPoint(target.position);

        // 뒤에 있으면 false
        if (localPos.z < 0)
        {
            return false;
        }

        // 거리 체크
        if (localPos.magnitude > _viewDistance)
        {
            return false;
        }

        // 각도 체크
        float angle = Mathf.Atan2(localPos.x, localPos.z) * Mathf.Rad2Deg;
        return Mathf.Abs(angle) < _viewAngle / 2f;
    }
}
```

### 🏗️ 4. 계층 구조 유틸리티

```csharp
public static class TransformExtensions
{
    // 모든 자손 찾기
    public static List<Transform> GetAllDescendants(this Transform parent)
    {
        var result = new List<Transform>();
        GetDescendantsRecursive(parent, result);
        return result;
    }

    private static void GetDescendantsRecursive(Transform current, List<Transform> list)
    {
        for (int i = 0; i < current.childCount; i++)
        {
            Transform child = current.GetChild(i);
            list.Add(child);
            GetDescendantsRecursive(child, list);
        }
    }

    // 이름으로 깊은 검색
    public static Transform FindDeep(this Transform parent, string name)
    {
        Transform result = parent.Find(name);
        if (result != null)
        {
            return result;
        }

        for (int i = 0; i < parent.childCount; i++)
        {
            result = parent.GetChild(i).FindDeep(name);
            if (result != null)
            {
                return result;
            }
        }
        return null;
    }
}
```

---

## 🔗 참고 자료

- [Unity 공식 Transform API 문서](https://docs.unity3d.com/ScriptReference/Transform.html)
- [Unity Manual - Transforms](https://docs.unity3d.com/6000.1/Documentation/Manual/class-Transform.html)
- [TransformPoint vs TransformDirection 차이점](https://irfanbaysal.medium.com/differences-between-transformvector-transformpoint-and-transformdirection-2df6f3ebbe11)
- [Unity C# Reference Source - Transform](https://github.com/Unity-Technologies/UnityCsReference/blob/master/Runtime/Transform/ScriptBindings/Transform.bindings.cs)
- [Unity Discussions - InverseTransformPoint 이해하기](https://discussions.unity.com/t/how-does-the-inversetransformpoint-work/863616)

---

*이 문서는 Unity 6000.3 버전 기준으로 작성되었습니다.*
