---
title: "Copilot 보조 권한(Assisted permissions) GA — 승인 창을 모델이 먼저 거른다"
date: 2026-10-28 06:53:00 +0900
categories: [IT News]
tags: [copilot, github, security, governance, ai-agent]
description: GitHub Copilot의 보조 권한이 GA가 됐다. LLM이 안전한 요청만 자동 통과시키는 방식의 동작 순서, 관리자 정책, 한계를 운영자 관점으로 정리했다.
---

> 이 글은 GitHub changelog "GitHub Copilot assisted permissions are generally available"(2026-10-26)과 GitHub 문서 "Allowing safe commands to run without manual approval"을 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록이 아니라 공개된 설명을 해석한 글입니다.
{: .prompt-info }

## 결론부터

Copilot이 명령 실행, 파일 경로 접근, URL 열기를 요청할 때 LLM이 먼저 평가해서 "clearly safe"로 분류한 것만 자동 진행하고, 나머지는 사람에게 묻는 모드가 GA가 됐습니다. 대상은 Copilot CLI, Copilot 앱, VS Code 세 곳입니다.

핵심은 "승인 창을 없앤다"가 아닙니다. 매번 누르는 수동 승인과 전부 열어 두는 Allow All 사이에 **중간 단계**가 생겼다는 점입니다. 다만 문서가 스스로 밝히듯 이 판정은 모델이 하는 것이라 틀릴 수 있고, 샌드박스나 접근 제한을 대신하지 못합니다. 편의 기능이지 보안 경계가 아닙니다.

## 무엇이 발표됐나

changelog 본문은 짧습니다. 켜는 방법은 표면마다 다릅니다.

- Copilot CLI: `/permissions`를 실행하고 assisted를 선택
- Copilot 앱: 모드 선택기에서 Tool permissions를 열고 Assisted 선택
- VS Code: 권한 선택기에서 Assisted permissions 선택

관리자 쪽에서는 엔터프라이즈 관리 설정으로 Allow All은 막으면서 Assisted는 허용할 수 있고, Assisted만 따로 끌 수도 있으며, 새 세션의 기본 모드를 지정할 수 있습니다. 기본값이 정책상 불가능하면 세션은 수동 승인으로 시작합니다. 문서에는 끄는 설정으로 `permissions.disableAssistedPermissionsMode: true`가 나옵니다.

## 판정은 어떤 순서로 일어나나

changelog에는 없고 문서에서 확인되는 내용이 운영상 더 중요합니다. 모델이 모든 요청을 보는 것이 아닙니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="보조 권한 판정 순서: 기존 규칙과 이전 승인, 모델 평가, 사용자 승인 순으로 진행되고 엔터프라이즈 거부 규칙은 항상 적용된다">
  <defs>
    <marker id="cap-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 보조 권한의 판정 순서</text>
  <rect x="8" y="50" width="150" height="56" rx="8" class="sk-box"/>
  <text x="83" y="74" text-anchor="middle" class="sk-label">도구 요청</text>
  <text x="83" y="92" text-anchor="middle" class="sk-sub">명령·경로·URL</text>
  <path d="M160,78 L196,78" class="sk-line" marker-end="url(#cap-m1)"/>
  <rect x="198" y="50" width="160" height="56" rx="8" class="sk-box"/>
  <text x="278" y="74" text-anchor="middle" class="sk-label">기존 규칙·이전 승인</text>
  <text x="278" y="92" text-anchor="middle" class="sk-sub">허용돼 있으면 바로 실행</text>
  <path d="M360,78 L396,78" class="sk-line" marker-end="url(#cap-m1)"/>
  <rect x="398" y="50" width="150" height="56" rx="8" class="sk-box-accent"/>
  <text x="473" y="74" text-anchor="middle" class="sk-label">모델 평가</text>
  <text x="473" y="92" text-anchor="middle" class="sk-sub">확신하면 자동 진행</text>
  <path d="M550,78 L586,78" class="sk-line" marker-end="url(#cap-m1)"/>
  <rect x="588" y="50" width="124" height="56" rx="8" class="sk-box-muted"/>
  <text x="650" y="74" text-anchor="middle" class="sk-label">사람 승인</text>
  <text x="650" y="92" text-anchor="middle" class="sk-sub">확신 못 하면</text>
  <rect x="198" y="150" width="514" height="40" rx="8" class="sk-box"/>
  <text x="455" y="175" text-anchor="middle" class="sk-label">엔터프라이즈 거부(deny) 규칙은 어느 단계에서나 항상 적용</text>
</svg>
<figcaption>모델은 "이미 해결되지 않은 요청"만 심사하고, 거부 규칙은 모델 판단보다 위에 있다.</figcaption>
</figure>

문서가 설명하는 흐름은 이렇습니다. 기존 규칙과 이전 사람 승인을 먼저 확인하고, 남은 요청 중 심사 대상인 것만 모델이 "안전한가, 그리고 사용자가 이 작업에 준 권한 범위에 맞는가"를 봅니다. 확신하면 진행하고, 아니면 묻습니다. 심사가 불가능하면 평소 권한 처리로 돌아갑니다. 심사 모델은 대화 모델과 다를 수 있고, 안 되면 세션의 활성 모델로 대체될 수 있다고 합니다.

엔터프라이즈 거부 규칙은 항상 적용됩니다. 반대로 CLI에서는 엔터프라이즈 "ask" 규칙(셸 명령, 파일 읽기·쓰기, URL)이 모델 승인으로 충족될 수 있다고 하니, 정책을 ask로 걸어 둔 조직은 이 부분을 눈여겨봐야 합니다. 또 샌드박스를 우회하려는 시도처럼 심사 대상이 아닌 요청은 평소처럼 처리됩니다.

## 문서가 밝힌 한계

이 기능을 평가할 때는 한계 목록이 본문입니다.

- 모델이 실수해서 위험하거나 의도하지 않은 동작을 승인할 수 있다고 문서가 직접 적고 있습니다.
- 샌드박싱이나 접근 제한의 대체재가 아닙니다.
- `--assisted-approval`은 `--allow-all-paths`, `--allow-all-urls`를 바꾸지 않습니다. `--allow-all`과 함께 쓰면 경로·URL 접근이 열린 채로 남을 수 있습니다.
- 보조 권한을 켤 수 없으면 CLI가 stderr로 경고하고, `--allow-all-tools` 옵션은 심사 없이 그대로 적용됩니다.
- 자동 모드(autopilot)와 비대화형 실행에서는 승인이 필요한 요청이 거부되고, 심사 설명이 Copilot에 돌아가 다른 방법을 시도하게 됩니다.
- 클라우드 세션에는 도구 권한 제어가 없습니다.

지원 요금제와 분류 정확도에 대한 수치는 changelog와 문서 어디에도 없습니다. 정확도를 말하는 글이 있다면 출처를 의심해도 좋습니다.

## 이전 글과 이어 읽기

에이전트를 "내부자"로 보고 권한을 통제해야 한다는 흐름은 [에이전트도 '내부자'다](/posts/securing-agentic-insider-guardian-agents/)에서, 서명·정책·사람 승인으로 에이전트를 묶는 설계는 [Chrysalis 글](/posts/governed-self-evolving-agent-chrysalis/)에서 다뤘습니다. 보조 권한은 그 원칙이 개발 도구의 실제 설정 항목으로 내려온 사례입니다. 다만 위 글들이 말한 "독립된 감시 계층"과 달리, 여기서는 심사하는 모델과 작업하는 모델이 같은 계열일 수 있다는 점이 다릅니다. 판정자가 피판정자와 컨텍스트를 공유하는 구조라서, 프롬프트 인젝션이 섞인 입력 앞에서 얼마나 버티는지는 별도로 물어야 할 질문입니다.

## 실무에서 달라지는 점

### 1. 관리자: 허용 범위부터 정한다

Allow All 차단과 Assisted 허용을 동시에 걸 수 있으니, "전부 열기 아니면 전부 막기" 대신 Assisted를 기본값으로 두는 설계가 가능해졌습니다. 먼저 확인할 것은 기본 모드 지정이 정책과 충돌하지 않는지입니다. 충돌하면 세션이 수동 승인으로 시작하므로, 개발자 입장에서는 "설정이 안 먹는다"로 보입니다. 사내 안내문에 이 동작을 적어 두면 문의가 줄 것입니다.

### 2. 보안 담당: ask 규칙과 deny 규칙을 구분해 본다

deny는 모델이 뚫지 못하지만, CLI의 ask 규칙은 모델 승인으로 충족될 수 있습니다. 지금 ask로 걸어 둔 규칙 중 "사람이 반드시 봐야 한다"는 의도였던 것은 deny로 옮길지 검토할 항목입니다. 도입 전에 `--allow-all-*` 플래그가 스크립트나 별칭에 남아 있는지도 점검하세요. 문서상 이 플래그들은 심사를 우회합니다.

### 3. 개발자: 비대화형·CI에서는 동작이 다르다

자동 모드와 비대화형 실행에서는 "묻는" 대신 "거부"됩니다. 파이프라인에서 에이전트를 돌리는 경우, 승인이 필요한 요청이 조용히 거부되고 에이전트가 우회 경로를 찾을 수 있습니다. 도입 전에 어떤 작업이 거부되는지 로그로 확인할 필요가 있습니다.

### 4. IT 강사: "자동 통과 = 검증됨"이 아님을 강조한다

교육할 때 가장 위험한 오해는 "모델이 승인했으니 안전하다"입니다. 문서가 한계로 적은 문장을 그대로 교재에 넣는 것을 권합니다. 수동 승인, Assisted, Allow All 세 단계의 차이를 표로 보여 주고, 샌드박스·최소 권한 같은 기존 통제는 그대로 필요하다는 점을 같이 가르치는 편이 낫습니다. 비교 실습은 같은 작업을 모드별로 돌려 승인 횟수와 자동 통과·차단 항목을 기록하는 방식이 적합해 보이지만, 이 글은 그것을 해본 결과가 아닙니다.

### 5. 도입 전 확인 목록

- 사용 중인 Copilot 요금제에서 쓸 수 있는지(공개 자료에 명시 없음, 직접 확인 필요)
- 심사 모델이 어떤 모델로 동작하는지, 조직 정책상 허용되는 모델인지
- 엔터프라이즈 관리 설정의 permissions 항목에서 기본 모드와 비활성화 옵션 확인

## 참고

- GitHub Changelog, GitHub Copilot assisted permissions are generally available (2026-10-26): https://github.blog/changelog/2026-10-26-github-copilot-assisted-permissions-generally-available
- GitHub Docs, Allowing safe commands to run without manual approval: https://docs.github.com/copilot/concepts/customizations/allowing-safe-commands
- GitHub Docs, Enterprise-managed settings (permissions): https://docs.github.com/enterprise-cloud@latest/copilot/reference/enterprise-administrators/enterprise-managed-settings#permissions
