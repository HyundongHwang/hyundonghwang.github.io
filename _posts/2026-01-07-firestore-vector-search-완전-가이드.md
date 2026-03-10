---
layout: post
title: "260107 Firestore Vector Search 완전 가이드"
comments: true
tags:
  - Firestore
  - Vector-Search
  - Google-Cloud
  - Python
  - LangChain
---

## 개요
Firestore Vector Search는 2024년에 출시된 Google의 공식 벡터 검색 솔루션으로, PostgreSQL의 pgvector와 유사한 기능을 Firestore 환경에서 제공합니다.

## 핵심 기능
- K-nearest neighbor (KNN) 벡터 검색 지원
- 최대 2048차원 벡터 저장 (권장: 768차원)
- COSINE, EUCLIDEAN 거리 측정 방식 지원
- 부등호 필터와 함께 벡터 검색 가능
- 거리 임계값 설정 가능
- 최대 1000개 결과 반환 제한

## Firebase Extensions 솔루션

### Vector Search with Firestore 확장

- 자동 임베딩 생성: 문서 추가/업데이트 시 자동 벡터 생성
- Gemini/Vertex AI 통합: Google AI 모델을 이용한 임베딩 생성
- Genkit SDK 기반: 확장 가능한 아키텍처 제공

### 설치 방법

```bash
firebase ext:install googlecloud/firestore-vector-search
```

## Python 구현 가이드

### 1. 환경 설정

라이브러리 설치

```bash
pip install google-cloud-firestore google-cloud-aiplatform langchain-google-firestore langchain-google-vertexai
```

필수 임포트

```python
from google.cloud import firestore
from google.cloud.firestore_v1.vector import Vector
from google.cloud.firestore_v1.base_vector_query import DistanceMeasure
from google.cloud.firestore_v1.base_query import FieldFilter
import numpy as np
```

### 2. CREATE - 벡터 문서 생성

단일 문서 생성

```python
def create_vector_document():
    """벡터 임베딩을 포함한 문서 생성"""
    db = firestore.Client()
    collection = db.collection("documents")

    doc_data = {
        "title": "Python 프로그래밍 가이드",
        "content": "Python은 간결하고 읽기 쉬운 프로그래밍 언어입니다.",
        "category": "programming",
        "embedding": Vector([0.1, 0.2, 0.3, 0.4, 0.5]),  # 실제로는 768차원
        "created_at": firestore.SERVER_TIMESTAMP
    }

    doc_ref = collection.add(doc_data)
    print(f"문서 생성됨: {doc_ref[1].id}")
    return doc_ref[1].id
```

대량 문서 생성

```python
def bulk_create_documents():
    """벡터 문서 대량 생성"""
    db = firestore.Client()
    collection = db.collection("documents")

    documents = [
        {
            "title": "Python 기초",
            "content": "Python 변수와 데이터 타입",
            "category": "programming",
            "embedding": Vector(np.random.rand(768).tolist())
        },
        {
            "title": "머신러닝 입문",
            "content": "머신러닝의 기본 개념과 알고리즘",
            "category": "ai",
            "embedding": Vector(np.random.rand(768).tolist())
        }
    ]

    batch = db.batch()
    for i, doc in enumerate(documents):
        doc_ref = collection.document(f"doc_{i}")
        batch.set(doc_ref, doc)

    batch.commit()
    print(f"{len(documents)}개 문서 생성 완료")
```

### 3. READ - 벡터 유사도 검색

기본 벡터 검색

```python
def vector_similarity_search(query_vector, limit=5):
    """벡터 유사도 검색"""
    db = firestore.Client()
    collection = db.collection("documents")

    vector_query = collection.find_nearest(
        vector_field="embedding",
        query_vector=Vector(query_vector),
        distance_measure=DistanceMeasure.COSINE,  # 또는 EUCLIDEAN
        limit=limit
    )

    results = []
    for doc in vector_query.stream():
        doc_dict = doc.to_dict()
        doc_dict['id'] = doc.id
        results.append(doc_dict)
        print(f"문서 ID: {doc.id}, 제목: {doc_dict['title']}")

    return results
```

필터링된 벡터 검색

```python
def filtered_vector_search(query_vector, category_filter):
    """필터링된 벡터 검색"""
    db = firestore.Client()
    collection = db.collection("documents")

    vector_query = collection.where("category", "==", category_filter).find_nearest(
        vector_field="embedding",
        query_vector=Vector(query_vector),
        distance_measure=DistanceMeasure.COSINE,
        limit=10
    )

    results = []
    for doc in vector_query.stream():
        doc_dict = doc.to_dict()
        doc_dict['id'] = doc.id
        results.append(doc_dict)

    return results
```

### 4. UPDATE - 벡터 문서 업데이트

벡터 임베딩 업데이트

```python
def update_vector_document(doc_id, new_embedding, **updates):
    """문서의 벡터 임베딩 업데이트"""
    db = firestore.Client()
    doc_ref = db.collection("documents").document(doc_id)

    update_data = {
        "embedding": Vector(new_embedding),
        "updated_at": firestore.SERVER_TIMESTAMP
    }
    update_data.update(updates)

    doc_ref.update(update_data)
    print(f"문서 {doc_id} 업데이트 완료")
```

콘텐츠와 임베딩 함께 업데이트

```python
def update_content_and_embedding(doc_id, new_content, new_embedding):
    """콘텐츠와 임베딩 함께 업데이트"""
    db = firestore.Client()
    doc_ref = db.collection("documents").document(doc_id)

    doc_ref.update({
        "content": new_content,
        "embedding": Vector(new_embedding),
        "updated_at": firestore.SERVER_TIMESTAMP
    })

    print(f"문서 {doc_id}의 내용과 임베딩 업데이트 완료")
```

### 5. DELETE - 벡터 문서 삭제

단일 문서 삭제

```python
def delete_vector_document(doc_id):
    """특정 문서 삭제"""
    db = firestore.Client()
    doc_ref = db.collection("documents").document(doc_id)

    if doc_ref.get().exists:
        doc_ref.delete()
        print(f"문서 {doc_id} 삭제 완료")
    else:
        print(f"문서 {doc_id}가 존재하지 않습니다")
```

카테고리별 문서 삭제

```python
def delete_documents_by_category(category):
    """카테고리별 문서 삭제"""
    db = firestore.Client()
    collection = db.collection("documents")

    docs = collection.where("category", "==", category).stream()

    batch = db.batch()
    deleted_count = 0

    for doc in docs:
        batch.delete(doc.reference)
        deleted_count += 1

    batch.commit()
    print(f"{category} 카테고리 문서 {deleted_count}개 삭제 완료")
```
    
## LangChain 통합

설정 및 기본 사용

```python
from langchain_google_firestore import FirestoreVectorStore
from langchain_google_vertexai import VertexAIEmbeddings

def setup_langchain_vector_store():
    """LangChain을 이용한 Firestore 벡터 스토어 설정"""
    embedding = VertexAIEmbeddings(
        model_name="textembedding-gecko@latest"
    )

    vector_store = FirestoreVectorStore(
        collection="langchain_docs",
        embedding=embedding
    )

    return vector_store

def langchain_crud_operations():
    """LangChain을 이용한 CRUD 작업"""
    vector_store = setup_langchain_vector_store()

    # CREATE - 텍스트 문서들 추가
    texts = [
        "Python은 객체지향 프로그래밍 언어입니다.",
        "머신러닝은 인공지능의 한 분야입니다.",
        "Django는 Python 웹 프레임워크입니다."
    ]

    ids = ["python_oop", "ml_basics", "django_web"]
    vector_store.add_texts(texts, ids=ids)

    # READ - 유사도 검색
    results = vector_store.similarity_search(
        "프로그래밍 언어에 대해 알려주세요",
        k=2
    )

    for result in results:
        print(f"유사한 문서: {result.page_content}")

    # UPDATE - 문서 업데이트 (같은 ID로 다시 추가)
    updated_text = ["Python은 동적 타이핑을 지원하는 프로그래밍 언어입니다."]
    vector_store.add_texts(updated_text, ids=["python_oop"])

    return vector_store
```

## 인프라 설정

벡터 인덱스 생성

```bash
# 벡터 인덱스 생성 (터미널에서 실행)
gcloud firestore indexes composite create \
  --collection-group=documents \
  --query-scope=COLLECTION \
  --field-config field-path=embedding,vector-config='{"dimension":"768", "flat": "{}"}' \
  --project=YOUR_PROJECT_ID
```

## 완전한 사용 예제

```python
if __name__ == "__main__":
    # 1. 문서 생성
    doc_id = create_vector_document()
    bulk_create_documents()

    # 2. 벡터 검색
    query_embedding = np.random.rand(768).tolist()  # 실제로는 AI 모델에서 생성
    results = vector_similarity_search(query_embedding, limit=3)

    # 3. 필터링된 검색
    filtered_results = filtered_vector_search(query_embedding, "programming")

    # 4. 문서 업데이트
    new_embedding = np.random.rand(768).tolist()
    update_vector_document(doc_id, new_embedding, title="업데이트된 제목")

    # 5. LangChain 사용
    vector_store = langchain_crud_operations()

    print("모든 CRUD 작업 완료!")
```

## 제약사항 및 고려사항
- 최대 차원수: 2048차원 (권장 768차원)
- 검색 결과 제한: 최대 1000개 문서
- 거리 측정: COSINE, EUCLIDEAN만 지원
- 실시간 리스너: 벡터 검색에서는 미지원
- 인덱스 필수: 벡터 검색 전 인덱스 생성 필요