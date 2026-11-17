---
title: "Foundry '지속적 개선' 발표 — GA와 프리뷰를 갈라 읽기"
date: 2026-11-18 06:51:00 +0900
categories: [IT News]
tags: [foundry, ai-agent, observability, cost]
description: Ignite 2026에서 공개된 Foundry 지속적 개선 기능 중 GA(Insights·음성 에이전트 등)와 프리뷰(옵티마이저·골든 데이터셋 등)를 나눠 실무 영향을 정리했다.
---

> 이 글은 Microsoft Azure 블로그의 "Continuous improvement comes to Microsoft Foundry"(2026-11-17)를 읽고 정리·재구성한 것입니다. 수치와 상태 표기는 모두 원문 기준이며, 직접 실행해 확인한 내용이 아닙니다.
{: .prompt-info }

## 결론부터

Foundry의 이번 발표는 "에이전트를 만들고 끝"이 아니라 **운영 데이터로 계속 고치는 루프**를 제품 기능으로 묶었다는 점이 핵심입니다. 다만 루프의 앞쪽(문제 발견 = Insights)은 GA이고, 뒤쪽(자동 최적화·골든 데이터셋·비용 보기)은 대부분 프리뷰입니다. 운영 환경에 바로 넣을 수 있는 것과 시범 적용만 할 것을 구분해서 읽어야 합니다.

## 원문이 말하는 구조

원문은 개선을 네 단계의 반복으로 설명합니다. 기준 세우기(Establish), 문제 찾기(Identify), 최적화(Optimize), 검증 후 반복(Validate)입니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="Foundry 지속적 개선 루프 네 단계와 각 단계의 GA 또는 프리뷰 상태">
  <defs>
    <marker id="fci-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="fci-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 개선 루프와 상태 표기</text>
  <rect x="8" y="40" width="150" height="90" rx="8" class="sk-box-muted"/>
  <text x="83" y="66" text-anchor="middle" class="sk-label">1 기준 세우기</text>
  <text x="83" y="88" text-anchor="middle" class="sk-sub">루브릭 평가(안전성)</text>
  <text x="83" y="104" text-anchor="middle" class="sk-sub">트레이스 주석·골든셋</text>
  <text x="83" y="120" text-anchor="middle" class="sk-mono">프리뷰</text>
  <rect x="190" y="40" width="150" height="90" rx="8" class="sk-box-accent"/>
  <text x="265" y="66" text-anchor="middle" class="sk-label">2 문제 찾기</text>
  <text x="265" y="88" text-anchor="middle" class="sk-sub">Insights</text>
  <text x="265" y="104" text-anchor="middle" class="sk-sub">LangGraph·CrewAI 지원</text>
  <text x="265" y="120" text-anchor="middle" class="sk-mono">GA</text>
  <rect x="372" y="40" width="150" height="90" rx="8" class="sk-box-muted"/>
  <text x="447" y="66" text-anchor="middle" class="sk-label">3 최적화</text>
  <text x="447" y="88" text-anchor="middle" class="sk-sub">스킬·비용 최적화</text>
  <text x="447" y="104" text-anchor="middle" class="sk-sub">Programmed Training</text>
  <text x="447" y="120" text-anchor="middle" class="sk-mono">프리뷰</text>
  <rect x="554" y="40" width="150" height="90" rx="8" class="sk-box"/>
  <text x="629" y="66" text-anchor="middle" class="sk-label">4 검증·반복</text>
  <text x="629" y="88" text-anchor="middle" class="sk-sub">지속 평가·대시보드</text>
  <text x="629" y="104" text-anchor="middle" class="sk-sub">CI에서 통계 검정</text>
  <text x="629" y="120" text-anchor="middle" class="sk-mono">기존 제공</text>
  <path d="M160,85 L188,85" class="sk-line" marker-end="url(#fci-m1)"/>
  <path d="M342,85 L370,85" class="sk-line" marker-end="url(#fci-m1)"/>
  <path d="M524,85 L552,85" class="sk-line" marker-end="url(#fci-m1)"/>
  <path d="M629,132 L629,185 L83,185 L83,134" class="sk-line-accent" marker-end="url(#fci-a1)"/>
  <text x="356" y="206" text-anchor="middle" class="sk-sub">새 버전이 다음 기준선이 된다</text>
</svg>
<figcaption>루프에서 지금 바로 운영에 쓸 수 있는 부분은 '문제 찾기'이고, 나머지는 프리뷰를 감안해야 한다.</figcaption>
</figure>

원문에서 상태가 명시된 항목은 아래와 같습니다.

| 구분 | 항목 |
|---|---|
| GA | hosted agents의 동시 세션 처리, Voice agents, Insights, LangGraph·LangChain·CrewAI 에이전트에 대한 Insights, Microsoft-Decision-1.1 |
| 프리뷰 | Microsoft Copilot harness, hosted agents의 빠른 시작, 안전성 루브릭 평가, 트레이스 주석, 평가 데이터셋, 골든 데이터셋, 인사이트를 옵티마이저로 보내기, Skill optimization, Cost optimization, Programmed Training, 에이전트용 Microsoft 365 계정, Cost in Foundry |
| 곧 제공(프리뷰) | hosted agents의 AKS 배포 |
| 상태 표기 없음 | Managed Training, Copilot Fusion, 런타임 Agent controls, Toolboxes·브라우저 자동화 업데이트 |

"상태 표기 없음"은 GA라는 뜻이 아닙니다. 표기가 없으면 문서나 포털에서 확인하기 전까지 프리뷰에 준해서 다뤄야 합니다.

## 숫자는 어떻게 읽을까

원문에는 눈에 띄는 숫자가 몇 개 있습니다. 모델 11,000개 이상, 모델 라우터로 최대 65% 절감, Alaska Airlines의 분류 정확도 81.2%에서 91.1%, Heidi의 노트당 비용 87% 절감 추정입니다. 이 가운데 65%는 "Our testing shows"라는 Microsoft 자체 테스트 표현이고, 나머지는 고객 사례입니다. 방법론·기간·비교 대상이 글에 없으므로 내 환경의 기대치로 옮기면 안 됩니다. 특히 "최대 65%"의 '최대'는 요청 분포가 단순한 질문 위주일 때의 상한으로 읽는 편이 안전합니다.

## 실무에서 달라지는 점

### 1. 에이전트 운영자: 먼저 Insights부터

운영 중인 에이전트가 있다면 GA인 Insights가 가장 부담이 적은 출발점입니다. 반복되는 문제를 묶어 근거와 추정 원인을 보여 주는 기능이라, 프리뷰 기능을 켜지 않아도 가치가 생깁니다. LangGraph·LangChain·CrewAI로 Foundry 밖에서 만든 에이전트도 대상이라고 하니, 프레임워크가 섞인 조직이라면 관측 창구를 하나로 모을 후보가 됩니다. 적용 전에 트레이스를 어디에 어떻게 보내는지(수집 설정)는 문서로 확인해야 합니다. 원문은 설정 경로를 알려 주지 않습니다.

### 2. 프리뷰 기능은 '운영 반영 금지, 비교용으로만'

옵티마이저는 원문 기준으로 사람이 적용하기 전에는 운영을 바꾸지 않는다고 합니다. 이 설계는 안전하지만, 프리뷰이므로 SLA와 과금 구조가 바뀔 수 있습니다. 앞서 정리한 [Copilot 하네스의 12월 1일 과금 시점](/posts/copilot-harness-foundry-agent-service-preview/)처럼 프리뷰가 과금 단계로 넘어가는 날짜를 놓치는 것이 가장 흔한 사고라서, 프리뷰 기능을 켤 때마다 종료·과금 시점을 메모해 두는 습관이 필요합니다.

### 3. 골든 데이터셋은 기능보다 조직 약속이 먼저

골든 데이터셋은 "새 버전이 이겨야 하는 고정 시험지"입니다. 도구가 프리뷰여도 개념은 지금 도입할 수 있습니다. 누가 좋은 답의 기준을 정하는지, 몇 건으로 시작하는지, 분기마다 누가 갱신하는지를 먼저 정하세요. 기준 없이 옵티마이저를 돌리면 "개선"이 무엇인지 말할 수 없습니다. 교육할 때도 이 순서(기준 → 측정 → 개선)를 강조할 만합니다.

### 4. 비용 담당자: Cost in Foundry와 라우터 실험을 분리

에이전트·모델·도구 지출을 한 화면에서 보고 예측까지 해 준다는 Cost in Foundry는 프리뷰입니다. 현행 청구서를 대체하는 것이 아니라 보조 뷰로 보고, 모델 라우터 절감 효과는 내 트래픽으로 A/B를 돌려 본 뒤에 예산에 반영하세요. 이전 글 [GPT-6 단가와 배포 옵션 정리](/posts/gpt-6-astra-sol-luna-foundry-pricing/)의 단가 표와 같이 놓고 보면 라우팅 전후 차이를 계산하기 쉽습니다.

### 5. 거버넌스: 런타임 통제의 범위를 확인

원문은 Foundry 밖에서 만든 에이전트의 런타임 가드레일 검사를 "in the future"라고 적었습니다. 지금은 Foundry 안에서 만든 에이전트가 대상이라는 뜻으로 읽힙니다. 다른 프레임워크를 쓰는 팀은 Insights(관측)와 런타임 통제(차단)가 같은 범위가 아니라는 점을 보안 검토서에 따로 적어야 합니다.

## 같이 보면 좋은 글

- [Foundry 음성 에이전트와 장시간 실행 복원력](/posts/foundry-voice-agents-long-running-resilience/) — 이번에 GA로 표기된 Voice agents의 직전 프리뷰 정리
- [Copilot Fusion GA](/posts/github-copilot-fusion-ga/) — 원문이 빌드 단계의 모델 라우팅으로 언급한 기능

## 참고

- Microsoft Azure Blog, "Continuous improvement comes to Microsoft Foundry" (2026-11-17): https://azure.microsoft.com/en-us/blog/continuous-improvement-comes-to-microsoft-foundry/
- Foundry 포털: https://ai.azure.com/home
- 원문이 링크한 모델 라우터 개념 문서: https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/model-router
- 원문이 링크한 Insights 문서: https://learn.microsoft.com/en-us/azure/foundry/observability/how-to/agent-insights
- 원문이 링크한 루브릭 평가자 문서: https://learn.microsoft.com/en-us/azure/foundry/concepts/evaluation-evaluators/rubric-evaluators
