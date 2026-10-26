---
title: "Foundry 9월 업데이트를 에이전트 두 개로 읽기 — Routines·Tool Search"
date: 2026-10-17 06:53:00 +0900
categories: [Azure AI]
tags: [foundry, automation, ai-agent]
description: Foundry 9월 업데이트의 Routines·Tool Search GA와 Model Router·Insights 프리뷰를 변경 이력 에이전트 예제 하나로 엮어 읽고, 도입 전 확인할 점을 정리했다.
---

> 이 글은 Microsoft Foundry 블로그의 「What's new in Microsoft Foundry, September 2026」을 읽고 정리·재구성한 것이다. 직접 실행해 본 기록이 아니며, 수치와 상태는 모두 원문 기준이다.
{: .prompt-info }

**결론부터.** 이번 9월 업데이트에서 실무에 바로 닿는 것은 GA가 된 **Routines**와 **Tool Search**다. Model Router의 세션 친화성과 Insights는 아직 프리뷰다. 그리고 원문이 보여 주는 가장 쓸 만한 부분은 기능 목록이 아니라, 이 기능들을 에이전트 두 개짜리 워크플로 하나에 엮은 예제와 거기 깔린 안전장치다. [10/11에 정리한 Foundry Agent Service 첫 에이전트 글](/posts/foundry-first-agent/)이 "에이전트 하나 만들기"였다면, 이번 글은 "에이전트를 언제 깨우고 어디까지 믿을지"에 가깝다.

## 예제: 변경 이력을 만드는 에이전트 두 개

작성자는 Foundry 공개 변경 사항을 모아 개발자용 릴리스 노트로 만드는 흐름을 에이전트 둘로 나눴다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="Routine이 생산자 에이전트를 깨우고, 보고서가 수신자 에이전트를 거쳐 Teams 카드 한 장으로 전달되는 흐름">
  <defs>
    <marker id="fs-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 두 에이전트 변경 이력 워크플로</text>
  <rect x="8" y="70" width="130" height="60" rx="8" class="sk-box"/>
  <text x="73" y="96" text-anchor="middle" class="sk-label">Routine</text>
  <text x="73" y="114" text-anchor="middle" class="sk-sub">월요일 일정</text>
  <rect x="190" y="70" width="150" height="60" rx="8" class="sk-box-accent"/>
  <text x="265" y="96" text-anchor="middle" class="sk-label">depend-agent</text>
  <text x="265" y="114" text-anchor="middle" class="sk-sub">근거·빠진 범위 기록</text>
  <rect x="390" y="70" width="150" height="60" rx="8" class="sk-box-accent"/>
  <text x="465" y="96" text-anchor="middle" class="sk-label">maf-update-agent</text>
  <text x="465" y="114" text-anchor="middle" class="sk-sub">릴리스 노트 작성</text>
  <rect x="590" y="70" width="122" height="60" rx="8" class="sk-box"/>
  <text x="651" y="96" text-anchor="middle" class="sk-label">Teams 카드</text>
  <text x="651" y="114" text-anchor="middle" class="sk-sub">승인된 곳 한 곳</text>
  <path d="M140,100 L188,100" class="sk-line" marker-end="url(#fs-m1)"/>
  <path d="M342,100 L388,100" class="sk-line" marker-end="url(#fs-m1)"/>
  <path d="M542,100 L588,100" class="sk-line" marker-end="url(#fs-m1)"/>
  <text x="365" y="160" text-anchor="middle" class="sk-mono">Invocations</text>
  <text x="565" y="160" text-anchor="middle" class="sk-mono">Activity</text>
</svg>
<figcaption>시작은 Routine, 인계는 Invocations, 전달은 Activity — 단계마다 프로토콜과 책임이 분리돼 있다.</figcaption>
</figure>

생산자 `depend-agent`는 GitHub Copilot SDK 기반이고, 수집한 근거와 빠진 범위를 보고서에 남긴다. 수신자 `maf-update-agent`는 Microsoft Agent Framework 기반으로, 그 보고서를 받아 릴리스 노트를 쓰고 Teams에 카드 한 장을 보낸다. 월요일 오전 Routine이 생산자를 시작한다.

## 기능 상태 한눈에

| 기능 | 상태 | 한 줄 요약 |
|---|---|---|
| Routines | GA | 타이머·반복 일정·GitHub 이슈 이벤트·지원되는 Teams 채널 이벤트로 에이전트 하나를 시작 |
| Tool Search | GA | Toolbox가 도구 정의 전부 대신 `tool_search`, `call_tool` 두 개만 노출 |
| Model Router 세션 친화성·메타데이터 | 프리뷰 | 관련 요청이 같은 모델을 우선 쓰게 하고, 라우팅 시도·폴백을 요청 메타데이터로 확인. 32개 리전 |
| Insights | 퍼블릭 프리뷰 | 반복되는 트레이스 행동을 근거·추정 원인·다음 조치가 붙은 발견 사항으로 묶음 |

원문이 선을 그어 둔 부분도 있다. Routines는 다중 에이전트 오케스트레이션이나 승인 워크플로를 대체하지 않는다. 에이전트가 스스로 알림을 예약하는 도구는 여전히 프리뷰다. 프롬프트 에이전트용 Toolbox도 프리뷰이고, 호스티드 에이전트용만 GA다. 세션 친화성은 캐시 적중이나 같은 모델 선택을 보장하지 않는다.

## 내가 눈여겨본 설계 포인트

1. **"요청 수락 ≠ 보고서 완성 ≠ 메시지 전달."** 앱이 작업을 먼저 저장한 뒤 `start` 요청을 받고, 전달 확인이 유실되면 재전송하지 않고 "불확실"로 표시해 사람이 본다. [Durable Task 글](/posts/durable-task-real-world-boundaries/)에서 정리한 "승인과 재실행을 분리하라"는 이야기와 같은 결이다.
2. **도구가 없으면 빈 결과가 아니라 실패.** 필요한 도구를 못 찾으면 빈 변경 이력을 내지 않고 명시적으로 실패한다. 조용히 틀리는 쪽이 더 위험하다는 관점이다.
3. **예제 프롬프트의 안전장치.** Routine은 비활성 상태로 만든다. 모델이 수신자를 고르지 못하게 고정된 Teams 대상만 쓴다. 트레이스 내용 수집은 기본 끈다. 배포, 스케줄 활성화, 메시지 전송, 유료 Insights 스캔은 승인 후에만 한다. 공개 저장소 하나와 한 주 분량으로 시작한다.

## 실무에서 달라지는 점

- **Routines를 쓰려는 운영자:** 설정, 신원, 실행 기록이 Foundry 프로젝트에 남는다. 그러니 누가 Routine을 만들 수 있는지, 어떤 신원으로 돌아가는지를 프로젝트 권한에서 먼저 정리한다. 예제처럼 비활성 상태로 만들어 수동 실행부터 해 보는 순서를 사내 절차로 삼을 만하다.
- **정기 실행을 약속하기 전에:** 원문의 10월 13일 데모는 수동 디스패치로 전달과 카드 이동만 확인했다. 달력 일정에 따른 월요일 자동 실행까지 검증된 것은 아니다. 도입 검토 때 이 차이를 분명히 해 두어야 한다.
- **비용 점검:** 소스 수집과 작성은 모델과 도구를 호출하고, 호스티드 실행은 활성 컴퓨트 비용이 든다. 트레이싱과 Insights는 모니터링·판정 모델 비용을 더할 수 있고, Azure Monitor 트레이스 샘플링 기본값이 100%라고 원문이 적었다. 롤아웃 전에 트레이스 양을 보는 것이 좋다. Model Router 가격은 원문에 없다.
- **Tool Search 도입:** 도구가 많은 Toolbox일수록 후보지만, 원문 데모는 도구 탐색이 되는지만 보였고 토큰 절감은 측정하지 않았다. 절감 효과는 내 워크로드에서 따로 재야 한다. 읽기 전용 도구 몇 개로 시작하고, 앱 코드에서 허용 범위(저장소·경로·메서드)를 한 번 더 좁히는 구성을 교육 때 강조하고 싶다.
- **Model Router 쓸 때:** 배포 이름만으로는 실제 모델을 알 수 없고, 메타데이터의 `switch`는 정상적인 폴백일 수 있다. 관련 요청에는 같은 세션 ID, 무관한 작업에는 다른 ID를 쓰라는 안내를 강의 자료에 넣을 만하다. 안전·용량·쿼터 조건은 그대로 적용된다.

## 아직 모르는 것

보고서와 소스 기준선은 90일 수명이 있다. 원문은 9월 말로 예고된 루브릭 평가, 데이터셋 생성, Agent Optimizer GA가 실제로 나왔는지 확인해 주지 않는다. Insights도 발견 사항 하나를 골라 연결된 트레이스를 보고, 같은 입력으로 제안된 변경을 시험한 뒤 적용하라고 권한다. 자동으로 고쳐 주는 도구로 읽으면 안 된다.

## 참고

- [What's new in Microsoft Foundry, September 2026 (Foundry 블로그)](https://devblogs.microsoft.com/foundry/whats-new-in-microsoft-foundry-september-2026/)
- [Foundry Agent Service로 첫 에이전트 만들기](/posts/foundry-first-agent/)
- [Durable Task가 해 주는 일, 해 주지 않는 일](/posts/durable-task-real-world-boundaries/)
