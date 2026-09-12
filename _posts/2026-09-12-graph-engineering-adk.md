---
title: "그래프 엔지니어링 — 거대 프롬프트가 환각하는 진짜 이유와 그 구조적 처방"
date: 2026-09-12 09:10:00 +0900
categories: [AI Agent]
tags: [ai-agent, graph-engineering, google-adk, orchestration, workflow]
description: 프롬프트 하나에 전부 담은 에이전트는 왜 자신 있게 거짓말을 하는가. Google ADK 2.0의 팬아웃·조인·라우터 세 패턴으로 LLM 호출 1회짜리 멀티에이전트 워크플로를 만드는 법.
---

Google Cloud Tech 채널에 올라온 6분짜리 영상 **[Graph Engineering with ADK](https://www.youtube.com/watch?v=Mzr7byMFy_4)** 를 정리했습니다.
발표자는 Annie, 주제는 **그래프 엔지니어링(Graph Engineering)** 입니다.

한 줄로 요약하면 이렇습니다.

> **에이전트가 환각하는 건 프롬프트가 나빠서가 아니라 구조가 없어서다.
> 처방은 더 좋은 프롬프트가 아니라 그래프다.**

영상은 마라톤 레이스 코치 에이전트 하나를 끝까지 끌고 가면서
"프롬프트 한 덩어리" → "노드 2개 그래프" → "팬아웃·조인·라우터가 들어간 그래프"로 세 번 리팩터링합니다.
그 과정에서 **LLM 호출 횟수는 끝까지 1회**로 유지됩니다. 이게 이 영상의 진짜 하이라이트입니다.

---

## 1. 문제 — 거대 프롬프트 하나짜리 에이전트

예제 상황은 단순합니다. 마라톤을 앞둔 러너가 에이전트에게 질문 하나를 던집니다.

> "이 레이스를 어떻게 뛰어야 하나요?"

첫 번째 시도는 **에이전트 하나, 거대 프롬프트 하나**입니다.
프롬프트 안에 워크플로 전체가 글로 적혀 있습니다.

- 날씨를 가져와라
- 코스를 분석해라
- 내 체력 상태를 확인해라
- 그걸로 레이스 전략을 세워라

돌려보면 답이 나옵니다. 그것도 **아주 자신 있고 아주 구체적으로.**
문제는 여기서 시작됩니다.

> 이 에이전트에는 **날씨 API가 연결돼 있지 않고, 코스 데이터도 없습니다.**
> 그러니까 답에 등장한 숫자는 **전부 지어낸 것**입니다.

영상은 이걸 "질병(disease)"이라고 부릅니다. 증상이 아니라 병입니다.

> **모든 단계가 단 한 번의 모델 호출 안에서 살면,
> 무엇도 가져올 수 없고, 무엇도 테스트할 수 없고, 무엇도 신뢰할 수 없다.**

여기서 중요한 포인트는 **"프롬프트를 더 잘 쓰면 해결된다"가 아니라는 것**입니다.
날씨 API가 없는 건 문장력의 문제가 아닙니다. 배선의 문제입니다.

---

## 2. 네 번째 층 — 프롬프트, 컨텍스트, 루프, 그리고 그래프

영상은 그래프 엔지니어링을 **지금까지 쌓아온 계층의 다음 층**으로 위치시킵니다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="프롬프트 엔지니어링, 컨텍스트 엔지니어링, 루프 엔지니어링, 그래프 엔지니어링 네 계층이 순서대로 이어지는 그림">
  <defs>
    <marker id="ar-g1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">① 에이전트 엔지니어링의 네 계층</text>

  <rect x="8" y="54" width="158" height="84" rx="9" class="sk-box"/>
  <text x="87" y="82" text-anchor="middle" class="sk-label">프롬프트</text>
  <text x="87" y="102" text-anchor="middle" class="sk-sub">모델에게</text>
  <text x="87" y="118" text-anchor="middle" class="sk-sub">무엇을 말할까</text>

  <path d="M170,96 L186,96" class="sk-line" marker-end="url(#ar-g1)"/>

  <rect x="190" y="54" width="158" height="84" rx="9" class="sk-box"/>
  <text x="269" y="82" text-anchor="middle" class="sk-label">컨텍스트</text>
  <text x="269" y="102" text-anchor="middle" class="sk-sub">모델 주변에</text>
  <text x="269" y="118" text-anchor="middle" class="sk-sub">무엇을 둘까</text>

  <path d="M352,96 L368,96" class="sk-line" marker-end="url(#ar-g1)"/>

  <rect x="372" y="54" width="158" height="84" rx="9" class="sk-box"/>
  <text x="451" y="82" text-anchor="middle" class="sk-label">루프</text>
  <text x="451" y="102" text-anchor="middle" class="sk-sub">계획 · 실행 · 점검</text>
  <text x="451" y="118" text-anchor="middle" class="sk-sub">사이클을 돌게</text>

  <path d="M534,96 L550,96" class="sk-line" marker-end="url(#ar-g1)"/>

  <rect x="554" y="54" width="158" height="84" rx="9" class="sk-box-accent"/>
  <text x="633" y="82" text-anchor="middle" class="sk-label">그래프</text>
  <text x="633" y="102" text-anchor="middle" class="sk-sub">여러 작업 조각을</text>
  <text x="633" y="118" text-anchor="middle" class="sk-sub">배선해서 묶는다</text>

  <text x="360" y="172" text-anchor="middle" class="sk-sub">조각 = 노드(node) · 배선 = 엣지(edge)</text>
</svg>
<figcaption>그래프 엔지니어링은 프롬프트를 대체하는 게 아니라, 프롬프트가 감당 못 하는 층을 새로 얹는 것입니다.</figcaption>
</figure>

정의는 의외로 담백합니다.

> **시스템을 위한 그래프 워크플로를 만드는 것.
> 그래프 안의 노드에는 에이전트를 둘 수도 있고, 결정론적인 함수를 둘 수도 있다.**

"에이전트 또는 함수"가 **같은 급의 노드**라는 점이 이 정의의 핵심입니다. 바로 다음 절에서 이게 왜 중요한지 드러납니다.

---

## 3. 가장 작은 그래프 — 노드 2개

리팩터링 1단계는 거창하지 않습니다. **문제를 풀 수 있는 가장 작은 그래프**를 만듭니다.

먼저 레이스 당일 컨디션을 가져오고, 그다음 전략을 짠다. 노드 두 개면 됩니다.

| 노드 | 정체 | 비용 |
|---|---|---|
| `fetch` | 평범한 파이썬 함수 | 모델 없음, 비용 0 |
| `advise` | 에이전트 | LLM 호출 1회 |

<figure class="sketch">
<svg viewBox="0 0 720 270" role="img" aria-label="왼쪽은 모든 단계를 담은 거대 프롬프트 하나가 환각을 내놓는 구조, 오른쪽은 함수 노드와 에이전트 노드 두 개로 나뉜 그래프 구조">
  <defs>
    <marker id="ar-g2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ar-ga2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">② 거대 프롬프트 하나 vs 노드 2개 그래프</text>

  <text x="8" y="44" class="sk-sub">시도 1 — 전부 프롬프트 안에</text>
  <rect x="8" y="56" width="300" height="150" rx="10" class="sk-box-muted"/>
  <text x="158" y="82" text-anchor="middle" class="sk-label">에이전트 1개 · 프롬프트 1개</text>
  <text x="158" y="106" text-anchor="middle" class="sk-sub">날씨 조회 → 코스 분석 → 체력 확인 → 전략 수립</text>
  <path d="M28,122 L288,122" class="sk-line" stroke-dasharray="4 4"/>
  <text x="158" y="146" text-anchor="middle" class="sk-sub">연결된 API 없음 · 실행된 단계 없음</text>
  <text x="158" y="172" text-anchor="middle" class="sk-mono">모든 숫자 = 지어낸 값</text>
  <text x="158" y="192" text-anchor="middle" class="sk-sub">가져올 수도, 테스트할 수도, 믿을 수도 없다</text>

  <path d="M316,130 L356,130" class="sk-line-accent" marker-end="url(#ar-ga2)"/>
  <text x="336" y="120" text-anchor="middle" class="sk-sub">구조화</text>

  <text x="366" y="44" class="sk-sub">시도 2 — 가장 작은 그래프</text>
  <rect x="366" y="56" width="68" height="56" rx="8" class="sk-box"/>
  <text x="400" y="88" text-anchor="middle" class="sk-label">start</text>

  <path d="M438,84 L474,84" class="sk-line" marker-end="url(#ar-g2)"/>

  <rect x="478" y="56" width="120" height="56" rx="8" class="sk-box"/>
  <text x="538" y="80" text-anchor="middle" class="sk-label">fetch</text>
  <text x="538" y="98" text-anchor="middle" class="sk-sub">파이썬 함수</text>

  <path d="M602,84 L638,84" class="sk-line" marker-end="url(#ar-g2)"/>

  <rect x="642" y="56" width="70" height="56" rx="8" class="sk-box-accent"/>
  <text x="677" y="80" text-anchor="middle" class="sk-label">advise</text>
  <text x="677" y="98" text-anchor="middle" class="sk-sub">에이전트</text>

  <rect x="366" y="134" width="346" height="72" rx="9" class="sk-box"/>
  <text x="539" y="160" text-anchor="middle" class="sk-sub">함수든 에이전트든 — 같은 목록, 같은 배선</text>
  <text x="539" y="182" text-anchor="middle" class="sk-mono">start → fetch → advise</text>
  <text x="539" y="199" text-anchor="middle" class="sk-sub">이 한 줄이 오케스트레이션의 전부다</text>

  <text x="360" y="242" text-anchor="middle" class="sk-sub">LLM 비용: 왼쪽 1회(전부 환각) · 오른쪽 1회(실제 데이터 기반)</text>
  <text x="360" y="260" text-anchor="middle" class="sk-sub">같은 비용, 전혀 다른 신뢰도</text>
</svg>
<figcaption>노드를 쪼갠다고 비용이 늘지 않습니다. 늘어나는 건 검증 가능성입니다.</figcaption>
</figure>

영상에서 "사람들이 놀라는 부분"이라고 콕 집어 말하는 지점이 있습니다.

> **함수와 에이전트가 같은 목록에, 같은 방식으로 배선된다.**

`start → fetch → advise`. 이 엣지 정의가 오케스트레이션의 **전부**입니다.
"함수를 부르는 코드"와 "에이전트를 부르는 코드"가 따로 있는 게 아닙니다.

그리고 원칙 하나가 나옵니다. 사실 대부분 이미 알고 있는 원칙입니다.

> **예측 가능한 일은 함수로, 추론은 모델로.**
> (Predictable work goes in functions, reasoning goes in the model.)

알고 있는데 안 지키는 이유는 보통 하나입니다. 프롬프트 하나에 다 때려넣는 게 당장은 빠르니까요.

---

## 4. 세 가지 패턴 — 팬아웃, 조인, 라우터

두 번째 리팩터링에서 그래프가 진짜 그래프다워집니다.

전략을 세우려면 입력이 **세 개** 필요합니다. 날씨, 코스, 체력.
그런데 이 셋은 **서로 의존하지 않습니다.** 날씨를 알아야 코스를 분석할 수 있는 게 아닙니다.

그럼 순서대로 할 이유가 없습니다.

<figure class="sketch">
<svg viewBox="0 0 720 330" role="img" aria-label="start에서 날씨 코스 체력 세 노드로 팬아웃하고, 조인 노드에서 합쳐진 뒤 라우터가 더움 보통 추움 세 전략 에이전트 중 하나를 고르는 그래프">
  <defs>
    <marker id="ar-g3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ar-ga3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">③ 팬아웃 → 조인 → 라우터</text>

  <text x="104" y="42" class="sk-sub">팬아웃 — 동시에 출발</text>
  <text x="286" y="42" class="sk-sub">조인</text>
  <text x="398" y="42" class="sk-sub">라우터</text>
  <text x="512" y="42" class="sk-sub">전략 에이전트 (택 1)</text>

  <rect x="6" y="132" width="66" height="50" rx="8" class="sk-box"/>
  <text x="39" y="162" text-anchor="middle" class="sk-label">start</text>

  <path d="M76,157 C92,157 88,80 100,80" class="sk-line" marker-end="url(#ar-g3)"/>
  <path d="M76,157 L100,157" class="sk-line" marker-end="url(#ar-g3)"/>
  <path d="M76,157 C92,157 88,234 100,234" class="sk-line" marker-end="url(#ar-g3)"/>

  <rect x="104" y="56" width="146" height="48" rx="8" class="sk-box"/>
  <text x="177" y="78" text-anchor="middle" class="sk-label">날씨 조회</text>
  <text x="177" y="94" text-anchor="middle" class="sk-sub">함수 · LLM 0회</text>

  <rect x="104" y="133" width="146" height="48" rx="8" class="sk-box"/>
  <text x="177" y="155" text-anchor="middle" class="sk-label">코스 분석</text>
  <text x="177" y="171" text-anchor="middle" class="sk-sub">함수 · LLM 0회</text>

  <rect x="104" y="210" width="146" height="48" rx="8" class="sk-box"/>
  <text x="177" y="232" text-anchor="middle" class="sk-label">체력 확인</text>
  <text x="177" y="248" text-anchor="middle" class="sk-sub">함수 · LLM 0회</text>

  <path d="M254,80 C270,80 268,145 282,145" class="sk-line" marker-end="url(#ar-g3)"/>
  <path d="M254,157 L282,157" class="sk-line" marker-end="url(#ar-g3)"/>
  <path d="M254,234 C270,234 268,169 282,169" class="sk-line" marker-end="url(#ar-g3)"/>

  <rect x="286" y="118" width="82" height="78" rx="9" class="sk-box"/>
  <text x="327" y="146" text-anchor="middle" class="sk-label">join</text>
  <text x="327" y="164" text-anchor="middle" class="sk-sub">전부 대기</text>
  <text x="327" y="180" text-anchor="middle" class="sk-sub">dict로 묶음</text>

  <path d="M372,157 L394,157" class="sk-line" marker-end="url(#ar-g3)"/>

  <rect x="398" y="118" width="86" height="78" rx="9" class="sk-box-accent"/>
  <text x="441" y="146" text-anchor="middle" class="sk-label">router</text>
  <text x="441" y="164" text-anchor="middle" class="sk-sub">if 문</text>
  <text x="441" y="180" text-anchor="middle" class="sk-sub">LLM 0회</text>

  <path d="M488,145 C504,145 500,80 516,80" class="sk-line-accent" marker-end="url(#ar-ga3)"/>
  <path d="M488,157 L516,157" class="sk-line" stroke-dasharray="4 4" marker-end="url(#ar-g3)"/>
  <path d="M488,169 C504,169 500,234 516,234" class="sk-line" stroke-dasharray="4 4" marker-end="url(#ar-g3)"/>

  <rect x="520" y="56" width="192" height="48" rx="8" class="sk-box-accent"/>
  <text x="616" y="78" text-anchor="middle" class="sk-label">더운 날 전략</text>
  <text x="616" y="94" text-anchor="middle" class="sk-sub">선택됨 · LLM 1회</text>

  <rect x="520" y="133" width="192" height="48" rx="8" class="sk-box-muted"/>
  <text x="616" y="155" text-anchor="middle" class="sk-label">보통 날 전략</text>
  <text x="616" y="171" text-anchor="middle" class="sk-sub">실행 안 됨</text>

  <rect x="520" y="210" width="192" height="48" rx="8" class="sk-box-muted"/>
  <text x="616" y="232" text-anchor="middle" class="sk-label">추운 날 전략</text>
  <text x="616" y="248" text-anchor="middle" class="sk-sub">실행 안 됨</text>

  <text x="360" y="296" text-anchor="middle" class="sk-mono">총 LLM 호출 = 1회</text>
  <text x="360" y="316" text-anchor="middle" class="sk-sub">노드가 5개로 늘었는데 모델을 부르는 노드는 여전히 하나뿐이다</text>
</svg>
<figcaption>노드 수와 LLM 비용은 비례하지 않습니다. 비례하는 건 "모델이 필요한 노드"의 수뿐입니다.</figcaption>
</figure>

### 패턴 1 — 팬아웃(Fan-out)

`start`에서 엣지를 세 개 뽑습니다. 세 개의 fetch가 **동시에 출발해 동시에 처리**됩니다.

여기에 ADK 2.0이 하나 더 얹는 게 있습니다. **다이나믹 팬아웃(dynamic fan-out)** 입니다.

> 병렬 처리를 해야 하는데 **몇 개가 동시에 돌지 미리 모르는 경우**,
> 다이나믹 워크플로로 **런타임에 그래프 모양을 만든다.**

정적으로 "3갈래"라고 못 박는 게 아니라, 입력을 보고 갈래 수를 정한다는 뜻입니다.

### 패턴 2 — 조인(Join)

세 갈래가 다 끝나면 **조인 노드**에서 만납니다. 영상은 이걸 "거대한 시스템 속 신디사이저"에 비유합니다.

조인 노드가 하는 일은 두 가지입니다.

1. **세 갈래를 전부 기다린다** — 가장 느린 branch에 맞춰 기다립니다
2. **결과를 노드 이름을 키로 하는 딕셔너리 하나로 묶는다**

여기가 실무자 입장에서 제일 반가운 대목입니다.

> **머저(merger)를 직접 짤 필요가 없다. 애그리게이터 에이전트를 만들 필요도 없다.
> 조인 노드가 그래프 안에서 그냥 동작한다.**

"결과 세 개를 합쳐줘"라고 LLM에게 시키는 순간 비용이 붙고 비결정성이 붙습니다.
그걸 그래프 원시 기능으로 흡수한 겁니다.

### 패턴 3 — 라우터(Router)

마지막이자, 영상이 **"다들 제일 쉽게 틀리는 모양"** 이라고 경고한 패턴입니다.

상황은 이렇습니다. 더운 날 / 보통 날 / 추운 날 — 전략이 셋이고, 전문 에이전트도 셋입니다. 하나를 골라야 합니다.

선택지는 두 개입니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="LLM 라우터와 결정론적 라우터의 비교. 열린 집합에는 LLM 라우터, 닫힌 집합에는 결정론적 라우터가 맞다">
  <defs>
    <marker id="ar-g4" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">④ 라우터를 고르는 기준 — 집합이 닫혀 있는가</text>

  <rect x="8" y="46" width="344" height="160" rx="10" class="sk-box"/>
  <text x="180" y="72" text-anchor="middle" class="sk-label">LLM 라우터</text>
  <text x="180" y="92" text-anchor="middle" class="sk-sub">모델에게 분류를 시킨다</text>
  <path d="M28,106 L332,106" class="sk-line" stroke-dasharray="4 4"/>
  <text x="180" y="130" text-anchor="middle" class="sk-mono">열린 집합 (open set)</text>
  <text x="180" y="152" text-anchor="middle" class="sk-sub">사용자가 자유 텍스트로 뭐든 입력 —</text>
  <text x="180" y="168" text-anchor="middle" class="sk-sub">if 문이 읽을 신호 자체가 없다</text>
  <text x="180" y="192" text-anchor="middle" class="sk-sub">토큰 비용 발생 · 비결정적이라 오독 가능</text>

  <rect x="368" y="46" width="344" height="160" rx="10" class="sk-box-accent"/>
  <text x="540" y="72" text-anchor="middle" class="sk-label">결정론적 라우터</text>
  <text x="540" y="92" text-anchor="middle" class="sk-sub">고정된 조건으로 분기한다</text>
  <path d="M388,106 L692,106" class="sk-line" stroke-dasharray="4 4"/>
  <text x="540" y="130" text-anchor="middle" class="sk-mono">닫힌 집합 (closed set)</text>
  <text x="540" y="152" text-anchor="middle" class="sk-sub">경우의 수가 정해져 있고</text>
  <text x="540" y="168" text-anchor="middle" class="sk-sub">판단 신호가 이미 데이터 안에 있다</text>
  <text x="540" y="192" text-anchor="middle" class="sk-sub">LLM 비용 0 · 항상 같은 결과</text>

  <text x="360" y="232" text-anchor="middle" class="sk-sub">마라톤 예제는 더움/보통/추움 — 닫힌 집합이므로 결정론적 라우터를 쓴다</text>
</svg>
<figcaption>라우터 선택의 기준은 "똑똑한가"가 아니라 "집합이 닫혀 있는가"입니다.</figcaption>
</figure>

**옵션 1 — LLM 라우터.** 모델에게 분류를 시킵니다. 동작은 합니다. 다만,

- **토큰 비용이 든다**
- **AI는 비결정적이라 잘못 읽을 수 있다**

**옵션 2 — 결정론적 라우터.** 고정된 조건을 가진 라우터 노드를 그래프에 둡니다.
hot → 더운 날 전략, normal → 보통 날 전략, cold → 추운 날 전략.

그럼 **언제 무엇을 쓰나.** 영상이 준 판별 기준은 명확합니다.

| 상황 | 성격 | 맞는 라우터 |
|---|---|---|
| 사용자가 자유 텍스트로 아무거나 입력 — if 문이 읽을 신호가 없다 | **열린 집합** | LLM 라우터 |
| 경우의 수가 정해져 있고 판단 신호가 이미 데이터에 들어 있다 | **닫힌 집합** | 결정론적 라우터 |

마라톤 예제는 기온이라는 신호가 데이터에 있고 경우의 수가 셋으로 닫혀 있으니 **결정론적 라우터**를 씁니다.
이유도 두 개입니다. **더 신뢰할 수 있고, 더 싸다.**

---

## 5. 비용을 세어보자 — 여전히 LLM 호출 1회

패턴 셋을 다 넣고 나서 영상은 비용을 셉니다.

- 병렬 fetch 3개
- 조인 1개
- 전략가(strategist) 1개

노드가 다섯입니다. 그런데 **LLM 호출은 정확히 1회**입니다.

> 모델 호출이 필요한 노드는 **전략가 에이전트 하나뿐**이고,
> 나머지는 전부 모델이 필요 없는 로직이기 때문입니다.

이 지점이 "멀티에이전트 = 비싸다"는 통념을 정면으로 건드립니다.
비싸지는 건 에이전트를 나눠서가 아니라, **모델이 필요 없는 일까지 모델에게 시켜서**입니다.

처음의 거대 프롬프트도 LLM 호출은 1회였습니다. 같은 1회를 쓰고도
한쪽은 전부 지어낸 숫자를 내놨고, 다른 쪽은 실제 데이터 위에서 답을 냈습니다.

---

## 6. 그런데, 그래프를 쓰면 안 되는 경우

영상이 좋았던 이유 중 하나는 여기서 브레이크를 밟는다는 점입니다.

> **가끔은 그래프가 아예 필요 없다.**

판단을 위한 질문 두 개를 줍니다.

**질문 1 — 입력이 도착하기 전에 워크플로를 그릴 수 있는가?**
그릴 수 있다면 그리고, 그래프 워크플로를 쓰면 됩니다.

**질문 2 — 모양 자체가 입력에 따라 달라진다면?**
예로 든 게 **딥리서치** 시나리오입니다. 그래프가 정확히 어떻게 생겼는지 미리 알 수 없습니다.
이 경우 ADK 2.0의 **다이나믹 워크플로**로, **코드가 런타임에 그래프 모양을 결정**하게 합니다.

즉 "그래프냐 아니냐"가 아니라 **"모양을 언제 정하느냐"** 의 문제로 바뀝니다.

| 워크플로 모양 | 결정 시점 | 도구 |
|---|---|---|
| 정적 — 미리 그릴 수 있다 | 설계 시점 | 그래프 워크플로 |
| 동적 — 입력에 따라 달라진다 | 런타임 | 다이나믹 워크플로 / 다이나믹 팬아웃 |

---

## 7. 정리하며

이 영상에서 건져갈 것을 네 줄로 줄이면 이렇습니다.

1. **환각은 프롬프트 품질 문제가 아니라 구조 문제다.** 한 번의 모델 호출 안에 모든 단계를 넣으면 아무것도 검증할 수 없다.
2. **예측 가능한 일은 함수로, 추론은 모델로.** 그리고 그래프에서 둘은 같은 급의 노드다.
3. **팬아웃 · 조인 · 라우터.** 합치는 일(조인)과 고르는 일(라우터)을 LLM에게 시키지 않는 것만으로 비용과 비결정성이 같이 빠진다.
4. **닫힌 집합이면 if 문, 열린 집합이면 LLM.** 라우터를 고르는 기준은 이거 하나다.

개인적으로 가장 오래 남은 문장은 조인 노드 설명이었습니다.
"머저를 직접 짤 필요도, 애그리게이터 에이전트를 만들 필요도 없다" — 병렬 결과를 합치겠다고
LLM 호출을 하나 더 쓰는 건 실제로 흔히 하는 실수인데, 그게 프레임워크 기능으로 내려가 있다는 건 꽤 큰 차이입니다.

**원본 자료**

- 영상: [Graph Engineering with ADK](https://www.youtube.com/watch?v=Mzr7byMFy_4) — Google Cloud Tech, 6분 9초
- 실습: [ADK Codelab](https://codelabs.developers.google.com/adk2/instructions#0)
- 문서: [adk.dev](https://adk.dev)
