---
title: "AI Search 하이브리드 검색 설정 — 요청 구조와 가중치, 막히는 곳"
date: 2026-11-28 06:53:00 +0900
categories: [Azure AI]
tags: [azure, rag, foundry]
description: Azure AI Search 하이브리드 쿼리를 문서 기준 순서로 정리했다. vectorQueries 구조, RRF, weight, 시맨틱 랭커 조합, 막히는 지점을 다룬다.
---

> 이 글은 Microsoft Learn의 Azure AI Search 하이브리드 검색 문서를 읽고 정리·재구성한 것이다. 문서 기준 순서이며, 직접 실행한 기록이 아니다. 문서에서 제품명이 "Foundry IQ (Azure AI Search)"로 표기되기도 한다.
{: .prompt-info }

## 결론 3줄

- 하이브리드 검색은 한 요청에 `search`(전문 검색)와 `vectorQueries`(벡터)를 함께 넣으면 되고, 결과는 RRF로 합쳐진다.
- 시맨틱 랭커를 얹을 땐 벡터 쿼리의 `k`를 50으로 맞춘다. 입력이 모자라면 재순위 품질이 떨어진다.
- RRF 점수는 0.03 안팎으로 낮게 나오는 게 정상이다. 점수 절대값으로 컷오프하지 않는다.

## 단계별 순서 (문서 기준)

### 1. 인덱스에 벡터 필드가 있는지 확인한다

인덱스 정의에 `vectorSearch` 섹션이 있고, 검색 가능한(`searchable`) 텍스트 필드와 벡터 필드가 함께 있어야 한다. 문서는 Import data 마법사로 만드는 것을 권한다. 문서와 질의에는 같은 임베딩 모델을 써야 한다.

### 2. 권한을 맞춘다

쿼리만 한다면 `Search Index Data Reader`, 인덱스를 만들거나 고치려면 `Search Index Data Contributor`가 필요하다.

### 3. Search explorer에서 JSON 뷰로 테스트한다

Azure Portal에서 검색 서비스를 열고 **Search management > Indexes**에서 인덱스를 고른다. 첫 탭이 **Search explorer**다. **View > JSON view**로 바꾸면 요청 본문을 그대로 붙여 넣을 수 있다. **Query Options**에서 **Hide vector values in search results**를 켜 두면 결과가 읽기 쉬워진다.

### 4. 요청 본문을 만든다

REST는 `POST https://{서비스}.search.windows.net/indexes/{인덱스}/docs/search?api-version=2026-10-01` 형식이다. 문서 기준 최소 구조는 다음과 같다. 벡터 값은 생략했다.

```json
{
  "search": "historic hotel walk to restaurants and shopping",
  "vectorQueries": [
    {
      "kind": "vector",
      "vector": [ ... ],
      "fields": "DescriptionVector",
      "k": 10
    }
  ],
  "select": "HotelName, Description, Address/City",
  "top": 10
}
```

`kind`는 `vector`(배열을 직접 전달)와 `text`(문자열 전달, 필드에 vectorizer 필요) 두 가지다. `select`에는 사람이 읽는 텍스트 필드를 넣고 벡터 필드는 빼는 게 낫다. 문서 예제 일부에는 `top`이 따옴표 문자열로 적힌 곳이 있지만 정수로 쓰는 것이 맞다.

### 5. 가중치를 조절한다

`vectorQueries` 항목마다 `weight`를 줄 수 있다. 기본은 1.0이고 0보다 큰 양수여야 한다. RRF 전에 해당 쿼리의 순위 점수에 곱해진다. 가중치는 벡터 쿼리에만 있고 텍스트 쿼리는 암묵적으로 1.0이다. 어떤 값이 맞는지 보려면 `debug: "vector"`로 하위 점수를 확인한다.

### 6. 시맨틱 랭커를 얹는다

`queryType: "semantic"`과 `semanticConfiguration`을 추가한다. 문서는 시맨틱 랭커가 입력을 최대 50건 받는다고 설명하고, 벡터 쿼리의 `k`를 50으로 두라고 한다. 시맨틱 점수(`@search.rerankerScore`)는 0~4 범위이고 RRF 이후 별도로 계산된다.

## 방식 비교 — 무엇을 고를까

| 상황 | 전문 검색(BM25) | 벡터 | 하이브리드(RRF) |
|---|---|---|---|
| 제품 코드·이름·날짜 같은 정확 일치 | 강함 | 약함 | 강함 |
| 표현이 달라도 의미가 같은 질문 | 약함 | 강함 | 강함 |
| 일치 문서가 없을 때 | 0건 가능 | 항상 `k`건 반환 | 텍스트 쪽이 비면 벡터가 채움 |
| 점수 범위 | 상한 없음 | Cosine 0.333~1.00 | 합쳐지는 쿼리 수에 묶임, 값이 작음 |
| 추가 비용 | 낮음 | 임베딩 필요 | 둘 다 필요 |

사내 문서 검색처럼 규정 번호와 자연어 질문이 섞이는 경우는 하이브리드가 기본값이 된다. 반대로 사용자가 항상 정확한 ID로만 찾는다면 전문 검색만으로 충분하다.

<figure class="sketch">
<svg viewBox="0 0 720 260" role="img" aria-label="하이브리드 검색에서 텍스트 순위와 벡터 순위가 RRF로 합쳐지고 시맨틱 랭커가 재순위하는 흐름">
  <defs>
    <marker id="hyb-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="hyb-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 하이브리드 쿼리의 처리 순서</text>
  <rect x="8" y="50" width="170" height="52" rx="8" class="sk-box"/>
  <text x="93" y="74" text-anchor="middle" class="sk-label">search (BM25)</text>
  <text x="93" y="92" text-anchor="middle" class="sk-sub">정확 일치·필터</text>
  <rect x="8" y="140" width="170" height="52" rx="8" class="sk-box"/>
  <text x="93" y="164" text-anchor="middle" class="sk-label">vectorQueries</text>
  <text x="93" y="182" text-anchor="middle" class="sk-sub">k, weight</text>
  <path d="M180,76 L250,112" class="sk-line" marker-end="url(#hyb-m1)"/>
  <path d="M180,166 L250,130" class="sk-line" marker-end="url(#hyb-m1)"/>
  <rect x="252" y="90" width="150" height="62" rx="8" class="sk-box-accent"/>
  <text x="327" y="118" text-anchor="middle" class="sk-label">RRF 병합</text>
  <text x="327" y="136" text-anchor="middle" class="sk-sub">1/(rank+k) 합산</text>
  <path d="M404,121 L464,121" class="sk-line-accent" marker-end="url(#hyb-a1)"/>
  <rect x="466" y="90" width="120" height="62" rx="8" class="sk-box-muted"/>
  <text x="526" y="118" text-anchor="middle" class="sk-label">시맨틱 랭커</text>
  <text x="526" y="136" text-anchor="middle" class="sk-sub">선택, 입력 50건</text>
  <path d="M588,121 L636,121" class="sk-line" marker-end="url(#hyb-m1)"/>
  <text x="640" y="126" class="sk-label">top</text>
  <text x="8" y="236" class="sk-sub">두 목록을 순위 기준으로 합치기 때문에 BM25와 코사인 점수의 단위 차이가 문제 되지 않는다</text>
</svg>
<figcaption>하이브리드는 점수가 아니라 순위를 합치고, 시맨틱 랭커는 그 뒤에 선택적으로 붙는다.</figcaption>
</figure>

## 자주 막히는 지점

### 1. RRF 점수가 너무 낮아 보인다

문서의 예제에서도 강한 일치가 0.0328 같은 값으로 나온다. RRF는 `1/(rank + k)`를 합산하는 방식이라 값이 작다. 점수 임계값으로 걸러내려는 시도는 맞지 않는다. 문서는 `threshold`(프리뷰, 2026-11-01-preview)가 단일 벡터 쿼리용이고 하이브리드에는 어울리지 않는다고 설명한다. 하이브리드의 품질은 하위 점수와 상위 결과를 직접 보며 판단한다.

### 2. 벡터 쿼리는 항상 k건을 돌려준다

관련 없는 문서라도 가장 가까운 k건이 반환된다. 정확 일치가 있어야 하는 질의라면 텍스트 쪽 매칭이 더해져야 순위가 안정된다. 벡터만 쓰는 구성에서 "엉뚱한 결과가 나온다"는 문제의 상당수가 여기서 나온다.

### 3. `orderby`를 넣었더니 관련도 순서가 사라진다

`orderby`는 관련도 정렬을 덮어쓴다. 하이브리드 검색에서는 빼는 것이 원칙이다.

### 4. 시맨틱을 켰는데 결과가 빈약하다

`k`와 텍스트 후보 수의 합이 50에 못 미치거나, 필터를 `postFilter`로 걸어 후보가 줄면 재순위 입력이 모자란다. 기본값 `preFilter`가 후보를 가장 많이 확보한다. `strictPostFilter`(프리뷰)는 시맨틱 랭커와 함께 쓰기를 권하지 않는다고 문서에 적혀 있다.

### 5. 작은 티어에서 429나 지연 급증이 생긴다

공격적인 텍스트 가중, 큰 `maxTextRecallSize`, 높은 `efSearch`·`maxConnections`, 시맨틱 재순위를 한꺼번에 쓰면 부하가 몰린다. 문서가 권하는 순서는 `efSearch`(예: 800 → 128~192)와 `maxConnections`(64 → 32)를 낮추고, 시맨틱 적용 범위를 줄이고, 마지막에 레플리카를 늘리는 것이다.

### 4번과 별개로 확인할 점 — 미검증 필드

개요 문서 예제의 `oversampling` 속성은 안정 버전 필드 목록에서 확인하지 못했다. 쓰기 전에 직접 확인한다. `threshold`·`hybridSearch.maxTextRecallSize`는 2026-11-01-preview 기능이므로 프리뷰 버전에서만 동작한다.

## 도입 전 점검표

- 인덱스에 `vectorSearch`와 텍스트·벡터 필드가 모두 있는가
- 질의 임베딩과 문서 임베딩이 같은 모델인가
- 쓰는 API 버전이 안정(2026-10-01)인가, 프리뷰 기능이 섞였는가
- 시맨틱 사용 시 `k`와 텍스트 후보의 합이 50 이상인가
- `select`에서 벡터 필드를 뺐는가

문서 쪽 청킹 전처리는 [Azure SQL로 RAG 청킹하기](/posts/azure-sql-ai-generate-chunks-rag-guide/) 글에서 이어 볼 수 있다.

## 참고

- [Hybrid search overview (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview)
- [Create a hybrid query (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/search/hybrid-search-how-to-query)
- [Relevance scoring in hybrid search using RRF (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/search/hybrid-search-ranking)
- [Vector search ranking (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/search/vector-search-ranking)
- [Create a vector query (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/search/vector-search-how-to-query)
