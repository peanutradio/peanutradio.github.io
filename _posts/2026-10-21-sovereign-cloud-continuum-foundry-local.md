---
title: "Sovereign Cloud 연속선 — Foundry Local GA 예고와 상태별 정리"
date: 2026-10-21 06:51:00 +0900
categories: [IT News]
tags: [sovereign-cloud, foundry-local, azure-local, confidential-computing, ignite]
description: MS가 Sovereign Cloud를 통제 수준의 연속선으로 정리했다. 외부 키 관리 GA, Foundry Local GA 예고 등 항목을 상태별로 구분해 읽는다.
---

> 이 글은 Official Microsoft Blog의 "Advancing digital sovereignty with greater control and choice"(2026-10-20, Douglas Phillips)를 읽고 정리·재구성한 것입니다.
{: .prompt-info }

## 결론부터

이 발표에서 한국 IT 운영자에게 의미 있는 것은 세 가지입니다. **Azure Managed HSM 외부 키 관리가 GA**, **Azure Local의 기밀 VM과 SQL Server가 GA**, 그리고 **Foundry Local과 GitHub Enterprise Local이 Ignite에서 GA 예정**이라는 예고입니다. 반대로 발표에 나오는 항목 상당수는 프리뷰이거나 "Ignite에서 공개"라서 아직 쓸 수 있는 게 아닙니다. 그래서 이 글은 소식을 **상태별로 나눠** 읽습니다.

## 프레임: 워크로드마다 통제 수준을 고른다

원문은 Sovereign Public Cloud(Azure), Sovereign Private Cloud(Azure Local), 파트너 생태계를 하나의 "연속선"으로 묶고, 워크로드마다 필요한 통제 수준을 고르라고 말합니다. 전부 폐쇄망으로 가라는 얘기가 아니라는 점이 핵심입니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="공개 클라우드에서 프라이빗 클라우드로 통제 수준이 올라가는 연속선과 항목별 상태">
  <defs>
    <marker id="sov-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 통제 수준 연속선과 항목 상태</text>
  <path d="M20,60 L700,60" class="sk-line-accent" marker-end="url(#sov-m1)"/>
  <text x="20" y="48" class="sk-sub">낮음</text>
  <text x="660" y="48" class="sk-sub">높음</text>
  <rect x="8" y="80" width="220" height="120" rx="8" class="sk-box"/>
  <text x="118" y="104" text-anchor="middle" class="sk-label">Sovereign Public Cloud</text>
  <text x="118" y="130" text-anchor="middle" class="sk-sub">외부 키 관리 — GA</text>
  <text x="118" y="150" text-anchor="middle" class="sk-sub">Control Panel — Ignite 공개 프리뷰</text>
  <text x="118" y="170" text-anchor="middle" class="sk-sub">Clean Rooms — 제한 프리뷰</text>
  <rect x="250" y="80" width="220" height="120" rx="8" class="sk-box-accent"/>
  <text x="360" y="104" text-anchor="middle" class="sk-label">Sovereign Private Cloud</text>
  <text x="360" y="130" text-anchor="middle" class="sk-sub">기밀 VM · SQL Server — GA</text>
  <text x="360" y="150" text-anchor="middle" class="sk-sub">Foundry Local — Ignite GA 예정</text>
  <text x="360" y="170" text-anchor="middle" class="sk-sub">Microsoft 365 Local — 승인 고객 프리뷰</text>
  <rect x="492" y="80" width="220" height="120" rx="8" class="sk-box-muted"/>
  <text x="602" y="104" text-anchor="middle" class="sk-label">파트너 생태계</text>
  <text x="602" y="130" text-anchor="middle" class="sk-sub">Bleu Cloud(프랑스)</text>
  <text x="602" y="150" text-anchor="middle" class="sk-sub">Delos Cloud(독일)</text>
  <text x="602" y="170" text-anchor="middle" class="sk-sub">국내 직접 관련성 낮음</text>
</svg>
<figcaption>같은 발표 안에서도 "지금 쓸 수 있는 것"과 "예고된 것"이 섞여 있어, 상태를 구분하지 않으면 도입 계획이 틀어진다.</figcaption>
</figure>

## 상태별 정리 (원문 기준)

| 상태 | 항목 |
|---|---|
| GA | Azure Managed HSM 외부 키 관리, SQL Server on Azure Local, Azure Local 기밀 VM |
| Ignite에서 GA 예정 | Foundry Local, GitHub Enterprise Local |
| Ignite에서 공개 프리뷰 | Sovereign Control Panel |
| 프리뷰 | 연결-비연결 전환, DocumentDB·Blob Storage on Azure Local, Microsoft 365 Local(승인 고객) |
| 제한 프리뷰 | Azure Confidential Clean Rooms |
| EU·EFTA 한정 | Data Guardian, Production Touch Ledger |

가격, 대부분의 리전 가용성, 기능 세부는 글에 없습니다. 발표 성격이 강한 글이므로 수치 판단은 하지 않겠습니다.

## 내 해석

[Maia 글](/posts/azure-maia-multi-gigawatt/)이 "클라우드 안에서 추론 비용을 낮추는 쪽"이었다면, 이번 발표는 "추론을 어디에서 돌릴지 통제하는 쪽"입니다. Foundry Local이 연결·비연결 운영과 로컬 데이터 질의("에이전틱 검색")를 지원한다고 하니, 에이전트를 망 분리 환경에서 돌려야 하는 조직에는 후보가 생기는 셈입니다. 다만 GA는 아직 예고 단계이고, 모델 카탈로그가 실제로 얼마나 쓸 만한지는 확인된 바 없습니다. 모델 위험을 "어디에 어떻게 쓰나"로 보는 관점은 [오픈 웨이트 모델 위험 관리](/posts/open-weight-model-risk-deployment-context/) 글과 같이 읽으면 좋습니다.

## 실무에서 달라지는 점

- **보안·키 관리 담당자**: 외부 키 관리 GA로 루트 암호화 키를 직접 운영하는 하드웨어에 두는 선택지가 공식화됐습니다. 지원되는 Azure 서비스 범위와 운영 부담(키 보관 장애가 곧 서비스 장애)을 도입 전에 따져야 합니다.
- **온프레미스·망 분리 운영자**: Azure Local 환경이라면 기밀 VM과 SQL Server GA는 바로 검토 대상입니다. Foundry Local과 GitHub Enterprise Local은 Ignite(11월 17~20일) 이후 실제 GA 문서가 나오면 PoC를 잡는 순서가 안전합니다.
- **M365 관리자**: Microsoft 365 Local은 승인 고객 프리뷰라 일반 계획에 넣을 수 없습니다. 로드맵 참고용으로만 분류하세요.
- **IT 강사**: 교육 자료에서 "GA / 예정 / 프리뷰"를 색이나 표로 구분해 주세요. 수강생이 발표 문장을 현재 기능으로 오해하기 쉬운 유형의 글입니다.
- **무시해도 되는 것**: EU·EFTA 한정 항목과 프랑스·독일 파트너 클라우드는 한국 조직에 직접 관련이 적습니다.

## 참고

- [Advancing digital sovereignty with greater control and choice — Official Microsoft Blog (2026-10-20)](https://blogs.microsoft.com/blog/2026/10/20/advancing-digital-sovereignty-with-greater-control-and-choice/)
