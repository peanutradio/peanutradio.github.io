---
title: "Microsoft Copilot 개편 — Home, Code, Autopilot 세 갈래로"
date: 2026-09-27 09:40:00 +0900
categories: [IT News]
tags: [copilot, microsoft-365, autopilot, ai-agent, managed-runtime]
description: 9월 25일 Microsoft가 새 Copilot을 발표했습니다. Chat과 Cowork를 합친 Home, 자연어로 앱을 만드는 Code, 알아서 계속 일하는 Autopilot, 그리고 이들을 떠받치는 Managed Runtime을 정리합니다.
---

![Microsoft 레드먼드 캠퍼스 92번 건물](/assets/img/posts/copilot-home-code-autopilot.jpg)
_사진: [Jiaqian AirplaneFan](https://commons.wikimedia.org/wiki/File:Building_92_of_Microsoft_Redmond_Campus_-_panoramio.jpg), [CC BY 3.0](https://creativecommons.org/licenses/by/3.0) — 2015년 촬영 자료 사진, 이번 발표와 무관_

9월 25일 Microsoft가 **새 Copilot**을 발표했습니다. 결론부터 말하면, Copilot 앱이 **Home · Code · Autopilot** 세 갈래로 다시 짜였고, 그 밑에 만든 것을 테넌트 안에서 돌려주는 **Managed Runtime**이 깔렸습니다. "대화하는 도구"에서 "일을 맡기고, 만들고, 돌리는 곳"으로 성격이 바뀌었다고 보는 게 맞습니다.

> 이 글은 Microsoft 공식 블로그의 발표문(Jared Spataro, 2026-09-25)과 연결된 Managed Runtime 발표문을 읽고 정리·재구성한 것입니다.
{: .prompt-info }

## 세 가지 기능

**Home — Chat과 Cowork가 한곳에.** Home은 Copilot의 새 시작 화면입니다. 짧은 질문·초안은 Chat이, 끝까지 맡기는 작업(RFP 답변, 고객 브리핑 등)은 Cowork가 처리합니다. 여기에 **Office in Copilot**이 붙어 Word·Excel·PowerPoint 기능이 Copilot 안으로 들어왔습니다. 예산 모델이나 발표 자료를 요청하면 실제로 편집 가능한 파일이 만들어지고, 동료를 @멘션하면 Copilot과 Office 양쪽에서 변경이 동기화됩니다.

**Code — 자연어로 앱을 만든다.** 원문은 솔루션 만들기를 "메모를 쓰거나 예산을 모델링하는 것만큼 기본적인 기술"로 만들겠다고 합니다. 트래커, 대시보드, 자동화, 팀과 공유하는 사내 앱까지 말로 설명하면 Copilot이 만듭니다. **GitHub Copilot과 같은 기반 기술**을 쓰고, 샌드박스에서 돌며, 테넌트 안에 안전하게 호스팅할 수 있습니다. 전문 개발자는 계속 GitHub Copilot을 쓰되 Copilot 플랫폼과의 연결이 강화됩니다.

**Autopilot — 퇴근해도 일하는 동료.** 이전 이름은 **Scout**입니다. 이름·역할·목표를 주면 채널을 지켜보고, 스레드를 후속 처리하고, 반복 업무를 돌리고, 며칠 뒤 프로젝트를 다시 이어받습니다. 프롬프트를 기다리지 않는다는 점이 핵심입니다. 원문 예시는 공급업체 리뷰 전체 운영 — 일정과 역산 계획을 세우고 회의 준비, 후속 조치, 이해관계자 연락까지 맡는 것입니다. 클라우드에서 돌고, 테넌트 안에 **자체 ID·메모리·컴퓨터·작업 공간**을 가지며, Teams·Outlook에서 동료처럼 @멘션합니다.

<figure class="sketch">
<svg viewBox="0 0 720 300" role="img" aria-label="새 Copilot 구조 — Home, Code, Autopilot 세 기능이 Microsoft IQ 위에 있고, 만든 앱은 Copilot Studio 앱과 함께 Managed Runtime에서 실행된다">
  <defs>
    <marker id="cp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 새 Copilot — 세 기능, 하나의 기반</text>

  <rect x="8" y="36" width="704" height="30" rx="8" class="sk-box"/>
  <text x="360" y="56" text-anchor="middle" class="sk-label">Copilot 앱</text>

  <rect x="8" y="80" width="220" height="70" rx="8" class="sk-box"/>
  <text x="118" y="106" text-anchor="middle" class="sk-label">Home</text>
  <text x="118" y="124" text-anchor="middle" class="sk-sub">Chat + Cowork + Office in Copilot</text>
  <text x="118" y="140" text-anchor="middle" class="sk-sub">묻고 · 맡긴다</text>

  <rect x="250" y="80" width="220" height="70" rx="8" class="sk-box-accent"/>
  <text x="360" y="106" text-anchor="middle" class="sk-label">Code</text>
  <text x="360" y="124" text-anchor="middle" class="sk-sub">자연어로 앱·대시보드·워크플로</text>
  <text x="360" y="140" text-anchor="middle" class="sk-sub">GitHub Copilot과 같은 기반 기술</text>

  <rect x="492" y="80" width="220" height="70" rx="8" class="sk-box"/>
  <text x="602" y="106" text-anchor="middle" class="sk-label">Autopilot</text>
  <text x="602" y="124" text-anchor="middle" class="sk-sub">구 Scout · 계속 일하는 에이전트</text>
  <text x="602" y="140" text-anchor="middle" class="sk-sub">자체 ID · 메모리 · 작업 공간</text>

  <rect x="8" y="164" width="704" height="30" rx="8" class="sk-box"/>
  <text x="360" y="184" text-anchor="middle" class="sk-label">Microsoft IQ — 조직의 업무 맥락으로 그라운딩</text>

  <path d="M118,150 L118,160" class="sk-line"/>
  <path d="M360,194 L360,226" class="sk-line-accent" marker-end="url(#cp-m1)"/>
  <path d="M118,194 C118,212 250,212 300,228" class="sk-line" marker-end="url(#cp-m1)"/>
  <text x="140" y="222" class="sk-sub">Cowork 앱</text>
  <rect x="560" y="206" width="152" height="40" rx="8" class="sk-box-muted"/>
  <text x="636" y="231" text-anchor="middle" class="sk-sub">Copilot Studio 앱</text>
  <path d="M560,226 C500,226 460,232 420,236" class="sk-line" marker-end="url(#cp-m1)"/>

  <rect x="180" y="234" width="360" height="56" rx="10" class="sk-box-accent"/>
  <text x="360" y="258" text-anchor="middle" class="sk-label">Copilot Managed Runtime</text>
  <text x="360" y="276" text-anchor="middle" class="sk-sub">테넌트 안 호스팅 · IT가 통제 · 프리뷰</text>
</svg>
<figcaption>누가 어디서 만들든 결국 같은 런타임에서 돌고, IT가 한곳에서 통제한다 — 이번 개편의 뼈대는 이것입니다.</figcaption>
</figure>

## 진짜 핵심은 Managed Runtime

제가 가장 크게 본 건 **Microsoft Copilot Managed Runtime**입니다. 회사의 Microsoft 365 환경 안에서 코드를 안전하게 실행하는 호스팅 인프라로, IT가 통제하고 사용자는 앱 공유·실데이터 연결만 신경 쓰면 됩니다. 별도 발표문에 따르면 **Cowork, Code, Copilot Studio**에서 만든 앱을 모두 이 위에서 돌리고, Entra ID로 접근을 인증하며, 조직 정책으로 커넥터·데이터·엔드포인트를 통제합니다. 앱 목록은 M365 관리 센터에 모이고, SDK로 서드파티·프로 개발 도구도 붙을 수 있습니다.

Copilot Studio로 에이전트를 만들어 온 입장에서 이게 반갑습니다. 지금까지 "누가 만든 앱이 어디서 도는가"는 거버넌스 논의의 가장 골치 아픈 부분이었는데, 실행 위치를 하나로 모으는 방향이 분명해졌습니다.

## 과금 — 정액과 사용량의 분리

과금 구조도 바뀝니다. 일상적인 AI(Chat, Office 앱)는 **사용자 구독 라이선스(USL)** 정액으로, Cowork·Code·Autopilot 같은 에이전트 작업은 **사용량 기반 과금(UBB)**으로 나뉩니다. 요청마다 정확도·속도·비용을 따져 모델을 고르는 **Auto** 라우팅도 들어갑니다. 관리자용 FinOps 기능도 함께 나와, 그룹별로 쓸 수 있는 모델 계열을 제한하고 Agent 365·Code·Managed Runtime의 지출 정책을 관리할 수 있습니다.

<figure class="sketch">
<svg viewBox="0 0 720 170" role="img" aria-label="과금 구조 — 일상 AI는 사용자 구독 라이선스 정액, 에이전트 작업은 사용량 기반 과금">
  <defs>
    <marker id="cp-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 과금은 두 갈래</text>
  <rect x="8" y="40" width="330" height="80" rx="10" class="sk-box"/>
  <text x="173" y="66" text-anchor="middle" class="sk-label">USL — 정액</text>
  <text x="173" y="86" text-anchor="middle" class="sk-sub">Chat · Office 앱의 일상 AI</text>
  <text x="173" y="104" text-anchor="middle" class="sk-mono">Auto 모델 라우팅</text>
  <rect x="382" y="40" width="330" height="80" rx="10" class="sk-box-accent"/>
  <text x="547" y="66" text-anchor="middle" class="sk-label">UBB — 사용량 기반</text>
  <text x="547" y="86" text-anchor="middle" class="sk-sub">Cowork · Code · Autopilot 에이전트 작업</text>
  <text x="547" y="104" text-anchor="middle" class="sk-sub">FinOps로 지출 정책 관리</text>
  <path d="M338,80 L378,80" class="sk-line" marker-end="url(#cp-m2)"/>
  <text x="360" y="150" text-anchor="middle" class="sk-sub">쓰는 만큼 늘어나는 쪽은 에이전트다 — 비용 통제 도구가 같이 나온 이유</text>
</svg>
<figcaption>에이전트가 오래 일할수록 비용이 늘어나는 구조라, 이제 관리자는 모델 선택과 지출 정책까지 챙겨야 합니다.</figcaption>
</figure>

## 언제 쓸 수 있나

| 항목 | 일정(원문 기준) |
|---|---|
| Home, Code | 수 주 안에 Frontier 프로그램부터 롤아웃 |
| Autopilot | 이달 말 프라이빗 프리뷰로 확대 |
| Code | 올해 안에 Microsoft 365 Premium·Pro 구독자 대상 프리뷰 |
| Today(능동형 커맨드 센터) | 10월 프라이빗 프리뷰 |
| Teams의 @Copilot | 이달 말 프라이빗 프리뷰 |
| Managed Runtime | 프리뷰 중, Code 안에서 사용 가능 예정 |

당장 일반 테넌트에서 다 쓸 수 있는 건 아닙니다. 대부분 Frontier·프리뷰 단계라, 저는 우선 Managed Runtime 위에서 Copilot Studio 앱이 어떻게 관리되는지부터 확인해 볼 생각입니다. 11월 17~20일 Microsoft Ignite에서 더 구체적인 내용이 나올 것으로 보입니다.

## 참고

- [Introducing the new Copilot with Home, Code and Autopilot — Official Microsoft Blog (2026-09-25)](https://blogs.microsoft.com/blog/2026/09/25/introducing-the-new-copilot-with-home-code-and-autopilot/)
- [Build where you want, run with confidence: Now Microsoft hosts and manages the code created by Copilot — Microsoft Copilot Blog (2026-09-25)](https://www.microsoft.com/en-us/copilot/blog/copilot-studio/build-where-you-want-run-with-confidence-now-microsoft-hosts-and-manages-the-code-created-by-copilot/)
- [Microsoft 365 Copilot Frontier program](https://www.microsoft.com/en-us/microsoft-365-copilot/frontier-program)
