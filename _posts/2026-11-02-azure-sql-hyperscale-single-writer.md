---
title: "Azure SQL Hyperscale은 분산 SQL인가 — 쓰기 노드 하나의 의미"
date: 2026-11-02 06:53:00 +0900
categories: [IT News]
tags: [azure, cost, ai-agent]
description: Azure SQL Database Hyperscale의 구조를 쓰기 노드 하나라는 기준으로 읽고, 에이전트·AI 워크로드를 올리기 전 확인할 점을 정리했다.
---

> 이 글은 Microsoft Azure SQL 블로그의 "Is Azure SQL Database Hyperscale distributed SQL?"(Anna Hoffman, 2026-10-30)을 읽고 정리·재구성한 것이다. 수치는 모두 원문 기준이며 직접 실행해 확인한 값이 아니다.
{: .prompt-info }

## 결론부터

Hyperscale은 "분산 SQL이냐 아니냐"로 고르는 제품이 아니다. 원문의 표현대로 분산된 저장·로그 구조 위에 **쓰기를 받는 SQL 엔진이 하나**인 설계다. 그래서 선택 기준은 이름표가 아니라 **내 워크로드의 쓰기량이 노드 하나로 감당되는가**다. 읽기는 레플리카로 넓힐 수 있고, 쓰기는 넓힐 수 없다. 이 한 줄이 이 글의 전부이고, 나머지는 그 한 줄을 실무에 대입한 것이다.

## 원문이 말하는 구조

원문은 Hyperscale을 네 층으로 설명한다.

### 컴퓨트 레플리카

쿼리 처리기가 도는 곳이다. 쓰기를 받는 프라이머리 하나와 읽기용 레플리카들이 여기에 속한다.

### 로그 서비스

내구성을 책임지고 로그를 각 레플리카와 페이지 서버로 퍼뜨린다. 프라이머리가 커밋하면 로그가 여기에 먼저 남는다.

### 페이지 서버

각자 데이터의 한 조각을 맡는다. 데이터가 커지면 조각이 늘어나는 방식이다.

### Azure Storage

장기 보관본과 스냅샷 백업이 있는 곳이다.

<figure class="sketch">
<svg viewBox="0 0 720 280" role="img" aria-label="Hyperscale 구조: 쓰기 프라이머리 하나가 로그 서비스로 쓰고, 로그가 페이지 서버와 읽기 레플리카로 퍼지며, 페이지 서버 아래에 Azure Storage가 있다">
  <defs>
    <marker id="hs-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="hs-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 쓰기 경로는 하나, 읽기 경로는 여러 개</text>
  <rect x="8" y="50" width="170" height="56" rx="8" class="sk-box-accent"/>
  <text x="93" y="74" text-anchor="middle" class="sk-label">프라이머리 (쓰기)</text>
  <text x="93" y="92" text-anchor="middle" class="sk-sub">쓰기 노드는 항상 1개</text>
  <rect x="8" y="150" width="170" height="56" rx="8" class="sk-box"/>
  <text x="93" y="174" text-anchor="middle" class="sk-label">HA·네임드 레플리카</text>
  <text x="93" y="192" text-anchor="middle" class="sk-sub">읽기 전용, 크기 각각</text>
  <rect x="270" y="50" width="160" height="56" rx="8" class="sk-box"/>
  <text x="350" y="74" text-anchor="middle" class="sk-label">로그 서비스</text>
  <text x="350" y="92" text-anchor="middle" class="sk-sub">내구성 · 로그 분배</text>
  <rect x="270" y="150" width="160" height="56" rx="8" class="sk-box"/>
  <text x="350" y="174" text-anchor="middle" class="sk-label">페이지 서버</text>
  <text x="350" y="192" text-anchor="middle" class="sk-sub">데이터 조각별 담당</text>
  <rect x="520" y="150" width="190" height="56" rx="8" class="sk-box-muted"/>
  <text x="615" y="174" text-anchor="middle" class="sk-label">Azure Storage</text>
  <text x="615" y="192" text-anchor="middle" class="sk-sub">장기 보관 · 스냅샷 백업</text>
  <path d="M180,78 L268,78" class="sk-line-accent" marker-end="url(#hs-a1)"/>
  <path d="M350,108 L350,148" class="sk-line" marker-end="url(#hs-m1)"/>
  <path d="M432,178 L518,178" class="sk-line" marker-end="url(#hs-m1)"/>
  <path d="M268,185 L180,185" class="sk-line" marker-end="url(#hs-m1)"/>
  <text x="0" y="250" class="sk-sub">굵은 선: 쓰기 경로 / 가는 선: 로그·데이터 전파</text>
</svg>
<figcaption>분산되는 것은 저장과 읽기이고, 쓰기를 결정하는 지점은 하나다.</figcaption>
</figure>

원문은 쓰기 노드를 하나로 둔 것이 의도된 선택이라고 말한다. 쓰기 경로에 분산 커밋 프로토콜을 넣지 않고, 쓰기 충돌 해결과 다중 리전 쓰기 충돌을 피하려는 것이다. 대신 SQL Server의 트랜잭션 의미와 T-SQL 표면은 그대로여서 기존 앱을 고쳐 쓸 필요가 없다는 것이 원문의 주장이다.

## 원문의 숫자들

- 저장소는 최대 128TB까지 자동으로 늘어난다. 최소 크기는 10GB다.
- 로그 처리량은 최대 150MB/s, 컴퓨트는 Premium 시리즈에서 최대 192 vCore다.
- HA 레플리카는 최대 4개, 네임드 레플리카는 최대 30개이며 네임드 레플리카는 각자 다른 컴퓨트 크기를 가질 수 있다. 레플리카를 추가해도 데이터를 복사하지 않는다.
- 백업은 컴퓨트를 쓰지 않는 스냅샷이고, 복원 시간은 DB 크기와 거의 무관하다고 한다.
- 가격은 소프트웨어 라이선스와 I/O 요금이 따로 없는 단일 가격이다. 구체적인 금액은 원문에 없다.
- 서버리스의 자동 일시 중지·재개는 프리뷰다.
- T-SQL 기능 목록에 벡터가 들어 있지만 한도나 성능 설명은 없다.

벤치마크도 있다. Microsoft가 의뢰한 Principled Technologies 보고서에서 HammerDB TPROC-C, 2.3TB, 16 vCore 조건으로 Aurora PostgreSQL I/O-Optimized와 비교해 성능은 최대 52%, 가격 대비 성능은 최대 51% 앞섰다고 한다. 의뢰 사실은 글쓴이도 밝혔고, 실제 워크로드는 다르니 직접 테스트하라고 권한다. 나도 이 수치는 "이런 조건에서 이런 결과가 발표됐다" 정도로만 읽는다.

## 내 해석: 에이전트 시대에 쓰기 하나가 더 중요해진 이유

원문은 에이전트를 직접 다루지 않는다. 다만 네임드 레플리카를 "보고·분석·AI 워크로드 분리"에 쓰라는 대목은 에이전트 운영에 그대로 옮겨 볼 만하다. 이것은 원문이 아니라 내 추론이다.

사람이 쓰는 업무 앱은 쓰기가 완만하다. 반면 에이전트는 상태·메모리·툴 호출 기록을 자주 기록한다. 에이전트가 늘수록 읽기보다 **쓰기 빈도**가 먼저 병목이 될 수 있다. 쓰기 노드가 하나인 구조에서는 로그 처리량 150MB/s와 프라이머리 vCore 상한이 곧 천장이다. 반대로 에이전트가 만든 데이터를 조회·분석하는 일은 네임드 레플리카로 프라이머리에서 떼어낼 수 있다. 에이전트 기억을 어디에 둘지는 [에이전트 장기 기억 설계](/posts/agent-memory-facts-vs-understanding/) 글에서 다뤘고, 그 기억의 "사실" 쪽 저장소로 Hyperscale을 고려한다면 이 쓰기 천장이 첫 질문이 된다.

비용 쪽은 [Azure Maia 글](/posts/azure-maia-multi-gigawatt/)처럼 AI 비용이 인프라 전반으로 번지는 흐름과 이어진다. I/O 요금이 따로 없다는 점은 예측 가능성에 유리하지만, 단일 가격에 무엇이 포함되는지는 견적 단계에서 확인할 일이다.

## 실무에서 달라지는 점

### 쓰기 부하부터 잰다

도입을 검토하는 DBA와 아키텍트는 vCore 수보다 먼저 피크 시간의 초당 로그 생성량을 본다. 150MB/s가 상한이므로 배치 적재, 에이전트 상태 기록, 감사 로그 같은 쓰기 몰림이 한 DB에 겹치는지 확인해야 한다. 다중 쓰기·다중 리전 쓰기가 필요한 설계라면 처음부터 다른 선택지를 봐야 한다.

### 읽기 워크로드를 레플리카로 분리한다

보고서, BI, 에이전트의 검색성 조회는 네임드 레플리카로 보내는 설계를 먼저 그려 본다. 레플리카마다 컴퓨트 크기를 다르게 할 수 있으니 용도별로 크기를 나누는 것이 원문이 제시하는 방향이다. 다만 레플리카별 과금 방식과 읽기 지연은 원문에 없으므로 가격표와 문서에서 별도로 확인한다.

### 벤치마크 수치는 견적서에 쓰지 않는다

52%, 51%는 벤더 의뢰 결과이고 조건도 2.3TB, 16 vCore 하나다. 내부 보고서에 쓸 때는 의뢰 사실과 조건을 함께 적고, 실제 데이터로 PoC를 돌린 결과를 근거로 삼는다. 보고서 방법론이 공개돼 있다고 하니 우리 워크로드와 닮았는지부터 대조하는 것이 순서다.

### 프리뷰 기능은 운영에 넣지 않는다

서버리스 자동 일시 중지·재개는 프리뷰다. 개발·테스트 DB의 비용 절감에는 매력적이지만 운영 SLA 대상으로 가정하면 안 된다. 벡터 기능도 원문에는 이름만 있고 한도가 없으니, 벡터 검색을 쓸 계획이면 지원 범위와 성능을 문서에서 따로 확인한다.

### 교육할 때는 "분산"의 두 가지 뜻을 나눈다

IT 강사라면 "저장은 분산, 쓰기는 단일"이라는 구분을 강조할 만하다. 분산 SQL이라는 말을 듣고 모든 노드에 쓸 수 있다고 오해하는 경우가 많다. 이 구조를 그림으로 보여 주면 수평 확장과 쓰기 확장이 다른 문제라는 점을 설명하기 쉽다.

## 마치며

원문은 홍보 성격이 있는 제품 블로그다. 구조 설명은 유용하지만 한국 리전 정보, 에이전트 사례, 벡터 성능은 없다. 그래서 이 글의 결론도 "Hyperscale이 좋다"가 아니라 "쓰기 하나라는 전제를 알고 고르라"에 그친다.

## 참고

- [Is Azure SQL Database Hyperscale distributed SQL? — Azure SQL Dev Corner (Anna Hoffman, 2026-10-30)](https://devblogs.microsoft.com/azure-sql/is-azure-sql-database-hyperscale-distributed-sql/)
- 원문이 안내한 벤치마크 보고서: aka.ms/hyperscalebenchmark
- 원문이 안내한 시리즈: aka.ms/azuresqlfoundationseries
- 관련 글: [에이전트 장기 기억 설계](/posts/agent-memory-facts-vs-understanding/), [Azure Maia, 멀티 기가와트로](/posts/azure-maia-multi-gigawatt/)
