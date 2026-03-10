---
layout: post
title: 260215 LiteDB byte변환시 FullQuery 성능변화
comments: true
tags:
- performance
- litedb
- research
- analysis
---
# ⚡ 260215 LiteDB float[]→byte[] 변환 시 Full Query 성능 변화

> 💡 이전 문서 [LiteDB Full Query 전체 읽기 성능 분석](./260215_1600_LiteDB_FullQuery_전체읽기_성능분석.md)에서 MyMesh 50만건 Full Query가 OOM으로 사실상 불가능하다고 분석하였다. 본 문서에서는 **float[]를 byte[]로 변환 저장(BsonBinary)** 했을 때 Full Query 성능이 얼마나 개선되는지를 정량적으로 분석한다.

---

## 📌 1. 핵심 질문

```
float[] vertexList를 그대로 저장 (BsonArray)  →  Full Query 성능?
float[] vertexList를 byte[]로 변환 저장 (BsonBinary)  →  Full Query 성능?
둘의 차이는?
```

결론부터 말하면, **byte[] 변환은 디스크 크기, 역직렬화 속도, 메모리 사용 모든 면에서 극적인 개선**을 가져온다.

---

## 🏗️ 2. BSON 포맷 레벨: BsonArray vs BsonBinary 구조

### 🗂️ 2.1 BsonArray (타입 0x04) — float[]를 그대로 저장

BSON 스펙에 따르면, 배열은 내부적으로 **문서와 동일한 구조**로 저장된다. 각 요소마다:

```
[1B 타입코드] [키 cstring "0","1","2"...] [8B double 값]
```

| 구성 요소 | 크기 | 설명 |
|-----------|------|------|
| 타입 바이트 | 1 byte | 0x01 (double) |
| 키 이름 | 2~5 bytes | "0"\0 ~ "999"\0 (인덱스 문자열 + null) |
| 값 | **8 bytes** | float가 **double로 확장 저장** |
| 문서 헤더/종료 | 5 bytes | int32 길이 + 0x00 종료 |

> ✨ **핵심**: BSON에는 float32 타입이 없다. `float`은 항상 `double`(8bytes)로 변환되어 **데이터 자체가 2배**로 팽창한다.

> 💡 참고: [BSON 스펙](https://bsonspec.org/spec.html)

### 🗂️ 2.2 BsonBinary (타입 0x05) — byte[]로 변환 저장

```
[4B 전체 길이] [1B 서브타입 0x00] [N bytes 원시 데이터]
```

| 구성 요소 | 크기 | 설명 |
|-----------|------|------|
| 길이 | 4 bytes | int32 |
| 서브타입 | 1 byte | 0x00 (generic) |
| 데이터 | **N bytes** | float 원본 그대로 (4bytes/float) |

오버헤드는 **고정 5bytes**뿐이다.

---

## ⚖️ 3. 정량적 크기 비교: float 1,000개 기준

### 🔹 3.1 바이트 단위 계산

**BsonArray (float 1,000개):**

```
키 이름 크기:
  "0"~"9"     → 2B × 10개  = 20 bytes
  "10"~"99"   → 3B × 90개  = 270 bytes
  "100"~"999" → 4B × 900개 = 3,600 bytes
  합계: 3,890 bytes

요소 크기:
  (1B 타입 + 키 cstring + 8B double) × 1,000개
  = 1,000 + 3,890 + 8,000 = 12,890 bytes

문서 래핑:
  4B 길이 + 12,890 + 1B 종료 = 12,895 bytes

총: ~12.6 KB
```

**BsonBinary (float 1,000개 → byte 4,000개):**

```
4B 길이 + 1B 서브타입 + 4,000B 데이터 = 4,005 bytes

총: ~3.9 KB
```

### ⚖️ 3.2 비교표

| 항목 | BsonArray | BsonBinary | 차이 |
|------|-----------|------------|------|
| **데이터 영역** | 8,000 B (double 8B × 1000) | 4,000 B (float 4B × 1000) | **2x** |
| **키/메타 오버헤드** | 4,890 B | 5 B | **978x** |
| **필드 총 크기** | **~12,895 bytes** | **~4,005 bytes** | **3.2x** |
| 원본 대비 팽창률 | 3.22배 | 1.00배 | |

> 💡 **float 1,000개 기준, BsonBinary는 BsonArray의 31% 크기만 차지한다.**

---

## ⚖️ 4. 역직렬화 경로 비교

### 🔹 4.1 BsonArray 역직렬화

```
반복 1,000회:
  ┌→ 타입 바이트 읽기 (1B)
  ├→ 키 cstring 스캔 (null 종료 탐색)
  ├→ double 값 읽기 (8B)
  ├→ new BsonValue(double) 힙 할당    ← GC 객체 생성
  ├→ float로 다운캐스트
  └→ List<BsonValue>.Add()

총 GC 객체 생성: ~2,001개 (BsonValue 1,000 + List + BsonArray)
총 힙 할당: ~68 KB (BsonValue 56B × 1,000 + List 오버헤드)
```

> 💡 참고: [LiteDB 메모리 할당 이슈 #1062](https://github.com/mbdavid/LiteDB/issues/1062) — BsonValue 래핑으로 20MB 문서가 200MB 메모리 사용 (7~10x 팽창)

### 🔹 4.2 BsonBinary 역직렬화

```
1회:
  ┌→ int32 길이 읽기 (4B)
  ├→ subtype 읽기 (1B)
  ├→ ReadBytes(N) → Buffer.BlockCopy 1회  ← 네이티브 memcpy
  └→ new byte[N] 힙 할당 1회

이후 float[] 복원:
  ┌→ new float[N/4] 힙 할당 1회
  └→ Buffer.BlockCopy(bytes → floats) 1회  ← 네이티브 memcpy

총 GC 객체 생성: 3개 (BsonValue, byte[], float[])
총 힙 할당: ~8 KB (byte[] 4KB + float[] 4KB)
```

> 💡 `Buffer.BlockCopy`는 .NET 런타임 내부에서 **네이티브 memcpy를 호출**하며, 1KB 복사에 ~23ns 소요된다. 사실상 병목이 될 수 없는 수준이다.

### ⚡ 4.3 역직렬화 성능 비교

| 항목 | BsonArray | BsonBinary | 차이 |
|------|-----------|------------|------|
| **파싱 반복 횟수** | 1,000회 (요소별) | 1회 (블록) | **1,000x** |
| **GC 객체 생성** | ~2,001개 | 3개 | **~667x** |
| **힙 할당량** | ~68 KB | ~8 KB | **~8.5x** |
| **실 데이터 비율** | 5.9% (4KB/68KB) | 50% (4KB/8KB) | |
| **추정 역직렬화 시간** | ~50~100 μs/문서 | **~5~10 μs/문서** | **~5~10x** |

> 💡 **추정 근거**: BsonArray는 요소별 파싱 + 객체 생성 비용이 지배적이고, BsonBinary는 memcpy 한 번이므로 이론적으로 **5~10배 빠를 것**으로 추정된다. GitHub에서 이 둘을 직접 비교한 공식 벤치마크는 발견되지 않았으나, 메모리 할당 패턴 분석에 기반한 추정이다.

---

## ⚖️ 5. MyMesh 50만건 Full Query: BsonArray vs BsonBinary

### 🔹 5.1 중형 메시 기준 (정점 1,000개)

하나의 MyMesh 문서에 포함되는 배열:
- `vertexList`: float × 3,000 (x,y,z × 1,000정점)
- `indexList`: int × 1,000
- `triangleIndexList`: int × 3,000

| 항목 | BsonArray 저장 | BsonBinary 저장 | 개선율 |
|------|---------------|----------------|--------|
| **문서 크기 (디스크)** | ~28 KB | **~28 KB → ~9.2 KB** | **3.0x 감소** |
| 50만건 DB 파일 크기 | ~13 GB | **~4.3 GB** | **3.0x 감소** |

> 💡 문서 크기 산출:
> 💡 - BsonArray: vertexList 38.7KB + indexList 12.9KB + triIndexList 38.7KB + 기타 0.1KB ≈ **~90 KB** (이전 문서에서 ~28KB로 산출한 것은 원본 크기 기준이었음. BSON 팽창 반영 시 실제 디스크는 더 큼)
> 💡 - BsonBinary: vertexList 12KB + indexList 4KB + triIndexList 12KB + 기타 0.1KB ≈ **~28 KB**

**BSON 팽창을 정확히 반영한 재산출:**

| 필드 | 원본 크기 | BsonArray 디스크 | BsonBinary 디스크 |
|------|----------|-----------------|------------------|
| vertexList (3,000 floats) | 12 KB | ~38.7 KB | ~12.0 KB |
| indexList (1,000 ints) | 4 KB | ~12.9 KB | ~4.0 KB |
| triIndexList (3,000 ints) | 12 KB | ~38.7 KB | ~12.0 KB |
| id + modelItemUid | 0.1 KB | ~0.1 KB | ~0.1 KB |
| **문서 합계** | **28 KB** | **~90 KB** | **~28 KB** |
| **50만건 합계** | 13 GB | **~42 GB** | **~13 GB** |

### ⚡ 5.2 Full Query 성능 비교 (50만건)

#### ▫️ 디스크 I/O

| 항목 | BsonArray | BsonBinary | 개선 |
|------|-----------|------------|------|
| DB 파일 크기 | ~42 GB | ~13 GB | **3.2x 감소** |
| 8KB 페이지 수 | ~5,500,000 | ~1,700,000 | **3.2x 감소** |
| SSD 순차 읽기 시간 | ~84초 | ~26초 | **3.2x 빠름** |

#### ▫️ 역직렬화

| 항목 | BsonArray | BsonBinary | 개선 |
|------|-----------|------------|------|
| 문서당 파싱 횟수 | ~7,000회 (요소별) | 3회 (블록) | **~2,300x** |
| 문서당 GC 객체 | ~14,000개 | ~9개 | **~1,500x** |
| 문서당 역직렬화 시간 | ~350~700 μs | ~35~70 μs | **~10x** |
| **50만건 총 역직렬화** | **~3~6분** | **~18~35초** | **~5~10x** |

#### ▫️ 메모리 사용

| 항목 | BsonArray | BsonBinary | 개선 |
|------|-----------|------------|------|
| 문서당 힙 할당 | ~476 KB | ~56 KB | **~8.5x 감소** |
| 50만건 총 메모리 | **~220 GB (OOM)** | **~26 GB (여전히 OOM)** | 3.2x 감소이나 **여전히 불가** |
| 메모리 증폭률 | ~5.2x | ~2.0x | |

#### 🗄️ Full Query 총 소요시간

| 항목 | BsonArray | BsonBinary | 개선 |
|------|-----------|------------|------|
| 디스크 I/O | ~84초 | ~26초 | 3.2x |
| 역직렬화 | ~3~6분 | ~18~35초 | 5~10x |
| **총 소요시간** | **~5~7분** | **~1~2분** | **~3~5x** |
| **실현 가능성** | **OOM (불가능)** | **OOM (불가능)** | 둘 다 ToList() 불가 |

### ⚖️ 5.3 스트리밍 모드 (IEnumerable) 에서의 비교

`FindAll().ToList()` 대신 `foreach`로 스트리밍 소비할 경우:

| 항목 | BsonArray + 스트리밍 | BsonBinary + 스트리밍 | 개선 |
|------|---------------------|----------------------|------|
| **총 소요시간** | ~5~7분 | **~1~2분** | **~3~5x** |
| **피크 메모리** | ~500 MB~1 GB | **~60~120 MB** | **~8x** |
| **GC 부하** | 극심 (70억 객체 생성) | 낮음 (~450만 객체) | **~1,500x** |
| **실현 가능성** | 가능하나 매우 느림 | **실용적** | |

> ✨ **핵심**: byte[] 변환 + 스트리밍 조합이면 MyMesh 50만건 Full Query가 **~1~2분, 메모리 ~100MB 내**에서 실현 가능해진다.

---

## 📌 6. MyProp 테이블은 영향 없음

MyProp은 `float[]`/`int[]` 배열이 없으므로 BsonArray→BsonBinary 전환의 영향을 받지 않는다.

| 항목 | 변경 전 | 변경 후 | 차이 |
|------|--------|--------|------|
| 문서 크기 | ~200 B | ~200 B | 없음 |
| Full Query 시간 | ~15~25초 | ~15~25초 | 없음 |
| 메모리 | ~300~600 MB | ~300~600 MB | 없음 |

---

## ⚖️ 7. 페이징 모드에서의 비교 (10,000건/배치)

50만건을 10,000건씩 페이징으로 읽을 때:

| 항목 | BsonArray | BsonBinary | 개선 |
|------|-----------|------------|------|
| 배치당 소요시간 | ~7~14초 | **~1.2~2.4초** | ~5~6x |
| 배치당 메모리 | ~4.4 GB | **~0.5 GB** | ~8.5x |
| 총 소요시간 (50배치) | ~6~12분 | **~1~2분** | ~5x |
| 총 피크 메모리 | ~4.4 GB (위험) | **~0.5 GB (안전)** | ~8.5x |

---

## ⚖️ 8. 종합 비교표

### 🎮 MyMesh 50만건 (중형 메시, 정점 1,000개)

| 시나리오 | BsonArray | BsonBinary | 개선율 |
|----------|-----------|------------|--------|
| **DB 파일 크기** | ~42 GB | ~13 GB | **3.2x 감소** |
| **Full Query (ToList)** | OOM 불가능 | OOM 불가능 | 둘 다 불가 |
| **Full Query (스트리밍)** | ~5~7분, ~1GB | **~1~2분, ~100MB** | **시간 3~5x, 메모리 8x** |
| **페이징 (10K건/배치)** | ~6~12분, ~4.4GB | **~1~2분, ~0.5GB** | **시간 5x, 메모리 8.5x** |
| **GC 객체 수 (전체)** | ~70억 개 | ~450만 개 | **~1,500x 감소** |
| **역직렬화 속도** | ~350~700 μs/건 | ~35~70 μs/건 | **~10x** |

### 🔹 변환 효과 요약 다이어그램

```
float[] → BsonArray 저장          float[] → byte[] → BsonBinary 저장
─────────────────────             ────────────────────────────────

디스크:  ████████████ 42GB         디스크:  ████ 13GB          (3.2x↓)
시간:    ████████████ 5~7min       시간:    ███ 1~2min         (3~5x↓)
메모리:  █████████ ~1GB(스트림)     메모리:  █ ~100MB(스트림)    (8x↓)
GC객체:  ████████████ 70억         GC객체:  █ 450만            (1,500x↓)
```

---

## 📌 9. 최종 권장사항

### 🔹 9.1 byte[] 변환은 필수

| 판정 | 내용 |
|------|------|
| **결론** | float[]/int[]의 byte[] 변환 저장은 **선택이 아닌 필수** |
| **디스크** | 3.2배 감소 (42GB → 13GB) |
| **속도** | 3~5배 향상 (스트리밍 기준) |
| **메모리** | 8~8.5배 감소 |
| **GC 부하** | 1,500배 감소 — Unity 프레임 끊김 방지에 결정적 |

### ⚠️ 9.2 그래도 남는 한계

byte[] 변환을 적용하더라도:

| 한계 | 설명 |
|------|------|
| **ToList() 일괄 로드** | 50만건 × 28KB = ~13GB → 여전히 OOM |
| **스트리밍 1~2분** | 실시간 응답에는 부적합 |
| **LiteDB 자체 한계** | 단일 인덱스만 지원, BSON 파싱 오버헤드 잔존 |

### 🚀 9.3 추가 최적화 전략

byte[] 변환 이후에도 성능이 부족하다면:

| 전략 | 기대 효과 | 비고 |
|------|----------|------|
| **SQLite BLOB 전환** | 추가 2~3배 속도 향상 | DataReader 스트리밍으로 메모리 최소화 |
| **인덱스 쿼리로 대체** | Full Query 자체를 회피 | modelItemUid 인덱스로 필요한 건만 조회 |
| **Projection** | 메시 배열 제외한 메타데이터만 조회 | `Select(x => x.ModelItemUid)` |
| **메모리 캐시 계층** | DB 조회 최소화 | Dictionary 캐시 + LRU 전략 |

---

## 🔗 참고 자료

- [BSON 공식 스펙](https://bsonspec.org/spec.html)
- [LiteDB Object Mapping 문서](https://www.litedb.org/docs/object-mapping/)
- [LiteDB 데이터 구조 문서](https://www.litedb.org/docs/data-structure/)
- [LiteDB BsonDocument 문서](https://www.litedb.org/docs/bsondocument/)
- [LiteDB 메모리 할당 이슈 #1062 — 7~10x 팽창](https://github.com/mbdavid/LiteDB/issues/1062)
- [LiteDB byte[] 저장 시 9% 크기 감소 #2443](https://github.com/litedb-org/LiteDB/issues/2443)
- [LiteDB 메모리 3GB 이슈 #865](https://github.com/mbdavid/LiteDB/issues/865)
- [LiteDB 100만건 10분+ 이슈 #1822](https://github.com/mbdavid/LiteDB/issues/1822)
- [LiteDB BsonMapper 소스코드](https://github.com/litedb-org/LiteDB/blob/master/LiteDB/Client/Mapper/BsonMapper.cs)
- [LiteDB BufferReader 소스코드](https://github.com/litedb-org/LiteDB/blob/master/LiteDB/Engine/Disk/Streams/BufferReader.cs)
- [.NET Buffer.BlockCopy — 내부적으로 memcpy 호출](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/Buffer.cs)
- [SoloDB vs LiteDB 메모리 할당 555배 차이](https://unconcurrent.com/articles/SoloDBvsLiteDB.html)
