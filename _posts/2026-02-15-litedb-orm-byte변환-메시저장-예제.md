---
layout: post
title: 260215 LiteDB ORM byte변환 메시저장 예제
comments: true
tags:
- unity
- performance
- litedb
- repository-pattern
- debugging
---
# 🗂️ 260215 LiteDB ORM byte[] 변환 메시 저장 예제

> 💡 이전 문서 [LiteDB 성능 체크 - Unity 환경](./260215_1530_LiteDB_성능체크_Unity_환경.md)에서 `float[]` → `byte[]` 변환 저장을 권장하였다. 본 문서에서는 LiteDB의 ORM 매핑 기능을 활용하여 **비즈니스 코드에서는 `float[]`로 사용하되, DB에는 `byte[]`로 저장/복원**되는 구체적인 예제를 다룬다.

---

## 🎯 1. 핵심 개념: 왜 byte[]로 변환하는가?

`float[]`을 LiteDB에 그대로 저장하면 BSON `BsonArray`로 직렬화된다. 이때 **각 float 요소마다** 타입 바이트 + 값 8bytes(double 변환)로 저장되어 엄청난 오버헤드가 발생한다.

```
float[] 직접 저장 (BsonArray):
  요소 1개 = 타입(1B) + double값(8B) = 9 bytes
  float 1,000개 → 9,000 bytes (원본 4,000 bytes의 2.25배)

byte[] 변환 저장 (BsonBinary):
  헤더(5B) + 원본 바이트 그대로
  float 1,000개 → 4,005 bytes (원본과 거의 동일)
```

| 방식 | 1,000 floats | 10,000 floats | 오버헤드 |
|------|-------------|---------------|---------|
| **float[] 직접** | 9 KB | 90 KB | **~2.25x** |
| **byte[] 변환** | 4 KB | 40 KB | **~1.0x** |

> 💡 참고: [LiteDB BSON 문서 모델](https://deepwiki.com/litedb-org/LiteDB/2.5-bson-document-model)

---

## 📌 2. 방법 A: `BsonIgnore` + 수동 변환 프로퍼티

가장 직관적인 방법. ORM 클래스에 **실제 저장용 `byte[]` 프로퍼티**와 **비즈니스용 `float[]` 프로퍼티**를 분리한다.

### 🔹 2.1 ORM 클래스 정의

```csharp
using LiteDB;
using System;
using System.Runtime.InteropServices;

public class MyMesh
{
    // ── DB 저장 필드 ──
    [BsonId(true)]
    public long Id { get; set; }

    public string ModelItemUid { get; set; }  // char[64]

    // DB에 byte[]로 저장되는 실제 컬럼
    [BsonField("vertex")]
    public byte[] VertexRaw { get; set; }

    [BsonField("index")]
    public byte[] IndexRaw { get; set; }

    [BsonField("triIndex")]
    public byte[] TriangleIndexRaw { get; set; }

    // ── 비즈니스 코드용 (DB 저장 제외) ──
    [BsonIgnore]
    public float[] VertexList
    {
        get => BytesToFloats(VertexRaw);
        set => VertexRaw = FloatsToBytes(value);
    }

    [BsonIgnore]
    public int[] IndexList
    {
        get => BytesToInts(IndexRaw);
        set => IndexRaw = IntsToBytes(value);
    }

    [BsonIgnore]
    public int[] TriangleIndexList
    {
        get => BytesToInts(TriangleIndexRaw);
        set => TriangleIndexRaw = IntsToBytes(value);
    }

    // ── 변환 유틸 (static) ──
    static byte[] FloatsToBytes(float[] arr)
    {
        if (arr == null || arr.Length == 0) return Array.Empty<byte>();
        var bytes = new byte[arr.Length * sizeof(float)];
        Buffer.BlockCopy(arr, 0, bytes, 0, bytes.Length);
        return bytes;
    }

    static float[] BytesToFloats(byte[] bytes)
    {
        if (bytes == null || bytes.Length == 0) return Array.Empty<float>();
        var arr = new float[bytes.Length / sizeof(float)];
        Buffer.BlockCopy(bytes, 0, arr, 0, bytes.Length);
        return arr;
    }

    static byte[] IntsToBytes(int[] arr)
    {
        if (arr == null || arr.Length == 0) return Array.Empty<byte>();
        var bytes = new byte[arr.Length * sizeof(int)];
        Buffer.BlockCopy(arr, 0, bytes, 0, bytes.Length);
        return bytes;
    }

    static int[] BytesToInts(byte[] bytes)
    {
        if (bytes == null || bytes.Length == 0) return Array.Empty<int>();
        var arr = new int[bytes.Length / sizeof(int)];
        Buffer.BlockCopy(bytes, 0, arr, 0, bytes.Length);
        return arr;
    }
}
```

### 🧪 2.2 CRUD 사용 예제

```csharp
using LiteDB;
using System.Collections.Generic;
using System.Linq;

public class MeshRepository
{
    private readonly LiteDatabase _db;
    private readonly ILiteCollection<MyMesh> _col;

    public MeshRepository(string dbPath)
    {
        // Direct 모드 (Unity 권장)
        _db = new LiteDatabase($"Filename={dbPath};Connection=direct");
        _col = _db.GetCollection<MyMesh>("meshes");

        // modelItemUid 인덱스 생성
        _col.EnsureIndex(x => x.ModelItemUid);
    }

    // ── INSERT ──
    public void Insert(MyMesh mesh)
    {
        _col.Insert(mesh);
    }

    // ── BULK INSERT ──
    public void InsertBulk(List<MyMesh> meshes)
    {
        // 반드시 List<T>로 전달 (IEnumerable 대비 13배 빠름)
        _col.InsertBulk(meshes);
    }

    // ── READ (단건, 인덱스 쿼리) ──
    public MyMesh FindByUid(string uid)
    {
        return _col.FindOne(x => x.ModelItemUid == uid);
    }

    // ── READ (전체) ──
    public IEnumerable<MyMesh> FindAll()
    {
        // ToList() 하지 않고 IEnumerable로 반환 → 메모리 효율
        return _col.FindAll();
    }

    // ── UPDATE ──
    public bool Update(MyMesh mesh)
    {
        return _col.Update(mesh);
    }

    // ── DELETE ──
    public bool Delete(long id)
    {
        return _col.Delete(id);
    }

    public void Dispose() => _db?.Dispose();
}
```

### 🔹 2.3 Unity에서 사용

```csharp
using UnityEngine;
using System.Collections.Generic;

public class MeshStorageExample : MonoBehaviour
{
    private MeshRepository _repo;

    void Start()
    {
        // Application.persistentDataPath = Unity 앱 데이터 경로
        string dbPath = System.IO.Path.Combine(
            Application.persistentDataPath, "meshdata.db");
        _repo = new MeshRepository(dbPath);
    }

    // ── 저장 예시 ──
    void SaveMesh(Mesh unityMesh, string modelItemUid)
    {
        var myMesh = new MyMesh
        {
            ModelItemUid = modelItemUid,
            // float[], int[] 그대로 할당 → 내부에서 byte[] 자동 변환
            VertexList = ConvertVector3ToFloats(unityMesh.vertices),
            IndexList = unityMesh.triangles,
            TriangleIndexList = unityMesh.triangles
        };

        _repo.Insert(myMesh);
        Debug.Log($"저장 완료: {modelItemUid}");
    }

    // ── 조회 예시 ──
    void LoadMesh(string modelItemUid)
    {
        var myMesh = _repo.FindByUid(modelItemUid);
        if (myMesh == null) return;

        // byte[]에서 float[], int[]로 자동 복원
        float[] vertices = myMesh.VertexList;
        int[] indices = myMesh.IndexList;

        Debug.Log($"정점 수: {vertices.Length / 3}, 인덱스 수: {indices.Length}");
    }

    // ── 대량 저장 예시 ──
    void BulkSave(List<MyMesh> meshes)
    {
        _repo.InsertBulk(meshes);
        Debug.Log($"Bulk 저장 완료: {meshes.Count}건");
    }

    // ── Vector3[] → float[] 변환 유틸 ──
    float[] ConvertVector3ToFloats(Vector3[] vectors)
    {
        var floats = new float[vectors.Length * 3];
        for (int i = 0; i < vectors.Length; i++)
        {
            floats[i * 3]     = vectors[i].x;
            floats[i * 3 + 1] = vectors[i].y;
            floats[i * 3 + 2] = vectors[i].z;
        }
        return floats;
    }

    void OnDestroy() => _repo?.Dispose();
}
```

> 💡 **장점**: 코드가 직관적이고, 비즈니스 로직에서 `float[]`/`int[]`를 자연스럽게 사용
> ⚠️ **단점**: getter/setter 호출 시마다 변환 발생 → 빈번한 접근 시 성능 주의

---

## 📌 3. 방법 B: `RegisterType`으로 전역 커스텀 직렬화

`BsonMapper.Global.RegisterType<T>`를 사용하면, ORM 클래스에서 **`float[]`를 그대로 프로퍼티로 사용**하면서도 DB 저장 시 자동으로 `byte[]`로 변환된다.

### 🔹 3.1 전역 매퍼 등록

```csharp
using LiteDB;
using System;

/// <summary>
/// 앱 시작 시 한 번만 호출
/// </summary>
public static class LiteDbMapperConfig
{
    public static void Initialize()
    {
        // float[] → BsonBinary 자동 변환 등록
        BsonMapper.Global.RegisterType<float[]>(
            serialize: floats =>
            {
                if (floats == null || floats.Length == 0)
                    return new BsonValue(Array.Empty<byte>());

                var bytes = new byte[floats.Length * sizeof(float)];
                Buffer.BlockCopy(floats, 0, bytes, 0, bytes.Length);
                return new BsonValue(bytes);
            },
            deserialize: bson =>
            {
                var bytes = bson.AsBinary;
                if (bytes == null || bytes.Length == 0)
                    return Array.Empty<float>();

                var floats = new float[bytes.Length / sizeof(float)];
                Buffer.BlockCopy(bytes, 0, floats, 0, bytes.Length);
                return floats;
            }
        );

        // int[] → BsonBinary 자동 변환 등록
        BsonMapper.Global.RegisterType<int[]>(
            serialize: ints =>
            {
                if (ints == null || ints.Length == 0)
                    return new BsonValue(Array.Empty<byte>());

                var bytes = new byte[ints.Length * sizeof(int)];
                Buffer.BlockCopy(ints, 0, bytes, 0, bytes.Length);
                return new BsonValue(bytes);
            },
            deserialize: bson =>
            {
                var bytes = bson.AsBinary;
                if (bytes == null || bytes.Length == 0)
                    return Array.Empty<int>();

                var ints = new int[bytes.Length / sizeof(int)];
                Buffer.BlockCopy(bytes, 0, ints, 0, bytes.Length);
                return ints;
            }
        );
    }
}
```

### 🔹 3.2 ORM 클래스 (깔끔한 버전)

```csharp
using LiteDB;

public class MyMesh
{
    [BsonId(true)]
    public long Id { get; set; }

    public string ModelItemUid { get; set; }

    // float[]로 선언하지만, RegisterType에 의해 byte[]로 저장됨
    public float[] VertexList { get; set; }

    public int[] IndexList { get; set; }

    public int[] TriangleIndexList { get; set; }
}

public class MyProp
{
    [BsonId(true)]
    public long Id { get; set; }

    public string ModelItemUid { get; set; }

    public string Name { get; set; }

    public string Value { get; set; }
}
```

### 🔹 3.3 Unity 초기화 및 사용

```csharp
using UnityEngine;

public class AppBootstrap : MonoBehaviour
{
    void Awake()
    {
        // 앱 시작 시 한 번만 호출
        LiteDbMapperConfig.Initialize();
    }
}
```

```csharp
using LiteDB;
using UnityEngine;

public class MeshService : MonoBehaviour
{
    private LiteDatabase _db;
    private ILiteCollection<MyMesh> _meshCol;
    private ILiteCollection<MyProp> _propCol;

    void Start()
    {
        string dbPath = System.IO.Path.Combine(
            Application.persistentDataPath, "project.db");

        _db = new LiteDatabase($"Filename={dbPath};Connection=direct");
        _meshCol = _db.GetCollection<MyMesh>("meshes");
        _propCol = _db.GetCollection<MyProp>("props");

        _meshCol.EnsureIndex(x => x.ModelItemUid);
        _propCol.EnsureIndex(x => x.ModelItemUid);
        _propCol.EnsureIndex(x => x.Name);
    }

    // ── 저장: float[]를 그대로 넣으면 byte[]로 자동 변환 ──
    public void SaveMesh(string uid, float[] vertices, int[] indices, int[] triIndices)
    {
        var mesh = new MyMesh
        {
            ModelItemUid = uid,
            VertexList = vertices,           // ← float[] 그대로
            IndexList = indices,             // ← int[] 그대로
            TriangleIndexList = triIndices   // ← int[] 그대로
        };
        _meshCol.Insert(mesh);
    }

    // ── 조회: byte[]에서 float[]로 자동 복원 ──
    public MyMesh LoadMesh(string uid)
    {
        var mesh = _meshCol.FindOne(x => x.ModelItemUid == uid);
        // mesh.VertexList → float[] 자동 복원
        // mesh.IndexList  → int[] 자동 복원
        return mesh;
    }

    // ── Bulk 저장 ──
    public void SaveMeshBulk(System.Collections.Generic.List<MyMesh> meshes)
    {
        _meshCol.InsertBulk(meshes);
    }

    // ── 스트리밍 전체 조회 (메모리 효율) ──
    public void ProcessAllMeshes(System.Action<MyMesh> processor)
    {
        foreach (var mesh in _meshCol.FindAll())
        {
            processor(mesh);
            // 처리 후 GC 가능
        }
    }

    void OnDestroy() => _db?.Dispose();
}
```

> 💡 **장점**: ORM 클래스가 깔끔, 비즈니스 코드에서 변환 코드 불필요
> 💡 **단점**: `int[]`의 전역 RegisterType이 다른 컬렉션에도 영향 → 의도치 않은 부작용 가능

---

## ⚖️ 4. 방법 A vs B 비교

| 항목 | 방법 A (BsonIgnore + 수동) | 방법 B (RegisterType 전역) |
|------|--------------------------|--------------------------|
| **ORM 클래스** | 프로퍼티 쌍 필요 (Raw + 비즈니스) | 깔끔한 단일 프로퍼티 |
| **변환 코드 위치** | 클래스 내부 (명시적) | 전역 매퍼 (암묵적) |
| **영향 범위** | 해당 클래스만 | **모든 float[], int[] 프로퍼티** |
| **부작용 위험** | 없음 | 다른 컬렉션의 int[] 배열도 byte[]로 저장됨 |
| **IL2CPP 호환** | Buffer.BlockCopy → 안전 | Buffer.BlockCopy → 안전 |
| **디버깅** | Raw 필드 직접 확인 가능 | LiteDB Studio에서 binary blob으로 보임 |
| **권장 상황** | 메시 전용 클래스가 있을 때 | 프로젝트 전체에서 배열을 byte로 저장할 때 |

> 💡 **권장**: 대부분의 경우 **방법 A**가 안전하다. 방법 B는 전역 영향이므로 `int[]`가 다른 곳에서도 사용된다면 의도치 않게 byte[]로 저장될 수 있다.

---

## 🔗 5. IL2CPP 호환성 참고

### ⚖️ Buffer.BlockCopy vs MemoryMarshal

| API | IL2CPP | Mono | Burst | 비고 |
|-----|--------|------|-------|------|
| **Buffer.BlockCopy** | **안전** | 안전 | N/A | .NET 1.0부터 존재, 모든 환경 호환 |
| **MemoryMarshal.Cast** | **주의** | 안전 | 미지원 | IL2CPP에서 Span/ReadOnlySpan 관련 이슈 보고 |
| **BitConverter.GetBytes** | 안전 | 안전 | N/A | 단건 변환만 가능, 배열에 비효율 |
| **unsafe fixed** | 안전 | 안전 | 안전 | 가장 빠르나 unsafe 컨텍스트 필요 |

> 💡 **결론**: Unity IL2CPP 환경에서는 **`Buffer.BlockCopy`가 가장 안전하고 효율적**인 선택이다. `MemoryMarshal`은 IL2CPP에서 `ReadOnlySpan` 관련 이슈가 보고된 바 있다.

> 💡 참고: [Unity IL2CPP ReadOnlySpan 이슈](https://discussions.unity.com/t/il2cpp-doesnt-support-readonlymemory-t-or-readonlyspan-byte-method-parameters/914544), [MemoryMarshal.Cast Unity 지원](https://discussions.unity.com/t/support-memorymarshal-cast/939492)

---

## 📌 6. 전체 흐름 요약

```
[Unity 비즈니스 코드]
    mesh.VertexList = new float[] { 1.0f, 2.0f, 3.0f, ... };
            │
            ▼
    ┌─────────────────────────────────┐
    │  방법 A: setter에서 변환         │
    │  VertexRaw = FloatsToBytes(v)   │
    │                                 │
    │  방법 B: RegisterType가 가로챔   │
    │  serialize(floats) → bytes      │
    └─────────────────────────────────┘
            │
            ▼
    [LiteDB 내부]
    BsonValue(byte[]) → BSON Binary 타입으로 디스크 저장
    헤더 5bytes + 원본 바이트 그대로 (오버헤드 거의 없음)
            │
            ▼
    [조회 시]
    BSON Binary → byte[] → Buffer.BlockCopy → float[]
    mesh.VertexList 접근 시 자동 복원
```

---

## 🔗 참고 자료

- [LiteDB Object Mapping 공식 문서](https://www.litedb.org/docs/object-mapping/)
- [LiteDB BsonDocument 문서](https://www.litedb.org/docs/bsondocument/)
- [LiteDB RegisterType v5 사용 예시 이슈 #1599](https://github.com/mbdavid/LiteDB/issues/1599)
- [LiteDB BsonMapper 소스코드](https://github.com/litedb-org/LiteDB/blob/master/LiteDB/Client/Mapper/BsonMapper.cs)
- [LiteDB BSON 문서 모델 (DeepWiki)](https://deepwiki.com/litedb-org/LiteDB/2.5-bson-document-model)
- [LiteDB Object Mapping (DeepWiki)](https://deepwiki.com/litedb-org/LiteDB/3.2-object-mapping)
- [Unity IL2CPP ReadOnlySpan 이슈](https://discussions.unity.com/t/il2cpp-doesnt-support-readonlymemory-t-or-readonlyspan-byte-method-parameters/914544)
- [Unity MemoryMarshal.Cast 지원 토론](https://discussions.unity.com/t/support-memorymarshal-cast/939492)
- [C# float↔byte 변환 방법](https://markheath.net/post/how-to-convert-byte-to-short-or-float)
