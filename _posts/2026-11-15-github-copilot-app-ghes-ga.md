---
title: "Copilot 앱, GitHub Enterprise Server 지원 GA — 사내 GHES 팀의 점검표"
date: 2026-11-15 06:53:00 +0900
categories: [IT News]
tags: [copilot, github, devops, governance]
description: GitHub Copilot 앱이 GHES 계정을 정식 지원합니다. 로그인 순서, 앱 1.2.0 조건, 클라우드 세션 제외 같은 제약을 사내 운영자 관점으로 정리했습니다.
---

> 이 글은 GitHub 블로그 changelog의 "GitHub Enterprise Server support in the GitHub Copilot app"(2026-11-13) 발표를 읽고 정리·재구성한 것입니다. 직접 설치해 확인한 내용이 아니며, 원문에 없는 부분은 제 해석이라고 표시했습니다.
{: .prompt-info }

## 결론부터

GitHub Copilot 앱에 GitHub Enterprise Server(GHES) 계정을 추가해, 사내 서버의 원격 저장소·이슈·PR을 앱에서 다룰 수 있게 됐습니다. 이전에는 opt-in 프리뷰였고 이번에 generally available(GA)이 됐습니다. 다만 **GHES에서는 클라우드 세션을 쓸 수 없고**, 앱 1.2.0 이상과 GitHub.com 또는 GHE.com 계정 로그인이 먼저 필요합니다. 사내망에 GHES를 두고 Copilot 도입을 미뤄 온 팀이라면 검토를 시작할 만한 시점입니다. 다만 "GHES에서도 클라우드 쪽 기능이 전부 된다"는 뜻은 아닙니다.

## 원문이 말하는 것

원문은 짧고, 확인된 사실은 다음이 전부입니다.

- GHES 계정을 추가하면 원격 저장소, 이슈, PR 작업을 할 수 있다.
- 상태는 프리뷰에서 GA로 바뀌었다.
- 먼저 GitHub.com 또는 GHE.com 계정으로 로그인해야 한다.
- 추가 경로는 Settings > Accounts > Add account > GitHub Enterprise Server.
- Copilot 앱 1.2.0 이상에서 쓸 수 있다.
- GHES에서는 클라우드 세션을 쓸 수 없다.
- 자세한 절차는 GHES 문서의 "sign in on Copilot app" 페이지에 있다.

요금제나 라이선스 조건은 원문에 없습니다. 이 글에서도 추측하지 않겠습니다.

## 구조를 그림으로 보면

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="Copilot 앱에 먼저 GitHub.com 또는 GHE.com 계정으로 로그인한 뒤 GHES 계정을 추가하는 순서와, GHES에서는 클라우드 세션이 빠진다는 점">
  <defs>
    <marker id="gh-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="gh-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 계정을 붙이는 순서</text>
  <rect x="8" y="50" width="190" height="56" rx="8" class="sk-box"/>
  <text x="103" y="74" text-anchor="middle" class="sk-label">1. 먼저 로그인</text>
  <text x="103" y="92" text-anchor="middle" class="sk-sub">GitHub.com 또는 GHE.com</text>
  <rect x="265" y="50" width="190" height="56" rx="8" class="sk-box"/>
  <text x="360" y="74" text-anchor="middle" class="sk-label">2. Add account</text>
  <text x="360" y="92" text-anchor="middle" class="sk-sub">Settings &gt; Accounts</text>
  <rect x="522" y="50" width="190" height="56" rx="8" class="sk-box-accent"/>
  <text x="617" y="74" text-anchor="middle" class="sk-label">3. GHES 선택</text>
  <text x="617" y="92" text-anchor="middle" class="sk-sub">저장소·이슈·PR</text>
  <path d="M200,78 L262,78" class="sk-line" marker-end="url(#gh-m1)"/>
  <path d="M457,78 L519,78" class="sk-line-accent" marker-end="url(#gh-a1)"/>
  <rect x="522" y="140" width="190" height="52" rx="8" class="sk-box-muted"/>
  <text x="617" y="164" text-anchor="middle" class="sk-label">클라우드 세션</text>
  <text x="617" y="182" text-anchor="middle" class="sk-sub">GHES에서는 사용 불가</text>
  <path d="M617,108 L617,138" class="sk-line" marker-end="url(#gh-m1)"/>
</svg>
<figcaption>GHES는 단독으로 붙지 않고 다른 계정 로그인 위에 얹히며, 클라우드 세션은 빠진다.</figcaption>
</figure>

## 제 해석: 눈여겨볼 두 가지

### 1. "GHES만 쓰는 조직"에게는 문턱이 하나 남는다

원문대로라면 GHES 계정은 GitHub.com 또는 GHE.com 계정으로 로그인한 뒤에 추가합니다. 즉 앱 자체에 로그인하는 정체성은 클라우드 쪽 계정이고, GHES는 그 위에 붙는 계정입니다. 완전 폐쇄망이라 GitHub.com 계정 로그인 자체가 막힌 환경에서 이 흐름이 어떻게 되는지는 원문에 없습니다. 이 부분은 GHES 문서 페이지에서 확인해야 합니다. 도입 전에 가장 먼저 풀어야 할 질문이라고 봅니다.

### 2. 클라우드 세션이 빠진다는 것의 의미

이 글에서는 클라우드 세션을 "Copilot이 원격 환경에서 돌며 작업하는 방식"으로 이해하고 있지만, 원문은 정의를 설명하지 않습니다. 확실한 것은 GHES에서 그 방식이 안 된다는 점뿐입니다. 그렇다면 GHES 사용자는 앱에서 저장소·이슈·PR을 다루는 쪽에 한정된다고 읽는 것이 안전합니다. 같은 앱을 쓰는 GitHub.com 사용자와 GHES 사용자가 서로 다른 기능 범위를 갖게 되므로, 사내 교육 자료에서 이 차이를 분리해 두지 않으면 "내 앱에는 왜 그 버튼이 없나요" 질문이 반복될 가능성이 큽니다.

## 실무에서 달라지는 점

### 1. GHES 관리자: 도입 전 문서부터 읽기

원문이 링크한 GHES 문서의 "sign in on Copilot app" 페이지가 실제 절차의 기준입니다. 서버 버전 요건이나 관리자 측 설정이 있는지는 원문 changelog만으로는 알 수 없으므로, 파일럿 전에 이 문서에서 확인해야 합니다. 저는 이 문서의 세부 내용까지는 읽지 못했습니다.

### 2. 클라이언트 배포 담당: 앱 버전을 1.2.0 이상으로

GA된 기능이 1.2.0 이상에서만 동작하므로, 개발자 PC의 앱 버전을 먼저 점검해야 합니다. 앱 업데이트를 사용자에게 맡기는 조직이라면 "메뉴에 GHES가 안 보인다"의 첫 번째 원인이 버전일 가능성이 높습니다. 배포 방식(자동 업데이트 여부)은 조직마다 달라서, 파일럿 대상자의 버전 확인을 체크리스트 첫 줄에 두길 권합니다.

### 3. 라이선스 담당: 요금제 조건은 별도로 확인

원문에는 Copilot 요금제나 좌석 조건이 없습니다. GHES 사용자에게 Copilot 라이선스를 어떻게 연결하는지, 어느 계정의 권한이 적용되는지는 지금 정보로는 단정할 수 없습니다. 비용 승인 전에 GitHub 영업 담당 또는 GHES 문서로 확인하세요. 최근 [Copilot 모델 6종 폐기 소식](/posts/copilot-six-models-deprecated-dec-14/)처럼 Copilot 쪽 변경이 잦아, 계약·정책 확인은 문서화해 두는 편이 낫습니다.

### 4. 보안·거버넌스 담당: 계정이 둘이 된다는 점

한 앱에 GitHub.com(또는 GHE.com) 계정과 GHES 계정이 함께 붙습니다. 사내 코드와 외부 클라우드 계정이 한 클라이언트에 공존하므로, 개인 계정과 업무 계정을 섞어 쓰지 않도록 사용 지침을 정리해 두는 것이 좋습니다. 이는 원문에 있는 경고가 아니라 제 운영 관점의 제안입니다.

### 5. IT 강사: 교육에서 강조할 것

실습 순서는 "로그인 → Add account → GHES 선택" 세 단계로 짧습니다. 그래서 오히려 순서를 건너뛰는 실수가 생기기 쉽습니다. 먼저 클라우드 계정으로 로그인해야 한다는 점, 그리고 클라우드 세션이 안 된다는 점을 슬라이드에 눈에 띄게 적어 두세요. 저는 직접 실행해 보지 않았으므로, 강의 전에는 반드시 본인 환경에서 한 번 따라 해 보시길 권합니다.

## 정리

- GHES 지원은 프리뷰를 끝내고 GA가 됐다.
- 앱 1.2.0 이상, GitHub.com 또는 GHE.com 로그인이 먼저 필요하다.
- GHES에서는 클라우드 세션을 쓸 수 없다.
- 요금제·폐쇄망 로그인·서버 요건은 원문에 없으니 GHES 문서에서 확인한다.

## 참고

- [GitHub Enterprise Server support in the GitHub Copilot app (GitHub Changelog, 2026-11-13)](https://github.blog/changelog/2026-11-13-github-enterprise-server-support-in-the-github-copilot-app)
- [Copilot GHES documentation — sign in on Copilot app](https://docs.github.com/enterprise-server@latest/copilot/copilot-on-ghes/sign-in-on-copilot-app)
- 이어 읽기: [Copilot 모델 6종 12/14 폐기 — 모델을 고정해 쓰는 팀의 점검표](/posts/copilot-six-models-deprecated-dec-14/)
