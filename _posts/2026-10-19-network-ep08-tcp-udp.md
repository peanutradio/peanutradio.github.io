---
title: "네트워크 기초 EP.08 — TCP와 UDP, 등기우편과 엽서의 차이"
date: 2026-10-19 08:09:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, lecture]
description: 확인 도장을 받는 등기우편 TCP와 넣으면 끝인 엽서 UDP를 순서 번호, 다시 보내기, 쓰임새로 비교하는 약 3분 강의와 정리.
---

**TCP는 확인하며 빠짐없이, UDP는 확인 없이 빠르게 보냅니다.** 이번 편은 인터넷이 데이터를 보내는 두 가지 방식을 등기우편과 엽서에 빗대어 나란히 달려 보게 합니다. 약 3분, 4개 챕터의 3D 강의예요.

> 소리를 켜고 보세요. 약 3분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='rbP6-HVVWDk' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> 택배를 보낼 때, 확실하게 보낼까요, 빨리 보낼까요?

[지난 편](/posts/network-ep07-port/)에서는 컴퓨터 안의 문 번호, 포트를 배웠습니다. 문을 찾아 들어간 다음에는 "어떻게 보낼지"를 골라야 합니다. 인터넷에도 확실한 방법과 빠른 방법, 두 가지가 있어요.

| 챕터 | 내용 |
|---|---|
| 1. 등기우편처럼 확인하며: TCP | 순서 번호, 받았다는 확인, 빠진 것만 다시 보내기 |
| 2. 엽서처럼 빠르게: UDP | 확인도 재전송도 없이 보내기만 |
| 3. 나란히 달려 보기 | 느리지만 빠짐없는 트럭 vs 빠르지만 가끔 빠지는 오토바이 |
| 4. 어디에 쓸까요? | 웹·파일·메시지는 TCP, 영상통화·게임·생방송은 UDP |

## 1. 등기우편처럼 확인하며: TCP

등기우편을 보내면 받는 사람이 도장을 찍어 줘야 배달이 끝납니다. **TCP**가 바로 그렇게 일합니다.

보낼 상자마다 1번, 2번, 3번 **순서 번호**를 붙입니다. 받는 쪽은 상자를 받을 때마다 "잘 받았어요" 하고 확인을 돌려줍니다. 그런데 중간에 상자 하나가 사라지면 어떻게 될까요? 그 상자에 대한 확인이 오지 않으니, 보내는 쪽은 **그 상자만 다시 보냅니다.** 받는 쪽은 번호를 보고 순서대로 다시 맞춥니다.

그래서 TCP는 조금 느려도 **빠짐없이, 순서대로** 도착합니다. 1편에서 본 3-way 핸드셰이크도 이렇게 확실하게 보내려는 TCP의 약속이었어요.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="TCP는 상자 1, 2, 3에 번호를 붙여 보내고 받는 쪽이 확인을 돌려준다. 2번 상자가 사라지면 확인이 오지 않아 2번만 다시 보낸다">
  <defs>
    <marker id="ep8-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep8-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① TCP: 번호 붙이고, 확인받고, 빠진 것만 다시</text>
  <rect x="10" y="36" width="110" height="180" rx="8" class="sk-box"/>
  <text x="65" y="60" text-anchor="middle" class="sk-label">보내는 쪽</text>
  <rect x="600" y="36" width="110" height="180" rx="8" class="sk-box"/>
  <text x="655" y="60" text-anchor="middle" class="sk-label">받는 쪽</text>
  <path d="M122,80 L598,80" class="sk-line" marker-end="url(#ep8-m1)"/>
  <text x="360" y="74" text-anchor="middle" class="sk-mono">상자 1</text>
  <path d="M598,98 L122,98" class="sk-line" marker-end="url(#ep8-m1)"/>
  <text x="360" y="112" text-anchor="middle" class="sk-sub">"1번 잘 받았어요"</text>
  <path d="M122,134 L340,134" class="sk-line" marker-end="url(#ep8-m1)"/>
  <text x="230" y="128" text-anchor="middle" class="sk-mono">상자 2</text>
  <rect x="345" y="122" width="90" height="24" rx="6" class="sk-box-muted"/>
  <text x="390" y="139" text-anchor="middle" class="sk-sub">중간에 사라짐</text>
  <text x="520" y="139" text-anchor="middle" class="sk-sub">확인이 안 온다</text>
  <path d="M122,182 L598,182" class="sk-line-accent" marker-end="url(#ep8-a1)"/>
  <text x="360" y="176" text-anchor="middle" class="sk-mono">상자 2 다시 보내기</text>
  <text x="360" y="206" text-anchor="middle" class="sk-sub">받는 쪽은 번호를 보고 1 → 2 → 3 순서대로 맞춘다</text>
</svg>
<figcaption>TCP가 느린 이유와 믿을 만한 이유는 같다 — 하나하나 확인을 받기 때문이다.</figcaption>
</figure>

## 2. 엽서처럼 빠르게: UDP

엽서는 어떨까요? 우체통에 넣으면 끝입니다. 도착했는지 확인하지 않아요.

**UDP**가 이렇게 보냅니다. 확인도, 다시 보내기도 없이 계속 보내기만 하죠. 가끔 한 장쯤 사라져도 그냥 넘어갑니다. 대신 기다리고 확인하는 과정이 없으니 **훨씬 빠르고 가볍습니다.**

## 3. 나란히 달려 보기

같은 길을 나란히 달려 보면 차이가 확 보입니다. TCP 트럭은 확인하느라 조금 느리지만 하나도 빠짐없이 도착합니다. UDP 오토바이는 확인 없이 쌩쌩 달려서 먼저 도착합니다. 다만 짐 한두 개가 빠질 수는 있어요.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="같은 길에서 TCP 트럭은 확인하며 느리게 가지만 짐이 모두 도착하고, UDP 오토바이는 먼저 도착하지만 짐 하나가 빠질 수 있다">
  <defs>
    <marker id="ep8-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep8-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 같은 길, 다른 성격</text>
  <rect x="10" y="44" width="120" height="44" rx="8" class="sk-box"/>
  <text x="70" y="64" text-anchor="middle" class="sk-label">TCP 트럭</text>
  <text x="70" y="80" text-anchor="middle" class="sk-sub">등기우편</text>
  <path d="M132,66 L470,66" class="sk-line" marker-end="url(#ep8-m2)"/>
  <text x="300" y="58" text-anchor="middle" class="sk-sub">확인 · 확인 · 확인 …</text>
  <rect x="480" y="44" width="230" height="44" rx="8" class="sk-box"/>
  <text x="595" y="64" text-anchor="middle" class="sk-label">조금 늦게 도착</text>
  <text x="595" y="80" text-anchor="middle" class="sk-mono">1 2 3 4 5 전부</text>
  <rect x="10" y="118" width="120" height="44" rx="8" class="sk-box-accent"/>
  <text x="70" y="138" text-anchor="middle" class="sk-label">UDP 오토바이</text>
  <text x="70" y="154" text-anchor="middle" class="sk-sub">엽서</text>
  <path d="M132,140 L470,140" class="sk-line-accent" marker-end="url(#ep8-a2)"/>
  <text x="300" y="132" text-anchor="middle" class="sk-sub">확인 없이 쌩쌩</text>
  <rect x="480" y="118" width="230" height="44" rx="8" class="sk-box"/>
  <text x="595" y="138" text-anchor="middle" class="sk-label">먼저 도착</text>
  <text x="595" y="154" text-anchor="middle" class="sk-mono">1 2 _ 4 5</text>
  <text x="360" y="190" text-anchor="middle" class="sk-sub">빠짐없음이 중요하면 트럭, 지금 이 순간이 중요하면 오토바이</text>
</svg>
<figcaption>어느 쪽이 더 좋은 게 아니라, 무엇을 보내느냐에 따라 맞는 쪽이 다르다.</figcaption>
</figure>

## 4. 어디에 쓸까요?

그럼 언제 무엇을 쓸까요?

- **TCP**: 웹페이지, 파일, 메시지처럼 한 글자라도 빠지면 안 되는 것
- **UDP**: 영상통화, 게임, 생방송처럼 조금 빠져도 빨리 가는 게 중요한 것

영상통화 화면이 잠깐 깨지거나 끊기는 건, 사라진 엽서를 다시 보내지 않고 그냥 넘어갔기 때문입니다. 1초 전 화면을 다시 받아 봐야 소용없으니, 차라리 다음 화면으로 넘어가는 편이 낫거든요.

## 한 줄 정리

> **TCP**는 등기우편처럼 확인하며 빠짐없이, **UDP**는 엽서처럼 확인 없이 빠르게.
{: .prompt-info }

## 퀴즈

1. 한 글자도 빠지면 안 되는 웹페이지는? (TCP / UDP)
2. 영상통화나 게임에 더 잘 맞는 것은? (TCP / UDP)

<details markdown="1">
<summary>정답 보기</summary>

1. **TCP** — 확인하고, 빠진 건 다시 보내니까요.
2. **UDP** — 조금 빠져도 빨리 가는 게 더 중요해요.

</details>

## 집에서 해보기: UDP로 열린 문 찾아보기

명령 창(Windows: 명령 프롬프트, Mac: 터미널)에 아래 명령을 입력해 보세요.

```bash
# Windows
netstat -an -p UDP

# Mac
netstat -an -p udp
```

UDP 줄에는 LISTENING 같은 상태가 없습니다. 확인 없이 보내는 엽서라서요. 그리고 영상통화가 끊길 때 왜 그런지, 오늘 배운 말로 한번 설명해 보세요.

## 실무에서 달라지는 점

- **방화벽 규칙은 TCP와 UDP를 따로 연다.** 같은 포트 번호라도 프로토콜이 다르면 다른 문입니다. Azure NSG나 Windows 방화벽 규칙에서 프로토콜 칸을 잘못 고르면 "포트는 열었는데 안 된다"는 상황이 생깁니다.
- **Teams 통화 품질 문제는 UDP 경로부터 본다.** Microsoft의 네트워크 가이드도 실시간 미디어에 UDP를 권장하고, UDP가 막히면 TCP로 우회하면서 품질이 떨어질 수 있습니다. 회사 방화벽에서 UDP가 열려 있는지가 첫 확인 대상입니다.
- **DNS는 UDP의 대표 손님이다.** 짧은 질문 하나, 짧은 답 하나라 확인 절차 없이 빠르게 주고받는 UDP가 잘 맞습니다. DNS 장애를 볼 때 포트 53을 TCP와 UDP 둘 다 확인하는 이유가 여기 있습니다.

## 시리즈 다른 편

- [네트워크 기초 EP.07 — 포트, 한 집에 문이 여러 개](/posts/network-ep07-port/)
- 네트워크 기초 EP.09 — 연결을 맺고 끊기, 전화 예절로 배우는 TCP (발행 예정)
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
