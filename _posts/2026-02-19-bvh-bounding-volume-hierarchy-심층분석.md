---
layout: post
title: 260219 BVH Bounding Volume Hierarchy 심층분석
comments: true
tags:
- unity
- performance
- optimization
- gpu
- raytracing
- bvh
- sat
- cryptography
- research
- analysis
---
# 🔍 260219 BVH_Bounding_Volume_Hierarchy_심층분석

> 📝 **작성일**: 2026-02-19
> 💡 **주제**: BVH(Bounding Volume Hierarchy) 알고리즘, 구축 전략, 순회 기법, 최적화 방안 및 산업 활용 사례 심층 분석
> 💡 **분량**: PDF 5페이지 상당 상세 분석

---

## 📚 목차

1. [BVH 기본 개념 및 원리](#1-bvh-기본-개념-및-원리)
2. [바운딩 볼륨 종류와 트레이드오프](#2-바운딩-볼륨-종류와-트레이드오프)
3. [BVH 구축 알고리즘](#3-bvh-구축-알고리즘)
4. [BVH 순회(Traversal) 알고리즘](#4-bvh-순회traversal-알고리즘)
5. [BVH 최적화 기법](#5-bvh-최적화-기법)
6. [실제 활용 사례](#6-실제-활용-사례)
7. [다른 공간 분할 구조와의 비교](#7-다른-공간-분할-구조와의-비교)
8. [결론](#8-결론)
9. [참고 자료](#9-참고-자료)

---

## 🧠 1. BVH 기본 개념 및 원리

### 🔹 1.1 정의

**BVH(Bounding Volume Hierarchy)**는 3D 공간의 기하학적 객체(geometric objects) 집합을 계층적 트리 구조로 조직하는 공간 가속 구조(spatial acceleration structure)이다. 모든 기하학적 객체는 트리의 리프 노드(leaf node)에 저장되며, 각 내부 노드(internal node)는 자식 노드에 포함된 모든 프리미티브(primitive)를 감싸는 바운딩 볼륨(bounding volume)을 저장한다.

핵심 원리는 다음과 같다:

- **리프 노드**: 실제 기하학적 객체(삼각형, 메시 등)를 저장
- **내부 노드**: 자식 노드 전체를 감싸는 바운딩 볼륨을 저장
- **루트 노드**: 전체 장면의 모든 객체를 감싸는 최상위 바운딩 볼륨

### 🔹 1.2 가지치기(Pruning) 원리

BVH의 핵심 가치는 **가지치기(pruning)**에 있다. 레이(ray)나 질의 영역이 부모 노드의 바운딩 볼륨과 교차하지 않으면, 해당 노드 아래의 모든 자식 노드를 검사할 필요가 없다. 이를 통해 O(n) 복잡도의 전수 검사를 평균적으로 O(log n) 수준으로 줄일 수 있다.

```
[루트 바운딩 볼륨]
      |
   교차 테스트 ----> 교차하지 않음 => 전체 서브트리 가지치기(Skip)
      |
   교차함 => 자식 노드로 재귀 탐색
     / \
  [좌측]  [우측]
   / \     / \
 [A] [B] [C] [D]   <-- 리프 노드(실제 객체)
```

이 가지치기 메커니즘은 충돌 감지(collision detection), 레이트레이싱(ray tracing), 최근접 탐색(nearest neighbor search) 등에서 동일하게 적용된다. 부모 볼륨이 교차하지 않으면 자식 전체를 건너뛰기 때문에, 잘 구축된 BVH는 전체 객체 중 극히 일부만 실제로 검사하게 된다.

### 🏗️ 1.3 트리 구조

일반적으로 BVH는 **이진 트리(binary tree)** 형태로 구축된다. 메모리 효율을 위해 노드를 깊이 우선(depth-first) 순서로 선형 배열에 배치하는 방식이 널리 사용되며, 이 경우 첫 번째 자식은 부모 노드 바로 다음에 위치하므로 포인터 없이 접근 가능하고, 두 번째 자식의 오프셋만 별도로 저장하면 된다.

---

## 📌 2. 바운딩 볼륨 종류와 트레이드오프

BVH에서 사용하는 바운딩 볼륨의 종류에 따라 **적합도(tightness)**와 **연산 비용(computational cost)** 사이의 트레이드오프가 발생한다.

### 🔹 2.1 AABB (Axis-Aligned Bounding Box)

- **정의**: 세계 좌표축에 정렬된 직육면체. min(x, y, z)과 max(x, y, z) 두 점으로 정의
- **장점**: 교차 테스트가 매우 빠름(단순 비교 연산), 구축 비용이 낮음, 메모리 효율적(6개 float)
- **단점**: 축 정렬이므로 회전된 객체에 대해 과도한 여백(dead space) 발생
- **사용 빈도**: 현대 BVH 구현의 대다수가 AABB를 기본으로 채택

### 🔹 2.2 OBB (Oriented Bounding Box)

- **정의**: 객체 방향에 맞춰 회전된 직육면체
- **장점**: 가장 타이트(tight)한 피팅 가능, 회전된 세장형(elongated) 객체에 유리
- **단점**: 교차 테스트 비용이 높음(SAT 알고리즘 등 필요), 구축 비용이 큼
- **사용 사례**: 로봇공학, 정밀 충돌 감지 등 정확도가 중시되는 분야

### 🔹 2.3 Bounding Sphere

- **정의**: 중심점과 반지름으로 정의된 구
- **장점**: 교차 테스트가 극히 단순(거리 비교), 회전 불변(rotation-invariant)
- **단점**: 피팅이 가장 느슨하여 불필요한 교차 판정이 많이 발생
- **사용 사례**: 대략적인 초기 필터링(broadphase)

### 🔹 2.4 k-DOP (Discrete Oriented Polytope)

- **정의**: k개의 고정 방향 슬랩(slab)의 교차로 정의되는 볼록 다면체
- **장점**: AABB보다 타이트한 피팅, OBB보다 빠른 교차 테스트
- **단점**: AABB보다 메모리 사용량 증가, k 값이 커질수록 비용 증가
- **사용 사례**: AABB와 OBB 사이의 절충이 필요한 경우

### ⚖️ 2.5 비교 요약

| 바운딩 볼륨 | 적합도(Tightness) | 교차 테스트 비용 | 구축 비용 | 메모리 | 대표 용도 |
|---|---|---|---|---|---|
| **AABB** | 중간 | 매우 낮음 | 매우 낮음 | 24 bytes | 레이트레이싱, 게임 물리 |
| **OBB** | 높음 | 높음 | 높음 | 60+ bytes | 로봇공학, 정밀 시뮬레이션 |
| **Sphere** | 낮음 | 매우 낮음 | 낮음 | 16 bytes | Broadphase 필터링 |
| **k-DOP** | 중~높음 | 중간 | 중간 | k x 4 bytes | 변형체 충돌 감지 |

실무에서 가장 널리 사용되는 것은 **AABB**이다. Unity Physics, NVIDIA OptiX, Embree 등 주요 엔진과 프레임워크 모두 AABB 기반 BVH를 기본으로 채택하고 있다.

---

## 🧠 3. BVH 구축 알고리즘

BVH 구축 방식은 크게 **Top-down**, **Bottom-up**, **Insertion**, **LBVH** 네 가지 범주로 나뉜다.

### 🔹 3.1 Top-down 구축

가장 직관적이고 널리 사용되는 방식이다.

**알고리즘 흐름:**
1. 전체 프리미티브 집합에 대한 바운딩 볼륨을 계산하여 루트 노드 생성
2. 분할 기준(splitting criterion)에 따라 프리미티브를 두 부분 집합으로 분할
3. 각 부분 집합에 대해 재귀적으로 1~2 과정 반복
4. 프리미티브가 충분히 적으면(보통 1~4개) 리프 노드 생성

**분할 전략:**
- **중간점 분할(Midpoint Split)**: 가장 긴 축의 중앙에서 분할. O(n) 시간, 품질 보통
- **중앙값 분할(Median Split)**: 프리미티브 개수를 균등하게 이등분. 균형 잡힌 트리 보장
- **SAH 분할**: 비용 함수를 최소화하는 최적 분할점 탐색. 가장 높은 품질

```
Top-down BVH 구축 의사코드:

function BuildBVH(primitives):
    node = new Node()
    node.bbox = ComputeBounds(primitives)

    if primitives.count <= MAX_LEAF_SIZE:
        node.primitives = primitives    // 리프 노드
        return node

    axis = ChooseSplitAxis(primitives)  // 가장 긴 축 또는 SAH 기반
    splitPos = ChooseSplitPosition(primitives, axis)  // 중간점, 중앙값 또는 SAH

    (left, right) = Partition(primitives, axis, splitPos)

    node.left  = BuildBVH(left)
    node.right = BuildBVH(right)
    return node
```

### 🔹 3.2 Surface Area Heuristic (SAH)

SAH는 BVH 품질 최적화의 핵심 알고리즘으로, 레이트레이싱에서 가장 널리 사용되는 분할 비용 모델이다.

**핵심 원리**: 노드와의 교차 확률은 해당 노드의 바운딩 볼륨 표면적에 비례한다는 가정에 기반한다.

**비용 함수:**
```
C(A, B) = t_trav + P(A) * N(A) * t_isect + P(B) * N(B) * t_isect
```

여기서:
- `t_trav`: 노드 순회 비용 (상수)
- `P(A)`, `P(B)`: 자식 A, B와 교차할 확률 = SA(child) / SA(parent)
- `N(A)`, `N(B)`: 자식 A, B에 포함된 프리미티브 수
- `t_isect`: 프리미티브 교차 테스트 비용
- `SA(x)`: 바운딩 볼륨 x의 표면적

**구현 방식 (Binned SAH):**
1. 분할 축(x, y, z)마다 공간을 N개의 빈(bin)으로 분할 (보통 12~32개)
2. 각 프리미티브의 중심(centroid)이 속하는 빈에 할당
3. 각 분할 위치에 대해 좌/우 자식의 바운딩 박스와 프리미티브 수 계산
4. SAH 비용이 최소인 분할 위치를 선택

**장점**: 높은 품질의 BVH 생성, 레이트레이싱 성능 최적화에 탁월
**단점**: 구축 시간이 O(n log^2 n)으로 비교적 높음, 다수 패스 필요

### 🔹 3.3 Bottom-up 구축

리프에서 시작하여 가장 가까운 노드 쌍을 반복적으로 병합해 올라가는 방식이다.

**알고리즘 흐름:**
1. 각 프리미티브를 리프 노드로 생성
2. 가장 가까운(또는 병합 비용이 최소인) 노드 쌍을 탐색
3. 두 노드를 하나의 부모 노드로 병합
4. 루트 노드가 남을 때까지 2~3 반복

**장점**: 이론적으로 최적에 가까운 트리 구조 가능
**단점**: 탐욕적(greedy) 병합이 O(n^2) 이상의 비용 소요, 대규모 장면에서 비현실적

### 🧠 3.4 LBVH (Linear BVH)

SAH의 높은 구축 비용을 해결하기 위해 개발된 고속 구축 방식이다.

**핵심 아이디어**: BVH 구축을 **정렬 문제(sorting problem)**로 변환한다. 3D 좌표를 1D **Morton 코드(Z-order curve)**로 매핑하면, 공간적으로 인접한 객체들이 1D 상에서도 인접하게 된다.

**Morton 코드 계산:**
```
3D 좌표 (x, y, z)를 이진 표현:
x = x2 x1 x0
y = y2 y1 y0
z = z2 z1 z0

Morton 코드 = z2 y2 x2 z1 y1 x1 z0 y0 x0
(각 축의 비트를 인터리빙)
```

**LBVH 구축 절차:**
1. 각 프리미티브의 중심 좌표로부터 Morton 코드 계산
2. Morton 코드 기준으로 프리미티브를 정렬 (병렬 기수 정렬 가능)
3. 정렬된 배열에서 Morton 코드의 최상위 차이 비트(highest differing bit)를 기준으로 분할
4. 바운딩 박스를 리프에서 루트 방향으로 병렬 계산 (bottom-up reduction)

**시간 복잡도**: O(n log n) -- 정렬이 지배적, 나머지는 선형
**장점**: GPU에서 완전 병렬화 가능, 실시간 구축
**단점**: SAH 대비 트리 품질이 다소 낮을 수 있음

### 🧠 3.5 HLBVH (Hierarchical LBVH)

LBVH와 SAH의 장점을 결합한 하이브리드 방식이다.

- **하위 레벨**: Morton 코드 기반 LBVH로 소규모 클러스터(treelet) 구축 -- 고속
- **상위 레벨**: 생성된 treelet들을 SAH로 결합 -- 고품질

이 방식은 GPU에서의 실시간 구축과 높은 트리 품질을 동시에 달성한다.

### ⚖️ 3.6 구축 알고리즘 비교

| 알고리즘 | 구축 시간 | 트리 품질 | 병렬화 | 대표 용도 |
|---|---|---|---|---|
| **Top-down (Midpoint)** | O(n log n) | 낮음 | 어려움 | 프로토타이핑 |
| **Top-down (SAH)** | O(n log^2 n) | 높음 | 부분적 | 정적 장면 레이트레이싱 |
| **Bottom-up** | O(n^2) | 매우 높음 | 어려움 | 소규모 장면 |
| **LBVH** | O(n log n) | 중간 | 완전 병렬 | GPU 실시간 |
| **HLBVH** | O(n log n) | 중~높음 | 완전 병렬 | GPU 고품질 실시간 |

---

## 🧠 4. BVH 순회(Traversal) 알고리즘

### 🧠 4.1 Ray-BVH 교차 테스트 (레이트레이싱)

레이트레이싱에서 BVH 순회는 가장 핵심적인 연산이다. 기본 알고리즘은 다음과 같다:

```
function TraverseBVH(ray, node):
    if not IntersectAABB(ray, node.bbox):
        return NO_HIT                    // 가지치기

    if node.isLeaf:
        return IntersectPrimitives(ray, node.primitives)

    hitLeft  = TraverseBVH(ray, node.left)
    hitRight = TraverseBVH(ray, node.right)

    return CloserHit(hitLeft, hitRight)
```

**Ray-AABB 교차 테스트 (Slab Method):**

가장 빠른 Ray-AABB 교차 알고리즘은 **슬랩 방법(Slab Method)**이다. AABB를 3쌍의 평행 평면(slab)으로 간주하고, 레이가 각 슬랩을 통과하는 구간을 계산하여 교차 여부를 판별한다.

```
function IntersectAABB(ray, bbox):
    // 각 축에 대해 진입/이탈 t 값 계산
    tx1 = (bbox.min.x - ray.origin.x) * ray.invDir.x
    tx2 = (bbox.max.x - ray.origin.x) * ray.invDir.x

    tmin = min(tx1, tx2)
    tmax = max(tx1, tx2)

    ty1 = (bbox.min.y - ray.origin.y) * ray.invDir.y
    ty2 = (bbox.max.y - ray.origin.y) * ray.invDir.y

    tmin = max(tmin, min(ty1, ty2))
    tmax = min(tmax, max(ty1, ty2))

    tz1 = (bbox.min.z - ray.origin.z) * ray.invDir.z
    tz2 = (bbox.max.z - ray.origin.z) * ray.invDir.z

    tmin = max(tmin, min(tz1, tz2))
    tmax = min(tmax, max(tz1, tz2))

    return tmax >= max(tmin, 0)   // tmax >= 0이면 전방 교차
```

이 알고리즘은 분기(branch)가 거의 없어 GPU에서 특히 효율적이다.

### 🚀 4.2 순회 최적화 기법

**가까운 자식 우선 순회 (Front-to-Back Traversal):**
레이 방향을 고려하여 가까운 자식 노드를 먼저 방문한다. 가까운 쪽에서 교차점을 찾으면, 먼 쪽 자식의 tmin이 이미 찾은 교차점보다 멀 경우 가지치기할 수 있다.

**스택리스 순회 (Stackless Traversal):**
GPU에서는 스레드별 스택 메모리가 제한적이므로, **스킵 링크(skip link)** 또는 **이스케이프 인덱스(escape index)**를 노드에 추가하여 서브트리를 건너뛸 때 스택 없이 다음 후보 노드로 즉시 이동할 수 있게 한다. 이는 GPU에서 스레드 분기(divergence)와 스레드별 메모리 사용량을 최소화한다.

### 🧠 4.3 충돌 감지용 순회

두 BVH 트리 간의 충돌 감지(BVH-BVH collision detection)는 다음과 같이 수행된다:

```
function CollisionTest(nodeA, nodeB):
    if not Overlap(nodeA.bbox, nodeB.bbox):
        return                           // 가지치기

    if nodeA.isLeaf and nodeB.isLeaf:
        DetailedCollisionTest(nodeA, nodeB)
        return

    // 더 큰 노드를 분할하여 재귀
    if nodeA.volume > nodeB.volume:
        CollisionTest(nodeA.left,  nodeB)
        CollisionTest(nodeA.right, nodeB)
    else:
        CollisionTest(nodeA, nodeB.left)
        CollisionTest(nodeA, nodeB.right)
```

### 🔹 4.4 최근접 객체 탐색 (Nearest Neighbor)

BVH를 이용한 최근접 점 탐색에서는 현재까지 발견된 최소 거리를 유지하면서, 노드의 바운딩 볼륨까지의 최소 거리가 현재 최소 거리보다 크면 해당 서브트리를 가지치기한다.

---

## 🚀 5. BVH 최적화 기법

### 🔹 5.1 트리 재적합(Refitting)

동적 장면에서 객체가 이동하면 BVH를 갱신해야 한다. **Refitting**은 트리 구조를 유지한 채 바운딩 볼륨만 재계산하는 방식이다.

**장점:**
- 구축 대비 극히 빠름 (리프에서 루트까지 bottom-up으로 바운딩 박스만 갱신)
- 애니메이션 업데이트 비용보다 적을 수 있음

**단점:**
- 객체가 크게 이동하면 바운딩 볼륨 간 중첩(overlap)이 증가하여 트리 품질 저하
- 극단적인 경우 순회 성능이 급격히 하락

**적용 전략:**
- 매 프레임 refitting, 일정 간격으로 전체 재구축
- 트리 품질 지표(예: SAH 비용)를 모니터링하여 재구축 시점 결정

### 🔹 5.2 동적 객체 처리 전략

| 전략 | 방식 | 장점 | 단점 |
|---|---|---|---|
| **Refitting** | BV만 갱신 | 매우 빠름 | 품질 저하 가능 |
| **전체 재구축** | 트리 전체 새로 구축 | 최적 품질 | 구축 비용 높음 |
| **부분 재구축** | 품질 저하된 서브트리만 재구축 | 비용/품질 절충 | 구현 복잡 |
| **비동기 재구축** | 여러 프레임에 걸쳐 점진적 재구축 | 프레임 시간 안정 | 일시적 품질 저하 |
| **삽입/삭제** | 개별 객체 단위 트리 수정 | 유연한 동적 처리 | 트리 불균형 가능 |

### 🧠 5.3 GPU 가속 BVH

**NVIDIA의 병렬 BVH 구축 (Karras 2012):**

Tero Karras가 제안한 알고리즘은 GPU에서 BVH를 완전히 병렬로 구축한다.

1. **Morton 코드 계산**: 각 프리미티브의 중심에 대해 30비트 Morton 코드 계산 (GPU 커널)
2. **병렬 기수 정렬**: Morton 코드 기준으로 프리미티브 정렬 (CUB/Thrust 라이브러리)
3. **계층 생성**: 정렬된 배열에서 각 내부 노드의 범위를 독립적으로 결정 (GPU 커널)
4. **바운딩 박스 계산**: 리프에서 루트로 원자적(atomic) 카운터를 이용한 bottom-up 리덕션

모든 단계가 GPU에서 완전 병렬화되며, 수백만 개의 프리미티브를 밀리초 단위로 처리할 수 있다.

### 🚀 5.4 메모리 레이아웃 최적화

- **깊이 우선 배치(DFS layout)**: 깊이 우선 순서로 노드를 배열에 저장. 첫 번째 자식은 부모 바로 다음에 위치
- **캐시 친화적 배치**: 자주 함께 접근되는 노드를 메모리상 인접하게 배치하여 캐시 적중률 향상
- **노드 압축**: 32바이트 또는 64바이트 경계에 맞춘 컴팩트 노드 포맷

### 🧠 5.5 Wide BVH (Multi-branching BVH)

전통적인 이진 BVH 대신 4-way 또는 8-way BVH를 사용하면:
- 트리 깊이 감소로 순회 횟수 절감
- SIMD 명령어로 여러 자식 노드를 동시 교차 테스트
- NVIDIA의 RT Core는 내부적으로 multi-way BVH를 사용하는 것으로 알려져 있음

---

## 🏢 6. 실제 활용 사례

### 🔹 6.1 레이트레이싱 -- NVIDIA RTX / OptiX

**NVIDIA OptiX**는 GPU 레이트레이싱 전용 프레임워크로, BVH를 핵심 가속 구조로 사용한다.

- **RT Core**: NVIDIA RTX GPU의 하드웨어 유닛으로, BVH 순회와 Ray-삼각형 교차 테스트를 전용 하드웨어에서 수행
- **성능**: RTX 하드웨어 가속으로 일반 셰이더 대비 2~3배(일반), 최대 15~30배(최적화) 속도 향상
- **자동 BVH 구축**: OptiX가 장면 지오메트리에 대해 자동으로 최적 BVH를 구축하고, 동적 장면에 대한 refitting/rebuilding 제공
- **Vulkan/DirectX Ray Tracing**: 마이크로소프트 DXR과 Vulkan RT 확장에서도 BVH 기반 가속 구조(Acceleration Structure)를 표준으로 채택

> 💡 참고: [NVIDIA OptiX Ray Tracing Engine](https://developer.nvidia.com/rtx/ray-tracing/optix), [OptiX 8 Blog](https://developer.nvidia.com/blog/flexible-and-powerful-ray-tracing-with-optix-8/)

### 🔹 6.2 게임 엔진

**Unity Physics:**
- Unity의 DOTS(Data-Oriented Technology Stack) 기반 물리 엔진은 AABB 기반 BVH를 사용하여 콜라이더를 공간적으로 조직
- Broadphase 단계에서 BVH를 통해 잠재적 충돌 쌍을 빠르게 필터링
- 메모리 소비 절감과 빠른 오버랩 테스트를 위해 AABB를 강체(rigid body) 바운딩 볼륨으로 채택

> 💡 참고: [Unity Physics Simulation Pipeline](https://docs.unity3d.com/Packages/com.unity.physics@1.3/manual/concepts-simulation.html), [Unity BVH Discussion](https://discussions.unity.com/t/anyone-knows-what-kind-of-bvh-is-used-in-unitys-new-physics/750753)

**Unreal Engine 5:**
- UE5는 BVH를 기본 broadphase 알고리즘으로 사용
- 각 이동 가능 객체를 AABB로 감싸 BVH에 배치
- Nanite 가상화 지오메트리 시스템에서도 BVH 기반 가시성 판정 활용
- Lumen 글로벌 일루미네이션에서 소프트웨어 레이트레이싱 시 BVH 사용

### 🎮 6.3 영화 VFX / 프로덕션 렌더링

**Pixar RenderMan:**
- Pixar의 모든 영화에 사용되는 프로덕션 렌더러
- CPU 경로에서 Embree와 유사한 BVH 구축/교차 방식 사용
- GPU 경로에서 CUDA 기반 BVH(OptiX 대안) 사용
- 폴리메시별(per-primitive) BVH를 구축하며, 내부 노드가 메인 BVH 구조를 따름
- RenderMan XPU는 CPU와 GPU에서 동시에 레이트레이싱을 수행하며, 양쪽 모두 BVH 기반

> 💡 참고: [RenderMan: An Advanced Path-Tracing Architecture](https://dl.acm.org/doi/10.1145/3182162), [RenderMan XPU (HPG 2025)](https://graphics.pixar.com/library/RenderManXPU/paper.pdf)

**Weta Digital Manuka:**
- 반지의 제왕, 아바타 등의 VFX에 사용된 프로덕션 렌더러
- 최종 테셀레이션된 지오메트리에 대해 단일 BVH를 구축
- 멀티레벨 BVH 대신 단일 BVH를 사용하여 레이트레이싱 효율 극대화
- 비용이 큰 온디맨드 테셀레이션 마이크로폴리곤 캐싱을 회피

> 💡 참고: [Manuka: A Batch-Shading Architecture](https://dl.acm.org/doi/10.1145/3182161), [Weta Digital Manuka](https://www.pixelsham.com/2019/03/29/weta-digital-manuka-and-gazebo-renderers/)

### 🔹 6.4 물리 시뮬레이션

- **입자(particle) 시뮬레이션**: N-body 문제에서 BVH를 이용한 충돌 감지 및 근접 입자 탐색
- **변형체(deformable body)**: 천(cloth), 연체(soft body) 시뮬레이션에서 자기 충돌(self-collision) 감지에 BVH 활용
- **유체 시뮬레이션**: SPH(Smoothed Particle Hydrodynamics)에서 이웃 탐색 가속

### 🏢 6.5 CAD/BIM 충돌 감지

BIM(Building Information Modeling)에서 클래시 감지(clash detection)는 설계 단계에서 건축, 구조, 기계, 전기, 배관(MEP) 시스템 간의 간섭을 사전에 발견하는 핵심 프로세스이다.

- **Autodesk Navisworks**: 다수의 3D 모델을 통합하여 BVH 기반 공간 인덱싱으로 간섭 검사 수행
- **Hard Clash**: 두 부재가 동일 공간을 점유하는 기하학적 간섭 -- BVH의 정밀 교차 테스트로 감지
- **Soft Clash**: 유지보수 공간 등 버퍼 영역 침범 -- 확장된 바운딩 볼륨으로 감지
- 수십만 개의 BIM 요소에 대해 BVH 없이는 클래시 감지가 비현실적으로 느려짐

> 💡 참고: [Spatial Glossary: Clash Detection](https://www.spatial.com/glossary/what-is-clash-detection), [Autodesk BIM Clash Detection](https://www.autodesk.com/blogs/construction/bim-clash-detection/)

### 🏢 6.6 기타 활용 영역

- **RTNN (Ray Tracing based Nearest Neighbor)**: NVIDIA RT Core를 활용한 최근접 이웃 탐색 가속
- **RTIndeX**: 하드웨어 레이트레이싱을 데이터베이스 인덱싱에 활용
- **의료 영상**: CT/MRI 볼륨 렌더링에서 빈 공간 건너뛰기(empty space skipping)
- **GIS/지리 정보**: 대규모 지형 데이터에서의 가시성 분석 및 공간 질의

> 💡 참고: [RTNN Paper](https://arxiv.org/pdf/2201.01366), [RTIndeX Paper](https://arxiv.org/pdf/2303.01139)

---

## 🏗️ 7. 다른 공간 분할 구조와의 비교

### ⚖️ 7.1 비교 대상

| 특성 | BVH | KD-Tree | Octree | BSP Tree |
|---|---|---|---|---|
| **분할 방식** | 객체 분할(Object partitioning) | 공간 분할(Space partitioning) | 공간 균등 분할 | 임의 평면 분할 |
| **객체 중복** | 없음 (각 객체는 하나의 리프에만) | 있음 (객체가 여러 노드에 참조) | 있음 | 있음 |
| **동적 갱신** | Refitting으로 빠른 갱신 가능 | 갱신 어려움, 재구축 필요 | 비교적 쉬운 갱신 | 재구축 필요 |
| **구축 비용** | 중간 | 중간~높음 | 낮음 | 높음 |
| **순회 성능** | 우수 | 우수 (높은 깊이 복잡도에서) | 보통 | 우수 (정적 장면) |
| **메모리** | 효율적 | 보통 | 높음 (빈 노드 존재) | 높음 |
| **GPU 친화성** | 매우 높음 | 보통 | 보통 | 낮음 |

### ⚖️ 7.2 상세 비교

**BVH vs KD-Tree:**
- KD-Tree는 높은 깊이 복잡도(depth complexity)를 가진 복잡한 장면에서 이론적으로 더 높은 성능을 보일 수 있다
- 그러나 BVH는 객체 중복이 없어 메모리 효율적이고, refitting을 통한 동적 갱신이 가능하다
- GPU 병렬화 측면에서 BVH가 압도적으로 유리하여, 현대 실시간 렌더링에서는 BVH가 사실상 표준

**BVH vs Octree:**
- Octree는 공간을 균등하게 8분할하므로 구축이 단순하고 연산이 2의 거듭제곱 기반
- 객체가 Octree 경계를 걸치면 여러 노드에 중복 등록되거나 상위 노드에 남게 됨
- BVH는 월드 그리드에 종속되지 않아 더 유연하며, 객체 이동 시 바운딩만 갱신하면 됨

**BVH vs BSP Tree:**
- BSP Tree는 임의 평면으로 분할하여 정적 가시성 판정에 강점
- 구축 및 유지 비용이 매우 높아 동적 장면에 부적합
- 현대 게임 엔진에서는 BSP 대신 BVH 기반 방식으로 전환하는 추세

### 🎯 7.3 결론: 왜 BVH가 주류인가

1. **객체 중복 없음**: 메모리 효율적이고 구현이 깔끔
2. **동적 갱신 용이**: Refitting으로 프레임마다 빠르게 갱신 가능
3. **GPU 병렬화**: LBVH/HLBVH로 GPU에서 밀리초 단위 구축 가능
4. **하드웨어 지원**: NVIDIA RT Core가 BVH 순회를 하드웨어로 가속
5. **범용성**: 레이트레이싱, 충돌 감지, 공간 질의 등 다양한 용도에 동일 구조 사용 가능

---

## 📌 8. 결론

BVH(Bounding Volume Hierarchy)는 3D 컴퓨팅의 핵심 가속 구조로, "부모 볼륨이 교차하지 않으면 자식 전체를 가지치기한다"는 단순하면서도 강력한 원리에 기반한다.

**핵심 요약:**

- **바운딩 볼륨**: AABB가 비용 대비 효과에서 가장 실용적이며 산업 표준으로 자리잡았다
- **구축 알고리즘**: 정적 장면에서는 SAH 기반 Top-down이 최고 품질을, 동적/실시간 환경에서는 LBVH/HLBVH가 GPU 병렬 구축으로 최적의 속도를 제공한다
- **순회 알고리즘**: Slab Method 기반 Ray-AABB 교차 테스트와 가까운 자식 우선 순회가 핵심이며, GPU에서는 스택리스 순회가 필수적이다
- **최적화**: Refitting, 비동기 재구축, Wide BVH, 캐시 친화적 메모리 배치 등으로 실전 성능을 극대화한다
- **활용**: NVIDIA RTX 하드웨어 레이트레이싱, Unity/Unreal 물리 엔진, Pixar/Weta 영화 프로덕션 렌더러, CAD/BIM 클래시 감지까지 3D 컴퓨팅 전반에 걸쳐 BVH가 사실상의 표준 가속 구조이다

BVH는 앞으로도 실시간 레이트레이싱의 대중화, GPU 컴퓨팅 능력의 확대, 그리고 하드웨어 가속의 진화와 함께 그 중요성이 더욱 커질 것이다.

---

## 🔗 9. 참고 자료

### 🔹 서적 및 교재
- [Physically Based Rendering (PBR Book) - BVH Chapter](https://www.pbr-book.org/3ed-2018/Primitives_and_Intersection_Acceleration/Bounding_Volume_Hierarchies)
- [University of Illinois CS 418 - BVH Lecture](https://cs418.cs.illinois.edu/website/text/bvh.html)
- [Scratchapixel - Introduction to Acceleration Structures (BVH)](https://www.scratchapixel.com/lessons/3d-basic-rendering/introduction-acceleration-structure/bounding-volume-hierarchy-BVH-part2.html)
- [University of Freiburg - Bounding Volume Hierarchies (Lecture Notes)](https://cg.informatik.uni-freiburg.de/course_notes/sim_06_boundingVolumeHierarchies.pdf)

### 🔹 논문 및 기술 문서
- [Fast Parallel Construction of High-Quality BVH (Karras 2013, NVIDIA)](https://research.nvidia.com/sites/default/files/pubs/2013-07_Fast-Parallel-Construction/karras2013hpg_paper.pdf)
- [HLBVH: Hierarchical LBVH Construction for Real-Time (NVIDIA 2010)](https://research.nvidia.com/sites/default/files/pubs/2010-06_HLBVH-Hierarchical-LBVH/HLBVH-final.pdf)
- [Fast BVH Construction on GPUs (Lauterbach et al. 2009)](https://luebke.us/publications/eg09.pdf)
- [Maximizing Parallelism in BVH Construction (Karras 2012, NVIDIA)](https://research.nvidia.com/sites/default/files/pubs/2012-06_Maximizing-Parallelism-in/karras2012hpg_paper.pdf)
- [Parallel Reinsertion for BVH Optimization](https://dcgi.fel.cvut.cz/projects/prbvh/prbvh_eg.pdf)
- [Revising Apetrei's BVH Construction for Stackless Traversal (arXiv 2024)](https://arxiv.org/html/2402.00665)
- [Implicit Bounding Volumes and BVH (Stanford Dissertation)](http://graphics.stanford.edu/~anguyen/papers/bvh-anguyen.pdf)
- [BVH Construction (Lund University Project Report)](https://fileadmin.cs.lth.se/cs/Education/EDAN35/projects/2022/Sanden-BVH.pdf)

### 🔹 NVIDIA 공식 자료
- [NVIDIA OptiX Ray Tracing Engine](https://developer.nvidia.com/rtx/ray-tracing/optix)
- [Flexible and Powerful Ray Tracing with OptiX 8](https://developer.nvidia.com/blog/flexible-and-powerful-ray-tracing-with-optix-8/)
- [NVIDIA OptiX Ray Tracing Powered by RTX](https://developer.nvidia.com/blog/nvidia-optix-ray-tracing-powered-rtx/)
- [Thinking Parallel, Part II: Tree Traversal on the GPU](https://developer.nvidia.com/blog/thinking-parallel-part-ii-tree-traversal-gpu/)
- [Thinking Parallel, Part III: Tree Construction on the GPU](https://developer.nvidia.com/blog/thinking-parallel-part-iii-tree-construction-gpu/)
- [RTX Beyond Ray Tracing (HPG 2019)](https://www.sci.utah.edu/~will/papers/rtx-points-hpg19.pdf)
- [RTNN: Accelerating Neighbor Search Using Hardware Ray Tracing](https://arxiv.org/pdf/2201.01366)

### 🔹 게임 엔진 / 실무
- [Unity Physics - Simulation Pipeline (BVH)](https://docs.unity3d.com/Packages/com.unity.physics@1.3/manual/concepts-simulation.html)
- [Unity Forum - BVH in Unity Physics](https://discussions.unity.com/t/anyone-knows-what-kind-of-bvh-is-used-in-unitys-new-physics/750753)
- [Collision Detection Techniques (GarageFarm)](https://garagefarm.net/blog/collision-detection-in-2d-and-3d-concepts-and-techniques)

### 🎮 영화 VFX / 프로덕션 렌더러
- [Pixar RenderMan: An Advanced Path-Tracing Architecture](https://dl.acm.org/doi/10.1145/3182162)
- [Pixar RenderMan XPU (HPG 2025)](https://graphics.pixar.com/library/RenderManXPU/paper.pdf)
- [Weta Digital Manuka Renderer](https://dl.acm.org/doi/10.1145/3182161)
- [Accelerating Blender Cycles Using NVIDIA RTX](https://code.blender.org/2019/07/accelerating-cycles-using-nvidia-rtx/)

### 🏢 CAD/BIM
- [Spatial.com - What is Clash Detection](https://www.spatial.com/glossary/what-is-clash-detection)
- [Autodesk - BIM Clash Detection Guide](https://www.autodesk.com/blogs/construction/bim-clash-detection/)

### 🧠 알고리즘 구현 / 튜토리얼
- [Jacco's Blog - How to Build a BVH (시리즈)](https://jacco.ompf2.com/2022/04/13/how-to-build-a-bvh-part-1-basics/)
- [Jacco's Blog - BVH Part 9a: To the GPU](https://jacco.ompf2.com/2022/06/03/how-to-build-a-bvh-part-9a-to-the-gpu/)
- [Medium - SAH BVH Accelerators](https://medium.com/@bromanz/how-to-create-awesome-accelerators-the-surface-area-heuristic-e14b5dec6160)
- [Medium - Ray-AABB Intersection for BVH Traversal](https://medium.com/@bromanz/another-view-on-the-classic-ray-aabb-intersection-algorithm-for-bvh-traversal-41125138b525)
- [Fast Branchless Ray/Bounding Box Intersections](https://tavianator.com/2011/ray_box.html)
- [Python BVH with SAH Construction](https://python.plainenglish.io/constructs-bounding-volume-hierarchy-bvh-with-surface-area-heuristic-sah-in-python-89c14afb2f03)
- [GitHub - kt_bvh: SAH BVH2/4 Builder](https://github.com/karltechno/kt_bvh)
- [GitHub - LBVH: Parallel Linear BVH on GPU](https://github.com/ToruNiina/lbvh)

### ⚖️ 공간 분할 비교
- [Wikipedia - Bounding Volume Hierarchy](https://en.wikipedia.org/wiki/Bounding_volume_hierarchy)
- [GitHub Wiki - Comparing Spatial Partitioning Methods](https://github.com/SlaggyWolfie/slaggy-engine/wiki/Comparing-Spatial-Partitioning-Methods-%7C-Octree,-Kd-tree,-&-BSP-tree)
- [ResearchGate - BVH vs KD-Trees on Many-Core Architectures](https://www.researchgate.net/publication/282678485_Bounding_volume_hierarchies_versus_Kd-trees_on_contemporary_many-core_architectures)
- [TU Wien - Spatial Acceleration Structures (Lecture)](https://www.cg.tuwien.ac.at/sites/default/files/course/4411/attachments/05_spatial_acceleration_0.pdf)
- [EmergentMind - BVH Topic Overview](https://www.emergentmind.com/topics/bounding-volume-hierarchy-bvh)

---

```text
리서치 요청
md 파일 생성
백그라운드에서 sub agent 로 진행

BVH
- Bounding Volume Hierachy
- 객체들의 바운딩 볼륨의 트리 구조
- 부모 볼륨이 교차 하지 않으면 자식 전체를 가지치기
- 구체적인 알고리즘 ? 사례?
```
