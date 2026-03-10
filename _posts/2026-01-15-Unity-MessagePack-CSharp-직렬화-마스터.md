---
layout: post
title: 260115 Unity MessagePack-CSharp 직렬화 마스터
comments: true
tags:
- tag0
- tag1
- tag2
---

# 🚀 Unity MessagePack-CSharp 직렬화 완전정복

> **"JSON보다 10배 빠르고, 바이너리 크기는 절반 이하!"**
> 

> Unity 게임 개발에서 고성능 직렬화의 필수 도구, MessagePack-CSharp를 마스터하세요.
> 

---

# 📚 목차

1. MessagePack 개요
2. 바이너리 포맷 상세 분석
3. Unity 설치 및 설정
4. Attribute 기반 직렬화
5. Code Generation & Source Generator
6. AOT/IL2CPP 완벽 대응
7. Custom Formatter 마스터
8. Resolver 시스템 심화
9. Unity 특화 타입 처리
10. LZ4 압축 최적화
11. 네트워크 통신 아키텍처
12. 게임 세이브/로드 시스템
13. JSON vs MessagePack 성능 비교
14. 멀티플레이어 패킷 설계
15. TypelessFormatter & 다형성
16. Union 타입 활용법
17. 버전 호환성 전략
18. 디버깅 및 검사 도구
19. 실전 예제: 인벤토리 시스템
20. 실전 예제: 랭킹 시스템
21. 고급 최적화 테크닉
22. 트러블슈팅 가이드

---

# 🎯 1. MessagePack 개요

## 1.1 MessagePack이란?

MessagePack은 **"JSON과 같지만 빠르고 작은"** 효율적인 바이너리 직렬화 포맷입니다. 🎁

### 핵심 특징

- ⚡ **10배 빠른 속도**: MessagePack-CSharp는 다른 C# 직렬화 라이브러리보다 10배 이상 빠릅니다
- 💾 **작은 크기**: JSON 대비 30-50% 수준의 바이너리 크기
- 🌍 **다중 언어 지원**: 50개 이상의 프로그래밍 언어 지원
- 🎮 **Unity 최적화**: Unity 전용 타입 및 IL2CPP 완벽 지원

### 실제 크기 비교

| 숫자 값 | JSON 크기 | MessagePack 크기 |
| --- | --- | --- |
| 10 | 2 bytes | 1 byte |
| 100 | 3 bytes | 1 byte |
| -108 | 4 bytes | 1 byte |
| 65535 | 5 bytes | 3 bytes |

## 1.2 왜 Unity 게임에 필요한가? 🎮

### 사용 시나리오

1. **🌐 네트워크 멀티플레이어**
    - 실시간 플레이어 위치/상태 동기화
    - 채팅 메시지 전송
    - 게임 이벤트 브로드캐스트
2. **💾 세이브/로드 시스템**
    - 플레이어 프로필 저장
    - 게임 진행 상황 저장
    - 설정 데이터 영구 보관
3. **📦 애셋 번들 데이터**
    - 대량의 게임 데이터 패킹
    - 다운로드 가능한 콘텐츠(DLC)
    - 메타데이터 관리
4. **🔄 서버 통신**
    - REST API 요청/응답
    - WebSocket 메시지
    - 랭킹/리더보드 동기화

---

# 🔬 2. 바이너리 포맷 상세 분석

## 2.1 MessagePack 타입 시스템

MessagePack은 다음과 같은 타입들을 지원합니다:

### 기본 타입

```
• Integer (int8, int16, int32, int64, uint8, uint16, uint32, uint64)
• Float (float32, float64)
• Boolean
• Nil (null)
• String (UTF-8)
• Binary (byte array)
```

### 컬렉션 타입

```
• Array (순서가 있는 리스트)
• Map (키-값 딕셔너리)
```

### Extension 타입

```
• Timestamp (날짜/시간)
• Custom Extension Types (사용자 정의 타입)
```

## 2.2 바이너리 인코딩 예제 📊

### 정수 인코딩

**Positive FixInt (0~127)**

```
값: 42
바이트: [0x2a]
크기: 1 byte
```

**uint 16**

```
값: 1000
바이트: [0xcd, 0x03, 0xe8]
크기: 3 bytes
```

### 문자열 인코딩

**FixStr (길이 0~31)**

```
값: "hello"
바이트: [0xa5, 0x68, 0x65, 0x6c, 0x6c, 0x6f]
      [길이5, 'h', 'e', 'l', 'l', 'o']
크기: 6 bytes
```

### 배열 인코딩

**FixArray (길이 0~15)**

```
값: [1, 2, 3]
바이트: [0x93, 0x01, 0x02, 0x03]
      [길이3, 값1, 값2, 값3]
크기: 4 bytes
```

## 2.3 MessagePack vs JSON 바이트 비교 ⚖️

### 예제 객체

```csharp
public class Player
{
    public int Id { get; set; } = 12345;
    public string Name { get; set; } = "Warrior";
    public float Health { get; set; } = 100.0f;
    public bool IsAlive { get; set; } = true;
}
```

### JSON 인코딩 결과

```json
{"Id":12345,"Name":"Warrior","Health":100.0,"IsAlive":true}
```

**크기: 60 bytes**

### MessagePack 인코딩 결과

```
[0x84, 0xa2, 0x49, 0x64, 0xcd, 0x30, 0x39, ...]
```

**크기: 약 28 bytes (53% 절약!)**

---

# 💿 3. Unity 설치 및 설정

## 3.1 시스템 요구사항 🔧

### Unity 버전

- **최소 요구사항**: Unity 2022.3.12f1
- **권장**: Unity 2022 LTS 이상
- **이유**: C# Source Generator 지원을 위해 필요

### .NET 환경

- **.NET Standard 2.1** 이상
- **C# 9.0** 이상 권장 (Source Generator 활용)

## 3.2 설치 방법 (2단계 필수!) 📦

> ⚠️ **중요**: 두 단계 모두 반드시 수행해야 합니다!
> 

### Step 1: NuGet에서 MessagePack 설치

1. **NuGetForUnity** 설치
    - Unity Package Manager에서 NuGetForUnity 추가
    - 또는 GitHub에서 다운로드: [NuGetForUnity](https://github.com/GlitchEnzo/NuGetForUnity)
2. **MessagePack 패키지 설치**
    
    ```
    1. Unity 메뉴: Window → NuGet → Manage NuGet Packages
    2. "MessagePack" 검색
    3. "MessagePack" 선택 후 Install 버튼 클릭
    ```
    

### Step 2: Unity 전용 패키지 설치

1. **Package Manager 열기**
    - Unity 메뉴: Window → Package Manager
2. **Git URL로 패키지 추가**
    
    ```
    1. 좌측 상단 "+" 버튼 클릭
    2. "Add package from git URL" 선택
    3. 다음 URL 입력:
    ```
    

```
https://github.com/MessagePack-CSharp/MessagePack-CSharp.git?path=src/MessagePack.UnityClient/Assets/Scripts/MessagePack
```

## 3.3 설치 확인 ✅

### 테스트 스크립트 작성

```csharp
using UnityEngine;
using MessagePack;
using System;

public class MessagePackTest : MonoBehaviour
{
    [MessagePackObject]
    public class TestData
    {
        [Key(0)]
        public int Value { get; set; }
        
        [Key(1)]
        public string Name { get; set; }
    }
    
    void Start()
    {
        // 직렬화 테스트
        var data = new TestData { Value = 42, Name = "Test" };
        byte[] bytes = MessagePackSerializer.Serialize(data);
        
        Debug.Log($"✅ MessagePack 설치 성공!");
        Debug.Log($"📦 Serialized size: {bytes.Length} bytes");
        
        // 역직렬화 테스트
        var restored = MessagePackSerializer.Deserialize<TestData>(bytes);
        Debug.Log($"🔄 Deserialized: Value={restored.Value}, Name={[restored.Name](http://restored.Name)}");
    }
}
```

### 예상 출력

```
✅ MessagePack 설치 성공!
📦 Serialized size: 8 bytes
🔄 Deserialized: Value=42, Name=Test
```

---

# 🏷️ 4. Attribute 기반 직렬화

## 4.1 MessagePackObject Attribute 🎯

### 기본 사용법

```csharp
using MessagePack;

[MessagePackObject]
public class PlayerData
{
    [Key(0)]
    public int PlayerId { get; set; }
    
    [Key(1)]
    public string PlayerName { get; set; }
    
    [Key(2)]
    public float Experience { get; set; }
    
    [Key(3)]
    public int Level { get; set; }
}
```

### Key Attribute 규칙

✅ **DO (권장)**

- 0부터 시작하는 연속된 정수 사용
- 순서대로 부여 (0, 1, 2, 3...)
- 삭제된 필드의 Key는 재사용하지 않기

❌ **DON'T (비권장)**

- 중간에 Key 건너뛰기 (0, 1, 3, 5...)
- 음수 Key 사용
- Key 순서 무작위 배치

## 4.2 String Key vs Integer Key 🔑

### Integer Key (권장) ⚡

```csharp
[MessagePackObject]
public class FastPlayer
{
    [Key(0)] public int Id { get; set; }
    [Key(1)] public string Name { get; set; }
}
```

**장점**

- ✅ 최소 바이너리 크기
- ✅ 최고 성능
- ✅ 네트워크 전송에 최적

**단점**

- ⚠️ 가독성 낮음
- ⚠️ 필드 순서 관리 필요

### String Key (선택) 📝

```csharp
[MessagePackObject(true)]  // keyAsPropertyName: true
public class ReadablePlayer
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

**장점**

- ✅ 높은 가독성
- ✅ JSON과 호환 가능
- ✅ 디버깅 용이

**단점**

- ⚠️ 크기 증가 (약 20-30%)
- ⚠️ 성능 저하 (약 15-20%)

## 4.3 IgnoreMember Attribute 🚫

```csharp
[MessagePackObject]
public class GameEntity
{
    [Key(0)]
    public int EntityId { get; set; }
    
    [Key(1)]
    public Vector3 Position { get; set; }
    
    // 직렬화에서 제외
    [IgnoreMember]
    public GameObject CachedGameObject { get; set; }
    
    [IgnoreMember]
    public Transform CachedTransform { get; set; }
}
```

### 제외해야 하는 필드 타입

- 🚫 Unity의 직렬화 불가능 타입 (GameObject, Component 등)
- 🚫 임시 캐시 데이터
- 🚫 계산된 속성 (Computed Properties)
- 🚫 순환 참조가 있는 객체

## 4.4 생성자 패턴 🏗️

### 기본 생성자 (권장)

```csharp
[MessagePackObject]
public class Item
{
    [Key(0)] public int ItemId { get; set; }
    [Key(1)] public string ItemName { get; set; }
    [Key(2)] public int Quantity { get; set; }
    
    // 파라미터 없는 생성자 (필수)
    public Item()
    {
    }
}
```

### SerializationConstructor (고급)

```csharp
[MessagePackObject]
public class ImmutableItem
{
    [Key(0)] public int ItemId { get; }
    [Key(1)] public string ItemName { get; }
    [Key(2)] public int Quantity { get; }
    
    [SerializationConstructor]
    public ImmutableItem(int itemId, string itemName, int quantity)
    {
        ItemId = itemId;
        ItemName = itemName;
        Quantity = quantity;
    }
}
```

**장점**

- ✅ Immutable 객체 지원
- ✅ 유효성 검사 로직 포함 가능
- ✅ 읽기 전용 속성 지원

---

# ⚙️ 5. Code Generation & Source Generator

## 5.1 Source Generator란? 🔧

MessagePack v3부터는 **C# Source Generator**를 사용하여 컴파일 타임에 Formatter 코드를 자동 생성합니다.

### 기존 방식 (mpc.exe) vs 새로운 방식 (Source Generator)

| 항목 | mpc.exe (구버전) | Source Generator (v3+) |
| --- | --- | --- |
| 실행 시점 | 수동 실행 필요 | 자동 (빌드 시) |
| Unity 통합 | 별도 설정 필요 | NuGet 설치 시 자동 |
| 유지보수 | 번거로움 | 편리함 |
| IL2CPP 지원 | 수동 생성 | 자동 지원 |
| 에러 감지 | 런타임 | 컴파일 타임 |

## 5.2 Source Generator 동작 원리 🔍

### Step 1: Attribute 분석

컴파일러가 `[MessagePackObject]` 어트리뷰트가 붙은 클래스를 탐지합니다.

```csharp
[MessagePackObject]  // ← Source Generator가 이것을 감지
public class PlayerStats
{
    [Key(0)] public int Health { get; set; }
    [Key(1)] public int Mana { get; set; }
}
```

### Step 2: Formatter 자동 생성

Source Generator가 다음과 같은 코드를 자동으로 생성합니다:

```csharp
// 자동 생성된 코드 (MessagePackGenerated.cs)
namespace MessagePack.Formatters
{
    public sealed class PlayerStatsFormatter : IMessagePackFormatter<PlayerStats>
    {
        public void Serialize(
            ref MessagePackWriter writer, 
            PlayerStats value, 
            MessagePackSerializerOptions options)
        {
            if (value == null)
            {
                writer.WriteNil();
                return;
            }
            
            writer.WriteArrayHeader(2);
            writer.Write([value.Health](http://value.Health));
            writer.Write(value.Mana);
        }
        
        public PlayerStats Deserialize(
            ref MessagePackReader reader, 
            MessagePackSerializerOptions options)
        {
            if (reader.TryReadNil())
                return null;
            
            var length = reader.ReadArrayHeader();
            var result = new PlayerStats();
            
            for (int i = 0; i < length; i++)
            {
                switch (i)
                {
                    case 0:
                        [result.Health](http://result.Health) = reader.ReadInt32();
                        break;
                    case 1:
                        result.Mana = reader.ReadInt32();
                        break;
                    default:
                        reader.Skip();
                        break;
                }
            }
            
            return result;
        }
    }
}
```

### Step 3: Resolver 등록

생성된 Formatter들은 자동으로 Resolver에 등록됩니다.

```csharp
// 자동 생성된 Resolver
public sealed class GeneratedResolver : IFormatterResolver
{
    public static readonly GeneratedResolver Instance = new GeneratedResolver();
    
    public IMessagePackFormatter<T> GetFormatter<T>()
    {
        return FormatterCache<T>.Formatter;
    }
    
    private static class FormatterCache<T>
    {
        public static readonly IMessagePackFormatter<T> Formatter;
        
        static FormatterCache()
        {
            // 타입별 Formatter 매핑
            if (typeof(T) == typeof(PlayerStats))
            {
                Formatter = (IMessagePackFormatter<T>)(object)new PlayerStatsFormatter();
            }
            // ... 다른 타입들
        }
    }
}
```

## 5.3 Source Generator 설정 확인 ✅

### Unity에서 확인하기

1. **생성된 파일 위치**
    
    ```
    Library/Bee/artifacts/[config]/MessagePackGenerated/
    ```
    
2. **Visual Studio에서 확인**
    - Solution Explorer에서 Dependencies → Analyzers → MessagePack 확장
    - 생성된 소스 파일 확인 가능
3. **로그 확인**
    
    ```csharp
    Debug.Log($"Resolver 타입: {MessagePackSerializer.DefaultOptions.Resolver.GetType()}");
    ```
    

---

# 🎮 6. AOT/IL2CPP 완벽 대응

## 6.1 AOT란? 🔍

**AOT (Ahead-Of-Time) Compilation**은 런타임이 아닌 빌드 타임에 코드를 컴파일하는 방식입니다.

### Unity에서 AOT를 사용하는 플랫폼

- 📱 **iOS** (Apple 정책에 의해 강제)
- 🎮 **Nintendo Switch**
- 🎯 **PlayStation**
- 📦 **Xbox**
- 🌐 **WebGL** (일부 제약)

## 6.2 IL2CPP 문제점과 해결 ⚠️

### 문제 1: 동적 코드 생성 불가

**문제 상황**

```csharp
// ❌ IL2CPP에서 실패하는 코드
var options = MessagePackSerializerOptions.Standard
    .WithResolver(ContractlessStandardResolver.Instance);  // 런타임 코드 생성 시도
```

**에러 메시지**

```
ExecutionEngineException: Attempting to call method 
'MessagePack.Formatters.DynamicFormatter<T>::Serialize' 
for which no ahead of time (AOT) code was generated.
```

### 해결책 1: Source Generator 사용 (v3+)

```csharp
// ✅ v3에서는 자동으로 AOT 대응 코드 생성
[MessagePackObject]
public class GameData
{
    [Key(0)] public int Score { get; set; }
}

// NuGet 설치 시 Source Generator가 자동으로 AOT 코드 생성
```

### 해결책 2: StaticCompositeResolver 사용

```csharp
using MessagePack;
using MessagePack.Resolvers;
using UnityEngine;

public class MessagePackInitializer
{
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
    private static void Initialize()
    {
        // IL2CPP 대응 Resolver 설정
        StaticCompositeResolver.Instance.Register(
            // 1. 자동 생성된 Resolver (가장 우선)
            GeneratedResolver.Instance,
            
            // 2. Unity 전용 Resolver
            MessagePack.Unity.UnityResolver.Instance,
            
            // 3. Unity Blit Resolver (Vector3[] 등 고속 직렬화)
            MessagePack.Unity.Extension.UnityBlitWithPrimitiveArrayResolver.Instance,
            
            // 4. 기본 타입 Resolver
            StandardResolver.Instance
        );
        
        var options = MessagePackSerializerOptions.Standard
            .WithResolver(StaticCompositeResolver.Instance);
        
        MessagePackSerializer.DefaultOptions = options;
        
        Debug.Log("✅ MessagePack IL2CPP 설정 완료!");
    }
}
```

## 6.3 Unity Build 설정 🔧

### Android IL2CPP 설정

```
1. File → Build Settings → Android
2. Player Settings → Other Settings
3. Scripting Backend: IL2CPP
4. Target Architectures: ARM64 ✅
5. API Compatibility Level: .NET Standard 2.1
```

### iOS 설정

```
1. File → Build Settings → iOS
2. Player Settings → Other Settings
3. Scripting Backend: IL2CPP (기본값)
4. Target SDK: Device SDK
5. Architecture: ARM64
```

## 6.4 IL2CPP 테스트 스크립트 🧪

```csharp
using UnityEngine;
using MessagePack;
using System.Collections.Generic;

public class IL2CPPTest : MonoBehaviour
{
    [MessagePackObject]
    public class TestData
    {
        [Key(0)] public int Value { get; set; }
        [Key(1)] public string Name { get; set; }
        [Key(2)] public List<int> Numbers { get; set; }
        [Key(3)] public Vector3 Position { get; set; }
    }
    
    void Start()
    {
        try
        {
            // 복잡한 데이터 직렬화 테스트
            var data = new TestData
            {
                Value = 12345,
                Name = "IL2CPP Test",
                Numbers = new List<int> { 1, 2, 3, 4, 5 },
                Position = new Vector3(10f, 20f, 30f)
            };
            
            byte[] bytes = MessagePackSerializer.Serialize(data);
            var restored = MessagePackSerializer.Deserialize<TestData>(bytes);
            
            Debug.Log($"✅ IL2CPP 직렬화 성공!");
            Debug.Log($"📦 크기: {bytes.Length} bytes");
            Debug.Log($"🔄 복원: Value={restored.Value}, Position={restored.Position}");
        }
        catch (System.Exception ex)
        {
            Debug.LogError($"❌ IL2CPP 오류: {ex.Message}");
        }
    }
}
```

---

# 🎨 7. Custom Formatter 마스터

## 7.1 Custom Formatter란? 🖌️

Custom Formatter는 **제3자 라이브러리 타입**이나 **특수한 직렬화 로직**이 필요한 경우에 사용합니다.

### 사용 시나리오

- 🔒 암호화된 직렬화
- 🗜️ 압축 알고리즘 적용
- 🎯 특정 필드만 선택적 직렬화
- 🔄 레거시 포맷 호환
- 🎮 Unity 커스텀 컴포넌트

## 7.2 IMessagePackFormatter 인터페이스 📋

```csharp
public interface IMessagePackFormatter<T>
{
    void Serialize(
        ref MessagePackWriter writer, 
        T value, 
        MessagePackSerializerOptions options);
    
    T Deserialize(
        ref MessagePackReader reader, 
        MessagePackSerializerOptions options);
}
```

## 7.3 예제 1: Color32 Formatter 🎨

```csharp
using MessagePack;
using MessagePack.Formatters;
using UnityEngine;

public sealed class Color32Formatter : IMessagePackFormatter<Color32>
{
    // 싱글톤 인스턴스
    public static readonly Color32Formatter Instance = new Color32Formatter();
    
    private Color32Formatter() { }
    
    public void Serialize(
        ref MessagePackWriter writer, 
        Color32 value, 
        MessagePackSerializerOptions options)
    {
        // 4개의 byte를 uint로 패킹 (4 bytes → 5 bytes)
        writer.WriteArrayHeader(4);
        writer.Write(value.r);
        writer.Write(value.g);
        writer.Write(value.b);
        writer.Write(value.a);
    }
    
    public Color32 Deserialize(
        ref MessagePackReader reader, 
        MessagePackSerializerOptions options)
    {
        var count = reader.ReadArrayHeader();
        
        byte r = reader.ReadByte();
        byte g = reader.ReadByte();
        byte b = reader.ReadByte();
        byte a = reader.ReadByte();
        
        return new Color32(r, g, b, a);
    }
}
```

## 7.4 예제 2: 압축 String Formatter 🗜️

```csharp
using MessagePack;
using MessagePack.Formatters;
using [System.IO](http://System.IO);
using [System.IO](http://System.IO).Compression;
using System.Text;

public sealed class CompressedStringFormatter : IMessagePackFormatter<string>
{
    public static readonly CompressedStringFormatter Instance = new CompressedStringFormatter();
    
    // 이 크기 이상일 때만 압축
    private const int CompressionThreshold = 128;
    
    public void Serialize(
        ref MessagePackWriter writer, 
        string value, 
        MessagePackSerializerOptions options)
    {
        if (value == null)
        {
            writer.WriteNil();
            return;
        }
        
        byte[] bytes = Encoding.UTF8.GetBytes(value);
        
        // 작은 문자열은 압축하지 않음
        if (bytes.Length < CompressionThreshold)
        {
            writer.WriteArrayHeader(2);
            writer.Write(false);  // 압축 안함
            writer.Write(bytes);
            return;
        }
        
        // GZip 압축
        using (var output = new MemoryStream())
        {
            using (var gzip = new GZipStream(output, CompressionMode.Compress))
            {
                gzip.Write(bytes, 0, bytes.Length);
            }
            
            byte[] compressed = output.ToArray();
            
            writer.WriteArrayHeader(2);
            writer.Write(true);  // 압축됨
            writer.Write(compressed);
        }
    }
    
    public string Deserialize(
        ref MessagePackReader reader, 
        MessagePackSerializerOptions options)
    {
        if (reader.TryReadNil())
            return null;
        
        var count = reader.ReadArrayHeader();
        bool isCompressed = reader.ReadBoolean();
        byte[] bytes = reader.ReadBytes().Value.ToArray();
        
        if (!isCompressed)
        {
            return Encoding.UTF8.GetString(bytes);
        }
        
        // GZip 압축 해제
        using (var input = new MemoryStream(bytes))
        using (var gzip = new GZipStream(input, CompressionMode.Decompress))
        using (var output = new MemoryStream())
        {
            gzip.CopyTo(output);
            return Encoding.UTF8.GetString(output.ToArray());
        }
    }
}
```

## 7.5 예제 3: 암호화 Formatter 🔒

```csharp
using MessagePack;
using MessagePack.Formatters;
using [System.Security](http://System.Security).Cryptography;
using [System.IO](http://System.IO);

public sealed class EncryptedDataFormatter<T> : IMessagePackFormatter<T>
{
    private readonly byte[] _key;
    private readonly byte[] _iv;
    
    public EncryptedDataFormatter(byte[] key, byte[] iv)
    {
        _key = key;
        _iv = iv;
    }
    
    public void Serialize(
        ref MessagePackWriter writer, 
        T value, 
        MessagePackSerializerOptions options)
    {
        // 1. 원본 데이터 직렬화
        byte[] plainBytes = MessagePackSerializer.Serialize(value, options);
        
        // 2. AES 암호화
        using (var aes = Aes.Create())
        {
            aes.Key = _key;
            aes.IV = _iv;
            
            using (var encryptor = aes.CreateEncryptor())
            using (var ms = new MemoryStream())
            {
                using (var cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Write))
                {
                    cs.Write(plainBytes, 0, plainBytes.Length);
                }
                
                byte[] encrypted = ms.ToArray();
                writer.Write(encrypted);
            }
        }
    }
    
    public T Deserialize(
        ref MessagePackReader reader, 
        MessagePackSerializerOptions options)
    {
        byte[] encrypted = reader.ReadBytes().Value.ToArray();
        
        // 1. AES 복호화
        using (var aes = Aes.Create())
        {
            aes.Key = _key;
            aes.IV = _iv;
            
            using (var decryptor = aes.CreateDecryptor())
            using (var ms = new MemoryStream(encrypted))
            using (var cs = new CryptoStream(ms, decryptor, [CryptoStreamMode.Read](http://CryptoStreamMode.Read)))
            using (var output = new MemoryStream())
            {
                cs.CopyTo(output);
                byte[] plainBytes = output.ToArray();
                
                // 2. 원본 데이터 역직렬화
                return MessagePackSerializer.Deserialize<T>(plainBytes, options);
            }
        }
    }
}
```

## 7.6 Custom Formatter 등록 📝

```csharp
using MessagePack;
using MessagePack.Resolvers;
using UnityEngine;

public class CustomFormatterInitializer
{
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
    private static void Initialize()
    {
        // Custom Resolver 생성
        var resolver = CompositeResolver.Create(
            // Custom Formatters
            new IMessagePackFormatter[]
            {
                Color32Formatter.Instance,
                CompressedStringFormatter.Instance,
            },
            // Standard Resolvers
            new IFormatterResolver[]
            {
                GeneratedResolver.Instance,
                StandardResolver.Instance
            }
        );
        
        var options = MessagePackSerializerOptions.Standard.WithResolver(resolver);
        MessagePackSerializer.DefaultOptions = options;
        
        Debug.Log("✅ Custom Formatter 등록 완료!");
    }
}
```

---

# 🔧 8. Resolver 시스템 심화

## 8.1 Resolver란? 🔍

**Resolver**는 타입에 맞는 Formatter를 찾아주는 역할을 합니다.

```
타입 T → Resolver → IMessagePackFormatter<T> → Serialize/Deserialize
```

## 8.2 Built-in Resolvers 📚

### StandardResolver

```csharp
StandardResolver.Instance
```

**지원 타입**

- Primitive types (int, float, string, bool 등)
- Collection types (List, Dictionary, Array 등)
- Nullable types
- DateTime, TimeSpan, Guid
- BigInteger, Uri

### ContractlessStandardResolver

```csharp
ContractlessStandardResolver.Instance
```

**특징**

- ✅ [MessagePackObject] Attribute 불필요
- ✅ 자동으로 모든 public 속성 직렬화
- ❌ IL2CPP에서 사용 불가 (동적 코드 생성)

### UnityResolver

```csharp
MessagePack.Unity.UnityResolver.Instance
```

**지원 타입**

```csharp
• Vector2, Vector3, Vector4
• Quaternion
• Color, Color32
• Bounds
• Rect
• Matrix4x4
• AnimationCurve, Keyframe
• Gradient
• RectOffset
• LayerMask
• Vector2Int, Vector3Int
• RangeInt, RectInt, BoundsInt
```

### UnityBlitResolver

```csharp
MessagePack.Unity.Extension.UnityBlitWithPrimitiveArrayResolver.Instance
```

**특징**

- ⚡ **극한의 성능**: 메모리 직접 복사 (Blitting)
- 🎯 **배열 특화**: Vector3[], Quaternion[], Color[] 등
- 📦 **Extension Format**: typecode 30~39 사용

**성능 비교**

```
JsonUtility.FromJson<Vector3[]>():  100ms
MessagePack (일반):                  15ms  (6.7배 빠름)
MessagePack (Blit):                   5ms  (20배 빠름!)
```

## 8.3 Resolver 우선순위 체계 📊

```csharp
StaticCompositeResolver.Instance.Register(
    // 1순위: Custom Formatters
    CustomResolver.Instance,
    
    // 2순위: 프로젝트 Generated Resolver
    GeneratedResolver.Instance,
    
    // 3순위: Unity 특화 Resolver
    UnityBlitResolver.Instance,
    UnityResolver.Instance,
    
    // 4순위: 표준 Resolver
    StandardResolver.Instance
);
```

**동작 원리**

1. 위에서 아래로 순서대로 탐색
2. 첫 번째로 Formatter를 찾으면 즉시 반환
3. 없으면 다음 Resolver로 이동
4. 모두 실패하면 예외 발생

## 8.4 Custom Resolver 구현 🛠️

```csharp
using MessagePack;
using MessagePack.Formatters;

public sealed class GameDataResolver : IFormatterResolver
{
    public static readonly GameDataResolver Instance = new GameDataResolver();
    
    private GameDataResolver() { }
    
    public IMessagePackFormatter<T> GetFormatter<T>()
    {
        return FormatterCache<T>.Formatter;
    }
    
    private static class FormatterCache<T>
    {
        public static readonly IMessagePackFormatter<T> Formatter;
        
        static FormatterCache()
        {
            // 타입별 Formatter 매핑
            if (typeof(T) == typeof(Color32))
            {
                Formatter = (IMessagePackFormatter<T>)(object)Color32Formatter.Instance;
            }
            else if (typeof(T) == typeof(string))
            {
                Formatter = (IMessagePackFormatter<T>)(object)CompressedStringFormatter.Instance;
            }
            // ... 다른 타입들
        }
    }
}
```

---

# 🎮 9. Unity 특화 타입 처리

## 9.1 Vector3 직렬화 📍

### 기본 사용법

```csharp
using UnityEngine;
using MessagePack;

[MessagePackObject]
public class TransformData
{
    [Key(0)] public Vector3 Position { get; set; }
    [Key(1)] public Vector3 Rotation { get; set; }
    [Key(2)] public Vector3 Scale { get; set; }
}

// 사용 예제
var data = new TransformData
{
    Position = new Vector3(10f, 20f, 30f),
    Rotation = new Vector3(0f, 90f, 0f),
    Scale = [Vector3.one](http://Vector3.one)
};

byte[] bytes = MessagePackSerializer.Serialize(data);
var restored = MessagePackSerializer.Deserialize<TransformData>(bytes);

Debug.Log($"Position: {restored.Position}");  // (10.0, 20.0, 30.0)
```

### Vector3 배열 고속 직렬화 ⚡

```csharp
using MessagePack;
using UnityEngine;

public class MeshDataOptimized
{
    // UnityBlitResolver를 사용하면 20배 빠름!
    public Vector3[] Vertices { get; set; }
    public Vector3[] Normals { get; set; }
    public Vector2[] UVs { get; set; }
    public int[] Triangles { get; set; }
}

// 초기화 코드에서 UnityBlitResolver 등록 필수
StaticCompositeResolver.Instance.Register(
    MessagePack.Unity.Extension.UnityBlitWithPrimitiveArrayResolver.Instance,
    // ... other resolvers
);
```

**성능 비교 (10,000 Vertices)**

```
JsonUtility:        2500ms
MessagePack:         200ms  (12.5배)
MessagePack (Blit):  125ms  (20배!)
```

## 9.2 Quaternion 직렬화 🔄

```csharp
[MessagePackObject]
public class RotationData
{
    [Key(0)] public Quaternion Rotation { get; set; }
    [Key(1)] public Quaternion[] BoneRotations { get; set; }  // 애니메이션 본 회전
}

// 사용 예제
var data = new RotationData
{
    Rotation = Quaternion.Euler(45f, 90f, 180f),
    BoneRotations = new Quaternion[]
    {
        Quaternion.identity,
        Quaternion.Euler(10f, 0f, 0f),
        Quaternion.Euler(0f, 20f, 0f)
    }
};

byte[] bytes = MessagePackSerializer.Serialize(data);
```

## 9.3 Color & Color32 🎨

### Color (float)

```csharp
[MessagePackObject]
public class MaterialData
{
    [Key(0)] public Color MainColor { get; set; }        // 16 bytes (RGBA float)
    [Key(1)] public Color EmissionColor { get; set; }
}

var data = new MaterialData
{
    MainColor = new Color(1f, 0.5f, 0.25f, 1f),
    EmissionColor = [Color.red](http://Color.red)
};
```

### Color32 (byte) - 메모리 효율적

```csharp
[MessagePackObject]
public class SpriteData
{
    [Key(0)] public Color32 TintColor { get; set; }      // 4 bytes (RGBA byte)
    [Key(1)] public Color32[] Pixels { get; set; }       // 텍스처 데이터
}

var data = new SpriteData
{
    TintColor = new Color32(255, 128, 64, 255),
    Pixels = new Color32[1024 * 1024]  // 1MB
};
```

**크기 비교**

```
Color:   16 bytes per color
Color32:  4 bytes per color  (75% 절약)
```

## 9.4 Unity 커스텀 타입 🛠️

### Transform Snapshot

```csharp
using UnityEngine;
using MessagePack;

[MessagePackObject]
public struct TransformSnapshot
{
    [Key(0)] public Vector3 Position { get; set; }
    [Key(1)] public Quaternion Rotation { get; set; }
    [Key(2)] public Vector3 LocalScale { get; set; }
    
    // Transform에서 스냅샷 생성
    public static TransformSnapshot Capture(Transform transform)
    {
        return new TransformSnapshot
        {
            Position = transform.position,
            Rotation = transform.rotation,
            LocalScale = transform.localScale
        };
    }
    
    // Transform에 적용
    public void ApplyTo(Transform transform)
    {
        transform.position = Position;
        transform.rotation = Rotation;
        transform.localScale = LocalScale;
    }
}
```

### Bounds & Rect

```csharp
[MessagePackObject]
public class ColliderData
{
    [Key(0)] public Bounds Bounds { get; set; }
    [Key(1)] public Rect ScreenRect { get; set; }
    [Key(2)] public RectInt GridRect { get; set; }
}

var data = new ColliderData
{
    Bounds = new Bounds([Vector3.zero](http://Vector3.zero), [Vector3.one](http://Vector3.one) * 10f),
    ScreenRect = new Rect(0, 0, 1920, 1080),
    GridRect = new RectInt(0, 0, 64, 64)
};
```

### AnimationCurve

```csharp
[MessagePackObject]
public class AnimationData
{
    [Key(0)] public AnimationCurve SpeedCurve { get; set; }
    [Key(1)] public Keyframe[] CustomKeyframes { get; set; }
}

var data = new AnimationData
{
    SpeedCurve = AnimationCurve.EaseInOut(0f, 0f, 1f, 1f),
    CustomKeyframes = new Keyframe[]
    {
        new Keyframe(0f, 0f),
        new Keyframe(0.5f, 1f),
        new Keyframe(1f, 0f)
    }
};
```

---

# 🗜️ 10. LZ4 압축 최적화

## 10.1 LZ4 압축이란? 📦

**LZ4**는 극히 빠른 압축 알고리즘입니다.

### 특징 비교

| 항목 | LZ4 | GZip | Zstd |
| --- | --- | --- | --- |
| 압축 속도 | ★★★★★ | ★★ | ★★★ |
| 압축률 | ★★★ | ★★★★ | ★★★★★ |
| 해제 속도 | ★★★★★ | ★★★ | ★★★★ |
| 사용 예 | 실시간 게임 | 파일 저장 | 근최 DLC |

### 실제 성능 데이터

```
원본 크기:    5000 bytes
LZ4 압축:     2700 bytes  (46% 감소)
압축 시간:    0.15ms
해제 시간:    0.08ms
```

## 10.2 LZ4 사용법 🛠️

### 기본 사용

```csharp
using MessagePack;
using UnityEngine;

public class LZ4Example : MonoBehaviour
{
    [MessagePackObject]
    public class LargeData
    {
        [Key(0)] public string[] Messages { get; set; }
        [Key(1)] public int[] Numbers { get; set; }
    }
    
    void TestLZ4Compression()
    {
        var data = new LargeData
        {
            Messages = new string[1000],
            Numbers = new int[1000]
        };
        
        // LZ4 옵션 설정
        var options = MessagePackSerializerOptions.Standard
            .WithCompression(MessagePackCompression.Lz4BlockArray);
        
        // 압축 직렬화
        byte[] compressed = MessagePackSerializer.Serialize(data, options);
        
        // 일반 직렬화 비교
        byte[] normal = MessagePackSerializer.Serialize(data);
        
        Debug.Log($"일반: {normal.Length} bytes");
        Debug.Log($"LZ4: {compressed.Length} bytes");
        Debug.Log($"압축률: {(1f - (float)compressed.Length / normal.Length) * 100f:F1}%");
        
        // 자동 해제되어 복원
        var restored = MessagePackSerializer.Deserialize<LargeData>(compressed, options);
    }
}
```

## 10.3 LZ4 압축 모드 📊

### Lz4Block vs Lz4BlockArray

**Lz4Block** - 단일 블록

```csharp
var options = MessagePackSerializerOptions.Standard
    .WithCompression(MessagePackCompression.Lz4Block);
```

- ✅ 단순한 구조
- ❌ 큰 데이터에 비효율적

**Lz4BlockArray** - 여러 블록 (권장)

```csharp
var options = MessagePackSerializerOptions.Standard
    .WithCompression(MessagePackCompression.Lz4BlockArray);
```

- ✅ 큰 데이터에 최적화
- ✅ 스트리밍 가능
- ✅ 부분 압축 해제 가능

## 10.4 압축 적용 기준 📝

### 언제 LZ4를 사용할까?

✅ **사용 권장**

- 네트워크 전송 데이터
- 1KB 이상의 데이터
- 반복적인 패턴이 있는 데이터
- 큰 JSON 유사 구조

❌ **사용 비권장**

- 이미 작은 데이터 (<100 bytes)
- 랜덤 바이너리 데이터
- 실시간 60FPS 게임 루프 내부
- 이미 압축된 데이터 (PNG, MP3 등)

### 성능 테스트 스크립트

```csharp
using System.Diagnostics;
using UnityEngine;
using MessagePack;

public class CompressionBenchmark : MonoBehaviour
{
    [MessagePackObject]
    public class TestData
    {
        [Key(0)] public string LongText { get; set; }
        [Key(1)] public int[] Numbers { get; set; }
    }
    
    void Start()
    {
        var data = new TestData
        {
            LongText = new string('A', 10000),  // 10KB 문자열
            Numbers = new int[1000]
        };
        
        // 테스트 1: 압축 없음
        var sw1 = Stopwatch.StartNew();
        byte[] normal = MessagePackSerializer.Serialize(data);
        sw1.Stop();
        
        // 테스트 2: LZ4 압축
        var options = MessagePackSerializerOptions.Standard
            .WithCompression(MessagePackCompression.Lz4BlockArray);
        
        var sw2 = Stopwatch.StartNew();
        byte[] compressed = MessagePackSerializer.Serialize(data, options);
        sw2.Stop();
        
        // 결과 출력
        Debug.Log($"=== 압축 비교 ===");
        Debug.Log($"일반: {normal.Length:N0} bytes, {sw1.Elapsed.TotalMilliseconds:F3}ms");
        Debug.Log($"LZ4:  {compressed.Length:N0} bytes, {sw2.Elapsed.TotalMilliseconds:F3}ms");
        Debug.Log($"크기 감소: {(1f - (float)compressed.Length / normal.Length) * 100f:F1}%");
        
        // 해제 성능 테스트
        var sw3 = Stopwatch.StartNew();
        var restored = MessagePackSerializer.Deserialize<TestData>(compressed, options);
        sw3.Stop();
        
        Debug.Log($"해제: {sw3.Elapsed.TotalMilliseconds:F3}ms");
    }
}
```

---

# 🌐 11. 네트워크 통신 아키텍처

## 11.1 패킷 설계 원칙 📦

### 기본 원칙

1. **크기 최소화**: 한 패킷은 64 bytes 이하 권장
2. **타입 식별자**: 첫 바이트는 패킷 타입 ID
3. **고정 크기**: 동적 할당 피하기
4. **부호 범위**: ushort 사용 (0~65535)
5. **비트 패킹**: bool 플래그들은 하나의 byte로

## 11.2 패킷 타입 설계 🏷️

### Packet Type Enum

```csharp
public enum PacketType : byte
{
    // 플레이어 이동 (1~10)
    PlayerMove = 1,
    PlayerJump = 2,
    PlayerAttack = 3,
    
    // 채팅 (11~20)
    ChatMessage = 11,
    
    // 게임 상태 (21~30)
    GameStart = 21,
    GameEnd = 22,
    
    // 동기화 (31~40)
    SyncTransform = 31,
    SyncHealth = 32,
    
    // 인벤토리 (41~50)
    ItemAdd = 41,
    ItemRemove = 42,
    ItemUse = 43,
}
```

## 11.3 기본 패킷 구조 📦

### Packet Base Class

```csharp
using MessagePack;

[Union(0, typeof(PlayerMovePacket))]
[Union(1, typeof(PlayerAttackPacket))]
[Union(2, typeof(ChatMessagePacket))]
[MessagePackObject]
public abstract class Packet
{
    [Key(0)]
    public byte PacketType { get; set; }
    
    [Key(1)]
    public uint Timestamp { get; set; }  // 밀리초 타임스탬프
}
```

### 플레이어 이동 패킷

```csharp
[MessagePackObject]
public class PlayerMovePacket : Packet
{
    [Key(2)] public ushort PlayerId { get; set; }      // 2 bytes
    [Key(3)] public Vector3 Position { get; set; }     // 12 bytes (3 * float)
    [Key(4)] public byte Rotation { get; set; }        // 1 byte (0~255 = 0~360도)
    [Key(5)] public byte Flags { get; set; }           // 1 byte (비트 플래그)
    
    // 총 크기: 약 18 bytes
    
    // Flags 비트 마스크
    public bool IsRunning
    {
        get => (Flags & 0x01) != 0;
        set => Flags = value ? (byte)(Flags | 0x01) : (byte)(Flags & ~0x01);
    }
    
    public bool IsJumping
    {
        get => (Flags & 0x02) != 0;
        set => Flags = value ? (byte)(Flags | 0x02) : (byte)(Flags & ~0x02);
    }
    
    public bool IsCrouching
    {
        get => (Flags & 0x04) != 0;
        set => Flags = value ? (byte)(Flags | 0x04) : (byte)(Flags & ~0x04);
    }
}
```

### 공격 패킷

```csharp
[MessagePackObject]
public class PlayerAttackPacket : Packet
{
    [Key(2)] public ushort AttackerId { get; set; }    // 2 bytes
    [Key(3)] public ushort TargetId { get; set; }      // 2 bytes
    [Key(4)] public byte SkillId { get; set; }         // 1 byte
    [Key(5)] public ushort Damage { get; set; }        // 2 bytes
    [Key(6)] public byte CriticalFlag { get; set; }    // 1 byte (0=normal, 1=crit)
    
    // 총 크기: 약 10 bytes
}
```

## 11.4 Network Manager 구현 🌐

```csharp
using UnityEngine;
using MessagePack;
using System;
using [System.Net](http://System.Net).Sockets;

public class NetworkManager : MonoBehaviour
{
    private TcpClient _client;
    private NetworkStream _stream;
    
    // LZ4 압축 옵션
    private MessagePackSerializerOptions _options;
    
    void Start()
    {
        // MessagePack 설정
        _options = MessagePackSerializerOptions.Standard
            .WithCompression(MessagePackCompression.Lz4BlockArray);
        
        ConnectToServer("127.0.0.1", 7777);
    }
    
    void ConnectToServer(string host, int port)
    {
        _client = new TcpClient();
        _client.Connect(host, port);
        _stream = _client.GetStream();
        
        Debug.Log($"✅ 서버 연결: {host}:{port}");
    }
    
    // 패킷 전송
    public void SendPacket<T>(T packet) where T : Packet
    {
        try
        {
            // 1. 직렬화
            byte[] data = MessagePackSerializer.Serialize(packet, _options);
            
            // 2. 크기 전송 (4 bytes)
            byte[] lengthPrefix = BitConverter.GetBytes(data.Length);
            _stream.Write(lengthPrefix, 0, 4);
            
            // 3. 데이터 전송
            _stream.Write(data, 0, data.Length);
            
            Debug.Log($"📤 패킷 전송: {typeof(T).Name} ({data.Length} bytes)");
        }
        catch (Exception ex)
        {
            Debug.LogError($"❌ 전송 실패: {ex.Message}");
        }
    }
    
    // 패킷 수신
    public T ReceivePacket<T>() where T : Packet
    {
        try
        {
            // 1. 크기 수신 (4 bytes)
            byte[] lengthBuffer = new byte[4];
            _[stream.Read](http://stream.Read)(lengthBuffer, 0, 4);
            int length = BitConverter.ToInt32(lengthBuffer, 0);
            
            // 2. 데이터 수신
            byte[] dataBuffer = new byte[length];
            int totalRead = 0;
            while (totalRead < length)
            {
                int read = _[stream.Read](http://stream.Read)(dataBuffer, totalRead, length - totalRead);
                totalRead += read;
            }
            
            // 3. 역직렬화
            T packet = MessagePackSerializer.Deserialize<T>(dataBuffer, _options);
            
            Debug.Log($"📥 패킷 수신: {typeof(T).Name} ({length} bytes)");
            return packet;
        }
        catch (Exception ex)
        {
            Debug.LogError($"❌ 수신 실패: {ex.Message}");
            return null;
        }
    }
}
```

## 11.5 패킷 최적화 테크닉 ⚡

### 1. Delta Compression

```csharp
[MessagePackObject]
public class PlayerStateDelta
{
    [Key(0)] public ushort PlayerId { get; set; }
    
    // 번경된 값만 전송 (비트 플래그로 판별)
    [Key(1)] public byte ChangedFlags { get; set; }  // 어떤 필드가 변경되었는지
    [Key(2)] public Vector3? Position { get; set; }  // ChangedFlags & 0x01
    [Key(3)] public byte? Health { get; set; }       // ChangedFlags & 0x02
    [Key(4)] public ushort? Score { get; set; }      // ChangedFlags & 0x04
}
```

### 2. Quantization (양자화)

```csharp
public static class QuantizationHelper
{
    // float (4 bytes) → ushort (2 bytes)
    public static ushort QuantizeFloat(float value, float min, float max)
    {
        float normalized = Mathf.Clamp01((value - min) / (max - min));
        return (ushort)(normalized * 65535f);
    }
    
    public static float DequantizeFloat(ushort value, float min, float max)
    {
        float normalized = value / 65535f;
        return min + normalized * (max - min);
    }
    
    // Vector3 양자화 (12 bytes → 6 bytes)
    public static void QuantizeVector3(Vector3 v, out ushort x, out ushort y, out ushort z)
    {
        x = QuantizeFloat(v.x, -1000f, 1000f);
        y = QuantizeFloat(v.y, 0f, 500f);
        z = QuantizeFloat(v.z, -1000f, 1000f);
    }
}

```

---

# 💾 12. 게임 세이브/로드 시스템

## 12.1 세이브 데이터 구조 🏗️

참고: [Unity Save System Documentation](https://discussions.unity.com/t/save-system/936555)

```csharp
[MessagePackObject]
public class SaveData
{
    [Key(0)] public string PlayerId { get; set; }
    [Key(1)] public int Level { get; set; }
    [Key(2)] public Vector3 Position { get; set; }
    [Key(3)] public List<Item> Inventory { get; set; }
}
```

---

# 📈 13. JSON vs MessagePack 성능

참고:

- [MessagePack vs JSON Performance](https://neuecc.medium.com/messagepack-for-c-v2-new-era-of-net-core-unity-i-o-pipelines-6950643c1053)
- [Unity Serialization Comparison](https://discussions.unity.com/t/json-bson-messagepack-protobuf-etc/846512)

**Vector3[] 10,000개 직렬화**

- JsonUtility: 2500ms
- MessagePack: 200ms (12.5배)
- MessagePack Blit: 125ms (20배!)

---

# 🎮 14. 멀티플레이어 패킷

참고: [Unity Multiplayer Networking](https://moldstud.com/articles/p-creating-custom-network-messages-in-unity-a-comprehensive-developers-guide)

**베스트 프랙티스**

- 패킷 크기: 64 bytes 이하
- 첨 바이트는 패킷 ID
- 고정 크기 구조 선호
- ushort 사용 (0-65535)

---

# 🔄 15. TypelessFormatter

참고: [MessagePack Polymorphism](https://discussions.unity.com/t/messagepack-polymorphism-issue/811144)

```csharp
// 다형성 지원
var data = MessagePackSerializer.Typeless.Serialize(obj);
var restored = MessagePackSerializer.Typeless.Deserialize(data);
```

⚠️ **보안 주의**: 신뢰할 수 없는 데이터에는 사용 금지

---

# 🏷️ 16. Union 타입

참고: [MessagePack Union Types](https://github.com/MessagePack-CSharp/MessagePack-CSharp)

```csharp
[Union(0, typeof(DamageEvent))]
[Union(1, typeof(HealEvent))]
public interface IGameEvent { }

[MessagePackObject]
public class DamageEvent : IGameEvent
{
    [Key(0)] public int Damage { get; set; }
}
```

**중요**: 직렬화 시 명시적 타입 지정 필수

```csharp
MessagePackSerializer.Serialize<IGameEvent>(evt);
```

---

# 🔄 17. 버전 호환성

참고: [MessagePack Versioning](https://github.com/msgpack/msgpack/issues/142)

**호환성 전략**

1. **Array 방식** (권장)
    - 새 필드는 항상 끝에 추가
    - 구 버전은 추가 필드 무시
    - 신 버전은 기본값 사용
2. **Map 방식**
    - String Key 사용
    - 유연하지만 크기 증가

```csharp
[MessagePackObject]
public class VersionedData
{
    [Key(0)] public int Version { get; set; } = 2;
    [Key(1)] public string Name { get; set; }
    [Key(2)] public int? NewField { get; set; }  // v2에서 추가
}
```

---

# 🔍 18. 디버깅 도구

참고: [MessagePack Debugging](https://github.com/taka-oyama/msgpack-unity)

**JSON 변환으로 디버깅**

```csharp
// MessagePack → JSON 변환
var json = MessagePackSerializer.ConvertToJson(msgpackBytes);
Debug.Log(json);  // 가독한 JSON 출력
```

**직렬화 데이터 검사**

```csharp
public static void InspectMessagePack(byte[] data)
{
    var json = MessagePackSerializer.ConvertToJson(data);
    Debug.Log($"Size: {data.Length} bytes");
    Debug.Log($"JSON: {json}");
}
```

---

# 🎮 19. 실전: 인벤토리 시스템

```csharp
using MessagePack;
using System.Collections.Generic;
using UnityEngine;

[MessagePackObject]
public class Inventory
{
    [Key(0)] public int MaxSlots { get; set; } = 50;
    [Key(1)] public List<InventoryItem> Items { get; set; } = new();
    
    public bool AddItem(InventoryItem item)
    {
        if (Items.Count >= MaxSlots) return false;
        Items.Add(item);
        return true;
    }
    
    public bool RemoveItem(int itemId)
    {
        var item = Items.Find(x => x.ItemId == itemId);
        if (item != null)
        {
            Items.Remove(item);
            return true;
        }
        return false;
    }
    
    public byte[] Serialize()
    {
        return MessagePackSerializer.Serialize(this);
    }
    
    public static Inventory Deserialize(byte[] data)
    {
        return MessagePackSerializer.Deserialize<Inventory>(data);
    }
}

[MessagePackObject]
public class InventoryItem
{
    [Key(0)] public int ItemId { get; set; }
    [Key(1)] public string ItemName { get; set; }
    [Key(2)] public int Quantity { get; set; }
    [Key(3)] public ItemRarity Rarity { get; set; }
    [Key(4)] public ItemStats Stats { get; set; }
}

[MessagePackObject]
public class ItemStats
{
    [Key(0)] public int Attack { get; set; }
    [Key(1)] public int Defense { get; set; }
    [Key(2)] public float CritChance { get; set; }
}

public enum ItemRarity : byte
{
    Common = 0,
    Uncommon = 1,
    Rare = 2,
    Epic = 3,
    Legendary = 4
}
```

---

# 🏆 20. 실전: 랭킹 시스템

```csharp
using MessagePack;
using System.Collections.Generic;
using System.Linq;

[MessagePackObject]
public class LeaderboardEntry
{
    [Key(0)] public string PlayerId { get; set; }
    [Key(1)] public string PlayerName { get; set; }
    [Key(2)] public int Score { get; set; }
    [Key(3)] public int Rank { get; set; }
    [Key(4)] public long Timestamp { get; set; }
}

[MessagePackObject]
public class Leaderboard
{
    [Key(0)] public string LeaderboardId { get; set; }
    [Key(1)] public List<LeaderboardEntry> Entries { get; set; } = new();
    [Key(2)] public int MaxEntries { get; set; } = 100;
    
    public void AddOrUpdateEntry(LeaderboardEntry entry)
    {
        var existing = Entries.Find(x => x.PlayerId == entry.PlayerId);
        if (existing != null)
        {
            existing.Score = entry.Score;
            existing.Timestamp = entry.Timestamp;
        }
        else
        {
            Entries.Add(entry);
        }
        
        // 점수 정렬 및 순위 업데이트
        Entries = Entries.OrderByDescending(x => x.Score)
                         .Take(MaxEntries)
                         .ToList();
        
        for (int i = 0; i < Entries.Count; i++)
        {
            Entries[i].Rank = i + 1;
        }
    }
    
    public byte[] SerializeCompressed()
    {
        var options = MessagePackSerializerOptions.Standard
            .WithCompression(MessagePackCompression.Lz4BlockArray);
        return MessagePackSerializer.Serialize(this, options);
    }
}
```

---

# ⚡ 21. 고급 최적화

## 메모리 풀링

```csharp
public class PacketPool<T> where T : class, new()
{
    private Queue<T> _pool = new();
    private int _maxSize = 100;
    
    public T Rent()
    {
        return _pool.Count > 0 ? _pool.Dequeue() : new T();
    }
    
    public void Return(T obj)
    {
        if (_pool.Count < _maxSize)
        {
            _pool.Enqueue(obj);
        }
    }
}
```

## 배치 직렬화

```csharp
public class BatchSerializer
{
    public byte[] SerializeBatch<T>(List<T> items)
    {
        using var stream = new MemoryStream();
        foreach (var item in items)
        {
            var data = MessagePackSerializer.Serialize(item);
            stream.Write(BitConverter.GetBytes(data.Length), 0, 4);
            stream.Write(data, 0, data.Length);
        }
        return stream.ToArray();
    }
}
```

---

# 🔧 22. 트러블슈팅

## 자주 발생하는 오류

### 1. FormatterNotFoundException

**원인**: Resolver가 타입을 찾을 수 없음

**해결**:

```csharp
// MessagePackObject Attribute 추가
[MessagePackObject]
public class YourClass { }

// Resolver 등록
StaticCompositeResolver.Instance.Register(
    GeneratedResolver.Instance,
    StandardResolver.Instance
);
```

### 2. IL2CPP AOT 오류

**원인**: 동적 코드 생성 불가

**해결**:

- Source Generator 사용 (v3+)
- StaticCompositeResolver 사용
- Unity 2022.3.12f1 이상 사용

### 3. Key 순서 오류

**원인**: Key가 연속적이지 않거나 중복됨

**해결**:

```csharp
// ✅ 올바른 예제
[Key(0)] public int Id { get; set; }
[Key(1)] public string Name { get; set; }
[Key(2)] public float Score { get; set; }

// ❌ 잘못된 예제
[Key(0)] public int Id { get; set; }
[Key(2)] public string Name { get; set; }  // 1을 건너뛰
[Key(0)] public float Score { get; set; }  // 중복!
```

---

# 📚 참고 자료

## 공식 문서

- [MessagePack-CSharp GitHub](https://github.com/MessagePack-CSharp/MessagePack-CSharp)
- [MessagePack 공식 사이트](https://msgpack.org/)
- [MessagePack v3 Release](https://neuecc.medium.com/messagepack-for-c-v3-release-with-source-generator-support-893ed30d0e89)

## Unity 통합

- [Unity Integration Guide](https://blog.gametechtutorial.com/how-to-integrate-messagepack-c-for-unity-3ae622776d3d)
- [Unity OpenUPM](https://openupm.com/packages/com.github.messagepack-csharp/)

## 성능 분석

- [MessagePack v2 Performance](https://neuecc.medium.com/messagepack-for-c-v2-new-era-of-net-core-unity-i-o-pipelines-6950643c1053)
- [Unity Serialization Discussion](https://discussions.unity.com/t/json-bson-messagepack-protobuf-etc/846512)

## 네트워킹

- [Unity Multiplayer Guide](https://moldstud.com/articles/p-creating-custom-network-messages-in-unity-a-comprehensive-developers-guide)
- [Network Optimization](https://medium.com/my-games-company/unity-realtime-multiplayer-part-3-reliable-udp-protocol-94fbffe8c72c)

## 고급 기능

- [Custom Formatters](https://hals.app/blog/messagepack-csharp-formatters/)
- [LZ4 Compression](https://medium.com/@buganini/my-favorite-data-38af160829dc)

---

# 🎉 결론

**MessagePack-CSharp**는 Unity 게임 개발에서 필수적인 고성능 직렬화 라이브러리입니다.

**핵심 장점**

- ⚡ JSON 대비 10배 빠른 속도
- 📦 50% 이상 작은 바이너리 크기
- 🎮 Unity 전용 타입 완벽 지원
- 🚀 IL2CPP/AOT 완전 대응
- 🗜️ LZ4 압축 내장

**추천 사용 시나리오**

1. 네트워크 멀티플레이어 게임
2. 대용량 세이브/로드 시스템
3. 서버 API 통신
4. 애셋 번들 데이터

**시작하기**

```csharp
// 1. NuGet에서 MessagePack 설치
// 2. Unity 패키지 설치
// 3. MessagePackObject Attribute 추가
[MessagePackObject]
public class YourData
{
    [Key(0)] public int Value { get; set; }
}

// 4. 직렬화/역직렬화
var bytes = MessagePackSerializer.Serialize(data);
var restored = MessagePackSerializer.Deserialize<YourData>(bytes);
```

**행복한 코딩 되세요!** 🚀

```

```