---
title: "Azure DevOps Agentic Autofix 프리뷰 — 보안 알림을 PR로, 과금은 크레딧"
date: 2026-11-24 06:51:00 +0900
categories: [IT News]
tags: [devops, security, github, cost]
description: Azure DevOps용 GitHub Advanced Security에 Agentic Autofix가 퍼블릭 프리뷰로 들어왔다. 켜는 위치, 실행 신원, 크레딧 과금, 도입 전 점검점을 정리했다.
---

> 이 글은 Azure DevOps 블로그의 "Agentic Autofix for GitHub Advanced Security for Azure DevOps is in public preview"(2026-11-23, Laura Jiang)를 읽고 정리·재구성한 것입니다. 상태·수치는 원문 기준이며, 직접 실행해 확인한 내용이 아닙니다.
{: .prompt-info }

## 결론부터

Azure DevOps의 보안 알림(코드 스캔·의존성)에서 **Generate fix를 누르면 에이전트가 수정안을 만들어 PR로 올려 주는 기능**이 퍼블릭 프리뷰로 열렸습니다. 원문은 6월 프라이빗 프리뷰, 10월 제한 프리뷰를 거쳤다고 설명합니다. 핵심은 세 가지입니다.

- 켜는 것과 실행하는 것이 분리돼 있습니다. 설정을 켜도 스캔이나 수정이 저절로 돌지 않고, 사람이 알림마다 고릅니다.
- 에이전트는 PR만 열고 머지하지 않습니다.
- 수정 1건마다 토큰이 GitHub AI 크레딧(1크레딧 = 0.01달러)으로 환산돼 Azure 구독에 청구됩니다. 좌석 라이선스와 별개의 종량 비용이라는 점이 운영자에게 가장 큰 변화입니다.

## 원문이 말하는 것

### 켜는 위치

조직 설정(Organization settings) 또는 프로젝트 설정(Project settings)의 Repositories에서 Advanced Security 아래 "Autofix for security alerts"를 선택합니다. 이 설정 하나가 CodeQL과 서드파티 SARIF 도구의 코드 스캔 알림, 그리고 의존성 취약점 알림을 모두 대상으로 삼는다고 합니다. 코드 보안(Code Security) 단독 라이선스는 옵션 패널에서 같은 항목을 선택하고 Apply 합니다.

### 실행과 결과물

Repos > Advanced Security에서 지원되는 알림을 열고 Generate fix를 선택하면 Azure Pipelines 잡이 대기열에 들어가고, `GitHub Copilot ({organization})`이라는 전용 신원으로 PR을 엽니다. PR에는 알림 요약이 붙고 "Copilot Autofix" 라벨이 달려 필터링할 수 있습니다. 일시 오류나 저장소 정책에 막히면 알림 상세에 실패 상태가 나오고, 원인을 고친 뒤 Retry fix를 누릅니다. 모든 알림에서 수정이 나오는 것은 아니라고 원문이 밝힙니다.

### 커스터마이징

`.github/copilot-instructions.md`에 코딩 표준·명명 규칙·테스트 명령을 적어 두면 에이전트가 참고합니다. 스킬은 `.github/skills/<skill-name>/SKILL.md` 또는 `.agents/skills/<skill-name>/SKILL.md`에 둡니다. 원문 예시는 "승인된 입력 검증 헬퍼를 쓰라"는 지침과, 사내 패키지 업데이트 절차를 담은 의존성 스킬입니다. 정적인 한 줄 제안이 아니라 저장소 맥락을 탐색해 여러 파일을 고칠 수 있다는 설명입니다.

### 과금

수정 생성은 Azure DevOps 조직에 연결된 Azure 구독에 청구됩니다. Cost Management에는 "AI Credit for AzDO GAC" 미터로 나타나고, 조직명·프로젝트명 태그가 붙어 프로젝트별로 나눠 볼 수 있습니다. 비용은 코드 맥락 크기와 수정 복잡도에 따라 달라지며, 예산 알림(budget alert)을 미터 또는 프로젝트 태그 기준으로 걸 수 있다고 합니다. 알림 일괄 선택(batch alert selection)은 로드맵 항목입니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="Agentic Autofix 흐름: 알림 선택, Generate fix, 파이프라인 잡, PR, 사람 검토와 머지, 재스캔">
  <defs>
    <marker id="af-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="af-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① Autofix가 도는 순서 — 사람이 누르고, 사람이 닫는다</text>
  <rect x="8" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="68" y="75" text-anchor="middle" class="sk-label">알림 선택</text>
  <text x="68" y="93" text-anchor="middle" class="sk-sub">사람이 고른다</text>
  <rect x="158" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="218" y="75" text-anchor="middle" class="sk-label">Generate fix</text>
  <text x="218" y="93" text-anchor="middle" class="sk-sub">여기서 과금 시작</text>
  <rect x="308" y="50" width="120" height="56" rx="8" class="sk-box-accent"/>
  <text x="368" y="75" text-anchor="middle" class="sk-label">Pipelines 잡</text>
  <text x="368" y="93" text-anchor="middle" class="sk-sub">전용 신원으로 실행</text>
  <rect x="458" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="518" y="75" text-anchor="middle" class="sk-label">PR 생성</text>
  <text x="518" y="93" text-anchor="middle" class="sk-sub">Copilot Autofix 라벨</text>
  <rect x="608" y="50" width="104" height="56" rx="8" class="sk-box"/>
  <text x="660" y="75" text-anchor="middle" class="sk-label">검토·머지</text>
  <text x="660" y="93" text-anchor="middle" class="sk-sub">자동 머지 없음</text>
  <path d="M130,78 L156,78" class="sk-line" marker-end="url(#af-m1)"/>
  <path d="M280,78 L306,78" class="sk-line" marker-end="url(#af-m1)"/>
  <path d="M430,78 L456,78" class="sk-line" marker-end="url(#af-m1)"/>
  <path d="M580,78 L606,78" class="sk-line-accent" marker-end="url(#af-a1)"/>
  <rect x="158" y="150" width="270" height="44" rx="8" class="sk-box-muted"/>
  <text x="293" y="170" text-anchor="middle" class="sk-label">실패 시 Retry fix</text>
  <text x="293" y="186" text-anchor="middle" class="sk-sub">정책 차단·일시 오류는 원인 해결 후</text>
  <path d="M368,108 L368,148" class="sk-line" marker-end="url(#af-m1)"/>
</svg>
<figcaption>자동화되는 것은 "수정안 작성과 PR 생성"까지이고, 머지와 재스캔 확인은 여전히 사람의 일이다.</figcaption>
</figure>

## 실무에서 달라지는 점

### 1. 비용 구조가 좌석에서 종량으로 늘어난다

GitHub Advanced Security 또는 Code Security 라이선스가 접근 조건이지만, 수정 생성 자체는 별도 크레딧 과금입니다. 이전에 정리한 [Azure Repos Copilot 코드 리뷰 GA](/posts/azure-repos-copilot-code-review-ga/) 때도 설정·과금 점검이 먼저였는데, 이번에는 한 단계 더 나아가 "알림 한 건당 비용"이 생깁니다. 운영자는 켜기 전에 Azure 구독 연결 상태와 Cost Management 접근 권한 보유자를 확인해야 합니다.

### 2. 조직 전체 스위치 대신 프로젝트 단위로 시작하는 편이 안전하다

설정은 조직·프로젝트·저장소 수준에서 켤 수 있습니다. 비용 태그가 조직·프로젝트 단위로 붙으므로, 처음에는 시범 프로젝트 하나에서 켜고 미터와 태그가 기대대로 찍히는지 본 뒤 예산 알림을 걸고 넓히는 순서가 합리적입니다. 이 순서는 원문 내용에서 제가 끌어낸 권장안입니다.

### 3. 에이전트 신원과 브랜치 정책을 먼저 점검한다

PR은 `GitHub Copilot ({organization})` 신원으로 열립니다. 브랜치 정책(필수 리뷰어, 빌드 검증, 작업 항목 연결)이 이 신원의 PR을 막으면 원문대로 실패 상태가 됩니다. 정책을 느슨하게 풀기보다, 이 신원이 PR을 열 수 있는지와 리뷰 규칙이 그대로 적용되는지를 확인하는 것이 맞습니다. 정확한 권한 요건은 원문이 공식 문서로 미뤄 두었으므로 문서를 확인해야 합니다.

### 4. 지침 파일이 곧 품질 관리 수단이다

`.github/copilot-instructions.md`와 SKILL.md 경로가 지원되므로, 사내 표준 라이브러리나 테스트 명령을 적어 두는 일이 수정 품질을 좌우합니다. 스킬 작성법은 [SKILL.md 첫 스킬 만들기](/posts/agent-skills-skill-md-setup-guide/)에 정리해 두었습니다. 다만 Azure DevOps Autofix가 어떤 범위까지 이 파일들을 읽는지는 원문 수준 이상으로 확인된 바 없습니다.

### 5. 교육할 때는 "머지 전 검증"을 강조한다

원문은 diff 검토, 빌드·테스트 실행, 정상 PR 절차로 머지, 대상 브랜치 재스캔으로 알림 종료 확인까지를 권합니다. 교육에서는 "AI가 고쳤으니 안전하다"가 아니라 "수정안 하나가 들어온 PR"로 다루고, 재스캔으로 알림이 닫혔는지를 확인하는 단계를 빼먹지 않도록 짚어야 합니다.

## 아직 확인되지 않은 것

원문에는 지원 언어, 구체적인 CodeQL 알림 범주, 권한 요건이 나와 있지 않습니다. 퍼블릭 프리뷰이므로 프로덕션 저장소에 일괄 적용하기보다 범위를 좁혀 검증하고, 일괄 선택 기능이 들어오면 크레딧 소모 패턴이 달라질 수 있다는 점도 염두에 두세요. Copilot 앱 연동 소식은 [Azure DevOps 팀의 11월 19일](/posts/azure-devops-copilot-app-preview-and-elm-ga/)에서 이어 읽을 수 있습니다.

## 참고

- 원문: [Agentic Autofix for GitHub Advanced Security for Azure DevOps is in public preview](https://devblogs.microsoft.com/devops/agentic-autofix-for-advanced-security-for-azure-devops-public-preview/)
- [Copilot Autofix 문서](https://learn.microsoft.com/azure/devops/repos/security/github-advanced-security-autofix)
- [청구 개요 — Copilot Autofix 사용 요금](https://learn.microsoft.com/azure/devops/repos/security/github-advanced-security-billing#copilot-autofix-usage-charges)
- [Azure 예산 알림 만들기](https://learn.microsoft.com/azure/cost-management-billing/costs/tutorial-acm-create-budgets)
- [10월 제한 프리뷰 소개 글](https://devblogs.microsoft.com/devops/introducing-agentic-autofix-for-code-and-dependencies/)
- [알림 일괄 선택 로드맵](https://learn.microsoft.com/azure/devops/release-notes/roadmap/2026/ghazdo/batch-alert)
