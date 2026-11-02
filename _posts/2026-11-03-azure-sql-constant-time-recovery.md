---
title: "Azure SQL 상수 시간 복구(ADR) — 12시간 롤백이 3분이 된 구조"
date: 2026-11-03 06:53:00 +0900
categories: [IT News]
tags: [azure, devops]
description: 대용량 트랜잭션 롤백이 작업량에 비례해 길어지던 문제를 Accelerated Database Recovery가 어떻게 풀었는지, 운영자가 확인할 설정과 함께 정리했다.
---

> 이 글은 Microsoft Azure SQL 블로그의 "Under the Hood" 시리즈 2편(Anna Hoffman, 2026-11-02)을 읽고 정리·재구성한 것입니다. 수치는 모두 원문이 인용한 2019년 VLDB 논문 값이며, 직접 측정한 결과가 아닙니다.
{: .prompt-info }

## 결론부터

Azure SQL이 쓰는 상수 시간 복구(Constant Time Recovery, 제품 이름은 Accelerated Database Recovery, ADR)는 "복구 시간이 작업량에 비례한다"는 전통적 구조를 깨는 기능입니다. 핵심은 되돌리기(undo)를 로그를 거꾸로 돌며 행마다 수행하지 않고, 행 버전을 DB 안에 남겨 두고 "이 트랜잭션은 취소됨"이라는 표시만 하는 것입니다.

운영자 입장에서 정리하면 이렇습니다.

- Azure SQL Database는 이미 항상 켜져 있고 끌 수 없으므로 할 일이 없습니다.
- SQL Server 2019 이상은 지원하지만 기본값이 꺼짐이라, 대용량 배치가 있는 인스턴스는 켤지 판단해야 합니다.
- 공짜는 아닙니다. 논문 기준 TPC-C에서 처리량이 13.8% 줄었습니다.

이 글은 [Hyperscale 글](/posts/azure-sql-hyperscale-single-writer/)과 같은 시리즈의 연속입니다. 그 글이 "쓰기 노드가 하나"라는 구조를 다뤘다면, 이번 글은 그 쓰기 노드가 죽었다 살아날 때 얼마나 걸리는가를 다룹니다.

## 문제: 롤백 시간은 작업량에 비례한다

원문의 사례는 한 트랜잭션으로 수억 행을 적재하던 중 무관한 재시작이 끼어든 경우입니다. DB는 테이블을 다시 열기 전에 그 변경을 한 행씩 되돌려야 했고, 논문에 따르면 복구에 12시간이 걸리는 동안 해당 테이블은 완전히 접근 불가였습니다.

원인은 1990년대 초부터 쓰인 ARIES 복구 알고리즘(분석·재실행·되돌리기 3단계)의 되돌리기 단계입니다. 미커밋 트랜잭션이 한 일을 전부 취소해야 하니, 큰 트랜잭션일수록 복구가 길어집니다. 장애와 유지보수가 잦은 클라우드에서는 이것이 곧 가용성 문제가 됩니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="전통적 복구와 ADR 복구의 되돌리기 방식 비교">
  <defs>
    <marker id="ctr-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ctr-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 되돌리기 방식 비교</text>
  <text x="8" y="50" class="sk-label">전통적 ARIES</text>
  <rect x="8" y="64" width="150" height="50" rx="8" class="sk-box-muted"/>
  <text x="83" y="86" text-anchor="middle" class="sk-label">장애·재시작</text>
  <text x="83" y="104" text-anchor="middle" class="sk-sub">미커밋 트랜잭션 발견</text>
  <path d="M160,89 L240,89" class="sk-line" marker-end="url(#ctr-m1)"/>
  <rect x="244" y="64" width="250" height="50" rx="8" class="sk-box-muted"/>
  <text x="369" y="86" text-anchor="middle" class="sk-label">모든 변경을 행마다 되돌림</text>
  <text x="369" y="104" text-anchor="middle" class="sk-sub">시간 = 작업량에 비례</text>
  <path d="M496,89 L546,89" class="sk-line" marker-end="url(#ctr-m1)"/>
  <rect x="550" y="64" width="160" height="50" rx="8" class="sk-box-muted"/>
  <text x="630" y="94" text-anchor="middle" class="sk-label">그 뒤에야 사용 가능</text>
  <text x="8" y="154" class="sk-label">ADR (CTR)</text>
  <rect x="8" y="168" width="150" height="50" rx="8" class="sk-box"/>
  <text x="83" y="190" text-anchor="middle" class="sk-label">장애·재시작</text>
  <text x="83" y="208" text-anchor="middle" class="sk-sub">미커밋 트랜잭션 발견</text>
  <path d="M160,193 L240,193" class="sk-line-accent" marker-end="url(#ctr-m2)"/>
  <rect x="244" y="168" width="250" height="50" rx="8" class="sk-box-accent"/>
  <text x="369" y="190" text-anchor="middle" class="sk-label">트랜잭션 ID를 '취소됨'으로 표시</text>
  <text x="369" y="208" text-anchor="middle" class="sk-sub">읽기는 이전 버전을 선택</text>
  <path d="M496,193 L546,193" class="sk-line-accent" marker-end="url(#ctr-m2)"/>
  <rect x="550" y="168" width="160" height="50" rx="8" class="sk-box-accent"/>
  <text x="630" y="190" text-anchor="middle" class="sk-label">즉시 사용 가능</text>
  <text x="630" y="208" text-anchor="middle" class="sk-sub">정리는 백그라운드</text>
</svg>
<figcaption>되돌리기를 "작업"에서 "표시"로 바꾸면 복구 시간이 작업량과 분리된다.</figcaption>
</figure>

## 구조: 세 가지 부품

### 영속 버전 저장소(PVS)

스냅샷 격리용 행 버전은 원래 tempdb에 있어 재시작하면 사라졌습니다. ADR은 이 버전을 DB 안에 영속적으로 보관합니다. 작은 변경은 현재 행 옆에 바이트 차이(diff)로, 큰 버전은 별도 테이블에 둡니다.

### 논리적 되돌리기

트랜잭션이 취소되면 그 ID를 메모리의 취소 트랜잭션 맵에 넣습니다. 읽기가 그 트랜잭션이 만든 버전을 만나면 건너뛰고 이전 버전을 읽습니다. 잠금은 바로 풀리고, 실제 정리는 백그라운드가 천천히 합니다.

### 보조 로그(SLog)

버전으로 표현할 수 없는 작업(일부 메타데이터 변경, 잠금 획득 등)은 작은 별도 로그에 기록합니다. 양이 적어서 상수 시간에 재실행·되돌리기가 가능합니다.

저는 이 설계에서 "ARIES를 버리지 않고 버전 관리를 얹었다"는 점이 인상적이었습니다. 스토리지 엔진을 통째로 다시 만들지 않고도 가장 아픈 한 단계만 도려낸 것입니다.

## 수치 읽기 (2019년 논문 기준)

| 항목 | 전통 방식 | CTR |
|---|---|---|
| 복구 시간 99.99 백분위 | 10분 초과 | 1.3분 |
| 복구 시간 99.999 백분위 | 1시간 초과 | 3분 |
| 1천만 건 삽입 로그 | 5,101MB | 99MB |
| 5천만 건 삭제 로그 | 38,255MB | 147MB |

평균이 아니라 꼬리(99.99, 99.999 백분위)가 크게 줄었다는 점을 봐야 합니다. 99.9 백분위는 68초에서 60초로 거의 같습니다. 즉 평소 복구는 원래도 빨랐고, ADR은 "아주 드물게 터지는 긴 복구"를 없앱니다. 반대 비용으로 TPC-C(갱신 중심)는 13.8%, TPC-E(혼합)는 2.4% 처리량이 줄었습니다. 이 값은 2019년 시점이고 이후 SQL Server 2022·2025의 개선은 별도 수치가 공개되지 않았습니다.

## 실무에서 달라지는 점

### Azure SQL Database·Managed Instance 운영자

설정할 것은 없습니다. 다만 "대용량 배치 중 장애가 나도 몇 시간씩 테이블이 잠긴다"는 오래된 걱정은 근거가 약해졌습니다. 야간 적재를 잘게 쪼개라는 기존 가이드는 이제 복구 시간보다 로그·잠금 관리 목적으로 설명하는 편이 정확합니다. 원문은 Managed Instance를 별도로 언급하지 않으므로, 적용 여부는 Learn 문서에서 확인해야 합니다.

### 사내 SQL Server 2019 이상 운영자

기본값이 꺼짐이라 켜야만 효과가 있습니다. 켜기 전에 확인할 것은 세 가지입니다. 현재 빌드가 최신 누적 업데이트인지(초기 구현은 버전 정리 스레드가 하나라 바쁜 시스템에서 저장소가 커질 수 있었고, 2022에서 다중 스레드 정리로 개선됐다고 합니다), 갱신이 많은 워크로드의 처리량 영향, 데이터 파일에서 PVS가 차지할 공간입니다. 원문도 바쁜 DB에 켜기 전 최신 버전을 쓰라고 권합니다.

### 모니터링 담당자

PVS 크기가 새 감시 항목이 됩니다. 오래 열린 트랜잭션이 있으면 정리가 막혀 저장소가 커질 수 있다는 점은 제 추론입니다. 원문이 직접 다룬 내용은 아닙니다. 임계치 알림을 PVS 크기에도 걸어 두는 것을 검토할 만합니다.

### SQL Server 2025 도입 검토자

2025는 ADR을 tempdb까지 확장해 임시 테이블이 낀 대형 롤백도 빨라진다고 합니다. 또한 원문에 따르면 최적화된 잠금(optimized locking)은 PVS가 전제 조건입니다. 그 기능을 쓸 계획이면 ADR 활성화가 사실상 선행 작업이라는 뜻입니다. 이 부분은 후속 글에서 다룬다고 하니 나오면 이어서 정리하겠습니다.

### 교육할 때 강조할 점

"롤백은 로그를 거꾸로 되감는 것"이라는 모델을 가르쳐 왔다면 ADR 환경에서는 설명이 달라집니다. 롤백이 즉시 끝나 보여도 정리 작업은 뒤에서 계속된다는 것, 그리고 같은 DB라도 설정에 따라 복구 시간의 성격이 다르다는 것을 짚어 주는 것이 좋습니다.

## 참고

- 원문: [Anna Hoffman, "How Microsoft SQL learned to recover without undoing everything first" (Under the Hood, part 2)](https://devblogs.microsoft.com/azure-sql/your-server-wasnt-slow-it-was-waiting-we-fixed-it-with-constant-time-recovery/)
- 논문: [Constant Time Recovery in Azure SQL Database, VLDB 2019](https://www.vldb.org/pvldb/vol12/p2143-antonopoulos.pdf)
- 문서: [Accelerated Database Recovery 개념](https://learn.microsoft.com/sql/relational-databases/accelerated-database-recovery-concepts)
- 문서: [Accelerated Database Recovery 관리·활성화](https://learn.microsoft.com/sql/relational-databases/accelerated-database-recovery-management)
- 이전 글: [Azure SQL Hyperscale은 분산 SQL인가](/posts/azure-sql-hyperscale-single-writer/)
