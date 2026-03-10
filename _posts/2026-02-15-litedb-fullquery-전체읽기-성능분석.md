---
layout: post
title: 260215 LiteDB FullQuery 전체읽기 성능분석
comments: true
tags:
- unity
- performance
- optimization
- litedb
- ai
- research
- analysis
---
# ⚡ 260215 LiteDB Full Query (전체 읽기) 성능 분석

> 💡 이전 문서 [LiteDB 성능 체크 - Unity 환경](./260215_1530_LiteDB_성능체크_Unity_환경.md)의 후속 리서치로, **조건절 없이 모든 레코드를 한번에 메모리로 읽어들이는 Full Query** 성능에 집중하여 분석한다.

---

## 🗄️ 1. Full Query란?

```csharp
// LiteDB - 조건 없이 전체 컬렉션을 메모리로 로드
var allMeshes = meshCol.FindAll().ToList();
var allProps  = propCol.FindAll().ToList();

// SQLite 동등 표현
// SELECT * FROM MyMesh;
// SELECT * FROM MyProp;
```

Full Query는 **인덱스를 전혀 사용하지 않고**, 컬렉션의 모든 문서를 순차적으로 읽어 메모리에 적재하는 작업이다. LiteDB 내부에서는 다음 과정을 거친다:

```
[디스크 페이지 읽기] → [BSON 바이트 조립] → [BSON 역직렬화] → [C# 객체 매핑] → [List<T> 적재]
```

각 단계마다 병목이 발생하며, 특히 **BSON 역직렬화**와 **메모리 할당**이 핵심 병목이다.

---

## 🗄️ 2. LiteDB Full Query의 내부 동작

### 🔹 2.1 페이지 기반 순차 읽기

LiteDB v5의 데이터 파일은 **8KB 페이지** 단위로 구성된다:

| 구성 요소 | 설명 |
|-----------|------|
| **Header Page** | DB 메타데이터 (1페이지) |
| **Collection Page** | 컬렉션 정보, 인덱스 참조 |
| **Index Page** | Skip List 인덱스 노드 |
| **Data Page** | BSON 직렬화된 문서 데이터 |

Full Query 시 모든 **Data Page**를 순차적으로 읽으며, 각 페이지에서 문서를 추출한다.

> 💡 참고: [LiteDB 데이터 구조 문서](https://www.litedb.org/docs/data-structure/)

### 🔹 2.2 BSON 역직렬화 오버헤드

BSON 역직렬화에서 발생하는 오버헤드:

| 원인 | 영향 |
|------|------|
| **BsonValue 래핑** | 모든 값이 BsonValue 객체로 래핑됨 → 원본보다 메모리 사용량 증가 |
| **배열 요소별 타입 바이트** | `float[]`의 각 요소마다 타입 정보 1byte 추가 |
| **문서 전체 로드** | v4에서는 전체 byte[]를 먼저 조립 후 역직렬화, v5에서는 스트리밍 개선 |
| **GC 압력** | 대량 객체 생성으로 .NET GC 빈번 발동 |

> 💡 참고: [LiteDB 메모리 할당 이슈 #1062](https://github.com/mbdavid/LiteDB/issues/1062)

### 🔹 2.3 메모리 증폭 현상

실제 보고된 메모리 증폭 사례:

| 시나리오 | 데이터 크기 | 실제 메모리 사용 | 증폭률 |
|----------|-----------|----------------|--------|
| 20~30MB 객체 1건 조회 | 20~30 MB | **~200 MB** | **7~10x** |
| 800MB DB에서 2,000건 조회 (Run 메서드) | 800 MB | **~3 GB** | **3.75x** |
| 800MB DB에서 2,000건 조회 (Engine.Find) | 800 MB | **~400~600 MB** | **0.5~0.75x** |

> ✨ **핵심 교훈**: `FindAll().ToList()`는 모든 결과를 `List<T>`로 즉시 실체화하므로, 메모리 증폭이 극대화된다. `IEnumerable` 기반 스트리밍 소비가 메모리 효율에서 훨씬 유리하다.

> 💡 참고: [메모리 3GB 이슈 #865](https://github.com/mbdavid/LiteDB/issues/865), [200MB 할당 이슈 #1062](https://github.com/mbdavid/LiteDB/issues/1062)

---

## 🏗️ 3. 대상 스키마에 대한 Full Query 예상 성능

### 🔹 3.1 MyProp (경량 문서, ~200 bytes × 500,000건)

```
총 데이터 크기: ~95 MB (디스크)
```

| 측정 항목 | 예상치 | 산출 근거 |
|-----------|--------|----------|
| **디스크 읽기 시간** | ~1~2초 | SSD 기준 95MB ÷ 500MB/s 순차 읽기 |
| **BSON 역직렬화** | ~10~20초 | 200bytes/문서 × 50만건, 문서당 ~20~40μs |
| **C# 객체 매핑** | 역직렬화에 포함 | BsonMapper.ToObject 비용 |
| **총 소요시간** | **~15~25초** | 디스크 I/O + 역직렬화 + 객체 생성 |
| **메모리 사용** | **~300~600 MB** | 95MB × 3~6x 메모리 증폭 (BsonValue 래핑 + List 오버헤드) |

### 🎮 3.2 MyMesh (대용량 문서, ~28 KB × 500,000건, 중형 메시 기준)

```
총 데이터 크기: ~13 GB (디스크)
```

| 측정 항목 | 예상치 | 산출 근거 |
|-----------|--------|----------|
| **디스크 읽기 시간** | ~26~40초 | SSD 기준 13GB ÷ 500MB/s, 페이지 오버헤드 포함 |
| **BSON 역직렬화** | **~5~15분** | 28KB/문서 × 50만건, 배열 요소별 역직렬화 비용 극대화 |
| **C# 객체 매핑** | 역직렬화에 포함 | float[]/int[] 배열 재생성 비용 |
| **총 소요시간** | **~8~20분** | 디스크 I/O + 역직렬화 병목 |
| **메모리 사용** | **~40~80 GB** | 13GB × 3~6x 메모리 증폭 → **OutOfMemoryException 확실** |

> 💡 **결론**: MyMesh 50만건 Full Query는 **물리적으로 불가능**하다. 일반 PC의 RAM(16~32GB)을 초과하며, GC 압력으로 인한 앱 프리징과 OOM이 발생한다.

### 🔹 3.3 실제 보고 사례와의 대조

| 사례 | 조건 | 결과 |
|------|------|------|
| [이슈 #1822](https://github.com/mbdavid/LiteDB/issues/1822) | 100만건, Find() | **10분 이상** 소요, DB 타임아웃 |
| [이슈 #865](https://github.com/mbdavid/LiteDB/issues/865) | 2,000건, 800MB DB | 메모리 **3GB** 사용 |
| [이슈 #1062](https://github.com/mbdavid/LiteDB/issues/1062) | 대형 문서 1건 | 20MB 문서에 **200MB** 할당 |
| [이슈 #1214](https://github.com/mbdavid/LiteDB/issues/1214) | 대규모 테이블 | COUNT 쿼리만으로 **수 분** 소요 |

---

## ⚖️ 4. SQLite Full Query와의 비교

### 🔹 4.1 SQLite 읽기 메커니즘

SQLite는 LiteDB와 근본적으로 다른 읽기 방식을 사용한다:

```
[디스크 페이지 읽기] → [행 데이터 디코딩] → [DataReader 커서] → [필요 시 객체 매핑]
```

| 비교 항목 | LiteDB | SQLite |
|-----------|--------|--------|
| **직렬화 형식** | BSON (타입 메타데이터 풍부) | 네이티브 바이너리 (최소 오버헤드) |
| **역직렬화** | 전체 문서 파싱 필수 | 열 단위 접근 가능 |
| **메모리 모델** | 전체 문서를 C# 객체로 실체화 | DataReader 커서 기반 (행 단위) |
| **읽기 속도** | ~20,000~25,000 rec/s (소형 문서) | ~100,000+ rec/s |
| **대용량 바이너리** | BSON 배열 오버헤드 | BLOB 직접 저장 (오버헤드 없음) |
| **순차 읽기 처리량** | 가변 (문서 복잡도 의존) | ~370 MB/s (디스크 속도 근접) |

> 💡 참고: [SQLite 성능 튜닝](https://phiresky.github.io/blog/2020/sqlite-performance-tuning/), [SQLite 최적화](https://www.powersync.com/blog/sqlite-optimizations-for-ultra-high-performance)

### ⚖️ 4.2 MyProp Full Query 비교 (500,000건, ~200 bytes)

| 항목 | LiteDB | SQLite | 차이 |
|------|--------|--------|------|
| **예상 소요시간** | ~15~25초 | **~2~5초** | SQLite **3~10x 빠름** |
| **메모리 사용** | ~300~600 MB | **~100~200 MB** | SQLite **2~3x 효율적** |
| **디스크 I/O** | 95 MB (BSON 오버헤드) | ~60 MB (컴팩트) | SQLite 파일 크기 작음 |
| **GC 부하** | 높음 (BsonValue 래핑) | 낮음 (직접 매핑) | |

### ⚖️ 4.3 MyMesh Full Query 비교 (500,000건, ~28 KB 중형 메시)

| 항목 | LiteDB | SQLite | 차이 |
|------|--------|--------|------|
| **예상 소요시간** | ~8~20분 | **~1~3분** | SQLite **5~10x 빠름** |
| **메모리 사용** | ~40~80 GB (**OOM**) | **~15~30 GB** (여전히 위험) | |
| **실현 가능성** | **불가능** (OOM) | **조건부 가능** (스트리밍 필수) | |

> ✨ **핵심**: MyMesh 50만건 Full Query는 **어떤 임베디드 DB에서도 한번에 메모리 로드가 비현실적**이다. 스트리밍/페이징 전략이 필수다.

### ⚖️ 4.4 메모리 할당 효율 비교

SoloDB(SQLite 기반)와 LiteDB의 실측 비교:

| 항목 | LiteDB | SoloDB (SQLite 기반) | 차이 |
|------|--------|---------------------|------|
| **GroupBy+Count 쿼리 시간** | 50.08 ms | 21.50 ms | SQLite 2.3x 빠름 |
| **메모리 할당** | 30.37 MB | 56.05 KB | **LiteDB 555x 더 많이 할당** |

> 💡 참고: [SoloDB vs LiteDB 벤치마크](https://unconcurrent.com/articles/SoloDBvsLiteDB.html)

---

## 🗄️ 5. Full Query 대안 전략

50만건을 한번에 메모리에 올리는 것이 필요한 경우, 다음 전략을 검토해야 한다:

### 🔹 5.1 스트리밍 소비 (IEnumerable)

```csharp
// 나쁜 예 - 전체 실체화 (OOM 위험)
var all = col.FindAll().ToList();  // 전체 메모리 로드

// 좋은 예 - 스트리밍 처리 (메모리 효율)
foreach (var item in col.FindAll())
{
    ProcessItem(item);  // 한 건씩 처리 후 GC 가능
}
```

메모리 절감 효과: **75~80%** (이슈 #865 실측)

### 🔹 5.2 페이징 읽기

```csharp
int pageSize = 10_000;
int skip = 0;

while (true)
{
    var batch = col.Find(Query.All(), skip, pageSize).ToList();
    if (batch.Count == 0) break;

    ProcessBatch(batch);
    skip += pageSize;
    batch = null;  // GC 유도
}
```

| 페이지 크기 | 메모리 사용 (MyProp) | 메모리 사용 (MyMesh 중형) |
|------------|---------------------|------------------------|
| 1,000건 | ~0.6~1.2 MB | ~84~168 MB |
| 10,000건 | ~6~12 MB | ~840 MB~1.7 GB |
| 50,000건 | ~30~60 MB | **~4.2~8.4 GB** |

### 🔹 5.3 필드 선택적 조회 (Projection)

```csharp
// 전체 문서 대신 필요한 필드만 조회
var uids = col.Query()
    .Select(x => x.ModelItemUid)
    .ToList();
// 메시 배열 데이터를 제외하면 메모리 극적 감소
```

### 🔹 5.4 SQLite + 스트리밍 (최적 대안)

```csharp
// SQLite DataReader - 행 단위 커서, 최소 메모리
using var cmd = conn.CreateCommand();
cmd.CommandText = "SELECT * FROM MyMesh";
using var reader = cmd.ExecuteReader();

while (reader.Read())
{
    var uid = reader.GetString(1);
    var vertexBlob = (byte[])reader["vertexList"];
    // 행 단위 처리 → 메모리 사용 최소
}
```

---

## ⚖️ 6. 종합 비교표

### ⚡ 6.1 Full Query 속도 비교 (500,000건)

| 테이블 | 문서 크기 | LiteDB FindAll().ToList() | SQLite SELECT * (DataReader) | 차이 |
|--------|----------|--------------------------|------------------------------|------|
| **MyProp** | ~200 B | ~15~25초 | ~2~5초 | SQLite **3~10x** |
| **MyMesh (소형)** | ~3 KB | ~2~4분 | ~15~40초 | SQLite **4~6x** |
| **MyMesh (중형)** | ~28 KB | ~8~20분 | ~1~3분 | SQLite **5~10x** |
| **MyMesh (대형)** | ~280 KB | **OOM / 수 시간** | ~10~30분 | SQLite 유일 가능 |

### ⚖️ 6.2 Full Query 메모리 사용 비교 (500,000건)

| 테이블 | 디스크 크기 | LiteDB 메모리 | SQLite 메모리 (ToList) | SQLite 메모리 (DataReader) |
|--------|-----------|-------------|---------------------|--------------------------|
| **MyProp** | ~95 MB | ~300~600 MB | ~150~300 MB | **~수 MB** (스트리밍) |
| **MyMesh (중형)** | ~13 GB | **OOM** | ~15~30 GB | **~수십 MB** (스트리밍) |

### 🔹 6.3 최종 판정

| 시나리오 | LiteDB Full Query | SQLite Full Query |
|----------|-------------------|-------------------|
| MyProp 50만건 일괄 로드 | **가능하나 느림** (~20초, ~500MB) | **실용적** (~3초, ~200MB) |
| MyProp 50만건 스트리밍 | **실용적** (메모리 ~수십 MB) | **최적** (메모리 ~수 MB) |
| MyMesh 50만건 일괄 로드 | **불가능** (OOM) | **위험** (RAM 부족 가능) |
| MyMesh 50만건 스트리밍 | **매우 느림** (~10~20분) | **느리지만 가능** (~2~3분) |
| MyMesh 50만건 페이징(10K) | **가능** (~10~20분, ~1.7GB/배치) | **실용적** (~2~3분, ~280MB/배치) |

---

## 📌 7. 권장사항

### 🔹 7.1 MyProp 테이블

- **LiteDB**로도 Full Query 가능하지만, `FindAll().ToList()` 대신 **스트리밍(`IEnumerable`) 방식** 권장
- 전체 메모리 로드가 필수라면 ~500MB 메모리 예산 확보 필요
- SQLite로 전환 시 **3~10배 속도 향상** 기대

### 🎮 7.2 MyMesh 테이블

- **LiteDB Full Query는 사실상 불가능** (OOM 확실)
- SQLite를 사용하더라도 **반드시 스트리밍/페이징** 적용 필수
- 최적 전략: **SQLite BLOB 저장 + DataReader 스트리밍**
- `float[]` → `byte[]` 변환 저장으로 직렬화 오버헤드 제거

### 🏗️ 7.3 아키텍처 제안

```
[Unity App]
    │
    ├── MyProp (50만건, 경량)
    │     └── LiteDB 또는 SQLite (둘 다 가능)
    │         └── FindAll() 스트리밍 or 인덱스 쿼리
    │
    └── MyMesh (50만건, 대용량)
          └── SQLite (강력 권장)
              ├── 메타데이터 (id, uid) → 일반 컬럼
              └── 메시 데이터 (vertex, index, tri) → BLOB 컬럼 (byte[])
                  └── DataReader 스트리밍으로 건별 처리
```

---

## 🔗 참고 자료

- [LiteDB 공식 문서](https://www.litedb.org/docs/)
- [LiteDB 데이터 구조](https://www.litedb.org/docs/data-structure/)
- [LiteDB 100만건 조회 10분+ 이슈 #1822](https://github.com/mbdavid/LiteDB/issues/1822)
- [LiteDB 메모리 3GB 사용 이슈 #865](https://github.com/mbdavid/LiteDB/issues/865)
- [LiteDB 200MB 메모리 할당 이슈 #1062](https://github.com/mbdavid/LiteDB/issues/1062)
- [LiteDB COUNT 쿼리 수 분 소요 #1214](https://github.com/mbdavid/LiteDB/issues/1214)
- [LiteDB FindAll 루프 감지 버그 #2525](https://github.com/litedb-org/LiteDB/issues/2525)
- [LiteDB 메모리 사용 이슈 #670](https://github.com/litedb-org/LiteDB/issues/670)
- [SoloDB vs LiteDB 성능 비교](https://unconcurrent.com/articles/SoloDBvsLiteDB.html)
- [SQLite 성능 튜닝](https://phiresky.github.io/blog/2020/sqlite-performance-tuning/)
- [SQLite 고성능 최적화](https://www.powersync.com/blog/sqlite-optimizations-for-ultra-high-performance)
- [임베디드 DB 비교 (SQLite, Realm, LiteDB)](https://www.dajbych.net/comparison-of-databases-for-uwp-apps-sqlite-realm-litedb)
- [로컬 스토리지 성능 비교](https://codingsight.com/in-search-of-fast-local-storage/)
- [LiteDB-Perf 벤치마크](https://github.com/mbdavid/LiteDB-Perf)
