---
title: "네트워크 기초 EP.12 — DHCP, 이사 오면 주소를 자동으로"
date: 2026-10-23 07:55:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, dhcp, ip-address, lease, lecture]
description: 새 기기가 손을 들면 공유기가 IP 주소를 자동으로 빌려 주는 DHCP. 자동 주소 배정, 임대 기간, 주소 충돌 방지를 따라가는 약 2분 반 강의와 정리.
---

**DHCP는 새 기기에게 IP 주소를 자동으로 빌려 주는 관리사무소입니다. 임대 기간이 지나면 돌려받아서 다시 빌려 줍니다.** 이번 편은 와이파이에 붙기만 했는데 주소가 생기는 이유를 전입신고에 빗대어 풀어 봅니다. 약 2분 반, 4개 챕터의 3D 강의예요.

> 소리를 켜고 보세요. 약 2분 반짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='LABZSk66CCI' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> 새 휴대폰을 와이파이에 연결하면, 아무것도 안 했는데 IP 주소가 생겨요. 누가 준 걸까요?

[지난 편](/posts/network-ep11-https/)에서는 주소창 자물쇠의 비밀, HTTPS를 배웠습니다. 이번에는 다시 맨 앞으로 돌아갑니다. 웹페이지를 요청하기도 전에, 기기가 네트워크에 처음 들어오는 순간 무슨 일이 일어나는지 봅니다.

| 챕터 | 내용 |
|---|---|
| 1. 이사 오면 전입신고 | 관리사무소가 빈 주소를 알려 주고 장부에 적는다 |
| 2. 손 들면 번호표: DHCP | "주소 좀 주세요!" → 제안 → 수락 → 확정 |
| 3. 빌려 쓰는 주소: 임대 기간 | 연장하거나, 돌려받아 다시 빌려 주기 |
| 4. 번호가 겹치지 않게 | 장부 덕분에 같은 주소를 두 번 주지 않는다 |

## 1. 이사 오면 전입신고

새 동네로 이사 오면 관리사무소에 가서 전입신고를 하죠. 그러면 관리사무소가 비어 있는 집 주소를 알려 주고, 장부에 적어 둡니다. 네트워크에도 이 관리사무소 역할을 하는 장치가 있어요. 우리 집에서는 보통 공유기입니다.

## 2. 손 들면 번호표: DHCP

우리 집에 새 휴대폰이 오면, 와이파이에 붙자마자 크게 외칩니다. "주소 좀 주세요!"

1. 휴대폰: "주소 좀 주세요!"
2. 공유기: "192.168.0.23 비어 있어요, 이거 쓰세요."
3. 휴대폰: "그거 쓸게요."
4. 공유기: "확정!" 하고 장부에 적습니다.

<figure class="sketch">
<svg viewBox="0 0 720 270" role="img" aria-label="새 휴대폰이 주소를 달라고 외치면 공유기가 빈 주소를 제안하고, 휴대폰이 쓰겠다고 하면 공유기가 확정하고 장부에 적는다. 이때 게이트웨이와 DNS 주소도 함께 알려 준다">
  <defs>
    <marker id="ep12-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep12-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 손 들고, 제안받고, 수락하고, 확정</text>
  <rect x="10" y="36" width="130" height="190" rx="8" class="sk-box"/>
  <text x="75" y="60" text-anchor="middle" class="sk-label">새 휴대폰</text>
  <text x="75" y="78" text-anchor="middle" class="sk-sub">아직 주소 없음</text>
  <rect x="580" y="36" width="130" height="190" rx="8" class="sk-box"/>
  <text x="645" y="60" text-anchor="middle" class="sk-label">공유기</text>
  <text x="645" y="78" text-anchor="middle" class="sk-sub">관리사무소</text>
  <path d="M142,100 L578,100" class="sk-line" marker-end="url(#ep12-m1)"/>
  <text x="360" y="94" text-anchor="middle" class="sk-sub">1 · "주소 좀 주세요!" (모두에게 외침)</text>
  <path d="M578,136 L142,136" class="sk-line" marker-end="url(#ep12-m1)"/>
  <text x="360" y="130" text-anchor="middle" class="sk-mono">2 · 192.168.0.23 비어 있어요</text>
  <path d="M142,172 L578,172" class="sk-line" marker-end="url(#ep12-m1)"/>
  <text x="360" y="166" text-anchor="middle" class="sk-sub">3 · "그거 쓸게요"</text>
  <path d="M578,208 L142,208" class="sk-line-accent" marker-end="url(#ep12-a1)"/>
  <text x="360" y="202" text-anchor="middle" class="sk-label">4 · 확정! (장부에 기록)</text>
  <rect x="160" y="232" width="400" height="28" rx="6" class="sk-box-accent"/>
  <text x="360" y="251" text-anchor="middle" class="sk-sub">함께 받는 것: IP 주소 · 기본 게이트웨이 · DNS 서버 주소</text>
</svg>
<figcaption>우리가 주소를 직접 입력하지 않아도 되는 건, 이 네 마디 대화가 자동으로 오가기 때문이다.</figcaption>
</figure>

이렇게 주소를 자동으로 나눠 주는 일을 **DHCP**라고 합니다. 덕분에 우리는 주소를 직접 입력할 필요가 없죠. 그리고 공유기는 주소만 주는 게 아니라 **기본 게이트웨이**와 **DNS 서버 주소**도 같이 알려 줍니다. 앞에서 배운 바로 그것들이에요. 밖으로 나가는 문과 이름을 주소로 바꿔 주는 전화번호부까지 한 번에 챙겨 주는 셈입니다.

## 3. 빌려 쓰는 주소: 임대 기간

그런데 이 주소는 평생 주는 게 아니라 **빌려 주는** 겁니다. 이걸 **임대 기간**이라고 해요.

<figure class="sketch">
<svg viewBox="0 0 720 170" role="img" aria-label="빌린 주소는 기간이 끝나기 전에 더 쓰겠다고 하면 연장되고, 집을 떠난 기기의 주소는 기간이 끝나면 돌려받아 새 기기에게 다시 빌려 준다">
  <defs>
    <marker id="ep12-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep12-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 주소는 빌려 쓰는 것</text>
  <rect x="10" y="64" width="150" height="50" rx="8" class="sk-box-accent"/>
  <text x="85" y="86" text-anchor="middle" class="sk-label">주소 빌림</text>
  <text x="85" y="104" text-anchor="middle" class="sk-sub">임대 기간 시작</text>
  <path d="M162,80 L318,50" class="sk-line" marker-end="url(#ep12-m2)"/>
  <path d="M162,98 L318,128" class="sk-line" marker-end="url(#ep12-m2)"/>
  <rect x="320" y="28" width="180" height="44" rx="8" class="sk-box"/>
  <text x="410" y="48" text-anchor="middle" class="sk-label">아직 집에 있음</text>
  <text x="410" y="64" text-anchor="middle" class="sk-sub">끝나기 전 "더 쓸게요"</text>
  <rect x="320" y="106" width="180" height="44" rx="8" class="sk-box-muted"/>
  <text x="410" y="126" text-anchor="middle" class="sk-label">집을 떠남</text>
  <text x="410" y="142" text-anchor="middle" class="sk-sub">기간 끝 → 돌려받음</text>
  <path d="M502,50 L568,50" class="sk-line-accent" marker-end="url(#ep12-a2)"/>
  <rect x="570" y="28" width="140" height="44" rx="8" class="sk-box"/>
  <text x="640" y="54" text-anchor="middle" class="sk-label">연장</text>
  <path d="M502,128 L568,128" class="sk-line" marker-end="url(#ep12-m2)"/>
  <rect x="570" y="106" width="140" height="44" rx="8" class="sk-box"/>
  <text x="640" y="126" text-anchor="middle" class="sk-label">새 기기에게</text>
  <text x="640" y="142" text-anchor="middle" class="sk-sub">다시 빌려 줌</text>
</svg>
<figcaption>빌려 주는 방식이라서, 한정된 주소로도 드나드는 기기들을 계속 받아 낼 수 있다.</figcaption>
</figure>

기간이 끝나기 전에 기기가 "더 쓸게요" 하면 연장해 줍니다. 집을 떠난 기기의 주소는 기간이 끝나면 돌려받아서, 새로 온 기기에게 다시 빌려 줍니다.

## 4. 번호가 겹치지 않게

만약 두 기기가 같은 주소를 쓰면 어떻게 될까요? 택배가 어디로 가야 할지 헷갈리겠죠. 그래서 공유기는 장부를 보고 **이미 빌려 준 주소는 다시 주지 않습니다.**

주소를 손으로 직접 정하면 이런 충돌이 생길 수 있습니다. 장부에 없는 주소를 누군가 임의로 쓰고 있으면 관리사무소도 모르니까요. 그래서 보통은 DHCP에게 맡기는 편이 편하고 안전합니다.

## 한 줄 정리

> DHCP는 새 기기에게 **주소를 자동으로 빌려 주는** 관리사무소. **임대 기간**이 지나면 돌려받아요.
{: .prompt-info }

## 퀴즈

1. 새 기기에게 주소를 자동으로 나눠 주는 것은? (DHCP / DNS)
2. DHCP로 받은 주소는? (정해진 기간 동안 빌려 쓴다 / 평생 내 것이다)

<details markdown="1">
<summary>정답 보기</summary>

1. **DHCP** — DNS는 이름 → 주소, DHCP는 주소 나눠 주기예요.
2. **정해진 기간 동안 빌려 쓴다** — 임대 기간이 끝나면 연장하거나 돌려줘요.

</details>

## 집에서 해보기: 내 주소의 임대 기간 확인하기

명령 창(Windows: 명령 프롬프트, Mac: 터미널)에 아래 명령을 입력해 보세요.

```bash
# Windows
ipconfig /all

# Mac
ipconfig getpacket en0
```

Windows는 "임대 시작 날짜"와 "임대 만료 날짜", Mac은 `lease_time` 줄을 찾아보세요. 내 주소를 언제 빌렸고 언제까지 쓸 수 있는지 나옵니다. Mac에서 와이파이 인터페이스 이름이 en0이 아니라면 아무것도 안 나올 수 있어요.

## 실무에서 달라지는 점

- **"169.254로 시작하는 주소"는 DHCP 실패 신호다.** DHCP 서버에게서 답을 못 받으면 Windows는 스스로 169.254.x.x 주소를 붙입니다. 이 주소가 보이면 케이블·와이파이·DHCP 서버 쪽부터 확인합니다.
- **Azure VNet에서도 VM 주소는 DHCP로 받는다.** Azure가 주소를 나눠 주기 때문에, 고정 IP가 필요하면 VM 안에서 손으로 바꾸지 않고 Azure 쪽 NIC 설정에서 정적으로 지정하는 게 원칙입니다. 이번 편의 "손으로 정하면 충돌" 이야기의 클라우드 버전이에요.
- **주소가 바닥나면 새 기기가 못 들어온다.** 행사장이나 회의실처럼 기기가 한꺼번에 몰리는 곳에서는 주소 범위와 임대 기간이 실제 접속 장애로 이어집니다. 사람이 자주 바뀌는 게스트 와이파이에서 임대 기간을 짧게 잡는 이유입니다.

## 시리즈 다른 편

- [네트워크 기초 EP.11 — HTTPS, 주소창 자물쇠의 비밀](/posts/network-ep11-https/)
- 다음 편 예고 **EP.13 와이파이 — 공기로 보내는 편지**: 라디오 채널처럼, 전파를 골라 들어요.
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
