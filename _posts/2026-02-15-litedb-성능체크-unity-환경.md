---
layout: post
title: 260215 LiteDB 성능체크 Unity 환경
comments: true
tags:
- unity
- performance
- optimization
- litedb
- windows
- api
- research
- analysis
---
# ⚡ 260215 LiteDB 성능 체크 - Unity 환경

## 🧭 1. LiteDB 개요

LiteDB는 .NET 기반의 임베디드 NoSQL 문서 데이터베이스로, 단일 파일(`*.db`)에 데이터를 저장한다. MongoDB와 유사한 API를 제공하며, 서버 설치 없이 DLL 하나로 동작한다.

| 항목 | 내용 |
|------|------|
| **최신 버전** | v5.0.x |
| **저장 형식** | BSON (Binary JSON) |
| **문서 크기 제한** | 최대 16MB (BSON 직렬화 후) |
| **인덱스 필드 제한** | 1024 bytes (BSON 직렬화 후) |
| **인덱스 구현** | Skip List (O(log n), 최대 32 레벨) |
| **트랜잭션** | 지원 (Write Lock 기반) |
| **라이선스** | MIT |

> 💡 참고: [LiteDB 공식 문서](https://www.litedb.org/docs/)

---

## 📌 2. 대상 환경

| 항목 | 스펙 |
|------|------|
| **OS** | Windows 11 |
| **플랫폼** | Unity App |
| **스크립트 백엔드** | IL2CPP 또는 .NET (Mono) |
| **접근 방식** | 단일 프로세스, 단일/멀티 스레드 |

### ⚠️ 2.1 IL2CPP 호환성 주의사항

IL2CPP 환경에서 LiteDB를 사용할 때 다음 이슈에 주의해야 한다:

| 이슈 | 설명 | 영향도 |
|------|------|--------|
| **Reflection.Emit 미지원** | IL2CPP는 AOT 컴파일이므로 동적 코드 생성 불가 | **Critical** - LiteDB v4 이하에서 심각한 문제 |
| **Named Mutex 미지원** | IL2CPP 일부 플랫폼에서 Shared 모드 사용 불가 | **High** - Direct 모드 사용 필수 |
| **ARM 메모리 정렬** | Android ARMv7에서 Release 빌드 시 SIGBUS 크래시 | **Medium** - Windows에서는 해당 없음 |
| **LINQ Expression 제한** | 일부 LINQ 표현식이 AOT에서 작동하지 않을 수 있음 | **Medium** |

> 💡 참고: [LiteDB IL2CPP Mutex 이슈](https://github.com/litedb-org/LiteDB/issues/1935), [IL2CPP 크래시 이슈](https://github.com/mbdavid/LiteDB/issues/1759)

### 🗄️ 2.2 IL2CPP 대안: UltraLiteDB

LiteDB 4.0 기반의 경량 포크로, IL2CPP AOT 환경에 최적화되어 있다:

- LINQ, 동적 코드 생성 제거
- 단일 스레드 전용 (Unity에 적합)
- 단순 Key-Value 저장소 수준으로 기능 축소
- 쿼리/인덱스가 최상위 필드에만 제한

> 💡 참고: [UltraLiteDB GitHub](https://github.com/rejemy/UltraLiteDB)

---

## 🏗️ 3. 데이터 스키마 분석

### 🎮 3.1 MyMesh 테이블

```
MyMesh
├── id             : long (auto increment)     ~8 bytes
├── modelItemUid   : char[64]                  ~64 bytes
├── vertexList     : float[]                   가변
├── indexList      : int[]                     가변
└── triangleIndexList : int[]                  가변
```

**문서 크기 추정 (메시 복잡도별):**

| 메시 규모 | 정점 수 | vertexList | indexList | triIndexList | 문서 총 크기 |
|-----------|---------|------------|-----------|--------------|-------------|
| 소형 | 100개 | 1.2 KB | 0.4 KB | 1.2 KB | **~3 KB** |
| 중형 | 1,000개 | 12 KB | 4 KB | 12 KB | **~28 KB** |
| 대형 | 10,000개 | 120 KB | 40 KB | 120 KB | **~280 KB** |
| 초대형 | 50,000개 | 600 KB | 200 KB | 600 KB | **~1.4 MB** |

> 💡 **산출 근거**: vertexList는 x,y,z 3개 float(12bytes) × 정점 수, indexList는 int(4bytes) × 정점 수, triangleIndexList는 int(4bytes) × (정점 수 × 3, 삼각형 인덱스)

**핵심 제약**:
- BSON 직렬화 시 배열 메타데이터(타입, 길이) 오버헤드 추가 → 실제 크기는 원본 대비 **10~20% 증가**
- 문서 크기 16MB 제한이므로 초대형 메시는 분할 저장 필요
- `float[]`, `int[]`는 BSON `BsonArray`로 직렬화되며, 개별 요소마다 타입 바이트가 추가됨

### 🔹 3.2 MyProp 테이블

```
MyProp
├── id             : long (auto increment)     ~8 bytes
├── modelItemUid   : char[64]                  ~64 bytes
├── name           : str[64]                   ~64 bytes
└── value          : str[64]                   ~64 bytes
```

**문서 크기 추정: ~200 bytes** (BSON 메타데이터 포함)

### 🔹 3.3 데이터 규모 요약

| 테이블 | 행 수 | 문서 크기(평균) | 총 데이터 크기(추정) |
|--------|-------|----------------|---------------------|
| **MyMesh** | 500,000 | 28 KB (중형 기준) | **~13 GB** |
| **MyProp** | 500,000 | 200 bytes | **~95 MB** |

> ⚠️ **경고**: MyMesh 테이블은 메시 복잡도에 따라 DB 파일 크기가 수 GB~수십 GB에 달할 수 있다. LiteDB는 이 규모에서 성능 저하가 상당히 심각할 수 있다.

---

## ⚡ 4. 성능 벤치마크 분석

### ⚡ 4.1 기준 벤치마크 (LiteDB v3, 5,000건 기준)

LiteDB 공식 벤치마크 결과:

| 작업 | 속도 (records/sec) | 비고 |
|------|-------------------|------|
| **Insert** | 1,000 | 건별 삽입 |
| **Bulk Insert** | 21,184 | InsertBulk 사용 |
| **Bulk Insert (Exclusive)** | 22,775 | Journal 비활성화 |
| **Update** | 1,361 | |
| **Query** | 24,467 | 인덱스 사용 시 |
| **Delete** | 31,722 | |

> 💡 참고: [LiteDB-Perf GitHub](https://github.com/mbdavid/LiteDB-Perf)

### ⚡ 4.2 대상 데이터에 대한 예상 성능

#### ▫️ MyProp 테이블 (경량 문서, ~200 bytes)

| 작업 | 예상 속도 | 500,000건 소요시간 | 근거 |
|------|----------|-------------------|------|
| **단건 Insert** | ~800~1,000 rec/s | **~8~10분** | 공식 벤치마크 기준, 문서 크기 작아 유사 |
| **Bulk Insert** | ~15,000~20,000 rec/s | **~25~33초** | InsertBulk + List 전달 시 |
| **단건 Read (인덱스)** | <0.1ms/건 | **즉시** | Skip List O(log n), ~19 step |
| **Bulk Read (전체)** | ~20,000~25,000 rec/s | **~20~25초** | 순차 풀스캔 |
| **Where Query (인덱스)** | <1ms/건 | **즉시** | 인덱스 있을 경우 |
| **Where Query (풀스캔)** | ~10,000~15,000 rec/s | **~33~50초** | 인덱스 없을 경우, 문서 역직렬화 필요 |

#### 🎮 MyMesh 테이블 (대용량 문서, ~28 KB 중형 기준)

| 작업 | 예상 속도 | 500,000건 소요시간 | 근거 |
|------|----------|-------------------|------|
| **단건 Insert** | ~100~300 rec/s | **~28~83분** | BSON 직렬화 + 디스크 I/O 부하 큼 |
| **Bulk Insert** | ~2,000~5,000 rec/s | **~1.5~4분** | 대용량 문서 배열 직렬화 오버헤드 |
| **단건 Read (인덱스)** | ~1~5ms/건 | **~1~5ms** | 인덱스 검색 빠르나 역직렬화 비용 |
| **Bulk Read (전체)** | ~1,000~3,000 rec/s | **~2.5~8분** | 순차 I/O + 역직렬화 병목 |
| **Where Query (인덱스)** | ~1~5ms/건 | **~1~5ms** | 인덱스 조회 자체는 빠름 |
| **Where Query (풀스캔)** | ~500~1,500 rec/s | **~5.5~16분** | 모든 문서 역직렬화 필요 |

> 💡 **산출 근거**: 공식 벤치마크(5,000건, 소형 문서)를 기반으로, 문서 크기 증가에 따른 BSON 직렬화/역직렬화 비용과 디스크 I/O 증가를 반영한 추정치이다. 실제 성능은 디스크 속도(SSD vs HDD), 메모리, 인덱스 구성에 따라 크게 달라진다.

### 🔹 4.3 인덱스 쿼리 상세

LiteDB의 인덱스는 Skip List 기반이다:

```
1,000,000건 기준 → 약 20 step으로 문서 탐색 (O(log n))
  500,000건 기준 → 약 19 step으로 문서 탐색
```

| 쿼리 유형 | 인덱스 유무 | 성능 | 비고 |
|-----------|-----------|------|------|
| `FindById(id)` | _id 자동 인덱스 | **<0.1ms** | 기본 키 조회 |
| `Find(x => x.modelItemUid == "...")` | 인덱스 있음 | **<1ms** | 수동 인덱스 생성 필요 |
| `Find(x => x.modelItemUid == "...")` | 인덱스 없음 | **수 초~수십 초** | 전체 풀스캔 |
| `Find(x => x.name == "..." && x.value == "...")` | 복합 조건 | **주의** | **인덱스 1개만 사용**, 나머지는 풀스캔 |

> 💡 **중요**: LiteDB는 쿼리당 **인덱스 1개만** 사용한다. 복합 조건 쿼리에서도 하나의 인덱스만 활용되고, 나머지 조건은 필터링(풀스캔)으로 처리된다.

> 💡 참고: [LiteDB 인덱스 문서](https://www.litedb.org/docs/indexes/), [쿼리 인덱스 제한 이슈](https://github.com/litedb-org/LiteDB/issues/2305)

---

## ⚡ 5. 연결 모드별 성능

| 모드 | 설명 | 성능 | 권장 환경 |
|------|------|------|----------|
| **Direct** | 단일 프로세스 전용, 파일 독점 | **최고** (캐시 유지, 헤더 체크 생략) | Unity 앱 (권장) |
| **Shared** | 멀티 프로세스 공유 | 느림 (매 작업마다 파일 open/lock/close) | 다중 프로세스 접근 시 |

```csharp
// Unity 환경 권장 연결 설정
var db = new LiteDatabase("Filename=mydata.db;Connection=direct");
```

> 💡 참고: [LiteDB Connection String 문서](https://www.litedb.org/docs/connection-string/)

---

## 🚀 6. 최적화 권장사항

### ⚠️ 6.1 InsertBulk 사용 시 주의점

```csharp
// 나쁜 예 - IEnumerable (Lazy) 전달 → 57.7초/10,000건
col.InsertBulk(GetDocumentsLazy());

// 좋은 예 - List (Materialized) 전달 → 4.3초/10,000건
col.InsertBulk(GetDocuments().ToList());
```

> 💡 **13배 성능 차이** 발생. InsertBulk에는 반드시 미리 평가된(materialized) 컬렉션을 전달해야 한다.

> 💡 참고: [InsertBulk IEnumerable vs List 이슈](https://github.com/mbdavid/LiteDB/issues/961)

### 🔹 6.2 인덱스 전략

```csharp
// modelItemUid에 인덱스 생성 (필수)
col.EnsureIndex(x => x.ModelItemUid);

// MyProp의 name 필드에 인덱스 생성
propCol.EnsureIndex(x => x.Name);
```

- `modelItemUid`는 양쪽 테이블 모두에서 **반드시 인덱스 생성** 필요
- 인덱스는 Insert 성능을 다소 저하시키지만, Query 성능을 극적으로 향상시킴
- 인덱스 값은 BSON 1024 bytes 이내여야 함 (char[64]는 충분)

### 🎮 6.3 MyMesh 대용량 배열 처리

MyMesh의 `float[]`, `int[]` 배열은 BSON 직렬화 시 상당한 오버헤드가 발생한다:

| 전략 | 설명 | 장점 | 단점 |
|------|------|------|------|
| **직접 저장** | float[]를 그대로 BsonArray로 저장 | 간단한 코드 | 직렬화 오버헤드 큼 |
| **byte[] 변환** | float[]를 byte[]로 변환 후 BsonBinary로 저장 | BSON 오버헤드 감소 | 변환 코드 필요 |
| **FileStorage 활용** | 메시 데이터를 FileStorage에 스트림으로 저장 | 16MB 제한 회피 | API 복잡도 증가 |
| **외부 파일 분리** | 메시 바이너리를 별도 파일로 저장, DB에는 경로만 기록 | 최고 성능 | 파일 관리 필요 |

> 💡 **권장**: `byte[]` 변환 방식이 성능과 편의성의 균형이 가장 좋다.

```csharp
// float[] → byte[] 변환으로 BSON 오버헤드 감소
public byte[] VertexData { get; set; }

// 변환 유틸
byte[] FloatsToBytes(float[] arr) =>
    MemoryMarshal.AsBytes(arr.AsSpan()).ToArray();
float[] BytesToFloats(byte[] bytes) =>
    MemoryMarshal.Cast<byte, float>(bytes.AsSpan()).ToArray();
```

---

## ⚖️ 7. LiteDB vs SQLite 비교

| 항목 | LiteDB | SQLite |
|------|--------|--------|
| **타입** | NoSQL (문서형) | RDBMS (관계형) |
| **API** | C# 네이티브 LINQ | SQL 쿼리 |
| **Bulk Insert** | 느림 (SQLite 대비) | 빠름 |
| **인덱스** | 단일 인덱스 쿼리만 | 복합 인덱스 지원 |
| **파일 크기** | 큼 (BSON 오버헤드) | 작음 |
| **대용량 데이터** | 성능 저하 심함 | 상대적으로 안정적 |
| **Unity IL2CPP** | 이슈 있음 | 안정적 (네이티브) |
| **메모리 사용** | 높음 (30MB+ 쿼리 시) | 낮음 |
| **설치 편의성** | DLL 하나 | 네이티브 바인딩 필요 |

> 💡 참고: [LiteDB vs SQLite 비교](https://tomaskohl.com/code/2020-04-07/trying-out-litedb/), [SoloDB vs LiteDB](https://unconcurrent.com/articles/SoloDBvsLiteDB.html)

---

## 🔍 8. 종합 평가

### 🔹 8.1 시나리오별 판정

| 시나리오 | 적합도 | 판정 |
|----------|--------|------|
| MyProp 50만건 CRUD | **적합** | 문서 크기 작아 성능 양호, 인덱스 활용 시 쿼리 빠름 |
| MyMesh 50만건 (소형 메시) | **조건부 적합** | bulk 작업 수 분 소요, 인덱스 쿼리는 밀리초 수준 |
| MyMesh 50만건 (중형 메시) | **부적합 위험** | DB 파일 13GB+, bulk 작업 수십 분, 풀스캔 쿼리 수십 분 |
| MyMesh 50만건 (대형 메시) | **부적합** | DB 파일 100GB+, LiteDB 설계 범위 초과 |
| IL2CPP 환경 | **주의 필요** | v5는 대부분 호환되나 일부 LINQ/Reflection 이슈 존재 |

### 🔹 8.2 최종 권장사항

1. **MyProp 테이블**: LiteDB 사용 적합. `modelItemUid`와 `name`에 인덱스를 생성하면 50만건에서도 밀리초 수준의 쿼리 응답 가능.

2. **MyMesh 테이블**: 메시 데이터의 크기에 따라 크게 달라짐.
   - 소형 메시(~3KB): LiteDB로 가능하나 bulk 작업에 수 분 소요
   - 중형 이상: **SQLite 또는 별도 바이너리 파일 저장을 강력히 권장**. LiteDB의 BSON 직렬화 오버헤드와 단일 인덱스 제한이 병목이 됨.

3. **IL2CPP 환경**:
   - LiteDB v5 + Direct 모드로 Windows에서는 대체로 안정적
   - 문제 발생 시 UltraLiteDB를 대안으로 고려
   - `link.xml`에 LiteDB 어셈블리 보존 설정 추가 권장

4. **연결 모드**: Unity 앱 특성상 반드시 **Direct 모드** 사용 (Shared 모드 대비 수 배 빠름)

5. **InsertBulk 최적화**: `IEnumerable` 대신 `List<T>`로 미리 평가하여 전달 (13배 성능 차이)

---

## 🔗 참고 자료

- [LiteDB 공식 문서](https://www.litedb.org/docs/)
- [LiteDB GitHub](https://github.com/litedb-org/LiteDB)
- [LiteDB-Perf 벤치마크](https://github.com/mbdavid/LiteDB-Perf)
- [LiteDB 인덱스 문서](https://www.litedb.org/docs/indexes/)
- [LiteDB Connection String](https://www.litedb.org/docs/connection-string/)
- [LiteDB 데이터 구조](https://www.litedb.org/docs/data-structure/)
- [LiteDB FileStorage](https://www.litedb.org/docs/filestorage/)
- [LiteDB v4→v5 쿼리 성능 이슈](https://github.com/litedb-org/LiteDB/issues/2023)
- [LiteDB v5.0.18 성능 저하 버그](https://github.com/litedb-org/LiteDB/issues/2451)
- [LiteDB 복합 쿼리 인덱스 제한](https://github.com/litedb-org/LiteDB/issues/2305)
- [InsertBulk IEnumerable vs List 성능](https://github.com/mbdavid/LiteDB/issues/961)
- [LiteDB IL2CPP Shared Connection 이슈](https://github.com/litedb-org/LiteDB/issues/1935)
- [LiteDB IL2CPP ARM 크래시 이슈](https://github.com/mbdavid/LiteDB/issues/1759)
- [UltraLiteDB (Unity IL2CPP용)](https://github.com/rejemy/UltraLiteDB)
- [LiteDB vs SQLite 비교](https://tomaskohl.com/code/2020-04-07/trying-out-litedb/)
- [SoloDB vs LiteDB 성능 비교](https://unconcurrent.com/articles/SoloDBvsLiteDB.html)
- [LiteDB And Unity 블로그](https://www.hedberggames.com/blog/lite-db-and-unity)
- [UnityLiteDB](https://github.com/Ayrik/UnityLiteDB)
