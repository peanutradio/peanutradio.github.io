---
title: "Azure Repos Copilot 코드 리뷰 GA — 켜기 전에 볼 설정·과금 점검표"
date: 2026-11-21 06:53:00 +0900
categories: [IT News]
tags: [copilot, azure, devops, cost]
description: Azure Repos의 Copilot 코드 리뷰가 GA가 됐다. 활성화 위치, 자동 리뷰, 사용량 과금 구조를 정리하고 도입 전 확인할 점을 짚는다.
---

> 이 글은 Azure DevOps 블로그의 "Copilot Code Reviews for Azure Repos is generally available"(2026-11-20, Dan Hellem)을 읽고 정리·재구성한 것입니다. 해석과 점검 항목은 이 블로그의 의견입니다.
{: .prompt-info }

## 결론부터

Azure Repos의 Copilot 코드 리뷰가 정식 출시(GA)됐습니다. 6월 비공개 프리뷰, 8월 공개 프리뷰를 거쳤습니다. 기능 자체보다 눈여겨볼 곳은 **과금**입니다. 사용량 기반이고 청구가 Azure 구독으로 가기 때문에, 켜는 순간부터 비용이 생기는 기능입니다. 그래서 "켤 수 있나"보다 "어디까지 켜고 비용을 어떻게 지켜볼까"가 도입의 핵심 질문이 됩니다.

바로 전날 나온 [Azure DevOps의 Copilot 앱 연동(프리뷰)](/posts/azure-devops-copilot-app-preview-and-elm-ga/)이 "코딩 에이전트를 붙이는" 소식이었다면, 이번 글은 "PR 리뷰 단계에 Copilot을 넣는" 소식입니다. 둘 다 Azure DevOps 쪽이 GitHub Copilot 생태계로 이어지는 흐름이라는 점은 제 해석입니다.

## 원문이 말하는 것

### 켜는 곳과 범위

조직 관리자가 **Organization settings > Repos > Repositories**에서 활성화합니다. 전체 프로젝트·저장소에 한 번에 켜거나, 프로젝트·저장소 관리자가 각자 설정하도록 위임할 수 있습니다. 설정 변경은 Azure DevOps 감사 로그에 남아서 어디에 켜져 있고 누가 바꿨는지 추적됩니다.

### 리뷰를 조정하는 세 가지

커스텀 지침은 조직·프로젝트·저장소 단위로 둘 수 있고 skills를 참조할 수 있습니다. 리뷰 강도(effort level)는 프로젝트·저장소 기본값으로 Lite 또는 Balanced를 고르고, 리뷰마다 바꿀 수도 있습니다. 가벼운 리뷰는 credit을 덜 씁니다. 자동 리뷰는 대상 브랜치에 branch policy를 추가하면 draft를 포함한 새 PR마다 실행됩니다. 수동으로는 PR의 Reviewers 섹션에서 GitHub Copilot 옆 Request를 누릅니다.

### 리뷰 결과와 한계

Copilot은 문제가 의심되는 줄에 코멘트를 달고, 가능하면 한 번에 적용할 수 있는 수정 제안을 붙입니다. 제안을 적용·커밋한 뒤에는 관련 코멘트를 한꺼번에 해결 처리할 수 있습니다. 반대로 **PR을 승인하지도, 머지를 막지도 않습니다.** 승인 책임은 사람에게 남습니다.

### 과금

리뷰 한 번마다 토큰이 소비되고, 이를 GitHub AI credit으로 환산해 credit당 0.01달러로 계산합니다. 비용은 주로 PR 크기와 선택한 강도로 정해진다고 합니다. Azure Cost Management에는 별도 meter로 나타나고, 프로젝트 태그가 붙어 프로젝트별로 거르거나 묶을 수 있습니다. 예산 알림도 걸 수 있습니다. 저자는 1~2개 저장소로 시작해 사용량과 비용을 본 뒤 넓히라고 권합니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="Copilot 코드 리뷰가 PR에서 실행되고 비용이 Azure 구독으로 청구되는 흐름">
  <defs>
    <marker id="arc-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="arc-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 리뷰 한 번이 지나가는 길</text>
  <rect x="8" y="60" width="150" height="56" rx="8" class="sk-box"/>
  <text x="83" y="84" text-anchor="middle" class="sk-label">새 PR</text>
  <text x="83" y="102" text-anchor="middle" class="sk-sub">자동(정책) 또는 수동 요청</text>
  <rect x="200" y="60" width="150" height="56" rx="8" class="sk-box-accent"/>
  <text x="275" y="84" text-anchor="middle" class="sk-label">Copilot 리뷰</text>
  <text x="275" y="102" text-anchor="middle" class="sk-sub">Lite / Balanced</text>
  <rect x="392" y="60" width="150" height="56" rx="8" class="sk-box"/>
  <text x="467" y="84" text-anchor="middle" class="sk-label">코멘트·수정 제안</text>
  <text x="467" y="102" text-anchor="middle" class="sk-sub">승인·머지 차단은 안 함</text>
  <rect x="200" y="160" width="150" height="52" rx="8" class="sk-box-muted"/>
  <text x="275" y="182" text-anchor="middle" class="sk-label">토큰 → AI credit</text>
  <text x="275" y="200" text-anchor="middle" class="sk-sub">credit당 $0.01</text>
  <rect x="392" y="160" width="150" height="52" rx="8" class="sk-box"/>
  <text x="467" y="182" text-anchor="middle" class="sk-label">Azure 구독 청구</text>
  <text x="467" y="200" text-anchor="middle" class="sk-sub">Cost Management meter</text>
  <path d="M160,88 L198,88" class="sk-line" marker-end="url(#arc-m1)"/>
  <path d="M352,88 L390,88" class="sk-line" marker-end="url(#arc-m1)"/>
  <path d="M275,118 L275,158" class="sk-line-accent" marker-end="url(#arc-a1)"/>
  <path d="M352,186 L390,186" class="sk-line-accent" marker-end="url(#arc-a1)"/>
</svg>
<figcaption>리뷰는 PR마다 돌고, 그 횟수만큼 Azure 구독에 비용이 쌓인다.</figcaption>
</figure>

## 원문에 없는 것

글에 없는 내용도 분명히 해 둡니다. PR 한 건당 평균 비용, 월 무료 credit, Lite·Balanced 외 강도 이름, branch policy를 거는 구체적인 메뉴 경로는 원문에 나오지 않습니다. 지역·플랜 제한도 언급이 없습니다. 자세한 내용은 문서와 과금 FAQ 링크로 넘기고 있어서, 이 글에서는 그 부분을 추정하지 않았습니다.

## 실무에서 달라지는 점

### Azure 구독 소유자와 FinOps 담당: 청구 대상부터 확인

비용이 Azure DevOps 조직에 연결된 Azure 구독으로 청구됩니다. 개발팀이 켜기만 하면 구독 소유자는 모르는 채 지출이 생길 수 있습니다. 활성화 전에 어느 구독에 청구되는지 확인하고, Cost Management의 Copilot 코드 리뷰 meter에 예산 알림을 먼저 걸어 두는 순서가 안전합니다. credit 단가가 0.01달러이므로 100 credit이 1달러라는 계산까지만 원문으로 확정되고, 리뷰당 credit 수는 직접 관찰해야 알 수 있습니다.

### Azure DevOps 조직 관리자: 위임 범위를 먼저 정하기

전체 활성화와 프로젝트·저장소 관리자 위임 중 하나를 골라야 합니다. 위임하면 편하지만 비용이 흩어집니다. 처음에는 조직 관리자가 대상 저장소를 정하고, 프로젝트 태그 기준 비용 보고서가 나오는 걸 본 뒤 위임을 열어 주는 편이 관리하기 쉽습니다. 설정 변경이 감사 로그에 남으므로 누가 켰는지 확인하는 절차로도 쓸 수 있습니다.

### 자동 리뷰를 걸 팀: 시작은 Balanced보다 작게

자동 리뷰는 draft PR까지 대상이 됩니다. 푸시가 잦은 팀이라면 PR 수가 곧 비용입니다. 원문은 가벼운 리뷰가 credit을 덜 쓴다고만 말하므로, 기본값을 Lite로 두고 중요한 브랜치나 큰 PR에서만 강도를 올리는 운영을 먼저 검토해 볼 만합니다. 이 운영 방식은 제 제안이며 효과는 실제 비용 데이터로 확인해야 합니다.

### 리뷰 규칙 담당: 커스텀 지침이 품질을 좌우

코딩 표준을 지침으로 두면 리뷰가 팀 기준에 가까워집니다. 조직·프로젝트·저장소 세 단계로 둘 수 있으니, 공통 규칙은 조직에, 예외는 저장소에 두는 식으로 나누면 중복을 줄일 수 있습니다. 이미 GitHub 쪽에서 [Copilot 코드 리뷰 설정을 API로 관리하는 방법](/posts/copilot-code-review-api-management/)과 [해결률 지표](/posts/copilot-code-review-usage-metrics-addressed-rate/)를 다룬 적이 있는데, 같은 질문이 Azure Repos에서도 생길 겁니다. 다만 Azure Repos에서 같은 API·지표가 제공되는지는 원문에 없으므로 확인이 필요합니다.

### 교육할 때: 승인 책임은 사람에게

Copilot은 승인도 머지 차단도 하지 않습니다. 교육 자료에서는 "Copilot 코멘트가 없으면 통과"라는 오해를 먼저 막아야 합니다. 수정 제안을 한 번에 적용할 수 있다는 점도 양날입니다. 적용 전에 변경 의도를 읽는 습관을 같이 가르쳐야 리뷰 자동화가 품질 저하로 이어지지 않습니다.

## 참고

- [Copilot Code Reviews for Azure Repos is generally available — Azure DevOps Blog](https://devblogs.microsoft.com/devops/copilot-code-reviews-for-azure-repos-is-generally-available/)
- [Copilot Code Review 문서 (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/devops/repos/git/copilot-code-reviews?view=azure-devops)
- [과금 FAQ (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/devops/repos/git/copilot-code-reviews-faq?view=azure-devops)
- [프로젝트 단위 비용 보고 — Sprint 276 릴리스 노트](https://learn.microsoft.com/en-us/azure/devops/release-notes/2026/sprint-276-update#project-level-cost-reporting-for-copilot-code-reviews)
- [공개 로드맵](https://learn.microsoft.com/en-us/azure/devops/release-notes/features-timeline#copilot-code-reviews-for-azure-devops)
- [Azure DevOps integration with the GitHub Copilot app (public preview)](https://devblogs.microsoft.com/devops/azure-devops-integration-with-the-github-copilot-app-public-preview/)
