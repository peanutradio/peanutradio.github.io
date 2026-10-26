---
title: "Agent Framework 벡터 스토어 — 검색을 도구로, 대화 기록까지 검색 대상으로"
date: 2026-10-16 06:53:00 +0900
categories: [AI Agent]
tags: [ai-agent, rag, automation]
description: Agent Framework Python이 Semantic Kernel의 벡터 스토어를 옮겨 왔다. 검색을 에이전트 도구로 노출하고 긴 대화는 검색 가능한 기록으로 빼는 구조를 정리한다.
---

> 이 글은 Microsoft Agent Framework 블로그의 "Vector Stores in Python Agent Framework" 발표(2026-10-15, Eduard van Valkenburg)를 읽고 정리·재구성한 것입니다. 글의 코드를 직접 실행해 본 기록은 아닙니다.
{: .prompt-info }

## 결론부터

이번 발표의 핵심은 새 벡터 DB가 아니라 **"검색을 에이전트가 쓰는 도구로 만드는 표준 길"**이 생겼다는 점입니다. 데이터 모델을 한 번 정의하면 인메모리부터 Azure AI Search, pgvector까지 같은 코드로 갈아 끼울 수 있고, 검색 필터 값을 모델이 채우게 하거나 긴 대화를 검색 가능한 저장소로 빼는 것까지 프레임워크가 맡습니다. 다만 벡터 스토어 API는 **experimental**, 통합 패키지는 **alpha 또는 beta**입니다. 운영에 바로 넣을 단계는 아닙니다.

## 원문이 말하는 것 (요약)

- Semantic Kernel의 벡터 스토어 API를 Agent Framework Python으로 옮겼습니다. 모델 등록, 배치 API, 필터, 임베딩 옵션이 다듬어졌습니다.
- `VectorStoreField`로 key/data/vector 필드를 표시하고 `@vectorstoremodel(collection_name=...)`로 컬렉션을 정의합니다. 원문 예제의 벡터 필드는 1536차원, 거리 함수는 `cosine_similarity`입니다.
- `create_vector_search_tool`은 컬렉션 검색을 에이전트 도구로 감쌉니다. 필터 안의 `Param`은 모델이 채우는 인자가 되고, `Literal` 값은 스키마의 enum이 됩니다. 모델이 값을 생략하면 그 필터는 빠집니다.
- `VectorCollectionContextProvider`는 upsert·get·delete·search 도구를 묶어 붙이고, `VectorStoreHistoryProvider`는 전체 대화를 벡터 컬렉션에 저장한 뒤 압축한 작업 집합만 컨텍스트에 올립니다. `include_search_tool=True`면 전체 기록을 검색하는 도구도 생깁니다.
- 지원 저장소는 Azure AI Search, Cosmos DB for NoSQL, Azure DocumentDB, MongoDB, Oracle AI Database 26ai, Redis, SQL Server/Azure SQL, PostgreSQL(pgvector), DuckDB, Qdrant, 그리고 테스트용 인메모리입니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="에이전트가 검색 도구와 대화 기록 도구를 통해 같은 컬렉션 인터페이스 뒤의 여러 저장소에 접근하는 구조">
  <defs>
    <marker id="avs-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 에이전트 → 도구 → 같은 컬렉션 API → 저장소</text>
  <rect x="8" y="60" width="130" height="60" rx="8" class="sk-box-accent"/>
  <text x="73" y="86" text-anchor="middle" class="sk-label">에이전트</text>
  <text x="73" y="104" text-anchor="middle" class="sk-sub">모델이 인자를 채움</text>
  <rect x="190" y="40" width="190" height="46" rx="8" class="sk-box"/>
  <text x="285" y="60" text-anchor="middle" class="sk-mono">create_vector_search_tool</text>
  <text x="285" y="76" text-anchor="middle" class="sk-sub">Param → 도구 인자</text>
  <rect x="190" y="104" width="190" height="46" rx="8" class="sk-box"/>
  <text x="285" y="124" text-anchor="middle" class="sk-mono">VectorStoreHistoryProvider</text>
  <text x="285" y="140" text-anchor="middle" class="sk-sub">전체 기록 검색 + 압축 컨텍스트</text>
  <rect x="430" y="72" width="130" height="60" rx="8" class="sk-box-accent"/>
  <text x="495" y="98" text-anchor="middle" class="sk-label">컬렉션 API</text>
  <text x="495" y="116" text-anchor="middle" class="sk-sub">upsert / search</text>
  <rect x="595" y="40" width="117" height="40" rx="8" class="sk-box-muted"/>
  <text x="653" y="64" text-anchor="middle" class="sk-sub">인메모리(테스트)</text>
  <rect x="595" y="92" width="117" height="40" rx="8" class="sk-box-muted"/>
  <text x="653" y="116" text-anchor="middle" class="sk-sub">AI Search</text>
  <rect x="595" y="144" width="117" height="40" rx="8" class="sk-box-muted"/>
  <text x="653" y="168" text-anchor="middle" class="sk-sub">pgvector 등</text>
  <path d="M140,80 L188,64" class="sk-line" marker-end="url(#avs-m1)"/>
  <path d="M140,100 L188,126" class="sk-line" marker-end="url(#avs-m1)"/>
  <path d="M382,63 L428,90" class="sk-line" marker-end="url(#avs-m1)"/>
  <path d="M382,127 L428,115" class="sk-line" marker-end="url(#avs-m1)"/>
  <path d="M562,90 L593,64" class="sk-line" marker-end="url(#avs-m1)"/>
  <path d="M562,102 L593,112" class="sk-line" marker-end="url(#avs-m1)"/>
  <path d="M562,116 L593,160" class="sk-line" marker-end="url(#avs-m1)"/>
</svg>
<figcaption>에이전트가 보는 것은 도구뿐이고, 저장소는 컬렉션 API 뒤에서 바꿔 끼울 수 있다.</figcaption>
</figure>

## 내가 보는 의미

앞서 [벡터 DB 없이 RAG — SharePoint Embedded Retrieval API](/posts/spe-retrieval-api-mcp-no-vector-db/) 글은 "인덱싱을 서비스에 맡기는" 쪽이었습니다. 이번 글은 정반대로 **내가 컬렉션과 임베딩을 직접 쥐는** 쪽입니다. 둘은 경쟁이라기보다 선택 기준이 다릅니다. 데이터가 이미 SharePoint에 있고 권한을 그대로 따라가야 하면 앞쪽, 데이터 모델·필터·임베딩을 직접 통제해야 하면 이쪽입니다.

흥미로운 부분은 `Param` 필터입니다. "category는 engineering, operations, product 중 하나"라는 제약이 도구 스키마의 enum으로 들어가므로, 모델이 임의의 문자열로 검색하는 일을 스키마 단계에서 줄일 수 있습니다. 다만 이것이 검색 품질이나 권한 통제를 대신해 주지는 않습니다. 필터는 어디까지나 모델에게 주는 힌트이고, 접근 제어는 저장소 쪽에서 따로 걸어야 합니다. 이 부분은 원문이 다루지 않은 제 해석입니다.

## 실무에서 달라지는 점

1. **Semantic Kernel 벡터 스토어를 쓰던 개발자**: 개념은 그대로 이어지지만 API가 정리·변경되었다고 원문이 밝힙니다. 이전을 계획한다면 모델 등록과 배치 API부터 대조해 보세요. 이전 시점은 experimental 딱지가 떨어진 뒤로 잡는 편이 안전합니다.
2. **사내 PoC를 만드는 운영자**: 인메모리 컬렉션은 재시작하면 사라지고, 예제 임베딩은 OpenAI `text-embedding-3-small`이라 API 키가 필요합니다. 사내 보안 정책상 외부 키를 못 쓴다면 임베딩 제공자를 어디로 할지부터 정해야 합니다. 임베딩 호출 비용은 업서트와 검색 양쪽에서 발생합니다.
3. **저장소 선정**: 지원 목록이 넓어도 인덱스·필터·검색 방식은 저장소마다 다르다고 원문이 못 박았습니다. 이미 운영 중인 Azure AI Search, PostgreSQL, SQL Server가 있다면 새 DB를 들이기 전에 그쪽 패키지가 필요한 필터를 지원하는지 먼저 확인하세요.
4. **긴 대화 에이전트의 컨텍스트 관리**: `VectorStoreHistoryProvider`는 토큰을 줄이는 도구이면서 대화 전체를 저장소에 남기는 도구이기도 합니다. 대화에 개인정보·고객 정보가 섞일 수 있으니 보관 기간, 삭제 방법, 접근 권한을 도입 전에 정해야 합니다. 요약 압축이 어떤 정보를 버리는지도 확인이 필요합니다.
5. **교육할 때 강조할 점**: "벡터 검색 = RAG"가 아니라 "검색을 도구로 노출하는 설계"라는 틀로 설명하는 편이 좋습니다. 도구 이름·설명·결과 모양(`result_mapper`)이 모델의 호출 품질을 좌우하므로, 수강생이 검색 쿼리보다 도구 설계에 시간을 쓰게 하는 것이 낫겠습니다.

## 아직 확인되지 않은 것

원문에는 성능 수치가 없습니다. 저장소별 지연, 대용량에서의 동작, 압축된 작업 집합의 품질은 알 수 없고, 제가 직접 돌려 본 것도 아닙니다. 샘플은 `microsoft/agent-framework` 저장소의 `python/samples/02-agents/vector_stores`에 있으니, 도입 전에는 거기서 필터와 기록 검색 동작을 자기 데이터로 확인하는 것이 순서입니다. 에이전트 쓰기 도구(upsert, delete)를 붙일 때의 승인 설계는 [Durable Task 승인 설계 글](/posts/durable-task-real-world-boundaries/)과 함께 보면 도움이 됩니다.

## 영상으로 보기

기초부터 짚고 싶다면 이 영상을 먼저 보세요. (AI 에이전트 기초 EP.03 — LLM은 어떻게 말을 할까)

{% include embed/youtube.html id='Ay9-NSw1wYI' %}

> 📺 [Peanut Radio AI 유튜브 채널](https://www.youtube.com/@PeanutRadioAI) — AI 시대의 IT, 기초부터 실무까지
{: .prompt-tip }

## 참고

- [Vector Stores in Python Agent Framework: familiar foundations, streamlined for agents — Microsoft Agent Framework 블로그](https://devblogs.microsoft.com/agent-framework/vector-stores-in-python-agent-framework-familiar-foundations-streamlined-for-agents/)
- [벡터 DB 없이 RAG — SharePoint Embedded Retrieval API와 spe-mcp](/posts/spe-retrieval-api-mcp-no-vector-db/)
- [Durable Task가 해 주는 일, 해 주지 않는 일](/posts/durable-task-real-world-boundaries/)
