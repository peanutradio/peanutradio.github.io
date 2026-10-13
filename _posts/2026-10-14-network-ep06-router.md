---
title: "네트워크 기초 EP.06 — 라우터, 길을 알려 주는 교차로"
date: 2026-10-14 07:55:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, router, routing-table, default-gateway, lecture]
description: 갈림길마다 서 있는 길 안내 담당 라우터를 라우팅 테이블, 기본 게이트웨이, 홉과 우회까지 3D 지도 위에서 따라가는 약 3분 강의와 정리.
---

**라우터는 인터넷 갈림길의 길 안내입니다. 라우팅 테이블이라는 표지판 목록을 보고 데이터를 다음 갈림길로 보내 줍니다.** 이번 편은 택배 한 상자가 집을 나서 웹 서버에 닿기까지 지나는 갈림길을 따라갑니다. 약 3분, 4개 챕터의 3D 강의예요.

> 소리를 켜고 보세요. 약 3분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='C47o0SpHYx0' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> 주소를 알았으니, 이제 어떤 길로 가야 할까요? 인터넷에는 길이 수없이 많은데 말이에요.

[지난 편](/posts/network-ep05-mac-switch/)에서는 기기마다 하나뿐인 이름 MAC 주소와, 같은 동네 안에서 배달해 주는 스위치를 배웠습니다. 그럼 동네 밖으로 나가는 택배는 실제로 어떤 길을 따라 움직일까요? 이번 편은 그 길을 정하는 장비 이야기입니다.

| 챕터 | 내용 |
|---|---|
| 1. 갈림길마다 길 안내: 라우터 | 받는 곳 IP를 보고 다음 갈림길로 |
| 2. 라우터의 표지판 목록: 라우팅 테이블 | 주소별로 보낼 길이 적힌 목록 |
| 3. 동네 밖으로 나가는 출구: 기본 게이트웨이 | 집 밖으로 가는 건 일단 공유기로 |
| 4. 몇 번 갈아탔을까: 홉, 그리고 돌아가기 | 라우터 하나 = 1홉, 막히면 우회 |

## 1. 갈림길마다 길 안내: 라우터

먼 곳까지 차를 몰고 갈 때, 갈림길이 나올 때마다 어디로 갈지 정해야 하죠. 그럴 때 표지판이나 내비게이션이 방향을 알려 줍니다.

인터넷의 갈림길마다 서 있는 길 안내 담당이 바로 **라우터**입니다. 라우터는 택배에 적힌 **받는 곳 IP 주소**를 보고, 다음 갈림길 쪽으로 보내 줍니다. 목적지까지 한 번에 데려다주는 게 아니라, "다음은 저쪽"이라고 한 칸씩 넘겨 주는 방식이에요.

## 2. 라우터의 표지판 목록: 라우팅 테이블

라우터는 머릿속에 표지판 목록을 가지고 있습니다. 이걸 **라우팅 테이블**이라고 합니다. "어느 주소로 가는 택배는 어느 길로 보내라", 이렇게 적혀 있죠.

택배가 오면 라우터는 받는 곳 주소와 맞는 줄을 찾아서 그 길로 내보냅니다. 목록에 없는 주소는 어떻게 할까요? "그 밖의 모든 곳"이라는 큰길로 보냅니다. 목록이 아무리 짧아도 갈 곳을 잃는 택배가 없는 이유입니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="라우터는 택배의 받는 곳 주소를 라우팅 테이블과 맞춰 보고, 맞는 줄이 있으면 그 길로, 없으면 그 밖의 모든 곳으로 가는 큰길로 내보낸다">
  <defs>
    <marker id="ep6-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep6-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 라우팅 테이블: 맞는 줄을 찾고, 없으면 큰길로</text>
  <rect x="10" y="80" width="130" height="56" rx="8" class="sk-box"/>
  <text x="75" y="104" text-anchor="middle" class="sk-label">택배</text>
  <text x="75" y="122" text-anchor="middle" class="sk-sub">받는 곳 IP 확인</text>
  <path d="M142,108 L198,108" class="sk-line" marker-end="url(#ep6-m1)"/>
  <rect x="200" y="40" width="250" height="150" rx="8" class="sk-box-accent"/>
  <text x="325" y="62" text-anchor="middle" class="sk-label">라우터의 표지판 목록</text>
  <rect x="215" y="76" width="220" height="30" rx="6" class="sk-box"/><text x="325" y="96" text-anchor="middle" class="sk-sub">A 동네로 가는 주소 → 1번 길</text>
  <rect x="215" y="112" width="220" height="30" rx="6" class="sk-box"/><text x="325" y="132" text-anchor="middle" class="sk-sub">B 동네로 가는 주소 → 2번 길</text>
  <rect x="215" y="148" width="220" height="30" rx="6" class="sk-box-muted"/><text x="325" y="168" text-anchor="middle" class="sk-sub">그 밖의 모든 곳 → 큰길</text>
  <path d="M452,91 L558,70" class="sk-line" marker-end="url(#ep6-m1)"/>
  <path d="M452,127 L558,127" class="sk-line" marker-end="url(#ep6-m1)"/>
  <path d="M452,163 L558,184" class="sk-line-accent" marker-end="url(#ep6-m2)"/>
  <rect x="560" y="52" width="150" height="36" rx="8" class="sk-box"/><text x="635" y="75" text-anchor="middle" class="sk-label">1번 길</text>
  <rect x="560" y="109" width="150" height="36" rx="8" class="sk-box"/><text x="635" y="132" text-anchor="middle" class="sk-label">2번 길</text>
  <rect x="560" y="166" width="150" height="36" rx="8" class="sk-box"/><text x="635" y="189" text-anchor="middle" class="sk-label">큰길</text>
</svg>
<figcaption>라우터는 모든 목적지를 알 필요가 없다. 모르는 곳은 큰길로 넘기면 된다.</figcaption>
</figure>

## 3. 동네 밖으로 나가는 출구: 기본 게이트웨이

그럼 우리 집 컴퓨터는 택배를 처음에 어디로 보낼까요? 집 안의 노트북에게는 바로 보내면 됩니다. 하지만 **집 밖으로 가는 택배는 무조건 우리 집 공유기로 먼저** 보냅니다. 동네 밖으로 나가는 출구 같은 곳이죠.

이 출구를 **기본 게이트웨이**라고 합니다. 보통 `192.168.0.1` 같은 주소를 가지고 있어요. 3편 숙제에서 `ipconfig`를 쳤을 때 함께 나오는 "기본 게이트웨이"가 바로 이 공유기입니다. 우리 컴퓨터 입장에서는 이 출구 하나만 알면 세상 어디로든 보낼 수 있습니다.

## 4. 몇 번 갈아탔을까: 홉, 그리고 돌아가기

택배는 웹 서버까지 가는 동안 라우터를 여러 개 지납니다. 라우터 하나를 지날 때마다 한 칸, 이걸 **홉**이라고 셉니다. 하나, 둘, 셋.

<figure class="sketch">
<svg viewBox="0 0 720 210" role="img" aria-label="집에서 출발한 택배가 라우터를 하나 지날 때마다 1홉씩 세며 웹 서버로 가고, 한 길이 막히면 다른 라우터를 거쳐 돌아간다">
  <defs>
    <marker id="ep6-m3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep6-m4" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 라우터 하나 = 1홉, 막히면 돌아간다</text>
  <rect x="10" y="60" width="100" height="44" rx="8" class="sk-box"/><text x="60" y="87" text-anchor="middle" class="sk-label">우리 집</text>
  <circle cx="190" cy="82" r="24" class="sk-box-accent"/><text x="190" y="87" text-anchor="middle" class="sk-sub">1홉</text>
  <circle cx="320" cy="82" r="24" class="sk-box-accent"/><text x="320" y="87" text-anchor="middle" class="sk-sub">2홉</text>
  <circle cx="450" cy="82" r="24" class="sk-box-muted"/><text x="450" y="87" text-anchor="middle" class="sk-sub">막힘</text>
  <circle cx="450" cy="162" r="24" class="sk-box-accent"/><text x="450" y="167" text-anchor="middle" class="sk-sub">3홉</text>
  <rect x="600" y="60" width="110" height="44" rx="8" class="sk-box"/><text x="655" y="87" text-anchor="middle" class="sk-label">웹 서버</text>
  <path d="M112,82 L164,82" class="sk-line" marker-end="url(#ep6-m3)"/>
  <path d="M216,82 L294,82" class="sk-line" marker-end="url(#ep6-m3)"/>
  <path d="M346,82 L424,82" class="sk-line"/>
  <path d="M476,82 L598,82" class="sk-line"/>
  <path d="M336,100 L428,150" class="sk-line-accent" marker-end="url(#ep6-m4)"/>
  <path d="M474,152 L604,104" class="sk-line-accent" marker-end="url(#ep6-m4)"/>
  <text x="560" y="160" text-anchor="middle" class="sk-sub">라우터끼리 알려 주고 우회</text>
</svg>
<figcaption>길이 하나뿐이 아니라서, 갈림길 하나가 고장 나도 인터넷 전체는 멈추지 않는다.</figcaption>
</figure>

한 길이 막히면 라우터들은 서로 알려 주고 다른 길로 돌아갑니다. 그래서 인터넷은 어딘가 고장이 나도 웬만해선 멈추지 않아요. 2편에서 본 "네트워크들의 그물"이 이렇게 힘을 발휘합니다.

## 한 줄 정리

> 라우터는 **인터넷 갈림길의 길 안내**. **라우팅 테이블**을 보고 택배를 다음 갈림길로 보낸다.
{: .prompt-info }

## 퀴즈

1. 라우터가 길을 정할 때 보는 것은? (라우팅 테이블 / 연락처)
2. 라우터 하나를 지나는 것을 뭐라고 셀까요? (홉 / 바이트)

<details markdown="1">
<summary>정답 보기</summary>

1. **라우팅 테이블** — 주소마다 어느 길로 보낼지 적힌 표지판 목록이에요.
2. **홉** — 하나, 둘, 셋… 몇 홉을 지났는지 셉니다.

</details>

## 집에서 해보기: 웹 서버까지 몇 홉인지 세어 보기

명령 창(Windows: 명령 프롬프트, Mac: 터미널)에 아래 명령을 입력해 보세요.

```bash
# Windows
tracert naver.com

# Mac
traceroute naver.com
```

한 줄이 라우터 하나, 즉 1홉입니다. 별표(`*`)는 대답하지 않은 라우터라서 괜찮아요. 몇 홉을 지나나요?

## 실무에서 달라지는 점

- **"인터넷만 안 돼요"는 기본 게이트웨이부터 본다.** 같은 LAN 안은 되는데 바깥이 안 된다면, 출구인 기본 게이트웨이 설정이나 공유기 쪽을 먼저 의심하는 게 순서입니다.
- **클라우드에도 라우팅 테이블이 있다.** Azure에서는 서브넷마다 경로 테이블(사용자 정의 경로)을 붙여 "이 대역은 방화벽을 거쳐라" 같은 표지판을 직접 세웁니다. 경로를 잘못 적으면 서버는 멀쩡한데 통신만 끊깁니다.
- **tracert는 "어디서 끊겼나"를 보여 주는 첫 도구다.** M365나 외부 서비스가 느릴 때, 몇 번째 홉에서 지연이 커지는지 보면 우리 쪽 문제인지 바깥 구간 문제인지 가르는 단서가 됩니다.

## 시리즈 다른 편

- [네트워크 기초 EP.05 — MAC 주소와 스위치, 같은 동네 안에서 찾아가기](/posts/network-ep05-mac-switch/)
- 다음 편 **EP.07 포트 — 한 집에 문이 여러 개**: 건물 1층은 은행, 2층은 병원처럼 문마다 하는 일이 달라요.
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
