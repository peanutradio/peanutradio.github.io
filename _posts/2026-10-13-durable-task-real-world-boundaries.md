---
title: "Durable Task가 해 주는 일, 해 주지 않는 일 — 에이전트 승인 설계"
date: 2026-10-13 06:53:00 +0900
categories: [AI Agent]
tags: [durable-task, ai-agent, orchestration, idempotency, human-in-the-loop]
description: Durable Task는 어디까지 진행했는지는 기억하지만 바깥 세상의 결과는 모른다. 에이전트 워크플로에서 직접 설계해야 할 승인·조정·장애 테스트 규칙을 정리했다.
---

> 이 글은 Tamir Dresher가 .NET 블로그에 올린 "Durable Task in the Real World"(2026-10-12)를 읽고 정리·재구성한 것입니다. 원문의 코드는 컴파일된 샘플이 아니라 설명용이며, 아래 해석과 체크리스트는 이 블로그의 관점입니다.
{: .prompt-info }

## 결론: 내구성 실행은 "내 위치"만 기억한다

Durable Task는 프로세스가 어디까지 진행했고, 무엇을 기다리고 있고, 어디서 재개할지를 기억해 줍니다. 하지만 결제가 실제로 승인됐는지, Azure 작업이 아직 도는지 같은 **외부 세계의 결과**는 알려 주지 않습니다. 그건 애플리케이션이 권위 있는 저장소, 안정적인 작업 ID, 조정(reconcile)으로 책임져야 합니다. 원문이 반복하는 판단 기준은 하나입니다. "기억하고 재개하는 코드가 그 자체로 하나의 시스템이 되어 버렸는가?"

앞서 정리한 [Azure Functions Dynamic Workflows](/posts/azure-functions-dynamic-workflows/)와 [Agent bindings](/posts/azure-functions-agent-bindings/)가 Durable을 실행 엔진으로 쓴다면, 이 글은 그 위에서 **무엇을 내가 직접 설계해야 하는지**에 대한 답입니다.

## 원문이 든 다섯 사례 (요약)

원문은 송장 승인, 테넌트 프로비저닝, 구독 결제, 속보 웹훅, AI 인시던트 어시스턴트를 다룹니다. 공통 패턴만 추리면 이렇습니다.

- 승인·완료 같은 외부 이벤트는 "깨우는 힌트"일 뿐, 결과는 권위 있는 저장소(검토 저장소, 결제 원장)가 정합니다.
- "수락됨"과 "준비됨"은 다른 상태입니다. 타임아웃은 거절이 아닙니다. 결제 상태에 `Unknown`을 두는 이유입니다.
- 보상(compensation)은 되돌리기 버튼이 아닙니다. 새 요청을 막고, 진행 중인 작업을 조정하고, 실패한 시도가 소유한 리소스만 지우고, 그래도 모르면 사람에게 넘깁니다(`NeedsIntervention`).
- 속보 웹훅 사례는 Durable Task만으로는 부족한 대표 예로 언급됩니다. 오래된 리비전을 거부하는 일은 목적지 쪽이 원자적으로 해야 합니다.

## 에이전트에 해당하는 다섯 규칙

AI 인시던트 어시스턴트(5번 사례)에서 원문이 제시한 규칙은 일반 에이전트 설계에도 그대로 옮길 수 있습니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="증거 수집, 제안 생성, 승인 대기, 실행으로 이어지는 흐름과 각 단계의 규칙">
  <defs>
    <marker id="dt-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 인시던트 어시스턴트의 네 단계와 지켜야 할 것</text>
  <rect x="8" y="50" width="150" height="60" rx="8" class="sk-box"/>
  <text x="83" y="76" text-anchor="middle" class="sk-label">증거 수집</text>
  <text x="83" y="95" text-anchor="middle" class="sk-sub">결과를 기록</text>
  <rect x="198" y="50" width="150" height="60" rx="8" class="sk-box"/>
  <text x="273" y="76" text-anchor="middle" class="sk-label">제안 생성</text>
  <text x="273" y="95" text-anchor="middle" class="sk-sub">모델 호출은 activity 안</text>
  <rect x="388" y="50" width="150" height="60" rx="8" class="sk-box-accent"/>
  <text x="463" y="76" text-anchor="middle" class="sk-label">승인 대기</text>
  <text x="463" y="95" text-anchor="middle" class="sk-sub">제안 ID+지문에 묶음</text>
  <rect x="578" y="50" width="134" height="60" rx="8" class="sk-box"/>
  <text x="645" y="76" text-anchor="middle" class="sk-label">실행</text>
  <text x="645" y="95" text-anchor="middle" class="sk-sub">정책 재확인</text>
  <path d="M160,80 L196,80" class="sk-line" marker-end="url(#dt-m1)"/>
  <path d="M350,80 L386,80" class="sk-line" marker-end="url(#dt-m1)"/>
  <path d="M540,80 L576,80" class="sk-line" marker-end="url(#dt-m1)"/>
  <rect x="8" y="150" width="704" height="56" rx="8" class="sk-box-muted"/>
  <text x="360" y="174" text-anchor="middle" class="sk-label">불확실한 부작용은 다시 실행하지 말고 먼저 조정(reconcile)</text>
  <text x="360" y="192" text-anchor="middle" class="sk-sub">실행 키와 영수증으로 확인</text>
</svg>
<figcaption>승인은 "대화"가 아니라 저장된 특정 제안에 묶여야 하고, 실행 직전에 다시 검증되어야 한다.</figcaption>
</figure>

1. 비결정적인 모델 호출은 activity 안에 두고 결과를 안정적인 생성 ID로 저장합니다.
2. 오케스트레이터는 다시 만든 계획이 아니라 저장된 제안의 참조(제안 ID와 지문)만 다룹니다.
3. 승인은 특정 제안과 고정된 마감 시각에 묶고, 하나의 최종 결정만 원자적으로 기록합니다.
4. 승인 이벤트는 힌트입니다. 실행 직전에 권한, 지문, 현재 정책을 다시 확인합니다.
5. 파괴적일 수 있는 도구를 증거 없이 재시도하지 않습니다.

## 장애 테스트: 워커를 네 지점에서 죽인다

원문은 워커를 네 곳에서 강제 종료해 보라고 제안합니다.

| 죽이는 지점 | 기대 동작 |
|---|---|
| 텔레메트리 수집 기록 후 | 재생 시 다시 수집하지 않음 |
| 모델 생성 후, 결과 기록 전 | 저장된 제안을 찾아 조정. 조용히 새 계획으로 바꾸지 않음 |
| 승인 대기 중 | 재시작 후에도 대기 계속, 마감은 밀리지 않음 |
| 조치 성공 후, 완료 기록 전 | 실행 키와 영수증으로 조정. 맹목적 재실행 금지 |

불변식은 "승인된 제안만 실행된다, 불확실한 부작용은 먼저 조정한다" 하나입니다. 원문은 .NET 기반 복구 실습 저장소(durable-task-recovery-lab)도 소개합니다.

## 실무에서 달라지는 점

- **에이전트 워크플로를 설계하는 개발자·운영자**: 설계 문서에 "권위 있는 상태가 어디인가, 재시도 ID는 무엇인가, 불확실한 외부 부작용은 무엇인가" 세 줄을 먼저 적게 하세요. 원문이 Durable 런타임을 고르기 전에 하라고 권하는 일입니다.
- **승인 단계가 있는 자동화(변경 관리, 장애 조치)**: 승인 화면이 보여 주는 대상과 실행되는 대상이 같은 제안 ID·지문인지 점검 항목에 넣으세요. 채팅 기록이나 "OK" 이벤트만 보고 실행하는 구조라면 고쳐야 합니다.
- **검수·테스트 담당**: 정상 경로 테스트 외에 위 네 지점의 강제 종료 시나리오를 인수 기준으로 추가하세요. 특히 "조치 후 기록 전" 종료는 이중 실행을 드러내는 가장 중요한 케이스입니다.
- **도입 여부 판단**: 단일 배포를 ARM/Bicep이 이미 처리하거나, 기존 큐·아웃박스·예약 작업이 깔끔히 돌면 굳이 넣지 않는다는 것이 원문의 입장입니다. 직접 만든 상태 머신이 커졌을 때를 도입 신호로 삼으세요.
- **IT 강사**: "Durable = 한 번만 실행 보장"이라는 오해를 먼저 깨 주세요. 기록된 결과는 재생되지만, 기록되기 전에 일어난 외부 효과는 별개라는 점이 수강생이 가장 자주 놓칠 부분일 것입니다.

## 확인할 것

원문 게시일이 소재 목록(10/9)과 본문(10/12)에서 다르게 보였고, 여기서는 본문 기준을 썼습니다. 구성 요소별 GA·프리뷰 상태는 원문에 없으므로 도입 전 공식 문서에서 확인해야 합니다. 원문 예시는 .NET이라 Python 중심의 Functions 글과 언어가 다릅니다. 원문은 후속으로 Durable Agents와 Durable Workflows(Microsoft Agent Framework 확장)를 다룰 예정이라고 합니다.

## 영상으로 보기

기초부터 짚고 싶다면 이 영상을 먼저 보세요. (AI 에이전트 기초 EP.01 — AI 에이전트란?)

{% include embed/youtube.html id='hVfiqgDm5gE' %}

> 📺 [Peanut Radio AI 유튜브 채널](https://www.youtube.com/@PeanutRadioAI) — AI 시대의 IT, 기초부터 실무까지
{: .prompt-tip }

## 참고

- [Durable Task in the Real World (.NET Blog, Tamir Dresher)](https://devblogs.microsoft.com/dotnet/durable-task-in-the-real-world/)
- [Durable Task .NET SDK](https://github.com/microsoft/durabletask-dotnet)
- [Durable Task 문서](https://learn.microsoft.com/azure/durable-task/)
- [durable-task-recovery-lab](https://github.com/tamirdresher/durable-task-recovery-lab)
- 이전 글: [Azure Functions Dynamic Workflows](/posts/azure-functions-dynamic-workflows/), [Azure Functions Agent bindings](/posts/azure-functions-agent-bindings/)
