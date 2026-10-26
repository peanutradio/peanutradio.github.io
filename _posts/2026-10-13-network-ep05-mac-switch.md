---
title: "네트워크 기초 EP.05 — MAC 주소와 스위치, 같은 동네 안에서 찾아가기"
date: 2026-10-13 07:55:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, lecture]
description: 기기마다 하나뿐인 이름 MAC 주소, 같은 층의 반장님 스위치, "이 주소 누구예요?" ARP까지 3D로 따라가는 약 3분 반 강의와 정리.
---

**MAC 주소는 기기마다 하나뿐인 이름입니다. 같은 동네(LAN) 안에서는 스위치가 이 이름을 보고 배달하고, 모르면 ARP로 물어봅니다.** IP 주소가 "사는 곳"이라면 MAC 주소는 "주민등록번호"예요. 약 3분 반, 4개 챕터의 3D 강의입니다.

> 소리를 켜고 보세요. 약 3분 반짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='6j13WRR7XGw' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> 같은 동네 안에서 기기들은 서로를 어떻게 찾아갈까요? IP 주소 말고 이름이 하나 더 있대요.

[지난 편](/posts/network-ep04-dns/)에서는 이름을 IP 주소로 바꿔 주는 전화번호부, DNS를 배웠습니다. 주소를 알았으니 이제 실제로 찾아갈 차례입니다. 이번 편은 먼저 한 동네, 그러니까 같은 공유기·같은 사무실 안에서 기기들이 서로를 찾는 방법입니다.

| 챕터 | 내용 |
|---|---|
| 1. 기기마다 하나뿐인 이름: MAC 주소 | 공장에서 붙여 주는, 바뀌지 않는 이름 |
| 2. 같은 층의 반장님: 스위치 | 어느 칸에 누가 있는지 표로 기억 |
| 3. 이 주소 누구예요?: ARP | IP 주소로 MAC 주소 알아내기 |
| 4. IP와 MAC, 같이 일해요 | 끝까지 가는 IP, 다음 칸까지의 MAC |

## 1. 기기마다 하나뿐인 이름: MAC 주소

사람에게는 태어날 때 받는 주민등록번호가 있죠. 이사를 가도 바뀌지 않습니다.

네트워크 기기에도 이런 번호가 있습니다. 공장에서 만들 때 붙여 주는 기기만의 이름, **MAC 주소**입니다. 숫자와 알파벳 열두 글자를 두 개씩 끊어서 `a4:83:e7:1c:20:5b`처럼 씁니다.

노트북을 들고 카페에 가면 IP 주소는 바뀌지만 MAC 주소는 그대로입니다. **IP 주소가 사는 곳 주소라면, MAC 주소는 주민등록번호**인 셈이죠.

## 2. 같은 층의 반장님: 스위치

아파트의 한 층을 떠올려 볼까요? 복도를 따라 집들이 쭉 있고, 이 층의 반장님은 어느 집에 누가 사는지 다 알아서 편지를 딱 그 집에만 전해 줍니다.

네트워크에도 이런 반장님이 있습니다. 기기들을 줄로 이어 주는 상자, **스위치**입니다. 스위치는 몇 번 칸(포트)에 어떤 MAC 주소의 기기가 꽂혀 있는지 표로 기억합니다. 그래서 PC가 프린터에게 보내면 다른 기기는 건드리지 않고 프린터에게만 전해 줍니다.

<figure class="sketch">
<svg viewBox="0 0 720 240" role="img" aria-label="스위치는 칸 번호와 MAC 주소를 짝지은 표를 보고, PC가 보낸 데이터를 프린터가 꽂힌 칸으로만 내보낸다">
  <defs>
    <marker id="ep5-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 스위치는 "몇 번 칸에 누가 있는지" 표로 기억한다</text>
  <rect x="250" y="40" width="220" height="110" rx="8" class="sk-box-accent"/>
  <text x="360" y="62" text-anchor="middle" class="sk-label">스위치의 표</text>
  <rect x="265" y="74" width="190" height="20" rx="4" class="sk-box"/><text x="360" y="89" text-anchor="middle" class="sk-sub">1번 칸 → PC (aa:11…)</text>
  <rect x="265" y="98" width="190" height="20" rx="4" class="sk-box"/><text x="360" y="113" text-anchor="middle" class="sk-sub">2번 칸 → 노트북 (bb:22…)</text>
  <rect x="265" y="122" width="190" height="20" rx="4" class="sk-box"/><text x="360" y="137" text-anchor="middle" class="sk-sub">3번 칸 → 프린터 (cc:33…)</text>
  <rect x="20" y="70" width="140" height="50" rx="8" class="sk-box"/>
  <text x="90" y="92" text-anchor="middle" class="sk-label">PC</text>
  <text x="90" y="110" text-anchor="middle" class="sk-sub">받는 곳: cc:33…</text>
  <path d="M162,95 L246,95" class="sk-line-accent" marker-end="url(#ep5-m1)"/>
  <rect x="560" y="50" width="140" height="40" rx="8" class="sk-box-muted"/>
  <text x="630" y="75" text-anchor="middle" class="sk-label">노트북</text>
  <rect x="560" y="120" width="140" height="40" rx="8" class="sk-box-accent"/>
  <text x="630" y="145" text-anchor="middle" class="sk-label">프린터</text>
  <path d="M472,70 L556,70" class="sk-line"/>
  <text x="514" y="62" text-anchor="middle" class="sk-sub">안 보냄</text>
  <path d="M472,132 L556,138" class="sk-line-accent" marker-end="url(#ep5-m1)"/>
  <text x="360" y="190" text-anchor="middle" class="sk-sub">받는 곳 MAC 주소를 표에서 찾아, 그 칸으로만 내보낸다</text>
</svg>
<figcaption>우리 집 공유기 뒤의 랜선 구멍들이 바로 스위치의 칸이다.</figcaption>
</figure>

우리 집 공유기 안에도 이런 스위치가 들어 있습니다. 랜선을 꽂는 구멍들이 바로 그 칸이에요.

## 3. 이 주소 누구예요?: ARP

그런데 PC가 프린터의 IP 주소만 알고 MAC 주소는 모른다면 어떻게 할까요? 반장님이라면 복도를 돌며 물어보겠죠. "이 주소 쓰는 분, 누구예요?" 그러면 주인만 문을 열고 대답합니다. "저예요!"

네트워크도 똑같습니다. PC가 동네 전체에 한꺼번에 물어보면(브로드캐스트), 주인인 프린터만 자기 MAC 주소로 대답합니다. 이렇게 **IP 주소로 MAC 주소를 알아내는 방법을 ARP**라고 합니다. 알아낸 답은 ARP 표에 적어 두고, 다음부터는 묻지 않고 바로 보냅니다.

## 4. IP와 MAC, 같이 일해요

그럼 동네 밖의 웹 서버에게 보낼 때는 어떨까요? 택배에는 주소가 두 개 붙습니다. **끝까지 가는 IP 주소**, 그리고 **바로 다음 칸의 MAC 주소**입니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="PC에서 공유기, 공유기에서 다음 라우터로 넘어갈 때마다 MAC 주소는 바뀌지만 IP 주소는 웹 서버까지 그대로 유지된다">
  <defs>
    <marker id="ep5-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② IP는 끝까지 그대로, MAC은 칸마다 바뀐다</text>
  <rect x="10" y="40" width="120" height="44" rx="8" class="sk-box"/><text x="70" y="67" text-anchor="middle" class="sk-label">PC</text>
  <rect x="210" y="40" width="120" height="44" rx="8" class="sk-box"/><text x="270" y="67" text-anchor="middle" class="sk-label">공유기</text>
  <rect x="410" y="40" width="120" height="44" rx="8" class="sk-box"/><text x="470" y="67" text-anchor="middle" class="sk-label">다음 라우터</text>
  <rect x="600" y="40" width="110" height="44" rx="8" class="sk-box"/><text x="655" y="67" text-anchor="middle" class="sk-label">웹 서버</text>
  <path d="M132,62 L206,62" class="sk-line" marker-end="url(#ep5-m2)"/>
  <path d="M332,62 L406,62" class="sk-line" marker-end="url(#ep5-m2)"/>
  <path d="M532,62 L596,62" class="sk-line" marker-end="url(#ep5-m2)"/>
  <rect x="110" y="110" width="120" height="30" rx="6" class="sk-box-muted"/><text x="170" y="130" text-anchor="middle" class="sk-sub">MAC: 공유기</text>
  <rect x="310" y="110" width="120" height="30" rx="6" class="sk-box-muted"/><text x="370" y="130" text-anchor="middle" class="sk-sub">MAC: 다음 라우터</text>
  <rect x="500" y="110" width="120" height="30" rx="6" class="sk-box-muted"/><text x="560" y="130" text-anchor="middle" class="sk-sub">MAC: 웹 서버</text>
  <rect x="110" y="160" width="510" height="30" rx="6" class="sk-box-accent"/><text x="365" y="180" text-anchor="middle" class="sk-label">IP: 웹 서버 주소 (처음부터 끝까지 그대로)</text>
</svg>
<figcaption>MAC 주소는 같은 동네 안에서, IP 주소는 동네와 동네 사이를 찾아갈 때 쓴다.</figcaption>
</figure>

웹 서버는 동네 밖이니까, PC는 먼저 우리 집 공유기의 MAC 주소를 적어 보냅니다. 공유기에 닿으면 MAC 주소는 다음 칸 것으로 바뀌지만 IP 주소는 끝까지 그대로입니다.

## 한 줄 정리

> MAC 주소는 **기기마다 하나뿐인 이름**. 같은 동네 안에서는 **스위치**가 MAC 주소를 보고 배달하고, 모르면 **ARP**로 물어본다.
{: .prompt-info }

## 퀴즈

1. 카페에 가도 바뀌지 않는 기기의 이름은? (IP 주소 / MAC 주소)
2. IP 주소로 MAC 주소를 알아내는 방법은? (ARP / DNS)

<details markdown="1">
<summary>정답 보기</summary>

1. **MAC 주소** — 공장에서 만들 때 붙여 주는 이름이라 어디를 가도 그대로예요.
2. **ARP** — 모두에게 "이 주소 누구예요?" 하고 물으면 주인만 대답해요.

</details>

## 집에서 해보기: 우리 컴퓨터의 ARP 표 보기

명령 창(Windows: 명령 프롬프트, Mac: 터미널)에 아래 명령을 입력해 보세요. Windows와 Mac 모두 같습니다.

```bash
arp -a
```

IP 주소 옆의 `aa-bb-cc-…`(Mac은 `aa:bb:cc:…`) 모양이 MAC 주소입니다. 우리 동네에서 최근에 대화한 이웃 기기들이에요. 기본 게이트웨이(보통 `192.168.0.1`) 줄도 찾아보세요.

## 실무에서 달라지는 점

- **"IP는 맞는데 안 붙는다"면 ARP 표를 본다.** 같은 IP를 두 기기가 쓰는 IP 충돌이 나면 ARP 표의 MAC 주소가 왔다 갔다 합니다. `arp -a`로 게이트웨이 MAC이 엉뚱하게 바뀌었는지 확인하는 게 첫 단서예요.
- **휴대폰·노트북은 Wi‑Fi마다 MAC을 바꾼다.** 요즘 OS는 사생활 보호용으로 '임의 MAC 주소'를 씁니다. MAC 주소로 기기를 허용하는 회사 Wi‑Fi나 DHCP 예약이 갑자기 안 되면 이 설정부터 의심하세요.
- **사무실 스위치의 칸은 VLAN으로 나뉜다.** 같은 스위치에 꽂혀 있어도 VLAN이 다르면 다른 동네라서 서로 못 찾습니다. "옆자리 프린터가 안 보여요"의 흔한 원인입니다.

## 시리즈 다른 편

- [네트워크 기초 EP.04 — DNS, 이름을 주소로 바꾸는 전화번호부](/posts/network-ep04-dns/)
- 다음 편 **EP.06 라우터 — 길을 알려 주는 교차로**: 동네 밖으로 나가면, 갈림길마다 방향을 정해 줘요.
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
