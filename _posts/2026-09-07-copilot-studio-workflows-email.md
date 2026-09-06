---
title: "Copilot Studio Workflows로 이메일 분류·처리 자동화하기"
date: 2026-09-07 08:30:00 +0900
categories: [Copilot Studio]
tags: [copilot-studio, workflows, mcp, automation, agent, m365]
description: 새 Workflows로 메일을 분류하고, 미팅을 잡고, 초안을 쓰고, 사람이 승인까지 하는 자동화를 만든다. 노드 구조와 막히기 쉬운 지점 정리.
---

Copilot Studio에 **Workflows**가 새로 들어왔습니다.
Microsoft의 Copilot Principal Engineer인 Shervin Shaffie가 이걸로
이메일 처리를 통째로 자동화하는 과정을 32분에 걸쳐 보여줬는데,
따라 만들면서 구조를 정리해봤습니다.

> **먼저 헷갈리는 것 하나.**
> M365 Copilot의 에이전트 목록에도 "Workflows"가 있습니다(Frontier 프로그램).
> 아이콘도 비슷합니다. 하지만 **이 글의 Workflows는 Copilot Studio 안의 것**이고,
> 훨씬 강력합니다. M365 Copilot 쪽은 훨씬 가볍고 기능이 제한적입니다.
{: .prompt-warning }

## 무엇을 만드는가

메일이 들어오면 **분류**하고, 종류에 따라 각각 다르게 처리합니다.

<figure class="sketch">
<svg viewBox="0 0 720 400" role="img" aria-label="이메일 자동화 워크플로 전체 구조 — 트리거, 분류, 4개 분기 처리">
  <defs>
    <marker id="cw-m" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="cw-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">① 전체 구조 — 분류 후 네 갈래</text>

  <rect x="8" y="168" width="118" height="58" rx="8" class="sk-box"/>
  <text x="67" y="192" text-anchor="middle" class="sk-label">트리거</text>
  <text x="67" y="210" text-anchor="middle" class="sk-sub">새 메일 도착</text>

  <path d="M128,197 L162,197" class="sk-line" marker-end="url(#cw-m)"/>

  <rect x="166" y="160" width="118" height="74" rx="8" class="sk-box-accent"/>
  <text x="225" y="188" text-anchor="middle" class="sk-label">Classify</text>
  <text x="225" y="206" text-anchor="middle" class="sk-sub">제목·본문·첨부</text>
  <text x="225" y="222" text-anchor="middle" class="sk-sub">중요도·발신자</text>

  <path d="M286,197 C316,197 316,52 348,52"   class="sk-line-accent" marker-end="url(#cw-a)"/>
  <path d="M286,197 C316,197 316,130 348,130" class="sk-line-accent" marker-end="url(#cw-a)"/>
  <path d="M286,197 C316,197 316,232 348,232" class="sk-line-accent" marker-end="url(#cw-a)"/>
  <path d="M286,197 C316,197 316,330 348,330" class="sk-line-accent" marker-end="url(#cw-a)"/>

  <rect x="352" y="30" width="150" height="44" rx="8" class="sk-box"/>
  <text x="427" y="49" text-anchor="middle" class="sk-label">meeting</text>
  <text x="427" y="65" text-anchor="middle" class="sk-sub">미팅 요청 메일</text>
  <path d="M504,52 L534,52" class="sk-line" marker-end="url(#cw-m)"/>
  <rect x="538" y="30" width="174" height="44" rx="8" class="sk-box-accent"/>
  <text x="625" y="49" text-anchor="middle" class="sk-label">meeting logic</text>
  <text x="625" y="65" text-anchor="middle" class="sk-mono">MCP: calendar</text>

  <rect x="352" y="108" width="150" height="44" rx="8" class="sk-box"/>
  <text x="427" y="127" text-anchor="middle" class="sk-label">reply requested</text>
  <text x="427" y="143" text-anchor="middle" class="sk-sub">답장이 필요한 메일</text>
  <path d="M504,130 L534,130" class="sk-line" marker-end="url(#cw-m)"/>
  <rect x="538" y="108" width="174" height="44" rx="8" class="sk-box-accent"/>
  <text x="625" y="127" text-anchor="middle" class="sk-label">M365 Copilot → drafter</text>
  <text x="625" y="143" text-anchor="middle" class="sk-mono">MCP: mail</text>

  <rect x="352" y="210" width="150" height="44" rx="8" class="sk-box"/>
  <text x="427" y="229" text-anchor="middle" class="sk-label">priority</text>
  <text x="427" y="245" text-anchor="middle" class="sk-sub">긴급·즉시 조치</text>
  <path d="M504,232 L534,232" class="sk-line-accent" marker-end="url(#cw-a)"/>
  <rect x="538" y="198" width="174" height="68" rx="8" class="sk-box-accent"/>
  <text x="625" y="220" text-anchor="middle" class="sk-label">사람이 승인</text>
  <text x="625" y="238" text-anchor="middle" class="sk-sub">Teams로 yes/no 질문</text>
  <text x="625" y="256" text-anchor="middle" class="sk-sub">→ if/else 분기</text>

  <rect x="352" y="308" width="150" height="44" rx="8" class="sk-box"/>
  <text x="427" y="327" text-anchor="middle" class="sk-label">informational</text>
  <text x="427" y="343" text-anchor="middle" class="sk-sub">그냥 정보성</text>
  <path d="M504,330 L534,330" class="sk-line" marker-end="url(#cw-m)"/>
  <rect x="538" y="308" width="174" height="44" rx="8" class="sk-box-accent"/>
  <text x="625" y="327" text-anchor="middle" class="sk-label">info sorter</text>
  <text x="625" y="343" text-anchor="middle" class="sk-mono">MCP: teams</text>

  <text x="360" y="384" text-anchor="middle" class="sk-sub">분류만 LLM이 하고, 실제 동작은 각 노드가 MCP 도구로 수행한다</text>
</svg>
<figcaption>핵심은 <strong>Classify 노드 하나로 분기가 갈린다</strong>는 것. 카테고리를 늘리면 분기가 늘어납니다.</figcaption>
</figure>

<figure class="shot">
  <div class="shot-icon">🖼</div>
  <div class="shot-what">Copilot Studio 캔버스에 올라간 완성된 워크플로 전체 모습</div>
  <div class="shot-time">영상 03:26</div>
  <figcaption>좌측 노드 팔레트와 우측 캔버스. tidy up으로 정렬하면 이 형태가 됩니다.</figcaption>
</figure>

## 1. 트리거 — 수동에서 자동으로

`copilotstudio.microsoft.com` → **Workflow** 선택하면 빈 캔버스가 나옵니다.

처음엔 `start` 노드가 **manual 트리거**로 잡혀 있습니다. 버튼을 눌러야 도는 상태죠.
자동으로 굴리려면 바꿔야 합니다. 선택지는 이렇습니다.

| 트리거 | 언제 쓰나 |
|---|---|
| manual | 수동 실행 (기본값) |
| **connector** | 외부 서비스 이벤트 — 이번에 쓸 것 |
| reoccurrence | 주기 실행 |
| HTTP request | 외부에서 호출 |
| 다른 에이전트 | 에이전트가 호출 |

`connector` → **Office 365 Copilot** → **when a new email arrives**를 골랐습니다.

커넥터 목록에는 Microsoft 것만 있는 게 아닙니다. Google, HelloSign 같은 것도 있습니다.

<figure class="shot">
  <div class="shot-icon">🖼</div>
  <div class="shot-what">커넥터 선택 화면 — Office 365 Copilot의 트리거 목록</div>
  <div class="shot-time">영상 07:10</div>
  <figcaption>when a new email arrives 외에 일정 생성·시작 임박 같은 트리거도 있습니다.</figcaption>
</figure>

> 🚨 **여기서 제일 많이 막힙니다.**
> 트리거 설정 안에 **connection**에 초록색 체크가 떠 있어야 합니다.
> 처음이라면 `create new connection`을 눌러 본인 계정으로 로그인해야 합니다.
> 라이선스가 맞으면 초록 체크가 뜹니다. **이게 없으면 뒤에 뭘 해도 안 돕니다.**
{: .prompt-danger }

`show all`을 누르면 트리거를 더 좁힐 수 있습니다. 특정 발신자한테서 온 메일에만 반응하게 하려면
이름이나 메일 주소를 넣으면 됩니다. 저는 전부 비워뒀습니다 — 모든 메일을 받겠다는 뜻입니다.

## 2. Classify — 분류 기준 만들기

`Classify` 노드를 캔버스로 끌어옵니다.

### 모델 선택이 곧 비용입니다

노드마다 모델을 고를 수 있습니다. Opus 5, GPT-5.5/5.6, Sonnet 4.6,
실험적인 Fable까지 나옵니다.

영상에서는 **Sonnet 4.6**을 골랐습니다. 이유가 명확합니다 —
**메일 분류에 Opus는 과합니다.** 성능은 충분하면서 훨씬 쌉니다.

> 노드가 10개면 모델 선택이 10번입니다. 여기서 비용이 갈립니다.
> 무거운 판단에만 큰 모델을 쓰고, 분류·정리는 작은 모델로 내리는 게 기본입니다.
{: .prompt-tip }

### 분류 기준과 동적 변수

프롬프트는 이런 식입니다.

```text
classify the email based on {body} {from} {subject} {importance} {attachments}
```

중괄호로 표시한 건 **동적 변수**입니다.
입력창의 **번개 아이콘**을 누르거나 `/`를 치면 메일에서 뽑을 수 있는 값들이 전부 뜹니다 —
from, to, cc, subject, body, importance, attachments 등등.

<figure class="shot">
  <div class="shot-icon">🖼</div>
  <div class="shot-what">Classify 노드 — 모델 선택 드롭다운과 동적 변수 삽입(번개 아이콘)</div>
  <div class="shot-time">영상 09:43</div>
  <figcaption>모델 목록과 dynamic content 패널이 함께 보이는 화면.</figcaption>
</figure>

### 카테고리 정의

기본 3개가 있고 더 추가할 수 있습니다. 네 개로 잡았습니다.

| 카테고리 | 설명 |
|---|---|
| `meeting` | an email that is requesting a new meeting |
| `reply requested` | an email that requires me to reply |
| `priority` | an email that is priority, time-sensitive, and requires immediate action |
| `informational` | an email that is just informational |

설명은 이렇게 한 줄이면 충분합니다.
분류가 애매하면 `add example`로 예시 텍스트를 넣어 기준을 더 명확히 할 수 있습니다.

카테고리를 추가하는 만큼 **캔버스에 분기가 자동으로 생깁니다.**

## 3. meeting — 에이전트에 MCP 도구를 붙인다

`meeting` 분기에 **agent 노드**를 연결합니다.

> ⚠️ **노드를 선으로 연결해야 변수가 넘어옵니다.**
> 연결 전에 번개 아이콘을 눌러도 아무것도 안 나옵니다.
> 선을 이은 다음에 다시 누르면 앞 노드의 값들이 전부 뜹니다.
> 이걸 모르면 "왜 변수가 안 보이지?"에서 한참 헤맵니다.
{: .prompt-warning }

지시문은 이렇게 넣었습니다.

```text
set up a 30 minute meeting with {from}
in the next 7 days.
use the information from the {body} of the email
to set the agenda and the title of the invite,
and include a Teams meeting link so that it's an online meeting,
and make sure that the time you select is free and available
on everyone's calendar, and only send out one invitation.
```

마지막 두 줄이 중요합니다. **"빈 시간을 확인해라", "초대는 한 번만 보내라"** —
이런 제약을 안 걸면 엉뚱하게 굴 수 있습니다.

### 도구는 MCP로

지시만으로는 아무것도 못 합니다. 실제로 일정을 만들려면 **도구**가 필요합니다.

`tools` → **MCP** → `calendar` → add.

기존 커넥터 방식 대신 MCP를 쓰는 걸 권하고 있었습니다.
추가하고 나서 calendar를 눌러보면 그 안에 **하위 도구들**이 쭉 뜹니다 —
`create event`(참석자 이름·메일을 넣어 새 일정 생성) 같은 것들이요.

MCP 서버 하나가 여러 도구를 한 번에 물고 오는 구조입니다.

<figure class="shot">
  <div class="shot-icon">🖼</div>
  <div class="shot-what">agent 노드에 MCP calendar 도구를 추가한 뒤 하위 도구 목록이 펼쳐진 화면</div>
  <div class="shot-time">영상 16:05</div>
  <figcaption>instructions · tools · knowledge 탭 구조는 Copilot Studio의 일반 에이전트와 같습니다.</figcaption>
</figure>

## 4. reply requested — M365 Copilot을 거쳐 초안까지

여기는 노드를 **두 개** 씁니다.

**① M365 Copilot 노드** — 내 OneDrive·SharePoint·Outlook·Teams 내용에 접근합니다.

```text
review the details of the email including the {from} {subject} {body} {attachments}
and using my content, draft an email reply
```

**② agent 노드** — 그 초안을 실제 **드래프트 폴더에 저장**합니다.

```text
review the draft from the previous step {body response from the Copilot agent}
and save the email as a draft that I can send to {from}
```

도구는 `MCP` → `mail`. 안에 draft 수정, 메시지 조회, 답장 등이 들어 있습니다.

앞 노드를 연결해두면 메일 원본 값뿐 아니라 **M365 Copilot 노드의 응답(conversation ID, 응답 본문)까지**
변수로 넘어옵니다. 이게 노드를 이어 쓰는 이유입니다.

## 5. priority — 사람이 중간에 개입한다

이 분기가 제일 재미있습니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="priority 분기 — 에이전트 판단 후 사람이 Teams에서 승인하고 결과에 따라 분기">
  <defs>
    <marker id="cw-h" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">② priority 분기 — human in the loop</text>

  <rect x="8" y="98" width="128" height="62" rx="8" class="sk-box-accent"/>
  <text x="72" y="122" text-anchor="middle" class="sk-label">priority logic</text>
  <text x="72" y="140" text-anchor="middle" class="sk-sub">긴급한지 판단·요약</text>

  <path d="M138,129 L170,129" class="sk-line-accent" marker-end="url(#cw-h)"/>

  <rect x="174" y="90" width="140" height="78" rx="8" class="sk-box-accent"/>
  <text x="244" y="114" text-anchor="middle" class="sk-label">human review</text>
  <text x="244" y="132" text-anchor="middle" class="sk-sub">Teams로 전송</text>
  <text x="244" y="150" text-anchor="middle" class="sk-mono">input: yes / no</text>

  <path d="M316,129 L348,129" class="sk-line-accent" marker-end="url(#cw-h)"/>

  <rect x="352" y="102" width="96" height="54" rx="8" class="sk-box"/>
  <text x="400" y="126" text-anchor="middle" class="sk-label">if / else</text>
  <text x="400" y="143" text-anchor="middle" class="sk-mono">== yes ?</text>

  <path d="M450,116 C476,116 476,58 502,58"  class="sk-line-accent" marker-end="url(#cw-h)"/>
  <path d="M450,142 C476,142 476,198 502,198" class="sk-line" marker-end="url(#cw-h)"/>
  <text x="476" y="80"  text-anchor="middle" class="sk-mono">yes</text>
  <text x="476" y="186" text-anchor="middle" class="sk-sub">no</text>

  <rect x="506" y="34" width="206" height="50" rx="8" class="sk-box-accent"/>
  <text x="609" y="55" text-anchor="middle" class="sk-label">immediate meeting</text>
  <text x="609" y="72" text-anchor="middle" class="sk-sub">30분 내 Teams 미팅 생성</text>

  <rect x="506" y="174" width="206" height="50" rx="8" class="sk-box"/>
  <text x="609" y="195" text-anchor="middle" class="sk-label">follow up later</text>
  <text x="609" y="212" text-anchor="middle" class="sk-sub">"24시간 내 연락드리겠습니다" 메일</text>

  <text x="360" y="244" text-anchor="middle" class="sk-sub">에이전트가 판단하고 → 사람이 확인하고 → 그 답에 따라 행동이 갈린다</text>
</svg>
<figcaption>자동화에 사람을 끼우는 지점. 에이전트 판단만 믿고 즉시 미팅을 잡지 않습니다.</figcaption>
</figure>

**① priority logic (agent)** — 긴급한지 판단하고, 맞으면 요약합니다. 도구는 필요 없습니다.

```text
review the email's body and importance and decide whether it's priority.
if it is, summarize it for me.
```

**② human review 노드** — 여기가 핵심입니다.

- **message**: 앞 에이전트의 응답을 번개 아이콘으로 가져옴
- **assign to**: 나 자신
- **channel**: **Teams** (메일보다 빨리 봄)
- **inputs**: `add input` → **yes/no** 형태

**③ if/else** — 값이 `yes`면 한쪽, 아니면 다른 쪽.

- **yes** → `immediate meeting` 에이전트. MCP `calendar` 도구로 30분 내 Teams 미팅 생성
- **no** → `follow up later` 에이전트. MCP `mail` 도구로 "긴급한 건 알겠고 24시간 내 연락드리겠다" 메일 발송

<figure class="shot">
  <div class="shot-icon">🖼</div>
  <div class="shot-what">human review 노드 설정 — assign to / channel(Teams) / yes-no input</div>
  <div class="shot-time">영상 23:20</div>
</figure>

## 6. informational — 지우고 알림만

```text
determine if this email is just informational.
if this email does not require any action from me
and is not asking me to set up or join a meeting,
it is most likely just information.
if so, delete the message and send me a brief Teams message.
```

도구는 `MCP` → `teams`.

`other` 분기는 비워뒀습니다. 어디에도 안 걸리는 메일이 뭔지 보고 나서 정하는 게 낫습니다.

## 7. 실행해보기

`save` → `publish` 후 **재생 아이콘**을 누르면 트리거 대기 상태로 들어갑니다.

priority로 테스트했습니다. 중요도 높음으로 메일을 하나 보냈더니,

| 단계 | 결과 |
|---|---|
| 트리거 감지 | 21초 |
| Classify → priority 판정 | 11.6초 |
| human review → Teams 카드 도착 | 요약 + yes/no 버튼 |
| `yes` 제출 | 즉시 다음 단계로 |
| immediate meeting | 발신자에게 Teams 링크 포함 초대 발송 |

<figure class="shot">
  <div class="shot-icon">🖼</div>
  <div class="shot-what">Teams로 도착한 human review 카드 — 메일 요약과 yes/no 버튼</div>
  <div class="shot-time">영상 29:35</div>
  <figcaption>Copilot Studio가 보낸 카드. 여기서 누르는 값이 if/else로 들어갑니다.</figcaption>
</figure>

실행이 끝난 뒤 각 노드를 클릭하면 **전체 응답과 판단 근거가 남아 있습니다.**
디버깅할 때 여기부터 봐야 합니다.

## 걸려 넘어지기 쉬운 곳 정리

몇 개는 실제로 시간 잡아먹는 것들입니다.

**① 저장을 안 하면 통째로 날아갑니다**
`save` 누르지 않고 홈이나 에이전트 화면으로 나가면 작업이 사라집니다.
publish는 마지막에 해도 되지만 **save는 자주** 해야 합니다.

**② connection 초록 체크**
트리거든 노드든 연결이 안 잡혀 있으면 아무것도 안 됩니다. 매 노드에서 확인해야 합니다.

**③ 선을 이어야 변수가 넘어온다**
앞에서 짚은 것. 연결 전에는 동적 변수 목록이 비어 있습니다.

**④ 노드 이름 바꾸기**
기본값이 `agent`, `agent 2`입니다. `meeting logic`, `email drafter`처럼
**만들면서 바로** 바꿔두는 게 낫습니다. 나중에 열 개 되면 못 알아봅니다.

**⑤ 문제 생기면 disable**
워크플로 목록에서 해당 항목을 **비활성화**할 수 있습니다.
자동화가 이상하게 돌면 일단 끄고, 고친 뒤 다시 publish하면 됩니다.

## 써보고 든 생각

인상적인 건 **MCP가 도구 계층으로 들어와 있다**는 점입니다.
`calendar`, `mail`, `teams` MCP 서버를 붙이면 그 안의 하위 도구들이 통째로 딸려옵니다.
커넥터를 하나씩 고르던 방식과 결이 다릅니다.
([MCP와 API가 어떻게 다른지는 따로 정리했습니다](/posts/mcp-vs-api/))

그리고 **human review 노드**. 완전 자동화가 아니라
"에이전트가 판단하고 사람이 확인하는" 구조를 노드 하나로 만들 수 있는 게 실무에서 중요합니다.
메일을 대신 보내거나 일정을 잡는 건 틀렸을 때 비용이 큰 동작이니까요.

다만 **모델 선택이 노드마다라는 점**은 양날입니다.
세밀하게 최적화할 수 있지만, 아무 생각 없이 기본값으로 두면 비용이 샙니다.

---

## 참고

- 원본 영상: [NEW Microsoft Workflows Will Automate Your Work | Masterclass By a Microsoft Engineer](https://www.youtube.com/watch?v=CzzF74wP-3g) — Collaboration Simplified (Shervin Shaffie, Microsoft Copilot Principal Engineer), 31분 54초
- 이 글은 위 영상의 내용을 따라가며 구조를 정리하고 재구성한 것입니다. 다이어그램은 직접 그렸습니다.
- 영상 설명에 명시된 대로, 시연은 특정 환경에서의 한 가지 구성 방법이며 모든 테넌트에서 그대로 동작한다는 보장은 없습니다.
