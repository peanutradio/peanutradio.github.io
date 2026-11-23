---
title: "네트워크 입문 Day 2 — IP는 어떻게 받아 올까: DHCP와 NAT"
date: 2026-11-23 18:00:00 +0900
categories: [Study Notes, 네트워크 입문]
tags: [networking, microsoft-365, hands-on]
description: 네트워크 입문 공부 노트 둘째 날. PC가 켜지자마자 주소를 받아 오는 DHCP의 네 번 대화(DORA), 169.254로 시작하는 주소가 뜻하는 것, 사내 주소가 밖에서 다른 주소로 보이는 NAT를 그림으로 잡고 PowerShell로 확인한다.
---

[Day 1](/posts/network-basics-day01-path-to-cloud/)에서 `ipconfig /all`로 내 PC의 IP 주소가 `192.168.10.57`이라는 걸 봤다.
그런데 나는 이 주소를 어디에도 입력한 적이 없다. 그럼 이 주소는 누가, 언제 준 걸까?
헬프데스크에서 자주 듣는 "인터넷이 안 돼요"의 상당수가 바로 이 단계에서 막힌다.

오늘 Day 2의 결론은 두 줄이다.

- PC는 켜질 때 **DHCP 서버와 네 번 대화(DORA)** 해서 주소를 **빌려** 온다. 아무도 대답하지 않으면 `169.254.x.x`를 스스로 붙이는데, 이 주소로는 인터넷이 안 된다
- 사내 주소(사설 IP)는 밖으로 나갈 때 출구에서 **공인 IP로 바뀐다(NAT)**. 그래서 밖에서 보는 내 주소는 `ipconfig`의 주소와 다르다


---

## 영상으로 보기

이 글의 내용과 실습을 영상으로도 정리했다. (네트워크 실습 랩 Day 2)

{% include embed/youtube.html id='ns0D2WD-bvI' %}

같은 주제의 쇼츠 2편은 이어서 이틀 동안 하나씩 공개된다.

---

## 1. 주소는 네 번의 대화로 빌려 온다 — DHCP

DHCP(Dynamic Host Configuration Protocol)는 네트워크에 들어온 장비에게 주소와 기본 설정을 자동으로 나눠 주는 규칙이다.
집에서는 공유기가, 회사에서는 보통 사내 DHCP 서버가 이 일을 한다.

<figure class="sketch">
<svg viewBox="0 0 720 300" role="img" aria-label="DHCP 네 단계 DORA — PC가 Discover로 묻고, 서버가 Offer로 주소를 제안하고, PC가 Request로 그 주소를 쓰겠다고 하고, 서버가 Ack로 확정한다">
  <defs>
    <marker id="nd2-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="nd2-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① DHCP — 네 번의 대화 (DORA)</text>

  <rect x="10" y="36" width="130" height="62" rx="8" class="sk-box"/>
  <text x="75" y="62" text-anchor="middle" class="sk-label">내 PC</text>
  <text x="75" y="82" text-anchor="middle" class="sk-sub">아직 주소 없음</text>
  <rect x="580" y="36" width="130" height="62" rx="8" class="sk-box-accent"/>
  <text x="645" y="62" text-anchor="middle" class="sk-label">DHCP 서버</text>
  <text x="645" y="82" text-anchor="middle" class="sk-sub">공유기·사내 서버</text>
  <path d="M75,100 L75,286" class="sk-line" stroke-dasharray="4 4"/>
  <path d="M645,100 L645,286" class="sk-line" stroke-dasharray="4 4"/>

  <path d="M80,130 L638,130" class="sk-line" marker-end="url(#nd2-m1)"/>
  <text x="360" y="122" text-anchor="middle" class="sk-label">1 Discover — "주소 줄 분 계세요?"</text>
  <path d="M640,170 L82,170" class="sk-line-accent" marker-end="url(#nd2-a1)"/>
  <text x="360" y="162" text-anchor="middle" class="sk-label">2 Offer — "192.168.10.57 어때요?"</text>
  <path d="M80,210 L638,210" class="sk-line" marker-end="url(#nd2-m1)"/>
  <text x="360" y="202" text-anchor="middle" class="sk-label">3 Request — "그 주소 쓸게요"</text>
  <path d="M640,250 L82,250" class="sk-line-accent" marker-end="url(#nd2-a1)"/>
  <text x="360" y="242" text-anchor="middle" class="sk-label">4 Ack — "확정, 기간 동안 빌려 줄게요"</text>
  <text x="360" y="282" text-anchor="middle" class="sk-sub">Discover·Request는 방송(브로드캐스트) · UDP 67/68번 포트</text>
</svg>
<figcaption>주소는 '받는' 게 아니라 '빌리는' 것이다. 그래서 기간(임대, lease)이 있다.</figcaption>
</figure>

### 기억할 세 가지

- **임대(lease)**

  주소에는 빌린 기간이 있다. Windows PC는 기간의 절반쯤 지나면 같은 서버에 갱신을 요청하고, 87.5%가 지나도 답이 없으면 아무 DHCP 서버에게나 다시 묻는다.

- **기본 게이트웨이·DNS 서버도 같이 온다**

  DHCP는 주소만 주지 않는다. Day 1에서 메모한 기본 게이트웨이, Day 3에서 볼 DNS 서버 주소도 이 대화로 함께 받아 온다.

- **169.254.x.x = 대답을 못 받았다는 신호**

  Discover를 보냈는데 아무 서버도 대답하지 않으면, Windows는 `169.254`로 시작하는 주소(APIPA)를 스스로 붙인다. 같은 망의 옆 PC와는 겨우 말이 통할지 몰라도, 게이트웨이가 없어서 인터넷은 안 된다.

> "IP가 169.254로 시작하고 인터넷이 안 돼요"라는 문의는 PC 문제가 아니라 **DHCP 대화가 실패한 것**부터 의심한다. 무선 연결 상태, 랜선, 공유기·사내 DHCP 서버 순서로 본다.
{: .prompt-tip }

---

## 2. 사내 주소는 출구에서 바뀐다 — 사설 IP와 NAT

`192.168.10.57` 같은 주소는 **사설 IP**다. 아래 세 범위는 회사·집 안에서만 쓰라고 정해 둔 주소라, 인터넷에서는 이 주소로 길을 찾지 않는다.

| 사설 IP 범위 | 흔히 보는 곳 |
|---|---|
| `10.0.0.0` ~ `10.255.255.255` | 큰 회사 사내망 |
| `172.16.0.0` ~ `172.31.255.255` | 회사망, 일부 휴대폰 핫스팟 |
| `192.168.0.0` ~ `192.168.255.255` | 집 공유기, 작은 사무실 |

그럼 사설 IP를 쓰는 PC가 어떻게 Microsoft 365와 통신할까? 출구(라우터·방화벽)가 나가는 요청의 **출발 주소를 자기의 공인 IP로 바꿔서** 내보내고, 돌아온 응답은 다시 원래 PC에게 돌려준다. 이것이 NAT(Network Address Translation)다.

<figure class="sketch">
<svg viewBox="0 0 720 260" role="img" aria-label="NAT — 사내 PC 세 대의 사설 IP가 출구에서 공인 IP 하나로 바뀌어 인터넷으로 나간다">
  <defs>
    <marker id="nd2-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="nd2-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② NAT — 사설 IP가 공인 IP로 바뀐다</text>

  <rect x="4" y="30" width="420" height="220" rx="10" class="sk-box-muted"/>
  <text x="16" y="50" class="sk-sub">사내망 · 192.168.10.0/24</text>
  <rect x="20" y="64" width="150" height="48" rx="8" class="sk-box"/>
  <text x="95" y="84" text-anchor="middle" class="sk-label">PC 1</text>
  <text x="95" y="102" text-anchor="middle" class="sk-sub">192.168.10.57</text>
  <rect x="20" y="124" width="150" height="48" rx="8" class="sk-box"/>
  <text x="95" y="144" text-anchor="middle" class="sk-label">PC 2</text>
  <text x="95" y="162" text-anchor="middle" class="sk-sub">192.168.10.58</text>
  <rect x="20" y="184" width="150" height="48" rx="8" class="sk-box"/>
  <text x="95" y="204" text-anchor="middle" class="sk-label">PC 3</text>
  <text x="95" y="222" text-anchor="middle" class="sk-sub">192.168.10.59</text>
  <path d="M172,88 L268,132" class="sk-line" marker-end="url(#nd2-m2)"/>
  <path d="M172,148 L268,148" class="sk-line" marker-end="url(#nd2-m2)"/>
  <path d="M172,208 L268,164" class="sk-line" marker-end="url(#nd2-m2)"/>
  <rect x="274" y="106" width="140" height="84" rx="10" class="sk-box-accent"/>
  <text x="344" y="134" text-anchor="middle" class="sk-label">출구 · NAT</text>
  <text x="344" y="154" text-anchor="middle" class="sk-sub">안쪽 192.168.10.1</text>
  <text x="344" y="172" text-anchor="middle" class="sk-sub">바깥 203.0.113.9</text>
  <path d="M416,148 L540,148" class="sk-line-accent" marker-end="url(#nd2-a2)"/>
  <text x="478" y="138" text-anchor="middle" class="sk-sub">출발 주소</text>
  <text x="478" y="170" text-anchor="middle" class="sk-sub">203.0.113.9로</text>
  <rect x="546" y="106" width="170" height="84" rx="10" class="sk-box"/>
  <text x="631" y="138" text-anchor="middle" class="sk-label">인터넷 · M365</text>
  <text x="631" y="160" text-anchor="middle" class="sk-sub">세 PC가 모두</text>
  <text x="631" y="176" text-anchor="middle" class="sk-sub">203.0.113.9로 보인다</text>
</svg>
<figcaption>밖에서 보면 사무실 전체가 공인 IP 하나다. 재택·핫스팟에서는 이 공인 IP가 달라진다.</figcaption>
</figure>

### 같이 알아 둘 것

- **/24 (서브넷)**

  `192.168.10.0/24`는 앞 24비트(`192.168.10`)가 같은 주소끼리 한 동네라는 뜻이다. 동네 안 주소는 256개지만, 맨 앞(`.0`, 동네 이름)과 맨 끝(`.255`, 방송용)을 빼서 장비에 줄 수 있는 건 254개다.

- **기본 게이트웨이**

  동네 밖으로 나가는 출입문이다. 목적지가 같은 동네가 아니면 PC는 일단 게이트웨이로 보낸다.

- **M365에서 달라 보이는 것**

  사무실과 재택의 공인 IP가 다르니, Microsoft 365 쪽 로그인 기록에는 접속 위치(IP)가 다르게 남는다. 회사가 "사내 IP에서만 접속 허용" 같은 조건을 걸어 두었다면 재택에서 결과가 달라지는 이유도 여기에 있다.

---

## 3. 오늘의 용어

| 약어 | 풀네임 | 한 줄 뜻 |
|---|---|---|
| DHCP | Dynamic Host Configuration Protocol | 주소와 기본 설정을 자동으로 빌려 주는 규칙 |
| DORA | Discover · Offer · Request · Acknowledge | DHCP 네 번의 대화 |
| APIPA | Automatic Private IP Addressing | DHCP 실패 시 붙는 169.254.x.x |
| NAT | Network Address Translation | 사설 IP를 공인 IP로 바꿔 내보냄 |
| CIDR | Classless Inter-Domain Routing | `/24`처럼 동네 크기를 적는 표기 |
| ARP | Address Resolution Protocol | 같은 동네에서 IP로 MAC 주소 찾기 |

---

## 4. 실습 — 내 주소가 어디서 왔는지 확인하기

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="실습 명령이 보는 곳 — ipconfig all과 Get-NetIPConfiguration은 PC가 받은 설정, release와 renew는 DHCP 대화, arp는 게이트웨이의 MAC, 브라우저 검색은 밖에서 보이는 공인 IP">
  <defs>
    <marker id="nd2-m3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">③ 명령마다 보는 곳</text>
  <rect x="10" y="40" width="150" height="60" rx="8" class="sk-box"/>
  <text x="85" y="66" text-anchor="middle" class="sk-label">내 PC</text>
  <text x="85" y="86" text-anchor="middle" class="sk-sub">받아 온 설정</text>
  <path d="M162,70 L266,70" class="sk-line" marker-end="url(#nd2-m3)"/>
  <rect x="272" y="40" width="170" height="60" rx="8" class="sk-box-accent"/>
  <text x="357" y="66" text-anchor="middle" class="sk-label">게이트웨이·DHCP</text>
  <text x="357" y="86" text-anchor="middle" class="sk-sub">192.168.10.1</text>
  <path d="M444,70 L548,70" class="sk-line" marker-end="url(#nd2-m3)"/>
  <rect x="554" y="40" width="160" height="60" rx="8" class="sk-box"/>
  <text x="634" y="66" text-anchor="middle" class="sk-label">인터넷</text>
  <text x="634" y="86" text-anchor="middle" class="sk-sub">공인 IP로 보임</text>
  <text x="85" y="128" text-anchor="middle" class="sk-sub">ipconfig /all</text>
  <text x="85" y="146" text-anchor="middle" class="sk-sub">Get-NetIPConfiguration</text>
  <text x="214" y="128" text-anchor="middle" class="sk-sub">release · renew</text>
  <text x="214" y="146" text-anchor="middle" class="sk-sub">(DORA 다시)</text>
  <text x="357" y="128" text-anchor="middle" class="sk-sub">arp -a</text>
  <text x="357" y="146" text-anchor="middle" class="sk-sub">(게이트웨이 MAC)</text>
  <text x="634" y="128" text-anchor="middle" class="sk-sub">브라우저 검색</text>
  <text x="634" y="146" text-anchor="middle" class="sk-sub">"what is my ip"</text>
  <text x="360" y="186" text-anchor="middle" class="sk-sub">결과의 주소·이름은 모두 설명용 예시 값이다</text>
</svg>
</figure>

### 실습 1. DHCP가 준 설정 읽기

```powershell
ipconfig /all
```

```text
무선 LAN 어댑터 Wi-Fi:
   DHCP 사용 . . . . . . . . . : 예
   IPv4 주소 . . . . . . . . . : 192.168.10.57(기본 설정)
   임대 시작 날짜. . . . . . . : 2026년 11월 23일 월요일 오전 9:02:11
   임대 만료 날짜. . . . . . . : 2026년 11월 24일 화요일 오전 9:02:11
   기본 게이트웨이 . . . . . . : 192.168.10.1
   DHCP 서버 . . . . . . . . . : 192.168.10.1
   DNS 서버. . . . . . . . . . : 192.168.10.1
```

- **DHCP 사용: 예** = 자동으로 받은 주소다
- **임대 시작·만료** = 빌린 기간
- **DHCP 서버** = 주소를 빌려 준 장비. 집에서는 보통 공유기 주소와 같다

### 실습 2. 주소를 반납하고 다시 받기

```powershell
ipconfig /release
ipconfig /renew
```

`/release` 직후에는 주소 칸이 비고 인터넷이 잠깐 끊긴다. `/renew` 뒤에 주소가 다시 채워지면 방금 DORA 네 번의 대화가 다시 일어난 것이다.

> Teams 회의 중이거나 원격으로 접속한 PC에서는 하지 않는다. 연결이 끊긴다. 회사 PC라면 사내 규칙도 먼저 확인한다.
{: .prompt-warning }

### 실습 3. 게이트웨이의 MAC 주소 찾기 (ARP)

```powershell
arp -a
```

```text
인터페이스: 192.168.10.57 --- 0xc
  인터넷 주소           물리적 주소           유형
  192.168.10.1          88-36-6c-12-ab-cd     동적
  192.168.10.255        ff-ff-ff-ff-ff-ff     정적
```

기본 게이트웨이 IP 옆이 게이트웨이의 MAC 주소다. **동적**은 ARP로 자동으로 알아낸 값이라는 뜻이고, `ff-ff-ff-ff-ff-ff`는 방송용 주소라 신경 쓰지 않아도 된다.
Day 1에서 본 "MAC은 다음 라우터까지"의 그 MAC이 바로 이것이다.

### 실습 4. 밖에서 보이는 내 주소

Edge에서 Bing을 열고 `what is my ip`를 검색한다. 맨 위에 보이는 주소를 실습 1의 IPv4 주소와 비교한다.
두 주소가 다르면 정상이다. PC는 `192.168.x.x`인데 밖에서는 공인 IP로 보인다. NAT가 일하고 있다는 증거다.
휴대폰 핫스팟에 붙여서 한 번 더 해 보면, PC 주소도 밖에서 보이는 주소도 둘 다 바뀌는 것을 볼 수 있다.

### 실습 5. 한 화면으로 요약 보기

```powershell
Get-NetIPConfiguration
```

```text
InterfaceAlias       : Wi-Fi
IPv4Address          : 192.168.10.57
IPv4DefaultGateway   : 192.168.10.1
DNSServer            : 192.168.10.1
```

`ipconfig /all`과 IP·게이트웨이·DNS 서버가 같으면 정상이다. 필요한 값만 짧게 보여 주는 PowerShell 명령이다.

---

## 5. 스스로 점검

<details>
<summary>Q1. "IP가 169.254로 시작하고 인터넷이 안 돼요" — 원인과 확인 순서는?</summary>
<p>DHCP 서버에게서 대답을 못 받아 Windows가 스스로 APIPA 주소를 붙인 상태다. Wi-Fi·랜선 연결 → <code>ipconfig /renew</code> → 공유기·사내 DHCP 서버 상태 순서로 확인한다.</p>
</details>

<details>
<summary>Q2. /24에서 장비에 줄 수 있는 주소는 왜 256개가 아니라 254개일까?</summary>
<p>맨 앞 주소(.0)는 네트워크 자체를, 맨 끝 주소(.255)는 동네 전체에 보내는 방송용으로 쓰기 때문이다.</p>
</details>

<details>
<summary>Q3. 사무실과 핫스팟에서 밖에서 보이는 내 주소가 다르면, M365 쪽에서는 무엇이 달라 보일까?</summary>
<p>로그인 기록의 접속 IP(위치)가 달라진다. 회사가 IP 기준 접속 조건을 걸어 두었다면 결과도 달라질 수 있다.</p>
</details>

다음 Day 3에서는 `learn.microsoft.com` 같은 이름이 어떻게 IP 주소로 바뀌는지(DNS)를 정리한다.

---

## 같이 보기

- [Day 1 — 내 PC의 요청은 어떤 길로 클라우드까지 가나](/posts/network-basics-day01-path-to-cloud/)
- [네트워크 기초 EP.03 IP 주소](/posts/network-ep03-ip-address/)
- [네트워크 기초 EP.12 DHCP](/posts/network-ep12-dhcp/)

---

## 참고

- [Troubleshooting guide for DHCP — Microsoft Learn](https://learn.microsoft.com/windows-server/troubleshoot/troubleshoot-dhcp-issue) (DORA, 50%·87.5% 갱신, APIPA, UDP 67/68)
- [ipconfig — Microsoft Learn](https://learn.microsoft.com/windows-server/administration/windows-commands/ipconfig)
- [arp — Microsoft Learn](https://learn.microsoft.com/windows-server/administration/windows-commands/arp)
- [Get-NetIPConfiguration — Microsoft Learn](https://learn.microsoft.com/powershell/module/nettcpip/get-netipconfiguration)
- [RFC 1918 — Address Allocation for Private Internets](https://www.rfc-editor.org/rfc/rfc1918)
