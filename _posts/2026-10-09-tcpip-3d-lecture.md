---
title: "네트워크 기초 EP.01 — 웹페이지 하나가 열리기까지, TCP/IP 4계층 큰 그림"
date: 2026-10-09 08:00:00 +0900
last_modified_at: 2026-10-10 22:00:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [tcp-ip, networking, encapsulation, 3-way-handshake, router, lecture]
description: 내 PC의 요청이 TCP/IP 4계층을 내려가 라우터를 지나 웹 서버에서 다시 올라가는 과정을 3D로 따라가는 약 4분, 5개 챕터 강의와 정리.
---

**주소창에 주소를 치고 엔터를 누른 뒤 1초 안에 무슨 일이 벌어질까.** 「네트워크 기초」 시리즈 첫 편은 그 1초를 천천히 펼쳐 봅니다. 내 PC에서 보낸 요청이 TCP/IP 4계층을 내려가고, 라우터를 지나, 웹 서버에서 다시 올라가는 길을 3D로 따라가요. 약 4분, 5개 챕터이고 음성 해설이 함께 나옵니다.

> 소리를 켜고 보세요.
{: .prompt-tip }


## 강의 보기

<video controls preload="metadata" playsinline poster="/assets/lectures/network-ep01/poster.jpg" style="width:100%;border-radius:12px">
  <source src="/assets/lectures/network-ep01/network-ep01.mp4" type="video/mp4">
</video>


## 오늘의 질문

> 웹페이지 하나가 열릴 때, 데이터는 어떤 길을 지나갈까요?

| 챕터 | 내용 |
|---|---|
| 1. TCP/IP 4계층 한눈에 | 응용 · 전송 · 인터넷 · 네트워크 접근, 각 층이 맡는 일 |
| 2. 연결 맺기: 3-way 핸드셰이크 | SYN → SYN+ACK → ACK |
| 3. 캡슐화: 헤더를 하나씩 붙이기 | 데이터 → 세그먼트 → 패킷 → 프레임 |
| 4. 라우터를 지나며 | 이더넷 헤더는 바꾸고, IP 주소는 그대로 |
| 5. 역캡슐화: 서버가 상자를 여는 순서 | 붙인 순서의 반대로 하나씩 벗기기 |

## 1. TCP/IP 4계층 한눈에

웹사이트 하나를 열 때 데이터는 네 개의 층을 지나갑니다. 위에서부터 차례로 보면 이렇습니다.

- **응용 계층**: 브라우저와 웹 서버가 HTTP라는 약속된 말로 대화하는 곳.
- **전송 계층**: TCP가 데이터를 나눠 보내고, 빠짐없이 도착했는지 챙기는 곳.
- **인터넷 계층**: IP 주소를 보고 목적지까지 가는 길을 찾는 곳.
- **네트워크 접근 계층**: 랜선이나 와이파이로 실제 신호를 내보내는 곳.

가운데 있는 라우터는 아래 두 층만 가지고 있습니다. 길 안내만 하면 되니까, 위쪽 층까지 볼 필요가 없거든요. 이 그림 하나가 앞으로 시리즈 전체의 지도 역할을 합니다.

## 2. 연결 맺기: 3-way 핸드셰이크

TCP는 데이터를 보내기 전에 서버와 먼저 연결을 맺습니다. 신호를 세 번 주고받아서 3-way 핸드셰이크라고 불러요.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="내 PC와 서버가 SYN, SYN+ACK, ACK 세 번의 신호로 연결을 맺는 3-way 핸드셰이크">
  <defs>
    <marker id="ep1-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 말 걸기 전에 세 번 확인한다</text>
  <rect x="40" y="30" width="140" height="34" rx="6" class="sk-box"/><text x="110" y="52" text-anchor="middle" class="sk-label">내 PC</text>
  <rect x="540" y="30" width="140" height="34" rx="6" class="sk-box"/><text x="610" y="52" text-anchor="middle" class="sk-label">웹 서버</text>
  <path d="M110,66 L110,192" class="sk-line"/><path d="M610,66 L610,192" class="sk-line"/>
  <path d="M112,86 L606,112" class="sk-line-accent" marker-end="url(#ep1-m1)"/>
  <text x="360" y="90" text-anchor="middle" class="sk-sub">SYN · "연결해도 될까요?"</text>
  <path d="M608,126 L114,152" class="sk-line-accent" marker-end="url(#ep1-m1)"/>
  <text x="360" y="130" text-anchor="middle" class="sk-sub">SYN+ACK · "좋아요, 저도 준비됐어요"</text>
  <path d="M112,166 L606,186" class="sk-line-accent" marker-end="url(#ep1-m1)"/>
  <text x="360" y="170" text-anchor="middle" class="sk-sub">ACK · "확인했어요" → 연결 열림</text>
</svg>
<figcaption>세 번째 ACK가 도착하면 두 컴퓨터의 전송 계층 사이에 믿을 수 있는 통로가 생긴다.</figcaption>
</figure>

## 3. 캡슐화: 헤더를 하나씩 붙이기

보낼 내용은 "index.html 페이지를 주세요"라는 HTTP 요청입니다. 이 데이터가 아래 계층으로 내려가면서 헤더라는 꼬리표를 하나씩 붙입니다. 붙일 때마다 부르는 이름도 바뀝니다.

| 계층 | 붙이는 것 | 담기는 정보 | 이름 |
|---|---|---|---|
| 응용 | (없음) | HTTP 요청 | 데이터 |
| 전송 | TCP 헤더 | 출발·도착 **포트 번호** | 세그먼트 |
| 인터넷 | IP 헤더 | 출발·도착 **IP 주소** | 패킷 |
| 네트워크 접근 | 이더넷 헤더 + FCS | **MAC 주소**, 오류 검사값 | 프레임 |

편지를 봉투에 넣고, 그 봉투를 다시 상자에 넣는 것과 같습니다. 중요한 점은 **각 계층은 자기가 붙인 헤더만 읽고 쓴다**는 것입니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="TCP/IP 4계층을 내려가며 헤더가 붙고, 서버에서 올라가며 헤더가 벗겨지는 캡슐화와 역캡슐화">
  <defs>
    <marker id="ep1-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 내려갈 때 붙이고, 올라갈 때 벗긴다</text>
  <text x="120" y="40" text-anchor="middle" class="sk-label">내 PC</text>
  <text x="600" y="40" text-anchor="middle" class="sk-label">웹 서버</text>
  <rect x="40" y="52" width="160" height="34" rx="6" class="sk-box"/><text x="120" y="74" text-anchor="middle" class="sk-sub">응용 · 데이터</text>
  <rect x="40" y="92" width="160" height="34" rx="6" class="sk-box"/><text x="120" y="114" text-anchor="middle" class="sk-sub">전송 · +TCP 헤더</text>
  <rect x="40" y="132" width="160" height="34" rx="6" class="sk-box"/><text x="120" y="154" text-anchor="middle" class="sk-sub">인터넷 · +IP 헤더</text>
  <rect x="40" y="172" width="160" height="34" rx="6" class="sk-box-accent"/><text x="120" y="194" text-anchor="middle" class="sk-sub">접근 · +이더넷/FCS</text>
  <rect x="520" y="52" width="160" height="34" rx="6" class="sk-box"/><text x="600" y="74" text-anchor="middle" class="sk-sub">응용 · 요청 읽기</text>
  <rect x="520" y="92" width="160" height="34" rx="6" class="sk-box"/><text x="600" y="114" text-anchor="middle" class="sk-sub">전송 · 포트 443 확인</text>
  <rect x="520" y="132" width="160" height="34" rx="6" class="sk-box"/><text x="600" y="154" text-anchor="middle" class="sk-sub">인터넷 · 내 IP 확인</text>
  <rect x="520" y="172" width="160" height="34" rx="6" class="sk-box-accent"/><text x="600" y="194" text-anchor="middle" class="sk-sub">접근 · 내 MAC 확인</text>
  <rect x="300" y="172" width="120" height="34" rx="6" class="sk-box-muted"/><text x="360" y="194" text-anchor="middle" class="sk-sub">라우터</text>
  <path d="M202,189 L298,189" class="sk-line" marker-end="url(#ep1-m2)"/>
  <path d="M422,189 L518,189" class="sk-line" marker-end="url(#ep1-m2)"/>
  <path d="M30,60 L30,190" class="sk-line-accent" marker-end="url(#ep1-m2)"/>
  <path d="M690,190 L690,60" class="sk-line-accent" marker-end="url(#ep1-m2)"/>
</svg>
<figcaption>보내는 쪽은 아래로 내려가며 헤더를 붙이고, 받는 쪽은 위로 올라가며 같은 순서를 거꾸로 밟는다.</figcaption>
</figure>

## 4. 라우터를 지나며

완성된 프레임이 케이블을 타고 라우터에 도착하면, 라우터는 이렇게 일합니다.

1. 이더넷 헤더와 FCS를 떼어냅니다. 이 구간에서 할 일은 끝났으니까요.
2. IP 헤더의 목적지 주소(강의 예시: 203.0.113.5)를 읽고, 라우팅 테이블을 보고 어느 쪽으로 보낼지 정합니다.
3. 다음 구간용 **새 이더넷 헤더**를 붙여서 내보냅니다.

그래서 **MAC 주소는 구간마다 바뀌지만, IP 주소는 끝까지 그대로**입니다. 이번 편 퀴즈의 정답이기도 해요.

## 5. 역캡슐화: 서버가 상자를 여는 순서

서버에 도착한 프레임은 붙일 때와 정반대 순서로 열립니다. 네트워크 접근 계층은 MAC 주소가 내 것인지 확인하고 이더넷 헤더와 FCS를 떼고, 인터넷 계층은 목적지 IP가 자기 주소인지 보고 IP 헤더를 떼고, 전송 계층은 포트 443을 보고 데이터를 웹 서버 프로그램에 넘깁니다. 웹 서버는 그제야 요청을 읽고 보내줄 페이지를 준비하죠. 서버의 응답은 이 과정을 거꾸로 한 번 더 하는 것입니다.

## 한 줄 정리

> 데이터는 보낼 때 위에서 아래로 헤더를 붙이고, 받을 때 아래에서 위로 떼어내며 네 개의 층을 지나간다.
{: .prompt-info }

## 퀴즈

1. TCP/IP는 몇 개의 층으로 되어 있을까요? (4개 / 7개)
2. 라우터를 지날 때마다 바뀌는 주소는? (MAC 주소 / IP 주소)

<details markdown="1">
<summary>정답 보기</summary>

1. **4개** — 응용 · 전송 · 인터넷 · 네트워크 접근.
2. **MAC 주소** — MAC 주소는 구간마다 바뀌고, IP 주소는 끝까지 그대로예요.

</details>

## 집에서 해보기: ping

데이터가 서버까지 갔다 오는 걸 직접 확인해 봅시다.

```bash
# Windows (명령 프롬프트)
ping naver.com

# Mac (터미널)
ping -c 4 naver.com
```

응답 시간(ms)이 보이면, 내 데이터가 네 층을 내려갔다가 서버에서 다시 올라오며 왕복한 거예요.

## 실무에서 달라지는 점

- **장애 분석은 계층 순서로 좁힌다.** "웹이 안 열려요" 문의를 받으면 링크(케이블·와이파이) → IP(ping) → 전송(포트 열림 여부) → 응용(HTTP 응답 코드) 순으로 내려가며 확인하면 원인을 빠르게 좁힐 수 있습니다.
- **방화벽 규칙은 계층 언어로 읽힌다.** Azure NSG나 Windows 방화벽 규칙의 "IP·포트·프로토콜(TCP/UDP)"은 각각 인터넷 계층과 전송 계층 정보입니다. 이번 편의 캡슐화 표가 곧 규칙 화면의 열 구성입니다.
- **ping이 된다고 서비스가 되는 건 아니다.** ping(ICMP)은 인터넷 계층까지만 확인합니다. 웹 서비스 점검은 443 포트 연결까지 봐야 합니다(Windows: `Test-NetConnection 사이트주소 -Port 443`).

## 시리즈 다른 편

- 다음 편: [네트워크 기초 EP.02 — 네트워크가 뭘까? 컴퓨터끼리 말하는 법](/posts/network-ep02-what-is-network/) — 실 전화기에서 시작해 우리 집 LAN, 온 세상 WAN과 인터넷까지.
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
