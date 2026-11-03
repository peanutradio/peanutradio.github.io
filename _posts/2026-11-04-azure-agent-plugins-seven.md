---
title: "Azure 에이전트 플러그인 7종 — 필요한 것만 깔아야 하는 이유"
date: 2026-11-04 06:53:00 +0900
categories: [IT News]
tags: [azure, ai-agent, foundry, cost]
description: Microsoft가 Copilot CLI·Claude Code·Cursor용 Azure 플러그인 7종을 정리했다. 설치 명령과 비용 분석 기능 이동, 확인 절차를 운영자 관점으로 읽는다.
---

> 이 글은 Azure SDK 블로그의 "Explore specialized Azure plugins for your AI agent" 발표를 읽고 정리·재구성한 것입니다. 원문 링크는 맨 아래 참고에 있습니다.
{: .prompt-info }

## 결론부터

Azure 작업을 에이전트에게 맡길 때 쓰는 플러그인이 `azure` 하나에서 일곱 개로 나뉘었습니다. 핵심은 두 가지입니다.

첫째, 전부 설치하는 방식이 아닙니다. 플러그인마다 에이전트의 컨텍스트 창을 일부 차지하므로 필요한 것만 설치하라는 것이 원문의 권고입니다.

둘째, 이미 `azure` 플러그인을 쓰던 사람은 확인할 것이 하나 있습니다. 비용 분석 스킬이 core `azure`에서 `azure-cost`로 옮겨졌기 때문에, 비용 질문을 계속 하려면 `azure-cost`를 따로 설치해야 합니다.

## 무엇이 발표됐나

마켓플레이스 등록은 `/plugin` 명령을 지원하는 클라이언트에서 한 번만 하면 됩니다.

```
/plugin marketplace add microsoft/azure-skills
```

그다음 플러그인마다 `/plugin install <이름>@azure-skills` 형태로 설치합니다. 원문은 GitHub Copilot CLI, Claude Code, Cursor를 대상으로 설명합니다. 업데이트는 클라이언트의 플러그인 관리자나 `/plugin update azure-cost@azure-skills` 같은 명령으로 합니다.

<figure class="sketch">
<svg viewBox="0 0 720 280" role="img" aria-label="core azure 플러그인을 중심으로 필요한 전문 플러그인만 추가하는 구조">
  <defs>
    <marker id="az-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 핵심 하나 + 필요한 전문 플러그인만</text>
  <rect x="8" y="110" width="170" height="64" rx="8" class="sk-box-accent"/>
  <text x="93" y="138" text-anchor="middle" class="sk-label">azure (core)</text>
  <text x="93" y="158" text-anchor="middle" class="sk-sub">배포·리소스·모니터링·탐색 스킬</text>
  <rect x="300" y="36" width="190" height="40" rx="8" class="sk-box"/>
  <text x="395" y="61" text-anchor="middle" class="sk-mono">aks-skills</text>
  <rect x="300" y="84" width="190" height="40" rx="8" class="sk-box"/>
  <text x="395" y="109" text-anchor="middle" class="sk-mono">azure-cost</text>
  <rect x="300" y="132" width="190" height="40" rx="8" class="sk-box"/>
  <text x="395" y="157" text-anchor="middle" class="sk-mono">azure-advisor</text>
  <rect x="300" y="180" width="190" height="40" rx="8" class="sk-box"/>
  <text x="395" y="205" text-anchor="middle" class="sk-mono">foundry-iq-skills</text>
  <rect x="300" y="228" width="190" height="40" rx="8" class="sk-box-muted"/>
  <text x="395" y="253" text-anchor="middle" class="sk-sub">kusto-graph · azure-local</text>
  <path d="M180,142 L296,142" class="sk-line-accent" marker-end="url(#az-m1)"/>
  <text x="238" y="132" text-anchor="middle" class="sk-sub">필요할 때만</text>
  <rect x="540" y="110" width="172" height="64" rx="8" class="sk-box"/>
  <text x="626" y="138" text-anchor="middle" class="sk-label">컨텍스트 창</text>
  <text x="626" y="158" text-anchor="middle" class="sk-sub">설치할수록 자리를 차지</text>
  <path d="M492,142 L536,142" class="sk-line" marker-end="url(#az-m1)"/>
</svg>
<figcaption>플러그인은 기능 목록이 아니라 컨텍스트 예산을 쓰는 선택지다.</figcaption>
</figure>

일곱 개의 역할은 이렇습니다.

### azure (core)

리소스, 배포, 모니터링 등 기본 Azure 작업입니다. 어떤 전문 플러그인이 필요한지 추천해 주는 탐색 스킬이 들어 있어서, 처음에는 이것만 설치하고 추천을 받는 순서가 자연스럽습니다.

### aks-skills

AKS 클러스터 운영 쪽입니다. Pending 상태 파드 조사, 오류 시그니처에 맞는 문서화된 해결책 연결, GPU·KAITO 추론 문제 해결, 선택한 노드에서의 제한된 패킷 캡처, 노드 확장·워크로드 배치·스토리지 비용 효율 작업이 포함됩니다. 원문은 `azure`와 함께 설치하라고 합니다. 리소스 변경과 패킷 캡처는 명시적 승인이 필요합니다.

### azure-cost

비용 분석, 추정, 최적화, 거버넌스를 다루며 Azure Resource Manager MCP 서버를 기반으로 합니다. 청구 지출과 이상 징후 조사, AKS 비용 분석, 지출 예측, 계획한 리소스의 비용 추정, 약정 옵션 평가, 예산과 비용 가드레일 관리가 범위입니다.

### azure-advisor

Advisor 권고를 찾아 요약하고, 서비스 종료의 영향을 받는 리소스를 식별하며, 조치 안내를 준비합니다. Advisor에 구체적인 조치 방법이 없으면 공개된 Microsoft Learn 문서로 대체하고 출처를 밝힙니다.

### azure-kusto-graph-skills

Azure Data Explorer의 그래프 분석용입니다. KQL 그래프 연산자(구성, 패턴 매칭, 최단 경로, 연결 요소, 영구 그래프 모델)와 IRQL 보안 헌팅 파이프라인 작성, 쿼리 결과의 그래프 매핑·시각화 생성을 안내합니다.

### azure-local-skills

Azure Local의 계획, 배포, 일상 운영, 업데이트, 워크로드, 네트워킹, 문제 해결입니다. 표준 배포와 다중 랙 배포는 절차가 서로 바꿔 쓸 수 없어 스킬이 따로 있고, 다중 랙만 프리뷰입니다. 둘 다 읽기 전용으로 시작하며 업데이트, 재이미징, 네트워크 변경, VM 전원 작업 같은 중단성 작업 전에 확인을 받습니다.

### foundry-iq-skills

Azure AI Search 기반 Foundry IQ 지식 베이스를 만들고, 연결하고, 질의하고, 문제를 해결합니다. 로컬 파일, Azure Blob Storage, Azure Data Lake Storage에서 시작할 수 있고, 콘텐츠가 검색 가능한지 확인한 뒤 에이전트에 연결할 수 있습니다. 리소스 생성이나 에이전트 연결 전에는 승인을 요청합니다. 다만 일반 검색 인덱스 개발이나 범용 에이전트 생성용은 아닙니다.

## 이 발표를 어떻게 읽을까

### 플러그인 분리는 하네스 설계의 문제다

예전에 정리한 [AI 하네스의 4가지 축](/posts/ai-harness-four-parts/)에서 컨텍스트 관리를 모델 바깥 소프트웨어의 핵심으로 다뤘습니다. 이번 발표는 그 이야기를 제품 구성으로 보여 줍니다. 스킬을 많이 붙일수록 에이전트가 똑똑해지는 게 아니라, 매번 읽어야 할 설명이 늘어납니다. 그래서 "탐색 스킬이 필요한 플러그인을 추천하고 사용자가 고른다"는 구조가 나온 것으로 읽힙니다. 이 해석은 원문이 직접 말한 것이 아니라 제 추론입니다.

### 같은 형식, 다른 클라이언트

스킬의 형식이 한 회사 것이 아니게 된 흐름은 [Agent Skills의 AAIF 이관](/posts/agent-skills-aaif-donation/) 글에서 다뤘습니다. 이번 플러그인 마켓플레이스가 Copilot CLI, Claude Code, Cursor를 함께 지원한다는 점은 그 흐름과 맞닿아 있습니다. 같은 Azure 지식을 클라이언트마다 다시 만들 필요가 줄어든다는 뜻입니다. 다만 클라이언트별 동작 차이는 원문에 없습니다.

### 확인 절차는 있지만 균일하지 않다

원문이 승인 절차를 명시한 곳은 AKS, Azure Local, Foundry IQ입니다. Azure Cost, Advisor, Kusto Graph에 대해서는 확인 동작이 적혀 있지 않습니다. 비용 쪽은 주로 조회성 작업이라 그럴 수 있지만, 예산이나 가드레일 "관리"가 범위에 들어 있으니 쓰기 동작이 어디까지인지는 직접 확인해야 합니다.

## 실무에서 달라지는 점

### 1. 기존 `azure` 플러그인 사용자는 비용 질문부터 점검

이미 core `azure`를 설치해 둔 사람은 업데이트 후 비용 분석이 사라진 것처럼 보일 수 있습니다. 비용 분석 스킬이 `azure-cost`로 이동했으니 `/plugin install azure-cost@azure-skills`를 추가해야 합니다. 팀 공용 설정이나 사내 가이드에 "비용은 core에 있다"고 적어 둔 문서가 있다면 고쳐야 합니다.

### 2. 설치 기준은 "역할별 최소 구성"으로

운영자는 `azure`에 `aks-skills`나 `azure-advisor`, FinOps 담당은 `azure-cost`, 검색·RAG 담당은 `foundry-iq-skills`처럼 역할에 따라 나누는 편이 컨텍스트 면에서 유리합니다. 전부 설치한 표준 이미지를 배포하기보다, 처음에는 `azure`만 깔고 탐색 스킬의 추천을 받게 하는 안을 검토할 만합니다.

### 3. 권한은 플러그인이 아니라 계정에서 결정된다

원문은 플러그인별 필요 도구, 자격 증명, 권한이 README에 있다고만 합니다. 에이전트가 Azure 리소스를 건드릴 때의 실제 권한은 로그인한 계정의 RBAC에 달려 있을 가능성이 크므로, 도입 전에 읽기 전용 역할로 먼저 시험하고 쓰기 권한은 필요한 구독·리소스 그룹에만 주는 편이 안전합니다. 플러그인의 "승인 요청"은 사람이 보는 마지막 관문이지 권한 제어를 대신하지 않습니다.

### 4. Azure Local 다중 랙은 프리뷰라는 점을 표시

표준 배포와 다중 랙 배포는 절차가 호환되지 않고 다중 랙은 프리뷰입니다. 사내 문서나 교육 자료에 이 플러그인을 소개할 때 어느 배포 방식을 대상으로 하는지, 프리뷰 여부를 함께 적어야 오해가 없습니다. 나머지 플러그인은 GA 여부가 원문에 없으니 "GA"라고 단정하지 않는 편이 좋습니다.

### 5. 교육에서는 "유무 비교"를 강조

IT 강사라면 같은 질문을 플러그인이 있을 때와 없을 때 비교해 보여 주는 구성이 설득력이 있을 것입니다. 예를 들어 Pending 파드 조사나 서비스 종료 영향 리소스 찾기가 좋은 소재입니다. 이 글에서 효과가 어느 정도인지는 말할 수 없습니다. 원문에도 성능 수치는 없으니, 수업 전에 직접 돌려 보고 결과를 기록해야 합니다.

## 도입 전 체크리스트

- 사용하는 클라이언트가 `/plugin` 명령을 지원하는가
- 설치할 플러그인의 README에서 필요한 도구와 권한을 확인했는가
- 비용 분석을 쓰던 팀에 `azure-cost` 별도 설치를 알렸는가
- 쓰기 동작이 있는 플러그인은 읽기 전용 계정에서 먼저 확인했는가
- 프리뷰 항목(Azure Local 다중 랙)을 문서에 프리뷰로 표기했는가

## 참고

- [Explore specialized Azure plugins for your AI agent — Azure SDK Blog](https://devblogs.microsoft.com/azure-sdk/explore-specialized-azure-plugins-for-your-ai-agent/)
- [microsoft/azure-skills — GitHub](https://github.com/microsoft/azure-skills)
- 이전 글: [모델 바깥의 소프트웨어, AI 하네스를 설계하는 4가지 축](/posts/ai-harness-four-parts/)
- 이전 글: [Agent Skills, Linux Foundation 산하 AAIF로](/posts/agent-skills-aaif-donation/)
