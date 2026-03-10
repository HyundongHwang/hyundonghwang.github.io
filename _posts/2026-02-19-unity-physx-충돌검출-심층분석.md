---
layout: post
title: 260219 Unity PhysX 충돌검출 심층분석
comments: true
tags:
- unity
- burst
- job-system
- ecs
- performance
- optimization
- gpu
- physx
- gjk
- api
---
# 🔍 260219 Unity PhysX 충돌검출 심층분석

> 📝 **작성일**: 2026-02-19
> 💡 **분류**: Game Physics / Collision Detection / Unity Engine
> 💡 **키워드**: Unity, PhysX, Collision Detection, Broad Phase, Narrow Phase, CCD, GJK, EPA, Collider

---

## 📚 목차

1. [PhysX 엔진 개요](#1-physx-엔진-개요)
2. [충돌 검출 아키텍처 (Collision Detection Pipeline)](#2-충돌-검출-아키텍처)
3. [Unity Collider 종류 및 특성](#3-unity-collider-종류-및-특성)
4. [Unity 충돌 검출 API](#4-unity-충돌-검출-api)
5. [충돌 검출 모드 및 설정](#5-충돌-검출-모드-및-설정)
6. [성능 최적화 기법](#6-성능-최적화-기법)
7. [고급 기능 및 활용 사례](#7-고급-기능-및-활용-사례)
8. [실제 활용 패턴](#8-실제-활용-패턴)
9. [참고 자료](#9-참고-자료)

---

## 🧭 1. PhysX 엔진 개요

### 🧠 1.1 NVIDIA PhysX란?

NVIDIA PhysX는 NVIDIA가 개발하고 유지보수하는 오픈소스 실시간 물리 시뮬레이션 엔진이다. 원래 NovodeX라는 이름으로 개발되었으며, 2004년 Ageia에 인수된 후 2008년 NVIDIA에 최종 인수되었다. 현재 PhysX SDK 5.x까지 발전하였으며, BSD-3 라이선스로 오픈소스화되어 있다.

PhysX는 다음과 같은 핵심 기능을 제공한다.

- **강체 역학(Rigid Body Dynamics)**: 질량, 관성, 힘, 토크 기반 시뮬레이션
- **충돌 검출(Collision Detection)**: Broad Phase / Narrow Phase 이중 구조
- **조인트 시스템(Joints)**: 다양한 물리적 연결 구속 조건
- **천 시뮬레이션(Cloth)**: 직물 물리
- **파티클 시스템(Particles)**: 유체 및 입자 시뮬레이션
- **차량 물리(Vehicle)**: 전용 차량 역학 모델
- **SDF(Signed Distance Field) 충돌**: PhysX 5.x에서 추가된 비볼록 형상 지원

> 💡 참고: [PhysX SDK - NVIDIA Developer](https://developer.nvidia.com/physx-sdk), [PhysX - Wikipedia](https://en.wikipedia.org/wiki/PhysX)

### 🕰️ 1.2 Unity에서의 PhysX 통합 역사

Unity 엔진은 초기부터 NVIDIA PhysX를 기본 3D 물리 엔진으로 채택하였다. 주요 버전별 통합 이력은 다음과 같다.

| Unity 버전 | PhysX 버전 | 주요 변화 |
|---|---|---|
| Unity 4.x | PhysX 2.x | 초기 통합, 기본적인 충돌/물리 |
| Unity 5.0 (2015) | PhysX 3.3 | 대규모 아키텍처 업그레이드, 새로운 Solver |
| Unity 2018.2 | PhysX 3.4.1 | 실험적 빌드로 제공, PCM 접촉 생성 도입 |
| Unity 2019.3 (2019) | PhysX 4.1 | 새로운 Broad Phase(ABP), Temporal GI Solver, Speculative CCD |
| Unity 2022.x | PhysX 4.x (개선) | ContactModifyEvent, 개선된 Contact Dispatch |
| Unity 6 (6000.x) | PhysX 4.x 기반 | 안정화 및 최적화, Physics 설정 UI 개편 |

PhysX 3.4에서 4.1로의 전환(Unity 2019.3)이 가장 큰 도약이었다. 이 업그레이드를 통해 ABP(Automatic Box Pruning) Broad Phase 알고리즘, Temporal Gauss-Seidel(TGS) Solver, Speculative CCD 등 핵심 기능이 추가되었다.

> 💡 참고: [Physics updates in Unity 2019.3 - Unity Blog](https://blog.unity.com/technology/physics-updates-in-unity-2019-3), [Unity Forum - PhysX 4.x Switch](https://discussions.unity.com/t/did-unity-switch-to-physx-4-x-sdk/925300)

### 🧠 1.3 Unity 6에서의 PhysX 현황

Unity 6(6000.x 시리즈)에서는 PhysX가 여전히 기본이자 유일한 내장 3D 물리 엔진이다. Edit > Project Settings > Physics > GameObject SDK에서 PhysX만 선택 가능하다. PhysX 5.x로의 전환은 아직 Unity 메인 릴리스에 포함되지 않았으나, 커뮤니티에서는 PhysX 5.6과 Unity를 통합하는 실험적 시도가 진행되고 있다.

PhysX 5.x가 가져올 주요 변화는 다음과 같다.

- **PABP(Parallel Automatic Box Pruning)**: 멀티스레드 Broad Phase
- **GPU Broadphase**: GPU 가속 Sweep-and-Prune
- **SDF 충돌**: Convex Decomposition 없이 비볼록 형상 직접 충돌 지원
- **Speculative + Sweep CCD 동시 활성화**: 두 CCD 방식의 하이브리드 사용

> 💡 참고: [PhysX 5.4.0 Documentation](https://nvidia-omniverse.github.io/PhysX/physx/5.4.0/docs/RigidBodyCollision.html), [Unity 6 PhysX Discussion](https://discussions.unity.com/t/unity-6-changed-physics-engine-into-physx-and-now-its-really-buggy/947677)

### ⚖️ 1.4 PhysX vs Havok (Unity Physics) 비교

Unity는 ECS/DOTS 기반 프로젝트를 위해 두 가지 대안적 물리 엔진을 제공한다.

| 비교 항목 | Built-in PhysX | Unity Physics (ECS) | Havok Physics for Unity |
|---|---|---|---|
| **아키텍처** | MonoBehaviour 기반 | ECS/DOTS 기반 | ECS/DOTS 기반 |
| **상태 관리** | 상태 유지(Stateful) | 상태 비저장(Stateless) | 상태 유지(Stateful) |
| **성능 특성** | 단일 스레드 위주 | Burst/Job System 최적화 | 지능적 캐싱, AAA급 안정성 |
| **결정론성** | 비결정론적 | 결정론적(Deterministic) | 결정론적 |
| **적합 시나리오** | 일반적 게임 개발 | 네트워크 동기화, 커스터마이징 | 대규모 오픈월드, AAA 타이틀 |
| **데이터 프로토콜** | 독자적 | 공통 ECS 프로토콜 | 공통 ECS 프로토콜 |
| **라이선스** | Unity에 포함 | Unity에 포함 | 별도 라이선스 |

Unity Physics와 Havok Physics는 동일한 ECS 데이터 프로토콜을 공유하므로, 게임 코드나 컨텐츠를 재작성하지 않고도 두 엔진 간 전환이 가능하다. Unity Physics는 상태 비저장 방식으로 네트워크 시나리오에 유리하고, Havok은 상태 유지 기반의 지능적 캐싱으로 대규모 씬에서 우수한 성능을 보인다.

> 💡 참고: [Unity Physics Design Philosophy](https://docs.unity3d.com/Packages/com.unity.physics@1.0/manual/design.html), [Physics for ECS - Unity Learn](https://learn.unity.com/tutorial/ecs-physics-havok-physics-for-unity-and-unity-physics)

---

## 🏗️ 2. 충돌 검출 아키텍처

### 🏗️ 2.1 전체 파이프라인 구조

PhysX의 충돌 검출은 크게 세 단계로 구성된다.

```
[Simulation Step]
      |
      v
+------------------+
| 1. Broad Phase   |  -- AABB 기반 대략적 겹침 검출 (후보 쌍 생성)
+------------------+
      |
      v
+------------------+
| 2. Narrow Phase  |  -- 정밀 형상 교차 검사 (실제 접촉점 계산)
+------------------+
      |
      v
+------------------+
| 3. Resolution    |  -- 접촉 응답 (힘 계산, 위치 보정)
+------------------+
      |
      v
[Integration]       -- 속도/위치 업데이트
```

이 파이프라인은 매 Physics 시뮬레이션 스텝(기본 Fixed Timestep 0.02초, 즉 50Hz)마다 실행된다.

### 🔹 2.2 Broad Phase (넓은 범위 검출)

Broad Phase의 목적은 씬 내 모든 Collider의 AABB(Axis-Aligned Bounding Box)를 비교하여, 실제로 겹칠 가능성이 있는 **후보 쌍(Candidate Pairs)**을 빠르게 걸러내는 것이다. 이 단계에서는 정밀한 형상 교차 검사를 수행하지 않는다.

PhysX가 지원하는 Broad Phase 알고리즘은 다음과 같다.

#### ▫️ 2.2.1 SAP (Sweep and Prune) - `eSAP`

가장 전통적인 알고리즘으로, 각 축을 따라 AABB의 시작점과 끝점을 정렬한 후 순차적으로 순회하며 겹침을 검출한다.

- **장점**: 많은 오브젝트가 Sleep 상태일 때 매우 효율적 (증분 업데이트)
- **단점**: 모든 오브젝트가 동시에 움직이거나, 대량의 오브젝트가 추가/제거될 때 성능이 급격히 저하
- **적합 시나리오**: 정적 오브젝트가 대다수이고, 움직이는 오브젝트가 소수인 경우

#### ▫️ 2.2.2 MBP (Multi-Box Pruning) - `eMBP`

PhysX 3.3에서 도입된 알고리즘으로, 공간을 여러 영역(Region)으로 분할하고 각 영역 내에서 독립적으로 Pruning을 수행한다.

- **장점**: SAP의 성능 저하 문제를 해결, 모든 오브젝트가 움직이는 경우에도 안정적
- **단점**: 사용자가 수동으로 월드 바운드(Broadphase Region)를 정의해야 함
- **적합 시나리오**: 대규모 동적 씬, 월드 크기가 예측 가능한 경우

#### ▫️ 2.2.3 ABP (Automatic Box Pruning) - `eABP`

PhysX 4.0에서 도입되었으며, MBP의 성능 이점을 유지하면서 SAP의 편의성(자동 월드 바운드 관리)을 결합한 알고리즘이다.

- **장점**: MBP 수준의 성능 + 자동 월드 바운드 관리
- **단점**: MBP 대비 약간의 오버헤드 (자동 관리 비용)
- **적합 시나리오**: 대부분의 일반적인 게임 씬 (PhysX 4.x+ 기본값으로 권장)

**Unity 2019.3 이후 ABP가 기본 Broad Phase 알고리즘**으로 설정되어 있으며, Project Settings > Physics에서 변경할 수 있다.

#### ▫️ 2.2.4 PABP (Parallel Automatic Box Pruning) - `ePABP`

PhysX 5.0에서 도입된 ABP의 멀티스레드 변형이다.

- **장점**: 대규모 씬에서 병렬 처리를 통한 성능 향상
- **단점**: 메모리 사용량 증가
- **적합 시나리오**: 수천~수만 개의 동적 오브젝트가 존재하는 대규모 시뮬레이션

#### 🎮 2.2.5 GPU Broadphase

PhysX 5.x에서 제공하는 GPU 가속 증분 Sweep-and-Prune 방식이다. 전통적 SAP의 이점과 GPU 병렬 처리의 장점을 결합한다.

> 💡 참고: [PhysX 5.4.0 - Rigid Body Collision](https://nvidia-omniverse.github.io/PhysX/physx/5.4.0/docs/RigidBodyCollision.html)

### 🔹 2.3 Narrow Phase (좁은 범위 검출)

Broad Phase에서 생성된 후보 쌍에 대해, 실제 형상 간의 정밀 교차 검사를 수행하고 접촉점(Contact Point)을 생성하는 단계이다.

#### 🧠 2.3.1 PCM (Persistent Contact Manifold) - PhysX 기본값

PhysX 3.4 이후 기본 Narrow Phase 알고리즘으로 채택된 **PCM(Persistent Contact Manifold)**은 거리 기반 충돌 검출 시스템이다.

```
PCM 동작 원리:
1. GJK 알고리즘으로 두 형상 간 최단 거리 또는 관통 깊이 계산
2. EPA 알고리즘으로 관통 시 최소 분리 벡터 계산
3. 접촉점 매니폴드를 프레임 간 재활용 (Persistent)
4. 새 프레임에서는 기존 접촉점 유효성 검증 후 필요시만 재계산
```

PCM의 핵심 장점은 프레임 간 접촉점을 재활용하여 Narrow Phase 비용을 크게 절감하는 것이다. Unity에서는 `PxSceneFlag::eENABLE_PCM`으로 제어된다.

> 💡 참고: [PhysX 5.4.1 - Advanced Collision Detection](https://nvidia-omniverse.github.io/PhysX/physx/5.4.1/docs/AdvancedCollisionDetection.html)

#### 🧠 2.3.2 GJK (Gilbert-Johnson-Keerthi) 알고리즘

두 볼록 형상(Convex Shape) 간의 최단 거리를 반복적으로 계산하는 알고리즘이다.

```
GJK 핵심 원리:
- 두 형상의 민코프스키 차(Minkowski Difference)를 직접 계산하지 않음
- 대신, 서포트 함수(Support Function)를 이용하여 심플렉스(Simplex)를 반복 구축
- 심플렉스가 원점을 포함하면 두 형상이 교차 -> EPA로 전환
- 심플렉스가 원점을 포함하지 않으면 비교차 -> 최단 거리 반환
```

GJK는 임의의 볼록 형상에 대해 일반적으로 적용 가능하며, 반복 횟수가 O(1)~O(n) 수준으로 매우 빠르다. PhysX는 내부적으로 Primitive Collider 쌍에 대해 전용 최적화된 GJK 변형을 사용한다.

#### ▫️ 2.3.3 EPA (Expanding Polytope Algorithm)

GJK가 두 형상의 교차를 확인한 후, 관통 깊이(Penetration Depth)와 관통 방향(Penetration Normal)을 계산하는 알고리즘이다.

```
EPA 동작 흐름:
1. GJK로부터 원점을 포함하는 심플렉스를 전달받음
2. 심플렉스를 폴리토프(Polytope)로 확장
3. 폴리토프의 면 중 원점에 가장 가까운 면을 찾음
4. 해당 면 방향으로 새로운 서포트 포인트를 추가하여 폴리토프 확장
5. 수렴할 때까지 반복 -> 최소 관통 벡터(MTV) 산출
```

EPA의 결과인 최소 관통 벡터(Minimum Translation Vector)는 충돌 응답(Collision Resolution) 단계에서 오브젝트를 분리하는 데 직접 사용된다.

#### 🧠 2.3.4 SAT (Separating Axis Theorem)

두 볼록 형상 사이에 분리축(Separating Axis)이 존재하면 두 형상은 교차하지 않는다는 정리에 기반한 알고리즘이다.

- PhysX의 **레거시 Narrow Phase**(PCM 이전)에서 주로 사용
- Box-Box, Box-Convex 등 특정 형상 쌍에 대해 전용 최적화 구현
- PCM 도입 이후에도 일부 특수 케이스에서 내부적으로 활용

> 💡 참고: [Toptal - Video Game Physics Part II](https://www.toptal.com/game/video-game-physics-part-ii-collision-detection-for-solid-objects), [Collision detection with SAT, GJK and EPA](https://glushchenkoblog.wordpress.com/2017/08/29/primitives-collision-detection-with-sat-gjk-and-epa/)

### 🧠 2.4 Collision Filtering (충돌 필터링)

PhysX는 3단계 필터링 시스템을 구현한다. 가장 효율적인 것부터 순서대로 나열하면 다음과 같다.

1. **PxPairFilteringMode**: Broadphase 단계에서 직접 키네마틱 쌍을 필터링. KEEP, SUPPRESS, KILL 모드 지원
2. **PxSimulationFilterShader**: 사용자 정의 C 함수로 Broadphase 쌍을 처리. 외부 메모리 접근 불가하여 GPU 실행에 적합
3. **PxSimulationFilterCallback**: 가장 유연하지만 가장 느림. Broadphase 이후 실행되며 전체 메모리 접근 가능

Unity에서는 이 3단계 시스템이 **Layer Collision Matrix**로 추상화되어 제공된다.

### 🔹 2.5 Contact Manifold 및 Collision Resolution

Narrow Phase에서 생성된 접촉점 집합(Contact Manifold)은 Solver로 전달되어 충돌 응답에 사용된다.

```
Contact Manifold -> Solver 처리 흐름:
1. 접촉점별 법선(Normal), 관통 깊이(Depth), 위치(Position) 정보 수집
2. 구속 조건(Constraint) 생성: 접촉 법선 방향 구속 + 마찰 구속
3. Solver 반복(Iteration): 구속 조건 만족하도록 속도/위치 보정
   - PGS (Projected Gauss-Seidel): 기본 Solver
   - TGS (Temporal Gauss-Seidel): PhysX 4.x+, 더 안정적
4. 최종 속도/위치 적분(Integration)
```

Unity에서 Solver Type은 Project Settings > Physics에서 선택 가능하다. TGS Solver는 조인트 안정성과 스태킹(Stacking) 시나리오에서 PGS보다 우수한 성능을 보인다.

---

## 💻 3. Unity Collider 종류 및 특성

### ⚡ 3.1 성능 순위별 Collider 분류

Unity 공식 문서에서 제시하는 Collider 성능 순위(가장 효율적인 순)는 다음과 같다.

| 순위 | Collider 타입 | 특성 | 권장 사용 |
|---|---|---|---|
| 1 | **Sphere Collider** | 가장 단순하고 효율적 | 둥근 오브젝트, 범용 상호작용 |
| 2 | **Capsule Collider** | Sphere보다 약간 복잡하지만 효율적 | 캐릭터, 기둥형 오브젝트 |
| 3 | **Box Collider** | 효율적이고 유연함 | 직육면체, 블록형 오브젝트 |
| 4 | **Convex Mesh Collider** | Primitive보다 비용 높음 | 복잡한 형상, 동적 오브젝트 |
| 5 | **Non-Convex Mesh Collider** | 가장 비용이 높음 | 정적 환경 지오메트리만 |

> 💡 참고: [Unity Manual - Collider types and performance](https://docs.unity3d.com/2022.3/Documentation/Manual/physics-optimization-cpu-collider-types.html)

### 💻 3.2 Primitive Colliders

**Box Collider**
- 6면 검사로 교차 판정, 매우 빠름
- Size, Center 속성으로 크기/위치 조절
- 건물, 벽, 상자 등 직육면체 형태에 적합

**Sphere Collider**
- 중심점과 반지름만으로 교차 판정, 가장 빠름
- Radius, Center 속성
- 구형 오브젝트, 근접 감지 영역, 폭발 범위 등에 적합

**Capsule Collider**
- 원통 + 양쪽 반구로 구성
- Radius, Height, Direction, Center 속성
- 캐릭터 컨트롤러, 팔다리, 탄환 등에 적합

### 🎮 3.3 Mesh Collider

Mesh Collider는 3D 메시의 실제 폴리곤 형상을 충돌 영역으로 사용한다.

**Convex Mesh Collider** (`Is Convex = true`)
- 최대 255개 삼각형으로 제한
- 오목한 부분을 채워 볼록 껍질(Convex Hull) 생성
- 동적 Rigidbody에 부착 가능
- 다른 Convex/Primitive Collider와 충돌 가능

**Non-Convex Mesh Collider** (`Is Convex = false`)
- 메시의 실제 형상 그대로 사용
- **정적(Static) 또는 키네마틱(Kinematic) 오브젝트만 가능**
- 두 Non-Convex Mesh Collider 간 충돌 불가
- 면이 단방향(One-sided)이므로 내부에서 외부로 통과 가능

**Convex Decomposition (볼록 분해)**
복잡한 오목 형상을 여러 볼록 형상으로 분해하여 동적 오브젝트에서도 정밀한 충돌을 구현하는 기법이다.
- **V-HACD**: 고전적 볼록 분해 알고리즘
- **CoACD**: 최신 볼록 분해 알고리즘 (커뮤니티에서 Unity 플러그인 제공)

> 💡 참고: [Unity Manual - Mesh Collider](https://docs.unity3d.com/Manual/class-MeshCollider.html), [CoACD Unity Plugin](https://discussions.unity.com/t/free-better-convex-mesh-collider-generator-coacd/931964)

### 💻 3.4 Terrain Collider

Unity Terrain 시스템과 연동되는 전용 Collider이다.
- Terrain Data의 높이맵(Heightmap)을 기반으로 충돌 영역 생성
- 트리/디테일 오브젝트에는 별도 Collider 필요
- 내부적으로 높이맵 기반 최적화된 Narrow Phase 사용

### 💻 3.5 Wheel Collider

차량 물리 시뮬레이션 전용 Collider이다.
- 내장 서스펜션, 타이어 마찰 모델, 슬립 기반 물리 시뮬레이션
- 레이캐스트 기반 지면 검출 (실제 메시 형상이 아닌 레이 사용)
- `motorTorque`, `brakeTorque`, `steerAngle` 등 차량 전용 프로퍼티
- WheelHit 구조체로 접촉 정보 접근

### 💻 3.6 Compound Collider (복합 Collider) 전략

단일 Rigidbody에 여러 자식 Collider를 부착하여 복잡한 형상을 근사하는 방법이다.

```
[GameObject with Rigidbody]
    |-- [Child] Box Collider (몸통)
    |-- [Child] Sphere Collider (머리)
    |-- [Child] Capsule Collider (팔)
    |-- [Child] Capsule Collider (다리)
```

- Mesh Collider보다 성능이 우수하면서 형상 근사가 가능
- 동적 오목 형상에 대한 효과적 대안
- 히트박스 시스템 구현의 기본 패턴

---

## 🧠 4. Unity 충돌 검출 API

### 🧠 4.1 Collision 콜백 (물리적 충돌)

두 오브젝트가 물리적으로 충돌할 때 호출되는 콜백이다. 최소한 하나의 오브젝트에 Rigidbody가 부착되어 있어야 하고, 두 Collider 모두 `isTrigger = false`여야 한다.

```csharp
// 충돌 시작 시 1회 호출
void OnCollisionEnter(Collision collision)
{
    // collision.contacts: 접촉점 배열
    // collision.relativeVelocity: 상대 속도
    // collision.impulse: 충돌 충격량 (Unity 2018.3+)
    // collision.gameObject: 상대 오브젝트

    ContactPoint contact = collision.GetContact(0);
    Vector3 hitPoint = contact.point;
    Vector3 hitNormal = contact.normal;
    float separation = contact.separation;
}

// 충돌 지속 중 매 Fixed Update마다 호출
void OnCollisionStay(Collision collision) { }

// 충돌 종료 시 1회 호출
void OnCollisionExit(Collision collision) { }
```

**ContactPoint 구조체** 주요 필드:
- `point`: 접촉 위치 (World Space)
- `normal`: 접촉 법선 (첫 번째 Collider 기준 외부 방향)
- `separation`: 두 Collider 간 거리 (음수면 관통)
- `thisCollider` / `otherCollider`: 접촉에 관여한 Collider 참조

### 🔹 4.2 Trigger 콜백 (영역 감지)

Collider의 `isTrigger = true`일 때 사용되는 콜백이다. 물리적 충돌 응답은 발생하지 않고, 영역 진입/이탈만 감지한다.

```csharp
void OnTriggerEnter(Collider other)
{
    // other: 트리거 영역에 진입한 Collider
    if (other.CompareTag("Player"))
    {
        // 플레이어가 영역에 진입
    }
}

void OnTriggerStay(Collider other) { }
void OnTriggerExit(Collider other) { }
```

**주의사항**: Trigger 콜백이 발생하려면 두 오브젝트 중 최소 하나에 Rigidbody(또는 Kinematic Rigidbody)가 부착되어 있어야 한다.

### 💻 4.3 Physics Cast 계열 API

특정 형상을 특정 방향으로 투사(Cast)하여 경로상의 충돌을 검출하는 API이다.

```csharp
// Raycast: 직선 광선 투사
bool hit = Physics.Raycast(origin, direction, out RaycastHit hitInfo, maxDistance, layerMask);

// SphereCast: 구 형상 투사
bool hit = Physics.SphereCast(origin, radius, direction, out RaycastHit hitInfo, maxDistance);

// BoxCast: 박스 형상 투사
bool hit = Physics.BoxCast(center, halfExtents, direction, out RaycastHit hitInfo,
                           orientation, maxDistance, layerMask);

// CapsuleCast: 캡슐 형상 투사
bool hit = Physics.CapsuleCast(point1, point2, radius, direction, out RaycastHit hitInfo,
                                maxDistance, layerMask);
```

각 Cast API는 **NonAlloc 변형**을 제공하여 GC 할당을 방지한다.

```csharp
// NonAlloc: 미리 할당된 배열 재사용 (GC 프리)
RaycastHit[] results = new RaycastHit[10];
int count = Physics.RaycastNonAlloc(origin, direction, results, maxDistance, layerMask);
```

### 💻 4.4 Physics Overlap 계열 API

특정 영역 내 존재하는 모든 Collider를 검출하는 API이다.

```csharp
// OverlapSphere: 구 영역 내 모든 Collider 검출
Collider[] colliders = Physics.OverlapSphere(center, radius, layerMask);

// OverlapBox: 박스 영역 내 모든 Collider 검출
Collider[] colliders = Physics.OverlapBox(center, halfExtents, orientation, layerMask);

// OverlapCapsule: 캡슐 영역 내 모든 Collider 검출
Collider[] colliders = Physics.OverlapCapsule(point0, point1, radius, layerMask);

// NonAlloc 변형 (권장)
Collider[] buffer = new Collider[50];
int count = Physics.OverlapSphereNonAlloc(center, radius, buffer, layerMask);
```

> 💡 **성능 팁**: `Physics.OverlapSphere`는 매 호출마다 배열을 새로 할당하므로 GC 부담이 있다. 프레임마다 호출하는 경우 반드시 `Physics.OverlapSphereNonAlloc`을 사용해야 한다.

### 💻 4.5 고급 쿼리 API

```csharp
// ComputePenetration: 두 Collider 간 최소 분리 벡터 계산
bool overlapping = Physics.ComputePenetration(
    colliderA, positionA, rotationA,
    colliderB, positionB, rotationB,
    out Vector3 direction, out float distance
);
// 제약: 하나 이상의 Collider가 Box/Sphere/Capsule/Convex Mesh여야 함
// 용도: 커스텀 캐릭터 컨트롤러의 디페네트레이션(Depenetration)

// ClosestPoint: 특정 위치에서 Collider 표면의 최근접점 계산
Vector3 closest = Physics.ClosestPoint(point, collider, colliderPosition, colliderRotation);
// 위치가 Collider 내부/경계면이면 입력 위치 그대로 반환
```

> 💡 참고: [Unity API - Physics.ComputePenetration](https://docs.unity3d.com/ScriptReference/Physics.ComputePenetration.html), [Unity API - Physics.ClosestPoint](https://docs.unity3d.com/ScriptReference/Physics.ClosestPoint.html)

### 🗄️ 4.6 Layer Collision Matrix 및 QueryTriggerInteraction

**Layer Collision Matrix**는 Edit > Project Settings > Physics에서 설정하며, 어떤 레이어 간에 충돌을 허용할지를 행렬로 정의한다.

```
          Player  Enemy  Bullet  Environment  Trigger
Player      O       O      O        O           O
Enemy       O       X      O        O           O
Bullet      O       O      X        O           X
Environment O       O      O        X           O
Trigger     O       O      X        O           X
```

**QueryTriggerInteraction**은 Physics 쿼리(Raycast, Overlap 등)가 Trigger Collider를 어떻게 처리할지 제어한다.

```csharp
public enum QueryTriggerInteraction
{
    UseGlobal = 0,  // Physics.queriesHitTriggers 전역 설정 따름
    Ignore = 1,     // Trigger 무시
    Collide = 2     // Trigger 포함
}

// 사용 예: Trigger를 무시하는 Raycast
Physics.Raycast(ray, out hit, maxDist, layerMask, QueryTriggerInteraction.Ignore);
```

> 💡 참고: [Unity API - QueryTriggerInteraction](https://docs.unity3d.com/ScriptReference/QueryTriggerInteraction.html), [Unity Manual - Layer-based collision detection](https://docs.unity3d.com/6000.3/Documentation/Manual/LayerBasedCollision.html)

---

## 🛠️ 5. 충돌 검출 모드 및 설정

### 🔹 5.1 Discrete Collision Detection

기본 모드이며, 각 물리 시뮬레이션 스텝의 시작과 끝 위치에서만 충돌을 검사한다.

```
프레임 N           프레임 N+1
[A]------->        -----[A]
     [B]                [B]

터널링 발생: A가 B를 완전히 통과해 버림
```

- **성능**: 가장 가볍고 빠름
- **한계**: 고속 이동 오브젝트가 얇은 Collider를 통과(Tunneling)할 수 있음
- **적합 시나리오**: 저속 오브젝트, 두꺼운 Collider, 성능이 중요한 경우

### 🔹 5.2 Continuous Collision Detection (CCD)

고속 이동 오브젝트의 터널링을 방지하기 위해 물리 스텝 사이의 경로를 예측하여 충돌을 검출한다.

#### ▫️ 5.2.1 Sweep-based CCD

오브젝트의 이동 경로를 따라 형상을 Sweep하여 최초 충돌 시점을 정밀하게 계산한다.

**Continuous** 모드:
- Dynamic Rigidbody가 **Static Collider**와의 터널링만 방지
- Dynamic-Dynamic 간에는 Discrete로 처리
- Box, Sphere, Capsule Collider만 지원

**Continuous Dynamic** 모드:
- Dynamic Rigidbody가 **Static 및 Dynamic Collider** 모두와의 터널링 방지
- 가장 높은 정확도, 가장 높은 비용

PhysX 설정 파라미터:
- `ccdMaxPasses`: Sweep 반복 횟수 (기본 1, 높을수록 정밀하지만 비용 증가)
- `setMinCCDAdvanceCoefficient()`: CCD 전진 계수 (기본 0.15, 0~1 범위)
- `eENABLE_CCD_FRICTION`: CCD 중 마찰 적용 여부 (기본 비활성)

#### ▫️ 5.2.2 Speculative CCD (Continuous Speculative)

Sweep 대신 오브젝트의 선형/각속도를 기반으로 AABB를 확장하여, 다음 프레임에서 가능한 모든 접촉을 예측하는 방식이다.

```
일반 AABB:     확장된 AABB (Speculative):
+---+          +----------+
| A |          |          |
+---+          |   A ->   |
               |          |
               +----------+

이동 방향으로 AABB를 확장하여 잠재적 충돌 감지
```

- **장점**:
  - Sweep-based보다 저렴한 연산 비용
  - 빠른 각운동(회전)에 의한 누락 접촉도 감지
  - **키네마틱 액터에도 적용 가능** (Sweep CCD는 불가)
  - 모든 Collider 타입에서 작동
- **단점**:
  - Solver의 가속도에 의해 완전한 관통이 발생하면 실패할 수 있음
  - 물리적으로 완벽하게 정확한 결과를 보장하지 않음
  - "유령 충돌(Ghost Collision)" 현상 가능 (실제로 충돌하지 않을 오브젝트와 반응)

**Unity 권장**: Continuous Speculative가 대부분의 경우 **최적의 CCD 모드**이며, 유사하거나 더 나은 터널링 방지 성능을 더 낮은 비용으로 제공한다.

**PhysX 5.x 하이브리드**: PhysX 5.0부터 Sweep-based CCD와 Speculative CCD를 **동시에 활성화**할 수 있어, 두 방식의 장점을 결합할 수 있다.

> 💡 참고: [Unity Manual - Continuous collision detection](https://docs.unity3d.com/Manual/ContinuousCollisionDetection.html), [Unity Manual - Speculative CCD](https://docs.unity3d.com/2023.2/Documentation/Manual/speculative-ccd.html), [PhysX 5.4.1 - Advanced Collision Detection](https://nvidia-omniverse.github.io/PhysX/physx/5.4.1/docs/AdvancedCollisionDetection.html)

### ⚖️ 5.3 충돌 검출 모드 비교 요약

| 모드 | 터널링 방지 대상 | 지원 Collider | 비용 | 비고 |
|---|---|---|---|---|
| Discrete | 없음 | 모두 | 최저 | 기본값 |
| Continuous | Static만 | Box/Sphere/Capsule | 중간 | Sweep 기반 |
| Continuous Dynamic | Static + Dynamic | Box/Sphere/Capsule | 최고 | Sweep 기반 |
| Continuous Speculative | Static + Dynamic | 모두 | 중~저 | AABB 확장 기반 (권장) |

### 🧠 5.4 Fixed Timestep과 충돌 검출

Unity의 물리 시뮬레이션은 **Fixed Timestep**(기본 0.02초 = 50Hz)마다 실행된다.

```csharp
// Fixed Timestep 설정
Time.fixedDeltaTime = 0.02f; // 50Hz (기본값)
Time.fixedDeltaTime = 0.01f; // 100Hz (더 정밀하지만 2배 비용)

// 수동 시뮬레이션 제어
Physics.autoSimulation = false; // 자동 시뮬레이션 비활성화
Physics.Simulate(Time.fixedDeltaTime); // 수동으로 시뮬레이션 스텝 실행
```

Fixed Timestep을 줄이면 더 정밀한 충돌 검출이 가능하지만, 시뮬레이션 비용이 비례하여 증가한다. CCD를 사용하는 것이 Timestep을 줄이는 것보다 일반적으로 효율적이다.

---

## ⚡ 6. 성능 최적화 기법

### 🚀 6.1 Layer Collision Matrix 최적화

가장 비용이 낮고 효과가 큰 최적화 기법이다.

```
최적화 전: 모든 레이어 간 충돌 활성 (기본값)
-> N개 레이어에 대해 N*(N+1)/2 쌍 검사

최적화 후: 필요한 쌍만 활성
-> 불필요한 Broad Phase 후보 쌍 자체를 제거
```

**실천 가이드라인**:
- 프로젝트 초기에 레이어 구조를 설계
- "Player", "Enemy", "Bullet", "Environment", "Trigger", "UI" 등 명확한 레이어 분리
- 주기적으로 매트릭스를 검토하여 불필요한 활성 쌍 비활성화
- 동일 레이어 내 충돌이 불필요한 경우 대각선 체크 해제

> 💡 참고: [Unity Manual - Layer collision matrix optimization](https://docs.unity3d.com/6000.3/Documentation/Manual/physics-optimization-cpu-collision-layers.html)

### 💻 6.2 Collider 단순화

```
성능 비용 순서:
Sphere < Capsule < Box << Convex Mesh <<< Non-Convex Mesh

실전 지침:
- 동적 오브젝트: Primitive Collider 또는 Compound Collider 사용
- 정적 환경: Non-Convex Mesh Collider 허용
- 복잡한 동적 형상: Convex Decomposition 또는 Compound Collider
```

### ⚖️ 6.3 Static vs Dynamic Collider

```
Static Collider: Collider만 있고 Rigidbody 없음
-> PhysX가 정적 데이터 구조에 저장, 매우 효율적
-> 절대 Transform을 직접 이동시키지 말 것! (Broadphase 전체 재구축 유발)

Dynamic Collider: Collider + Rigidbody
-> Rigidbody를 통해 이동해야 최적 성능
-> MovePosition/MoveRotation 또는 AddForce 사용

Kinematic Collider: Collider + Rigidbody (isKinematic = true)
-> 코드로 직접 Transform 제어 가능
-> 움직이지만 물리 힘의 영향을 받지 않는 오브젝트용
```

**핵심 주의점**: Rigidbody 없이 Collider만 있는 오브젝트의 Transform을 매 프레임 변경하면, PhysX 내부에서 정적 지오메트리가 매번 재구축되어 심각한 성능 저하가 발생한다.

### 🏢 6.4 Rigidbody Sleep 상태 활용

PhysX는 운동 에너지가 임계값 이하로 떨어진 Rigidbody를 자동으로 Sleep 상태로 전환한다.

```csharp
// Sleep 관련 API
rigidbody.sleepThreshold = 0.005f; // Sleep 진입 임계값 (기본 0.005)
rigidbody.WakeUp();                // 강제 깨움
rigidbody.Sleep();                 // 강제 Sleep
bool isSleeping = rigidbody.IsSleeping();

// 최적화 팁: WakeUp()을 매 프레임 호출하지 말 것
// Sleep 상태의 Rigidbody는 Broad Phase에서 거의 비용 0
```

### 🎮 6.5 Physics.BakeMesh

런타임에 Mesh Collider를 생성하면 메시 쿠킹(Cooking) 과정에서 CPU 스파이크가 발생한다.

```csharp
// 사전 베이킹으로 런타임 스파이크 방지
Physics.BakeMesh(mesh.GetInstanceID(), convex: false);

// 빌드 시 자동 베이킹: Player Settings > Prebake Collision Meshes (기본 활성)
// 에디터에서 씬/프리팹 저장 시 자동 베이킹

// 프로시저럴 메시의 경우: 로딩 화면에서 BakeMesh 호출 권장
```

> 💡 참고: [Unity - Mesh.Bake.PhysX.CollisionData Profiler Spikes](https://www.unity3dtips.com/fix-mesh-bake-physx-collisiondata-unity/)

### 🛠️ 6.6 Broadphase 설정 튜닝

Unity Project Settings > Physics에서 Broad Phase 알고리즘을 변경할 수 있다.

| 설정 | 적합 시나리오 |
|---|---|
| Automatic Box Pruning (기본) | 대부분의 게임 |
| Sweep and Prune | 정적 오브젝트 대다수, 이동 오브젝트 소수 |
| Multi Box Pruning | 대규모 월드, 수동 영역 관리 가능 시 |

### ⚡ 6.7 Profiler를 이용한 물리 성능 분석

Unity Profiler의 Physics 모듈에서 확인할 수 있는 핵심 지표:

- **Physics.Processing**: 전체 물리 시뮬레이션 시간
- **Physics.Simulation**: 시뮬레이션 스텝 실행 시간
- **Broadphase**: Broad Phase 소요 시간
- **NarrowPhase**: Narrow Phase 소요 시간
- **Solver**: 충돌 응답 Solver 시간
- **Contacts**: 활성 접촉점 수
- **Active Rigidbodies**: 활성(비-Sleep) Rigidbody 수

Physics Debug Visualization(Window > Analysis > Physics Debugger)으로 Collider 형상, 접촉점, AABB를 시각적으로 확인할 수 있다.

### 🚀 6.8 ECS/DOTS 기반 최적화 (Unity Physics)

```
전통적 PhysX (MonoBehaviour):
- 단일 스레드 위주 처리
- GC 할당 발생 (콜백 기반)
- 비결정론적

Unity Physics (ECS):
- Burst Compiler로 네이티브 코드 수준 최적화
- Job System으로 멀티스레드 병렬 처리
- 결정론적 시뮬레이션
- GC 할당 없음 (NativeArray 기반)
```

대규모 물리 시뮬레이션이 필요한 경우 ECS 기반 Unity Physics로의 마이그레이션이 근본적 성능 개선을 가져올 수 있다. 2025년 기준 Broadphase의 증분 업데이트(Incremental BVH)가 도입되어, 대규모 씬에서 소수만 변경되는 경우 극적인 성능 향상이 가능하다.

> 💡 참고: [Unity Physics Changelog 1.3.14](https://docs.unity3d.com/Packages/com.unity.physics@1.3/changelog/CHANGELOG.html), [Physics development status - May 2025](https://discussions.unity.com/t/physics-development-status-and-next-milestones-may-2025/1637988)

---

## 🏢 7. 고급 기능 및 활용 사례

### 🔹 7.1 ContactModifyEvent (Unity 2022.2+)

Unity 2022.2에서 도입된 **Contact Modification API**는 PhysX의 접촉 수정 기능을 Unity 스크립트에서 직접 활용할 수 있게 해준다.

```csharp
// 1. Collider에 수정 가능 플래그 설정
collider.hasModifiableContacts = true;

// 2. 콜백 등록
Physics.ContactModifyEvent += OnContactModify;

// 3. 접촉 수정 콜백 구현
void OnContactModify(PhysicsScene scene, NativeArray<ModifiableContactPair> pairs)
{
    for (int i = 0; i < pairs.Length; i++)
    {
        ModifiableContactPair pair = pairs[i];
        for (int j = 0; j < pair.contactCount; j++)
        {
            // 접촉점 위치, 법선, 분리 거리 수정 가능
            pair.SetNormal(j, Vector3.up);
            pair.SetSeparation(j, 0.01f);
            pair.SetMaxImpulse(j, 100f);
        }
    }
}

// CCD 전용 버전도 제공
Physics.ContactModifyEventCCD += OnContactModifyCCD;
```

**성능**: 전통적인 OnCollisionEnter 콜백 대비 약 **4배 빠른** 접촉 데이터 읽기 성능을 제공한다. NativeArray를 통한 직접 접근이 마샬링 오버헤드를 제거하기 때문이다.

**활용 예시**:
- 컨베이어 벨트: 접촉점에 목표 속도 설정
- 단방향 플랫폼: 특정 방향의 접촉만 허용
- 소프트 충돌: 최대 임펄스 제한
- 커스텀 마찰 모델: 접촉별 마찰 계수 동적 변경

> 💡 참고: [Unity API - Physics.ContactModifyEvent](https://docs.unity3d.com/ScriptReference/Physics.ContactModifyEvent.html), [WaveAccess - Contacts Modification API](https://medium.com/waveaccess/contacts-modification-api-in-unity-d65deba6f7c)

### 🔹 7.2 Ragdoll 물리

래그돌은 캐릭터의 각 뼈(Bone)에 Rigidbody와 Collider를 부착하고, CharacterJoint로 연결하여 물리 기반 자연스러운 쓰러짐/날아감 애니메이션을 구현하는 기법이다.

```
래그돌 구성 요소:
- 각 주요 뼈마다: Rigidbody + Collider (보통 Capsule) + CharacterJoint
- 머리, 몸통, 상완, 전완, 허벅지, 종아리, 발 등 10~15개 노드

전환 패턴:
1. 일반 상태: Animator 활성, Rigidbody.isKinematic = true
2. 래그돌 상태: Animator 비활성, Rigidbody.isKinematic = false
3. 블렌딩: 래그돌 -> 애니메이션 부드러운 전환 (Active Ragdoll)
```

**주의사항**: 래그돌 Collider가 서로 겹치면 PhysX가 이를 분리하려 시도하면서 뼈가 비정상적으로 변형될 수 있다.

> 💡 참고: [Creating Ragdolls - Unity Learn](https://learn.unity.com/tutorial/creating-ragdolls-2019)

### 💻 7.3 차량 물리 (WheelCollider)

```csharp
[System.Serializable]
public class AxleInfo
{
    public WheelCollider leftWheel;
    public WheelCollider rightWheel;
    public bool motor;    // 구동축 여부
    public bool steering; // 조향축 여부
}

public class CarController : MonoBehaviour
{
    public List<AxleInfo> axleInfos;
    public float maxMotorTorque;
    public float maxSteeringAngle;

    void FixedUpdate()
    {
        float motor = maxMotorTorque * Input.GetAxis("Vertical");
        float steering = maxSteeringAngle * Input.GetAxis("Horizontal");

        foreach (AxleInfo info in axleInfos)
        {
            if (info.steering)
            {
                info.leftWheel.steerAngle = steering;
                info.rightWheel.steerAngle = steering;
            }
            if (info.motor)
            {
                info.leftWheel.motorTorque = motor;
                info.rightWheel.motorTorque = motor;
            }
        }
    }
}
```

WheelCollider는 내부적으로 레이캐스트 기반 지면 검출과 슬립 기반 타이어 마찰 모델을 사용하며, 서스펜션 시뮬레이션까지 내장되어 있다.

> 💡 참고: [Unity Manual - Create a vehicle with Wheel Colliders](https://docs.unity3d.com/2019.4/Documentation/Manual/WheelColliderTutorial.html)

### 🏢 7.4 BIM/건축 분야에서의 Clash Detection 활용

BIM(Building Information Modeling)에서의 Clash Detection은 건축 모델의 각 분야(건축, 구조, 기계, 전기, 배관 등) 3D 요소 간 물리적 간섭을 사전에 검출하는 프로세스이다.

**Unity PhysX를 BIM Clash Detection에 활용하는 접근법**:

```
BIM 모델 (IFC/Revit)
      |
      v
[Unity로 임포트] -- FBX/glTF/IFC 파서
      |
      v
[Collider 자동 생성] -- MeshCollider 또는 Compound Collider
      |
      v
[Physics.OverlapBox / ComputePenetration]
      |
      v
[Clash Report 생성] -- 간섭 위치, 관련 요소, 심각도 분류
```

**Clash 유형과 PhysX 활용**:
- **Hard Clash**: 두 요소가 물리적으로 겹침 -> `Physics.ComputePenetration`으로 관통 깊이 계산
- **Soft Clash**: 최소 이격 거리 미확보 -> `Physics.ClosestPoint`로 최근접 거리 계산
- **Clearance Clash**: 유지보수 접근 공간 부족 -> `Physics.OverlapBox`로 영역 내 간섭 검출

> 💡 참고: [BIM Clash Detection - Autodesk](https://www.autodesk.com/blogs/construction/bim-clash-detection/), [Clash Detection in BIM - United-BIM](https://www.united-bim.com/what-is-clash-detection-in-bim-process-benefits-and-future-scope-in-modern-day-aec-industry/)

### ⚖️ 7.5 ECS 기반 Unity Physics vs 기존 PhysX

```
Built-in PhysX:
+ 풍부한 API, 성숙한 에코시스템
+ WheelCollider, Cloth, Ragdoll 등 전용 기능
+ 대부분의 Unity 에셋/튜토리얼 호환
- 단일 스레드 병목
- GC 할당 (콜백)
- 비결정론적

Unity Physics (ECS):
+ Burst + Job System 병렬 처리
+ 결정론적 시뮬레이션 (네트워크 동기화)
+ GC 프리
+ 커스터마이징 용이 (소스 접근 가능)
- API 상대적 미성숙
- 전용 기능 부족 (래그돌, 차량 등 직접 구현 필요)
- 학습 곡선 높음
```

---

## 🏢 8. 실제 활용 패턴

### 🔹 8.1 히트박스 시스템 (Hit Detection)

격투/슈팅 게임에서의 공격 판정 시스템이다.

```csharp
public class HitboxSystem : MonoBehaviour
{
    [Header("히트박스 설정")]
    public LayerMask hitLayer;
    public float attackRadius = 1.0f;

    private Collider[] hitBuffer = new Collider[20];

    public void PerformAttack(Vector3 attackOrigin)
    {
        // NonAlloc으로 GC 방지
        int hitCount = Physics.OverlapSphereNonAlloc(
            attackOrigin, attackRadius, hitBuffer, hitLayer,
            QueryTriggerInteraction.Collide
        );

        for (int i = 0; i < hitCount; i++)
        {
            IDamageable target = hitBuffer[i].GetComponent<IDamageable>();
            if (target != null)
            {
                // 가장 가까운 접촉점 계산
                Vector3 hitPoint = Physics.ClosestPoint(
                    attackOrigin, hitBuffer[i],
                    hitBuffer[i].transform.position,
                    hitBuffer[i].transform.rotation
                );
                target.TakeDamage(10f, hitPoint);
            }
        }
    }
}
```

### 🔹 8.2 근접 감지 (Proximity Detection)

Trigger Collider를 이용한 영역 기반 감지 시스템이다.

```csharp
public class ProximityDetector : MonoBehaviour
{
    public float detectionRadius = 10f;
    private SphereCollider triggerZone;
    private HashSet<Transform> detectedObjects = new HashSet<Transform>();

    void Start()
    {
        triggerZone = gameObject.AddComponent<SphereCollider>();
        triggerZone.isTrigger = true;
        triggerZone.radius = detectionRadius;
    }

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Enemy"))
        {
            detectedObjects.Add(other.transform);
            OnEnemyDetected(other.transform);
        }
    }

    void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("Enemy"))
        {
            detectedObjects.Remove(other.transform);
            OnEnemyLost(other.transform);
        }
    }

    void OnEnemyDetected(Transform enemy) { /* 경고 UI 표시 등 */ }
    void OnEnemyLost(Transform enemy) { /* 경고 해제 등 */ }
}
```

### 🔹 8.3 지형 위 오브젝트 배치

Raycast를 이용한 지형 표면 스냅 배치 시스템이다.

```csharp
public class TerrainPlacer : MonoBehaviour
{
    public LayerMask terrainLayer;

    public bool PlaceOnTerrain(Vector3 worldPosition, out Vector3 placedPosition, out Quaternion placedRotation)
    {
        Ray ray = new Ray(worldPosition + Vector3.up * 100f, Vector3.down);

        if (Physics.Raycast(ray, out RaycastHit hit, 200f, terrainLayer))
        {
            placedPosition = hit.point;
            // 지형 법선에 맞춰 회전
            placedRotation = Quaternion.FromToRotation(Vector3.up, hit.normal);
            return true;
        }

        placedPosition = worldPosition;
        placedRotation = Quaternion.identity;
        return false;
    }
}
```

### 🔹 8.4 물리 기반 파괴 시스템

충돌 충격량 기반의 오브젝트 파괴 시스템이다.

```csharp
public class Destructible : MonoBehaviour
{
    public float destructionThreshold = 50f;
    public GameObject fracturedPrefab;

    void OnCollisionEnter(Collision collision)
    {
        // 충돌 충격량으로 파괴 여부 판단
        float impactForce = collision.impulse.magnitude / Time.fixedDeltaTime;

        if (impactForce > destructionThreshold)
        {
            // 파편 생성
            GameObject fractured = Instantiate(fracturedPrefab,
                transform.position, transform.rotation);

            // 파편에 충돌 힘 전달
            foreach (Rigidbody rb in fractured.GetComponentsInChildren<Rigidbody>())
            {
                rb.AddExplosionForce(impactForce * 0.5f,
                    collision.GetContact(0).point, 5f);
            }

            Destroy(gameObject);
        }
    }
}
```

### 🧠 8.5 대규모 오브젝트 충돌 처리 전략

수천 개 이상의 오브젝트가 존재하는 대규모 씬에서의 최적화 전략을 정리한다.

```
1. 공간 분할 (Spatial Partitioning)
   - Unity Physics의 BVH (Bounding Volume Hierarchy) 활용
   - 필요 시 커스텀 옥트리/그리드 구현

2. LOD 기반 물리 (Physics LOD)
   - 원거리: 충돌 검출 비활성 또는 단순 Collider
   - 중거리: Primitive Collider만 활성
   - 근거리: 정밀 Mesh Collider 활성

3. 분할 시뮬레이션
   - Physics.autoSimulation = false
   - 프레임별로 영역을 나누어 순차 시뮬레이션
   - 또는 Physics Scene 분리 (Multi-Scene Physics)

4. 배치 처리
   - OverlapNonAlloc 계열 API로 GC 최소화
   - 결과를 프레임에 분산 처리 (시간 분할)
```

---

## 🔗 9. 참고 자료

### 🔹 공식 문서

| 출처 | URL |
|---|---|
| Unity Manual - Built-in 3D Physics | [https://docs.unity3d.com/6000.3/Documentation/Manual/PhysicsOverview.html](https://docs.unity3d.com/6000.3/Documentation/Manual/PhysicsOverview.html) |
| Unity Manual - CCD | [https://docs.unity3d.com/Manual/ContinuousCollisionDetection.html](https://docs.unity3d.com/Manual/ContinuousCollisionDetection.html) |
| Unity Manual - Speculative CCD | [https://docs.unity3d.com/2023.2/Documentation/Manual/speculative-ccd.html](https://docs.unity3d.com/2023.2/Documentation/Manual/speculative-ccd.html) |
| Unity Manual - Collider Types Performance | [https://docs.unity3d.com/2022.3/Documentation/Manual/physics-optimization-cpu-collider-types.html](https://docs.unity3d.com/2022.3/Documentation/Manual/physics-optimization-cpu-collider-types.html) |
| Unity Manual - Layer Collision Matrix | [https://docs.unity3d.com/6000.3/Documentation/Manual/physics-optimization-cpu-collision-layers.html](https://docs.unity3d.com/6000.3/Documentation/Manual/physics-optimization-cpu-collision-layers.html) |
| Unity Manual - Mesh Collider | [https://docs.unity3d.com/6000.3/Documentation/Manual/mesh-colliders-introduction.html](https://docs.unity3d.com/6000.3/Documentation/Manual/mesh-colliders-introduction.html) |
| Unity Manual - Layer-based Collision | [https://docs.unity3d.com/6000.3/Documentation/Manual/LayerBasedCollision.html](https://docs.unity3d.com/6000.3/Documentation/Manual/LayerBasedCollision.html) |
| Unity API - Physics.ComputePenetration | [https://docs.unity3d.com/ScriptReference/Physics.ComputePenetration.html](https://docs.unity3d.com/ScriptReference/Physics.ComputePenetration.html) |
| Unity API - Physics.ClosestPoint | [https://docs.unity3d.com/ScriptReference/Physics.ClosestPoint.html](https://docs.unity3d.com/ScriptReference/Physics.ClosestPoint.html) |
| Unity API - ContactPoint | [https://docs.unity3d.com/ScriptReference/ContactPoint.html](https://docs.unity3d.com/ScriptReference/ContactPoint.html) |
| Unity API - QueryTriggerInteraction | [https://docs.unity3d.com/ScriptReference/QueryTriggerInteraction.html](https://docs.unity3d.com/ScriptReference/QueryTriggerInteraction.html) |
| Unity API - Physics.ContactModifyEvent | [https://docs.unity3d.com/ScriptReference/Physics.ContactModifyEvent.html](https://docs.unity3d.com/ScriptReference/Physics.ContactModifyEvent.html) |
| Unity Physics Package (ECS) | [https://docs.unity3d.com/Packages/com.unity.physics@1.0/manual/design.html](https://docs.unity3d.com/Packages/com.unity.physics@1.0/manual/design.html) |

### 🧠 NVIDIA PhysX 문서

| 출처 | URL |
|---|---|
| PhysX 5.4.0 - Rigid Body Collision | [https://nvidia-omniverse.github.io/PhysX/physx/5.4.0/docs/RigidBodyCollision.html](https://nvidia-omniverse.github.io/PhysX/physx/5.4.0/docs/RigidBodyCollision.html) |
| PhysX 5.4.1 - Advanced Collision Detection | [https://nvidia-omniverse.github.io/PhysX/physx/5.4.1/docs/AdvancedCollisionDetection.html](https://nvidia-omniverse.github.io/PhysX/physx/5.4.1/docs/AdvancedCollisionDetection.html) |
| PhysX 5.5.0 - Advanced Collision Detection | [https://nvidia-omniverse.github.io/PhysX/physx/5.5.0/docs/AdvancedCollisionDetection.html](https://nvidia-omniverse.github.io/PhysX/physx/5.5.0/docs/AdvancedCollisionDetection.html) |
| PhysX SDK - NVIDIA Developer | [https://developer.nvidia.com/physx-sdk](https://developer.nvidia.com/physx-sdk) |
| PhysX - Wikipedia | [https://en.wikipedia.org/wiki/PhysX](https://en.wikipedia.org/wiki/PhysX) |

### 🔹 Unity 블로그 및 커뮤니티

| 출처 | URL |
|---|---|
| Physics updates in Unity 2019.3 | [https://blog.unity.com/technology/physics-updates-in-unity-2019-3](https://blog.unity.com/technology/physics-updates-in-unity-2019-3) |
| Physics changes in Unity 2022.2 | [https://unity.com/blog/engine-platform/physics-changes-in-unity-2022-2](https://unity.com/blog/engine-platform/physics-changes-in-unity-2022-2) |
| Physics development status (May 2025) | [https://discussions.unity.com/t/physics-development-status-and-next-milestones-may-2025/1637988](https://discussions.unity.com/t/physics-development-status-and-next-milestones-may-2025/1637988) |
| ECS Physics vs PhysX Test | [https://discussions.unity.com/t/ecs-physics-vs-physx-test/948446](https://discussions.unity.com/t/ecs-physics-vs-physx-test/948446) |
| Unity Forum - PhysX 4.x Switch | [https://discussions.unity.com/t/did-unity-switch-to-physx-4-x-sdk/925300](https://discussions.unity.com/t/did-unity-switch-to-physx-4-x-sdk/925300) |
| Contacts Modification API (WaveAccess) | [https://medium.com/waveaccess/contacts-modification-api-in-unity-d65deba6f7c](https://medium.com/waveaccess/contacts-modification-api-in-unity-d65deba6f7c) |
| CoACD Convex Decomposition Plugin | [https://discussions.unity.com/t/free-better-convex-mesh-collider-generator-coacd/931964](https://discussions.unity.com/t/free-better-convex-mesh-collider-generator-coacd/931964) |
| ClosestPoint and ContactPoint Tutorial | [https://www.immersivelimit.com/tutorials/closestpoint-and-contactpoint-collision-in-unity](https://www.immersivelimit.com/tutorials/closestpoint-and-contactpoint-collision-in-unity) |

### 🧠 알고리즘 및 학술 자료

| 출처 | URL |
|---|---|
| Video Game Physics Part II (Toptal) | [https://www.toptal.com/game/video-game-physics-part-ii-collision-detection-for-solid-objects](https://www.toptal.com/game/video-game-physics-part-ii-collision-detection-for-solid-objects) |
| Collision detection with SAT, GJK and EPA | [https://glushchenkoblog.wordpress.com/2017/08/29/primitives-collision-detection-with-sat-gjk-and-epa/](https://glushchenkoblog.wordpress.com/2017/08/29/primitives-collision-detection-with-sat-gjk-and-epa/) |
| Building a Collision Engine - GJK | [https://blog.hamaluik.ca/posts/building-a-collision-engine-part-1-2d-gjk-collision-detection/](https://blog.hamaluik.ca/posts/building-a-collision-engine-part-1-2d-gjk-collision-detection/) |
| Newcastle University - Broad/Narrow Phase | [https://research.ncl.ac.uk/game/mastersdegree/gametechnologies/physicstutorials/6accelerationstructures/Physics%20-%20Spatial%20Acceleration%20Structures.pdf](https://research.ncl.ac.uk/game/mastersdegree/gametechnologies/physicstutorials/6accelerationstructures/Physics%20-%20Spatial%20Acceleration%20Structures.pdf) |
| PhysX Creator Blog - GJK/EPA History | [http://www.codercorner.com/blog/?p=1303](http://www.codercorner.com/blog/?p=1303) |

---

```text
리서치 요청
md 파일 생성
백그라운드에서 sub agent 로 진행

Unity PhysX 기반 충돌 검출 기능
```
