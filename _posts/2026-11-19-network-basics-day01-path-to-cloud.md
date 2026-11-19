---
title: "네트워크 입문 Day 1 — 내 PC의 요청은 어떤 길로 클라우드까지 가나"
date: 2026-11-19 20:20:00 +0900
categories: [Study Notes, 네트워크 입문]
tags: [networking, microsoft-365, hands-on]
description: 10일짜리 네트워크 입문 공부 노트 첫날. PC에서 Microsoft 데이터센터까지 요청이 지나가는 장비와 주소 두 종류(MAC·IP)를 그림으로 잡고, PowerShell 명령 네 개로 내 PC에서 직접 확인한다.
---

M365나 Copilot 관련 일을 하다 보면 결국 이런 문의를 받는다. **"Copilot이 안 돼요."**
그 말 한마디 뒤에는 내 PC, 사내망, 인터넷, 인증, 클라우드 서비스까지 길게 이어진 길이 있다.
어디서 막혔는지 순서대로 짚으려면 그 길부터 알아야 해서, 10일 동안 네트워크 기초를 다시 공부하며 정리하기로 했다.

오늘 Day 1의 결론은 두 줄이다.

- 요청 하나는 **PC → 첫 연결 장비 → 사내망 출구 → 인터넷 → 데이터센터** 순서로 간다
- 길 위에서 쓰는 주소는 두 가지다. **MAC은 다음 라우터까지, IP는 끝까지**

---

## 이 시리즈를 공부하는 방법

하루에 한 주제씩, 같은 순서로 정리한다.

1. **그림** — 개념을 먼저 한 장으로 본다
2. **용어** — 약어는 영어 풀네임과 함께 적는다
3. **실습** — PowerShell 명령을 직접 쳐 보고, 정상 결과와 비교한다
4. **점검** — 질문 몇 개에 내 말로 답해 본다

| Day | 주제 | Day | 주제 |
|---|---|---|---|
| 1 | 길 위의 장비와 주소 | 6 | API와 JSON |
| 2 | IP 주소와 DHCP | 7 | 인증과 권한 |
| 3 | DNS | 8 | M365 연결 (프록시·VPN) |
| 4 | TCP·UDP와 포트 | 9 | Copilot 요청이 처리되는 길 |
| 5 | HTTP와 HTTPS | 10 | 장애 분석 종합 |

실습은 전부 Windows **PowerShell** 하나로 한다. `ipconfig`, `ping` 같은 예전 명령도 PowerShell에서 그대로 돌아간다.

---

## 1. 길 위의 장비 다섯 개

Teams에서 Copilot에게 "지난주 회의 요약해 줘"라고 치면, 그 요청은 아래 장비들을 차례로 지난다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="PC에서 Microsoft 데이터센터까지 요청이 지나가는 다섯 구간 — PC, AP 또는 스위치, 라우터와 방화벽, 인터넷, 데이터센터">
  <defs>
    <marker id="nd1-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="nd1-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 요청이 지나가는 길</text>

  <rect x="4" y="34" width="372" height="132" rx="10" class="sk-box-muted"/>
  <text x="16" y="52" class="sk-sub">사내망 · LAN</text>

  <rect x="16" y="66" width="104" height="60" rx="8" class="sk-box"/>
  <text x="68" y="92" text-anchor="middle" class="sk-label">내 PC</text>
  <text x="68" y="110" text-anchor="middle" class="sk-sub">요청하는 쪽</text>
  <path d="M122,96 L142,96" class="sk-line" marker-end="url(#nd1-m1)"/>

  <rect x="148" y="66" width="100" height="60" rx="8" class="sk-box"/>
  <text x="198" y="92" text-anchor="middle" class="sk-label">AP · 스위치</text>
  <text x="198" y="110" text-anchor="middle" class="sk-sub">Wi-Fi · 랜선</text>
  <path d="M250,96 L268,96" class="sk-line" marker-end="url(#nd1-m1)"/>

  <rect x="274" y="66" width="96" height="60" rx="8" class="sk-box-accent"/>
  <text x="322" y="92" text-anchor="middle" class="sk-label">라우터·방화벽</text>
  <text x="322" y="110" text-anchor="middle" class="sk-sub">출구 · 검문</text>

  <path d="M372,96 L414,96" class="sk-line-accent" marker-end="url(#nd1-a1)"/>
  <rect x="420" y="66" width="120" height="60" rx="8" class="sk-box"/>
  <text x="480" y="92" text-anchor="middle" class="sk-label">인터넷</text>
  <text x="480" y="110" text-anchor="middle" class="sk-sub">통신사 구간</text>
  <path d="M542,96 L578,96" class="sk-line" marker-end="url(#nd1-m1)"/>

  <rect x="584" y="58" width="132" height="76" rx="10" class="sk-box-accent"/>
  <text x="650" y="88" text-anchor="middle" class="sk-label">Microsoft</text>
  <text x="650" y="104" text-anchor="middle" class="sk-label">데이터센터</text>
  <text x="650" y="122" text-anchor="middle" class="sk-sub">응답하는 쪽 · M365</text>

  <path d="M68,182 L314,182" class="sk-line" stroke-dasharray="5 5"/>
  <text x="191" y="198" text-anchor="middle" class="sk-sub">MAC 주소: 라우터를 지날 때마다 바뀐다</text>
  <path d="M68,236 L650,236" class="sk-line-accent" marker-end="url(#nd1-a1)"/>
  <text x="360" y="228" text-anchor="middle" class="sk-sub">IP 주소: 출발부터 도착까지 목적지를 가리킨다</text>
</svg>
<figcaption>"Copilot이 안 돼요"를 풀 때 제일 먼저 물을 것은 '이 다섯 구간 중 어디까지 갔나'다.</figcaption>
</figure>

### 장비마다 하는 일

- **AP·스위치**

  PC를 사내망에 붙여 주는 장비다. 무선이면 AP(Access Point), 유선이면 스위치를 거친다.

- **라우터·방화벽**

  사내망에서 밖으로 나가는 출구다. 길을 정해 주는 일(라우터)과, 무엇을 내보내고 막을지 검사하는 일(방화벽)을 같이 하는 경우가 많다. 회사라면 여기에 프록시가 더 붙기도 한다(Day 8).

- **클라이언트와 서버**

  요청을 보내는 쪽이 클라이언트, 응답하는 쪽이 서버다. 오늘 예시에서는 내 PC의 Teams가 클라이언트, Microsoft 365가 서버다.

---

## 2. 데이터는 겹겹이 싸여서 간다

네트워크는 일을 층(계층)으로 나눠서 처리한다. 인터넷에서 쓰는 방식이 **TCP/IP 4계층**이다.
나는 이걸 **마트료시카 인형**으로 기억한다. 보낼 때는 안쪽 인형부터 하나씩 겉을 씌우고, 받는 쪽은 바깥부터 하나씩 연다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="TCP/IP 4계층을 겹겹이 싸인 상자로 표현 — 가장 안쪽이 응용, 그 바깥이 전송, 인터넷, 네트워크 접근 계층">
  <text x="0" y="14" class="sk-title">② 한 요청에 씌워지는 네 겹</text>

  <rect x="8" y="30" width="460" height="208" rx="12" class="sk-box"/>
  <text x="22" y="50" class="sk-label">네트워크 접근 계층</text>
  <text x="22" y="66" class="sk-sub">Wi-Fi·이더넷 · MAC 주소로 '같은 망 안에서'</text>

  <rect x="32" y="76" width="412" height="152" rx="10" class="sk-box"/>
  <text x="46" y="96" class="sk-label">인터넷 계층 · IP</text>
  <text x="46" y="112" class="sk-sub">출발지·목적지 IP 주소 · '끝까지'</text>

  <rect x="56" y="122" width="364" height="96" rx="8" class="sk-box"/>
  <text x="70" y="142" class="sk-label">전송 계층 · TCP / UDP</text>
  <text x="70" y="158" class="sk-sub">포트 번호 (HTTPS는 443) · 어느 프로그램으로</text>

  <rect x="80" y="168" width="316" height="42" rx="8" class="sk-box-accent"/>
  <text x="238" y="186" text-anchor="middle" class="sk-label">응용 계층 · HTTPS, DNS</text>
  <text x="238" y="202" text-anchor="middle" class="sk-sub">"지난주 회의 요약해 줘"</text>

  <text x="490" y="70" class="sk-label">보낼 때</text>
  <text x="490" y="88" class="sk-sub">안쪽(응용)부터 한 겹씩 씌운다</text>
  <text x="490" y="128" class="sk-label">받을 때</text>
  <text x="490" y="146" class="sk-sub">바깥부터 한 겹씩 벗겨서</text>
  <text x="490" y="162" class="sk-sub">마지막에 앱이 내용을 읽는다</text>
  <text x="490" y="202" class="sk-label">중간 장비는?</text>
  <text x="490" y="220" class="sk-sub">필요한 겹까지만 열어 보고 넘긴다</text>
</svg>
<figcaption>장애가 났을 때 '몇 번째 겹에서 문제가 생겼나'로 생각하면 확인할 곳이 좁혀진다.</figcaption>
</figure>

그래서 주소가 두 종류다. 바깥 겹의 **MAC 주소**는 같은 네트워크 안, 다음 라우터까지만 쓰이고 라우터를 하나 지날 때마다 새로 씌워진다. (AP·스위치는 MAC을 보고 전달만 할 뿐 바꾸지 않는다.)
안쪽 겹의 **IP 주소**는 출발부터 도착까지 '최종 목적지'를 가리킨다.

---

## 3. 오늘의 용어

| 약어 | 풀네임 | 한 줄 뜻 |
|---|---|---|
| LAN | Local Area Network | 사무실·집 안의 내부 네트워크 |
| AP | Access Point | Wi-Fi로 PC를 붙여 주는 장비 |
| MAC | Media Access Control | 네트워크 카드마다 붙은 고유 주소 |
| IP | Internet Protocol | 목적지까지 찾아가는 주소 체계 |
| TCP/IP | Transmission Control Protocol / Internet Protocol | 인터넷 통신 규칙 묶음 |
| ICMP | Internet Control Message Protocol | ping·tracert용 확인 신호 |
| 홉 (hop) | — | 요청이 거쳐 가는 장비 한 칸 |

---

## 4. 실습 — 내 PC에서 길 확인하기

명령 네 개가 각각 길의 어느 부분을 보여 주는지 먼저 그림으로 보고 시작한다.

<figure class="sketch">
<svg viewBox="0 0 720 214" role="img" aria-label="실습 명령 네 개가 경로의 어느 구간을 보여 주는지 — Get-NetAdapter는 PC의 카드, ipconfig는 PC 설정, ping은 한 지점까지 왕복, tracert는 전체 경로">
  <defs>
    <marker id="nd1-m3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">③ 명령 하나 = 길의 한 부분</text>

  <rect x="8" y="34" width="120" height="44" rx="8" class="sk-box"/>
  <text x="68" y="61" text-anchor="middle" class="sk-label">내 PC</text>
  <rect x="236" y="34" width="120" height="44" rx="8" class="sk-box"/>
  <text x="296" y="61" text-anchor="middle" class="sk-label">게이트웨이</text>
  <rect x="460" y="34" width="120" height="44" rx="8" class="sk-box"/>
  <text x="520" y="61" text-anchor="middle" class="sk-label">인터넷</text>
  <rect x="612" y="34" width="104" height="44" rx="8" class="sk-box"/>
  <text x="664" y="61" text-anchor="middle" class="sk-label">목적지</text>
  <path d="M130,56 L232,56" class="sk-line" marker-end="url(#nd1-m3)"/>
  <path d="M358,56 L456,56" class="sk-line" marker-end="url(#nd1-m3)"/>
  <path d="M582,56 L608,56" class="sk-line" marker-end="url(#nd1-m3)"/>

  <rect x="8" y="96" width="120" height="26" rx="6" class="sk-box-accent"/>
  <text x="68" y="113" text-anchor="middle" class="sk-mono">Get-NetAdapter</text>
  <rect x="8" y="128" width="120" height="26" rx="6" class="sk-box-accent"/>
  <text x="68" y="145" text-anchor="middle" class="sk-mono">ipconfig /all</text>

  <path d="M68,176 L296,176" class="sk-line" stroke-dasharray="5 5"/>
  <text x="182" y="170" text-anchor="middle" class="sk-mono">ping 게이트웨이</text>

  <path d="M68,206 L664,206" class="sk-line-accent"/>
  <text x="480" y="199" text-anchor="middle" class="sk-mono">tracert -d 목적지 (한 칸씩 전부)</text>
</svg>
<figcaption>PC 안부터 확인하고, 한 칸씩 바깥으로 넓혀 가는 것이 기본 순서다.</figcaption>
</figure>

### 준비: PowerShell 열기

키보드의 Windows 키를 누르고 `PowerShell`을 입력한 뒤 **Windows PowerShell**을 연다.
`PS C:\Users\이름>`이 보이면 준비가 끝난 것이다. 아래 예시 결과의 주소·이름은 모두 설명용 값이라, 내 PC에서는 다르게 나온다.

### 실습 1. 지금 쓰는 네트워크 카드 찾기

```powershell
Get-NetAdapter | Format-Table Name, Status, LinkSpeed
```

```text
Name      Status        LinkSpeed
----      ------        ---------
Wi-Fi     Up            866.7 Mbps
이더넷    Disconnected  0 bps
```

`Status`가 **Up**인 줄이 지금 쓰고 있는 연결이다. 노트북이면 보통 Wi-Fi가 Up이다.
랜선을 안 꽂았다면 이더넷이 Disconnected로 나오는 게 정상이다.

### 실습 2. 내 MAC 주소와 IP 주소, 출구 찾기

```powershell
ipconfig /all
```

```text
무선 LAN 어댑터 Wi-Fi:
   물리적 주소 . . . . . . . . : AA-BB-CC-12-34-56
   DHCP 사용 . . . . . . . . . : 예
   IPv4 주소 . . . . . . . . . : 192.168.10.57(기본 설정)
   서브넷 마스크 . . . . . . . : 255.255.255.0
   기본 게이트웨이 . . . . . . : 192.168.10.1
   DNS 서버. . . . . . . . . . : 192.168.10.1
```

결과가 길면 Up이었던 어댑터 부분만 본다. 세 줄을 메모해 둔다.

- **물리적 주소** = MAC 주소
- **IPv4 주소** = 내 PC의 IP 주소
- **기본 게이트웨이** = 사내망 출구(그림 ①의 라우터). 다음 실습에서 쓴다

### 실습 3. 출구까지 왕복 시간 재기

```powershell
ping 192.168.10.1        # 실습 2에서 메모한 기본 게이트웨이 주소
```

```text
192.168.10.1의 응답: 바이트=32 시간=2ms TTL=64
192.168.10.1의 응답: 바이트=32 시간=1ms TTL=64
192.168.10.1의 응답: 바이트=32 시간=1ms TTL=64
192.168.10.1의 응답: 바이트=32 시간=3ms TTL=64
    패킷: 보냄 = 4, 받음 = 4, 손실 = 0 (0% 손실)
```

Windows의 `ping`은 기본으로 네 번 보낸다. 게이트웨이는 바로 옆이라 몇 ms 안에 네 번 다 돌아오는 게 보통이다.

> 외부 사이트에 `ping`을 쳤는데 "요청 시간이 만료되었습니다"만 나와도 **서비스 장애라고 단정하면 안 된다.** 보안상 ping(ICMP) 응답을 꺼 둔 서버가 많다. 웹 페이지가 열리는지는 Day 4에서 다른 방법(`Test-NetConnection`)으로 확인한다.
{: .prompt-warning }

### 실습 4. 목적지까지 몇 칸을 지나는지 보기

```powershell
tracert -d learn.microsoft.com
```

```text
  1     2 ms     1 ms     2 ms  192.168.10.1
  2     5 ms     4 ms     5 ms  10.30.0.1
  3    11 ms     9 ms    10 ms  203.0.113.9
  4     *        *        *     요청 시간이 만료되었습니다.
  5    35 ms    34 ms    36 ms  198.51.100.20
  ...
```

한 줄이 한 칸(홉)이다. `-d`는 각 장비의 이름을 찾지 않게 해서 결과가 더 빨리 나온다.

- **1번 줄**은 실습 2의 기본 게이트웨이와 같아야 한다
- **별표(\*)** 는 그 장비가 응답만 안 한 것이라, 다음 줄이 이어지면 길은 살아 있다
- 몇 번째 줄부터 사내 주소(192.168.x, 10.x)가 아닌 주소가 나오는지 보면 **사내망을 벗어나는 지점**을 알 수 있다

끝까지 기다리기 지루하면 `Ctrl+C`로 멈춘다.

---

## 5. 스스로 점검

<details>
<summary>Q1. 그림 ①의 다섯 장비가 각각 하는 일을 한 줄씩 말해 보기</summary>
<p>PC = 요청을 만든다 · AP/스위치 = PC를 사내망에 붙인다 · 라우터/방화벽 = 밖으로 나가는 출구이자 검문 · 인터넷 = 통신사 구간 · 데이터센터 = 요청을 받아 응답한다.</p>
</details>

<details>
<summary>Q2. MAC 주소와 IP 주소는 각각 어디까지 쓰일까?</summary>
<p>MAC은 같은 네트워크 안, 다음 라우터까지만 쓰이고 라우터를 지날 때마다 새로 붙는다. 목적지 IP는 출발지에서 최종 목적지까지 계속 목적지를 가리킨다.</p>
</details>

<details>
<summary>Q3. ping은 실패하는데 웹 페이지는 잘 열린다. 무엇을 의심할까?</summary>
<p>상대 서버나 중간 방화벽이 ICMP(ping) 응답을 막아 둔 것일 가능성이 크다. 웹은 HTTPS(TCP 443)로 따로 통신하므로, ping 결과만으로 서비스 장애를 판단하지 않는다.</p>
</details>

다음 Day 2에서는 내 PC가 `192.168.10.57` 같은 주소를 **어떻게 받아 왔는지(DHCP)**, 그리고 사내 주소가 밖에서는 어떻게 바뀌어 보이는지(NAT)를 정리한다.

---

## 같이 보기

개념을 3D 그림으로 다시 보고 싶다면 네트워크 기초 영상 강의 정리 글이 이어진다.

- [EP.03 IP 주소](/posts/network-ep03-ip-address/)
- [EP.05 MAC 주소와 스위치](/posts/network-ep05-mac-switch/)
- [EP.06 라우터](/posts/network-ep06-router/)

---

## 참고

- [Get-NetAdapter — Microsoft Learn](https://learn.microsoft.com/powershell/module/netadapter/get-netadapter)
- [ipconfig — Microsoft Learn](https://learn.microsoft.com/windows-server/administration/windows-commands/ipconfig)
- [ping — Microsoft Learn](https://learn.microsoft.com/windows-server/administration/windows-commands/ping)
- [tracert — Microsoft Learn](https://learn.microsoft.com/windows-server/administration/windows-commands/tracert)
- [Microsoft 365 네트워크 연결 원칙 — Microsoft Learn](https://learn.microsoft.com/microsoft-365/enterprise/microsoft-365-network-connectivity-principles)
