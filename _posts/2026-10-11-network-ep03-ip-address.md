---
title: "네트워크 기초 EP.03 — IP 주소, 인터넷 세상의 집 주소"
date: 2026-10-11 07:55:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, ip-address, ipv4, nat, lecture]
description: 택배 주소와 아파트 동·호수, 정문 경비실 비유로 IP 주소의 생김새, 사설 IP와 공인 IP, NAT까지 3D로 따라가는 약 3분 강의와 정리.
---

**IP 주소는 인터넷의 집 주소입니다. 집 안에서는 사설 IP, 바깥에서는 공인 IP를 쓰고, 그 사이를 공유기가 NAT로 바꿔 줍니다.** 이번 편은 택배 한 상자를 따라가며 이 구조를 보여 줍니다. 약 3분, 4개 챕터의 3D 강의예요.

> 소리를 켜고 보세요. 약 3분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

<video controls preload="metadata" playsinline poster="/assets/lectures/network-ep03/poster.jpg" style="width:100%;border-radius:12px">
  <source src="/assets/lectures/network-ep03/network-ep03.mp4" type="video/mp4">
</video>


## 오늘의 질문

> 인터넷에서 컴퓨터는 서로를 어떻게 찾아갈까요?

[지난 2편](/posts/network-ep02-what-is-network/)에서는 약속대로 이어진 기기들이 네트워크이고, 네트워크들의 네트워크가 인터넷이라는 걸 봤습니다. 이렇게 이어진 수많은 기기 중에서 딱 한 대를 찾아가려면 무엇이 필요할까요? 답은 주소에 있습니다.

| 챕터 | 내용 |
|---|---|
| 1. 주소가 있어야 찾아가요 | 모든 컴퓨터에는 주소가 있다 |
| 2. IP 주소의 생김새: 숫자 네 덩어리 | IPv4, 0~255, 약 43억 개 |
| 3. 아파트 호수와 정문 주소 | 사설 IP와 공인 IP |
| 4. 정문 경비실이 하는 일: NAT | 주소를 바꿔 붙이고 장부에 적기 |

## 1. 주소가 있어야 찾아가요

택배를 보낼 때 꼭 필요한 게 있죠. 바로 받는 사람의 주소입니다. 주소가 없는 택배는 길 안내를 맡은 라우터도 어디로 보내야 할지 모릅니다.

인터넷도 똑같습니다. 모든 컴퓨터는 저마다 주소를 하나씩 가지고 있고, 데이터에 받는 곳 주소를 붙여야 웹 서버까지 정확히 도착합니다. 이 컴퓨터의 주소를 **IP 주소**라고 부릅니다. 인터넷 세상의 집 주소인 셈이에요.

## 2. IP 주소의 생김새: 숫자 네 덩어리

집 주소는 시, 구, 동, 호수처럼 큰 곳에서 작은 곳으로 좁혀 가죠. IP 주소도 비슷하게 **점으로 나눈 숫자 네 덩어리**로 되어 있습니다. `192.168.0.10`처럼요.

덩어리 하나에는 0부터 255까지의 숫자만 들어갈 수 있습니다. 이런 모양의 주소를 **IPv4**라고 합니다. 문제는 이 방식으로 만들 수 있는 주소가 약 43억 개뿐이라는 점입니다. 세상의 기기를 모두 담기엔 모자라죠. 그래서 다음 챕터의 아이디어가 등장합니다.

## 3. 아파트 호수와 정문 주소: 사설 IP · 공인 IP

아파트를 떠올려 보세요. 단지 안에서는 "101호"라고만 해도 집을 찾을 수 있습니다. 옆 단지에도 101호가 있지만 상관없어요. 각자 자기 단지 안에서만 쓰니까요.

우리 집 공유기가 바로 이 아파트 단지입니다. 집 안 기기들은 `192.168`로 시작하는 호수를 하나씩 받습니다. 이렇게 **집 안에서만 쓰는 주소를 사설 IP**라고 합니다. 옆집이 똑같은 번호를 써도 괜찮습니다.

그리고 단지 정문에는 세상에 하나뿐인 진짜 주소가 붙어 있습니다. 이게 **공인 IP**입니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="두 집의 공유기 안에서 같은 사설 IP 192.168.0.11을 써도 되지만, 각 공유기의 정문에는 세상에 하나뿐인 공인 IP가 붙어 있다">
  <text x="0" y="14" class="sk-title">① 단지 안의 호수(사설 IP)와 정문 주소(공인 IP)</text>
  <rect x="10" y="40" width="320" height="170" rx="10" class="sk-box-muted"/>
  <text x="170" y="62" text-anchor="middle" class="sk-label">우리 집 (공유기 안)</text>
  <rect x="30" y="80" width="130" height="40" rx="8" class="sk-box"/><text x="95" y="104" text-anchor="middle" class="sk-mono">192.168.0.10</text>
  <rect x="180" y="80" width="130" height="40" rx="8" class="sk-box"/><text x="245" y="104" text-anchor="middle" class="sk-mono">192.168.0.11</text>
  <rect x="80" y="150" width="180" height="44" rx="8" class="sk-box-accent"/><text x="170" y="170" text-anchor="middle" class="sk-label">정문 = 공인 IP</text><text x="170" y="186" text-anchor="middle" class="sk-sub">세상에 하나뿐</text>
  <rect x="390" y="40" width="320" height="170" rx="10" class="sk-box-muted"/>
  <text x="550" y="62" text-anchor="middle" class="sk-label">옆집 (공유기 안)</text>
  <rect x="410" y="80" width="130" height="40" rx="8" class="sk-box"/><text x="475" y="104" text-anchor="middle" class="sk-mono">192.168.0.10</text>
  <rect x="560" y="80" width="130" height="40" rx="8" class="sk-box"/><text x="625" y="104" text-anchor="middle" class="sk-mono">192.168.0.11</text>
  <rect x="460" y="150" width="180" height="44" rx="8" class="sk-box-accent"/><text x="550" y="170" text-anchor="middle" class="sk-label">정문 = 공인 IP</text><text x="550" y="186" text-anchor="middle" class="sk-sub">옆집과 다른 번호</text>
</svg>
<figcaption>사설 IP는 단지 안에서만 유일하면 되고, 바깥 세상에서 구별되는 건 정문의 공인 IP다.</figcaption>
</figure>

## 4. 정문 경비실이 하는 일: NAT

그럼 집 안의 컴퓨터가 바깥 웹 서버에 택배를 보내면 어떻게 될까요? 바깥 세상은 `192.168`로 시작하는 호수를 모릅니다.

택배가 정문에 오면 경비실이 **보내는 곳 주소를 단지의 공인 IP로 바꿔 붙입니다.** 그리고 장부에 적어 둡니다. "이 택배는 끝번호 11번 기기가 보낸 것." 답장이 정문에 도착하면 경비실은 장부를 보고 11번 기기에게 정확히 배달해 줍니다.

<figure class="sketch">
<svg viewBox="0 0 720 210" role="img" aria-label="집 안 11번 기기가 보낸 택배는 공유기에서 보내는 곳 주소가 공인 IP로 바뀌어 웹 서버로 가고, 답장은 공유기 장부를 보고 다시 11번 기기로 돌아온다">
  <defs>
    <marker id="ep3-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep3-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 정문 경비실(NAT): 주소를 바꾸고 장부에 적는다</text>
  <rect x="10" y="60" width="150" height="56" rx="8" class="sk-box"/>
  <text x="85" y="84" text-anchor="middle" class="sk-label">11번 기기</text>
  <text x="85" y="102" text-anchor="middle" class="sk-mono">192.168.0.11</text>
  <rect x="270" y="50" width="180" height="76" rx="8" class="sk-box-accent"/>
  <text x="360" y="74" text-anchor="middle" class="sk-label">공유기 = 경비실</text>
  <text x="360" y="92" text-anchor="middle" class="sk-sub">보내는 곳 → 공인 IP</text>
  <text x="360" y="110" text-anchor="middle" class="sk-sub">장부: "11번이 보냄"</text>
  <rect x="560" y="60" width="150" height="56" rx="8" class="sk-box"/>
  <text x="635" y="84" text-anchor="middle" class="sk-label">웹 서버</text>
  <text x="635" y="102" text-anchor="middle" class="sk-sub">공인 IP만 보인다</text>
  <path d="M162,78 L268,78" class="sk-line" marker-end="url(#ep3-m1)"/>
  <path d="M452,78 L558,78" class="sk-line" marker-end="url(#ep3-m1)"/>
  <path d="M558,102 L452,102" class="sk-line-accent" marker-end="url(#ep3-m2)"/>
  <path d="M268,102 L162,102" class="sk-line-accent" marker-end="url(#ep3-m2)"/>
  <text x="215" y="150" text-anchor="middle" class="sk-sub">답장: 장부 보고 11번에게</text>
  <text x="505" y="150" text-anchor="middle" class="sk-sub">답장은 정문 주소로 도착</text>
  <text x="360" y="190" text-anchor="middle" class="sk-sub">바깥에서는 집 안 기기의 사설 IP를 전혀 몰라도 된다</text>
</svg>
<figcaption>NAT 덕분에 집 안 기기 여러 대가 공인 IP 하나를 함께 쓰면서도 답장을 정확히 돌려받는다.</figcaption>
</figure>

이렇게 주소를 바꿔 주는 일을 **NAT**(Network Address Translation, 주소 변환)라고 합니다. 우리 집 공유기가 매일 하는 일이에요. 주소가 모자란 IPv4로도 이 많은 기기가 인터넷을 쓸 수 있는 이유가 여기에 있습니다.

## 한 줄 정리

> IP 주소는 **인터넷의 집 주소**. 집 안은 **사설 IP**, 바깥은 **공인 IP**, 그 사이를 공유기가 **NAT**로 바꿔 준다.
{: .prompt-info }

## 퀴즈

1. IPv4 주소는 숫자 몇 덩어리일까요? (두 덩어리 / 네 덩어리)
2. 우리 집 안에서만 쓰는 주소는? (사설 IP / 공인 IP)

<details markdown="1">
<summary>정답 보기</summary>

1. **네 덩어리** — `192.168.0.10`처럼 점으로 나눈 숫자 네 덩어리예요.
2. **사설 IP** — 아파트 호수처럼 단지(공유기) 안에서만 써요.

</details>

## 집에서 해보기: 내 컴퓨터의 IP 주소 찾아보기

명령 창(Windows: 명령 프롬프트, Mac: 터미널)에 아래 명령을 입력해 보세요.

```bash
# Windows
ipconfig

# Mac
ipconfig getifaddr en0
```

Windows는 "IPv4 주소" 줄을 보세요. `192.168`로 시작하는 숫자가 나오면, 그게 우리 집 안의 사설 IP입니다.

## 실무에서 달라지는 점

- **사설 IP 대역은 설계 단계에서 겹치지 않게 잡는다.** Azure 가상 네트워크(VNet)도 사설 IP 대역을 정해서 만듭니다. 집에서는 옆집과 같은 번호를 써도 괜찮지만, 사무실과 VNet을 VPN으로 이을 때 대역이 겹치면 서로를 찾지 못합니다.
- **"내 IP"는 두 개다.** `ipconfig`에 나오는 사설 IP와, 바깥 서비스가 보는 공인 IP는 다릅니다. M365의 조건부 액세스나 방화벽 허용 목록에 넣어야 하는 건 정문 주소, 즉 공인 IP입니다.
- **클라우드의 아웃바운드도 NAT를 거친다.** Azure의 NAT 게이트웨이처럼, 사설 IP만 가진 서버들이 공인 IP를 빌려 바깥으로 나가는 구조는 이번 편의 경비실 그림과 같습니다.

## 시리즈 다른 편

- [네트워크 기초 EP.02 — 네트워크가 뭘까? 컴퓨터끼리 말하는 법](/posts/network-ep02-what-is-network/)
- 다음 편 **EP.04 DNS — 이름을 주소로 바꾸는 전화번호부**: 휴대폰 연락처처럼, 이름만 알면 찾아갈 수 있어요.
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
