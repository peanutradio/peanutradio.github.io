---
title: "네트워크 기초 EP.11 — HTTPS, 주소창 자물쇠의 비밀"
date: 2026-10-22 07:55:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, security, lecture]
description: 엽서처럼 다 보이던 HTTP에 자물쇠 상자(암호화)와 신분증(인증서)을 더한 HTTPS. 열쇠를 안전하게 나누는 방법까지 따라가는 약 2분 강의와 정리.
---

**HTTPS는 HTTP에 자물쇠 상자(암호화)와 신분증(인증서)을 더한 것입니다.** 이번 편은 주소창의 작은 자물쇠 아이콘 하나에 무엇이 들어 있는지, 상자와 열쇠로 풀어 봅니다. 약 2분, 4개 챕터의 3D 강의예요.

> 소리를 켜고 보세요. 약 2분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='HeFn8s9Fkyo' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> 주소창의 작은 자물쇠, 무슨 뜻일까요? 내 비밀번호는 안전하게 가고 있을까요?

[지난 편](/posts/network-ep10-http/)에서는 웹의 주문서 약속, HTTP를 배웠습니다. 그런데 그 주문서에 로그인 비밀번호가 적혀 있다면 어떨까요? 이번 편은 그 주문서를 어떻게 지키는지에 대한 이야기입니다.

| 챕터 | 내용 |
|---|---|
| 1. 엽서처럼 다 보여요 | HTTP 주문서는 중간에서 다 읽힌다 |
| 2. 자물쇠 상자: 암호화 | 엿봐도 뒤죽박죽, 열쇠가 있어야 열린다 |
| 3. 열쇠를 안전하게 나누기 | 열린 자물쇠로 비밀 열쇠를 잠가 보내기 |
| 4. 진짜 서버 맞나요? 인증서 | 믿을 수 있는 기관이 도장 찍은 신분증 |

## 1. 엽서처럼 다 보여요

HTTP 주문서는 엽서처럼 내용이 다 보입니다. 중간에 누가 몰래 엿보면 내 비밀번호까지 그대로 읽힐 수 있어요. 공용 와이파이처럼 같은 길을 여러 사람이 쓰는 곳이라면 더 신경 쓰이는 문제죠.

## 2. 자물쇠 상자: 암호화

그래서 내용을 자물쇠 상자에 넣어서 보냅니다. 이걸 **암호화**라고 해요. 중간에서 엿봐도 뒤죽박죽 글자만 보입니다. 열쇠가 있는 사람만 열 수 있거든요.

그런데 문제가 하나 있습니다. **열쇠는 어떻게 서로 나눠 가질까요?** 열쇠를 그냥 보내면 그 열쇠도 엽서처럼 다 보일 테니까요.

## 3. 열쇠를 안전하게 나누기

여기가 이번 편에서 가장 재미있는 부분입니다.

1. 서버는 먼저 **열린 자물쇠**를 보내 줍니다. 누구나 이 자물쇠로 잠글 수는 있지만, 여는 열쇠는 서버만 가지고 있어요.
2. 내 컴퓨터는 둘만 쓸 **비밀 열쇠**를 상자에 넣고, 받은 자물쇠로 잠가서 보냅니다.
3. 그 상자는 서버만 열 수 있으니, 이제 **둘만 같은 비밀 열쇠**를 갖게 됩니다.
4. 그다음부터는 이 비밀 열쇠로 모든 대화를 잠가서 주고받습니다.

<figure class="sketch">
<svg viewBox="0 0 720 260" role="img" aria-label="서버가 누구나 잠글 수 있는 열린 자물쇠를 보내면, 내 컴퓨터가 비밀 열쇠를 그 자물쇠로 잠가 보내고, 이후 둘은 같은 비밀 열쇠로 대화를 잠근다">
  <defs>
    <marker id="ep11-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep11-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 열쇠를 엿보이지 않게 나누는 법</text>
  <rect x="10" y="36" width="130" height="200" rx="8" class="sk-box"/>
  <text x="75" y="60" text-anchor="middle" class="sk-label">내 컴퓨터</text>
  <rect x="580" y="36" width="130" height="200" rx="8" class="sk-box"/>
  <text x="645" y="60" text-anchor="middle" class="sk-label">서버</text>
  <text x="645" y="78" text-anchor="middle" class="sk-sub">여는 열쇠는 여기만</text>
  <path d="M578,98 L142,98" class="sk-line" marker-end="url(#ep11-m1)"/>
  <text x="360" y="92" text-anchor="middle" class="sk-sub">1 · 열린 자물쇠 (누구나 잠글 수 있음)</text>
  <path d="M142,140 L578,140" class="sk-line" marker-end="url(#ep11-m1)"/>
  <rect x="270" y="112" width="180" height="22" rx="6" class="sk-box-muted"/>
  <text x="360" y="128" text-anchor="middle" class="sk-sub">2 · 비밀 열쇠를 넣고 잠근 상자</text>
  <text x="360" y="158" text-anchor="middle" class="sk-sub">중간에서 가로채도 열 수 없다</text>
  <rect x="200" y="176" width="320" height="26" rx="6" class="sk-box-accent"/>
  <text x="360" y="194" text-anchor="middle" class="sk-label">3 · 이제 둘만 같은 비밀 열쇠</text>
  <path d="M142,220 L578,220" class="sk-line-accent" marker-start="url(#ep11-a1)" marker-end="url(#ep11-a1)"/>
  <text x="360" y="244" text-anchor="middle" class="sk-sub">4 · 이후 모든 대화를 이 열쇠로 잠가 주고받는다</text>
</svg>
<figcaption>열쇠를 보내지 않고 "잠그는 도구"만 공개하면, 엿보는 사람이 있어도 비밀 열쇠를 나눌 수 있다.</figcaption>
</figure>

## 4. 진짜 서버 맞나요? 인증서

그런데 상대가 정말 그 서버인지는 어떻게 알까요? 가짜 서버가 자기 자물쇠를 내밀면 소용이 없으니까요. 그래서 서버는 **신분증**을 보여 줍니다. 이걸 **인증서**라고 해요.

인증서는 믿을 수 있는 기관이 도장을 찍어 준 것이라서, 가짜는 바로 티가 납니다. 암호화와 인증서 확인을 모두 통과하면 주소창에 자물쇠가 뜹니다. 이게 **HTTPS**예요. 끝의 S는 안전하다(Secure)는 뜻입니다.

<figure class="sketch">
<svg viewBox="0 0 720 130" role="img" aria-label="HTTPS는 HTTP에 암호화와 인증서를 더한 것이다">
  <text x="0" y="14" class="sk-title">② HTTPS = HTTP + 두 가지</text>
  <rect x="10" y="40" width="160" height="70" rx="8" class="sk-box-muted"/>
  <text x="90" y="72" text-anchor="middle" class="sk-label">HTTP</text>
  <text x="90" y="90" text-anchor="middle" class="sk-sub">엽서 같은 주문서</text>
  <text x="195" y="80" text-anchor="middle" class="sk-label">+</text>
  <rect x="220" y="40" width="160" height="70" rx="8" class="sk-box"/>
  <text x="300" y="72" text-anchor="middle" class="sk-label">암호화</text>
  <text x="300" y="90" text-anchor="middle" class="sk-sub">자물쇠 상자</text>
  <text x="405" y="80" text-anchor="middle" class="sk-label">+</text>
  <rect x="430" y="40" width="160" height="70" rx="8" class="sk-box"/>
  <text x="510" y="72" text-anchor="middle" class="sk-label">인증서</text>
  <text x="510" y="90" text-anchor="middle" class="sk-sub">믿을 만한 신분증</text>
  <text x="615" y="80" text-anchor="middle" class="sk-label">=</text>
  <rect x="640" y="40" width="70" height="70" rx="8" class="sk-box-accent"/>
  <text x="675" y="80" text-anchor="middle" class="sk-mono">HTTPS</text>
</svg>
<figcaption>자물쇠 아이콘은 "내용이 숨겨졌다"와 "상대가 진짜다", 두 가지를 동시에 말한다.</figcaption>
</figure>

## 한 줄 정리

> HTTPS = HTTP + **자물쇠 상자**(암호화) + **신분증**(인증서).
{: .prompt-info }

## 퀴즈

1. 중간에서 엿봐도 못 읽게 만드는 것은? (암호화 / 캐시)
2. 서버가 진짜인지 보여 주는 신분증은? (인증서 / 주문서)

<details markdown="1">
<summary>정답 보기</summary>

1. **암호화** — 내용을 자물쇠 상자에 넣는 거예요.
2. **인증서** — 믿을 수 있는 기관이 도장을 찍어 준 신분증이에요.

</details>

## 집에서 해보기: 사이트의 신분증(인증서) 보기

자주 가는 사이트에서 주소창 왼쪽 자물쇠를 눌러 인증서를 열어 보세요.

```text
# Windows (크롬 · 엣지 주소창)
자물쇠 → 인증서

# Mac (크롬 · 사파리 주소창)
자물쇠 → 인증서 보기
```

누가 발급했는지(발급자)와 언제까지 쓸 수 있는지(유효 기간)를 찾아보세요. 사이트마다 발급자가 다른 것도 확인할 수 있습니다.

## 실무에서 달라지는 점

- **인증서는 만료된다.** 유효 기간이 지나면 브라우저가 경고를 띄우고 접속이 막힙니다. 사내 웹앱이나 Azure App Service에 붙인 사용자 지정 도메인 인증서는 만료일을 관리 대상으로 두고, 가능하면 자동 갱신을 씁니다.
- **회사 네트워크에서는 인증서 발급자가 다르게 보일 수 있다.** 보안 장비가 HTTPS 트래픽을 검사하도록 구성된 환경에서는 발급자가 원래 기관이 아니라 회사 장비로 표시되기도 합니다. "인증서 오류가 특정 망에서만 난다"면 이 가능성을 먼저 봅니다.
- **자물쇠는 "통신이 안전하다"이지 "사이트가 믿을 만하다"가 아니다.** 피싱 사이트도 인증서를 받을 수 있습니다. 보안 교육에서 자물쇠만 보고 안심하지 말라고 하는 이유이고, 주소 자체를 확인하는 습관이 함께 필요합니다.

## 시리즈 다른 편

- [네트워크 기초 EP.10 — HTTP, 웹이 대화하는 법](/posts/network-ep10-http/)
- 네트워크 기초 EP.12 — DHCP, 이사 오면 주소를 자동으로 (발행 예정)
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
