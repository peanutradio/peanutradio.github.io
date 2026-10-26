---
title: "Copilot Studio에 MCP 서버 붙이기 — 카탈로그 연결과 직접 등록 두 가지"
date: 2026-10-12 11:55:00 +0900
categories: [Copilot Studio]
tags: [copilot-studio, mcp, ai-agent]
description: Microsoft Learn Docs MCP를 Copilot Studio 에이전트에 연결해 공식 문서를 검색하고 출처 링크까지 달아 답하게 했다. 카탈로그에서 고르는 법과 내 MCP 서버 주소를 직접 넣는 화면을 순서대로 정리했다.
---

**결과부터.** "MS Learn 문서 도우미"라는 에이전트에 **Microsoft Learn Docs MCP Server** 하나를 도구로 붙였다. "Durable Functions는 언제 쓰는 게 좋아?"라고 물으니 에이전트가 스스로 `Microsoft Docs Search`를 한 번, `Microsoft Docs Fetch`를 두 번 부른 뒤, 한국어 요약과 learn.microsoft.com 출처 링크 5개를 붙여 답했다. 코드는 한 줄도 쓰지 않았다.

[지난번엔 MCP 서버를 직접 만들어 VS Code에 붙였는데](/posts/memo-mcp-first-server/), 이번엔 반대로 **이미 있는 MCP 서버를 로우코드 에이전트에 붙이는 쪽**이다.

<figure class="sketch">
<svg viewBox="0 0 720 210" role="img" aria-label="사용자 질문을 받은 Copilot Studio 에이전트가 MCP 서버의 검색 도구와 가져오기 도구를 차례로 호출하고 그 결과로 답한다">
  <defs>
    <marker id="csm-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/></marker>
    <marker id="csm-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/></marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 에이전트가 MCP 도구를 고르는 흐름</text>
  <rect x="8" y="78" width="120" height="52" rx="8" class="sk-box"/>
  <text x="68" y="101" text-anchor="middle" class="sk-label">질문</text>
  <text x="68" y="119" text-anchor="middle" class="sk-sub">Durable Functions?</text>
  <path d="M130,104 L176,104" class="sk-line" marker-end="url(#csm-m1)"/>
  <rect x="180" y="62" width="170" height="84" rx="8" class="sk-box-accent"/>
  <text x="265" y="90" text-anchor="middle" class="sk-label">Copilot Studio</text>
  <text x="265" y="108" text-anchor="middle" class="sk-sub">에이전트(오케스트레이터)</text>
  <text x="265" y="126" text-anchor="middle" class="sk-sub">어떤 도구를 몇 번 쓸지 결정</text>
  <path d="M352,90 L420,60" class="sk-line-accent" marker-end="url(#csm-m2)"/>
  <path d="M352,118 L420,150" class="sk-line-accent" marker-end="url(#csm-m2)"/>
  <rect x="424" y="36" width="160" height="46" rx="8" class="sk-box"/>
  <text x="504" y="56" text-anchor="middle" class="sk-mono">Docs Search ×1</text>
  <text x="504" y="72" text-anchor="middle" class="sk-sub">관련 문서 찾기</text>
  <rect x="424" y="128" width="160" height="46" rx="8" class="sk-box"/>
  <text x="504" y="148" text-anchor="middle" class="sk-mono">Docs Fetch ×2</text>
  <text x="504" y="164" text-anchor="middle" class="sk-sub">본문 가져오기</text>
  <rect x="604" y="78" width="110" height="52" rx="8" class="sk-box-accent"/>
  <text x="659" y="101" text-anchor="middle" class="sk-label">MCP 서버</text>
  <text x="659" y="119" text-anchor="middle" class="sk-sub">Microsoft Learn</text>
  <path d="M586,59 L602,92" class="sk-line" marker-end="url(#csm-m1)"/>
  <path d="M586,151 L602,118" class="sk-line" marker-end="url(#csm-m1)"/>
</svg>
<figcaption>도구 호출 순서를 내가 정하지 않았다. 지침에 "문서를 먼저 검색하라"는 한 줄만 있었다.</figcaption>
</figure>

## 준비물

- 크레딧이 있는 Copilot Studio 환경([FAQ 에이전트 글](/posts/copilot-studio-faq-agent/)에서 크레딧 때문에 한 번 막혔다)
- 연결할 MCP 서버. Copilot Studio는 **Streamable HTTP 전송만** 지원하고, SSE는 2025년 8월 이후 지원하지 않는다([공식 문서](https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-add-existing-server-to-agent))

## 1. 에이전트와 지침

새 에이전트를 만들고 지침은 세 줄만 넣었다.

```text
너는 Microsoft 기술 질문에 답하는 도우미야.
답하기 전에 반드시 Microsoft Learn MCP 도구로 공식 문서를 검색해.
답은 한국어로 짧게 요약하고, 참고한 문서 링크를 함께 알려 줘.
```

## 2. 카탈로그에서 MCP 서버 고르기

Tools 옆 **+** 를 누르면 "Add a tool" 창이 뜬다. ① **MCP** 탭에 Microsoft가 미리 등록해 둔 서버들이 보인다. Outlook 메일·캘린더, SharePoint, Teams, Dataverse, Fabric IQ 등. ② 오른쪽 **+ Add**는 직접 등록할 때 쓴다(5번에서).

![Add a tool 창의 MCP 탭](/assets/img/posts/copilot-studio-mcp-tools/01-mcp-catalog.png)
_① MCP 탭 ② 내 서버를 직접 등록하는 + Add_

① 검색창에 "Learn"을 치니 ② **Microsoft Learn Docs MCP Server**가 나왔다.

![Learn으로 검색](/assets/img/posts/copilot-studio-mcp-tools/02-search.png)
_① 검색 ② Microsoft Learn Docs MCP Server_

![서버 상세에서 Add](/assets/img/posts/copilot-studio-mcp-tools/03-detail.png)
_① 상세 화면에서 Add_

Add를 누르면 **연결(Connection)** 을 만드는 창이 뜬다. 여기서 눈여겨볼 건 ① **Premium** 표시다. MCP 서버도 Power Platform 커넥터로 붙기 때문에, 고객사 환경에서는 프리미엄 커넥터 라이선스와 DLP 정책에 걸릴 수 있다. ② Create를 누르면 끝.

![연결 만들기 창](/assets/img/posts/copilot-studio-mcp-tools/04-connection.png)
_① Premium 커넥터 ② Create_

![Tools에 붙은 MCP 서버](/assets/img/posts/copilot-studio-mcp-tools/05-added.png)
_① Tools에 Microsoft Learn Docs MCP Server가 붙었다. 지식의 Search all websites는 지웠다_

## 3. 테스트

Preview에서 "Azure Functions에서 Durable Functions는 언제 쓰는 게 좋아?"라고 물었다. ① 에이전트가 검색 1번, 가져오기 2번을 스스로 이어서 불렀다. 생각 과정도 보인다. "검색 결과가 에이전트 패턴 위주라 개요 문서를 가져오겠다" 같은 판단이다.

![도구 호출 과정](/assets/img/posts/copilot-studio-mcp-tools/08-tool-calls.png)
_① Used tool: Microsoft Docs Search → Fetch → Fetch_

답 끝에는 ① 참고 문서 링크 ② Citations가 붙었다. 답 중간에 ".NET in‑process 모델 지원 종료가 한 달 남았다"는 경고까지 문서에서 찾아 넣은 게 인상적이었다.

![출처 링크가 달린 답](/assets/img/posts/copilot-studio-mcp-tools/09-answer-links.png)
_① learn.microsoft.com 링크 ② Citations_

## 4. 내 MCP 서버를 직접 등록하려면

카탈로그에 없는 서버는 **+ Add → Model Context Protocol (MCP)** 로 등록한다.

![+ Add 메뉴](/assets/img/posts/copilot-studio-mcp-tools/06-custom-menu.png)
_① Model Context Protocol (MCP)_

① 서버 이름·설명 ② 서버 URL ③ 인증(None / API key / OAuth 2.0)을 넣는다. 아래 화면은 예시 값만 넣어 본 것이고, 실제로 추가하지는 않았다.

![MCP 서버 직접 등록 양식](/assets/img/posts/copilot-studio-mcp-tools/07-custom-form.png)
_① 이름·설명 ② 서버 URL ③ 인증 방식_

<figure class="sketch">
<svg viewBox="0 0 720 180" role="img" aria-label="카탈로그 MCP는 클릭 몇 번이면 되고, 직접 등록은 URL과 인증을 직접 넣어야 하며 서버 설명이 도구 선택에 영향을 준다">
  <text x="0" y="14" class="sk-title">② 두 가지 연결 방법</text>
  <rect x="8" y="36" width="340" height="128" rx="8" class="sk-box"/>
  <text x="178" y="60" text-anchor="middle" class="sk-label">카탈로그(MCP 탭)</text>
  <text x="178" y="86" text-anchor="middle" class="sk-sub">검색 → Add → Create</text>
  <text x="178" y="106" text-anchor="middle" class="sk-sub">Microsoft 1st-party 서버 위주</text>
  <text x="178" y="126" text-anchor="middle" class="sk-sub">연결이 커넥터로 만들어짐(Premium)</text>
  <text x="178" y="146" text-anchor="middle" class="sk-sub">데모·사내 M365 연동에 적합</text>
  <rect x="372" y="36" width="340" height="128" rx="8" class="sk-box-accent"/>
  <text x="542" y="60" text-anchor="middle" class="sk-label">직접 등록(+ Add → MCP)</text>
  <text x="542" y="86" text-anchor="middle" class="sk-sub">이름·설명·URL·인증 입력</text>
  <text x="542" y="106" text-anchor="middle" class="sk-sub">Streamable HTTP만 (SSE 불가)</text>
  <text x="542" y="126" text-anchor="middle" class="sk-sub">설명 문장이 도구 선택에 영향</text>
  <text x="542" y="146" text-anchor="middle" class="sk-sub">사내 API를 MCP로 감싸 붙일 때</text>
</svg>
<figcaption>직접 등록할 땐 "설명"을 대충 쓰면 안 된다. 에이전트는 그 문장을 보고 도구를 고른다.</figcaption>
</figure>

## 비용·라이선스

- 연결 자체는 무료였지만, **테스트 대화는 Copilot 크레딧을 쓴다.** 도구 호출이 3번 일어나는 질문은 단순 답보다 크레딧이 더 든다고 보는 게 맞다
- 커넥터에 Premium이 붙는다. 고객사에선 라이선스와 DLP 정책을 먼저 확인하자

## 써 보고 든 생각

- MCP의 장점이 로우코드에서 더 크게 느껴졌다. 커스텀 커넥터를 OpenAPI로 정의하던 일이 **URL 하나 붙이는 일**로 줄었다
- 대신 통제는 에이전트가 가져간다. 몇 번 부를지 내가 정하지 않으니, **실제 운영 전에 Preview에서 호출 횟수와 답 품질을 여러 질문으로 꼭 확인**해야 한다
- 다음엔 [지난번 만든 메모 MCP 서버](/posts/memo-mcp-first-server/)를 Streamable HTTP로 바꿔서 직접 등록까지 해 볼 생각이다

## 참고

- [Connect your agent to an existing Model Context Protocol (MCP) server — Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-add-existing-server-to-agent)
- [Quotas and limits — Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas)
- 관련 글: [MCP와 API는 무엇이 다른가](/posts/mcp-vs-api/), [MCP 서버를 처음 만들어봤다](/posts/memo-mcp-first-server/)
