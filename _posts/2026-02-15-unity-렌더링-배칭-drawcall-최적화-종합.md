---
layout: post
title: 260215 Unity 렌더링 배칭 DrawCall 최적화 종합
comments: true
tags:
- unity
- urp
- rendering
- performance
- optimization
- gpu
- transform
- api
- ai
- research
---
# 🛠️ Unity 렌더링 성능 최적화 종합 가이드

> 📝 **작성일**: 2026-02-15
> 💡 **대상 Unity 버전**: Unity 6 (6000.x) / URP / HDRP
> 💡 **범위**: GPU Instancing, SRP Batcher, Material/Mesh Batching, Draw Call 감소 기법

---

## 📚 목차

1. [Draw Call 기초](#1-draw-call-기초)
2. [GPU Instancing](#2-gpu-instancing)
3. [SRP Batcher](#3-srp-batcher)
4. [Static Batching](#4-static-batching)
5. [Dynamic Batching](#5-dynamic-batching)
6. [Mesh Batching (Combining)](#6-mesh-batching-combining)
7. [Draw Call 감소 보조 전략](#7-draw-call-감소-보조-전략)
8. [배칭 기법 종합 비교](#8-배칭-기법-종합-비교)
9. [Unity 6 최신 최적화 기법](#9-unity-6-최신-최적화-기법)
10. [프로파일링 및 분석 도구](#10-프로파일링-및-분석-도구)
11. [플랫폼별 Draw Call 예산](#11-플랫폼별-draw-call-예산)
12. [실무 최적화 워크플로우 및 체크리스트](#12-실무-최적화-워크플로우-및-체크리스트)
13. [참고 자료](#13-참고-자료)

---

## 📌 1. Draw Call 기초

### 🔹 1.1 Draw Call이란

Draw Call은 **CPU가 GPU에게 "이 오브젝트를 그려라"라고 명령하는 호출**이다. Graphics API(DirectX, OpenGL, Vulkan, Metal)를 통해 CPU가 GPU에 렌더링 명령을 전달하는 과정이다.

```
[CPU 측]                              [GPU 측]
1. 오브젝트 가시성 판단 (Culling)
2. 렌더 상태 설정 (Render State)
   - Shader 바인딩
   - Texture 바인딩
   - Material Property 설정
   - Blend Mode, Z-Test 등 설정
3. Draw Call 발행 ──────────────────> 4. 버텍스 처리 (Vertex Shader)
                                      5. 래스터라이제이션
                                      6. 프래그먼트 처리 (Fragment Shader)
                                      7. 프레임 버퍼 출력
```

**핵심**: Draw Call 자체의 비용보다 **직전의 렌더 상태(Render State) 변경이 더 비싸다**. CPU가 매번 렌더 상태를 변경하고 명령을 전달하는 과정에서 병목이 발생한다.

### ⚖️ 1.2 SetPass Call vs Draw Call vs Batch

| 용어 | 설명 | 비용 |
|------|------|------|
| **SetPass Call** | Material(Shader + Properties)을 GPU에 설정하는 호출. 렌더 상태 전환 포함 | **가장 높음** |
| **Draw Call** | 특정 메시를 그리라는 GPU 명령 | 중간 |
| **Batch** | Unity가 여러 Draw Call을 하나로 묶은 단위. Stats 패널에 표시 | 최적화 결과 |

```
Material A 설정 (SetPass Call #1)
  +-- Batch #1: Object 1, 2, 3 (동일 Material, Static Batching)
  +-- Batch #2: Object 4 (동일 Material, 별도 메시)

Material B 설정 (SetPass Call #2)
  +-- Batch #3: Object 5, 6 (동일 Material)

Material A 다시 설정 (SetPass Call #3)  <-- 불필요한 상태 전환!
  +-- Batch #4: Object 7
```

**SetPass Call 감소가 Draw Call 감소보다 더 중요하다.** SRP Batcher가 바로 이 점을 최적화한다.

### ⚡ 1.3 Draw Call이 성능에 미치는 영향

Draw Call당 **약 0.05ms ~ 0.1ms** 비용. 1,000개면 5~10ms를 CPU 렌더링에만 소비. 60fps 목표 시 프레임당 16.67ms 예산에서 치명적이다.

| 구분 | CPU Bound | GPU Bound |
|------|-----------|-----------|
| **원인** | Draw Call 과다, 스크립트 로직 과부하 | 셰이더 복잡도, 오버드로우, 해상도 |
| **증상** | GPU 유휴 시간 증가 | 프레임 드랍 |
| **해결** | Draw Call 감소, Batching 최적화 | 셰이더 단순화, 해상도 조정 |

---

## 🎮 2. GPU Instancing

### 🔹 2.1 개념과 원리

GPU Instancing은 **동일한 메시(Mesh)와 동일한 머티리얼(Material)을 사용하는 여러 GameObject를 단일 Draw Call로 렌더링**하는 최적화 기법이다.

```
[CPU]                              [GPU]
  |                                  |
  |-- 메시 데이터 (1회) ------------>|
  |-- Transform 배열 -------------->|  --> 인스턴스 #0 렌더링
  |-- Per-Instance 프로퍼티 -------->|  --> 인스턴스 #1 렌더링
  |-- Draw Call (1회) -------------->|  --> 인스턴스 #2 렌더링
  |                                  |  --> ...
  |                                  |  --> 인스턴스 #N 렌더링
```

1. CPU가 메시 데이터를 **한 번만** GPU에 전송
2. 각 인스턴스의 **변환 행렬(Transform Matrix)**과 **Per-Instance Properties**를 별도 버퍼로 전송
3. GPU가 하나의 Draw Call 내에서 메시를 여러 번 렌더링
4. 셰이더에서 `SV_InstanceID`로 각 인스턴스 식별

### 🔹 2.2 사용 조건과 제약

**인스턴싱 동작 조건:**

| 조건 | 설명 |
|------|------|
| 동일 메시 | 모든 인스턴스가 정확히 같은 Mesh 에셋 사용 |
| 동일 머티리얼 | 모든 인스턴스가 같은 Material 에셋 참조 |
| 셰이더 지원 | `#pragma multi_compile_instancing` 선언 필요 |
| Material 설정 | "Enable GPU Instancing" 체크 활성화 |
| 호환 컴포넌트 | **MeshRenderer** 컴포넌트만 지원 |

**인스턴싱이 깨지는 경우:**

- **SkinnedMeshRenderer** 사용 시
- 서로 다른 메시/머티리얼 인스턴스 혼재
- `MaterialPropertyBlock`에 **인스턴스 프로퍼티로 선언되지 않은** 프로퍼티 설정
- 서로 다른 라이트맵/라이트 프로브 영향
- 멀티 패스 셰이더의 2번째 패스부터
- SRP Batcher가 활성화된 상태에서 SRP 호환 셰이더 사용 시 (SRP Batcher 우선)

### ⚠️ 2.3 Constant Buffer 크기 제한

| 그래픽스 API | Constant Buffer 최대 크기 |
|-------------|--------------------------|
| DirectX 11/12 | 64 KB |
| OpenGL / OpenGL ES | 16 KB |
| Vulkan | 구현체에 따라 다름 (최소 16 KB) |
| Metal | 64 KB |

인스턴스당 `float4` (16 bytes) 하나인 경우 D3D11에서 약 4,096개, OpenGL에서 약 1,024개가 상한.

### 🔹 2.4 자동 인스턴싱과 수동 인스턴싱

**자동 인스턴싱**: 동일 메시+머티리얼 조건만 충족하면 Unity가 자동으로 배치.

**수동 인스턴싱 API:**

| API | 설명 | 최대 인스턴스 수 |
|-----|------|------------------|
| `Graphics.RenderMeshInstanced` | 권장 API | 기본 511개/호출 |
| `Graphics.RenderMeshIndirect` | GPU 버퍼에서 인스턴스 수 읽기 | GPU 메모리 한도 |
| `Graphics.DrawMeshInstanced` | 레거시 (deprecated 예정) | 1,023개/호출 |

```csharp
// RenderMeshInstanced 사용 예시
public class InstancedRenderer : MonoBehaviour
{
    public Mesh mesh;
    public Material material;
    public int instanceCount = 1000;

    private RenderParams _renderParams;
    private Matrix4x4[] _matrices;

    void Start()
    {
        _renderParams = new RenderParams(material);
        _matrices = new Matrix4x4[instanceCount];
        for (int i = 0; i < instanceCount; i++)
        {
            Vector3 position = Random.insideUnitSphere * 50f;
            _matrices[i] = Matrix4x4.TRS(position, Quaternion.identity, Vector3.one);
        }
    }

    void Update()
    {
        Graphics.RenderMeshInstanced(_renderParams, mesh, 0, _matrices);
    }
}
```

### 🔹 2.5 커스텀 셰이더에서 인스턴싱 지원

```hlsl
Shader "Custom/InstancedShader"
{
    Properties
    {
        _Color ("Color", Color) = (1, 1, 1, 1)
        _MainTex ("Texture", 2D) = "white" {}
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_instancing  // [1] 인스턴싱 배리언트 생성

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID  // [2] 인스턴스 ID 입력
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID  // [3] Fragment에서 사용 시 필요
            };

            sampler2D _MainTex;

            // [4] 인스턴스별 프로퍼티 버퍼
            UNITY_INSTANCING_BUFFER_START(Props)
                UNITY_DEFINE_INSTANCED_PROP(float4, _Color)
            UNITY_INSTANCING_BUFFER_END(Props)

            v2f vert(appdata v)
            {
                v2f o;
                UNITY_SETUP_INSTANCE_ID(v);         // [5] 필수: vert 최상단
                UNITY_TRANSFER_INSTANCE_ID(v, o);    // [6] frag로 ID 전달
                o.pos = UnityObjectToClipPos(v.vertex); // [7] 올바른 변환 함수
                o.uv = v.uv;
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                UNITY_SETUP_INSTANCE_ID(i);          // [8] frag에서 ID 설정
                fixed4 tex = tex2D(_MainTex, i.uv);
                fixed4 col = UNITY_ACCESS_INSTANCED_PROP(Props, _Color); // [9] 프로퍼티 접근
                return tex * col;
            }
            ENDCG
        }
    }
}
```

**핵심 매크로 정리:**

| 매크로 | 역할 |
|--------|------|
| `#pragma multi_compile_instancing` | 인스턴싱 셰이더 배리언트 생성 |
| `UNITY_VERTEX_INPUT_INSTANCE_ID` | `SV_InstanceID` 선언 |
| `UNITY_INSTANCING_BUFFER_START/END` | 인스턴스별 프로퍼티 버퍼 |
| `UNITY_DEFINE_INSTANCED_PROP` | 인스턴스별 프로퍼티 정의 |
| `UNITY_SETUP_INSTANCE_ID` | 현재 인스턴스 ID 설정 |
| `UNITY_TRANSFER_INSTANCE_ID` | vert→frag ID 전달 |
| `UNITY_ACCESS_INSTANCED_PROP` | 인스턴스별 값 읽기 |

### 🎮 2.6 URP/HDRP 지원 현황

| 파이프라인 | GPU Instancing | SRP Batcher 관계 |
|-----------|:-:|---|
| Built-in RP | 완전 지원 | SRP Batcher 없으므로 주요 배칭 기법 |
| URP | 지원 (SRP Batcher 비활성화 필요) | SRP Batcher가 우선 |
| HDRP | 지원 (SRP Batcher 비활성화 필요) | SRP Batcher가 우선 |

### ⚡ 2.7 성능 벤치마크

| 시나리오 | Draw Call (미사용) | Draw Call (GPU Instancing) | 개선율 |
|---------|-------------------|-----------------------------|--------|
| 동일 나무 1,000그루 | ~1,000 | ~1-2 | 99%+ |
| 동일 벽돌 10,000개 | ~10,000 | ~10-20 | 99%+ |
| 서로 다른 메시 100종 | ~100 | ~100 (효과 없음) | 0% |

---

## 📌 3. SRP Batcher

### 🔹 3.1 개념과 원리

SRP Batcher는 **Scriptable Render Pipeline(SRP) 전용 드로우 콜 최적화 시스템**이다. 기존 배칭이 "드로우 콜 수 감소"에 집중했다면, SRP Batcher는 **"드로우 콜의 비용 감소"**에 집중한다.

- Material 데이터를 **GPU 메모리에 영속적(Persistent)으로 유지**
- Material 내용이 변경되지 않으면 **렌더 상태 변경을 생략**
- 동일한 Shader Variant를 사용하는 Draw Call을 **SRP Batch로 묶어** 연속 실행

### 🔹 3.2 CBUFFER 기반 동작 방식

| CBUFFER | 이름 | 내용 | 갱신 빈도 |
|---------|------|------|-----------|
| CBuffer0 | 전역 데이터 | 시간, 조명 등 | 프레임당 1회 |
| CBuffer1 | UnityPerDraw | 오브젝트별 변환 행렬 | 오브젝트 이동 시 |
| CBuffer2 | UnityPerMaterial | Material 프로퍼티 | Material 변경 시 |

```
[전통적 방식]
Material A 설정 (Full State Change) -> Draw
Material B 설정 (Full State Change) -> Draw
Material A 설정 (Full State Change) -> Draw  <-- 불필요한 전체 상태 전환

[SRP Batcher]
Shader Variant 바인딩 (1회) -> Draw (Material A props)
                            -> Draw (Material B props)  <-- CBUFFER만 교체
                            -> Draw (Material A props)  <-- CBUFFER만 교체
```

Material 데이터는 한 번 GPU에 올라가면 변경되지 않는 한 재사용. CBUFFER 교체는 SetShaderPass 대비 훨씬 가벼운 연산.

### 🔹 3.3 SRP Batcher 호환 Shader 조건

**조건 1**: 빌트인 엔진 프로퍼티를 `UnityPerDraw` CBUFFER에 선언

```hlsl
CBUFFER_START(UnityPerDraw)
    float4x4 unity_ObjectToWorld;
    float4x4 unity_WorldToObject;
    float4 unity_LODFade;
    real4 unity_WorldTransformParams;
    // Lighting (SH coefficients 등)
CBUFFER_END
```

**조건 2**: 모든 Material 프로퍼티를 `UnityPerMaterial` CBUFFER에 선언

```hlsl
CBUFFER_START(UnityPerMaterial)
    float4 _BaseColor;
    float4 _BaseMap_ST;
    float _Metallic;
    float _Smoothness;
CBUFFER_END
```

**주의사항:**
- Texture 오브젝트(`TEXTURE2D`, `SAMPLER`)는 CBUFFER **바깥**에 선언
- Properties 블록의 모든 프로퍼티가 `UnityPerMaterial`에 빠짐없이 포함되어야 함
- `Core.hlsl` include 시 `UnityPerDraw`는 자동 포함
- Shader Inspector에서 "SRP Batcher: Compatible" 여부 즉시 확인 가능

**호환 Shader 예시 (URP Custom Unlit):**

```hlsl
Shader "Custom/SRPBatcherCompatible"
{
    Properties
    {
        _BaseColor ("Base Color", Color) = (1, 1, 1, 1)
        _BaseMap ("Base Map", 2D) = "white" {}
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" "RenderPipeline"="UniversalPipeline" }
        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            CBUFFER_START(UnityPerMaterial)
                float4 _BaseColor;
                float4 _BaseMap_ST;
            CBUFFER_END

            TEXTURE2D(_BaseMap);
            SAMPLER(sampler_BaseMap);

            struct Attributes { float4 positionOS : POSITION; float2 uv : TEXCOORD0; };
            struct Varyings { float4 positionHCS : SV_POSITION; float2 uv : TEXCOORD0; };

            Varyings vert(Attributes IN)
            {
                Varyings OUT;
                OUT.positionHCS = TransformObjectToHClip(IN.positionOS.xyz);
                OUT.uv = TRANSFORM_TEX(IN.uv, _BaseMap);
                return OUT;
            }

            half4 frag(Varyings IN) : SV_Target
            {
                half4 color = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv);
                return color * _BaseColor;
            }
            ENDHLSL
        }
    }
}
```

### 🔹 3.4 SRP Batcher가 동작하지 않는 경우

| 원인 | 설명 |
|------|------|
| **MaterialPropertyBlock 사용** | `Renderer.SetPropertyBlock()` 호출 시 즉시 제외 |
| **비호환 Shader** | CBUFFER 구조 미충족 |
| **Particle System** | 파티클 렌더러 비호환 |
| **Skinned Mesh Renderer** | 스킨드 메시 비호환 |
| **Built-in RP** | SRP 전용 기능 |
| **Shader Variant 불일치** | 같은 Shader라도 키워드 조합이 다르면 Batch 끊김 |
| **렌더 상태 변경** | Blend Mode, Stencil, Depth Write 등이 다르면 Batch 끊김 |

### 🎮 3.5 GPU Instancing과의 관계

**SRP Batcher가 GPU Instancing보다 우선순위가 높다.**

```
[배칭 우선순위 (높음 -> 낮음)]
1. Static Batching
2. SRP Batcher
3. GPU Instancing
4. Dynamic Batching
```

| 상황 | 권장 기법 |
|------|-----------|
| 다양한 메시 + 다양한 Material (같은 Shader) | **SRP Batcher** |
| 동일 메시 + 동일 Material 수천~수만 개 | **GPU Instancing** |
| Per-Instance 데이터 필수 | **GPU Instancing** |
| 일반적인 건축/기계 모델 뷰어 | **SRP Batcher** |

GPU Instancing을 사용하려면 SRP Batcher를 비활성화해야 한다:

```csharp
// 전역 비활성화 (다른 오브젝트 성능 저하 주의)
GraphicsSettings.useScriptableRenderPipelineBatching = false;
```

> 💡 **권장**: 전역 비활성화보다 **특정 셰이더만 SRP Batcher 비호환으로 만드는 것**이 더 나은 전략.

### 🛠️ 3.6 설정 방법

| 파이프라인 | 설정 | 기본값 |
|-----------|------|--------|
| URP | URP Asset > Inspector > Advanced > SRP Batcher | 활성화 |
| HDRP | HDRP Asset > Debug 모드 > Enable SRP Batcher | 활성화 (비활성화 비권장) |
| 코드 | `GraphicsSettings.useScriptableRenderPipelineBatching = true;` | - |

### ✅ 3.7 성능 이점

Unity 공식 벤치마크: **CPU 렌더링 시간 1.2x ~ 4x 향상**. 실제 테스트에서 17.9ms → 3.6ms (약 80% 감소) 사례 보고. 단, **GPU 성능이 아닌 CPU 렌더링 성능**을 최적화하므로 GPU-bound 프로젝트에서는 체감이 적을 수 있다.

### 🔹 3.8 실무 팁

- **Shader Variant 최소화**: Variant가 적을수록 하나의 SRP Batch에 더 많은 Draw Call 포함
- **MaterialPropertyBlock 사용 금지**: SRP Batcher 환경에서 가장 피해야 할 패턴
- **동일 Shader + 다수 Material 전략**: 같은 Shader를 쓰면 Material이 달라도 배칭됨
- **렌더 큐 안정화**: Custom Render Queue 값을 무분별하게 사용하면 배칭 끊어짐

---

## 📌 4. Static Batching

### 🔹 4.1 동작 원리

**움직이지 않는 정적 오브젝트들의 메시 데이터를 월드 좌표로 변환하여 하나의 통합 버텍스/인덱스 버퍼에 결합**하는 기법이다.

1. Static으로 표시된 오브젝트 수집
2. 각 메시의 버텍스를 월드 좌표계로 변환
3. 같은 Material 메시들을 하나의 큰 버퍼로 합침
4. 렌더링 시 통합 버퍼에서 필요한 부분만 인덱싱
5. 개별 오브젝트가 씬에 남아 **개별 컬링 가능**

### 🛠️ 4.2 설정

```
방법 1: Edit > Project Settings > Player > Other Settings > Static Batching 체크
방법 2: Inspector > Static 드롭다운 > Batching Static 체크
방법 3: 런타임 - StaticBatchingUtility.Combine(gameObject);
```

### 🔹 4.3 트레이드오프

| 항목 | 상세 |
|------|------|
| 버텍스 한도 | 배치당 최대 **64,000 버텍스**. 초과 시 자동 분할 |
| 메모리 증가 | **메시 데이터 복제**. 동일 메시 100개 → 100배 메모리 |
| CPU 성능 | 런타임 버텍스 변환 불필요 |
| 제약 | 결합된 오브젝트의 Transform 변경 시 배칭 해제 |

> ⚠️ **주의**: 동일 메시를 많이 사용하는 경우 GPU Instancing이 메모리 효율적. Static Batching은 **서로 다른 메시 + 같은 Material**인 정적 오브젝트에 유리.

---

## 📌 5. Dynamic Batching

### 🔹 5.1 동작 원리

**런타임에 동적으로** 작은 메시들의 버텍스를 매 프레임 CPU에서 월드 좌표로 변환하여 합침.

### 🔹 5.2 제약 조건

| 조건 | 제한 |
|------|------|
| 버텍스 수 | 최대 **300개 이하** |
| 버텍스 속성 | 최대 **900개의 속성** |
| Material | 동일 Material 인스턴스 필수 |
| 스케일 | 홀수축 네거티브 스케일 불가 |
| 멀티패스 | 첫 번째 패스만 가능 |

### 🔹 5.3 현대 환경에서의 위상

> 💡 **Unity 공식**: "대부분의 경우 Dynamic Batching은 더 이상 권장되지 않는다. CPU 오버헤드가 Draw Call 절약 효과보다 클 수 있기 때문이다."

- HDRP에서는 **미지원**
- SRP Batcher 활성화 시 자동 비활성화
- 구형 로우엔드 디바이스 타겟 시에만 고려

---

## 🎮 6. Mesh Batching (Combining)

### 🔹 6.1 개념

여러 개별 메시를 **하나의 통합 메시로 물리적으로 결합**하여 Draw Call을 줄이는 기법.

| 구분 | Material Batching | Mesh Combining |
|------|-------------------|----------------|
| 결합 대상 | 렌더링 명령 (Draw Call) | 메시 지오메트리 데이터 |
| 원본 유지 | 개별 메시 유지 | 새 통합 메시 생성 |
| 개별 컬링 | 가능 (Static Batching) | 불가 (하나의 바운딩 박스) |

### 🔹 6.2 StaticBatchingUtility.Combine

```csharp
// 루트 하위 모든 자식 결합
StaticBatchingUtility.Combine(rootGameObject);

// 특정 오브젝트 배열 결합
GameObject[] objects = new GameObject[] { obj1, obj2, obj3 };
StaticBatchingUtility.Combine(objects, rootGameObject);
```

- 내부적으로 통합 메시 생성하되 **개별 컬링 가능**
- 결합 후 자식 Transform 변경 불가
- 런타임 사용 시 메시에 **Read/Write Enabled** 필수

### 🎮 6.3 Mesh.CombineMeshes

```csharp
public void CombineChildMeshes()
{
    MeshFilter[] meshFilters = GetComponentsInChildren<MeshFilter>();
    CombineInstance[] combine = new CombineInstance[meshFilters.Length];

    for (int i = 0; i < meshFilters.Length; i++)
    {
        combine[i].mesh = meshFilters[i].sharedMesh;
        combine[i].transform = meshFilters[i].transform.localToWorldMatrix;
    }

    Mesh combinedMesh = new Mesh();
    combinedMesh.indexFormat = UnityEngine.Rendering.IndexFormat.UInt32; // 65535+ 버텍스
    combinedMesh.CombineMeshes(combine, true, true);
    GetComponent<MeshFilter>().sharedMesh = combinedMesh;
}
```

### ⚖️ 6.4 StaticBatchingUtility vs Mesh.CombineMeshes

| 비교 항목 | StaticBatchingUtility | Mesh.CombineMeshes |
|-----------|:---------------------:|:-------------------:|
| 개별 컬링 | 가능 | 불가 |
| 개별 활성화/비활성화 | 가능 | 불가 |
| Material 분리 | 자동 그룹 | 수동 관리 |
| 유연성 | 낮음 | 높음 |

### ⚠️ 6.5 장단점

| 장점 | 단점 |
|------|------|
| Draw Call 대폭 감소 | 개별 오브젝트 조작 불가 |
| SetPass Call 감소 | 바운딩 박스 비대화 → 컬링 효율 저하 |
| GPU가 큰 메시 하나를 그리는 것이 효율적 | 메모리 사용량 증가 |

**실무 주의사항:**

```
문제 1: 결합 후 오히려 FPS 하락
  → 바운딩 박스가 커져 컬링 안 됨. 공간 인접 오브젝트끼리만 결합.

문제 2: 65535 버텍스 초과 에러
  → mesh.indexFormat = IndexFormat.UInt32 설정

문제 3: Material 다른 오브젝트 결합 시 이점 미미
  → 서브메시별 별도 Draw Call 발생. 텍스처 아틀라스로 Material 통합.
```

### 🔹 6.6 LOD와의 관계

LOD와 Mesh Combining은 상충. 결합된 메시는 개별 LOD 전환이 불가.

```
[권장 전략]
LOD0 (근거리): 개별 오브젝트 → SRP Batcher / GPU Instancing
LOD1 (중거리): 선택적 결합 → StaticBatchingUtility
LOD2 (원거리): 적극적 결합 → Mesh.CombineMeshes
LOD3 (최원거리): Impostor / Billboard
```

---

## 📌 7. Draw Call 감소 보조 전략

### 🔹 7.1 Texture Atlas

여러 텍스처를 하나의 큰 텍스처로 합쳐 Material을 공유 가능하게 한다.

```
[Before]                          [After]
Texture A -> Material A           Combined Atlas -> Shared Material
Texture B -> Material B           (UV 좌표 재매핑)
Texture C -> Material C
= 3 Draw Calls                   = 1 Draw Call (Batching 가능)
```

- 아틀라스 크기가 너무 크면 메모리 낭비
- 텍스처 블리딩 방지를 위해 패딩 추가 필요
- 모바일 권장 최대: 2048x2048 ~ 4096x4096

### 🔹 7.2 Material 공유/통합

```csharp
// [나쁜 예] Material 복제 발생
renderer.material.color = Color.red;  // .material 접근 시 자동 복제!

// [좋은 예] SharedMaterial 사용
renderer.sharedMaterial = commonMaterial;

// [좋은 예] MaterialPropertyBlock (Built-in RP)
MaterialPropertyBlock props = new MaterialPropertyBlock();
props.SetColor("_Color", Color.red);
renderer.SetPropertyBlock(props);
```

| 접근 방식 | Material 복제 | 배칭 영향 | 메모리 |
|-----------|:-:|:-:|:-:|
| `renderer.material` | O (자동) | 배칭 파괴 | 증가 |
| `renderer.sharedMaterial` | X | 배칭 유지 | 변화 없음 |
| `renderer.materials[]` | O (배열 전체) | 배칭 파괴 | 증가 |
| `renderer.sharedMaterials[]` | X | 배칭 유지 | 변화 없음 |

> 💡 `.material` 접근으로 생성된 인스턴스는 명시적 `Destroy()` 없으면 메모리 누수 발생.

### 🔹 7.3 Occlusion Culling

카메라에 보이지 않는 오브젝트(다른 오브젝트에 가려진)를 렌더링에서 제외.

```
설정: Window > Rendering > Occlusion Culling > Bake
오브젝트: Inspector > Static > Occluder Static / Occludee Static
```

| 파라미터 | 권장값 |
|----------|--------|
| Smallest Occluder | 0.2 ~ 0.5m |
| Smallest Hole | 0.1 ~ 0.25m |

적합: 실내/도시 환경. 부적합: 오픈 월드, 투명 오브젝트 다수.

### 🔹 7.4 LOD (Level of Detail)

카메라 거리에 따라 메시 디테일을 자동 전환.

| LOD 레벨 | 폴리곤 비율 | 용도 |
|----------|:-:|---|
| LOD 0 | 100% | 가까운 거리 |
| LOD 1 | 40~60% | 중간 거리 |
| LOD 2 | 15~25% | 먼 거리 |
| Culled | 0% | 화면에서 제거 |

### 🔹 7.5 Camera Culling Layer

```csharp
// 레이어별 다른 컬링 거리
float[] distances = new float[32];
distances[LayerMask.NameToLayer("SmallProps")] = 50f;
distances[LayerMask.NameToLayer("Environment")] = 200f;
camera.layerCullDistances = distances;
```

---

## 🚀 8. 배칭 기법 종합 비교

### ⚖️ 8.1 기법별 비교표

| 기법 | Draw Call 감소 | CPU 비용 | 메모리 | 메시 제약 | Material 제약 | 런타임 동적 | 현대 권장도 |
|------|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Static Batching | 높음 | 낮음 | **높음** | 없음 (Static만) | 동일 Material | 불가 | 중 |
| Dynamic Batching | 중간 | **높음** | 낮음 | 300 버텍스 이하 | 동일 Material | 가능 | **낮음** |
| SRP Batcher | - (비용 감소) | 매우 낮음 | 낮음 | 없음 | 동일 Shader Variant | 가능 | **높음** |
| GPU Instancing | 매우 높음 | 낮음 | 낮음 | **동일 메시** | **동일 Material** | 가능 | 높음 |
| Mesh Combining | 매우 높음 | 일회성 | **높음** | 없음 | 수동 관리 | 부분 가능 | 중 |
| GPU Resident Drawer | 극히 높음 | 매우 낮음 | 중간 | 없음 | 자동 | 가능 | **최고** (Unity 6) |

### 🛠️ 8.2 우선순위 가이드 (URP/HDRP)

```
1순위: SRP Batcher (기본 활성화, 셰이더 호환만 확인)
2순위: GPU Resident Drawer (Unity 6, 대규모 씬)
3순위: GPU Instancing (동일 메시 대량 배치)
4순위: Static Batching (정적 환경, 메모리 여유 시)
5순위: Mesh Combining (특수 상황, 수동 제어 필요)
6순위: Dynamic Batching (구형 디바이스에서만)
```

### 🔹 8.3 의사결정 플로우

```
오브젝트가 움직이는가?
  +-- YES --> 동일 메시/Material이 대량인가?
  |             +-- YES --> GPU Instancing
  |             +-- NO  --> SRP Batcher (기본)
  +-- NO  --> Unity 6 사용 가능한가?
              +-- YES --> GPU Resident Drawer
              +-- NO  --> 동일 메시 반복?
                           +-- YES --> GPU Instancing
                           +-- NO  --> 메모리 여유?
                                       +-- YES --> Static Batching
                                       +-- NO  --> SRP Batcher (기본)
```

### 🏗️ 8.4 렌더 파이프라인별 지원 매트릭스

| 배칭 기법 | Built-in RP | URP | HDRP |
|-----------|:-:|:-:|:-:|
| Static Batching | O | O | O |
| Dynamic Batching | O | O | **X** |
| SRP Batcher | **X** | O (기본) | O (기본) |
| GPU Instancing | O | O | O |
| MaterialPropertyBlock | O (배칭 유지) | O (**SRP Batcher 깨짐**) | O (**SRP Batcher 깨짐**) |
| GPU Resident Drawer | **X** | O (Unity 6+) | O (Unity 6+) |

---

## 🚀 9. Unity 6 최신 최적화 기법

### 🎮 9.1 GPU Resident Drawer

Unity 6에서 도입된 가장 중요한 렌더링 최적화 기능. 내부적으로 **BatchRendererGroup API**를 사용하여 GPU에서 직접 인스턴싱 Draw Call 수행.

**성능 효과 (35,000+ 오브젝트 테스트):**

| 항목 | OFF | ON |
|------|:---:|:--:|
| Draw Calls | 43,500 | **128** |
| CPU 렌더링 시간 | 높음 | **대폭 감소** |

**설정:**

```
1. Project Settings > Graphics > Shader Stripping
   -> BatchRendererGroup Variants: "Keep All"
2. URP Asset -> SRP Batcher: 활성화
3. URP Asset -> GPU Resident Drawer: "Instanced Drawing"
4. Player Settings -> Static Batching: 비활성화 (충돌 방지)
5. Universal Renderer -> Rendering Path: "Forward+"
```

**제약**: Forward+ 필수, 스킨드 메시 미지원, Static Batching 비활성화 필요.

### 🎮 9.2 GPU Occlusion Culling

런타임에 동적으로 가려진 오브젝트를 GPU에서 판별. 기존 CPU 기반 Bake 방식 대비 더 정확하고 동적. GPU Resident Drawer와 결합 시 더 높은 효과.

### 🔹 9.3 기타 Unity 6 기능

| 기능 | 설명 |
|------|------|
| **Deferred+** (6.1) | 클러스터 기반 라이트 컬링, GPU Resident Drawer 완전 호환 |
| **Variable Rate Shading** (6.1) | 화면 영역별 셰이딩 해상도 조절, GPU 부하 10~30% 절감 |
| **Render Graph** | 자동 리소스 관리, 불필요한 렌더 패스 자동 제거 |
| **Shader Variant 최적화** | Fog/LOD Crossfade 관련 Variant 수 감소 옵션 |

---

## 🗂️ 10. 프로파일링 및 분석 도구

### 💻 10.1 Frame Debugger

```
Window > Analysis > Frame Debugger
```

1. Enable 클릭 → 프레임 캡처
2. 왼쪽: Draw Call 목록 (계층 구조)
3. 우측: Shader/Material/렌더 타겟/Batching 상태
4. SRP Batch 확인: `Render Camera > Render Opaques > RenderLoopNewBatcher.Draw`

**Batching 실패 사유 (Break Reason):**

| 사유 | 해결 |
|------|------|
| Different Materials | Material 통합/Texture Atlas |
| Non-instanceable property set | MPB 대신 Material Variant |
| Objects have different scale | Uniform scale 사용 |
| Shader disables batching | Shader 수정 |
| Shader is different | Shader 통합 |
| Shader keywords are different | Variant 최소화 |

### 🔹 10.2 Profiler

```
Window > Analysis > Profiler → Rendering 모듈
```

| 메트릭 | 모바일 주의 | PC 주의 |
|--------|:---:|:---:|
| SetPass Calls | < 50 | < 200 |
| Draw Calls | < 200 | < 2,500 |
| Triangles | < 100K | < 1M |

**CPU vs GPU Bound 판별:**
- CPU 시간 > GPU 시간 → CPU Bound → Draw Call 감소 필요
- GPU 시간 > CPU 시간 → GPU Bound → 셰이더/해상도 최적화

### 🔹 10.3 RenderDoc

GPU 타이밍 측정, 오버드로우 시각화, Shader 복잡도 분석에 사용. Frame Debugger로 볼 수 없는 GPU 측 병목 분석.

### 🔹 10.4 Profile Analyzer

```
Window > Analysis > Profile Analyzer
```

최적화 전후 데이터 비교 → 정량적 성능 개선 효과 측정.

---

## 📌 11. 플랫폼별 Draw Call 예산

| 항목 | 모바일 (저사양) | 모바일 (고사양) | PC (미드) | PC (하이엔드) |
|------|:-:|:-:|:-:|:-:|
| Draw Calls | < 100 | < 200 | < 1,500 | < 2,500+ |
| SetPass Calls | < 30 | < 50 | < 150 | < 300 |
| Triangles | < 50K | < 200K | < 500K | < 2M+ |
| Target FPS | 30fps | 60fps | 60fps | 60~144fps |
| 프레임 예산 | 33.3ms | 16.67ms | 16.67ms | 6.9~16.67ms |

**프레임 예산 배분 (16.67ms @ 60fps):**

```
렌더링 (CPU): 5~8ms
  컬링: 0.5~1ms / Draw Call 발행: 2~4ms / 기타: 1~2ms
스크립트 로직: 3~5ms
물리 연산: 1~2ms
애니메이션: 1~2ms
여유 버퍼: 2~3ms (스파이크 대비)
```

**모바일 특수 고려사항:**
- 드라이버 오버헤드: PC 대비 Draw Call당 CPU 비용 2~5배
- Vulkan/Metal 우선 사용 (OpenGL ES 대비 오버헤드 현저히 낮음)
- 발열: 평균 부하를 예산의 70% 수준으로 유지

---

## 🚀 12. 실무 최적화 워크플로우 및 체크리스트

### 🔹 12.1 4단계 워크플로우

```
Phase 1: 측정 (Measure)
  Profiler로 성능 기준선 / Frame Debugger로 Draw Call 분포
  CPU Bound vs GPU Bound 판별 / 병목 지점 식별

Phase 2: 분석 (Analyze)
  가장 큰 성능 영향 항목 식별 / Material/Shader 패턴 분석
  Batching 실패 원인 분석 / 오버드로우 분석

Phase 3: 최적화 (Optimize)
  우선순위에 따라 기법 적용 / 각 단계마다 성능 재측정
  시각적 품질과 성능 균형 확인

Phase 4: 검증 (Validate)
  타겟 디바이스 실측 / 다양한 씬/상황 검증 / 회귀 테스트
```

### 🚀 12.2 최적화 우선순위 (비용 대비 효과)

| 순위 | 기법 | 난이도 | 효과 |
|:---:|------|:---:|:---:|
| 1 | SRP Batcher 활성화 | 매우 쉬움 | 높음 |
| 2 | GPU Resident Drawer (Unity 6) | 쉬움 | 매우 높음 |
| 3 | Static Batching | 쉬움 | 중~높음 |
| 4 | Material 통합 (Texture Atlas) | 중간 | 높음 |
| 5 | Occlusion Culling | 중간 | 중~높음 |
| 6 | LOD 설정 | 중간 | 중간 |
| 7 | GPU Instancing | 중간 | 높음 |
| 8 | Mesh Combining | 높음 | 중~높음 |
| 9 | Camera Culling Layer | 쉬움 | 낮~중간 |
| 10 | Dynamic Batching | 쉬움 | 낮음 |

### 🔹 12.3 체크리스트

**Material / Texture:**
- [ ] 동일 Shader Material 수 최소화
- [ ] `.material` 대신 `.sharedMaterial` 사용
- [ ] Texture Atlas 활용으로 Material 통합
- [ ] MaterialPropertyBlock 사용 시 SRP Batcher 호환성 확인
- [ ] Shader Inspector에서 SRP Batcher 호환 확인

**Batching:**
- [ ] 정적 오브젝트에 Static 플래그 설정
- [ ] SRP Batcher 활성화 확인 (URP Asset)
- [ ] 동일 메시 대량 배치 시 GPU Instancing 고려
- [ ] Unity 6에서 GPU Resident Drawer 활성화 검토

**Culling:**
- [ ] Occlusion Culling Bake (실내/도시)
- [ ] Camera Culling Mask 설정
- [ ] Per-Layer Cull Distance 설정
- [ ] LOD Group 설정

**Mesh:**
- [ ] 불필요하게 높은 폴리곤 메시 최적화
- [ ] Mesh Combining 시 공간 분할로 컬링 효율 유지

**렌더링:**
- [ ] 불필요한 그림자 비활성화
- [ ] 반투명 오브젝트 수 최소화
- [ ] Realtime Light 수 최소화 (Baked Light 활용)
- [ ] Post-Processing 패스 최적화

### 🔹 12.4 흔한 실수

| 실수 | 영향 | 해결 |
|------|------|------|
| `renderer.material` 남용 | Material 인스턴스 복제 → 배칭 파괴 | `.sharedMaterial` 또는 MPB |
| 모든 오브젝트 Static 설정 | 메모리 과다 | 실제 정적 오브젝트만 |
| Texture Atlas 없이 Material 다량 | SetPass Call 폭증 | Atlas로 통합 |
| Shadow 미최적화 | Shadow Pass Draw Call 2배 | 필요 없는 오브젝트 Shadow 끄기 |
| Realtime Light 과다 | 라이트별 추가 Draw Call | Baked Lighting 활용 |
| 투명 오브젝트 과다 | 배칭 불가 + 정렬 비용 | Cutout 대체, 수 최소화 |

---

## 🔗 13. 참고 자료

### 🔹 Unity 공식 문서

- [Draw Call 최적화 소개 (Unity 6)](https://docs.unity3d.com/6000.2/Documentation/Manual/optimizing-draw-calls.html)
- [GPU Instancing (Unity 6000.3)](https://docs.unity3d.com/6000.3/Documentation/Manual/GPUInstancing.html)
- [GPU Instancing Shader 작성](https://docs.unity3d.com/Manual/gpu-instancing-shader.html)
- [SRP Batcher](https://docs.unity3d.com/Manual/SRPBatcher.html)
- [SRP Batcher (2023.2)](https://docs.unity3d.com/2023.2/Documentation/Manual/SRPBatcher.html)
- [URP SRP Batcher 호환 Shader](https://docs.unity3d.com/6000.3/Documentation/Manual/urp/shaders-in-universalrp-srp-batcher.html)
- [Static Batching](https://docs.unity3d.com/2023.1/Documentation/Manual/static-batching.html)
- [Dynamic Batching](https://docs.unity3d.com/6000.1/Documentation/Manual/dynamic-batching.html)
- [GPU Resident Drawer (URP)](https://docs.unity3d.com/6000.0/Documentation/Manual/urp/gpu-resident-drawer.html)
- [Occlusion Culling](https://docs.unity3d.com/6000.3/Documentation/Manual/OcclusionCulling.html)
- [StaticBatchingUtility.Combine API](https://docs.unity3d.com/ScriptReference/StaticBatchingUtility.Combine.html)
- [Mesh.CombineMeshes API](https://docs.unity3d.com/ScriptReference/Mesh.CombineMeshes.html)
- [Graphics.RenderMeshInstanced API](https://docs.unity3d.com/ScriptReference/Graphics.RenderMeshInstanced.html)
- [URP 성능 최적화](https://docs.unity3d.com/6000.3/Documentation/Manual/urp/configure-for-better-performance.html)

### 🔹 블로그 및 커뮤니티

- [SRP Batcher: Speed up your rendering - Unity Blog](https://unity.com/blog/engine-platform/srp-batcher-speed-up-your-rendering)
- [GPU Resident Drawer 성능 향상 - The Knights of U](https://theknightsofu.com/boost-performance-of-your-game-in-unity-6-with-gpu-resident-drawer/)
- [Draw Call Optimization Guide - TheGamedev.Guru](https://thegamedev.guru/unity-performance/draw-call-optimization/)
- [Catlike Coding - GPU Instancing Tutorial](https://catlikecoding.com/unity/tutorials/rendering/part-19/)
- [Unity Discussions - SRP Batcher vs GPU Instancing](https://discussions.unity.com/t/srp-batcher-vs-gpu-instancing/799622)
- [Unity Performance Optimization Guide 2025](https://generalistprogrammer.com/tutorials/unity-performance-optimization-complete-technical-guide-2025)

---

> 💡 본 문서는 웹 리서치 기반으로 작성되었으며, Unity 버전에 따라 내용이 변경될 수 있습니다.
> 💡 프로젝트 적용 시 반드시 Unity Profiler와 Frame Debugger로 실제 배칭 상태를 확인하시기 바랍니다.
