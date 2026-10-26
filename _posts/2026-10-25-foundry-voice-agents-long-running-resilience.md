---
title: "Foundry 음성 에이전트와 장시간 실행 복원력 — 프리뷰 항목 읽기"
date: 2026-10-25 06:53:00 +0900
categories: [Azure AI]
tags: [foundry, ai-agent, observability]
description: Foundry의 음성 에이전트와 장시간 실행 복원력은 프리뷰, 도구 검색·A2A·Routines는 GA다. 상태별로 나눠 도입 순서를 해석했다.
---

> 이 글은 Microsoft Azure 블로그의 "Ship agents faster with expanded model choice, voice agents, and continuous optimization"(9/24, Tina Schuchman)을 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록이 아니며, 토큰 감소 수치는 Microsoft 내부 평가입니다.
{: .prompt-info }

**결론부터.** 이 글의 신규 항목은 두 부류입니다. 음성 에이전트와 장시간 실행 복원력은 **퍼블릭 프리뷰**라 파일럿용이고, 도구 검색·A2A·Routines는 **GA**라 운영에 넣을 수 있습니다. 도구 검색과 Routines는 [Foundry 9월 업데이트 정리](/posts/foundry-sept-2026-two-agent-changelog/)에서 이미 다뤘으니 여기서는 짧게 넘기고 새 항목에 집중합니다.

## 상태별 정리

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="Foundry 항목을 GA, 프리뷰, 예정으로 나눈 스케치">
  <defs>
    <marker id="fvr-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 항목별 상태 (원문 게시 시점)</text>
  <rect x="8" y="40" width="220" height="170" rx="8" class="sk-box-accent"/>
  <text x="118" y="64" text-anchor="middle" class="sk-label">GA</text>
  <text x="118" y="96" text-anchor="middle" class="sk-sub">도구 검색 (Toolbox)</text>
  <text x="118" y="118" text-anchor="middle" class="sk-sub">A2A</text>
  <text x="118" y="140" text-anchor="middle" class="sk-sub">Routines</text>
  <rect x="250" y="40" width="220" height="170" rx="8" class="sk-box"/>
  <text x="360" y="64" text-anchor="middle" class="sk-label">퍼블릭 프리뷰</text>
  <text x="360" y="96" text-anchor="middle" class="sk-sub">음성 에이전트</text>
  <text x="360" y="118" text-anchor="middle" class="sk-sub">장시간 실행 복원력</text>
  <text x="360" y="140" text-anchor="middle" class="sk-sub">Insights</text>
  <rect x="492" y="40" width="220" height="170" rx="8" class="sk-box-muted"/>
  <text x="602" y="64" text-anchor="middle" class="sk-label">예정 · 미확인</text>
  <text x="602" y="96" text-anchor="middle" class="sk-sub">에이전트 옵티마이저</text>
  <text x="602" y="118" text-anchor="middle" class="sk-sub">VS Code용 툴킷</text>
  <text x="602" y="140" text-anchor="middle" class="sk-sub">이그레스 통제(상태 미표기)</text>
</svg>
<figcaption>운영에 넣을 것(GA)과 시험할 것(프리뷰)을 먼저 갈라야 도입 순서가 보인다.</figcaption>
</figure>

## 새 항목 두 가지

**음성 에이전트(프리뷰)**: 프롬프트 에이전트와 호스티드 에이전트를 모두 지원하는 새 에이전트 유형입니다. 원문은 80개 이상 언어, 140개 이상 로캘을 지원한다고 하고, 배포 대상은 웹, Teams, Teams Phone, Twilio 기반 전화입니다. 맞춤 음성과 아바타 옵션도 언급됩니다. 반면 음성 관측성(트레이스, 대화 기록, 음성 평가)의 상태는 원문에 없습니다.

**장시간 실행 복원력(프리뷰)**: 요청 연결이 끊겨도 호스티드 에이전트 작업이 계속되고, 호스팅 프로세스가 멈춰도 복구됩니다. Microsoft Agent Framework는 내구성 체크포인트에서 워크플로를 재개할 수 있습니다.

## GA·예정 항목 요약

도구 검색은 내부 평가(도구 4.4만 개 이상, 질의 7천 개 이상 공개 벤치마크)에서 입력 토큰이 도구 50개 구성은 60% 넘게, 1,000개 구성은 97% 넘게 줄었다고 합니다. 기준선은 프롬프트 캐시를 쓴 전체 카탈로그 방식입니다. Insights(프리뷰)는 운영 트레이스에서 반복 문제를 찾아 줍니다. 에이전트 옵티마이저는 글 게시 시점에 GA 전이었고 "이달 중 GA 예정"이라고 했는데, 지금 GA인지는 확인하지 못했습니다.

## 실무에서 달라지는 점

1. **음성 고객 응대를 검토하는 팀**: 프리뷰라서 서비스 수준 약속(SLA)을 걸 운영에는 이릅니다. Teams Phone이나 Twilio 연동 파일럿부터 하고, 관측성 상태가 불명확하니 통화 기록과 품질 평가를 어떻게 남길지 먼저 정하세요.
2. **오래 걸리는 에이전트 작업 담당**: 장시간 복원력은 연결 끊김 때의 재시도 설계를 줄여 줄 수 있습니다. 다만 프리뷰이고, 승인 대기가 길어지는 업무라면 [Durable Task 승인 설계](/posts/durable-task-real-world-boundaries/)와 역할이 어떻게 나뉘는지 검증이 필요합니다(이건 제 확인 과제입니다).
3. **Toolbox 운영자**: 도구가 수십 개 이상이면 도구 검색을 켜기 전후 입력 토큰을 직접 재 보세요. 60%, 97%는 내부 평가 수치라 우리 환경과 다를 수 있습니다.
4. **네트워크·보안 담당**: 이그레스 통제는 허용·거부 규칙과, 결정을 Application Insights에 남기는 감사 모드가 핵심입니다. 상태가 명시되지 않았으니 가용 여부부터 문의하세요. 먼저 감사 모드로 어디로 나가는지 보고 규칙을 강제하는 순서가 자연스럽습니다(제 해석).
5. **강사 관점**: 같은 Foundry 글에서도 "GA와 프리뷰를 구분해서 말하기"를 교육 포인트로 삼겠습니다. 가격과 리전 정보가 원문에 없다는 점도 함께 알려 줍니다.

## 참고

- [Ship agents faster with expanded model choice, voice agents, and continuous optimization — Azure Blog](https://azure.microsoft.com/en-us/blog/ship-agents-faster-with-expanded-model-choice-voice-agents-and-continuous-optimization/)
