---
layout: post
title: 260205 unity burst compiler guide
comments: true
tags:
- unity
- burst
- job-system
- performance
- optimization
- research
- guide
- analysis
---
# 🛠️ 260205 Unity Burst Compiler 완벽 가이드

> 💡 고성능 C# 코드를 위한 LLVM 기반 최적화 컴파일러

---

## 📚 목차
1. [Burst 소개](#burst-소개)
2. [작동 원리](#작동-원리)
3. [장단점](#장단점)
4. [간단한 예제](#간단한-예제)
5. [실용적인 복잡한 예제](#실용적인-복잡한-예제)

---

## 🧭 Burst 소개

### 🚀 Burst란?

**Burst**는 Unity에서 제공하는 고성능 컴파일러로, **HPC# (High-Performance C#)** 이라 불리는 C#의 제한된 하위 집합을 **LLVM**을 사용하여 고도로 최적화된 네이티브 코드로 변환합니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Burst 컴파일 파이프라인                       │
└─────────────────────────────────────────────────────────────────┘

    C# 코드           IL 바이트코드         LLVM IR          네이티브 코드
   ┌─────────┐       ┌─────────┐        ┌─────────┐       ┌─────────┐
   │ HPC#    │ ───▶  │  .NET   │ ───▶   │  LLVM   │ ───▶  │  x86    │
   │ Subset  │       │   IL    │        │   IR    │       │  ARM    │
   └─────────┘       └─────────┘        └─────────┘       └─────────┘
                                              │
                                              ▼
                                     ┌─────────────────┐
                                     │  최적화 패스     │
                                     │  • 벡터화       │
                                     │  • SIMD 변환   │
                                     │  • 인라이닝     │
                                     └─────────────────┘
```

### ✨ 핵심 특징

| 특징 | 설명 |
|:---|:---|
| **LLVM 기반** | 업계 표준 컴파일러 인프라 사용 |
| **SIMD 최적화** | 단일 명령으로 다중 데이터 처리 |
| **Job System 통합** | 멀티스레드 병렬 처리 지원 |
| **AOT/JIT 지원** | 에디터(JIT)와 빌드(AOT) 모두 지원 |
| **크로스 플랫폼** | x86, ARM, WebGL 등 다양한 플랫폼 |

### ⚡ 성능 향상 사례

```
┌─────────────────────────────────────────────────────────────────┐
│                    실제 성능 비교 사례                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   일반 C# (Mono)     ████████████████████████████████  4000ms   │
│   .NET 최적화        ██                                  20ms   │
│   Burst 적용         ██                                  20ms   │
│                                                                 │
│   ─────────────────────────────────────────────────────────────│
│                                                                 │
│   Boids (300개)                                                 │
│   Update Loop        ██████████████████                  18ms   │
│   Job (No Burst)     ████████                           8.5ms   │
│   Job + Burst        █                                 0.47ms   │
│                                                                 │
│   ─────────────────────────────────────────────────────────────│
│                                                                 │
│   메시 처리 (80,000 vertices)                                   │
│   기존 방식          █████████████████████████            5ms   │
│   Burst + Job        █                                  0.1ms   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📌 작동 원리

### 🗂️ 컴파일 과정 상세

```
┌─────────────────────────────────────────────────────────────────┐
│                    Burst 컴파일 상세 과정                        │
└─────────────────────────────────────────────────────────────────┘

   1. C# 소스 코드
      │
      ▼
   2. Roslyn 컴파일러 → IL (Intermediate Language)
      │
      ▼
   3. Burst 분석
      ├── HPC# 규칙 검증
      ├── Managed 타입 체크
      └── 안전성 분석
      │
      ▼
   4. LLVM IR 변환
      │
      ▼
   5. LLVM 최적화 패스
      ├── Dead Code Elimination
      ├── Loop Unrolling
      ├── Auto-Vectorization (SIMD)
      ├── Instruction Combining
      └── Memory Access 최적화
      │
      ▼
   6. 네이티브 코드 생성
      ├── x86/x64 (SSE, AVX)
      ├── ARM (NEON)
      └── WebAssembly (WASM)
```

### 🚀 SIMD 최적화

**SIMD (Single Instruction Multiple Data)** 는 하나의 명령어로 여러 데이터를 동시에 처리하는 기술입니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                    SIMD 연산 비교                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   스칼라 연산 (일반):                                            │
│   ┌───┐   ┌───┐       ┌───┐                                    │
│   │ A │ + │ B │   =   │ C │    ← 1개 연산/명령어                │
│   └───┘   └───┘       └───┘                                    │
│                                                                 │
│   SIMD 연산 (float4):                                           │
│   ┌───┬───┬───┬───┐   ┌───┬───┬───┬───┐   ┌───┬───┬───┬───┐   │
│   │A1 │A2 │A3 │A4 │ + │B1 │B2 │B3 │B4 │ = │C1 │C2 │C3 │C4 │   │
│   └───┴───┴───┴───┘   └───┴───┴───┴───┘   └───┴───┴───┴───┘   │
│                            ↑                                    │
│                    4개 연산/명령어 (4배 빠름!)                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 🔹 Unity.Mathematics 타입

Burst와 함께 사용되는 수학 라이브러리로, SIMD 레지스터에 직접 매핑됩니다.

```csharp
// SIMD 최적화된 타입들
float4 position;    // 4개의 float을 하나의 SIMD 레지스터에
int4 indices;       // 4개의 int를 하나의 SIMD 레지스터에
bool4 conditions;   // 4개의 bool을 하나의 SIMD 레지스터에

// 수학 연산도 SIMD 명령어로 변환
float4 result = math.dot(a, b);
float4 normalized = math.normalize(vector);
```

### 🗂️ 컴파일 모드

```
┌─────────────────────────────────────────────────────────────────┐
│                    Burst 컴파일 모드                             │
├──────────────┬──────────────────────────────────────────────────┤
│              │                                                  │
│     JIT      │  에디터 Play 모드에서 사용                        │
│  (Just-In-   │  • 처음 Job 실행 시 컴파일                       │
│    Time)     │  • 비동기: Mono로 먼저 실행 후 Burst로 전환      │
│              │  • 동기: Burst 컴파일 완료까지 대기               │
│              │                                                  │
├──────────────┼──────────────────────────────────────────────────┤
│              │                                                  │
│     AOT      │  빌드 시 사용                                    │
│  (Ahead-Of-  │  • 빌드 타임에 모든 Job 미리 컴파일              │
│    Time)     │  • 런타임 컴파일 오버헤드 없음                   │
│              │  • 플랫폼별 최적화된 바이너리 생성               │
│              │                                                  │
└──────────────┴──────────────────────────────────────────────────┘
```

---

## ⚠️ 장단점

### ✅ 장점

```
┌─────────────────────────────────────────────────────────────────┐
│  ✅ 장점                                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🚀 극적인 성능 향상                                            │
│     • 일반 C# 대비 10~100배 이상 성능 향상 가능                 │
│     • SIMD 자동 벡터화로 병렬 데이터 처리                       │
│     • 초기화 작업 최대 33배 빠름 (Burst 1.7 기준)               │
│                                                                 │
│  🔧 사용 편의성                                                 │
│     • [BurstCompile] 속성 하나로 적용                           │
│     • 기존 Job System 코드에 쉽게 적용                          │
│     • Unity Editor에 기본 포함 (별도 설치 불필요)               │
│                                                                 │
│  🧵 멀티스레딩 지원                                             │
│     • Burst Job은 워커 스레드에서 병렬 실행 가능                │
│     • 일반 C# Job은 GC 제한으로 메인 스레드만 가능              │
│                                                                 │
│  🔍 디버깅 도구                                                 │
│     • Burst Inspector로 생성된 어셈블리 확인                    │
│     • LLVM IR 최적화 전/후 비교 가능                            │
│     • 벡터화 성공/실패 진단                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ⚠️ 단점 및 제한사항

```
┌─────────────────────────────────────────────────────────────────┐
│  ❌ 단점 및 제한사항                                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📦 지원 타입 제한 (HPC# 제약)                                  │
│     • Managed 타입 사용 불가 (class, string, delegate)          │
│     • 참조 타입 불가 → struct만 사용                            │
│     • 가비지 컬렉션 대상 객체 접근 불가                         │
│                                                                 │
│  🚫 지원하지 않는 기능                                          │
│     • 가상 메서드 (virtual)                                     │
│     • try/catch/finally (제한적 지원)                           │
│     • foreach (일부 컬렉션)                                     │
│     • LINQ                                                      │
│     • Reflection                                                │
│                                                                 │
│  ⚠️ 플랫폼 제한                                                 │
│     • Windows ↔ Mac ↔ Linux 크로스 컴파일 불가                 │
│     • Windows에서 iOS 빌드 시 Burst 미적용                      │
│     • 플랫폼별 빌드 도구 필요 (IL2CPP와 유사)                   │
│                                                                 │
│  🎯 설계 고려사항                                               │
│     • 초기 설계 단계에서 Burst 적용 고려 필요                   │
│     • 기존 코드 마이그레이션 비용 발생                          │
│     • NativeArray 등 특수 컬렉션 사용 필수                      │
│                                                                 │
│  📊 정밀도 손실 가능성                                          │
│     • FloatMode.Fast 사용 시 정밀도 저하                        │
│     • 명령어 재배열로 인한 결과 차이                            │
│     • NaN/Infinity 처리 차이                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ⚖️ 사용 가능/불가능 타입 비교

| 카테고리 | 사용 가능 ✅ | 사용 불가 ❌ |
|:---|:---|:---|
| **기본 타입** | int, float, bool, byte 등 | - |
| **구조체** | struct (unmanaged) | class |
| **문자열** | FixedString | string |
| **컬렉션** | NativeArray, NativeList | List, Array, Dictionary |
| **수학** | Unity.Mathematics | System.Math (일부) |
| **포인터** | unsafe 포인터 | managed 참조 |

---

## 🧪 간단한 예제

### 🧪 예제 1: 기본 IJob

가장 간단한 Burst Job 예제입니다.

```csharp
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;
using UnityEngine;

public class BasicBurstExample : MonoBehaviour
{
    // [BurstCompile] 속성으로 Burst 컴파일 활성화
    [BurstCompile]
    struct AddNumbersJob : IJob
    {
        // 입력 데이터 (읽기 전용)
        [ReadOnly] public NativeArray<float> InputA;
        [ReadOnly] public NativeArray<float> InputB;

        // 출력 데이터 (쓰기)
        [WriteOnly] public NativeArray<float> Output;

        public void Execute()
        {
            // 배열의 모든 요소를 더함
            for (int i = 0; i < InputA.Length; i++)
            {
                Output[i] = InputA[i] + InputB[i];
            }
        }
    }

    void Start()
    {
        int dataSize = 1000;

        // NativeArray 생성 (Allocator.TempJob = 임시 할당)
        var inputA = new NativeArray<float>(dataSize, Allocator.TempJob);
        var inputB = new NativeArray<float>(dataSize, Allocator.TempJob);
        var output = new NativeArray<float>(dataSize, Allocator.TempJob);

        // 데이터 초기화
        for (int i = 0; i < dataSize; i++)
        {
            inputA[i] = i;
            inputB[i] = i * 2;
        }

        // Job 생성 및 스케줄링
        var job = new AddNumbersJob
        {
            InputA = inputA,
            InputB = inputB,
            Output = output
        };

        // Job 스케줄 및 완료 대기
        JobHandle handle = job.Schedule();
        handle.Complete();

        // 결과 확인
        Debug.Log($"Result[0] = {output[0]}, Result[999] = {output[999]}");

        // 메모리 해제 (필수!)
        inputA.Dispose();
        inputB.Dispose();
        output.Dispose();
    }
}
```

### 🧪 예제 2: IJobParallelFor (병렬 처리)

대량의 데이터를 병렬로 처리하는 예제입니다.

```csharp
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;
using UnityEngine;

public class ParallelBurstExample : MonoBehaviour
{
    [BurstCompile]
    struct SquareNumbersJob : IJobParallelFor
    {
        [ReadOnly] public NativeArray<float> Input;
        [WriteOnly] public NativeArray<float> Output;

        // index는 자동으로 분배됨 (0 ~ Length-1)
        public void Execute(int index)
        {
            Output[index] = Input[index] * Input[index];
        }
    }

    void Start()
    {
        int dataSize = 100000;

        var input = new NativeArray<float>(dataSize, Allocator.TempJob);
        var output = new NativeArray<float>(dataSize, Allocator.TempJob);

        // 초기화
        for (int i = 0; i < dataSize; i++)
        {
            input[i] = i;
        }

        var job = new SquareNumbersJob
        {
            Input = input,
            Output = output
        };

        // Schedule(배열길이, 배치크기)
        // 배치크기: 한 스레드가 처리할 항목 수 (64~128 권장)
        JobHandle handle = job.Schedule(dataSize, 64);
        handle.Complete();

        Debug.Log($"100^2 = {output[100]}, 1000^2 = {output[1000]}");

        input.Dispose();
        output.Dispose();
    }
}
```

### 🗂️ 예제 3: Static Method 컴파일

Job 없이 정적 메서드도 Burst 컴파일할 수 있습니다.

```csharp
using Unity.Burst;
using Unity.Mathematics;
using UnityEngine;

// 클래스에도 [BurstCompile] 필요
[BurstCompile]
public static class BurstMathHelper
{
    // 정적 메서드에 [BurstCompile] 적용
    [BurstCompile]
    public static float CalculateDistance(in float3 a, in float3 b)
    {
        float3 diff = a - b;
        return math.sqrt(math.dot(diff, diff));
    }

    [BurstCompile]
    public static void NormalizeArray(ref NativeArray<float3> vectors)
    {
        for (int i = 0; i < vectors.Length; i++)
        {
            vectors[i] = math.normalize(vectors[i]);
        }
    }
}
```

---

## 🧪 실용적인 복잡한 예제

### 🧪 예제 1: Boids 시뮬레이션 (군집 행동)

수천 마리의 새 또는 물고기가 무리 지어 움직이는 시뮬레이션입니다.

```csharp
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;
using Unity.Mathematics;
using UnityEngine;

public class BoidsSimulation : MonoBehaviour
{
    [Header("Simulation Settings")]
    public int boidCount = 1000;
    public float separationWeight = 1.5f;
    public float alignmentWeight = 1.0f;
    public float cohesionWeight = 1.0f;
    public float maxSpeed = 5f;
    public float perceptionRadius = 2.5f;

    private NativeArray<float3> _positions;
    private NativeArray<float3> _velocities;
    private NativeArray<float3> _accelerations;

    // Boid 행동 계산 Job
    [BurstCompile]
    struct CalculateBoidBehaviorJob : IJobParallelFor
    {
        [ReadOnly] public NativeArray<float3> Positions;
        [ReadOnly] public NativeArray<float3> Velocities;
        [WriteOnly] public NativeArray<float3> Accelerations;

        public float SeparationWeight;
        public float AlignmentWeight;
        public float CohesionWeight;
        public float PerceptionRadius;
        public float PerceptionRadiusSq;  // 제곱값 미리 계산

        public void Execute(int index)
        {
            float3 myPos = Positions[index];
            float3 myVel = Velocities[index];

            float3 separation = float3.zero;
            float3 alignment = float3.zero;
            float3 cohesion = float3.zero;
            int neighborCount = 0;

            // 모든 다른 Boid와의 관계 계산
            for (int i = 0; i < Positions.Length; i++)
            {
                // 자기 자신은 건너뜀
                if (i == index)
                {
                    continue;
                }

                float3 otherPos = Positions[i];
                float3 diff = myPos - otherPos;
                float distSq = math.lengthsq(diff);

                // 인식 범위 내의 이웃만 처리
                if (distSq >= PerceptionRadiusSq)
                {
                    continue;
                }

                float dist = math.sqrt(distSq);

                // Separation: 너무 가까운 이웃으로부터 멀어지기
                if (dist > 0.001f)
                {
                    separation += diff / dist;
                }

                // Alignment: 이웃들의 평균 속도 방향으로
                alignment += Velocities[i];

                // Cohesion: 이웃들의 중심으로 이동
                cohesion += otherPos;

                neighborCount++;
            }

            float3 acceleration = float3.zero;

            if (neighborCount > 0)
            {
                // 평균 계산 및 가중치 적용
                separation = separation / neighborCount * SeparationWeight;
                alignment = (alignment / neighborCount - myVel) * AlignmentWeight;
                cohesion = (cohesion / neighborCount - myPos) * CohesionWeight;

                acceleration = separation + alignment + cohesion;
            }

            Accelerations[index] = acceleration;
        }
    }

    // 위치 업데이트 Job
    [BurstCompile]
    struct UpdatePositionsJob : IJobParallelFor
    {
        public NativeArray<float3> Positions;
        public NativeArray<float3> Velocities;
        [ReadOnly] public NativeArray<float3> Accelerations;

        public float DeltaTime;
        public float MaxSpeed;
        public float3 BoundsMin;
        public float3 BoundsMax;

        public void Execute(int index)
        {
            float3 vel = Velocities[index] + Accelerations[index] * DeltaTime;

            // 최대 속도 제한
            float speed = math.length(vel);
            if (speed > MaxSpeed)
            {
                vel = vel / speed * MaxSpeed;
            }

            float3 pos = Positions[index] + vel * DeltaTime;

            // 경계 처리 (wrap around)
            pos = math.fmod(pos - BoundsMin + (BoundsMax - BoundsMin),
                           BoundsMax - BoundsMin) + BoundsMin;

            Positions[index] = pos;
            Velocities[index] = vel;
        }
    }

    void Start()
    {
        // NativeArray 할당 (Persistent = 장기 사용)
        _positions = new NativeArray<float3>(boidCount, Allocator.Persistent);
        _velocities = new NativeArray<float3>(boidCount, Allocator.Persistent);
        _accelerations = new NativeArray<float3>(boidCount, Allocator.Persistent);

        // 초기 위치/속도 랜덤 설정
        var random = new Unity.Mathematics.Random(12345);
        for (int i = 0; i < boidCount; i++)
        {
            _positions[i] = random.NextFloat3(-10f, 10f);
            _velocities[i] = random.NextFloat3Direction() * maxSpeed * 0.5f;
        }
    }

    void Update()
    {
        // Job 1: Boid 행동 계산
        var behaviorJob = new CalculateBoidBehaviorJob
        {
            Positions = _positions,
            Velocities = _velocities,
            Accelerations = _accelerations,
            SeparationWeight = separationWeight,
            AlignmentWeight = alignmentWeight,
            CohesionWeight = cohesionWeight,
            PerceptionRadius = perceptionRadius,
            PerceptionRadiusSq = perceptionRadius * perceptionRadius
        };

        JobHandle behaviorHandle = behaviorJob.Schedule(boidCount, 64);

        // Job 2: 위치 업데이트 (Job 1 완료 후 실행)
        var updateJob = new UpdatePositionsJob
        {
            Positions = _positions,
            Velocities = _velocities,
            Accelerations = _accelerations,
            DeltaTime = Time.deltaTime,
            MaxSpeed = maxSpeed,
            BoundsMin = new float3(-20f, -20f, -20f),
            BoundsMax = new float3(20f, 20f, 20f)
        };

        JobHandle updateHandle = updateJob.Schedule(boidCount, 64, behaviorHandle);
        updateHandle.Complete();

        // 여기서 _positions를 사용하여 렌더링
        // (GPU Instancing 또는 개별 Transform 업데이트)
    }

    void OnDestroy()
    {
        // 메모리 해제 필수!
        if (_positions.IsCreated) _positions.Dispose();
        if (_velocities.IsCreated) _velocities.Dispose();
        if (_accelerations.IsCreated) _accelerations.Dispose();
    }
}
```

### 🧪 예제 2: 메시 변형 (Wave Deformation)

실시간으로 메시 버텍스를 변형하여 파도 효과를 만드는 예제입니다.

```csharp
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;
using Unity.Mathematics;
using UnityEngine;

[RequireComponent(typeof(MeshFilter))]
public class WaveMeshDeformation : MonoBehaviour
{
    [Header("Wave Settings")]
    public float waveHeight = 0.5f;
    public float waveFrequency = 1f;
    public float waveSpeed = 1f;

    private Mesh _mesh;
    private NativeArray<float3> _originalVertices;
    private NativeArray<float3> _modifiedVertices;
    private NativeArray<float3> _normals;

    // 파도 변형 Job
    [BurstCompile]
    struct WaveDeformJob : IJobParallelFor
    {
        [ReadOnly] public NativeArray<float3> OriginalVertices;
        [WriteOnly] public NativeArray<float3> ModifiedVertices;

        public float Time;
        public float WaveHeight;
        public float WaveFrequency;
        public float WaveSpeed;

        public void Execute(int index)
        {
            float3 vertex = OriginalVertices[index];

            // Perlin noise를 사용한 파도 계산
            float xOffset = vertex.x * WaveFrequency + Time * WaveSpeed;
            float zOffset = vertex.z * WaveFrequency + Time * WaveSpeed * 0.7f;

            // noise.cnoise는 -1 ~ 1 범위의 Perlin noise 반환
            float noiseValue = noise.cnoise(new float2(xOffset, zOffset));

            // Y 좌표에 파도 높이 적용
            vertex.y += noiseValue * WaveHeight;

            ModifiedVertices[index] = vertex;
        }
    }

    // 노말 재계산 Job
    [BurstCompile]
    struct RecalculateNormalsJob : IJobParallelFor
    {
        [ReadOnly] public NativeArray<float3> Vertices;
        [WriteOnly] public NativeArray<float3> Normals;

        public float WaveFrequency;
        public float WaveHeight;
        public float Time;
        public float WaveSpeed;

        public void Execute(int index)
        {
            float3 vertex = Vertices[index];

            // 주변 포인트의 기울기로 노말 근사 계산
            float delta = 0.01f;

            float xOffset = vertex.x * WaveFrequency + Time * WaveSpeed;
            float zOffset = vertex.z * WaveFrequency + Time * WaveSpeed * 0.7f;

            // X 방향 기울기
            float heightL = noise.cnoise(new float2(xOffset - delta, zOffset)) * WaveHeight;
            float heightR = noise.cnoise(new float2(xOffset + delta, zOffset)) * WaveHeight;
            float dX = (heightR - heightL) / (2 * delta);

            // Z 방향 기울기
            float heightD = noise.cnoise(new float2(xOffset, zOffset - delta)) * WaveHeight;
            float heightU = noise.cnoise(new float2(xOffset, zOffset + delta)) * WaveHeight;
            float dZ = (heightU - heightD) / (2 * delta);

            // 노말 계산 (cross product의 결과)
            float3 normal = math.normalize(new float3(-dX, 1, -dZ));

            Normals[index] = normal;
        }
    }

    void Start()
    {
        _mesh = GetComponent<MeshFilter>().mesh;
        _mesh.MarkDynamic();  // 자주 변경되는 메시임을 알림

        Vector3[] vertices = _mesh.vertices;
        int vertexCount = vertices.Length;

        // NativeArray 할당
        _originalVertices = new NativeArray<float3>(vertexCount, Allocator.Persistent);
        _modifiedVertices = new NativeArray<float3>(vertexCount, Allocator.Persistent);
        _normals = new NativeArray<float3>(vertexCount, Allocator.Persistent);

        // 원본 버텍스 복사
        for (int i = 0; i < vertexCount; i++)
        {
            _originalVertices[i] = vertices[i];
        }
    }

    void Update()
    {
        // Job 1: 파도 변형
        var deformJob = new WaveDeformJob
        {
            OriginalVertices = _originalVertices,
            ModifiedVertices = _modifiedVertices,
            Time = Time.time,
            WaveHeight = waveHeight,
            WaveFrequency = waveFrequency,
            WaveSpeed = waveSpeed
        };

        JobHandle deformHandle = deformJob.Schedule(_originalVertices.Length, 64);

        // Job 2: 노말 재계산 (변형 후)
        var normalJob = new RecalculateNormalsJob
        {
            Vertices = _modifiedVertices,
            Normals = _normals,
            WaveFrequency = waveFrequency,
            WaveHeight = waveHeight,
            Time = Time.time,
            WaveSpeed = waveSpeed
        };

        JobHandle normalHandle = normalJob.Schedule(_originalVertices.Length, 64, deformHandle);
        normalHandle.Complete();

        // 메시에 결과 적용
        _mesh.SetVertices(_modifiedVertices);
        _mesh.SetNormals(_normals);
    }

    void OnDestroy()
    {
        if (_originalVertices.IsCreated) _originalVertices.Dispose();
        if (_modifiedVertices.IsCreated) _modifiedVertices.Dispose();
        if (_normals.IsCreated) _normals.Dispose();
    }
}
```

### 🧪 예제 3: Function Pointer (동적 함수 호출)

런타임에 다른 Burst 컴파일된 함수를 선택하여 호출하는 고급 패턴입니다.

```csharp
using System;
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;
using UnityEngine;

// Function Pointer 델리게이트 정의
// IL2CPP 호환을 위해 UnmanagedFunctionPointer 필요
[System.Runtime.InteropServices.UnmanagedFunctionPointer(
    System.Runtime.InteropServices.CallingConvention.Cdecl)]
public delegate float MathOperationDelegate(float a, float b);

// Burst 컴파일될 정적 메서드들을 포함하는 클래스
[BurstCompile]
public static class BurstMathOperations
{
    [BurstCompile]
    [AOT.MonoPInvokeCallback(typeof(MathOperationDelegate))]
    public static float Add(float a, float b) => a + b;

    [BurstCompile]
    [AOT.MonoPInvokeCallback(typeof(MathOperationDelegate))]
    public static float Multiply(float a, float b) => a * b;

    [BurstCompile]
    [AOT.MonoPInvokeCallback(typeof(MathOperationDelegate))]
    public static float Power(float a, float b) => Unity.Mathematics.math.pow(a, b);

    [BurstCompile]
    [AOT.MonoPInvokeCallback(typeof(MathOperationDelegate))]
    public static float Max(float a, float b) => Unity.Mathematics.math.max(a, b);
}

public class FunctionPointerExample : MonoBehaviour
{
    public enum Operation { Add, Multiply, Power, Max }
    public Operation selectedOperation = Operation.Add;

    // Burst 컴파일된 Function Pointer 캐시
    private FunctionPointer<MathOperationDelegate> _addPtr;
    private FunctionPointer<MathOperationDelegate> _mulPtr;
    private FunctionPointer<MathOperationDelegate> _powPtr;
    private FunctionPointer<MathOperationDelegate> _maxPtr;

    // Function Pointer를 사용하는 Job
    [BurstCompile]
    struct ProcessArrayJob : IJobParallelFor
    {
        [ReadOnly] public NativeArray<float> InputA;
        [ReadOnly] public NativeArray<float> InputB;
        [WriteOnly] public NativeArray<float> Output;

        // Function Pointer (런타임에 결정)
        public FunctionPointer<MathOperationDelegate> Operation;

        public void Execute(int index)
        {
            // Function Pointer를 통한 Burst 컴파일된 함수 호출
            Output[index] = Operation.Invoke(InputA[index], InputB[index]);
        }
    }

    void Start()
    {
        // Function Pointer 컴파일 (한 번만 수행)
        _addPtr = BurstCompiler.CompileFunctionPointer<MathOperationDelegate>(
            BurstMathOperations.Add);
        _mulPtr = BurstCompiler.CompileFunctionPointer<MathOperationDelegate>(
            BurstMathOperations.Multiply);
        _powPtr = BurstCompiler.CompileFunctionPointer<MathOperationDelegate>(
            BurstMathOperations.Power);
        _maxPtr = BurstCompiler.CompileFunctionPointer<MathOperationDelegate>(
            BurstMathOperations.Max);
    }

    void Update()
    {
        if (!Input.GetKeyDown(KeyCode.Space))
        {
            return;
        }

        int dataSize = 10000;
        var inputA = new NativeArray<float>(dataSize, Allocator.TempJob);
        var inputB = new NativeArray<float>(dataSize, Allocator.TempJob);
        var output = new NativeArray<float>(dataSize, Allocator.TempJob);

        // 데이터 초기화
        for (int i = 0; i < dataSize; i++)
        {
            inputA[i] = i * 0.01f;
            inputB[i] = 2f;
        }

        // 선택된 연산에 따라 Function Pointer 선택
        FunctionPointer<MathOperationDelegate> selectedPtr = selectedOperation switch
        {
            Operation.Add => _addPtr,
            Operation.Multiply => _mulPtr,
            Operation.Power => _powPtr,
            Operation.Max => _maxPtr,
            _ => _addPtr
        };

        var job = new ProcessArrayJob
        {
            InputA = inputA,
            InputB = inputB,
            Output = output,
            Operation = selectedPtr
        };

        job.Schedule(dataSize, 64).Complete();

        Debug.Log($"Operation: {selectedOperation}");
        Debug.Log($"Result[100] = {output[100]}, Result[5000] = {output[5000]}");

        inputA.Dispose();
        inputB.Dispose();
        output.Dispose();
    }
}
```

---

## 🚀 최적화 팁

### 🚀 Burst Inspector 활용

```
Menu: Jobs > Burst > Open Inspector

┌─────────────────────────────────────────────────────────────────┐
│  Burst Inspector 탭                                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  • Assembly           : 최종 네이티브 코드 (x86, ARM 등)        │
│  • LLVM IR (Unopt)    : 최적화 전 LLVM IR                       │
│  • LLVM IR (Opt)      : 최적화 후 LLVM IR                       │
│  • Diagnostics        : 최적화 성공/실패 진단                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ⚡ 성능 최적화 체크리스트

```
┌─────────────────────────────────────────────────────────────────┐
│  ✅ Burst 최적화 체크리스트                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  □ float4, int4 등 SIMD 타입 사용                               │
│  □ [ReadOnly], [WriteOnly] 속성 명시                            │
│  □ Unity.Mathematics 함수 사용 (math.*)                         │
│  □ 분기(if/else) 최소화                                         │
│  □ 메모리 접근 패턴 순차적으로                                  │
│  □ IJobParallelFor 배치 크기 64~128                             │
│  □ NativeArray 사전 할당 (TempJob vs Persistent)                │
│  □ FloatMode.Fast 고려 (정밀도 손실 감안)                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 🚀 BurstCompile 옵션

```csharp
[BurstCompile(
    FloatPrecision.Standard,     // 또는 Low, Medium, High
    FloatMode.Fast,              // 또는 Default, Strict, Deterministic
    CompileSynchronously = false,
    OptimizeFor = OptimizeFor.Performance  // 또는 Size, Balanced
)]
struct MyJob : IJob { ... }
```

---

## 🔗 참고 자료

### 🔹 공식 문서
- [Unity Burst 공식 문서](https://docs.unity3d.com/Packages/com.unity.burst@1.8/manual/index.html)
- [Burst Get Started Guide](https://docs.unity3d.com/Packages/com.unity.burst@1.8/manual/getting-started.html)
- [C#/.NET Type Support](https://docs.unity3d.com/Packages/com.unity.burst@1.8/manual/csharp-type-support.html)
- [Function Pointers](https://docs.unity3d.com/Packages/com.unity.burst@1.8/manual/csharp-function-pointers.html)

### 🔹 튜토리얼
- [Unity Learn - Get started with Unity's Job System](https://learn.unity.com/course/basics-dots-jobs-entities/tutorial/get-started-with-unitys-job-system)
- [Unity Learn - Getting the most out of Burst](https://learn.unity.com/course/dots-best-practices/unit/part-3-implementation-and-optimization/tutorial/part-3-4-getting-the-most-out-of-burst)
- [Code Monkey - DOTS Tutorial](https://unitycodemonkey.com/video.php?v=4ZYn9sR3btg)
- [Kodeco - Unity Job System Tutorial](https://www.kodeco.com/7880445-unity-job-system-and-burst-compiler-getting-started)

### 🔹 블로그 및 커뮤니티
- [Unity Blog - Burst 1.7 소개](https://blog.unity.com/kr/engine-platform/raising-your-game-with-burst-1-7)
- [Hedberg Games - Introduction to Burst](https://www.hedberggames.com/blog/introduction-to-burst)
- [Medium - Getting Started with Unity DOTS Part 3](https://nikolayk.medium.com/getting-started-with-unity-dots-part-3-burst-compiler-7d639274ca1e)
- [Burst에 대해 (한국어)](https://velog.io/@ckleee/Unity3D-Burst%EC%97%90-%EB%8C%80%ED%95%B4)

### 🧪 GitHub 예제
- [unity-jobsystem-boids](https://github.com/komietty/unity-jobsystem-boids) - Boids 예제
- [unity_burst_shape_matching](https://github.com/nialltl/unity_burst_shape_matching) - Shape Matching
- [JobSystemTutorial](https://github.com/smitdylan2001/JobSystemTutorial) - 기본 튜토리얼

---

> 📝 **작성일**: 2026-02-05
> 💡 **출처**: Unity 공식 문서 및 웹 리서치 기반
