---
title: "Azure DevOps 팀의 11월 19일 — Copilot 앱 연동(프리뷰)과 ELM 이전(GA)"
date: 2026-11-20 06:51:00 +0900
categories: [IT News]
tags: [github, devops, copilot]
description: Copilot 앱이 Azure DevOps 저장소에서 에이전트 세션을 돌리는 기능은 프리뷰, Azure DevOps에서 GitHub로 옮기는 ELM은 GA. 둘을 상태별로 나눠 읽는다.
---

> 이 글은 Azure DevOps 블로그의 "Azure DevOps integration with the GitHub Copilot app (public preview)"와 GitHub changelog의 "Enterprise Live Migrations from Azure DevOps generally available", 그리고 Microsoft Learn의 ELM 개요 문서를 읽고 정리·재구성한 것이다.
{: .prompt-info }

## 결론 먼저

2026년 11월 19일에 Azure DevOps와 GitHub를 잇는 소식이 두 건 나왔다. 상태가 서로 다르다.

- Copilot 앱에서 Azure DevOps 저장소로 에이전트 세션을 돌리는 기능은 **퍼블릭 프리뷰**다.
- Azure DevOps Services에서 GitHub Enterprise Cloud(데이터 레지던시)로 저장소를 옮기는 Enterprise Live Migrations(ELM)는 **GA**다.

둘을 한 묶음으로 읽으면 "Azure DevOps를 쓰면서 GitHub 쪽 도구를 붙이는 길"과 "아예 GitHub로 옮기는 길"이 동시에 열렸다는 그림이 나온다. 다만 프리뷰와 GA를 같은 무게로 보면 안 된다. 운영 계획에 넣을 수 있는 것은 GA 쪽이고, 프리뷰는 시험 대상이다.

<figure class="sketch">
<svg viewBox="0 0 720 260" role="img" aria-label="Azure DevOps에서 두 갈래: 남아서 Copilot 앱을 붙이는 길과 GitHub로 이전하는 길">
  <defs>
    <marker id="ado-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ado-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① Azure DevOps 저장소가 갈 수 있는 두 길</text>
  <rect x="8" y="90" width="150" height="60" rx="8" class="sk-box"/>
  <text x="83" y="116" text-anchor="middle" class="sk-label">Azure Repos</text>
  <text x="83" y="136" text-anchor="middle" class="sk-sub">기존 저장소</text>
  <path d="M160,105 L260,60" class="sk-line" marker-end="url(#ado-m1)"/>
  <path d="M160,135 L260,190" class="sk-line-accent" marker-end="url(#ado-a1)"/>
  <rect x="262" y="30" width="230" height="60" rx="8" class="sk-box-muted"/>
  <text x="377" y="56" text-anchor="middle" class="sk-label">남아서 Copilot 앱 연결</text>
  <text x="377" y="76" text-anchor="middle" class="sk-sub">퍼블릭 프리뷰</text>
  <rect x="262" y="160" width="230" height="60" rx="8" class="sk-box-accent"/>
  <text x="377" y="186" text-anchor="middle" class="sk-label">ELM으로 GitHub 이전</text>
  <text x="377" y="206" text-anchor="middle" class="sk-sub">GA · 이전 중에도 작업 가능</text>
  <path d="M494,60 L560,60" class="sk-line" marker-end="url(#ado-m1)"/>
  <path d="M494,190 L560,190" class="sk-line-accent" marker-end="url(#ado-a1)"/>
  <text x="565" y="64" class="sk-sub">브랜치·커밋·PR을 에이전트가</text>
  <text x="565" y="194" class="sk-sub">GitHub Enterprise Cloud</text>
</svg>
<figcaption>같은 날 나왔어도 한쪽은 시험용, 한쪽은 운영 계획에 넣을 수 있는 정식 기능이다.</figcaption>
</figure>

## 1. Copilot 앱의 Azure DevOps 연동 (퍼블릭 프리뷰)

### 무엇을 하나

글에 따르면 Copilot 앱에서 Azure DevOps 저장소를 대상으로 에이전트 세션을 시작하면, 에이전트가 브랜치를 만들고 코드를 고치고 커밋하고 PR을 연다. 세션은 여러 개를 병렬로 돌릴 수 있고, 앱 안에서 코드 리뷰·댓글·PR 관리도 한다. 'Agent Merge'를 켜면 에이전트가 PR을 지켜보다가 리뷰 댓글에 대응하고 CI 문제를 풀고, 준비되면 자동 병합까지 한다고 설명한다. 이 Agent Merge에는 글에서 GA·프리뷰 구분이 따로 붙어 있지 않다.

### 켜는 순서 (원문 기준)

1. **Settings** > **Experimental**에서 **Azure DevOps integration**을 켠다.
2. 권장 단계로, **Customize** > **Installed** > **Add server**에서 **Azure DevOps**(Azure DevOps Remote MCP Server)를 추가한다. 조직 이름 입력은 선택이다.
3. 새 세션을 만들고 **Add from Azure DevOps**를 골라 저장소를 선택한다.

메뉴 이름에 Experimental이 들어간다는 점이 이 기능의 성격을 보여 준다. 글은 프리뷰 기간에 빈틈을 메우겠다며, 작업 항목 보기·열기, 작업 항목에서 코딩 세션 시작, Projects 지원을 예정 사항으로 적었다. 즉 지금은 작업 항목 중심의 흐름이 비어 있다. 요금제 조건은 글에 나와 있지 않다.

### 이어 읽을 글

Copilot 앱이 사내 저장소 환경으로 넓어지는 흐름은 [Copilot 앱, GitHub Enterprise Server 지원 GA](/posts/github-copilot-app-ghes-ga/)에서 이어진다. 두 번째 단계의 MCP 서버 쪽 인증과 클라이언트별 설정은 [Azure DevOps 원격 MCP 서버 연결](/posts/azure-devops-remote-mcp-server-setup-guide/)에 정리해 두었다.

## 2. ELM — Azure DevOps에서 GitHub로 이전 (GA)

### 무엇이 GA인가

changelog는 Azure DevOps Services에서 GitHub Enterprise Cloud with data residency로 저장소를 옮기는 Enterprise Live Migrations가 GA라고 알린다. 핵심은 팀이 계속 일하는 동안 변경을 지속적으로 동기화하고, 원하는 시점에 짧은 컷오버로 전환한다는 점이다. 포털의 안내형 화면이나 Azure DevOps CLI로 관리한다.

### Microsoft Learn 개요 문서에서 확인되는 것

changelog에는 없고 Learn 문서에만 있는 운영 정보가 많아서 따로 적는다.

- 지원 범위는 Azure DevOps Services → GitHub Enterprise Cloud with data residency다. Azure DevOps Server(온프레미스)를 쓰고 있다면 먼저 Services로 옮겨야 한다고 문서가 적었다.
- 컷오버 때 읽기 전용 구간이 생기며, 문서는 대부분의 저장소에서 보통 30분 미만이라고 설명한다.
- 마이그레이션은 Azure Pipelines의 셀프호스트 Linux 에이전트에서 돈다. 에이전트는 이전이 끝날 때까지 켜 둬야 한다. ELM 자체는 무료이고, 조직의 Azure Pipelines 병렬 작업 한도를 넘길 때만 비용이 생길 수 있다고 한다.
- 사전 검증 한계: 이력 속 단일 파일 400MB 초과, 푸시 팩 2GB 초과, 브랜치·태그 이름 255바이트 초과는 검증이 실패한다.
- 검증 성공 후 24시간 안에 본 이전을 시작해야 하고, 시작 후 21일 안에 컷오버를 끝내야 한다.
- 옮겨지는 것: Git 이력, Git LFS 객체, 브랜치·태그, 브랜치 정책(GitHub 브랜치 룰셋으로), PR 메타데이터. 삭제된 브랜치의 abandoned PR은 제외된다.
- 옮겨지지 않는 것: 작업 항목, 파이프라인 정의, 릴리스, 위키, 테스트 계획·결과, Azure Artifacts.

## 실무에서 달라지는 점

### 1. 이전 대상 조직은 먼저 Services인지 확인

사내에 Azure DevOps Server가 남아 있다면 ELM의 직접 대상이 아니다. 문서 기준으로 Services로의 이전이 선행되므로, 일정표에 단계가 하나 더 들어간다. 이 점은 도입 검토 초기에 확인할 항목이다.

### 2. 저장소 사전 정리 목록을 만든다

이력에 400MB를 넘는 파일이 있거나 ref 이름이 긴 저장소는 검증에서 막힌다. 해결책은 파일 제거나 Git LFS 이전이다. 이력을 고치는 작업은 팀 합의가 필요해서, 이전 날짜보다 먼저 시작해야 한다. 검증 24시간, 컷오버 21일이라는 시간 제한도 운영 캘린더에 적어 둬야 한다.

### 3. 파이프라인·작업 항목·위키는 별도 계획

저장소만 옮겨지고 작업 항목, 파이프라인 정의, 위키는 남는다. changelog는 Azure Boards 연결 생성과 Azure Pipelines 재연결을 도와준다고 하지만, Learn 문서는 파이프라인 재연결 항목에 "(future)"라고 표시한다. 두 문서의 표현이 다르므로 실제 범위는 본인 환경에서 확인해야 한다. 정의 자체는 Azure DevOps에 남는다는 점은 둘 다 같다.

### 4. 권한과 토큰은 보안 담당과 함께

문서상 PAT는 Azure DevOps 서비스 연결에만 저장되고, 짧은 만료와 교체를 권한다. PAT 스코프는 문서 안에서도 표기가 어긋나 있어서(본문 목록과 역할 표가 다름), 발급 전에 Prerequisites 페이지를 직접 열어 확인하는 편이 안전하다. 감사 로그는 **Organization settings** > **Auditing**에서 Enterprise Live Migrations 영역으로 걸러 볼 수 있다.

### 5. 교육할 때는 프리뷰와 GA를 분리해서 말한다

Copilot 앱 연동은 Experimental 메뉴 안의 프리뷰다. 시연 용도로는 좋지만 "이제 Azure DevOps에서 Copilot 앱으로 개발하세요"라고 안내하면 과하다. 반대로 ELM은 정식이므로 이전 계획 수업에서는 제한 수치와 비이전 항목을 표로 보여 주는 쪽이 도움이 된다.

## 내 해석: 두 소식을 어떻게 묶어 읽나

Azure DevOps 사용 조직의 선택지는 이제 "남아서 에이전트 도구를 붙인다"와 "옮긴다"로 나뉜다. 프리뷰 연동은 작업 항목 연계가 아직 예정 단계라서, Boards를 중심으로 일하는 팀에게는 효용이 제한적일 수 있다. 이 부분은 글에서 확인된 예정 사항을 바탕으로 한 내 추정이다. 반대로 저장소 이력 중심으로 일하는 팀이라면 프리뷰를 작은 저장소 하나에 시험해 보고, 장기 방향은 ELM 제한 수치를 기준으로 판단하는 순서가 합리적이다. 두 길 모두 아직 비어 있는 부분(작업 항목, 파이프라인)이 같다는 점도 눈여겨볼 만하다.

## 참고

- Azure DevOps 블로그, Azure DevOps integration with the GitHub Copilot app (public preview): https://devblogs.microsoft.com/devops/azure-devops-integration-with-the-github-copilot-app-public-preview/
- GitHub changelog, Enterprise Live Migrations from Azure DevOps generally available: https://github.blog/changelog/2026-11-19-enterprise-live-migrations-from-azure-devops-generally-available
- Microsoft Learn, Introduction to Enterprise Live Migrations (ELM): https://learn.microsoft.com/azure/devops/repos/enterprise-live-migrations/overview
- GitHub Docs, GitHub Copilot app: https://docs.github.com/en/copilot/concepts/copilot-surfaces/github-copilot-app
