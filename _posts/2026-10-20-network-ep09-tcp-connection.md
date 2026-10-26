---
title: "네트워크 기초 EP.09 — 연결을 맺고 끊기, 전화 예절로 배우는 TCP"
date: 2026-10-20 07:55:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [networking, lecture]
description: 여보세요로 시작하는 전화 예절처럼 TCP가 3-way로 연결을 맺고, 4-way로 예의 있게 끊고, 안 들리면 다시 보내는 과정을 따라가는 약 2분 강의와 정리.
---

**TCP는 세 번 주고받아 연결을 맺고, 네 번 주고받아 끊습니다. 중간에 빠지면 다시 보냅니다.** 이번 편은 이 과정을 우리가 매일 하는 전화 통화에 그대로 겹쳐 봅니다. 약 2분, 4개 챕터의 3D 강의예요.

> 소리를 켜고 보세요. 약 2분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='mY1YE_30Vq8' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [네트워크 기초 재생목록](https://www.youtube.com/playlist?list=PLXk7eX-lR6ys) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }


## 오늘의 질문

> TCP는 데이터를 보내기 전에 꼭 연결부터 맺어요. 어떻게 맺고, 어떻게 끊을까요?

[지난 편](/posts/network-ep08-tcp-udp/)에서는 등기우편 같은 TCP와 엽서 같은 UDP를 비교했습니다. 이번에는 그 등기우편이 오가기 전과 후, 연결을 여는 순간과 닫는 순간을 들여다봅니다.

| 챕터 | 내용 |
|---|---|
| 1. 전화 예절: 여보세요? | 용건 전에 서로 들리는지부터 확인 |
| 2. 연결 맺기: 3-way 핸드셰이크 | SYN → SYN+ACK → ACK |
| 3. 예의 있게 끊기: 4-way 종료 | FIN → ACK → FIN → ACK |
| 4. 안 들리면 다시 말해요: 재전송 | 확인이 안 오면 그 부분만 다시 |

## 1. 전화 예절: 여보세요?

전화를 걸면 바로 용건부터 말하지 않죠. 먼저 서로 들리는지 확인합니다.

- "여보세요?"
- "네, 들려요."
- "저도 잘 들려요, 말씀하세요."

이렇게 세 번 주고받고 나서야 본격적으로 대화를 시작합니다. 어색해 보여도, 이 짧은 확인 덕분에 양쪽 모두 "상대가 내 말을 들을 준비가 됐다"는 걸 알게 됩니다.

## 2. 연결 맺기: 3-way 핸드셰이크

TCP도 똑같습니다. 1편에서 잠깐 봤던 **3-way 핸드셰이크**예요.

1. 내 PC → 서버: **SYN** — "연결해도 될까요?"
2. 서버 → 내 PC: **SYN + ACK** — "좋아요, 저도 준비됐어요."
3. 내 PC → 서버: **ACK** — "확인했어요."

세 번째 ACK가 도착하면 연결이 열립니다. 이제부터 지난 편에서 본 번호 붙인 상자들이 오갈 수 있어요.

<figure class="sketch">
<svg viewBox="0 0 720 330" role="img" aria-label="연결을 맺을 때는 SYN, SYN+ACK, ACK 세 번을 주고받고, 끊을 때는 FIN, ACK, FIN, ACK 네 번을 주고받는다">
  <defs>
    <marker id="ep9-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep9-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 맺을 때 3번, 끊을 때 4번</text>
  <rect x="60" y="30" width="120" height="34" rx="8" class="sk-box"/>
  <text x="120" y="52" text-anchor="middle" class="sk-label">내 PC</text>
  <rect x="540" y="30" width="120" height="34" rx="8" class="sk-box"/>
  <text x="600" y="52" text-anchor="middle" class="sk-label">서버</text>
  <path d="M120,66 L120,320 M600,66 L600,320" class="sk-line"/>
  <text x="10" y="92" class="sk-sub">맺기</text>
  <path d="M122,90 L598,108" class="sk-line-accent" marker-end="url(#ep9-a1)"/>
  <text x="360" y="92" text-anchor="middle" class="sk-mono">SYN</text>
  <path d="M598,122 L122,140" class="sk-line-accent" marker-end="url(#ep9-a1)"/>
  <text x="360" y="124" text-anchor="middle" class="sk-mono">SYN + ACK</text>
  <path d="M122,154 L598,172" class="sk-line-accent" marker-end="url(#ep9-a1)"/>
  <text x="360" y="156" text-anchor="middle" class="sk-mono">ACK</text>
  <rect x="260" y="182" width="200" height="24" rx="6" class="sk-box-accent"/>
  <text x="360" y="199" text-anchor="middle" class="sk-sub">연결 열림 · 데이터 주고받기</text>
  <text x="10" y="226" class="sk-sub">끊기</text>
  <path d="M122,222 L598,236" class="sk-line" marker-end="url(#ep9-m1)"/>
  <text x="360" y="224" text-anchor="middle" class="sk-mono">FIN</text>
  <path d="M598,248 L122,262" class="sk-line" marker-end="url(#ep9-m1)"/>
  <text x="360" y="250" text-anchor="middle" class="sk-mono">ACK</text>
  <path d="M598,276 L122,290" class="sk-line" marker-end="url(#ep9-m1)"/>
  <text x="360" y="278" text-anchor="middle" class="sk-mono">FIN</text>
  <path d="M122,302 L598,316" class="sk-line" marker-end="url(#ep9-m1)"/>
  <text x="360" y="304" text-anchor="middle" class="sk-mono">ACK</text>
</svg>
<figcaption>끊을 때 한 번이 더 많은 건, 서버가 남은 말을 다 할 때까지 기다려 주기 때문이다.</figcaption>
</figure>

## 3. 예의 있게 끊기: 4-way 종료

전화를 끊을 때도 갑자기 뚝 끊으면 실례죠. 그래서 TCP는 끊을 때 **네 번** 주고받습니다.

1. 내 PC: **FIN** — "이제 그만 보낼게요."
2. 서버: **ACK** — "알겠어요."
3. 서버도 남은 걸 다 보낸 뒤: **FIN** — "저도 끝났어요."
4. 내 PC: **ACK** — "안녕히."

이걸 **4-way 종료**라고 합니다. 핵심은 2번과 3번 사이입니다. 내가 그만 보내겠다고 해도 서버는 아직 할 말이 남아 있을 수 있어요. 그래서 "알겠어요"와 "저도 끝났어요"를 따로 보냅니다. 양쪽 모두 할 말을 다 했는지 확인하고 끊는 거예요.

## 4. 안 들리면 다시 말해요: 재전송

통화 중에 말이 끊기면 "잘 안 들렸어요, 다시 말해 주세요" 하죠. TCP도 보낸 데이터에 대한 확인이 **정해진 시간 안에** 오지 않으면 그 부분을 다시 보냅니다. 이걸 **재전송**이라고 해요.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="데이터를 보낸 뒤 정해진 시간 안에 확인이 오지 않으면 TCP는 같은 데이터를 다시 보내고, 그다음에 확인을 받는다">
  <defs>
    <marker id="ep9-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ep9-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 기다려도 확인이 없으면, 다시 말한다</text>
  <rect x="10" y="40" width="110" height="140" rx="8" class="sk-box"/>
  <text x="65" y="64" text-anchor="middle" class="sk-label">내 PC</text>
  <rect x="600" y="40" width="110" height="140" rx="8" class="sk-box"/>
  <text x="655" y="64" text-anchor="middle" class="sk-label">서버</text>
  <path d="M122,84 L330,84" class="sk-line" marker-end="url(#ep9-m2)"/>
  <text x="226" y="78" text-anchor="middle" class="sk-mono">데이터</text>
  <rect x="335" y="72" width="100" height="24" rx="6" class="sk-box-muted"/>
  <text x="385" y="89" text-anchor="middle" class="sk-sub">중간에 끊김</text>
  <text x="65" y="112" text-anchor="middle" class="sk-sub">정해진 시간 대기…</text>
  <path d="M122,134 L598,134" class="sk-line-accent" marker-end="url(#ep9-a2)"/>
  <text x="360" y="128" text-anchor="middle" class="sk-mono">같은 데이터 재전송</text>
  <path d="M598,160 L122,160" class="sk-line" marker-end="url(#ep9-m2)"/>
  <text x="360" y="176" text-anchor="middle" class="sk-mono">ACK</text>
</svg>
<figcaption>TCP는 "받았다"는 말을 들을 때까지 같은 말을 반복한다 — 그래서 늦어질 순 있어도 빠지진 않는다.</figcaption>
</figure>

그래서 인터넷이 잠깐 불안해도 웹페이지는 끝까지 제대로 열립니다. 지난 편에서 "그냥 넘어간다"고 했던 UDP와 가장 크게 갈리는 지점이 바로 여기예요.

## 한 줄 정리

> TCP는 **3번** 주고받아 연결하고, **4번** 주고받아 끊어요. 안 들리면 **다시 보내기**.
{: .prompt-info }

## 퀴즈

1. TCP가 연결을 맺을 때 주고받는 횟수는? (3번 / 4번)
2. 보낸 데이터의 확인이 안 오면 TCP는? (다시 보낸다 / 그냥 넘어간다)

<details markdown="1">
<summary>정답 보기</summary>

1. **3번** — SYN → SYN+ACK → ACK, 3-way 핸드셰이크예요.
2. **다시 보낸다** — 재전송! 그냥 넘어가는 건 UDP예요.

</details>

## 집에서 해보기: 지금 통화 중인 연결 보기

명령 창(Windows: 명령 프롬프트, Mac: 터미널)에 아래 명령을 입력해 보세요.

```bash
# Windows
netstat -an | findstr ESTABLISHED

# Mac
netstat -an | grep ESTABLISHED
```

`ESTABLISHED`는 지금 연결된 통화, `TIME_WAIT`은 막 끊은 통화예요. 브라우저 탭을 몇 개 닫고 다시 실행해 보면 목록이 바뀌는 게 보입니다.

## 실무에서 달라지는 점

- **"연결 시간 초과"는 3-way 단계에서 막혔다는 신호다.** SYN을 보냈는데 답이 없으면 방화벽이나 NSG가 중간에서 막고 있을 가능성이 큽니다. Windows에서는 `Test-NetConnection 서버주소 -Port 443`으로 TCP 연결이 맺어지는지부터 확인하면 범위가 확 줄어듭니다.
- **TIME_WAIT이 쌓이는 것 자체는 정상이다.** 끊은 직후 잠시 남는 상태라서 연결을 자주 맺고 끊는 서버에서는 많이 보입니다. 숫자만 보고 장애로 판단하기보다 연결을 재사용하는지부터 봅니다.
- **재전송이 많으면 느려진다.** TCP는 결국 도착시키지만, 다시 보내는 만큼 시간이 걸립니다. VPN이나 회선이 불안한 지점에서 "끊기진 않는데 느리다"는 증상이 나오는 이유입니다.

## 시리즈 다른 편

- [네트워크 기초 EP.08 — TCP와 UDP, 등기우편과 엽서의 차이](/posts/network-ep08-tcp-udp/)
- 네트워크 기초 EP.10 — HTTP, 웹이 대화하는 법 (발행 예정)
- 시리즈 전체 목록: [네트워크 기초](/categories/네트워크-기초/)
