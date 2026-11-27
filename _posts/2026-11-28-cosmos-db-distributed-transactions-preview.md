---
title: "Cosmos DB 분산 트랜잭션 프리뷰 — saga 없이 파티션을 넘는 원자 쓰기"
date: 2026-11-28 06:51:00 +0900
categories: [IT News]
tags: [azure, ai-agent, devops]
description: Azure Cosmos DB for NoSQL의 분산 트랜잭션이 퍼블릭 프리뷰에 들어왔다. 조건·한도·오류 코드와 도입 전 확인할 점을 정리했다.
---

> 이 글은 Azure Cosmos DB 블로그의 「All-or-nothing distributed transactions in Azure Cosmos DB」(2026-11-24)를 읽고 정리·재구성한 것이다. 코드 샘플은 원문에서 확인한 내용이며, 직접 실행해 검증한 것이 아니다.
{: .prompt-info }

## 결론 먼저

- Cosmos DB for NoSQL이 여러 논리 파티션·컨테이너·데이터베이스에 걸친 쓰기를 하나의 원자 단위로 커밋하는 분산 트랜잭션을 퍼블릭 프리뷰로 내놨다.
- 지금은 .NET SDK 전용이고, 사용하려면 온보딩 양식을 내서 승인을 받아야 한다. GA가 아니고 SLA도 없다.
- 운영 워크로드에 바로 얹을 기능이 아니다. 지금 할 일은 "우리 시스템에서 saga·보상 로직으로 풀던 곳이 어디인가"를 목록으로 뽑아 두는 것이다.

## 무엇이 나왔나

원문의 예시는 계좌 이체다. Alice가 Bob에게 150달러를 보내면 쓰기가 세 번 일어난다. Alice 잔액 감소, Bob 잔액 증가, 원장(ledger) 기록 추가다. 샘플에서는 계좌 컨테이너와 월별 파티션 키를 쓰는 원장 컨테이너가 따로 있어서, 지금까지의 Cosmos DB로는 이 세 쓰기를 한 번에 묶을 수 없었다.

기존 도구는 두 가지였다. 하나는 transactional batch인데, 같은 컨테이너의 같은 파티션 키 안에서만 동작하고 작업 100개, 2MB, 5초 한도가 있다. 다른 하나는 saga 패턴이다. 단계마다 되돌리는 보상 작업을 앱이 직접 만들고 운영해야 한다.

이번 프리뷰는 서비스가 직접 two-phase commit을 돌린다. 참여 파티션에 먼저 준비(prepare)를 시키고, 모두 준비되면 커밋, 하나라도 실패하면 중단한다. 앱에 별도 코디네이터가 필요 없다는 게 원문의 설명이다.

<figure class="sketch">
<svg viewBox="0 0 720 280" role="img" aria-label="분산 트랜잭션에서 서비스가 준비 후 커밋하거나 중단하는 흐름">
  <defs>
    <marker id="cdt-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="cdt-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 분산 쓰기 트랜잭션의 흐름</text>
  <rect x="8" y="90" width="150" height="60" rx="8" class="sk-box"/>
  <text x="83" y="116" text-anchor="middle" class="sk-label">앱 (.NET)</text>
  <text x="83" y="134" text-anchor="middle" class="sk-sub">작업 최대 100개·2MB</text>
  <path d="M160,120 L210,120" class="sk-line" marker-end="url(#cdt-m1)"/>
  <rect x="212" y="80" width="170" height="80" rx="8" class="sk-box-accent"/>
  <text x="297" y="110" text-anchor="middle" class="sk-label">Cosmos DB 서비스</text>
  <text x="297" y="128" text-anchor="middle" class="sk-sub">two-phase commit 조율</text>
  <text x="297" y="144" text-anchor="middle" class="sk-sub">단일 쓰기 리전</text>
  <path d="M384,100 L450,60" class="sk-line-accent" marker-end="url(#cdt-a1)"/>
  <path d="M384,120 L450,120" class="sk-line-accent" marker-end="url(#cdt-a1)"/>
  <path d="M384,140 L450,190" class="sk-line-accent" marker-end="url(#cdt-a1)"/>
  <rect x="452" y="36" width="250" height="44" rx="8" class="sk-box"/>
  <text x="577" y="62" text-anchor="middle" class="sk-label">accounts / alice 잔액 −150</text>
  <rect x="452" y="98" width="250" height="44" rx="8" class="sk-box"/>
  <text x="577" y="124" text-anchor="middle" class="sk-label">accounts / bob 잔액 +150</text>
  <rect x="452" y="168" width="250" height="44" rx="8" class="sk-box"/>
  <text x="577" y="194" text-anchor="middle" class="sk-label">ledger / 2026-11 기록 추가</text>
  <text x="8" y="250" class="sk-sub">전부 준비되면 커밋, 하나라도 실패하면 전체 중단(452)</text>
</svg>
<figcaption>보상 로직을 앱이 아니라 서비스가 맡는다는 것이 이번 기능의 핵심이다.</figcaption>
</figure>

## 조건과 한도

원문 표에서 확인되는 조건을 모으면 다음과 같다.

| 항목 | 내용 |
|---|---|
| API | NoSQL만 |
| 처리량 | provisioned(수동·오토스케일) |
| 리전 | 단일 쓰기 리전 계정만. 멀티 쓰기는 계획 단계 |
| SDK | .NET v3 `Microsoft.Azure.Cosmos` 3.64.0-preview.0 이상. 다른 SDK는 "coming soon" |
| 신청 | 온보딩 양식, 보통 영업일 1~2일 |
| 한도 | 트랜잭션당 작업 100개, 2MB |
| 범위 | 한 계정·한 리전 안. 계정 두 개를 넘을 수 없음 |

진입점은 `CosmosClient.CreateDistributedWriteTransaction()`이다. CreateItem, UpsertItem, ReplaceItem, PatchItem, DeleteItem을 섞어 담을 수 있다. 읽기는 `CreateDistributedReadTransaction()`으로 ReadItem을 담으면 커밋된 하나의 스냅샷에서 값을 돌려준다.

인증은 Microsoft Entra ID 역할 기반이다. 샘플은 `Cosmos DB Built-in Data Contributor` 역할을 데이터베이스 범위로 부여한다. 참여하는 모든 컨테이너에 데이터 평면 권한이 있어야 한다는 점은 원문이 따로 짚는다.

## 동시성 제어와 오류 코드

샘플의 이체는 Alice 쪽 PatchItem에 `IfMatchEtag`와 `FilterPredicate`(잔액이 이체액 이상일 것)를 붙인다. 잔액 증감 자체는 서버에서 Increment로 처리한다. 누가 중간에 Alice 잔액을 바꾸면 ETag가 달라져 해당 작업이 412 Precondition Failed가 되고, 전체 트랜잭션은 452(aborted)로 끝난다. 이 경우 Bob은 입금되지 않고 원장도 남지 않는다. 원문의 지침은 충돌이 확인되면 낡은 제안을 재전송하지 말고 최신 상태를 읽어 다시 계산하라는 것이다.

SDK 옵션으로는 `MaxRetryAttemptsOnAbortedTransactions`(샘플 10)와 `MaxRetryWaitTimeOnAbortedTransactions`(샘플 30초)가 있다. 429 제한은 별도의 rate-limiting 재시도 설정이 처리한다. 응답 모드도 둘이다. Standard는 최종 결과가 정해진 뒤 반환하고, FastResponse는 먼저 반환하면서 서비스가 뒤에서 계속 처리한다. 원문은 둘의 원자성·내구성이 같다고 하고, FastResponse는 즉시 read-after-write가 필요 없는 쓰기 위주 워크로드용이라고 한다.

## 실무에서 달라지는 점

### 1. 지금은 PoC 전용으로 보는 게 맞다

프리뷰는 GA 전에 동작과 한도가 바뀔 수 있다고 원문이 명시한다. SLA도 없다. 운영 DB에 붙이기보다 별도 계정에서 이체·주문-재고 같은 대표 시나리오 하나만 검증하는 용도로 두는 편이 안전하다. 온보딩 신청부터 승인까지 영업일 기준 1~2일이 걸리므로 일정에 미리 넣어 두면 된다.

### 2. 멀티 리전 계정이라면 적용 범위를 먼저 확인한다

이 기능은 단일 쓰기 리전 계정에서만 쓸 수 있다. 멀티 쓰기 리전 계정을 운영하는 팀은 지금은 대상이 아니다. 단일 쓰기에 읽기 복제만 있는 구성이라도 주의할 점이 있다. 원자성은 쓰기 리전 안에서만 보장되고 복제는 비동기라서, 보조 리전 독자가 참여 파티션 사이에서 서로 다른 버전을 잠시 볼 수 있다. 읽기를 보조 리전으로 돌리는 설계라면 화면에 보이는 합계가 순간적으로 어긋날 수 있다는 점을 전제해야 한다.

### 3. 파티션 키 설계가 여전히 우선이다

같은 파티션 키 안에서 끝나는 작업은 transactional batch가 계속 맞는 도구다. 원문도 데이터가 한 파티션 키에 머물면 batch가 가장 적합하다고 한다. 분산 트랜잭션은 two-phase commit을 거치므로 단순 batch보다 비용과 지연이 클 것이라고 추정되지만, 원문에는 수치가 없다. 도입 전에 부하 시험으로 직접 재야 하는 항목이다. 파티션 키를 잘 잡아 batch로 풀리는 곳은 그대로 두고, 정말 경계를 넘는 곳만 후보로 올린다.

### 4. 에이전트 상태 저장소에는 해석이 필요하다

원문은 에이전트를 다루지 않는다. 여기서부터는 이 블로그의 해석이다. 에이전트가 사용자 프로필, 작업 큐, 감사 로그를 여러 컨테이너에 나눠 쓰는 구조라면 실패 중간 상태가 곧 데이터 오염이다. 지금까지는 보상 로직이나 멱등 키로 막았을 텐데, 이 기능이 GA되면 설계가 단순해질 여지가 있다. 에이전트 쪽 격리 이야기는 [에이전트 코드 실행 샌드박스, ACA 세션 풀](/posts/aca-dynamic-sessions-agent-sandbox-guide/) 글에서 이어 볼 수 있다. 다만 지금은 .NET 외 SDK가 없어서, Python이나 TypeScript로 에이전트를 만드는 팀은 당장 쓸 수 없다.

### 5. 교육할 때는 "ACID가 공짜로 오지 않는다"를 같이 가르친다

수강생이 가장 많이 오해하는 지점은 "분산 트랜잭션이 있으면 낙관적 동시성은 필요 없다"이다. 샘플은 오히려 ETag와 FilterPredicate를 함께 쓴다. 트랜잭션이 원자성을 주더라도 어떤 값을 근거로 쓰는지는 앱이 책임진다. 412와 452를 받았을 때 읽기-재계산-재시도 순서로 처리하는 코드까지 포함해 보여 주면 교육 효과가 크다.

## 아직 확인되지 않은 것

원문에는 트랜잭션 타임아웃, RU 소비 방식, .NET 외 SDK의 일정이 없다. Learn 문서와 샘플 저장소(`AzureCosmosDB/distributed-transactions-sample`)에 더 자세한 내용이 있을 수 있지만 이 글은 거기까지 검증하지 못했다. 비용이나 지연 수치가 필요하면 온보딩 후 직접 측정해야 한다.

## 참고

- [All-or-nothing distributed transactions in Azure Cosmos DB (Azure Cosmos DB Blog, 2026-11-24)](https://devblogs.microsoft.com/cosmosdb/all-or-nothing-distributed-transactions-in-azure-cosmos-db/)
- [분산 트랜잭션 구성·사용 가이드 (Microsoft Learn)](https://learn.microsoft.com/azure/cosmos-db/how-to-configure-and-use-distributed-transactions)
- [Transactional batch (Microsoft Learn)](https://learn.microsoft.com/azure/cosmos-db/transactional-batch)
- [낙관적 동시성과 단일 파티션 ACID (Microsoft Learn)](https://learn.microsoft.com/azure/cosmos-db/database-transactions-optimistic-concurrency)
- [부분 문서 업데이트와 FilterPredicate (Microsoft Learn)](https://learn.microsoft.com/azure/cosmos-db/partial-document-update)
- [샘플 저장소 (GitHub)](https://github.com/AzureCosmosDB/distributed-transactions-sample)
- [프리뷰 온보딩 양식](https://aka.ms/cosmosdb/dtx-onboard)
