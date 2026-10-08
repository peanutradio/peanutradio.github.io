---
title: "MS 9월 보안 업데이트 — 에이전트 트래픽까지 들어온 Zero Trust"
date: 2026-09-27 10:00:00 +0900
categories: [IT News]
tags: [security, zero-trust, purview, entra, defender, ai-agent]
description: Purview와 Entra Global Secure Access가 에이전트의 업로드까지 네트워크에서 막는 기능이 GA됐고, Defender에는 SIEM과 위협 보호를 합친 ISOC가 프리뷰로 나왔습니다.
---

![Microsoft 레드먼드 캠퍼스의 광장](/assets/img/posts/ms-security-september-2026.jpg)
_사진: [Jonathan Schilling](https://commons.wikimedia.org/wiki/File:Plaza_on_the_Microsoft_Redmond_campus.jpg), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0) — 2006년 촬영한 자료 사진으로, 이번 발표와는 관계없습니다_

Microsoft Security 블로그에 9월 정리 글(9/24)과 Defender SOC 발표 글(9/23)이 올라왔습니다.
에이전트를 만들어 배포하는 입장에서 중요한 건 두 가지입니다.

1. **GA** — Microsoft Purview와 Microsoft Entra Global Secure Access가 사람뿐 아니라 **사용자 대신 움직이는(on-behalf-of, OBO) 에이전트의 트래픽**까지 네트워크에서 검사해, 민감한 데이터가 승인되지 않은 AI 도구로 나가는 걸 막습니다.
2. **프리뷰** — Microsoft Defender에 SIEM과 위협 보호를 한 기반으로 합친 **ISOC(integrated security operations center)**가 나왔습니다.

이 글은 아래 두 원문을 읽고 정리·재구성한 것입니다.

## 1. OBO 에이전트가 올리는 파일도 네트워크에서 막는다

9월 정리 글이 내건 방향 중 하나가 **Zero Trust를 에이전트 트래픽까지 넓히는 것**입니다. 그 핵심이 Purview와 Entra의 조합입니다.

- 상태는 **Now generally available**입니다.
- Purview의 문맥 기반 분류와 정책을 **Entra가 네트워크 계층에서 강제**합니다.
- 민감한 파일과 텍스트를 **실시간으로 찾아서** 위험한 목적지로 공유되지 않게 막습니다.
- 적용 대상은 **사람의 행동과 OBO 에이전트 트래픽** 둘 다입니다.

원문 예시대로, 직원이나 OBO 에이전트가 민감한 문서를 **승인되지 않은 AI 도구**에 올리려 하면 데이터가 나가기 전에 정책이 전송을 멈춥니다.

<figure class="sketch">
<svg viewBox="0 0 720 300" role="img" aria-label="직원과 OBO 에이전트의 업로드가 Entra Global Secure Access를 지나며 Purview 정책으로 검사되고, 승인되지 않은 AI 도구로 가는 전송은 차단된다">
  <defs>
    <marker id="sec-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="sec-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">① 사람이든 에이전트든 같은 네트워크 검사를 지난다</text>

  <rect x="8" y="64" width="150" height="52" rx="8" class="sk-box"/>
  <text x="83" y="88" text-anchor="middle" class="sk-label">직원</text>
  <text x="83" y="105" text-anchor="middle" class="sk-sub">파일·텍스트 업로드</text>

  <rect x="8" y="164" width="150" height="52" rx="8" class="sk-box"/>
  <text x="83" y="188" text-anchor="middle" class="sk-label">OBO 에이전트</text>
  <text x="83" y="205" text-anchor="middle" class="sk-sub">사용자 대신 업로드</text>

  <path d="M160,90 L236,130" class="sk-line" marker-end="url(#sec-m1)"/>
  <path d="M160,190 L236,152" class="sk-line" marker-end="url(#sec-m1)"/>

  <rect x="240" y="78" width="220" height="130" rx="10" class="sk-box-accent"/>
  <text x="350" y="104" text-anchor="middle" class="sk-label">Entra Global Secure Access</text>
  <text x="350" y="122" text-anchor="middle" class="sk-sub">네트워크 계층에서 강제</text>
  <path d="M258,136 L442,136" class="sk-line" stroke-dasharray="4 4"/>
  <text x="350" y="160" text-anchor="middle" class="sk-label">Purview 분류·정책</text>
  <text x="350" y="180" text-anchor="middle" class="sk-sub">민감한 파일·텍스트를 실시간 탐지</text>

  <path d="M462,120 L540,82" class="sk-line" marker-end="url(#sec-m1)"/>
  <path d="M462,166 L540,204" class="sk-line-accent" marker-end="url(#sec-a1)"/>

  <rect x="546" y="56" width="166" height="52" rx="8" class="sk-box"/>
  <text x="629" y="80" text-anchor="middle" class="sk-label">허용된 목적지</text>
  <text x="629" y="97" text-anchor="middle" class="sk-sub">정책을 통과한 전송</text>

  <rect x="546" y="178" width="166" height="52" rx="8" class="sk-box-muted"/>
  <text x="629" y="202" text-anchor="middle" class="sk-label">승인되지 않은 AI 도구</text>
  <text x="629" y="219" text-anchor="middle" class="sk-sub">데이터가 나가기 전에 차단</text>

  <text x="360" y="272" text-anchor="middle" class="sk-sub">원문 예시: 직원이나 OBO 에이전트가 민감한 문서를 승인되지 않은 AI 도구에 올리려 할 때</text>
</svg>
<figcaption>에이전트를 별도 예외로 두지 않고, <strong>사람과 같은 네트워크 검사 경로에 태운다</strong>는 것이 이번 GA의 핵심입니다.</figcaption>
</figure>

에이전트를 배포하는 쪽에서는 OBO 에이전트가 외부 AI 서비스로 보내는 전송이 **조직의 Purview 정책에 막힐 수 있다**는 뜻입니다.
그래서 저는 업로드 실패를 "네트워크 오류"로 뭉개지 말고, 정책 차단을 전제로 에러 처리와 안내를 설계해야 한다고 봅니다.

## 2. Defender의 ISOC — SOC를 시스템 하나로

9/23 글(Rob Lefferts)의 문제의식은 이렇습니다. 공격자는 이미 에이전트로 실행을 자동화하는데, **보호와 운영이 따로 떨어진 시스템**이면 방어가 AI 속도를 따라갈 수 없고 에이전트도 그 복잡함을 물려받는다는 겁니다.

답으로 내놓은 게 **ISOC in Microsoft Defender**입니다. SIEM과 위협 보호를 합쳐, 사람과 에이전트가 같은 기반 위에서 보고 이해하고 행동하게 하는 구조입니다.
원문은 이 기반을 세 층으로 나눕니다.

<figure class="sketch">
<svg viewBox="0 0 720 280" role="img" aria-label="ISOC는 신호와 센서, 맥락, 액추에이터를 하나로 묶고, 그 위에서 사람과 에이전트가 같은 맥락과 통제 수단을 쓴다">
  <defs>
    <marker id="sec-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="sec-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">② ISOC — 세 층을 하나의 보호 루프로</text>

  <rect x="120" y="36" width="220" height="50" rx="8" class="sk-box"/>
  <text x="230" y="58" text-anchor="middle" class="sk-label">사람</text>
  <text x="230" y="75" text-anchor="middle" class="sk-sub">우선순위·판단·목표 설정</text>
  <rect x="380" y="36" width="220" height="50" rx="8" class="sk-box"/>
  <text x="490" y="58" text-anchor="middle" class="sk-label">에이전트</text>
  <text x="490" y="75" text-anchor="middle" class="sk-sub">속도와 규모로 계속 실행</text>

  <path d="M230,88 L230,112" class="sk-line" marker-end="url(#sec-m2)"/>
  <path d="M490,88 L490,112" class="sk-line" marker-end="url(#sec-m2)"/>

  <rect x="16" y="116" width="688" height="100" rx="10" class="sk-box-accent"/>
  <text x="360" y="136" text-anchor="middle" class="sk-mono">ISOC in Microsoft Defender (preview)</text>

  <rect x="40" y="148" width="180" height="52" rx="8" class="sk-box"/>
  <text x="130" y="171" text-anchor="middle" class="sk-label">신호·센서</text>
  <text x="130" y="188" text-anchor="middle" class="sk-sub">가시성</text>
  <rect x="270" y="148" width="180" height="52" rx="8" class="sk-box"/>
  <text x="360" y="171" text-anchor="middle" class="sk-label">맥락</text>
  <text x="360" y="188" text-anchor="middle" class="sk-sub">신호를 이해로</text>
  <rect x="500" y="148" width="180" height="52" rx="8" class="sk-box"/>
  <text x="590" y="171" text-anchor="middle" class="sk-label">액추에이터</text>
  <text x="590" y="188" text-anchor="middle" class="sk-sub">판단을 보호 조치로</text>

  <path d="M222,174 L266,174" class="sk-line-accent" marker-end="url(#sec-a2)"/>
  <path d="M452,174 L496,174" class="sk-line-accent" marker-end="url(#sec-a2)"/>
  <path d="M590,218 C590,256 130,256 130,218" class="sk-line-accent" marker-end="url(#sec-a2)"/>
  <text x="360" y="272" text-anchor="middle" class="sk-sub">통합 보호 루프 — 알게 된 것을 사전 보호 강화로 되돌린다</text>
</svg>
<figcaption>별도의 "에이전트 레이어"를 얹는 게 아니라, <strong>사람과 에이전트가 같은 맥락과 통제 수단을 공유</strong>하게 만드는 것이 ISOC의 주장입니다.</figcaption>
</figure>

원문에서 눈에 띈 문장은 이겁니다.

> 따로 조립해야 하는 에이전트 레이어도, 새로 끼워 맞춰야 하는 운영 모델도 없다.

사례는 Defender의 **attack disruption**입니다. 진행 중인 공격을 끊고 공격자의 다음 이동을 예상합니다. 2026년 7월 **Project Perception**과 함께 소개한 end-to-end cyber stack의 연장선이라고도 밝힙니다.

상태는 원문 표현 그대로 **"available in preview today"**입니다. 본문은 "SIEM과 위협 보호를 합친다"고만 하고 Microsoft Sentinel을 직접 언급하지는 않아서, 구체적인 제품 통합 범위는 추가 문서를 봐야 알 수 있습니다.

## 3. 그 밖의 9월 소식

| 제품 | 내용 |
|---|---|
| Defender + Security Copilot | 이메일 **detonation summary** — URL·파일 샌드박싱 결과를 AI가 설명 |
| Purview auto-labeling | 시뮬레이션 최대 **2,000만 항목**, adaptive scope로 최대 **5만 사이트** |
| Purview eDiscovery | 사용자 소유 SharePoint embedded 컨테이너(Loop, Copilot Pages, Copilot Notebooks 등) 검색·보존·검토·내보내기 |
| Purview Data Lifecycle Management | 비활성 SharePoint 콘텐츠만 아카이브 → Microsoft 365 Copilot 색인에서 빠짐 |
| Intune | Enterprise Application Management, Cloud PKI, Remote Help가 GCC High로 확대 |

## 정리하며

이번 달 발표를 한 줄로 줄이면 **"에이전트를 사람과 같은 통제선 안에 넣는다"**입니다.
데이터 쪽에서는 OBO 에이전트의 업로드를 사람과 똑같이 검사하고, 운영 쪽에서는 사람과 에이전트가 같은 SOC 기반을 쓰게 합니다.
다만 ISOC는 아직 프리뷰이고 원문도 기능 목록보다 방향 설명이 중심이라, 실제 차이는 써봐야 알 것 같습니다.

---

## 참고

- [What's new in Microsoft Security: September 2026](https://www.microsoft.com/en-us/security/blog/2026/09/24/whats-new-in-microsoft-security-september-2026/) — Alym Rayani, Microsoft Security Blog, 2026-09-24
- [Reimagining the SOC for the agentic era in Microsoft Defender](https://www.microsoft.com/en-us/security/blog/2026/09/23/reimagining-the-soc-for-the-agentic-era-in-microsoft-defender/) — Rob Lefferts, Microsoft Security Blog, 2026-09-23

이 글은 위 두 글의 내용을 정리하고 재구성한 것입니다. 다이어그램은 직접 그렸습니다.
