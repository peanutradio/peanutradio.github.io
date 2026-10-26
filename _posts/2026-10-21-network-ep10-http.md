---
title: "네트워크 기초 EP.10 — HTTP, 웹이 대화하는 법"
date: 2026-10-21 07:55:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, devops, lecture]
description: 식당 주문서처럼 요청하고 접시처럼 응답하는 HTTP. 요청과 응답, GET과 POST, 상태 코드 200·404·500을 따라가는 약 2분 강의와 정리.
---

**HTTP는 웹의 주문서 약속입니다. 브라우저가 요청하면 서버가 응답하고, 응답에는 200, 404 같은 상태 코드가 붙습니다.** 이번 편은 연결이 열린 뒤 브라우저와 웹 서버가 실제로 무슨 말을 주고받는지를 식당 주문에 빗대어 봅니다. 약 2분, 4개 챕터의 3D 강의예요.

> 소리를 켜고 보세요. 약 2분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='ZQ9IRRFfq60' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> 연결이 열렸으면, 브라우저와 웹 서버는 무슨 말로 대화할까요?

[지난 편](/posts/network-ep09-tcp-connection/)에서는 TCP가 연결을 맺고 끊는 전화 예절을 배웠습니다. 전화가 연결됐으니 이제 용건을 말할 차례입니다. 웹에서 그 용건을 말하는 방식이 HTTP예요.

| 챕터 | 내용 |
|---|---|
| 1. 식당 주문서로 생각해 봐요 | 주문서 한 장에 접시 하나 |
| 2. 웹의 주문서: 요청과 응답 | 브라우저는 요청, 서버는 응답 |
| 3. 주문의 종류: GET과 POST | "보여 주세요"와 "받아 주세요" |
| 4. 응답의 번호표: 상태 코드 | 200 · 404 · 500 |

## 1. 식당 주문서로 생각해 봐요

식당에서는 주문서에 먹고 싶은 걸 적어서 주방에 건넵니다. 주방은 주문서를 보고 음식을 만들어서 접시에 담아 내보내죠. **주문서 한 장에 접시 하나.** 묻고, 답하고. 아주 단순합니다.

## 2. 웹의 주문서: 요청과 응답

웹도 똑같습니다. 브라우저가 웹 서버에게 주문서를 보내는데, 이걸 **요청**(Request)이라고 합니다. 웹 서버는 웹페이지를 접시에 담아 돌려주고, 이게 **응답**(Response)입니다.

이 주문서를 쓰는 약속이 바로 **HTTP**입니다. 2편에서 본 프로토콜 중 하나죠. 정해진 형식으로 적어야 주방이 알아듣는 것처럼, 브라우저와 서버도 HTTP라는 형식에 맞춰 말해야 서로 알아듣습니다.

<figure class="sketch">
<svg viewBox="0 0 720 210" role="img" aria-label="브라우저가 HTTP 요청이라는 주문서를 보내면 웹 서버가 웹페이지와 상태 코드를 담은 응답을 돌려준다">
  <defs>
    <marker id="ep10-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep10-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 주문서 한 장, 접시 하나</text>
  <rect x="10" y="60" width="140" height="80" rx="8" class="sk-box"/>
  <text x="80" y="96" text-anchor="middle" class="sk-label">브라우저</text>
  <text x="80" y="114" text-anchor="middle" class="sk-sub">손님</text>
  <rect x="570" y="60" width="140" height="80" rx="8" class="sk-box"/>
  <text x="640" y="96" text-anchor="middle" class="sk-label">웹 서버</text>
  <text x="640" y="114" text-anchor="middle" class="sk-sub">주방</text>
  <path d="M152,78 L568,78" class="sk-line-accent" marker-end="url(#ep10-a1)"/>
  <rect x="270" y="40" width="180" height="26" rx="6" class="sk-box-accent"/>
  <text x="360" y="58" text-anchor="middle" class="sk-mono">요청: GET /index.html</text>
  <path d="M568,122 L152,122" class="sk-line" marker-end="url(#ep10-m1)"/>
  <rect x="270" y="134" width="180" height="40" rx="6" class="sk-box"/>
  <text x="360" y="151" text-anchor="middle" class="sk-mono">응답: 200</text>
  <text x="360" y="166" text-anchor="middle" class="sk-sub">+ 웹페이지 접시</text>
  <text x="360" y="200" text-anchor="middle" class="sk-sub">말을 먼저 거는 쪽은 언제나 브라우저</text>
</svg>
<figcaption>웹은 손님이 주문해야 접시가 나오는 구조다 — 서버가 먼저 페이지를 보내 주지 않는다.</figcaption>
</figure>

## 3. 주문의 종류: GET과 POST

주문서에도 종류가 있습니다. 가장 많이 쓰는 건 **GET**과 **POST**예요.

- **GET** — "보여 주세요." 웹페이지나 사진을 가져올 때 씁니다.
- **POST** — "이거 받아 주세요." 로그인하거나 글을 올릴 때처럼, 내 쪽에서 내용을 보낼 때 씁니다.

주소창에 주소를 치고 엔터를 누르는 건 GET, 로그인 버튼을 누르는 건 대개 POST라고 생각하면 쉽습니다.

## 4. 응답의 번호표: 상태 코드

응답에는 번호표가 붙어 옵니다. 이걸 **상태 코드**라고 해요.

<figure class="sketch">
<svg viewBox="0 0 720 150" role="img" aria-label="상태 코드 200은 잘 나왔다, 404는 그런 메뉴가 없다, 500은 주방에 문제가 생겼다는 뜻이다">
  <text x="0" y="14" class="sk-title">② 접시에 붙는 번호표</text>
  <rect x="10" y="36" width="220" height="96" rx="8" class="sk-box-accent"/>
  <text x="120" y="70" text-anchor="middle" class="sk-mono">200</text>
  <text x="120" y="94" text-anchor="middle" class="sk-label">잘 나왔어요</text>
  <text x="120" y="114" text-anchor="middle" class="sk-sub">주문한 페이지 도착</text>
  <rect x="250" y="36" width="220" height="96" rx="8" class="sk-box"/>
  <text x="360" y="70" text-anchor="middle" class="sk-mono">404</text>
  <text x="360" y="94" text-anchor="middle" class="sk-label">그런 메뉴는 없어요</text>
  <text x="360" y="114" text-anchor="middle" class="sk-sub">없는 주소를 찾음</text>
  <rect x="490" y="36" width="220" height="96" rx="8" class="sk-box"/>
  <text x="600" y="70" text-anchor="middle" class="sk-mono">500</text>
  <text x="600" y="94" text-anchor="middle" class="sk-label">주방에 문제가 생겼어요</text>
  <text x="600" y="114" text-anchor="middle" class="sk-sub">서버 쪽에 탈이 남</text>
</svg>
<figcaption>번호 하나만 봐도 문제가 손님 쪽 주문인지, 주방 쪽 사정인지 가를 수 있다.</figcaption>
</figure>

- **200** — 잘 나왔어요. 주문한 페이지가 무사히 왔다는 뜻입니다.
- **404** — 그런 메뉴는 없어요. 없는 주소를 찾았을 때 자주 보죠.
- **500** — 주방에 문제가 생겼어요. 서버 쪽에 탈이 났다는 뜻입니다.

## 한 줄 정리

> HTTP는 웹의 **주문서 약속**. **요청** 보내고 **응답** 받기, 응답에는 200 · 404 · 500 같은 번호표.
{: .prompt-info }

## 퀴즈

1. 웹페이지를 가져올 때 쓰는 주문은? (GET / POST)
2. 없는 주소를 찾으면 나오는 번호는? (200 / 404)

<details markdown="1">
<summary>정답 보기</summary>

1. **GET** — GET은 "보여 주세요", POST는 "받아 주세요"예요.
2. **404** — "그런 메뉴는 없어요"라는 뜻이에요.

</details>

## 집에서 해보기: 웹페이지의 주문서 직접 보기

크롬이나 엣지에서 개발자 도구를 열고 Network 탭을 연 다음 새로고침해 보세요.

```text
# Windows (크롬 · 엣지)
F12 → Network 탭 → 새로고침

# Mac (크롬)
Cmd + Option + I → Network 탭 → 새로고침
```

페이지 하나를 여는 데 주문서가 수십 장 오가는 게 보입니다. Status 칸이 상태 코드예요. 200이 아닌 번호가 섞여 있는지도 찾아보세요.

## 실무에서 달라지는 점

- **API도 결국 HTTP다.** Microsoft Graph나 Azure REST API를 호출하는 것도 GET·POST 주문서를 보내는 일입니다. Power Automate나 Copilot Studio에서 HTTP 액션을 쓸 때 메서드 칸에 무엇을 고를지가 이번 편 내용 그대로예요.
- **4xx와 5xx를 나눠 보면 원인 찾기가 빨라진다.** 400번대는 요청하는 쪽, 500번대는 서버 쪽 문제로 보는 게 출발점입니다. 권한이 없을 때 나오는 401·403도 같은 400번대예요.
- **Network 탭은 현업 진단 도구다.** 사용자가 "페이지가 안 떠요"라고 할 때 F12의 Network 탭 캡처를 받으면 어느 요청이 몇 번 코드로 실패했는지 바로 보입니다.

## 시리즈 다른 편

- [네트워크 기초 EP.09 — 연결을 맺고 끊기, 전화 예절로 배우는 TCP](/posts/network-ep09-tcp-connection/)
- 네트워크 기초 EP.11 — HTTPS, 주소창 자물쇠의 비밀 (발행 예정)
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
