---
title: "Fabric IQ가 Copilot 안으로 — MS 9월 데이터 발표 정리"
date: 2026-10-06 13:30:00 +0900
categories: [IT News]
tags: [microsoft-fabric, copilot, power-bi, sql-server, ai-agent]
description: Microsoft가 9월 28일 데이터 신기능을 발표했다. Fabric IQ가 Copilot Chat·Cowork에서 GA됐고, Database Hub·Agentic Data Engineering·SQL Server on Azure Local 등이 함께 나왔다.
---

> 이 글은 Official Microsoft Blog의 "New Microsoft data innovations unlock what only your business knows"(Jessica Hawk, 2026-09-28)를 읽고 정리·재구성한 것입니다.
{: .prompt-info }

**결론부터.** 이번 발표의 중심은 "Copilot이 우리 회사 숫자를 우리 회사 정의대로 답하게 만드는 것"이다. Fabric IQ가 Microsoft 365 Copilot Chat과 Cowork에서 정식 출시(GA)됐다. 이제 Copilot의 답이 사내 보고서와 같은 Power BI 시맨틱 모델 위에서 나온다. 에이전트를 만드는 입장에서는 "데이터 근거를 어디에 두느냐"에 대한 MS의 답이 Fabric 쪽으로 정리되는 흐름이다.

## 발표 한눈에 보기

| 기능 | 상태 | 한 줄 요약 |
|---|---|---|
| Fabric IQ in Copilot Chat·Cowork | GA | Copilot 답변을 Power BI 시맨틱 모델과 비즈니스 정의에 근거하게 함 |
| IQ Sharing in Fabric | 프리뷰 | 데이터와 비즈니스 맥락을 조직 경계 밖으로 안전하게 공유 |
| Observability in Fabric | GA | 워크스페이스를 가로지르는 모니터링, Monitor Hub 강화 |
| Fabric Apps | 신규 기능 | 거버넌스가 걸린 데이터 위에서 바로 AI 앱 구축 |
| Agentic Data Engineering | 신규 기능 | 엔지니어가 결과와 경계를 정하면 에이전트가 실행 |
| SQL Server on Azure Local | GA | 로컬 환경에서 SQL Server와 Azure 서비스 실행 (연결 끊김 모드는 프리뷰) |
| Database Hub in Fabric | 퍼블릭 프리뷰 | SQL Server·Azure SQL·PostgreSQL·Cosmos DB를 한곳에서 관리 |

가격 정보와 고객 사례는 원문에 없었다.

## 가장 큰 변화: Fabric IQ가 Copilot의 '정답지'가 된다

<figure class="sketch">
<svg viewBox="0 0 720 240" role="img" aria-label="Fabric IQ 구조 — Copilot Chat과 Cowork의 질문이 Power BI 시맨틱 모델과 비즈니스 정의를 거쳐 OneLake 데이터로 답한다">
  <defs>
    <marker id="fq-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="fq-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① Copilot 답변이 '보고서와 같은 숫자'가 되는 경로</text>

  <rect x="8" y="80" width="170" height="70" rx="8" class="sk-box"/>
  <text x="93" y="108" text-anchor="middle" class="sk-label">Copilot Chat · Cowork</text>
  <text x="93" y="128" text-anchor="middle" class="sk-sub">"지난 분기 매출은?"</text>

  <path d="M180,115 L236,115" class="sk-line-accent" marker-end="url(#fq-a1)"/>

  <rect x="242" y="60" width="220" height="110" rx="10" class="sk-box-accent"/>
  <text x="352" y="88" text-anchor="middle" class="sk-label">Fabric IQ</text>
  <text x="352" y="112" text-anchor="middle" class="sk-sub">Power BI 시맨틱 모델</text>
  <text x="352" y="130" text-anchor="middle" class="sk-sub">비즈니스 정의 · 기존 권한 유지</text>
  <text x="352" y="152" text-anchor="middle" class="sk-mono">GA · 기본 켜짐</text>

  <path d="M464,115 L520,115" class="sk-line" marker-end="url(#fq-m1)"/>

  <rect x="526" y="80" width="186" height="70" rx="8" class="sk-box"/>
  <text x="619" y="108" text-anchor="middle" class="sk-label">OneLake 데이터</text>
  <text x="619" y="128" text-anchor="middle" class="sk-sub">미러링 · 바로가기</text>

  <text x="360" y="215" text-anchor="middle" class="sk-sub">같은 질문에 보고서와 Copilot이 다른 숫자를 내던 문제를 '의미 계층'에서 막는다</text>
</svg>
<figcaption>Copilot이 숫자를 새로 계산하지 않고, 회사가 이미 합의한 정의를 따라 답하게 된다.</figcaption>
</figure>

원문은 Copilot의 답이 "조직의 보고서를 움직이는 바로 그 시맨틱 모델"에 근거할 수 있게 됐다고 설명한다. Fabric과 Power BI 고객에게는 기본으로 켜져 있고, 기존 접근 권한도 그대로 지켜진다고 한다.

실무에서 이게 중요한 이유는 단순하다. 영업팀 보고서의 "매출"과 Copilot이 말하는 "매출"이 다르면 아무도 Copilot을 믿지 않는다. 지금까지는 이걸 맞추려고 RAG 문서나 프롬프트에 정의를 길게 적어 넣는 식으로 버텼다. 이제는 Power BI에 이미 정리해 둔 정의를 그대로 쓰는 길이 열렸다.

## 에이전트 관점에서 볼 것

**Agentic Data Engineering.** 엔지니어가 원하는 결과와 지켜야 할 경계를 정하면, 복잡한 데이터 엔지니어링 작업을 에이전트가 스스로 실행한다는 기능이다. 파이프라인을 한 줄씩 짜는 대신 "무엇을, 어디까지"를 정의하는 쪽으로 일이 바뀐다는 메시지다.

**Fabric Apps.** 데이터 연결, 트랜잭션 처리, 백엔드 로직, 정책 기반 접근 제어를 갖춘 상태로 AI 앱을 거버넌스 데이터 위에 바로 만든다. 데이터를 앱 쪽으로 복사해 가는 대신 앱이 데이터 옆으로 오는 구조다.

**Database Hub (퍼블릭 프리뷰).** SQL Server, Azure SQL, PostgreSQL, Cosmos DB를 한 화면에서 관리한다. 원문에는 데이터베이스 전문 에이전트(specialist database agents)가 "곧 퍼블릭 프리뷰로" 나온다는 예고도 있다. 아직 날짜는 없다.

## 온프레미스 쪽 소식

SQL Server on Azure Local이 GA됐다. 로컬 환경에서 SQL Server와 Azure 서비스를 함께 돌릴 수 있다. 네트워크 연결이 제한된 곳을 위한 '연결 끊김(disconnected) 모드'는 아직 프리뷰다. 망분리 요건이 있는 공공·금융 쪽에서 눈여겨볼 대목이다.

## 정리하며

이번 발표는 개별 기능보다 방향이 더 눈에 띈다. Copilot의 답, 에이전트의 작업, 앱의 데이터가 모두 Fabric의 같은 의미 계층 위로 모이고 있다. Copilot Studio로 사내 에이전트를 만드는 입장이라면, 앞으로는 "지식 소스를 어떻게 붙일까"보다 "Fabric에 정의가 잘 정리돼 있는가"가 먼저 물어야 할 질문이 될 것 같다.

## 참고

- [New Microsoft data innovations unlock what only your business knows — Official Microsoft Blog (2026-09-28)](https://blogs.microsoft.com/blog/2026/09/28/new-microsoft-data-innovations-unlock-what-only-your-business-knows/)
