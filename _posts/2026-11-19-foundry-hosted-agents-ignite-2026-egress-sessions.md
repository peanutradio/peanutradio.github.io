---
title: "Foundry 호스티드 에이전트 — 동시 세션 6만, 이그레스 통제, 상태 구분"
date: 2026-11-19 06:53:00 +0900
categories: [IT News]
tags: [foundry, ai-agent, governance, security, azure]
description: Foundry Agent Service 호스티드 에이전트의 Ignite 2026 업데이트를 GA와 프리뷰로 갈라, 서브넷 IP·이그레스 통제·유휴 타임아웃의 운영 영향을 정리한다.
---

> 이 글은 Microsoft Foundry 블로그의 "Hosted agents in Foundry Agent Service at Ignite 2026" (2026-11-18) 발표를 읽고 정리·재구성한 것이다. 수치와 상태 표기는 원문 기준이며, 직접 실행해 확인한 내용이 아니다.
{: .prompt-info }

## 결론부터

이번 발표에서 운영팀이 바로 체감할 항목은 셋이다. 첫째, 서브넷 IP 공유 방식이 바뀌어 동시 세션 한도의 병목이 서브넷에서 구독 할당량으로 옮겨 갔다(GA). 둘째, 에이전트가 밖으로 나가는 호출을 Audit로 먼저 관찰하고 Enforce로 전환하는 이그레스 통제가 GA가 됐다. 셋째, 유휴 타임아웃을 에이전트 버전마다 조절할 수 있게 됐다(GA). 반면 사설 CA 신뢰, Azure Policy 거버넌스, 메모리 스냅샷, A2A passthrough는 아직 퍼블릭 프리뷰이고, 자체 AKS 배포는 "12월 초 프리뷰 목표"로 아직 나오지 않았다.

## 한눈에 보는 상태

원문의 표를 이 블로그 방식으로 다시 묶으면 다음과 같다.

- GA: 공유 서브넷 IP, 네트워크 이그레스 통제, 장기 실행 복구, 유휴 타임아웃 설정
- 퍼블릭 프리뷰: 사설 CA 신뢰, 이그레스용 Azure Policy 거버넌스, 메모리 스냅샷, A2A passthrough
- 예정: 자체 AKS 배포(프리뷰 12월 초 목표), Temporal 연동(프리뷰 2027년 초 계획)

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="호스티드 에이전트 업데이트를 GA, 프리뷰, 예정 세 칸으로 나눈 그림">
  <defs>
    <marker id="hag-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 호스티드 에이전트 업데이트 — 상태별 분류</text>
  <rect x="8" y="34" width="220" height="160" rx="8" class="sk-box-accent"/>
  <text x="118" y="58" text-anchor="middle" class="sk-label">GA</text>
  <text x="118" y="88" text-anchor="middle" class="sk-sub">공유 서브넷 IP</text>
  <text x="118" y="110" text-anchor="middle" class="sk-sub">이그레스 통제(Audit/Enforce)</text>
  <text x="118" y="132" text-anchor="middle" class="sk-sub">장기 실행 복구</text>
  <text x="118" y="154" text-anchor="middle" class="sk-sub">유휴 타임아웃 2분~4시간</text>
  <path d="M232,114 L252,114" class="sk-line" marker-end="url(#hag-m1)"/>
  <rect x="256" y="34" width="220" height="160" rx="8" class="sk-box"/>
  <text x="366" y="58" text-anchor="middle" class="sk-label">퍼블릭 프리뷰</text>
  <text x="366" y="88" text-anchor="middle" class="sk-sub">사설 CA 신뢰</text>
  <text x="366" y="110" text-anchor="middle" class="sk-sub">Azure Policy 이그레스 거버넌스</text>
  <text x="366" y="132" text-anchor="middle" class="sk-sub">메모리 스냅샷</text>
  <text x="366" y="154" text-anchor="middle" class="sk-sub">A2A passthrough</text>
  <path d="M480,114 L500,114" class="sk-line" marker-end="url(#hag-m1)"/>
  <rect x="504" y="34" width="208" height="160" rx="8" class="sk-box-muted"/>
  <text x="608" y="58" text-anchor="middle" class="sk-label">아직 없음</text>
  <text x="608" y="88" text-anchor="middle" class="sk-sub">자체 AKS 배포</text>
  <text x="608" y="108" text-anchor="middle" class="sk-sub">(프리뷰 12월 초 목표)</text>
  <text x="608" y="136" text-anchor="middle" class="sk-sub">Temporal 연동</text>
  <text x="608" y="156" text-anchor="middle" class="sk-sub">(프리뷰 2027년 초 계획)</text>
</svg>
<figcaption>도입 판단은 제목이 아니라 이 세 칸 중 어디에 속하느냐로 갈린다.</figcaption>
</figure>

## 원문에서 눈여겨볼 세부

### 1. 서브넷 IP 공유 — 병목이 옮겨 간다

원문에 따르면 예전에는 세션마다 위임된 서브넷에서 IP를 하나씩 받아 /24가 동시 세션 약 250개에서 막혔다. 이제는 계획 추정치로 /24에서 약 60,000개, 지원되는 가장 작은 /27에서 약 7,500개 동시 세션이다. 각 세션은 여전히 VM 격리 샌드박스에서 돈다. 원문이 직접 "보장이 아닌 계획 추정치"라고 못 박았고, 운영에는 /24 이상을 권장한다.

그러면 실제 상한은 무엇인가. 기본 할당량이 구독·리전당 동시 세션 2,000개 또는 1,000개(리전에 따라 다름)다. 어느 리전이 어느 쪽인지는 원문에 없다. 더 필요하면 지원 요청을 해야 한다.

### 2. 이그레스 통제 — Audit부터 시작하는 설계

에이전트별로 순서가 있는 규칙을 둔다. 호스트 허용·거부, 나가는 요청에 헤더 추가, 자체 게이트웨이 경유가 가능하다. 평가는 에이전트 샌드박스 밖에서 일어나고, 정책을 평가하지 못하면 요청은 거부된다. Azure Firewall 같은 기존 네트워크 통제를 대체하는 것이 아니라 보완이다.

Audit 모드는 기본 거부 규칙에 걸릴 요청을 로그로 남기되 통과시킨다. 다만 헤더 변경과 재작성은 Audit에서도 적용되므로 비운영 환경에서 먼저 확인하라고 원문이 경고한다. Enforce로 바꾼 뒤에는 새 에이전트 버전을 배포해야 한다. 실행 중인 세션은 시작할 때의 정책을 유지하기 때문이다.

### 3. 유휴 타임아웃과 복구 — 비용과 상태의 문제

유휴 타임아웃은 버전별로 2분에서 4시간, 기본 15분이다. 메모리 스냅샷을 쓰지 않으면 세션 파일은 유휴 구간에도 남고, 30일 비활성이면 세션이 삭제된다. 장기 실행 복구는 옵트인이며, 복구는 핸들러를 다시 실행하므로 이메일 발송 같은 동작이 중복되지 않게 직접 막아야 한다고 원문이 적었다.

### 4. 프리뷰 항목의 단서

Azure Policy 거버넌스는 Canada Central, Spain Central, Korea Central에서 시작한다. 사설 CA 신뢰는 원격 빌드와 mTLS를 아직 다루지 않는다. 메모리 스냅샷은 실행 중이던 세션이 아니라 준비 완료 시점의 상태를 복원하며, 스냅샷 세션에서는 $HOME을 포함한 메모리와 로컬 파일이 유휴 구간을 넘기지 못한다. A2A passthrough는 A2A v1.0, JSON-RPC만 되고 스트리밍과 푸시 알림은 아직 없다.

## 실무에서 달라지는 점

### 1. 네트워크 담당자: 서브넷 설계를 다시 본다

이전에 "세션 수만큼 IP가 필요하다"는 전제로 큰 서브넷을 잡았다면 그 전제는 바뀌었다. 다만 /24 이상 권장은 그대로이므로 줄이기보다 기존 설계를 유지하고, 한도 계산의 기준을 서브넷에서 리전 할당량으로 옮기면 된다. 도입 전에 쓰는 리전의 기본 할당량이 1,000인지 2,000인지는 원문으로 알 수 없으니 구독에서 직접 확인해야 한다.

### 2. 보안 담당자: Audit 기간을 일정에 넣는다

이그레스 통제는 켜자마자 차단하는 기능으로 쓰면 안 된다. Audit에서 에이전트가 실제로 어디를 호출하는지 로그를 모으고, 허용 목록을 만든 뒤 Enforce로 가는 순서다. Enforce 전환 후 새 버전 배포가 필요하다는 점을 변경 절차서에 적어 둔다. 오래 도는 세션은 옛 정책을 계속 쓴다는 뜻이기도 하다.

### 3. 거버넌스 담당자: 한국 리전은 프리뷰 시작 리전이다

Korea Central이 Azure Policy 거버넌스 프리뷰 첫 리전에 들어 있다. 구독이나 리소스 그룹 범위로 Audit를 먼저 걸어 위반 현황을 보고 Deny로 올리는 방식이라, 파일럿 구독 하나로 시험해 볼 만하다. 단 프리뷰이므로 운영 전체에 의무 정책으로 거는 것은 이르다. 이미 실행 중인 세션은 영향을 받지 않는다는 점도 기억해야 한다.

### 4. 에이전트 개발자: 상태를 밖으로 뺀다

복구, 스냅샷, 유휴 삭제 모두 "프로세스 안의 상태는 믿지 말라"로 모인다. 진행 상황은 외부 저장소에 저장하고, 부작용이 있는 동작에는 중복 방지 키를 두어야 한다. 메모리 스냅샷은 콜드 스타트를 줄여 주지만 유휴 후 로컬 파일이 사라지는 대가가 있다.

### 5. IT 강사: GA와 프리뷰를 표로 먼저 보여 준다

교육할 때는 기능 이름보다 상태 표를 먼저 띄우는 편이 낫다. 수강생이 프리뷰 기능을 운영 설계에 넣는 실수를 가장 많이 하는 지점이기 때문이다. 자체 AKS 배포는 12월 초 "목표"일 뿐 아직 나오지 않았다는 점, Temporal은 2027년 초 계획이라는 점을 따로 짚는다.

## 이어서 읽을 글

같은 Ignite 흐름에서 Foundry의 개선 루프를 다룬 [Foundry '지속적 개선' 발표 — GA와 프리뷰를 갈라 읽기](/posts/foundry-continuous-improvement-ignite-2026/), 배포 명령을 다룬 [Foundry용 azd 확장 1.0 GA](/posts/foundry-azd-extension-ga/), 과금 시점을 다룬 [Copilot 하네스가 Foundry Agent Service에](/posts/copilot-harness-foundry-agent-service-preview/)와 함께 보면 흐름이 이어진다.

## 참고

- [Hosted agents in Foundry Agent Service at Ignite 2026](https://devblogs.microsoft.com/foundry/hosted-agents-foundry-agent-service-ignite-2026/) — Microsoft Foundry Blog, 2026-11-18
- [서브넷 크기와 동시 세션](https://learn.microsoft.com/azure/foundry/agents/concepts/agents-networking-deep-dive#subnet-size-and-concurrent-sessions)
- [이그레스 통제](https://learn.microsoft.com/azure/foundry/agents/how-to/add-hosted-agent-guardrails#network-egress-controls)
- [이그레스 거버넌스(Azure Policy)](https://learn.microsoft.com/azure/foundry/agents/how-to/govern-hosted-agent-egress-azure-policy)
- [사설 인증 기관](https://learn.microsoft.com/azure/foundry/agents/how-to/private-certificate-authority)
- [장기 실행 복원력](https://learn.microsoft.com/azure/foundry/agents/concepts/long-running-agent-resilience)
- [호스티드 세션 관리](https://learn.microsoft.com/azure/foundry/agents/how-to/manage-hosted-sessions)
- [메모리 스냅샷](https://learn.microsoft.com/azure/foundry/agents/how-to/use-hosted-agent-memory-snapshots)
- [A2A 서버 호스팅](https://learn.microsoft.com/azure/foundry/agents/how-to/host-agent-to-agent-server)
