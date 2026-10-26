---
title: "스스로 진화하는 장애 대응 에이전트 — 서명·정책·사람 승인으로 묶는 Chrysalis"
date: 2026-10-23 06:53:00 +0900
categories: [AI Agent]
tags: [ai-agent, governance]
description: Google의 Project Chrysalis는 장애 대응 에이전트가 전술을 진화시켜도 실행 권한은 서명·정책·사람 승인 안에 두는 참조 설계다. 패턴만 떼어 읽고 모순도 짚었다.
---

> 이 글은 Google for Developers Blog의 "Architecting governed self-evolving AI agents"(2026-10-22, Anant Nawalgaria)를 읽고 정리·재구성한 것입니다. 글 속 수치는 모두 시뮬레이션 데모 값이며, 저는 직접 실행해 보지 않았습니다.
{: .prompt-info }

## 결론부터

이 글에서 가져갈 것은 제품이 아니라 **패턴 하나**입니다. 에이전트가 만든 코드나 규칙이 실행되기 전에 반드시 통과해야 하는 관문(검사, 서명, 정책, 사람 승인)을 앞에 세운다는 것입니다. 원문의 표현으로는 "에이전트가 전술을 진화시켜도 실행 권한은 정책, 키 출처, 사람의 통제 안에 있다"입니다. 반대로 15분에서 1.8초 같은 성능 수치는 참고만 하세요. 시뮬레이션이고 프로덕션 검증이 없습니다.

## 시나리오와 구성

블랙프라이데이에 캐시 키가 대량으로 밀려나 DB 커넥션 풀이 고갈되고 500 오류가 나는 장애를 다룹니다. 원문은 이때 DB 커넥션 풀을 재시작하는 반사적 대응이 오히려 꼬리 지연을 악화시킨다고 설명합니다. ADK 기반 감독 라우터가 진단 에이전트와 핫패치 생성 에이전트를 부리고, 이후 트래픽 셰이핑 전문 에이전트가 그래프에 추가됩니다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="에이전트 산출물이 입력 검사, 서명, 정책과 사람 승인, 감사 로그의 네 관문을 통과해야 실행되는 흐름">
  <defs>
    <marker id="ch-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">③ 실행 전 네 관문</text>
  <rect x="8" y="60" width="110" height="60" rx="8" class="sk-box-muted"/>
  <text x="63" y="86" text-anchor="middle" class="sk-label">에이전트</text>
  <text x="63" y="104" text-anchor="middle" class="sk-sub">패치·규칙 생성</text>
  <rect x="150" y="60" width="120" height="60" rx="8" class="sk-box"/>
  <text x="210" y="84" text-anchor="middle" class="sk-label">1 입력·코드 검사</text>
  <text x="210" y="102" text-anchor="middle" class="sk-sub">인젝션, AST</text>
  <rect x="290" y="60" width="120" height="60" rx="8" class="sk-box"/>
  <text x="350" y="84" text-anchor="middle" class="sk-label">2 서명</text>
  <text x="350" y="102" text-anchor="middle" class="sk-sub">KMS, 미서명 거부</text>
  <rect x="430" y="60" width="130" height="60" rx="8" class="sk-box-accent"/>
  <text x="495" y="84" text-anchor="middle" class="sk-label">3 정책·사람 승인</text>
  <text x="495" y="102" text-anchor="middle" class="sk-sub">OPA, SRE 토큰</text>
  <rect x="580" y="60" width="130" height="60" rx="8" class="sk-box"/>
  <text x="645" y="84" text-anchor="middle" class="sk-label">4 감사 로그</text>
  <text x="645" y="102" text-anchor="middle" class="sk-sub">전 단계 기록</text>
  <path d="M120,90 L146,90" class="sk-line" marker-end="url(#ch-m1)"/>
  <path d="M272,90 L286,90" class="sk-line" marker-end="url(#ch-m1)"/>
  <path d="M412,90 L426,90" class="sk-line" marker-end="url(#ch-m1)"/>
  <path d="M562,90 L576,90" class="sk-line" marker-end="url(#ch-m1)"/>
</svg>
<figcaption>생성은 자유롭게, 실행은 관문을 통과한 것만 허용한다.</figcaption>
</figure>

## 네 관문과 세 층의 학습

관문은 이렇게 이어집니다. 입력 단계에서 Model Armor가 프롬프트 인젝션을 거르고 AST 검사로 허용되지 않은 시스템 호출을 막습니다. Cloud KMS로 산출물에 서명하고 Binary Authorization이 서명 없는 것을 거부합니다. OPA 정책으로 SLA 가드레일을 강제하고, 구조 변경(Tier-2)은 SRE의 승인 토큰을 요구합니다. 모든 단계는 Cloud Logging에 남습니다.

학습은 세 층입니다. 임시 핫패치로 장애를 먼저 막고, 서명된 교훈을 Vector Search에 저장해 "이 상황에서 DB 재시작은 금물" 같은 금지 행동을 기억하며, 사람 승인 후 영구 전문 에이전트를 추가합니다. 개입별 기여는 전후 텔레메트리로 점수화합니다.

## 인용할 때 짚어야 할 모순

원문은 OPA 정책이 보충 속도를 최소 10 req/s로 요구한다고 하면서, 4단계에서 고른 최적 정책은 5.0 req/s입니다. 같은 가드레일이면 막혀야 할 정책으로 보이는데 글은 이를 설명하지 않습니다. 그래서 이 글을 "정책 게이트가 완벽히 작동하는 증거"로 인용하면 곤란합니다. "게이트 설계 예시"까지만 읽는 것이 안전합니다. 용어도 "Gemini platform"과 "Agent Platform"이 섞여 있고, GA·프리뷰 상태는 적혀 있지 않습니다. 구현은 GitHub의 참조 데모로 공개된다고 합니다.

## 실무에서 달라지는 점

- **에이전트에 쓰기 권한을 주려는 운영자:** "읽기 전용 → 제안 → 승인 후 실행"의 단계를 먼저 문서로 그리세요. 어떤 변경이 사람 승인 대상(Tier-2)인지 기준이 없으면 게이트를 만들 수 없습니다.
- **보안 담당:** 에이전트가 만든 스크립트·설정에 서명을 요구하고 미서명을 거부하는 구조가 자기 환경에서 어디에 해당하는지 찾아보세요. MS 쪽 대응(예: Entra, Foundry 쪽 기능)은 제 해석이며 원문에는 없습니다. [에이전트도 '내부자'다](/posts/securing-agentic-insider-guardian-agents/) 글과 함께 읽으면 비교가 쉽습니다.
- **승인 흐름 설계자:** 승인이 필요한 구간의 대기 처리는 [Durable Task가 해 주는 일](/posts/durable-task-real-world-boundaries/)에서 다룬 장기 대기 패턴과 맞물립니다.
- **도입 검토자:** 이 데모의 수치를 내부 보고서에 옮기지 마세요. 시뮬레이션이라는 단서를 반드시 같이 쓰세요.
- **IT 강사:** "에이전트가 똑똑해질수록 통제는 필요 없다"는 오해를 풀 사례로 쓸 수 있습니다. 진화하는 쪽과 권한을 쥔 쪽을 분리해 보여 주세요.

## 영상으로 보기

기초부터 짚고 싶다면 이 영상을 먼저 보세요. (AI 에이전트 기초 EP.01 — AI 에이전트란?)

{% include embed/youtube.html id='hVfiqgDm5gE' %}

> 📺 [Peanut Radio AI 유튜브 채널](https://www.youtube.com/@PeanutRadioAI) — AI 시대의 IT, 기초부터 실무까지
{: .prompt-tip }

## 참고

- Google for Developers Blog, Architecting governed self-evolving AI agents (2026-10-22): https://developers.googleblog.com/architecting-governed-self-evolving-ai-agents-on-gemini-agent-platform/
