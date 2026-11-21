---
title: "Azure SQL로 RAG 청킹하기 — 설정 순서와 막히는 지점"
date: 2026-11-22 06:53:00 +0900
categories: [Azure AI]
tags: [azure, rag, llm, cost]
description: AI_GENERATE_CHUNKS로 문서를 자르고 임베딩·벡터 검색까지 가는 순서를 Learn 문서와 개발 블로그 기준으로 정리하고, 오류 번호별 해결을 묶었다.
---

> 이 글은 Microsoft Learn의 AI_GENERATE_CHUNKS·VECTOR_SEARCH 문서와 Azure SQL 개발 블로그(2026-11-20)를 읽고 정리·재구성한 것이다. 직접 실행한 기록이 아니라 문서 기준 순서다.
{: .prompt-info }

## 결론 3줄

- 청킹은 DB 호환성 수준 170 이상에서 `AI_GENERATE_CHUNKS`로 하고, 기준은 토큰이 아니라 **문자 수**다.
- 데이터가 적으면(후보 벡터 5만 개 미만이라는 일반 지침) 인덱스 없이 `VECTOR_DISTANCE` 정확 검색으로 충분하고, 벡터 인덱스는 NULL이 아닌 벡터가 100개 이상일 때만 만들 수 있다.
- 인덱스 검색은 `SELECT TOP (N) WITH APPROXIMATE` 형식을 쓴다. 옛 `TOP_N` 인자는 최신 인덱스에서 오류가 난다.

에이전트의 지식 계층을 DB 안에서 구성하는 흐름은 [에이전트 장기 기억 설계](/posts/agent-memory-facts-vs-understanding/)와 이어서 보면 위치를 잡기 쉽다.

## 단계별 순서

### 1. 호환성 수준 확인

먼저 현재 수준을 본다.

```sql
SELECT name, compatibility_level FROM sys.databases WHERE name = DB_NAME();
```

170 미만이면 Learn 문서 기준으로 데이터베이스 엔진이 함수를 찾지 못한다. 올리는 `ALTER DATABASE ... SET COMPATIBILITY_LEVEL = 170;`은 기존 DB에서는 테스트 후 소유자 승인을 받고 실행하라고 블로그가 강조한다. 빈 테스트 DB에서 시작하는 편이 안전하다.

### 2. 임베딩용 외부 모델 등록

순서는 마스터 키 → 데이터베이스 범위 자격 증명 → 외부 모델이다. 블로그 예시의 핵심 줄은 다음과 같다.

```sql
CREATE EXTERNAL MODEL MyEmbeddingModel
WITH (
  LOCATION = '<endpoint>/openai/deployments/<deployment>/embeddings?api-version=2024-06-01',
  API_FORMAT = 'Azure OpenAI',
  MODEL_TYPE = EMBEDDINGS,
  MODEL = 'text-embedding-3-small',
  CREDENTIAL = [https://<your-resource>.openai.azure.com/]
);
```

LOCATION의 배포 이름은 모델 이름과 다를 수 있다. 블로그의 자격 증명 예시는 API 키 방식인데, 운영에서는 관리 ID가 지원되는 곳에서는 그쪽을 쓰고 키는 Key Vault 같은 승인된 저장소에 두라고 한다.

### 3. 테이블 만들기

블로그 예시는 원문 테이블(`dbo.Documents`)과 청크 테이블(`dbo.DocumentChunks`)로 나눈다. 청크 테이블에는 청크 순서, 오프셋, 길이, 방식 이름과 `VECTOR(1536)` 열을 둔다. text-embedding-3-small이 1,536차원이기 때문이다. 원문 키(`source_key`)를 유니크로 두어서 출처 추적이 가능하게 한다.

### 4. 청킹 미리보기

```sql
SELECT c.chunk_order, c.chunk_offset, c.chunk_length, c.chunk
FROM dbo.Documents d
CROSS APPLY AI_GENERATE_CHUNKS(SOURCE = d.full_text, CHUNK_TYPE = FIXED, CHUNK_SIZE = 500) AS c;
```

Learn 문서 기준으로 `CHUNK_TYPE`은 FIXED만 받는다. `OVERLAP`은 0~50 정수 퍼센트이고 기본값은 0이다. `ENABLE_CHUNK_SET_ID = 1`을 주면 `chunk_set_id` 열이 추가된다.

### 5. 청킹과 임베딩을 한 번에 저장

```sql
INSERT INTO dbo.DocumentChunks (...)
SELECT ..., AI_GENERATE_EMBEDDINGS(c.chunk USE MODEL MyEmbeddingModel)
FROM dbo.Documents d
CROSS APPLY AI_GENERATE_CHUNKS(SOURCE = d.full_text, CHUNK_TYPE = FIXED, CHUNK_SIZE = 500, OVERLAP = 15) AS c;
```

방식 이름 열에 `fixed_500`, `fixed_500_overlap15`처럼 설정을 적어 두면 같은 문서를 여러 설정으로 저장해 비교할 수 있다. 이 호출은 임베딩 엔드포인트를 부르므로 쿼터와 비용이 든다.

### 6. 정확 검색

```sql
DECLARE @qv VECTOR(1536) = AI_GENERATE_EMBEDDINGS(@query USE MODEL MyEmbeddingModel);
SELECT TOP (2) dc.chunk_text, VECTOR_DISTANCE('cosine', @qv, dc.embedding) AS distance
FROM dbo.DocumentChunks dc
ORDER BY distance ASC;
```

### 7. 규모가 커지면 벡터 인덱스와 근사 검색

```sql
CREATE VECTOR INDEX IX_Embedding_DiskANN ON dbo.DocumentChunks(embedding)
WITH (METRIC = 'cosine', TYPE = 'diskann');

SELECT TOP (2) WITH APPROXIMATE t.chunk_text, s.distance
FROM VECTOR_SEARCH(TABLE = dbo.DocumentChunks AS t, COLUMN = embedding,
                   SIMILAR_TO = @qv, METRIC = 'cosine') AS s
ORDER BY s.distance ASC;
```

인덱스의 METRIC과 검색의 METRIC이 같아야 인덱스가 쓰인다. 다르면 경고와 함께 정확 검색으로 돌아간다. Learn 기준으로 VECTOR_SEARCH는 Azure SQL Database에서 GA다(SQL Server 2025는 프리뷰). AI_GENERATE_CHUNKS 문서 페이지에는 GA·프리뷰 표기가 없어서, 상태는 확인하지 못했다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="문서가 청킹, 임베딩, 저장, 검색으로 이어지는 흐름">
  <defs>
    <marker id="sqc-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① Azure SQL 안에서 끝나는 RAG 준비 흐름</text>
  <rect x="8" y="60" width="130" height="60" rx="8" class="sk-box"/>
  <text x="73" y="86" text-anchor="middle" class="sk-label">원문 테이블</text>
  <text x="73" y="106" text-anchor="middle" class="sk-sub">full_text</text>
  <rect x="170" y="60" width="140" height="60" rx="8" class="sk-box-accent"/>
  <text x="240" y="86" text-anchor="middle" class="sk-label">청킹</text>
  <text x="240" y="106" text-anchor="middle" class="sk-mono">AI_GENERATE_CHUNKS</text>
  <rect x="342" y="60" width="150" height="60" rx="8" class="sk-box"/>
  <text x="417" y="86" text-anchor="middle" class="sk-label">임베딩(외부 호출)</text>
  <text x="417" y="106" text-anchor="middle" class="sk-sub">비용·쿼터 발생</text>
  <rect x="524" y="60" width="188" height="60" rx="8" class="sk-box"/>
  <text x="618" y="86" text-anchor="middle" class="sk-label">VECTOR(1536) 저장</text>
  <text x="618" y="106" text-anchor="middle" class="sk-sub">정확 검색 → 필요 시 인덱스</text>
  <path d="M140,90 L166,90" class="sk-line" marker-end="url(#sqc-m1)"/>
  <path d="M312,90 L338,90" class="sk-line" marker-end="url(#sqc-m1)"/>
  <path d="M494,90 L520,90" class="sk-line" marker-end="url(#sqc-m1)"/>
  <text x="360" y="176" text-anchor="middle" class="sk-sub">호환성 수준 170 이상이 전제이고, 임베딩 단계만 DB 밖 엔드포인트를 부른다.</text>
</svg>
<figcaption>청킹은 DB 안에서 끝나지만, 비용이 드는 곳은 DB 밖의 임베딩 호출이다.</figcaption>
</figure>

## 판단표: 어떤 설정과 검색을 고르나

| 상황 | 선택 | 근거·주의 |
|---|---|---|
| 첫 테스트, 청크 수 적음 | 정확 검색(`VECTOR_DISTANCE`), 인덱스 없음 | 후보 5만 개 미만이면 정확 검색 권장(일반 지침) |
| 문장 중간 절단이 걱정됨 | `OVERLAP` 추가 | 15%면 청크가 약 17.6% 늘어남(블로그 계산). 품질이 자동으로 오르진 않음 |
| 청크 크기 정하기 | 500·1,000·2,000자 비교 | 문자 수 기준이라 토큰 수와 다름 |
| 벡터가 수만 개 이상 | `CREATE VECTOR INDEX` + `WITH APPROXIMATE` | 인덱스는 NULL 아닌 벡터 100개 이상 필요 |
| 계획이 정확 검색을 택함 | 실행 계획 확인 | APPROXIMATE가 인덱스를 강제하지 않음. FORCE_ANN_ONLY는 진단용 |

## 자주 막히는 지점

### 1. 함수를 찾지 못한다

호환성 수준이 170 미만이면 `AI_GENERATE_CHUNKS`를 못 찾는다. 1단계의 조회부터 다시 본다.

### 2. 인덱스 생성에서 Msg 42266

블로그에 따르면 NULL이 아닌 벡터가 100개 미만일 때 난다. 여섯 개짜리 예제로는 인덱스를 만들지 말고 정확 검색만 쓴다.

### 3. 검색에서 Msg 42274

옛 `TOP_N = k` 인자를 최신 버전 인덱스와 같이 쓰면 난다. Learn 문서가 지시하는 `SELECT TOP (N) WITH APPROXIMATE` 형식으로 바꾼다. 관련해서 Msg 42248(APPROXIMATE에 ORDER BY 없음), Msg 42271(ORDER BY가 distance ASC가 아님)도 문서에 있다.

### 4. CROSS APPLY와 APPROXIMATE의 충돌

Learn 기준으로 같은 FROM 절에서 CROSS APPLY·OUTER APPLY와 `TOP (N) WITH APPROXIMATE`를 함께 쓸 수 없다. 검색을 서브쿼리로 넣고 그것을 APPLY한다.

### 5. 다시 채우려고 TRUNCATE했더니 막힌다

벡터 인덱스가 있는 테이블에는 TRUNCATE가 안 된다. 문서의 우회법은 인덱스 삭제 → TRUNCATE → 100행 이상 재적재 → 인덱스 재생성이다.

### 6. 임베딩 비용과 키 관리

임베딩 호출은 Azure OpenAI 배포가 있어야 하고 쿼터를 쓴다. 블로그의 API 키 자격 증명 예시는 학습용으로만 보고, 운영에서는 관리 ID나 Key Vault를 쓴다.

## 도입 전 점검표

- 테스트 DB에서 호환성 수준을 올려도 되는가
- 임베딩 모델 배포와 쿼터가 준비됐는가
- 원문 수정 시 재청킹 규칙이 있는가(블로그는 원문이 키당 불변이라고 가정하고, 바뀌면 새 키로 다시 자르라고 한다)
- 청크 설정별 검색 결과를 비교할 질문 세트가 있는가

## 참고

- Microsoft Learn, AI_GENERATE_CHUNKS (Transact-SQL): <https://learn.microsoft.com/en-us/sql/t-sql/functions/ai-generate-chunks-transact-sql>
- Microsoft Learn, VECTOR_SEARCH (Transact-SQL): <https://learn.microsoft.com/en-us/sql/t-sql/functions/vector-search-transact-sql>
- Azure SQL Dev Blog, How to chunk documents in Azure SQL Database for RAG (2026-11-20): <https://devblogs.microsoft.com/azure-sql/how-to-chunk-documents-in-azure-sql-database-for-rag/>
- 이어 읽기: [에이전트 장기 기억 설계](/posts/agent-memory-facts-vs-understanding/)
