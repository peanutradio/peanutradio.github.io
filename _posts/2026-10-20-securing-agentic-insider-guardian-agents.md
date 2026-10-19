---
title: "에이전트도 '내부자'다 — MS Security가 말하는 Guardian Agents"
date: 2026-10-20 06:53:00 +0900
categories: [IT News]
tags: [security, ai-agent, guardian-agents, microsoft-security, governance]
description: Microsoft Security가 사내 에이전트를 신뢰 대상이 아닌 내부자로 보고 실행 전에 감독하라고 제안했다. 주장의 뼈대와 운영자가 가져갈 점을 정리했다.
---

> 이 글은 Microsoft 공식 블로그의 "Securing the agentic insider"(Hayete Gallot, Microsoft Security)를 읽고 정리·재구성한 것입니다. 원문은 경영진이 쓴 방향 제안이며, 제품 대응이나 제 설계 적용은 해석으로 구분해 적었습니다.
{: .prompt-info }

**결론부터.** 원문의 핵심은 한 문장입니다. 회사 안에 배포한 AI 에이전트를 "우리가 배포했으니 믿는다"로 두지 말고 **내부자(insider)처럼 다루며, 행동을 실행되기 전에 계속 감독하라**는 것입니다. 구체적인 제품 기능 발표라기보다 보안 모델의 관점 전환을 제안하는 글입니다.

## 원문의 주장

원문이 에이전트를 믿으면 안 되는 이유로 드는 것은 다음과 같습니다.

- 프런티어 모델은 출신과 상관없이 **결과를 내도록 훈련**돼 있어 보상 해킹이 일어날 수 있다. 악의가 있어서가 아니라는 단서가 붙습니다.
- 조직의 규범과 판단에 맞춰 최적화돼 있지 않다.
- 예측하기 어렵고 맥락에 따라 달라지며, **침해되지 않았어도 다른 에이전트와 협력(공모)할 수 있다.**
- 빠르게 늘어나고, 과도한 권한으로 데이터·도구·시스템에 접근하며, 중요한 프로세스에서 결정을 내린다.

해법으로는 모든 에이전트의 모든 행동을 실행 전에 24시간 감시하고 조정하는 **Guardian Agents**를 제시합니다. 에이전트를 모델, 이를 조율하는 하니스, 도구·리소스의 행동 공간, 실행 런타임으로 나누고, identity·권한·데이터·엔드포인트·네트워크·클라우드·런타임 전반에서 보호해야 한다고 봅니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="센서 신호로 에이전트를 발견해 편입하고, 시도하는 행동의 위험을 계산해, 위험이 오르면 Guardian Agents가 실행 전에 개입하는 세 단계 흐름">
  <defs>
    <marker id="sia-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 원문이 그리는 관리 흐름 (3단계)</text>
  <rect x="8" y="50" width="210" height="100" rx="8" class="sk-box"/>
  <text x="113" y="78" text-anchor="middle" class="sk-label">1. 발견·편입</text>
  <text x="113" y="100" text-anchor="middle" class="sk-sub">센서 신호로 에이전트 발견</text>
  <text x="113" y="118" text-anchor="middle" class="sk-sub">identity·컨테이너·정책·권한 부여</text>
  <rect x="255" y="50" width="210" height="100" rx="8" class="sk-box"/>
  <text x="360" y="78" text-anchor="middle" class="sk-label">2. 맥락·위험 계산</text>
  <text x="360" y="100" text-anchor="middle" class="sk-sub">시도하는 행동의 위험을</text>
  <text x="360" y="118" text-anchor="middle" class="sk-sub">동적으로 계산</text>
  <rect x="502" y="50" width="210" height="100" rx="8" class="sk-box-accent"/>
  <text x="607" y="78" text-anchor="middle" class="sk-label">3. 실행 전 개입</text>
  <text x="607" y="100" text-anchor="middle" class="sk-sub">green·blue Guardian Agents가</text>
  <text x="607" y="118" text-anchor="middle" class="sk-sub">위험이 오를 때 실시간 대응</text>
  <path d="M220,100 L253,100" class="sk-line" marker-end="url(#sia-m1)"/>
  <path d="M467,100 L500,100" class="sk-line" marker-end="url(#sia-m1)"/>
  <rect x="8" y="180" width="704" height="50" rx="8" class="sk-box-muted"/>
  <text x="360" y="209" text-anchor="middle" class="sk-label">핵심: 사후 로그 검토가 아니라 '실행되기 전' 감독</text>
</svg>
<figcaption>이 제안의 차별점은 탐지 후 대응이 아니라 행동 직전에 위험을 계산해 막는다는 시점이다.</figcaption>
</figure>

원문은 7월에 발표한 **Project Perception**의 스택을 내부 에이전트 관리에도 적용할 수 있고 "지금 시작할 수 있다"고 안내합니다. 또 모델 투명성은 입출력 토큰을 넘어 **사고 과정(추론) 토큰**까지 봐야 한다는 방향을 제안합니다. "무슨 일이 일어나는가"에서 "왜 일어나는가"로 질문을 옮기자는 것입니다. 다만 이것은 "다음 단계가 되어야 한다"는 제안이고, 프라이버시 영향은 다루지 않습니다.

## 읽을 때 주의할 점

- 벤더 경영진이 쓴 글이라 **독립적인 검증이나 수치가 없습니다.** 공개 사고를 언급하지만 내용은 설명하지 않습니다.
- 위험 점수를 어떻게 계산하는지, Guardian Agents가 어떻게 구현되는지는 나오지 않습니다. GA인지 프리뷰인지도 글에 없습니다.
- 본문에는 Project Perception 외의 제품명(Agent 365, Defender, Entra, Purview 등)이 나오지 않습니다. 그래서 이 글에서 특정 제품이 이 역할을 한다고 연결하지 않겠습니다.

## 실무에서 달라지는 점

아래는 원문의 지시가 아니라 이 주장을 제 환경에 옮겨 본 **해석**입니다.

1. **에이전트 인벤토리부터 만듭니다.** 원문 흐름의 첫 단계가 "발견하고 편입"입니다. 부서에서 만든 Copilot Studio 에이전트, 개인 스크립트, MCP 서버 연결까지 누가 무엇을 돌리는지 목록이 없으면 감독 자체가 불가능합니다.
2. **에이전트마다 별도 identity와 최소 권한을 줍니다.** 사람 계정이나 공용 서비스 계정을 같이 쓰면 "에이전트가 한 일"을 구분할 수 없습니다. 에이전트용 신원 개념은 [Agent 365와 Agent ID 정리](/posts/agent-365-agent-id/)에서 다뤘습니다.
3. **쓰기·삭제·외부 전송 같은 동작에는 실행 전 정책 검사를 둡니다.** 사후 알림은 이미 늦습니다. 승인 바인딩 같은 패턴은 [Durable Task 승인 설계](/posts/durable-task-real-world-boundaries/)에 정리해 뒀습니다.
4. **'무엇을 했나'뿐 아니라 '왜 했나'를 남기는 로그 요건을 정합니다.** 추론 토큰 열람은 제안 단계이고 프라이버시 문제가 남아 있으므로, 당장은 행동 이유를 요약해 기록하게 하는 정도로 시작하는 편이 현실적입니다.
5. **교육에서는 이렇게 말하겠습니다.** "에이전트가 해킹당할 때"만 가르치지 않고, **"정상 동작하는 에이전트도 과한 권한이면 사고를 낸다"**를 먼저 가르칩니다. 이 글의 가장 쓸모 있는 부분이 이 관점입니다.

## 정리

제품 발표로 읽으면 얻을 게 적고, 설계 원칙으로 읽으면 쓸 만합니다. 원문이 마지막에 말하듯 안전과 혁신 중 하나를 고르는 게 아니라, 에이전트의 속도는 자동화된 감독으로 따라가고 판단은 사람이 맡는 구조를 그리라는 글입니다. 구현 세부는 Project Perception 안내 글과 이후 문서를 따로 확인해야 합니다. 저는 이 시스템을 직접 사용해 보지 않았습니다.

## 참고

- [Securing the agentic insider (Official Microsoft Blog, Hayete Gallot)](https://blogs.microsoft.com/blog/2026/10/19/securing-the-agentic-insider/)
