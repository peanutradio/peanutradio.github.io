---
title: "Copilot 하네스가 Foundry Agent Service에 — 12월 1일 과금 전 확인할 것"
date: 2026-10-29 06:53:00 +0900
categories: [IT News]
tags: [foundry, ai-agent, copilot, cost]
description: Microsoft가 Copilot 하네스를 Foundry Agent Service 퍼블릭 프리뷰로 내놨다. 구성, 세 가지 선택지, 12월 1일 과금과 운영자가 점검할 항목을 정리한다.
---

> 이 글은 Microsoft Foundry 블로그의 "Microsoft Copilot harness in Foundry Agent Service"(2026-10-28)와 Official Microsoft Blog의 소개글을 읽고 정리·재구성한 것입니다. 직접 배포해 본 기록이 아니며, 성능 수치는 원문 그대로 벤더 주장임을 표시했습니다.
{: .prompt-info }

## 결론부터

Foundry Agent Service에서 에이전트를 만들 때 **"에이전트 루프와 샌드박스는 Microsoft가 돌리고, 나는 지침·스킬·도구·모델만 고르는"** 선택지가 퍼블릭 프리뷰로 생겼습니다. 이름은 Microsoft Copilot harness입니다. 핵심은 기능 목록보다 **운영 책임의 경계가 옮겨졌다**는 점입니다. 그리고 2026-12-01부터 샌드박스 컴퓨트 과금이 시작되니, 프리뷰 기간에 써 보더라도 비용 모델은 지금 계산해 두는 편이 좋습니다.

## 하네스가 맡는 것, 내가 맡는 것

공식 블로그는 하네스를 "모델 주변의 모든 것", 즉 지침·컨텍스트·실행 환경·통제를 묶은 오케스트레이션 계층으로 설명합니다. Copilot Code, Copilot Studio, GitHub Copilot 앱·CLI·VS Code 같은 제품이 같은 하네스를 쓴다고 합니다. 하네스라는 개념 자체는 [AI 하네스를 설계하는 4가지 축](/posts/ai-harness-four-parts/)에서 정리했으니, 이 글은 이번 발표에서 무엇이 실제로 제공되는지에 집중합니다.

구성 요소는 네 가지입니다.

- **재사용 스킬**

  지침·스크립트·자료를 절차 단위로 묶어 여러 작업에 적용합니다. `SKILL.md` 형식이라 [Agent Skills의 AAIF 이관](/posts/agent-skills-aaif-donation/) 흐름과도 이어집니다.

- **연결 도구**

  지원되는 Foundry 도구와 MCP 도구를 붙이되, 인증과 승인을 명시적으로 설정합니다.

- **실행과 위임**

  파일·코드 작업과, 범위가 좁은 일을 맡기는 내장 서브에이전트를 메인 에이전트가 조율합니다.

- **소스 관리 가능한 정의**

  Foundry에서 만들고 시험할 수도, 애플리케이션 코드 옆에서 정의 파일로 관리할 수도 있습니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="하네스를 쓰는 경우 Microsoft가 맡는 층과 사용자가 맡는 층의 구분">
  <defs>
    <marker id="ch-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 책임 경계 — 하네스 사용 시</text>
  <rect x="8" y="40" width="330" height="190" rx="10" class="sk-box-accent"/>
  <text x="173" y="64" text-anchor="middle" class="sk-label">내가 맡는 것</text>
  <text x="173" y="100" text-anchor="middle" class="sk-sub">지침 (무엇을 하는 에이전트인가)</text>
  <text x="173" y="130" text-anchor="middle" class="sk-sub">스킬 (어떤 절차로 하는가)</text>
  <text x="173" y="160" text-anchor="middle" class="sk-sub">도구·MCP (인증·승인 설정)</text>
  <text x="173" y="190" text-anchor="middle" class="sk-sub">모델 선택</text>
  <path d="M340,135 L380,135" class="sk-line-accent" marker-end="url(#ch-m1)"/>
  <rect x="382" y="40" width="330" height="190" rx="10" class="sk-box"/>
  <text x="547" y="64" text-anchor="middle" class="sk-label">Microsoft가 맡는 것</text>
  <text x="547" y="100" text-anchor="middle" class="sk-sub">에이전트 루프</text>
  <text x="547" y="130" text-anchor="middle" class="sk-sub">샌드박스 실행 환경</text>
  <text x="547" y="160" text-anchor="middle" class="sk-sub">CPU·메모리·유휴 설정</text>
  <text x="547" y="190" text-anchor="middle" class="sk-sub">서브에이전트 위임</text>
</svg>
<figcaption>하네스는 "무엇을·어떻게 시킬지"만 남기고 "어떻게 돌릴지"를 플랫폼으로 넘기는 선택지다.</figcaption>
</figure>

## 선택지는 세 가지

원문은 에이전트 만드는 방식을 세 갈래로 나눕니다.

### 1. 하네스 없는 프롬프트 에이전트

지침과 지원되는 도구만으로 충분할 때입니다. 가장 단순하고, 이미 쓰는 에이전트는 그대로입니다.

### 2. 하네스를 쓰는 프롬프트 에이전트

스킬, 관리형 실행, 파일, 서브에이전트, 복잡한 도구 조율이 필요하지만 루프를 직접 소유하고 싶지는 않을 때입니다. 이번 발표의 주인공입니다.

### 3. 호스티드 에이전트

프레임워크, 에이전트 루프, 미들웨어, 의존성, 런타임 동작을 직접 바꾸고 싶을 때입니다. 호스팅만 Foundry가 맡습니다.

2번과 3번의 갈림길은 "루프를 고치고 싶은가"입니다. 대부분의 사내 업무 에이전트는 루프를 고칠 이유가 없으니 2번에서 시작하고, 막힐 때 3번으로 가는 순서가 자연스러워 보입니다. 다만 원문은 **생성 후에 하네스를 바꾸는 것은 아직 지원하지 않는다**고 밝히므로, "일단 만들고 나중에 바꾸자"는 안 통합니다.

## 정의는 azure.yaml 하나로

원문 예제(research-brief 에이전트)는 `azure.yaml`에 네 덩어리를 둡니다. 프로젝트(`azure.ai.project`), 스킬(`azure.ai.skill`, 스킬 폴더를 archive로 지정), 에이전트(`azure.ai.agent`, `kind: prompt`와 `harness: type: microsoft_copilot_preview`, 모델 배포명), 인프라(`provider: microsoft.foundry`)입니다. 도구는 `web_search`와 `code_interpreter` 두 개입니다. 배포와 호출은 아래 두 줄입니다.

```bash
azd deploy --all
azd ai agent invoke research-brief "<질문>"
```

전체 YAML은 원문에서 확인하세요. 눈여겨볼 점은 에이전트 정의가 **코드와 같은 저장소에서 리뷰·버전 관리 대상**이 된다는 것입니다. 포털에서 클릭으로 만든 에이전트는 누가 언제 지침을 바꿨는지 추적하기 어렵지만, 정의 파일이면 PR 한 건으로 남습니다.

<figure class="sketch">
<svg viewBox="0 0 720 150" role="img" aria-label="정의 파일에서 배포, 호출까지의 흐름">
  <defs>
    <marker id="ch-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 정의에서 호출까지</text>
  <rect x="8" y="44" width="150" height="60" rx="8" class="sk-box"/>
  <text x="83" y="70" text-anchor="middle" class="sk-label">SKILL.md</text>
  <text x="83" y="90" text-anchor="middle" class="sk-sub">절차 정의</text>
  <path d="M160,74 L200,74" class="sk-line" marker-end="url(#ch-m2)"/>
  <rect x="202" y="44" width="150" height="60" rx="8" class="sk-box"/>
  <text x="277" y="70" text-anchor="middle" class="sk-label">azure.yaml</text>
  <text x="277" y="90" text-anchor="middle" class="sk-sub">에이전트·도구 정의</text>
  <path d="M354,74 L394,74" class="sk-line" marker-end="url(#ch-m2)"/>
  <rect x="396" y="44" width="150" height="60" rx="8" class="sk-box-accent"/>
  <text x="471" y="70" text-anchor="middle" class="sk-mono">azd deploy --all</text>
  <text x="471" y="90" text-anchor="middle" class="sk-sub">Foundry에 배포</text>
  <path d="M548,74 L588,74" class="sk-line" marker-end="url(#ch-m2)"/>
  <rect x="590" y="44" width="122" height="60" rx="8" class="sk-box"/>
  <text x="651" y="70" text-anchor="middle" class="sk-label">invoke</text>
  <text x="651" y="90" text-anchor="middle" class="sk-sub">질문 던지기</text>
</svg>
<figcaption>에이전트가 "설정 화면의 값"이 아니라 "저장소의 파일"이 되면 변경 이력과 리뷰가 따라온다.</figcaption>
</figure>

## 가격: 12월 1일을 기준으로 계산해 두기

원문에 따르면 하네스 런타임은 2026-12-01 전까지 무료입니다. 이후에는 샌드박스 컴퓨트가 vCPU-시간당 $0.0994, 메모리가 GiB-시간당 $0.0118입니다. 모델 추론과 별도 과금되는 도구·연결 서비스는 각자의 요금이 붙고, 가격은 리전과 계약에 따라 다를 수 있다고 합니다.

감을 잡기 위해 제가 계산한 예시입니다(원문에 없는 가정). 1 vCPU, 2 GiB 샌드박스가 1시간 켜져 있다면 0.0994 + 0.0118 × 2 = 약 $0.123입니다. 단가만 보면 작지만, 에이전트가 오래 도는 작업이 많거나 동시에 여러 개가 뜨면 곱해집니다. 한편 CPU·메모리·유휴 설정은 Foundry가 관리한다고 하니 **내가 크기를 줄여 비용을 통제할 수단이 지금은 제한적**일 수 있습니다. 이 부분은 문서에서 확인이 필요합니다.

## 장시간 실행 복원력은 GA, 하지만 숙제가 남는다

같은 글에서 호스티드 에이전트의 장시간 실행 복원력이 GA라고 알렸습니다. 중단이 감지되면 Foundry가 **원래 입력으로 작업을 다시 시작**합니다. 대화 기록만으로 진행 중이던 작업이 복원되지는 않으므로, 진행 상태와 중간 결과는 코드에서 직접 저장해야 합니다. 외부 시스템을 바꾸는 에이전트라면 재시작 때 중복 실행을 막아야 하고, 원문의 예는 보고서를 만들기 전에 이미 있는지 확인하는 것입니다. 이 주제는 [Foundry 음성 에이전트와 장시간 실행 복원력](/posts/foundry-voice-agents-long-running-resilience/)에서 프리뷰 항목과 함께 다뤘습니다. "자동 재시작"이 곧 "멱등성 보장"은 아니라는 점이 핵심입니다.

## 실무에서 달라지는 점

### 1. 프리뷰 에이전트에 맡길 업무의 등급 정하기

운영 담당자는 프리뷰 하네스를 곧바로 업무 시스템에 붙이지 말고, 읽기 전용 도구 위주의 조사·요약 에이전트부터 시험하는 편이 안전합니다. 원문이 지원 모델·리전·도구·프리뷰 한계를 본문이 아니라 limits-quotas-regions 문서로 넘기므로, 우리 테넌트 리전에서 쓸 수 있는지 그 문서로 먼저 확인해야 합니다.

### 2. 12월 1일 이전에 비용 담당과 단가 공유

FinOps나 구독 관리자는 11월 중 측정 기간을 잡아 에이전트 한 건당 샌드박스 가동 시간을 기록해 두는 것이 좋습니다. 무료 구간의 사용량이 12월 청구서의 기준선이 되기 때문입니다. 모델 추론 비용은 별도이니 "하네스 무료"를 "에이전트 무료"로 안내하지 않도록 주의하세요.

### 3. 도구 승인과 인증 설정은 배포 전에 리뷰

MCP 도구에 붙이는 인증과 승인은 "명시적으로 설정"하는 항목입니다. 보안 담당자는 어떤 도구가 쓰기 권한을 가지는지, 사람 승인이 필요한 동작이 무엇인지를 정의 파일 리뷰 체크리스트에 넣을 만합니다. 에이전트 신원 관리는 [Agent 365와 Agent ID](/posts/agent-365-agent-id/) 글과 함께 보시면 연결됩니다.

### 4. 중단 후 재시작을 전제로 한 설계 교육

개발자와 강사에게는 "중단되면 처음 입력으로 다시 시작한다"를 먼저 강조해야 합니다. 체크포인트 저장과 "이미 했는지 확인하기" 패턴이 없으면 메일이 두 번 나가거나 문서가 중복 생성될 수 있습니다. 교육 때 일부러 중간에 멈추는 실습 시나리오를 넣으면 좋습니다.

### 5. 하네스 선택은 되돌릴 수 없다고 가정

생성 후 하네스 변경이 아직 안 되므로, 팀 표준 문서에 "루프를 바꿀 가능성이 있으면 처음부터 호스티드 에이전트"라는 기준을 적어 두면 재작업을 줄일 수 있습니다.

## 걸러 읽을 부분

공식 블로그의 "20개 이상의 모델"과 Rust 포팅에 따른 "업계 최고 수준 성능", "수백만 사용자" 같은 표현은 수치나 근거가 본문에 없는 벤더 주장입니다. 이 글도 직접 실행해 확인한 내용이 아닙니다. 실제 도입 판단은 우리 업무로 작은 시험을 돌린 뒤에 해야 합니다. GitHub 쪽 후속 발표는 [GitHub의 앰비언트 인텔리전스](/posts/github-ambient-intelligence-three-layers/)에서 다룬 하네스·컨트롤 플레인 구도와 같은 맥락이라, 같은 하네스가 여러 제품 면에 퍼지는 중이라는 정도로 읽으면 됩니다.

## 참고

- [Microsoft Copilot harness in Foundry Agent Service — Microsoft Foundry Blog (2026-10-28)](https://devblogs.microsoft.com/foundry/microsoft-copilot-harness-in-foundry/)
- [Introducing the Microsoft Copilot harness — Official Microsoft Blog (2026-10-28)](https://blogs.microsoft.com/blog/2026/10/28/introducing-the-microsoft-copilot-harness-unlocking-the-potential-of-truly-multi-model-agentic-work/)
