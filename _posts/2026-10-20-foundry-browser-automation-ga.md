---
title: "Foundry 브라우저 자동화 도구 GA — 에이전트에게 브라우저를 맡기는 법"
date: 2026-10-20 06:51:00 +0900
categories: [IT News]
tags: [foundry, browser-automation, playwright, ai-agent, mcp]
description: Foundry의 Browser Automation Tool이 정식 출시됐다. 관리형 클라우드 브라우저, Live View·Take control, 단계적 권한 개방까지 운영자 관점에서 정리했다.
---

> 이 글은 Microsoft Foundry 블로그의 "Browser Automation Tool GA" 발표를 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록이 아니라 원문 내용과 제 해석을 구분해 적었습니다.
{: .prompt-info }

**결론부터.** Foundry 에이전트가 웹 화면을 직접 조작하는 Browser Automation Tool이 GA가 됐습니다. API가 없는 사내 포털·관리 콘솔·벤더 사이트를 에이전트가 사람처럼 열어 처리하게 하는 도구입니다. 눈여겨볼 것은 기능 자체보다 **"브라우저를 에이전트가 쓰는 게 아니라 Azure 리소스로 운영한다"**는 구조와, 사람이 중간에 개입하는 장치입니다.

## 원문이 말하는 것

- 도구는 **Playwright Workspaces**로 구동됩니다. 격리된 클라우드 브라우저 세션이 필요할 때 만들어져서, 에이전트 환경에 브라우저 바이너리를 설치·운영할 필요가 없습니다.
- 에이전트는 현재 페이지를 살피고, 동작을 고르고, 실행한 뒤 결과 상태를 확인하고 다음으로 가는 **관찰·행동 루프**를 돕니다. 화면이 다시 그려지거나 대화상자가 뜨는 경우를 염두에 둔 설계라고 합니다.
- Playwright Workspaces의 사설망 연결을 쓰면 Azure 가상 네트워크 뒤의 앱에도 접근합니다. 이 연결은 프로비저닝 때 설정해야 합니다.
- 도구는 Foundry **Toolbox**에 담겨 MCP 호환 엔드포인트로 노출됩니다. 지원 에이전트 유형은 프롬프트 에이전트, 호스티드 에이전트(Microsoft Agent Framework 등), GitHub Copilot 하니스입니다.
- 사람 감독 장치는 두 가지입니다. **Live View**는 진행 중인 세션을 실시간으로 보는 것, **Take control**은 사람이 직접 브라우저를 조작해 막힌 상태를 풀고 다시 에이전트에게 넘기는 것입니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="에이전트가 Toolbox의 MCP 주소로 브라우저 도구를 호출하고, 격리된 클라우드 브라우저가 웹과 사설 앱에 접근하며, 사람은 Live View로 관찰하다 Take control로 개입하는 구조">
  <defs>
    <marker id="fba-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="fba-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 에이전트 → Toolbox → 클라우드 브라우저, 그리고 사람</text>
  <rect x="8" y="50" width="150" height="56" rx="8" class="sk-box"/>
  <text x="83" y="74" text-anchor="middle" class="sk-label">에이전트</text>
  <text x="83" y="92" text-anchor="middle" class="sk-sub">프롬프트 / 호스티드</text>
  <rect x="210" y="50" width="150" height="56" rx="8" class="sk-box"/>
  <text x="285" y="74" text-anchor="middle" class="sk-label">Toolbox</text>
  <text x="285" y="92" text-anchor="middle" class="sk-sub">MCP 엔드포인트</text>
  <rect x="412" y="50" width="150" height="56" rx="8" class="sk-box-accent"/>
  <text x="487" y="74" text-anchor="middle" class="sk-label">Playwright Workspace</text>
  <text x="487" y="92" text-anchor="middle" class="sk-sub">격리된 브라우저 세션</text>
  <rect x="590" y="50" width="122" height="56" rx="8" class="sk-box"/>
  <text x="651" y="74" text-anchor="middle" class="sk-label">웹 · 사설 앱</text>
  <text x="651" y="92" text-anchor="middle" class="sk-sub">VNet 뒤 포함</text>
  <path d="M160,78 L208,78" class="sk-line" marker-end="url(#fba-m1)"/>
  <path d="M362,78 L410,78" class="sk-line" marker-end="url(#fba-m1)"/>
  <path d="M564,78 L588,78" class="sk-line" marker-end="url(#fba-m1)"/>
  <rect x="412" y="150" width="150" height="52" rx="8" class="sk-box-muted"/>
  <text x="487" y="173" text-anchor="middle" class="sk-label">사람</text>
  <text x="487" y="191" text-anchor="middle" class="sk-sub">Live View · Take control</text>
  <path d="M487,148 L487,108" class="sk-line-accent" marker-end="url(#fba-a1)"/>
</svg>
<figcaption>브라우저는 에이전트 안이 아니라 별도 Azure 리소스이고, 사람은 그 세션 옆에서 지켜보다 개입한다.</figcaption>
</figure>

## 설정 순서

원문 기준으로 ① Playwright Workspace 만들기(사설 접근이면 프로비저닝 때 사설망 연결 추가), ② Foundry에서 브라우저 자동화 연결을 만들고 연결 ID 확인, ③ 브라우저 도구를 Toolbox에 넣어 게시, ④ Toolbox의 MCP URL을 에이전트에 연결입니다. URL 형태는 `{PROJECT_ENDPOINT}/toolboxes/{이름}/versions/{버전}/mcp?api-version=v1` 입니다. 샘플은 Python으로 `BrowserAutomationToolboxTool`로 Toolbox 버전을 만들고 `FoundryToolbox`로 에이전트에 붙입니다.

> 원문 코드 블록은 잔여 조각(`timeout=120.0)`)과 빠진 부분이 있어 그대로는 돌아가지 않아 보입니다. 따라 하기 전에 공식 문서와 샘플 저장소로 보완하세요.
{: .prompt-warning }

## 이 블로그의 해석: '권한을 단계적으로 연다'가 핵심

원문은 **탐색·페이지 검사 도구부터 시작하고, 클릭·입력·양식 작성·파일 업로드는 필요할 때만 켜라**고 권합니다. 저는 이게 이 발표의 가장 실용적인 문장이라고 봅니다. 브라우저 에이전트의 위험은 "읽기"가 아니라 "쓰기"에서 생기기 때문입니다. 읽기 전용으로 시작해 Live View로 움직임을 본 뒤 도구를 하나씩 여는 순서가 자연스럽습니다(이것은 제 설계 제안이며 원문의 검증된 절차는 아닙니다).

또 하나, 웹 페이지 내용은 **신뢰할 수 없는 입력**이라는 안내가 있습니다. 페이지 안에 숨은 지시문이 에이전트를 흔들 수 있으니 시스템 정책과 페이지 내용을 분리하고, 중요한 동작은 사람 승인을 거치라는 것입니다. 에이전트 권한 설계라는 같은 주제는 [Azure MCP Server 3.0 안정판](/posts/azure-mcp-server-3-stable/), [Durable Task 승인 설계](/posts/durable-task-real-world-boundaries/) 글에서도 다뤘습니다. 브라우저는 그 논의가 화면 단위로 내려온 사례입니다.

## 실무에서 달라지는 점

1. **API가 없어 RPA나 수작업으로 남겨 둔 업무를 다시 목록화합니다.** 벤더 포털의 월간 내려받기, 관리 콘솔의 상태 확인처럼 읽기 위주 업무가 첫 후보입니다. 쓰기 업무는 뒤로 미룹니다.
2. **Playwright Workspace는 별도 Azure 리소스입니다.** 담당 구독, 리소스 그룹, 비용 귀속을 먼저 정해야 합니다. 가격·한도·지원 리전은 원문에 없으므로 도입 전 공식 문서에서 확인해야 합니다.
3. **사내 앱을 붙일 계획이면 네트워크 설계가 먼저입니다.** 사설망 연결은 프로비저닝 때 설정하는 항목이라, 나중에 붙이기보다 처음부터 VNet 구성을 합의하는 편이 낫습니다.
4. **신원과 권한은 사람 계정이 아니라 에이전트용으로 분리합니다.** 원문은 Entra ID, 관리 ID, Azure RBAC 기반 최소 권한과 도구 허용 목록, 승인 절차를 권합니다. 에이전트가 로그인해 쓰는 계정이 관리자급이면 이 장치들이 무의미해집니다.
5. **트레이스에 민감 정보가 남을 수 있습니다.** 프롬프트와 도구 인자·결과가 기록될 수 있고, 민감 콘텐츠를 분리하는 기능은 아직 프리뷰입니다. 운영 전에 접근 권한과 보존 정책을 점검하세요.
6. **강의에서는 이렇게 강조하겠습니다.** "에이전트가 클릭한다"보다 "누가 지켜보고, 언제 멈추고, 어디까지 허용했나"가 설계의 본론입니다. Live View와 Take control은 그 질문의 답입니다.

## 아직 확인되지 않은 것

가격, 한도, 리전은 원문에 없습니다. 도구 자체는 GA지만 트레이스의 민감 콘텐츠 보호 워크플로는 프리뷰입니다. 저는 이 도구를 직접 실행해 보지 않았고, 위 내용은 발표 글 기준입니다.

앞선 Foundry 업데이트 흐름은 [Foundry 9월 업데이트 정리](/posts/foundry-sept-2026-two-agent-changelog/)에서 볼 수 있습니다. Toolbox가 도구의 공통 출입구가 되면서 브라우저도 그 안에 들어온 모양새입니다.

## 참고

- [Browser Automation Tool in Microsoft Foundry — GA (Microsoft Foundry Blog)](https://devblogs.microsoft.com/foundry/browser-automation-tool-ga/)
