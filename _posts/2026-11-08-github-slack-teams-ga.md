---
title: "@GitHub in Slack·Teams GA — 대화창이 코딩 세션의 입구가 된다"
date: 2026-11-08 16:19:00 +0900
categories: [IT News]
tags: [github, copilot, devops, automation]
description: Slack과 Teams에서 @GitHub로 버그 조사부터 PR 생성까지 하는 기능이 GA됐다. 달라진 점과 도입 전에 확인할 운영 항목을 정리했다.
---

> 이 글은 GitHub Changelog의 "@GitHub in Slack and Teams is generally available and more"(2026-11-06) 발표를 읽고 정리·재구성한 것입니다. 원문에 없는 부분은 제 해석이며 그렇게 표시했습니다.
{: .prompt-info }

## 결론부터

Slack과 Microsoft Teams 대화창에서 `@GitHub`를 불러 버그를 조사하고, 이슈를 만들고, 코드 변경을 구현해 풀 리퀘스트(PR)까지 여는 기능이 GA가 됐습니다. 이번 발표에서 눈에 띄는 변화는 세 가지입니다.

- Teams에서 Canvas로 계획·코드 변경·미리보기를 대화 옆에서 함께 검토
- 저장소를 메시지에서 자동으로 식별하는 간소화된 시작 과정
- `@GitHub`가 만든 이슈·PR 본문에 "Created from a Slack/Teams conversation with @username" 문구 추가

기능 자체보다 저는 세 번째 항목이 운영 관점에서 가장 의미 있다고 봅니다. 아래에서 이유를 풀겠습니다.

## 원문이 말하는 것

원문은 짧습니다. 핵심만 추리면 다음과 같습니다.

### 대화에서 코드까지

`@GitHub`는 대화 안에서 버그 조사, 이슈 생성, 변경 구현, PR 열기를 수행합니다. 개발자가 IDE나 웹으로 옮겨 가지 않아도 팀 논의가 곧 작업 요청이 되는 구조입니다.

### Teams의 Canvas

Teams에서는 Canvas로 구현 계획, 코드 변경, 미리보기를 대화와 나란히 볼 수 있습니다. 팀원이 같은 화면에서 의견을 남기고 구현을 다듬을 수 있다는 설명입니다.

### 시작과 안정성 개선

메시지 내용으로 저장소를 식별해 세션을 바로 시작하고, 로그인하지 않았거나 기본 저장소가 없을 때, 저장소를 바꾸고 싶을 때, 앱 권한이 필요할 때의 시작 과정도 쉬워졌다고 합니다. 후속 메시지에 응답하지 않던 문제도 고쳐졌습니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="Slack 또는 Teams 대화에서 @GitHub 호출, 저장소 식별, 코딩 세션, 이슈와 PR 생성, 출처 문구 기록으로 이어지는 흐름">
  <defs>
    <marker id="gst-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 대화에서 PR까지의 흐름</text>
  <rect x="8" y="70" width="130" height="60" rx="8" class="sk-box"/>
  <text x="73" y="96" text-anchor="middle" class="sk-label">Slack / Teams</text>
  <text x="73" y="114" text-anchor="middle" class="sk-sub">팀 대화</text>
  <rect x="170" y="70" width="130" height="60" rx="8" class="sk-box"/>
  <text x="235" y="96" text-anchor="middle" class="sk-label">@GitHub 호출</text>
  <text x="235" y="114" text-anchor="middle" class="sk-sub">저장소 자동 식별</text>
  <rect x="332" y="70" width="130" height="60" rx="8" class="sk-box"/>
  <text x="397" y="96" text-anchor="middle" class="sk-label">코딩 세션</text>
  <text x="397" y="114" text-anchor="middle" class="sk-sub">조사·구현</text>
  <rect x="494" y="70" width="140" height="60" rx="8" class="sk-box-accent"/>
  <text x="564" y="96" text-anchor="middle" class="sk-label">이슈 · PR</text>
  <text x="564" y="114" text-anchor="middle" class="sk-sub">출처 문구가 남는다</text>
  <path d="M140,100 L168,100" class="sk-line" marker-end="url(#gst-m1)"/>
  <path d="M302,100 L330,100" class="sk-line" marker-end="url(#gst-m1)"/>
  <path d="M464,100 L492,100" class="sk-line-accent" marker-end="url(#gst-m1)"/>
  <rect x="332" y="160" width="302" height="40" rx="8" class="sk-box-muted"/>
  <text x="483" y="185" text-anchor="middle" class="sk-sub">Teams는 Canvas로 계획·변경·미리보기를 대화 옆에서 검토</text>
</svg>
<figcaption>이번 GA의 핵심은 "대화가 작업 요청이 되고, 그 흔적이 PR에 남는다"는 것이다.</figcaption>
</figure>

## 제 해석: 입구가 바뀌면 통제 지점도 바뀐다

지금까지 Copilot 코딩 에이전트류 기능의 입구는 이슈, IDE, 웹 화면이었습니다. 이제 메신저가 입구가 됩니다. 메신저는 코드 권한이 없는 사람도 많이 참여하는 공간이라, 누가 어떤 맥락에서 작업을 요청했는지가 흐려지기 쉽습니다.

그래서 "Created from a Slack/Teams conversation with @username" 문구가 중요합니다. 이 문구는 감사 추적의 최소 단위가 됩니다. 나중에 PR을 보고 "이 변경은 누가 어느 대화에서 시작했는가"를 거슬러 올라갈 수 있기 때문입니다. 다만 이 문구가 권한 통제를 대신하지는 않습니다. 기록은 사후 수단이고, 사전 통제는 별도로 설계해야 합니다. (이 부분은 원문이 아니라 제 판단입니다.)

저장소 자동 식별도 양면이 있습니다. 시작은 편해지지만, 메시지에 저장소 이름이 비슷하게 언급되면 의도하지 않은 저장소에서 세션이 열릴 가능성도 생각해 볼 만합니다. 원문은 잘못 식별된 경우의 동작을 설명하지 않으므로, 도입 전에 직접 확인해야 할 항목입니다.

비슷한 맥락으로 최근 Copilot 코드 리뷰 지표를 다룬 [Copilot 코드 리뷰 해결률](/posts/copilot-code-review-usage-metrics-addressed-rate/) 글과 승인 흐름을 다룬 [Copilot 보조 권한 GA](/posts/copilot-assisted-permissions-ga/) 글을 함께 보시면, "에이전트가 만든 변경을 어떻게 측정하고 승인할 것인가"라는 같은 질문의 다른 조각을 볼 수 있습니다.

## 실무에서 달라지는 점

### GitHub 조직 관리자: 앱 설치 범위부터 정한다

Slack·Teams에서 `@GitHub`를 쓰려면 워크스페이스에 앱을 연결하고 GitHub 쪽 권한을 부여해야 합니다(원문은 "앱 접근이 필요한 경우"의 시작 과정이 쉬워졌다고만 언급). 어느 저장소까지 허용할지, 어느 채널에서 쓰게 할지 먼저 정하고 파일럿 팀 하나로 시작하는 편이 안전합니다.

### M365 운영자: Teams 쪽 앱 정책을 확인한다

Teams에서는 Teams 관리 센터의 앱 허용·차단 정책에 따라 사용자가 앱을 쓸 수 있는지가 갈립니다. Canvas 사용 조건과 필요한 라이선스·플랜은 원문에 적혀 있지 않습니다. 가정하지 말고 공식 문서에서 확인하십시오.

### 보안·감사 담당: 출처 문구를 검색 가능한 신호로 쓴다

PR 본문에 들어가는 고정 문구는 검색·리포팅 기준이 될 수 있습니다. 메신저 발(發) PR이 얼마나 되는지, 리뷰 없이 병합되는 비율은 어떤지 파악하는 데 활용할 만합니다. 브랜치 보호 규칙과 필수 리뷰는 그대로 두어야 합니다.

### 팀 리더: 대화 운영 규칙을 만든다

공개 채널에서 요청하면 대화 내용이 곧 작업 지시서가 되어 토큰·내부 URL 같은 민감 정보가 섞일 수 있습니다. "비밀 정보는 대화에 붙이지 않는다", "요청은 스레드 하나에 모은다" 같은 최소 규칙을 먼저 공유하십시오.

### IT 강사: 교육에서 강조할 점

"말로 시키면 PR이 열린다"는 편리함보다 "열린 PR은 사람이 읽고 책임진다"를 먼저 가르쳐야 합니다. 에이전트가 만든 PR도 일반 PR과 같은 리뷰 기준을 적용한다는 점을 실습 전에 못 박는 것을 권합니다.

## 정리

기능 목록만 보면 "메신저 연동 하나 더"로 보이지만, 실제 변화는 작업의 출발점이 대화로 이동했다는 데 있습니다. 편해진 만큼 앱 설치 범위, 저장소 식별, 대화 속 민감 정보, 리뷰 기준을 먼저 정리해 두면 도입 후의 혼란이 줄어듭니다.

## 참고

- [@GitHub in Slack and Teams is generally available and more — GitHub Changelog (2026-11-06)](https://github.blog/changelog/2026-11-06-github-copilot-in-slack-and-teams-is-generally-available)
- [Copilot 코드 리뷰 해결률 — 사용량 지표 API에 생긴 두 가지](/posts/copilot-code-review-usage-metrics-addressed-rate/)
- [Copilot 보조 권한(Assisted permissions) GA](/posts/copilot-assisted-permissions-ga/)
