---
title: "Fabric IQ, Copilot Chat·Cowork에서 GA — FabCon 발표를 상태별로 읽기"
date: 2026-10-26 06:53:00 +0900
categories: [IT News]
tags: [copilot, power-platform]
description: FabCon Europe 발표 중 Copilot Chat·Cowork의 Fabric IQ 근거 답변이 GA다. 프리뷰·예정 항목과 구분해, 관리자가 먼저 확인할 설정을 정리했다.
---

> 이 글은 Official Microsoft Blog의 "[New Microsoft data innovations unlock what only your business knows](https://blogs.microsoft.com/blog/2026/09/28/new-microsoft-data-innovations-unlock-what-only-your-business-knows/)"(2026-09-28, Jessica Hawk)를 읽고 정리·재구성한 것이다. 발표 후 4주가 지난 글이라 "지금 확인할 것" 위주로 썼다.
{: .prompt-info }

## 결론부터

발표 항목이 많지만 실무에서 바로 영향이 있는 것은 하나다. **Copilot Chat과 Cowork에서 Fabric IQ 기반 답변이 GA**이고, Fabric·Power BI 고객에게는 **기본으로 켜져 있다**. 나머지는 프리뷰이거나 "곧 프리뷰"이거나 상태 표기가 없다. 글이 가격과 수치를 전혀 제시하지 않으므로, 비용 영향은 이 글만으로 판단할 수 없다.

## 상태별로 나눠 보기

원문은 마케팅 톤으로 한 덩어리처럼 소개한다. 읽을 때는 상태부터 갈라 두는 편이 안전하다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="FabCon 발표 항목을 GA, 프리뷰, 예정, 상태 미표기로 나눈 표">
  <text x="0" y="14" class="sk-title">① 발표 항목의 상태 지도</text>
  <rect x="8" y="30" width="165" height="200" rx="8" class="sk-box-accent"/>
  <text x="90" y="52" text-anchor="middle" class="sk-label">GA</text>
  <text x="90" y="82" text-anchor="middle" class="sk-sub">Copilot Chat·Cowork의</text>
  <text x="90" y="98" text-anchor="middle" class="sk-sub">Fabric IQ 답변</text>
  <text x="90" y="128" text-anchor="middle" class="sk-sub">SQL Server on</text>
  <text x="90" y="144" text-anchor="middle" class="sk-sub">Azure Local</text>
  <rect x="190" y="30" width="165" height="200" rx="8" class="sk-box"/>
  <text x="272" y="52" text-anchor="middle" class="sk-label">프리뷰</text>
  <text x="272" y="82" text-anchor="middle" class="sk-sub">Database Hub (퍼블릭)</text>
  <text x="272" y="112" text-anchor="middle" class="sk-sub">IQ Sharing</text>
  <text x="272" y="142" text-anchor="middle" class="sk-sub">Azure Local</text>
  <text x="272" y="158" text-anchor="middle" class="sk-sub">연결 끊김 운영</text>
  <rect x="372" y="30" width="165" height="200" rx="8" class="sk-box-muted"/>
  <text x="454" y="52" text-anchor="middle" class="sk-label">예정</text>
  <text x="454" y="82" text-anchor="middle" class="sk-sub">SQL·PostgreSQL</text>
  <text x="454" y="98" text-anchor="middle" class="sk-sub">데이터베이스 에이전트</text>
  <text x="454" y="114" text-anchor="middle" class="sk-sub">(곧 퍼블릭 프리뷰)</text>
  <text x="454" y="144" text-anchor="middle" class="sk-sub">새 Code 경험 통합</text>
  <text x="454" y="160" text-anchor="middle" class="sk-sub">(Frontier 프로그램)</text>
  <rect x="554" y="30" width="158" height="200" rx="8" class="sk-box-muted"/>
  <text x="633" y="52" text-anchor="middle" class="sk-label">상태 미표기</text>
  <text x="633" y="82" text-anchor="middle" class="sk-sub">에이전트형</text>
  <text x="633" y="98" text-anchor="middle" class="sk-sub">데이터 엔지니어링</text>
  <text x="633" y="128" text-anchor="middle" class="sk-sub">Fabric Apps</text>
  <text x="633" y="158" text-anchor="middle" class="sk-sub">관측성(Observability)</text>
</svg>
<figcaption>도입 검토의 출발점은 "GA인가, 아닌가"이고, 이 발표에서 GA는 두 건뿐이다.</figcaption>
</figure>

### Fabric IQ 근거 답변 (GA)

Copilot Chat과 Cowork가 거버넌스가 적용된 Power BI 의미 모델, 지표, 관계, 비즈니스 정의를 근거로 답한다. 원문이 강조하는 효과는 "같은 지표는 누가 물어도 같은 숫자"다. 기존 접근 통제와 거버넌스는 유지된다고 한다. 새 Code 경험과의 통합은 Frontier 프로그램으로 예정이라 아직 쓸 수 없다.

### Database Hub와 데이터베이스 에이전트

Database Hub는 SQL Server, Azure SQL, PostgreSQL, Cosmos DB, Fabric의 SQL 데이터베이스를 한 곳에서 관리·감시하는 컨트롤 플레인으로, 퍼블릭 프리뷰다. SQL·PostgreSQL 전문 에이전트는 Database Hub와 VS Code에서 성능 병목 조사를 돕는 것으로 소개되며 "곧 퍼블릭 프리뷰" 단계다.

### 그 밖의 항목

IQ Sharing(프리뷰)은 데이터와 비즈니스 맥락을 외부 파트너와 안전하게 공유하는 기능이다. 에이전트형 데이터 엔지니어링은 엔지니어가 결과와 경계를 정하면 에이전트가 계획·실행·검증·개선을 하는 방식으로 소개되지만, 상태 표기도 구체 동작 설명도 없다.

## 실무에서 달라지는 점

### 1. M365·Power BI 관리자 — "기본으로 켜짐"부터 확인한다

Fabric·Power BI 고객에게 기본 활성이라는 것은, 따로 요청하지 않아도 Copilot 답변이 의미 모델을 근거로 쓸 수 있다는 뜻으로 읽힌다. 어떤 테넌트 설정이 이를 제어하는지는 원문에 없다. 관리 센터의 해당 설정 위치는 공식 문서에서 따로 찾아야 한다.

### 2. 데이터 소유자 — 의미 모델의 정의가 곧 답변의 품질이다

답변이 의미 모델의 지표와 비즈니스 정의를 근거로 한다면, 지표 이름이 모호하거나 정의가 부서마다 다른 모델은 그대로 모호한 답으로 이어질 것이다. 도입 전에 핵심 모델 몇 개의 지표 정의와 설명을 정리하는 것이 순서다. 이는 원문 문장에서 추론한 내 판단이다.

### 3. 권한 점검 — "기존 접근 통제가 적용된다"를 믿기 전에 확인한다

원문은 기존 통제가 유지된다고 말한다. 그래도 Copilot 답변에서 사용자가 볼 수 없어야 할 데이터가 나오지 않는지, 행 수준 보안을 쓰는 모델은 어떻게 동작하는지는 파일럿 계정으로 직접 확인해야 한다. 이 글은 직접 확인한 결과가 아니다.

### 4. 프리뷰 항목 — 운영 환경 도입 계획에 넣지 않는다

Database Hub, IQ Sharing, 연결 끊김 운영은 프리뷰다. 데이터베이스 에이전트는 아직 프리뷰조차 시작 전이다. 로드맵 표에 "검토 중"으로만 적어 두고 일정은 잡지 않는 편이 안전하다.

### 5. IT 강사 — 소개 수준 항목과 GA를 구분해 가르친다

에이전트형 데이터 엔지니어링은 방향성 소개일 뿐 상태와 동작이 공개되지 않았다. 수강생에게 "되는 것"과 "발표된 것"을 나눠 설명하고, 같은 구분을 라이선스 문의에도 적용하라고 안내한다. 가격이 원문에 없는 만큼 요금 질문에는 공식 가격 페이지를 확인하라고 답하는 것이 맞다.

## 이어 읽기

의미 있는 답변 근거를 어디에 두느냐는 문서 저장소 쪽에서도 같은 질문이다. SharePoint 쪽 사례는 [벡터 DB 없이 RAG — SharePoint Embedded Retrieval API와 spe-mcp](/posts/spe-retrieval-api-mcp-no-vector-db/)에서 다뤘다.

## 참고

- [New Microsoft data innovations unlock what only your business knows](https://blogs.microsoft.com/blog/2026/09/28/new-microsoft-data-innovations-unlock-what-only-your-business-knows/) — Official Microsoft Blog, 2026-09-28
