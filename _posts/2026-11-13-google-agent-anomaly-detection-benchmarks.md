---
title: "Google 에이전트 이상 탐지, 87%의 함정 — 벤치마크 읽는 법"
date: 2026-11-13 06:53:00 +0900
categories: [IT News]
tags: [ai-agent, security, governance, observability]
description: Google Agent Anomaly Detection의 외부 벤치마크 결과(1,774세션)를 분모부터 뜯어 읽고, 같은 날 나온 MS·IDC 거버넌스 조사와 함께 운영자 점검 항목을 정리했다.
---

> 이 글은 Google Developers Blog의 Agent Anomaly Detection 벤치마크 글(2026-11-12)과 Microsoft 공식 블로그의 IDC 조사 소개 글(2026-11-12)을 읽고 정리·재구성한 것입니다. 해석과 실무 영향 부분은 제 의견입니다. 직접 실행해 본 내용은 없습니다.
{: .prompt-info }

## 결론부터

Google이 Private Preview로 내놓은 Agent Anomaly Detection(이하 AADS)이 외부 벤치마크 5종에서 "실행된 유해 지시의 87%를 플래그했다"고 합니다. 이 숫자만 보면 훌륭해 보이지만, 분모가 "에이전트가 실제로 따라 한 유해 지시"입니다. 유해 세션 전체를 기준으로 하면 45~62%입니다. 이 글의 가치는 87%라는 숫자보다 **약점을 본문에 같이 적었다는 점**에 있다고 봅니다. 사후 탐지 도구를 어디까지 믿고 어디서부터 사전 차단과 사람 승인으로 메워야 하는지 그림이 나오기 때문입니다.

같은 날 Microsoft는 IDC 조사로 "거버넌스를 갖춘 조직이 ROI도 높다"는 이야기를 내놨습니다. 두 글을 겹쳐 보면, 에이전트를 운영할 때 **관측(탐지)·통제(차단)·사람(스킬)** 이 각각 따로 비어 있다는 점이 보입니다.

## Google 글에서 확인된 것

### 무엇을 측정했나

AADS는 에이전트의 트레이스와 로그를 비동기로 읽어, 에이전트가 의도된 경계를 벗어나 행동한 세션을 표시합니다. 지연을 더하지 않지만 차단도 하지 않고 보고만 합니다. 탐지 결과는 Security Command Center에도 나타납니다. 노리는 위험은 OWASP Top 10 for Agentic Applications(2026) 1위인 에이전트 목표 하이재킹입니다.

평가는 Google이 만들지 않은 공개 벤치마크 5종(AgentDojo, InjecAgent, Agent Security Bench, AgentDyn, AgentHarm)을 수정 없이 돌려 총 1,774세션을 채점했습니다. 에이전트는 ADK로 만들고 모델은 gemini-2.5-flash였으며, 유해 여부는 모델이 아니라 벤치마크의 결정적 코드 검사(예: 돈이 계좌에서 나갔는가)가 판정했습니다.

### 숫자를 분모별로 읽기

- 실행된 유해 지시 중 플래그 비율: AgentDojo 85%, InjecAgent 87%, AgentDyn 99%, AgentHarm 81%(종합 87%)
- 유해 세션 전체 기준: 45~62%
- 에이전트가 읽지도 않은 지시가 담긴 141세션 중 플래그: 1건
- 에이전트가 따르지 않고 버틴 인젝션: 11~25%만 플래그
- 정상 세션 오탐: 인젝션 계열 2~4%, AgentHarm 16%
- 장기 메모리 중독: 18건 중 4건 탐지
- 신규 에이전트: 정상 세션이 약 220개일 때 33%, 360개일 때 99%(AgentDyn)

오탐이 AgentHarm에서 높은 이유는 게시·터미널 실행·파일 다운로드처럼 영향이 큰 도구를 정상 업무에서도 쓰기 때문이라고 설명합니다. 한계로 단일 모델과 시뮬레이션 앱 결과이고, 실제 트래픽에서는 오탐률이 다를 수 있다고 본문이 스스로 밝힙니다. 이미지로 실린 OWASP 범주별 표는 제가 확인하지 못했습니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="유해 지시가 읽힘, 버팀, 실행됨으로 갈리고 탐지율 분모가 실행된 것임을 보여주는 도식">
  <defs>
    <marker id="ga-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ga-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 87%의 분모는 어디인가</text>
  <rect x="8" y="90" width="150" height="56" rx="8" class="sk-box"/>
  <text x="83" y="114" text-anchor="middle" class="sk-label">유해 지시 세션</text>
  <text x="83" y="132" text-anchor="middle" class="sk-sub">전체 기준 45~62%</text>
  <path d="M160,118 L230,50" class="sk-line" marker-end="url(#ga-m1)"/>
  <path d="M160,118 L230,118" class="sk-line" marker-end="url(#ga-m1)"/>
  <path d="M160,118 L230,196" class="sk-line-accent" marker-end="url(#ga-a2)"/>
  <rect x="232" y="26" width="200" height="48" rx="8" class="sk-box-muted"/>
  <text x="332" y="48" text-anchor="middle" class="sk-label">읽지도 않음</text>
  <text x="332" y="64" text-anchor="middle" class="sk-sub">141세션 중 1건 플래그</text>
  <rect x="232" y="94" width="200" height="48" rx="8" class="sk-box-muted"/>
  <text x="332" y="116" text-anchor="middle" class="sk-label">읽었지만 버팀</text>
  <text x="332" y="132" text-anchor="middle" class="sk-sub">11~25%만 플래그</text>
  <rect x="232" y="172" width="200" height="48" rx="8" class="sk-box-accent"/>
  <text x="332" y="194" text-anchor="middle" class="sk-label">실제로 실행됨</text>
  <text x="332" y="210" text-anchor="middle" class="sk-sub">여기가 87%의 분모</text>
  <path d="M434,196 L490,196" class="sk-line-accent" marker-end="url(#ga-a2)"/>
  <rect x="492" y="160" width="220" height="72" rx="8" class="sk-box"/>
  <text x="602" y="186" text-anchor="middle" class="sk-label">사후 탐지 + 사람 triage</text>
  <text x="602" y="206" text-anchor="middle" class="sk-sub">차단은 다른 계층의 일</text>
</svg>
<figcaption>87%는 "이미 실행된 사고" 중 잡은 비율이라서, 막는 도구가 아니라 놓친 것을 찾는 도구로 읽어야 한다.</figcaption>
</figure>

### Google이 권하는 조합

본문은 탐지 하나에 의존하지 말고 겹쳐 쓰라고 합니다. 프롬프트·응답 내용까지 포함한 ADK 트레이싱·로깅 켜기, Model Armor 같은 필터로 신뢰할 수 없는 입력 거르기, Semantic Governance Policy(Preview)로 도구 호출 차단, 결제·외부 메일·데이터 공유는 사람 승인, 도구 권한 최소화입니다. 상태는 Private Preview이고 접근 신청은 Google Cloud 문서의 개요 페이지를 따라야 합니다. 가격이나 설정 절차는 글에 없습니다.

## 같은 날 MS 글과 겹쳐 보기

Microsoft 공식 블로그(Alysa Taylor)는 Microsoft 후원 IDC 조사(2026년 7월 설문, N=5,204)를 소개했습니다. 신제품 발표는 없습니다. 주요 수치는 다음과 같습니다.

- AI 평균 ROI: Frontier Firms(AI 성숙도 상위 10%) 4배, 느린 도입자(하위 30%) 2.7배
- 에이전트 거버넌스 프로세스: 전체 65%, Frontier Firms 87%, 느린 도입자 51%
- 조직 전체 AI 스킬링 전략: 전체 20%, Frontier Firms 80%, 느린 도입자 3%
- 에이전트 대규모 롤아웃 96%, 에이전트 오케스트레이션 53%(모두 Frontier Firms 기준)

글 스스로 이 결과가 인과가 아닌 연관을 보여줄 뿐이고, ROI는 리더가 직접 보고한 값이라고 적었습니다. 후원 조사이니 "거버넌스를 하면 ROI가 4배가 된다"로 읽으면 안 됩니다. 다만 거버넌스를 가진 조직이 에이전트를 더 크게 굴린다는 방향성은 AADS 글의 "탐지는 통제의 마지막 층일 뿐"이라는 메시지와 맞닿아 있습니다. 전체 InfoBrief는 추후 공개이고, IDC의 Shari Lava가 11/18 Ignite에서 세션 3개를 진행합니다.

같은 날 올라온 Frontier Company 고객 사례 글은 홍보 성격이 강해 이 글에서는 다루지 않았습니다.

## 실무에서 달라지는 점

### 1. 탐지 도구를 도입 검토할 때 분모부터 묻기

벤더 숫자를 받으면 "실행된 것 중인지, 시도 전체 중인지"를 먼저 확인하는 게 좋겠습니다. AADS 글은 둘을 다 공개해서 비교가 가능하지만, 다른 제품 자료는 한 가지만 적는 경우가 많습니다. 구매·PoC 체크리스트에 분모 항목을 넣을 만합니다.

### 2. 로그 설계가 탐지의 전제다

AADS는 트레이스와 로그를 읽습니다. 프롬프트와 응답 내용이 빠진 로그라면 쓸 수 없습니다. 반대로 그 내용이 개인정보·기밀을 포함하므로 보관 기간, 접근 권한, 마스킹 정책을 도입 전에 보안팀과 정해야 합니다. Azure나 M365 환경에서 비슷한 도구를 검토할 때도 같은 질문이 따라옵니다.

### 3. 신규 에이전트는 첫 주에 탐지가 약하다고 가정하기

정상 세션 약 220개에서 33%, 360개에서 99%라는 격차는 새로 배포한 에이전트가 학습 기간 동안 사실상 보호받지 못한다는 뜻입니다. 출시 초기에는 사람 승인 범위를 넓히고, 승인을 줄이는 시점을 세션 수 같은 기준으로 정하는 편이 안전합니다. 같은 맥락의 승인 완화 설계는 [Copilot 보조 권한 GA](/posts/copilot-assisted-permissions-ga/) 글에서 다뤘습니다.

### 4. 오탐은 업무 성격에 따라 달라진다

도구 영향이 큰 에이전트(게시, 터미널, 다운로드)는 오탐이 16%까지 올라갔습니다. 본문 권고도 처음에는 결과가 많이 나오니 심각도와 위협 범주별로 분류하라는 것입니다. 인시던트 팀에 Critical·High만 보내는 라우팅을 먼저 만들어 두지 않으면 알림 피로가 생길 수 있습니다.

### 5. 교육할 때는 "탐지 ≠ 방어"를 강조하기

강의나 사내 교육에서 에이전트 보안을 설명한다면, 입력 필터·도구 호출 정책·사람 승인 같은 사전 계층과 사후 탐지를 구분해서 그려 주는 것이 좋겠습니다. 메모리 중독처럼 아직 약한 영역이 있다는 점도 같이 알려야 합니다. 자체 진화 에이전트를 서명·정책·사람 승인으로 묶는 구조는 [Chrysalis 글](/posts/governed-self-evolving-agent-chrysalis/)과 이어 읽을 수 있습니다.

## 남은 질문

AADS는 아직 Private Preview이고 GA 일정, 가격, 지원 범위는 공개되지 않았습니다. Microsoft 쪽은 Ignite(11/17~20)에서 에이전트 거버넌스 관련 발표가 있을지 지켜볼 만합니다. 다만 그건 아직 열리지 않은 행사라 예측으로만 남겨 두겠습니다.

## 참고

- [How Agent Anomaly Detection scores on benchmarks we didn't write — Google Developers Blog](https://developers.googleblog.com/how-agent-anomaly-detection-scores-on-benchmarks-we-didnt-write/)
- [Agent Anomaly Detection 개요 — Google Cloud 문서](https://docs.cloud.google.com/gemini-enterprise-agent-platform/agent-anomalies-overview)
- [Scaling AI value without losing control: Lessons from Frontier Firms — The Official Microsoft Blog](https://blogs.microsoft.com/blog/2026/11/12/scaling-ai-value-without-losing-control-lessons-from-frontier-firms/)
- [OWASP Top 10 for Agentic Applications for 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
