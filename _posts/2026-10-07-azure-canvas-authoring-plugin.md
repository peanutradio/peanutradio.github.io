---
title: "Canvas authoring 플러그인 — Azure 캔버스를 Copilot이 만들어 준다"
date: 2026-10-07 07:17:00 +0900
categories: [AI Agent]
tags: [copilot, azure, canvas, plugin, ai-agent]
description: Copilot 플러그인 Canvas authoring으로 Microsoft Canvas Toolkit 기반 Azure 캔버스 프로젝트를 생성하는 과정을 정리한다.
---

> 이 글은 Microsoft 개발자 블로그의 "Build Azure Canvases with Canvas authoring"(2026-10-06)을 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록이 아니라 발표 내용의 정리입니다.
{: .prompt-info }

## 결론부터

Azure 캔버스를 처음부터 손으로 짜지 않아도 된다. Copilot에 **Canvas authoring** 플러그인을 설치하면 Copilot이 네이티브 캔버스 워크플로를 따라 Microsoft Canvas Toolkit 기반 프로젝트를 만들어 준다. 글은 Azure 캔버스를 "개발자와 에이전트가 함께 일하는 공유 인터랙티브 작업 공간"으로 설명한다. 사람과 에이전트가 같은 화면을 보며 Azure 리소스를 다루는 접근이라 에이전트 설계 관점에서 읽을 만하다.

## 만드는 순서

<figure class="sketch">
<svg viewBox="0 0 720 210" role="img" aria-label="플러그인 설치부터 Azure 로그인까지 네 단계">
  <defs>
    <marker id="cv-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 설치에서 로그인까지</text>
  <rect x="8" y="50" width="160" height="70" rx="8" class="sk-box"/>
  <text x="88" y="78" text-anchor="middle" class="sk-label">플러그인 설치</text>
  <text x="88" y="98" text-anchor="middle" class="sk-sub">Awesome Copilot</text>
  <rect x="198" y="50" width="160" height="70" rx="8" class="sk-box-accent"/>
  <text x="278" y="78" text-anchor="middle" class="sk-label">자연어로 요청</text>
  <text x="278" y="98" text-anchor="middle" class="sk-sub">프로젝트 생성</text>
  <rect x="388" y="50" width="160" height="70" rx="8" class="sk-box"/>
  <text x="468" y="78" text-anchor="middle" class="sk-label">dist/ 설치</text>
  <text x="468" y="98" text-anchor="middle" class="sk-sub">provider 다시 로드</text>
  <rect x="578" y="50" width="134" height="70" rx="8" class="sk-box"/>
  <text x="645" y="78" text-anchor="middle" class="sk-label">Azure CLI 로그인</text>
  <text x="645" y="98" text-anchor="middle" class="sk-sub">구독 선택</text>
  <path d="M170,85 L196,85" class="sk-line" marker-end="url(#cv-m1)"/>
  <path d="M360,85 L386,85" class="sk-line" marker-end="url(#cv-m1)"/>
  <path d="M550,85 L576,85" class="sk-line" marker-end="url(#cv-m1)"/>
</svg>
<figcaption>사람이 하는 일은 요청과 설치, 로그인이고 코드 뼈대는 플러그인이 만든다.</figcaption>
</figure>

1. Copilot의 **Customize > Plugins**에서 Awesome Copilot 마켓플레이스를 열고 "Canvas authoring"을 설치한다
2. 작업할 폴더에서 새 프로젝트 세션을 시작한다
3. 예를 들어 "리소스 그룹을 나열하는 읽기 전용 Azure 캔버스를 만들어줘. 네이티브 create-canvas 스킬과 create-canvas-app 컴패니언의 Azure 스타터를 써"라고 요청한다
4. 생성된 README를 따라 `dist/` 디렉터리 전체를 설치하고 canvas provider를 다시 로드한다
5. Azure CLI로 로그인하고 캔버스에서 CLI 프로필을 불러온 뒤 구독을 고르고 **List resource groups**를 누른다

글은 이 플러그인이 덜어 주는 것을 인증, 구독 범위 지정, 공유 상태, 호스트 연동으로 정리한다. 캔버스를 만들 때마다 반복되는 배관 작업이고, 실제로 시간을 잡아먹는 부분이 여기다. 기본 Azure 스타터는 읽기 전용이며, 구독을 명시적으로 고른 뒤에만 리소스 그룹을 나열한다.

글이 든 응용 예는 Azure Container Apps 스케일링 관리 캔버스다. 요구사항을 "구독을 명시적으로 고르게 하고, 레플리카 변경은 적용 전에 미리보기로 보여 줘"처럼 안전장치 중심으로 적는다. 쓰기 작업이 있는 캔버스일수록 프롬프트에 확인 단계를 넣으라는 힌트로 읽힌다. 실제로 글은 스타터를 고칠 때 Azure 작업 범위를 명시적으로 유지하고, 캔버스가 사용자에게 쓰기 승인을 분명히 요청하지 않는 한 스타터의 취소·검증·읽기 전용 동작을 보존하라고 적는다.

## 밑바탕: Canvas Toolkit

생성되는 프로젝트의 기반은 `@microsoft/canvas-toolkit` npm 패키지다. 글이 꼽는 구성은 다음과 같다.

- **Azure 연결**: Azure CLI 또는 Azure SDK 자격 증명으로 인증, 테넌트·구독 선택, Azure Resource Manager API 헬퍼
- **UI 구성 요소**: 헤더·버튼·목록 공통 스타일, 아이콘, 라이트/다크 대비 테마 색
- **로컬 백엔드 서버**: 캔버스 UI를 서빙하고 요청을 처리한다. origin 검사, 콘텐츠 보안 정책(CSP), 입력 검증 헬퍼가 들어 있다
- **상태 관리와 실시간 업데이트**: 가벼운 상태 저장소와 변경 알림으로 사람과 에이전트가 움직일 때 화면을 최신으로 유지한다
- **에이전트 연동**: 캔버스의 버튼이 확장에 정의된 프롬프트로 에이전트에게 도움을 요청하는 브리지

글은 플러그인이 Azure 캔버스를 움직이는 것과 같은 종류의 툴킷 기반 프로젝트를 만들기 때문에, 별도 저작 런타임에 묶이지 않는다고 설명한다.

## 아직 모르는 것

- 글에는 preview인지 GA인지 표기가 없다. 도입 전에 상태를 따로 확인해야 한다
- 복잡한 시나리오는 플러그인 생성만으로 끝나지 않고 툴킷을 직접 써야 한다고 한다. 어디까지 자동 생성되는지는 써 봐야 알 수 있다
- 읽기 전용 예제가 중심이라, 쓰기 작업의 안전성은 글만으로 판단하기 어렵다

이 글은 9월 29일 Azure Canvases 소개의 후속이다. 개념은 앞 글에서, 만드는 법은 이번 글에서 다룬다.

## 실무에서 달라지는 점

사내 Azure 구독을 관리하거나 개발 도구를 배포하는 입장, 그리고 이걸 가르치는 입장에서 정리하면 이렇다.

1. **플러그인 허용 정책부터 정한다.** 설치 경로는 Copilot의 **Customize > Plugins**와 Awesome Copilot 마켓플레이스다. 사내에서 Copilot 확장·플러그인 설치를 통제하고 있다면, 이 마켓플레이스를 허용할지와 누가 승인할지를 먼저 정해야 한다. 다음 날 나온 [Spec Kit 확장 구조](/posts/spec-kit-extension-layers/)의 "조직 카탈로그만 설치 소스로 쓴다"는 원칙을 그대로 적용할 수 있다.
2. **권한은 Azure CLI 로그인 계정을 그대로 따른다.** 캔버스는 사용자의 CLI 프로필로 Azure에 붙는다. 캔버스에 읽기 전용 기본값이 있어도, 로그인한 계정에 Contributor 이상 권한이 있으면 코드가 바뀌는 순간 쓰기가 가능해진다. 캔버스 검증용으로는 Reader 역할만 가진 계정이나 테스트 구독을 따로 두는 편이 안전하다.
3. **생성된 코드를 리뷰 대상으로 다룬다.** `dist/` 전체를 설치해 로컬 백엔드가 뜨는 구조라, 사실상 사내 PC에서 도는 작은 웹 서버다. origin 검사·CSP·입력 검증이 툴킷에 있다고 해도, 생성된 코드가 그것을 제대로 쓰는지는 사람이 확인해야 한다.
4. **비용·라이선스는 글에 없다.** 글은 preview/GA 여부, Node·Copilot·Azure CLI 버전 요구 사항, 가격이나 라이선스를 밝히지 않는다. Copilot 구독 플랜에서 플러그인 기능을 쓸 수 있는지, 캔버스 호스트가 어떤 클라이언트인지는 도입 전에 문서로 확인할 항목이다.
5. **강의에서는 '쓰기 전 미리보기' 패턴을 강조한다.** Container Apps 예제의 "구독 명시 선택 → 레플리카 변경 미리보기 → 적용"은 에이전트가 인프라를 만질 때의 기본 안전 패턴이다. 캔버스 자체보다 이 흐름을 프롬프트로 요구하는 습관을 가르치는 게 더 오래 남는다.

## 참고

- [Build Azure Canvases with Canvas authoring](https://developer.microsoft.com/blog/build-azure-canvases-with-canvas-authoring/) — Microsoft Developer Blog, 2026-10-06
- [Canvas authoring 플러그인 페이지 — Awesome Copilot](https://awesome-copilot.github.com/plugin/canvas-authoring/)
