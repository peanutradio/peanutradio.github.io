---
title: "Azure DevOps 플러그인이 Microsoft Copilot에 들어왔다"
date: 2026-10-21 06:49:00 +0900
categories: [IT News]
tags: [azure-devops, copilot, cowork, plugin, backlog]
description: Azure DevOps 플러그인이 Copilot Chat·Cowork·Code에 들어왔다. 설치는 두 단계지만 GA 여부와 권한 요건은 글에 없어 도입 전 확인이 필요하다.
---

> 이 글은 Azure DevOps 블로그의 "Azure DevOps available in Microsoft Copilot"(2026-10-20, Dan Hellem)을 읽고 정리·재구성한 것입니다.
{: .prompt-info }

## 결론부터

Azure DevOps 플러그인이 Microsoft Copilot의 Chat, Cowork, Code 안에서 쓸 수 있게 됐습니다. 프로젝트, 작업 항목, PR, 테스트 계획, 파이프라인을 Copilot 대화 안에서 다루는 것이 목표입니다. 설치는 두 단계로 단순합니다. 다만 원문에는 **GA인지 프리뷰인지, 필요한 권한과 라이선스가 무엇인지** 적혀 있지 않습니다. 소개 글로 읽고, 실제 도입은 확인 후에 하는 게 맞습니다.

## 원문이 말하는 것

- 내장 스킬: 주의가 필요한 항목 찾기, 버그 분류(triage), 아이디어·기능 요청을 에픽·기능·사용자 스토리·인수 기준이 있는 백로그로 바꾸기.
- 다른 도구와 엮는 흐름: Word 요구사항 문서를 기능과 관련 작업 항목으로 바꾸기, 회의록 내용을 관련 작업 항목의 댓글로 달기.
- 설치: Cowork의 Customize에서 "Azure DevOps"를 검색해 플러그인을 설치하고, Azure DevOps 조직 이름을 입력한 뒤 Connect.
- 예시 프롬프트: 백로그를 검토해 오늘 집중할 상위 3개를 알려 달라.

수치나 성능 비교는 없는 순수 소개 글입니다. 글 아래 독자 댓글에 "필요한 요소가 아직 모든 사용자에게 열리지 않은 것 같다"는 말이 있지만, 글이 이를 확인해 주지는 않습니다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="Copilot 대화에서 Azure DevOps 플러그인을 거쳐 작업 항목, PR, 파이프라인으로 연결되는 구조">
  <defs>
    <marker id="ado-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 대화 → 플러그인 → 업무 데이터</text>
  <rect x="8" y="60" width="170" height="70" rx="8" class="sk-box"/>
  <text x="93" y="90" text-anchor="middle" class="sk-label">Copilot</text>
  <text x="93" y="110" text-anchor="middle" class="sk-sub">Chat · Cowork · Code</text>
  <rect x="270" y="60" width="170" height="70" rx="8" class="sk-box-accent"/>
  <text x="355" y="90" text-anchor="middle" class="sk-label">Azure DevOps 플러그인</text>
  <text x="355" y="110" text-anchor="middle" class="sk-sub">조직 이름으로 Connect</text>
  <rect x="532" y="30" width="180" height="40" rx="8" class="sk-box"/>
  <text x="622" y="55" text-anchor="middle" class="sk-label">작업 항목 · 백로그</text>
  <rect x="532" y="80" width="180" height="40" rx="8" class="sk-box"/>
  <text x="622" y="105" text-anchor="middle" class="sk-label">PR · 테스트 계획</text>
  <rect x="532" y="130" width="180" height="40" rx="8" class="sk-box"/>
  <text x="622" y="155" text-anchor="middle" class="sk-label">파이프라인</text>
  <path d="M180,95 L266,95" class="sk-line" marker-end="url(#ado-m1)"/>
  <path d="M442,95 L528,50" class="sk-line" marker-end="url(#ado-m1)"/>
  <path d="M442,95 L528,100" class="sk-line" marker-end="url(#ado-m1)"/>
  <path d="M442,95 L528,150" class="sk-line" marker-end="url(#ado-m1)"/>
</svg>
<figcaption>화면을 오가며 복사하던 일이 대화 한 곳으로 모이지만, 그 길목이 되는 플러그인의 권한 범위가 곧 위험 범위다.</figcaption>
</figure>

## 내 해석

이 소식의 핵심은 기능 목록보다 **Copilot 플러그인 생태계에 실제 업무 도구가 붙기 시작했다**는 점입니다. 에이전트에게 신원과 권한을 어떻게 줄지는 [Agent 365와 Agent ID 정리](/posts/agent-365-agent-id/)에서 다룬 문제와 그대로 이어집니다. 백로그를 읽는 것과 작업 항목을 만들고 고치는 것은 위험도가 전혀 다릅니다. 원문은 에이전트가 쓰기 작업을 어디까지 하는지 말하지 않으므로, 여기가 가장 먼저 확인할 지점입니다.

또 하나, 이 글의 시나리오(요구사항 문서 → 기능과 작업 항목 변환)는 결과물이 그럴듯해 보이기 쉬운 영역입니다. 생성된 인수 기준이 팀의 실제 정의와 맞는지는 사람이 검토해야 합니다.

## 실무에서 달라지는 점

- **Azure DevOps 관리자**: 플러그인이 연결되는 순간 Copilot이 어떤 권한으로 동작하는지부터 확인하세요. 사용자 본인의 권한을 따르는지, 별도 권한이 필요한지는 원문에 없습니다. 테스트 조직에서 읽기 전용 계정으로 먼저 연결해 보는 순서를 권합니다.
- **M365·Copilot 관리자**: 플러그인 설치 위치가 Cowork의 Customize입니다. 사용자가 스스로 설치할 수 있는지, 조직 정책으로 막을 수 있는지를 관리 센터에서 확인해 두는 편이 안전합니다. 라이선스 요건은 글에 없으니 별도로 확인해야 합니다.
- **프로젝트 매니저·팀장**: "백로그 정리"를 시킬 때 결과를 그대로 반영하지 말고, 에픽·스토리 초안으로만 쓰는 규칙을 먼저 정하세요. 인수 기준은 팀 템플릿과 대조하는 단계가 필요합니다.
- **IT 강사**: 교육에서는 "연결 자체"보다 "무엇을 읽고 무엇을 쓰는 도구인가"를 먼저 구분해서 보여 주는 게 좋습니다. 설치가 두 단계뿐이라 쉽게 시연되지만, 그래서 권한 설명이 빠지기 쉽습니다.
- **도입 보류 신호**: 독자 댓글처럼 일부 사용자에게 아직 열리지 않았을 수 있습니다. 파일럿 전에 우리 테넌트에서 플러그인이 실제로 보이는지부터 확인하세요.

## 참고

- [Azure DevOps available in Microsoft Copilot — Azure DevOps Blog (2026-10-20)](https://devblogs.microsoft.com/devops/azure-devops-available-in-microsoft-copilot/)
