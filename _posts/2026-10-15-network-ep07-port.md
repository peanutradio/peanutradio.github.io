---
title: "네트워크 기초 EP.07 — 포트, 한 집에 문이 여러 개"
date: 2026-10-15 07:55:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, security, lecture]
description: 창구가 여러 개인 건물 비유로 포트 번호, 80·443·22번 문, IP 주소와 포트의 조합, 열린 문과 닫힌 문까지 3D로 따라가는 약 3분 강의와 정리.
---

**IP 주소는 건물을 찾고, 포트 번호는 그 안의 정확한 문을 찾습니다. 웹은 80번과 443번 문을 씁니다.** 이번 편은 택배가 웹 서버 건물에 도착한 뒤, 건물 안에서 누구에게 전해지는지를 따라갑니다. 약 3분, 4개 챕터의 3D 강의예요.

> 소리를 켜고 보세요. 약 3분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='cs-dZdfzMvI' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> 택배가 웹 서버 건물에 도착했어요. 그런데 건물 안에 프로그램이 여러 개라면, 누구에게 전해야 할까요?

지금까지 IP 주소로 집 주소를 붙이고, DNS로 이름을 주소로 바꾸고, 스위치와 [지난 편](/posts/network-ep06-router/)의 라우터로 길을 찾는 것까지 봤습니다. 이제 택배는 건물 앞에 와 있습니다. 마지막 남은 질문은 "건물 안 어느 문으로?"입니다.

| 챕터 | 내용 |
|---|---|
| 1. 건물 하나에 창구가 여러 개 | IP는 하나, 프로그램은 여럿 |
| 2. 문마다 번호가 있어요: 포트 | 80, 443, 22번, 그리고 0~65535 |
| 3. IP 주소 + 포트 번호 = 정확한 창구 | 건물과 창구, 답장용 문 |
| 4. 열린 문과 닫힌 문 | 쓰는 문만 열어 둔다 |

## 1. 건물 하나에 창구가 여러 개

건물 하나에 은행, 병원, 우체국 창구가 나란히 있다고 해 볼까요? 건물 주소는 하나지만, 손님은 자기 일에 맞는 창구로 가야 합니다.

컴퓨터도 똑같습니다. 주소인 IP는 하나인데, 그 안에서는 여러 프로그램이 동시에 일하고 있거든요. 웹 서버 프로그램, 원격 접속 프로그램이 한 컴퓨터 안에 같이 있을 수 있습니다. 주소만으로는 그중 누구에게 줄지 정할 수 없습니다.

## 2. 문마다 번호가 있어요: 포트

그래서 컴퓨터에는 프로그램마다 번호가 붙은 문이 있습니다. 이걸 **포트**라고 합니다.

- 웹페이지는 **80번** 문
- 자물쇠가 달린 안전한 웹은 **443번** 문
- 멀리서 서버에 접속할 때는 **22번** 문

포트 번호는 0번부터 65535번까지 있습니다. 문이 정말 많죠.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="IP 주소 203.0.113.5인 건물 하나 안에 80번 웹, 443번 안전한 웹, 22번 원격 접속 문이 있고, 택배는 IP와 포트 번호를 함께 보고 정확한 문으로 들어간다">
  <defs>
    <marker id="ep7-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① IP 주소는 건물, 포트 번호는 그 안의 문</text>
  <rect x="10" y="90" width="170" height="56" rx="8" class="sk-box-accent"/>
  <text x="95" y="112" text-anchor="middle" class="sk-label">택배</text>
  <text x="95" y="132" text-anchor="middle" class="sk-mono">203.0.113.5:443</text>
  <rect x="300" y="36" width="410" height="180" rx="10" class="sk-box-muted"/>
  <text x="505" y="58" text-anchor="middle" class="sk-label">웹 서버 건물</text>
  <text x="505" y="76" text-anchor="middle" class="sk-mono">IP 203.0.113.5</text>
  <rect x="320" y="96" width="120" height="90" rx="8" class="sk-box"/>
  <text x="380" y="132" text-anchor="middle" class="sk-label">80번</text><text x="380" y="152" text-anchor="middle" class="sk-sub">웹</text>
  <rect x="455" y="96" width="120" height="90" rx="8" class="sk-box-accent"/>
  <text x="515" y="132" text-anchor="middle" class="sk-label">443번</text><text x="515" y="152" text-anchor="middle" class="sk-sub">안전한 웹</text>
  <rect x="590" y="96" width="110" height="90" rx="8" class="sk-box"/>
  <text x="645" y="132" text-anchor="middle" class="sk-label">22번</text><text x="645" y="152" text-anchor="middle" class="sk-sub">원격 접속</text>
  <path d="M182,118 C300,118 380,210 512,190" class="sk-line-accent" marker-end="url(#ep7-m1)"/>
</svg>
<figcaption>주소가 같아도 문 번호가 다르면 전혀 다른 프로그램에게 간다.</figcaption>
</figure>

## 3. IP 주소 + 포트 번호 = 정확한 창구

그래서 택배에는 IP 주소와 함께 문 번호까지 적습니다. "`203.0.113.5`의 443번 문", 이렇게요. **IP 주소가 건물을 찾아 주고, 포트 번호가 그 안의 정확한 창구를 찾아 줍니다.**

보내는 쪽도 마찬가지입니다. 답장을 받을 수 있게 내 컴퓨터도 잠깐 쓸 문을 하나 열어 둡니다. [1편](/posts/tcpip-3d-lecture/)에서 본 51234번 같은 번호가 바로 그 답장용 문이에요. 서버는 이 번호를 보고 답장을 정확히 돌려보냅니다.

## 4. 열린 문과 닫힌 문

그런데 문을 전부 열어 두면 어떻게 될까요? 아무나 들어올 수 있겠죠. 그래서 **쓰는 문만 열고, 나머지는 꼭 닫아 둡니다.**

<figure class="sketch">
<svg viewBox="0 0 720 190" role="img" aria-label="웹 서버는 443번 문만 열어 두고 나머지 문은 닫아 두어서, 모르는 손님이 닫힌 문을 두드려도 들어갈 수 없다">
  <defs>
    <marker id="ep7-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
    <marker id="ep7-m3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 쓰는 문만 열고, 나머지는 닫는다</text>
  <rect x="10" y="44" width="150" height="44" rx="8" class="sk-box"/><text x="85" y="71" text-anchor="middle" class="sk-label">웹 방문자</text>
  <rect x="10" y="118" width="150" height="44" rx="8" class="sk-box"/><text x="85" y="145" text-anchor="middle" class="sk-label">모르는 손님</text>
  <rect x="420" y="40" width="130" height="52" rx="8" class="sk-box-accent"/><text x="485" y="62" text-anchor="middle" class="sk-label">443번</text><text x="485" y="80" text-anchor="middle" class="sk-sub">열림</text>
  <rect x="420" y="114" width="130" height="52" rx="8" class="sk-box-muted"/><text x="485" y="136" text-anchor="middle" class="sk-label">그 밖의 문</text><text x="485" y="154" text-anchor="middle" class="sk-sub">닫힘</text>
  <path d="M162,66 L418,66" class="sk-line-accent" marker-end="url(#ep7-m2)"/>
  <path d="M162,140 L418,140" class="sk-line" marker-end="url(#ep7-m3)"/>
  <text x="290" y="56" text-anchor="middle" class="sk-sub">들어감</text>
  <text x="290" y="130" text-anchor="middle" class="sk-sub">두드려도 못 들어감</text>
  <text x="635" y="108" text-anchor="middle" class="sk-sub">열린 문이 적을수록</text>
  <text x="635" y="124" text-anchor="middle" class="sk-sub">지킬 곳도 적다</text>
</svg>
<figcaption>보안의 출발점은 필요 없는 문을 닫아 두는 것이다.</figcaption>
</figure>

모르는 손님이 닫힌 문을 두드려도 들어갈 수 없습니다. 문 단속을 맡는 경비원 이야기는 나중에 방화벽 편에서 자세히 하겠습니다.

## 한 줄 정리

> **IP 주소는 건물**을 찾고, **포트 번호는 그 안의 정확한 문**을 찾는다. 웹은 80번과 443번 문을 쓴다.
{: .prompt-info }

## 퀴즈

1. 자물쇠가 달린 안전한 웹이 쓰는 문 번호는? (443번 / 22번)
2. IP 주소가 건물이라면, 포트는? (건물 안의 문 / 도시 이름)

<details markdown="1">
<summary>정답 보기</summary>

1. **443번** — 주소창에 자물쇠가 보이는 사이트는 443번 문으로 들어가요.
2. **건물 안의 문** — IP + 포트 = 정확한 창구예요.

</details>

## 집에서 해보기: 내 컴퓨터에 열린 문 보기

명령 창(Windows: 명령 프롬프트, Mac: 터미널)에 아래 명령을 입력해 보세요. Windows와 Mac 모두 같습니다.

```bash
netstat -an
```

주소 끝의 `:숫자`(Mac은 `.숫자`)가 포트 번호입니다. `LISTENING`(Mac은 `LISTEN`)이라고 적힌 줄이 손님을 기다리는 열린 문이에요.

## 실무에서 달라지는 점

- **방화벽 규칙은 결국 "어느 IP의 몇 번 문"이다.** Azure의 네트워크 보안 그룹(NSG) 규칙도 원본·대상 주소와 포트 번호를 적는 칸으로 되어 있습니다. 이번 편의 그림이 규칙 한 줄과 그대로 대응합니다.
- **관리용 문은 인터넷에 열지 않는다.** 22번 같은 원격 접속 문을 바깥 전체에 열어 두는 건 "모르는 손님"을 부르는 일입니다. 클라우드에서는 접속 경로를 따로 두고 관리 포트는 닫아 두는 구성을 기본으로 생각하는 게 안전합니다.
- **"서버는 켜져 있는데 접속이 안 돼요"는 포트부터 본다.** 서버 쪽에서 해당 문이 LISTEN 상태인지, 중간 방화벽이 그 번호를 막고 있지 않은지 나눠서 보면 원인이 빨리 좁혀집니다.

## 시리즈 다른 편

- [네트워크 기초 EP.06 — 라우터, 길을 알려 주는 교차로](/posts/network-ep06-router/)
- 다음 편 **EP.08 TCP와 UDP — 등기우편과 엽서**: 확인 도장을 받을까, 빨리 보낼까?
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
