---
title: "네트워크 기초 EP.02 — 네트워크가 뭘까? 컴퓨터끼리 말하는 법"
date: 2026-10-10 21:40:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, lan, wan, internet, protocol, lecture]
description: 실 전화기에서 시작해 우리 집 LAN, 동네와 도시를 잇는 WAN, 네트워크의 네트워크인 인터넷, 그리고 프로토콜까지 3D로 따라가는 약 4분 강의와 정리.
---

**네트워크는 약속대로 이어진 기기들이고, 인터넷은 그런 네트워크들의 네트워크입니다.** 이번 편은 이 한 문장을 실 전화기 하나로 시작해서 온 세상까지 넓혀 가며 보여 줍니다. 약 4분, 4개 챕터의 3D 강의예요.

> 소리를 켜고 보세요. 약 4분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='tboae5f_8as' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> 네트워크가 뭘까요? 컴퓨터끼리는 어떻게 말을 주고받을까요?

[지난 1편](/posts/tcpip-3d-lecture/)에서는 웹페이지 하나가 열릴 때 데이터가 네 개의 층을 지나간다는 걸 봤습니다. 이번에는 한 발 물러서서, 그 데이터가 오가는 "길" 자체가 무엇인지부터 정리합니다.

| 챕터 | 내용 |
|---|---|
| 1. 실 전화기로 말하기 | 두 대만 이어져도 네트워크 |
| 2. 우리 집 네트워크: LAN | 공유기를 가운데 두는 이유, 와이파이 |
| 3. 동네에서 온 세상으로: WAN과 인터넷 | 통신사, WAN, 네트워크의 네트워크 |
| 4. 약속이 필요해: 프로토콜 | 이어져 있어도 약속이 없으면 대화가 안 된다 |

## 1. 실 전화기로 말하기

어릴 때 종이컵 두 개를 실로 이어 만든 실 전화기를 떠올려 보세요. 한쪽에서 말하면 목소리가 실을 타고 건너가 반대쪽 컵에서 들립니다. 컴퓨터도 똑같습니다. 두 컴퓨터를 선으로 이으면 실 대신 선을 타고 신호가 오갑니다.

이렇게 **기기들이 이어져서 서로 말을 주고받는 것**, 이것을 네트워크라고 부릅니다. 딱 두 대만 이어져도 이미 작은 네트워크입니다.

## 2. 우리 집 네트워크: LAN

우리 집에는 컴퓨터, 노트북, 휴대폰, TV, 게임기까지 기기가 정말 많습니다. 기기마다 서로 실을 하나씩 이으면 금방 엉켜 버리죠. 그래서 집 가운데에 **공유기**를 둡니다. 모든 기기가 공유기 하나에만 이어지면 끝입니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="기기 다섯 대를 서로 모두 연결하면 선이 열 개로 엉키지만, 공유기를 가운데 두면 선 다섯 개로 정리된다">
  <text x="0" y="14" class="sk-title">① 서로 다 잇기 vs 가운데에 공유기</text>
  <g>
    <path d="M170,50 L250,110 M170,50 L220,190 M170,50 L120,190 M170,50 L90,110 M250,110 L220,190 M250,110 L120,190 M250,110 L90,110 M220,190 L120,190 M220,190 L90,110 M120,190 L90,110" class="sk-line"/>
    <circle cx="170" cy="50" r="16" class="sk-box"/><circle cx="250" cy="110" r="16" class="sk-box"/><circle cx="220" cy="190" r="16" class="sk-box"/><circle cx="120" cy="190" r="16" class="sk-box"/><circle cx="90" cy="110" r="16" class="sk-box"/>
    <text x="170" y="214" text-anchor="middle" class="sk-sub">5대 → 선 10개, 한 대 늘 때마다 더 엉킨다</text>
  </g>
  <g>
    <path d="M540,120 L540,50 M540,120 L620,110 M540,120 L590,190 M540,120 L490,190 M540,120 L460,110" class="sk-line-accent"/>
    <circle cx="540" cy="50" r="16" class="sk-box"/><circle cx="620" cy="110" r="16" class="sk-box"/><circle cx="590" cy="190" r="16" class="sk-box"/><circle cx="490" cy="190" r="16" class="sk-box"/><circle cx="460" cy="110" r="16" class="sk-box"/>
    <rect x="510" y="106" width="60" height="28" rx="6" class="sk-box-accent"/><text x="540" y="125" text-anchor="middle" class="sk-sub">공유기</text>
    <text x="540" y="214" text-anchor="middle" class="sk-sub">5대 → 선 5개, 기기가 늘어도 1개씩만</text>
  </g>
</svg>
<figcaption>가운데 장비 하나에 모으면, 기기가 늘어도 연결은 단순하게 유지된다.</figcaption>
</figure>

와이파이는 **눈에 보이지 않는 실**입니다. 선 대신 전파로 공유기와 이어질 뿐, 구조는 같습니다. 이렇게 집이나 학교처럼 가까운 곳의 네트워크를 **LAN**(Local Area Network, 근거리 네트워크)이라고 부릅니다.

## 3. 동네에서 온 세상으로: WAN과 인터넷

우리 집 공유기는 집 밖으로도 이어져 있습니다. 옆집도, 앞집도 마찬가지입니다. 동네의 집들은 모두 **통신사**로 이어지는데, 통신사는 동네의 큰길 같은 곳입니다. 통신사는 다시 다른 동네, 다른 도시와 길게 이어집니다. 이렇게 멀리까지 넓게 이어진 네트워크를 **WAN**(Wide Area Network, 광역 네트워크)이라고 합니다. 1편에서 본 웹 서버도 사실은 저 멀리 다른 동네에 있었던 거예요.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="집 안의 LAN들이 통신사로 모이고, 통신사들이 WAN으로 이어져 인터넷이라는 네트워크의 네트워크가 된다">
  <defs>
    <marker id="ep2-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② LAN이 모여 WAN, WAN이 모여 인터넷</text>
  <rect x="10" y="40" width="120" height="40" rx="8" class="sk-box"/><text x="70" y="65" text-anchor="middle" class="sk-label">우리 집 LAN</text>
  <rect x="10" y="90" width="120" height="40" rx="8" class="sk-box"/><text x="70" y="115" text-anchor="middle" class="sk-label">옆집 LAN</text>
  <rect x="10" y="140" width="120" height="40" rx="8" class="sk-box"/><text x="70" y="165" text-anchor="middle" class="sk-label">학교 LAN</text>
  <rect x="210" y="85" width="120" height="50" rx="8" class="sk-box-accent"/><text x="270" y="108" text-anchor="middle" class="sk-label">통신사</text><text x="270" y="124" text-anchor="middle" class="sk-sub">동네의 큰길</text>
  <path d="M132,60 L208,100" class="sk-line" marker-end="url(#ep2-m1)"/>
  <path d="M132,110 L208,110" class="sk-line" marker-end="url(#ep2-m1)"/>
  <path d="M132,160 L208,122" class="sk-line" marker-end="url(#ep2-m1)"/>
  <path d="M332,110 L408,110" class="sk-line-accent" marker-end="url(#ep2-m1)"/>
  <text x="370" y="100" text-anchor="middle" class="sk-sub">WAN</text>
  <rect x="410" y="40" width="300" height="140" rx="10" class="sk-box-muted"/>
  <text x="560" y="64" text-anchor="middle" class="sk-label">인터넷</text>
  <text x="560" y="82" text-anchor="middle" class="sk-sub">네트워크들의 네트워크</text>
  <rect x="430" y="100" width="80" height="30" rx="6" class="sk-box"/><text x="470" y="120" text-anchor="middle" class="sk-sub">다른 도시</text>
  <rect x="520" y="100" width="80" height="30" rx="6" class="sk-box"/><text x="560" y="120" text-anchor="middle" class="sk-sub">해외</text>
  <rect x="610" y="100" width="80" height="30" rx="6" class="sk-box"/><text x="650" y="120" text-anchor="middle" class="sk-sub">웹 서버</text>
  <text x="560" y="160" text-anchor="middle" class="sk-sub">수많은 네트워크가 서로 이어진 거대한 그물</text>
</svg>
<figcaption>인터넷은 아주 큰 컴퓨터 한 대가 아니라, 작은 네트워크들이 서로 이어진 그물이다.</figcaption>
</figure>

이걸 전 세계로 넓히면 수많은 네트워크가 서로 이어진 거대한 그물이 됩니다. 이게 바로 **인터넷**이고, 그래서 인터넷을 "네트워크들의 네트워크"라고 부릅니다.

## 4. 약속이 필요해: 프로토콜

그런데 이어져 있기만 하면 대화가 될까요? 한쪽은 한국말로, 다른 쪽은 처음 듣는 말로 대답하면 서로 알아들을 수 없습니다. 그래서 컴퓨터들은 미리 약속을 정해 둡니다.

- 어떤 말(형식)로 할지
- 누가 먼저 말할지
- 다 끝나면 어떻게 알릴지

이런 컴퓨터끼리의 약속을 **프로토콜**이라고 부릅니다. 1편에서 본 TCP와 IP, 그리고 웹에서 쓰는 HTTP도 모두 프로토콜입니다. 1편의 "3-way 핸드셰이크"가 바로 "누가 먼저 말할지"를 정한 약속의 예입니다.

## 한 줄 정리

> 네트워크는 **약속대로 이어진 기기들**, 인터넷은 **네트워크의 네트워크**.
{: .prompt-info }

## 퀴즈

1. 우리 집 안의 작은 네트워크는? (LAN / WAN)
2. 인터넷은 무엇일까요? (아주 큰 컴퓨터 한 대 / 네트워크들이 이어진 네트워크)

<details markdown="1">
<summary>정답 보기</summary>

1. **LAN** — 집·학교처럼 가까운 곳의 네트워크예요.
2. **네트워크들이 이어진 네트워크** — 전 세계의 작은 네트워크들이 서로 이어진 거예요.

</details>

## 집에서 해보기: 우리 집 공유기에 이어진 기기 세어 보기

명령 창(Windows: 명령 프롬프트, Mac: 터미널)에 아래 명령을 입력해 보세요. Windows와 Mac 모두 같습니다.

```bash
arp -a
```

목록에 나오는 IP 주소 하나가 같은 LAN에 있는 이웃 기기 하나입니다. Windows는 "동적"이라고 적힌 줄만 세면 됩니다. 몇 대가 나오나요?

## 실무에서 달라지는 점

- **"인터넷이 안 돼요"는 대부분 LAN 문제다.** 공유기·와이파이·케이블처럼 내 LAN 구간부터 확인하면 헛걸음이 줄어듭니다. 같은 와이파이의 다른 기기는 되는지 먼저 물어보는 게 가장 빠른 첫 질문입니다.
- **클라우드도 LAN과 WAN의 조합이다.** Azure의 가상 네트워크(VNet)는 클라우드 안의 LAN이고, 사무실과 Azure를 잇는 VPN·ExpressRoute는 WAN 구간입니다. 이번 편의 그림을 그대로 옮겨 그리면 클라우드 네트워크 구성도가 됩니다.
- **프로토콜 이름이 곧 설정값이다.** 방화벽·보안 그룹 규칙의 "프로토콜" 칸에 TCP, UDP, ICMP를 고르는 이유가 여기 있습니다. 약속이 다르면 같은 선 위에서도 통하지 않습니다.

## 시리즈 다른 편

- [네트워크 기초 EP.01 — 웹페이지 하나가 열리기까지, TCP/IP 4계층 큰 그림](/posts/tcpip-3d-lecture/)
- 다음 편 **EP.03 IP 주소 — 인터넷 세상의 집 주소**: 아파트 동·호수처럼, 컴퓨터에도 주소가 있어요.
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
