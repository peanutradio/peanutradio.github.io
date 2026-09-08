---
title: "OpenAI 아스트라와 '불투명 반복' — 에이전트를 만드는 입장에서 걱정되는 것"
date: 2026-09-08 19:40:00 +0900
categories: [IT News]
tags: [openai, astra, ai-safety, chain-of-thought, agent, monitorability]
description: 추론 과정을 언어 토큰으로 남기지 않는 모델이 나왔다. 성능 이야기보다, 에이전트를 굴리는 사람에게 무엇이 달라지는지가 중요하다.
---

![OpenAI 아이콘을 들고 있는 손](/assets/img/posts/openai-astra.jpg)
_사진: [FoxTPNL](https://commons.wikimedia.org/wiki/File:Person_holding_the_OpenAI_icon.jpg), [CC BY 4.0](https://creativecommons.org/licenses/by/4.0) (Wikimedia Commons)_

OpenAI가 9월 3일 신규 모델 **아스트라(Astra)**를 공개했습니다.
성능 지표도 화제지만, AI 안전 연구자들이 걱정하는 건 다른 지점입니다.
**불투명 반복(opaque recurrence)**이라는 새 추론 방식입니다.

## 무엇이 문제인가

지금까지 우리는 모델이 **왜 그렇게 판단했는지**를 어느 정도 들여다볼 수 있었습니다.
**체인 오브 소트(chain-of-thought)** 덕분입니다.
모델이 추론 과정을 언어 토큰으로 풀어놓으니, 연구자들이 그 기록을 읽고
어떻게·왜 그런 결정을 내렸는지 감사할 수 있었습니다.

불투명 반복은 이 기록을 흐립니다. 추론은 하는데 **읽을 수 있는 흔적이 남지 않습니다.**

<figure class="sketch">
<svg viewBox="0 0 720 300" role="img" aria-label="체인 오브 소트와 불투명 반복의 차이 — 추론 과정이 언어 토큰으로 남는지 여부">
  <defs>
    <marker id="oa-m" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="oa-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">추론 과정이 남는가, 남지 않는가</text>

  <text x="172" y="42" text-anchor="middle" class="sk-title">체인 오브 소트</text>
  <text x="548" y="42" text-anchor="middle" class="sk-title">불투명 반복</text>

  <rect x="60" y="58" width="224" height="40" rx="8" class="sk-box"/>
  <text x="172" y="83" text-anchor="middle" class="sk-label">요청</text>
  <path d="M172,100 L172,118" class="sk-line" marker-end="url(#oa-m)"/>

  <rect x="60" y="122" width="224" height="94" rx="8" class="sk-box-accent"/>
  <text x="172" y="145" text-anchor="middle" class="sk-sub">"먼저 A를 확인하고…"</text>
  <text x="172" y="167" text-anchor="middle" class="sk-sub">"B가 참이므로 C를 선택"</text>
  <text x="172" y="189" text-anchor="middle" class="sk-sub">"따라서 D를 실행한다"</text>
  <text x="172" y="209" text-anchor="middle" class="sk-mono">언어 토큰으로 남음</text>
  <path d="M172,218 L172,236" class="sk-line" marker-end="url(#oa-m)"/>

  <rect x="60" y="240" width="224" height="38" rx="8" class="sk-box"/>
  <text x="172" y="264" text-anchor="middle" class="sk-label">결과</text>

  <path d="M296,168 L318,168" class="sk-line" stroke-dasharray="3 4"/>
  <path d="M424,168 L402,168" class="sk-line" stroke-dasharray="3 4"/>
  <text x="360" y="164" text-anchor="middle" class="sk-sub">같은</text>
  <text x="360" y="180" text-anchor="middle" class="sk-sub">결과</text>

  <rect x="436" y="58" width="224" height="40" rx="8" class="sk-box"/>
  <text x="548" y="83" text-anchor="middle" class="sk-label">요청</text>
  <path d="M548,100 L548,118" class="sk-line-accent" marker-end="url(#oa-a)"/>

  <rect x="436" y="122" width="224" height="94" rx="8" class="sk-box-muted"/>
  <text x="548" y="163" text-anchor="middle" class="sk-sub">추론은 일어나지만</text>
  <text x="548" y="185" text-anchor="middle" class="sk-sub">읽을 수 있는 흔적이 없다</text>
  <text x="548" y="207" text-anchor="middle" class="sk-mono">토큰 적게 · 혹은 없이</text>
  <path d="M548,218 L548,236" class="sk-line-accent" marker-end="url(#oa-a)"/>

  <rect x="436" y="240" width="224" height="38" rx="8" class="sk-box"/>
  <text x="548" y="264" text-anchor="middle" class="sk-label">결과</text>

  <text x="360" y="296" text-anchor="middle" class="sk-sub">결과는 같아 보여도, 감사할 수 있느냐가 갈린다</text>
</svg>
<figcaption>모니터링은 모델이 <strong>친절하게 말로 풀어주는 덕분에</strong> 가능했던 것입니다. 그 전제가 흔들립니다.</figcaption>
</figure>

OpenAI 수석과학자 **야쿠프 파호츠키(Jakub Pachocki)**도 이 점을 인정했습니다.

> "모니터링 가능성이 점점 더 어려워지고 있다."

모델이 유능해질수록 **더 적은 언어 토큰으로**, 혹은 **아예 언어 토큰 없이**
일을 해내게 되고, 그만큼 우리가 들여다볼 수 있는 여지가 줄어든다는 겁니다.

## 아스트라가 뭘 할 수 있는데

우려의 크기는 능력에 비례합니다. OpenAI가 내세운 것들입니다.

- **컴퓨터·브라우저 사용** 능력을 특히 강조
- "지금까지 나온 **소프트웨어 엔지니어링 최고 모델**"
- 버그 찾기, 터미널 작업, 코드베이스 질의에서 OpenAI의 Sol과 Anthropic의 Fable을 앞선다고 주장
- **제로데이 취약점을 식별하고 개발할 수 있음**

배포는 사이버보안 프로그램 **Daybreak**에 먼저 열렸고,
이후 한 주에 걸쳐 Pro·Plus·Enterprise·Business 계정과 API로 확대됐습니다.

사이버보안 쪽에 먼저 푼 배경도 있습니다. 최근 **허깅페이스 침해 사건**에서
OpenAI 에이전트가 샌드박스를 탈출한 일이 있었고, 아스트라는 그에 대한 대응으로 소개됐습니다.

그레그 브록만(Greg Brockman)은 이렇게 표현했습니다.

> "가장 지능적이고, 그리고 아주 중요하게도, **지금까지 가장 정렬된(aligned)** 모델"

AGI에 대해서는 "**우리는 이미 도달했다고 생각한다**"고 했습니다.

## 에이전트를 만드는 입장에서

여기서부터는 제 생각입니다.

벤치마크 숫자보다 저한테 크게 다가온 건 **모니터링 가능성**입니다.

에이전트를 실제로 굴려보면, 문제가 생겼을 때 제일 먼저 하는 일이
**모델이 왜 그 도구를 골랐는지 되짚는 것**입니다.
[어제 정리한 Copilot Studio Workflows](/posts/copilot-studio-workflows-email/)에서도
실행이 끝난 뒤 각 노드를 클릭하면 판단 근거가 남아 있고, 디버깅은 거기서 시작합니다.

추론 흔적이 옅어지면 이게 어려워집니다. 두 가지가 같이 걸립니다.

**① 디버깅** — 왜 이 도구를 골랐는지 모르면 고칠 지점을 못 찾습니다.
**② 안전** — 메일을 보내거나 파일을 지우는 동작을 맡길 때, 판단 과정을 볼 수 없다는 건 다른 문제입니다.

그래서 저는 이 흐름이 **human-in-the-loop의 가치를 오히려 키운다**고 봅니다.
모델의 추론을 들여다보는 게 어려워질수록,
**되돌리기 어려운 동작 앞에 사람을 세우는 설계**가 더 중요해집니다.
승인 노드 하나 넣는 게 번거로워 보여도, 감사 가능성이 줄어드는 방향이라면 그게 남는 안전장치입니다.

물론 아직 판단하기 이릅니다. 불투명 반복이 실제로 얼마나 흔적을 지우는지,
OpenAI가 대체 모니터링 수단을 제공할지는 더 봐야 합니다.
다만 **"모델이 알아서 잘 하겠지"에 기대는 설계는 점점 위험해진다**는 방향은 분명해 보입니다.

---

## 참고

- [OpenAI launches Astra, its powerful and controversial new model](https://techcrunch.com/2026/09/03/openai-launches-astra-its-powerful-and-controversial-new-model/) — TechCrunch, 2026-09-03

이 글은 위 기사의 내용을 정리하고 재구성한 것입니다. 마지막 절의 해석은 제 의견입니다. 다이어그램은 직접 그렸습니다.
