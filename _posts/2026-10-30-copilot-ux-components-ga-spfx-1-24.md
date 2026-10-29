---
title: "Copilot UX components GA — 에이전트가 텍스트 대신 화면으로 답한다"
date: 2026-10-30 06:53:00 +0900
categories: [IT News]
tags: [copilot, microsoft-365, ai-agent]
description: SPFx 1.24와 함께 Copilot UX components가 GA됐다. 에이전트가 Copilot 안에 대시보드·폼을 띄우는 방식과 라이선스, 지원 범위, 도입 전 점검 사항을 정리했다.
---

> 이 글은 Microsoft 365 Developer Blog의 "Copilot UX components are generally available" 발표(2026-10-28)를 읽고 정리·재구성한 것이다. 수치와 요건은 원문 기준이며, 직접 실행해 확인한 내용은 아니다.
{: .prompt-info }

## 결론부터

Copilot UX components가 SharePoint Framework(SPFx) 1.24와 함께 전 세계 GA가 됐다. 에이전트가 답을 글로만 주지 않고, 필요할 때 Copilot 캔버스 안에 대시보드·폼·승인 화면 같은 대화형 웹 화면을 직접 띄울 수 있다는 뜻이다.

내가 보기에 핵심은 기능 자체보다 **배포 모델**이다. JS·CSS를 테넌트 안에서 호스팅하고, 로그인한 사용자 권한으로 실행하며, 별도 웹 서버나 인증 구현이 필요 없다. 다만 사용자별 Copilot 애드온 라이선스가 있어야 렌더링되고, GA 시점의 지원 클라이언트도 제한적이다. 도입 여부는 기능보다 이 두 가지로 갈릴 가능성이 크다.

## 무엇이 GA됐나

원문 설명을 정리하면 다음과 같다.

### 컴포넌트의 정체

에이전트가 필요한 순간 Copilot 캔버스에 띄우는 대화형 웹 화면이다. 대시보드, 폼, 승인, 실시간 업무 데이터 같은 것이 예로 나온다. 사용자는 포털이나 다른 앱으로 이동하지 않고 대화 안에서 작업을 끝낸다. React를 포함해 어떤 JS 라이브러리든 쓸 수 있고, MCP Apps 모델 위에서 동작한다. 이전 자료에서는 SharePoint Copilot Apps, Copilot Components라는 이름으로 나왔다고 원문이 밝힌다.

### 두 가지 표시 모드

인라인은 대화 흐름 안에서 가볍게 처리하는 용도다. 전체 화면은 대시보드, 여러 단계의 폼, 지도, 상세 보기처럼 큰 화면이 필요한 경우에 쓴다. 컴포넌트마다 어느 모드를 지원할지, 모드별로 무엇을 그릴지 만드는 쪽이 정한다. Copilot의 라이트·다크 테마에도 맞출 수 있다.

### 요건과 지원 범위

- SPFx 1.24가 필요하다. 프리뷰 빌드를 쓰던 팀은 GA 버전으로 올려야 한다.
- Node.js는 24와 26을 지원한다. 최소 버전은 원문에 없다.
- 신규 SPFx 솔루션의 React 기본값은 18.x다. 기존 솔루션은 React 18 업그레이드 가이드를 따라야 한다.
- GA 시점 지원 클라이언트는 브라우저·데스크톱의 Microsoft Copilot과 Teams 경유 Copilot이다. 모바일은 아직 지원하지 않는다.
- 예정 항목은 Android·iOS 모바일, 새 Copilot 플러그인 모델 패키징, Cowork 등 더 많은 Copilot 화면이다. SPFx 1.25는 2026년 12월 또는 2027년 1월로 예고됐다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="텍스트 응답과 UX 컴포넌트 응답의 흐름 비교">
  <defs>
    <marker id="cux-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="cux-a1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 에이전트 응답이 '글'에서 '화면'으로</text>
  <text x="8" y="52" class="sk-sub">기존: 텍스트 응답</text>
  <rect x="8" y="64" width="150" height="52" rx="8" class="sk-box"/>
  <text x="83" y="86" text-anchor="middle" class="sk-label">사용자 요청</text>
  <text x="83" y="104" text-anchor="middle" class="sk-sub">"이번 달 승인 건 보여줘"</text>
  <rect x="270" y="64" width="150" height="52" rx="8" class="sk-box"/>
  <text x="345" y="86" text-anchor="middle" class="sk-label">에이전트</text>
  <text x="345" y="104" text-anchor="middle" class="sk-sub">표를 글로 설명</text>
  <rect x="532" y="64" width="170" height="52" rx="8" class="sk-box-muted"/>
  <text x="617" y="86" text-anchor="middle" class="sk-label">다른 포털로 이동</text>
  <text x="617" y="104" text-anchor="middle" class="sk-sub">승인은 거기서</text>
  <path d="M160,90 L268,90" class="sk-line" marker-end="url(#cux-m1)"/>
  <path d="M422,90 L530,90" class="sk-line" marker-end="url(#cux-m1)"/>
  <text x="8" y="160" class="sk-sub">GA: UX component</text>
  <rect x="8" y="172" width="150" height="52" rx="8" class="sk-box"/>
  <text x="83" y="194" text-anchor="middle" class="sk-label">사용자 요청</text>
  <text x="83" y="212" text-anchor="middle" class="sk-sub">같은 질문</text>
  <rect x="270" y="172" width="150" height="52" rx="8" class="sk-box"/>
  <text x="345" y="194" text-anchor="middle" class="sk-label">에이전트</text>
  <text x="345" y="212" text-anchor="middle" class="sk-sub">컴포넌트 호출</text>
  <rect x="532" y="172" width="170" height="52" rx="8" class="sk-box-accent"/>
  <text x="617" y="194" text-anchor="middle" class="sk-label">Copilot 캔버스</text>
  <text x="617" y="212" text-anchor="middle" class="sk-sub">대시보드·폼에서 바로 승인</text>
  <path d="M160,198 L268,198" class="sk-line" marker-end="url(#cux-m1)"/>
  <path d="M422,198 L530,198" class="sk-line-accent" marker-end="url(#cux-a1)"/>
</svg>
<figcaption>달라지는 것은 답의 형식이다. 사용자가 대화를 떠나지 않고 같은 자리에서 확인하고 승인한다.</figcaption>
</figure>

## 개발과 배포 모델

원문이 제시한 절차는 네 단계다.

1. SPFx 1.24를 설치한다: `npm install @microsoft/generator-sharepoint@latest --global`
2. Copilot UX component를 스캐폴딩하고 템플릿(Minimal, No framework, React)을 고른다.
3. Copilot Workbench에서 로컬 테스트한다. 인라인·전체 화면, 라이트·다크 테마, 입력 속성을 본다.
4. 패키지를 테넌트에 배포하고 Copilot에 노출한다.

선언형 에이전트와 컴포넌트가 하나의 SPFx 솔루션으로 함께 배포되고, 선언형 에이전트 매니페스트(신규는 v1.8)는 빌드 때 검증된다. JS·CSS는 배포 후 고객 M365 테넌트 안에서 호스팅되며, 등록과 라우팅이 자동이라 노출할 엔드포인트가 없다. 실행은 SSO로 로그인한 사용자 신원으로 이뤄지고 접근은 기존 권한을 따른다. 같은 SPFx 코드를 SharePoint 웹파트, Teams 탭, Outlook 앱으로도 쓸 수 있다고 한다.

비용 측면에서 컴포넌트 자체에는 소비 기반 과금이 없다. 다만 컴포넌트가 Work IQ 같은 소비 기반 API를 호출하면 그쪽 요금이 붙을 수 있다. 가격 수치는 원문에 없다.

## 해석: 이 발표를 어떻게 읽을까

### '에이전트 UI'의 병목이 인프라에서 라이선스로 옮겨간다

에이전트가 화면을 그리려면 보통 웹 서버, 인증, 엔드포인트 노출, CORS 같은 일을 먼저 풀어야 한다. 이 모델은 그 층을 플랫폼이 가져간다. 그래서 만드는 쪽 장벽은 낮아진다. 반대로 도입 장벽은 사용자별 애드온 라이선스로 옮겨간다. 라이선스가 없는 사용자에게는 컴포넌트가 렌더링되지 않고 안내 메시지만 보인다. 일부 사용자에게만 라이선스가 있는 조직이라면, 같은 에이전트가 사람에 따라 다르게 보일 수 있다는 뜻이다.

### 에이전트 하네스·ID 이야기와 이어진다

에이전트가 일을 하려면 모델 바깥의 실행 환경이 필요하다는 이야기는 [Copilot 하네스가 Foundry Agent Service에 들어온 글](/posts/copilot-harness-foundry-agent-service-preview/)에서 다뤘다. UX component는 그 바깥 환경의 '출력 쪽'에 해당한다. 사용자 권한으로 실행된다는 점은 [Agent 365와 Agent ID 정리](/posts/agent-365-agent-id/)에서 본 "에이전트에게도 신원이 필요하다"는 흐름과 같은 맥락이다. 다만 컴포넌트는 로그인한 사람의 권한으로 도는 것이라, 에이전트 자체의 신원 문제와는 구분해서 봐야 한다.

### 승인 UI는 편하지만 방패는 아니다

원문은 중요한 변경은 사람이 검토하고 확인한 뒤 업무 시스템으로 보내라고 권한다. 승인 화면을 에이전트가 띄워 주는 것은 편리하지만, 화면이 있다고 안전해지는 것은 아니다. 승인 화면에 무엇을 보여 줄지(변경 전후 값, 대상 건수)를 설계하는 일이 여전히 만드는 사람 몫이다. 이 부분은 원문에 없는 내 의견이다.

## 실무에서 달라지는 점

### 1. 라이선스 담당자는 대상 사용자부터 센다

컴포넌트를 쓸 사용자 전원에게 Microsoft Copilot 애드온 라이선스가 필요하다. 에이전트를 만들기 전에 누가 쓸지, 그 사람들이 라이선스를 이미 갖고 있는지부터 확인하는 것이 순서다. 가격은 원문에 없으니 계약 조건은 별도로 확인해야 한다.

### 2. 개발 환경 담당자는 Node와 React 버전을 점검한다

SPFx 1.24는 Node 24 또는 26을 지원하고, 신규 솔루션의 React 기본값은 18.x다. 기존 SPFx 솔루션을 재사용하려는 팀은 React 18 업그레이드 가이드를 먼저 검토해야 한다. 프리뷰 빌드로 시험하던 팀은 GA 버전으로 교체한다.

### 3. 사용자 범위를 안내할 때 모바일을 빼고 말한다

GA 시점에는 브라우저·데스크톱 Copilot과 Teams만 지원하고 모바일은 예정이다. 현장 직원이 주로 모바일을 쓰는 업무라면 지금은 대상이 아니다. 시연이나 교육에서 "Copilot에서 된다"고만 말하면 오해가 생기므로 지원 화면을 함께 밝히는 편이 낫다.

### 4. 보안 담당자는 '사용자 권한으로 실행'의 의미를 확인한다

접근이 기존 권한을 따른다는 것은 장점이지만, 컴포넌트가 호출하는 업무 API에는 추가 권한이나 동의가 필요할 수 있다. 어떤 API를 부르는지, 동의를 누가 주는지를 배포 전에 목록으로 만들어 두는 것이 좋다. Work IQ 같은 소비 기반 API를 부르면 요금도 따로 생길 수 있다.

### 5. 강사는 '화면이 뜬다'와 '변경이 안전하다'를 나눠 가르친다

교육에서는 인라인·전체 화면 모드, Copilot Workbench 로컬 테스트, 입력 속성 전달까지를 개발 흐름으로 설명하기 좋다. 여기에 확인 단계 없이 업무 시스템을 바꾸는 화면은 만들지 않는다는 원칙을 같이 강조하겠다. 이 글을 쓰는 시점에 나는 직접 실행해 보지 않았으므로, 실습 구성은 공식 튜토리얼 기준으로만 짤 수 있다.

## 참고

- [Copilot UX components are generally available — Microsoft 365 Developer Blog (2026-10-28)](https://devblogs.microsoft.com/microsoft365dev/copilot-ux-components-are-generally-available-bring-interactive-experiences-into-copilot/)
- [Get started](https://aka.ms/SPFx/Copilot) · [Documentation](https://aka.ms/SPFx/Copilot/docs) · [Sample gallery](https://aka.ms/SPFx/Copilot/Samples)
- [Build your first Copilot app — Microsoft Learn](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/copilot/get-started/build-your-first-copilot-app)
- [SPFx 1.24 release notes](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/release-1.24.0)
- [React 18 upgrade guidance](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/react-18-upgrade)
