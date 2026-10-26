---
title: "네트워크 기초 EP.04 — DNS, 이름을 주소로 바꾸는 전화번호부"
date: 2026-10-12 07:55:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, azure, lecture]
description: 휴대폰 연락처 비유로 도메인 이름, DNS 서버에게 묻는 과정, 루트에서 .com을 거쳐 찾아가는 길과 캐시까지 3D로 따라가는 약 3분 강의와 정리.
---

**DNS는 naver.com 같은 이름을 IP 주소로 바꿔 주는 인터넷의 전화번호부입니다.** 우리가 숫자 주소를 하나도 외우지 않고도 웹사이트를 여는 비밀이 여기에 있어요. 약 3분, 4개 챕터의 3D 강의입니다.

> 소리를 켜고 보세요. 약 3분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='SQtY8QFlcLk' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> 우리는 숫자 주소 대신 naver.com 같은 이름을 쓰는데, 컴퓨터는 그 주소를 어떻게 알아낼까요?

[지난 편](/posts/network-ep03-ip-address/)에서는 인터넷 세상의 집 주소인 IP 주소를 배웠습니다. 점으로 나눈 숫자 네 덩어리였죠. 그런데 정작 우리는 그 숫자를 입력한 적이 없습니다. 이번 편은 그 빈칸을 채웁니다.

| 챕터 | 내용 |
|---|---|
| 1. 이름만 알면 돼요: 휴대폰 연락처 | 도메인 이름과 DNS |
| 2. DNS 서버에게 물어봐요 | 이름을 묻고, 주소로 답을 받는다 |
| 3. 전화번호부를 차례로 넘겨요 | 루트 → .com → 네이버 담당 서버 |
| 4. 한 번 찾으면 메모해 둬요: 캐시 | 빠르게, 그리고 정해진 시간 뒤엔 지운다 |

## 1. 이름만 알면 돼요: 휴대폰 연락처

친구 전화번호를 다 외우고 있나요? 대부분은 연락처에서 이름만 누르죠. 이름을 누르면 휴대폰이 알아서 번호를 찾아 전화를 걸어 줍니다.

인터넷도 똑같습니다. 우리는 숫자 주소 대신 `naver.com` 같은 이름을 씁니다. 이걸 **도메인 이름**이라고 해요. 하지만 컴퓨터는 이 이름을 IP 주소로 바꿔야 찾아갈 수 있습니다. 이 일을 해 주는 인터넷의 전화번호부가 바로 **DNS**(Domain Name System)입니다.

## 2. DNS 서버에게 물어봐요

주소창에 `naver.com`을 치면 내 컴퓨터는 먼저 **DNS 서버**에게 물어봅니다. "네이버 주소가 뭐예요?" DNS 서버가 IP 주소로 대답해 주면, 그제야 내 컴퓨터는 그 주소로 웹페이지를 달라고 요청을 보냅니다.

순서가 중요합니다. **이름 → 주소 → 요청.** 이 모든 일이 눈 깜짝할 사이에 일어나기 때문에, 우리는 이름만 쳐도 바로 웹사이트가 열리는 것처럼 느낍니다.

## 3. 전화번호부를 차례로 넘겨요: 루트 → .com → naver

그런데 DNS 서버가 세상 모든 이름을 다 알고 있을까요? 아닙니다. 모르면 차례로 물어봅니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="DNS 서버는 루트 서버에게 .com 담당을 묻고, .com 서버에게 네이버 담당을 묻고, 네이버 담당 서버에게서 진짜 IP 주소를 받아 내 컴퓨터에 알려 준다">
  <defs>
    <marker id="ep4-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep4-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 큰 목차부터 차례로: 루트 → .com → 네이버 담당</text>
  <rect x="10" y="100" width="120" height="50" rx="8" class="sk-box"/>
  <text x="70" y="122" text-anchor="middle" class="sk-label">내 컴퓨터</text>
  <text x="70" y="140" text-anchor="middle" class="sk-mono">naver.com?</text>
  <rect x="200" y="100" width="130" height="50" rx="8" class="sk-box-accent"/>
  <text x="265" y="122" text-anchor="middle" class="sk-label">DNS 서버</text>
  <text x="265" y="140" text-anchor="middle" class="sk-sub">대신 물어봐 줌</text>
  <path d="M132,118 L198,118" class="sk-line" marker-end="url(#ep4-m1)"/>
  <path d="M198,134 L132,134" class="sk-line-accent" marker-end="url(#ep4-m2)"/>
  <text x="165" y="170" text-anchor="middle" class="sk-sub">④ IP 주소</text>
  <rect x="440" y="36" width="270" height="44" rx="8" class="sk-box"/>
  <text x="575" y="56" text-anchor="middle" class="sk-label">루트 서버</text>
  <text x="575" y="72" text-anchor="middle" class="sk-sub">① ".com은 저 서버에 물어봐"</text>
  <rect x="440" y="103" width="270" height="44" rx="8" class="sk-box"/>
  <text x="575" y="123" text-anchor="middle" class="sk-label">.com 서버</text>
  <text x="575" y="139" text-anchor="middle" class="sk-sub">② "네이버 담당은 저 서버야"</text>
  <rect x="440" y="170" width="270" height="44" rx="8" class="sk-box-accent"/>
  <text x="575" y="190" text-anchor="middle" class="sk-label">네이버 담당 서버</text>
  <text x="575" y="206" text-anchor="middle" class="sk-sub">③ 진짜 IP 주소를 알려 줌</text>
  <path d="M332,112 L438,62" class="sk-line" marker-end="url(#ep4-m1)"/>
  <path d="M332,125 L438,125" class="sk-line" marker-end="url(#ep4-m1)"/>
  <path d="M332,138 L438,188" class="sk-line" marker-end="url(#ep4-m1)"/>
</svg>
<figcaption>모든 이름을 아는 서버는 없다. 각 단계가 "다음에 누구에게 물을지"만 알면 된다.</figcaption>
</figure>

먼저 맨 꼭대기 **루트 서버**에게 묻습니다. ".com은 누가 알아요?" 루트가 **.com 서버**를 알려 주면 .com 서버에게 다시 묻고, .com 서버가 **네이버 담당 서버**를 알려 주면 거기서 드디어 진짜 주소를 받습니다. 두꺼운 전화번호부를 큰 목차부터 차례로 넘겨 가며 찾는 것과 같아요.

## 4. 한 번 찾으면 메모해 둬요: 캐시

한 번 찾은 주소는 메모해 둡니다. 자주 거는 번호를 즐겨찾기에 넣어 두는 것처럼요. 다음에 같은 이름을 치면 다시 묻지 않고 메모를 바로 꺼내 씁니다. 훨씬 빠르죠. 이렇게 잠깐 기억해 두는 것을 **캐시**라고 합니다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="처음에는 DNS 서버에게 물어 주소를 받지만, 두 번째부터는 메모해 둔 캐시에서 바로 꺼내 쓰고, 메모는 정해진 시간이 지나면 지운다">
  <defs>
    <marker id="ep4-m3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep4-m4" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 처음엔 묻고, 다음엔 메모를 꺼낸다</text>
  <text x="10" y="54" class="sk-label">처음</text>
  <rect x="80" y="34" width="130" height="34" rx="8" class="sk-box"/><text x="145" y="56" text-anchor="middle" class="sk-sub">naver.com 입력</text>
  <rect x="290" y="34" width="150" height="34" rx="8" class="sk-box"/><text x="365" y="56" text-anchor="middle" class="sk-sub">DNS 서버에게 질문</text>
  <rect x="520" y="34" width="190" height="34" rx="8" class="sk-box"/><text x="615" y="56" text-anchor="middle" class="sk-sub">주소 받고 메모에 적기</text>
  <path d="M212,51 L288,51" class="sk-line" marker-end="url(#ep4-m3)"/>
  <path d="M442,51 L518,51" class="sk-line" marker-end="url(#ep4-m3)"/>
  <text x="10" y="124" class="sk-label">다음</text>
  <rect x="80" y="104" width="130" height="34" rx="8" class="sk-box"/><text x="145" y="126" text-anchor="middle" class="sk-sub">naver.com 입력</text>
  <rect x="290" y="104" width="150" height="34" rx="8" class="sk-box-muted"/><text x="365" y="126" text-anchor="middle" class="sk-sub">질문 생략</text>
  <rect x="520" y="104" width="190" height="34" rx="8" class="sk-box-accent"/><text x="615" y="126" text-anchor="middle" class="sk-label">캐시에서 바로 꺼냄</text>
  <path d="M212,121 C300,160 440,160 518,128" class="sk-line-accent" marker-end="url(#ep4-m4)"/>
  <text x="615" y="168" text-anchor="middle" class="sk-sub">정해진 시간이 지나면 메모는 지운다</text>
</svg>
<figcaption>캐시는 속도를 위해 묻는 과정을 건너뛰고, 유효 시간은 주소가 바뀌는 경우를 대비한다.</figcaption>
</figure>

다만 메모는 정해진 시간이 지나면 지웁니다. 주소가 바뀔 수도 있으니까요. 빠르게 쓰되, 영원히 믿지는 않는 것. 이게 캐시의 균형입니다.

## 한 줄 정리

> DNS는 **naver.com 같은 이름을 IP 주소로 바꿔 주는** 인터넷의 전화번호부.
{: .prompt-info }

## 퀴즈

1. naver.com 같은 이름을 뭐라고 할까요? (도메인 이름 / IP 주소)
2. 한 번 찾은 주소를 잠깐 기억해 두는 것은? (캐시 / 프로토콜)

<details markdown="1">
<summary>정답 보기</summary>

1. **도메인 이름** — 사람이 외우기 쉬운 이름이에요. 컴퓨터는 이걸 IP 주소로 바꿔서 찾아가요.
2. **캐시** — 다음엔 다시 묻지 않고 바로 써서 빨라져요.

</details>

## 집에서 해보기: naver.com의 IP 주소 알아내기

명령 창(Windows: 명령 프롬프트, Mac: 터미널)에 아래 명령을 입력해 보세요. Windows와 Mac 모두 같습니다.

```bash
nslookup naver.com
```

"Address" 또는 "주소" 뒤에 나오는 숫자가 DNS가 찾아 준 네이버의 IP 주소입니다.

## 실무에서 달라지는 점

- **"사이트가 안 열려요"의 상당수는 DNS다.** IP 주소로는 닿는데 이름으로는 안 된다면 길이 아니라 전화번호부 문제입니다. `nslookup`으로 이름이 주소로 바뀌는지부터 확인하면 범위가 확 줄어듭니다.
- **M365 도입은 DNS 레코드 작업에서 시작한다.** 회사 도메인을 Microsoft 365에 연결할 때 도메인 확인과 메일 라우팅을 위해 DNS 레코드를 추가합니다. 전화번호부에 "우리 회사 메일은 여기로"라고 적어 넣는 일입니다.
- **주소를 바꿀 땐 캐시를 계산에 넣는다.** 서버를 Azure로 옮기며 DNS의 주소를 바꿔도, 이미 메모해 둔 곳은 정해진 시간이 지날 때까지 옛 주소를 씁니다. 그래서 전환 직후 "누구는 되고 누구는 안 되는" 상황이 생깁니다.

## 시리즈 다른 편

- [네트워크 기초 EP.03 — IP 주소, 인터넷 세상의 집 주소](/posts/network-ep03-ip-address/)
- 다음 편 **EP.05 MAC 주소와 스위치 — 같은 동네 안에서 찾아가기**: 주민등록번호처럼, 기기마다 하나뿐인 이름이 있어요.
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
