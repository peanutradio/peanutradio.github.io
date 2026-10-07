---
title: "MS가 사내 AI 전환에서 배운 5가지 — '도구 보급'은 전환이 아니다"
date: 2026-10-08 08:40:00 +0900
categories: [IT News]
tags: [microsoft, ai-transformation, copilot, ai-agent, frontier-company]
description: Microsoft가 자기 회사를 첫 고객(Customer Zero)으로 삼아 AI 전환을 하며 배운 다섯 가지 교훈을 정리했다. 영업·공급망·엔지니어링 사례와 수치, 고객에게 권하는 세 가지 패턴까지.
---

> 이 글은 Official Microsoft Blog의 "What we've learned from Microsoft's own AI transformation"(Kathleen Hogan, 2026-09-17)을 읽고 정리·재구성한 것입니다.
{: .prompt-info }

**결론부터.** Microsoft의 결론은 한 문장이다. "AI에 접근할 수 있느냐는 차별점이 되지 않는다." 차이를 만드는 건 사람을 참여시키고, 일하는 방식을 다시 설계하고, AI를 책임 있게 관리하는 조직의 능력이라는 것이다. 글을 쓴 사람은 Microsoft의 전략·전환 총괄(Chief Strategy and Transformation Officer) Kathleen Hogan이다. 자사를 '첫 고객'으로 삼아 수백 건의 내부 전환을 해 보고 나온 교훈이라 무게가 다르다.

## 처음엔 Microsoft도 틀렸다

흥미로운 고백부터 있다. Microsoft도 처음엔 AI를 여느 IT 도구처럼 '배포'했다. 라이선스를 나눠 주고 사용률을 올리는 식이다. 그리고 배운 건 **도구에 접근할 수 있다고 일이 바뀌지는 않는다**는 사실이었다.

## 다섯 가지 교훈

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="Microsoft의 AI 전환 다섯 가지 교훈 — 성과에서 시작, 워크플로 재설계, 직원 중심, 할 수 있는 일 확장, 지속 학습 루프">
  <defs>
    <marker id="mt-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 도구 보급 → 일의 재설계로</text>

  <rect x="8" y="50" width="130" height="80" rx="8" class="sk-box"/>
  <text x="73" y="80" text-anchor="middle" class="sk-label">1. 성과에서</text>
  <text x="73" y="98" text-anchor="middle" class="sk-label">시작</text>
  <text x="73" y="118" text-anchor="middle" class="sk-sub">기술이 아니라</text>

  <path d="M140,90 L150,90" class="sk-line" marker-end="url(#mt-m1)"/>
  <rect x="152" y="50" width="130" height="80" rx="8" class="sk-box"/>
  <text x="217" y="80" text-anchor="middle" class="sk-label">2. 워크플로</text>
  <text x="217" y="98" text-anchor="middle" class="sk-label">통째로 재설계</text>
  <text x="217" y="118" text-anchor="middle" class="sk-sub">개별 작업 말고</text>

  <path d="M284,90 L294,90" class="sk-line" marker-end="url(#mt-m1)"/>
  <rect x="296" y="50" width="130" height="80" rx="8" class="sk-box-accent"/>
  <text x="361" y="80" text-anchor="middle" class="sk-label">3. 직원이</text>
  <text x="361" y="98" text-anchor="middle" class="sk-label">중심</text>
  <text x="361" y="118" text-anchor="middle" class="sk-sub">팀 단위 학습</text>

  <path d="M428,90 L438,90" class="sk-line" marker-end="url(#mt-m1)"/>
  <rect x="440" y="50" width="130" height="80" rx="8" class="sk-box"/>
  <text x="505" y="80" text-anchor="middle" class="sk-label">4. 할 수 있는</text>
  <text x="505" y="98" text-anchor="middle" class="sk-label">일을 넓힘</text>
  <text x="505" y="118" text-anchor="middle" class="sk-mono">CI + AI = CA</text>

  <path d="M572,90 L582,90" class="sk-line" marker-end="url(#mt-m1)"/>
  <rect x="584" y="50" width="128" height="80" rx="8" class="sk-box"/>
  <text x="648" y="80" text-anchor="middle" class="sk-label">5. 함께 배우는</text>
  <text x="648" y="98" text-anchor="middle" class="sk-label">루프</text>
  <text x="648" y="118" text-anchor="middle" class="sk-sub">사람 ↔ AI</text>

  <path d="M648,132 C648,200 73,200 73,134" class="sk-line" stroke-dasharray="4 4" marker-end="url(#mt-m1)"/>
  <text x="360" y="225" text-anchor="middle" class="sk-sub">한 번의 도입이 아니라, 배운 것이 다시 다음 재설계로 돌아가는 순환</text>
</svg>
<figcaption>AI 전환의 단위는 '도구'가 아니라 '일하는 방식'이다.</figcaption>
</figure>

**1. 기술이 아니라 비즈니스 성과에서 시작한다.** 영업 조직 사례가 대표적이다. 사용률을 밀어붙이는 대신, 어카운트 매니저가 일주일을 어떻게 쓰는지부터 지도로 그렸다. 그리고 그 흐름에 맞춰 에이전트를 붙였다. 파이프라인 분석은 Analyst, 딜 패키지 작성은 Deal, 고객 인사이트는 Researcher가 맡았다. 결과는 어카운트 매니저 1인당 매출 9.4% 증가, 딜 성사율 20% 상승이다(2024년 1~6월, 영업 인력 687명 측정).

**2. 개별 작업이 아니라 워크플로 전체를 다시 설계한다.** 망가진 프로세스에 AI를 얹으면 효과가 작다. 클라우드 공급망 팀은 계획·조달·주문 처리·물류 전 과정을 먼저 단순하게 정리했다. 그다음 목적별 에이전트를 111개 이상 배치했다. 사이클 타임이 최대 75% 줄었고, 수요 계획 조사에 걸리던 시간이 5~7일에서 몇 시간으로 줄었다. 어떤 건은 20분 이내였다고 한다.

**3. 직원을 중심에 둔다.** 프로세스 어디가 막히는지, 어디에 판단이 필요한지는 현장 직원이 제일 잘 안다. Microsoft는 신입 엔지니어와 선배를 짝지어 AI와 함께 배우게 하는 PRAISE를 만들었다. 여러 주짜리 집중 프로그램 Camp AIR도 운영했다. Camp AIR는 엔지니어 3,000명 이상으로 커졌다. 여기서 얻은 교훈은 개인 교육보다 **팀 단위 학습**이 오래가는 변화를 만든다는 것이다.

**4. 효율이 아니라 '할 수 있는 일'을 넓힌다.** Microsoft는 이를 "CI + AI = CA"(Capability Add)라고 부른다. 사람의 강점과 AI의 강점을 합쳐, 전에는 현실적으로 못 하던 일을 하자는 것이다. Work Trend Index 조사에서 AI 사용자의 58%가 "전에 못 하던 일을 하게 됐다"고 답했다. 숙련 사용자는 80%였다. 엔지니어링 쪽에서는 9명짜리 팀이 Copilot Cowork를 첫 출시까지 35일 만에 내보냈다.

**5. 계속 배우는 루프를 만든다.** 사람은 피드백과 판단으로 AI를 더 쓸모 있게 만들고, AI는 사람이 더 빨리 배우게 돕는다. 이 양방향 순환에서 쌓인 조직 지식이 결국 경쟁력이 된다는 주장이다.

## 고객에게 권하는 세 가지 패턴

Microsoft는 수백 건의 내부 사례를 세 가지 '레시피'로 정리했다.

| 패턴 | 무엇을 바꾸나 | 위 사례로 보면 |
|---|---|---|
| Persona Acceleration | 특정 직무 한 명의 일 | 영업 어카운트 매니저 |
| AI-Powered Process Redesign | 하나의 워크플로 전체 | 클라우드 공급망 |
| AI-First Possibility | 처음부터 AI 전제로 새로 만드는 일 | 9명 팀의 35일 출시 |

이 경험을 고객에게 옮기려고 'Microsoft Frontier Company'라는 프로그램도 따로 만들었다.

## 에이전트를 만드는 입장에서 읽은 점

가장 와닿는 건 2번이다. 에이전트를 만들 때 흔히 "이 작업을 자동화하자"에서 출발한다. 하지만 Microsoft의 공급망 사례는 순서가 반대였다. 프로세스를 먼저 단순하게 만들고, 그 위에 에이전트를 100개 넘게 붙였다. 에이전트 수보다 '어디에 붙일지'를 정하는 설계가 성과를 갈랐다는 얘기다.

1번의 영업 사례도 같은 메시지다. 에이전트 이름(Analyst, Deal, Researcher)이 기능이 아니라 **사람의 일주일 흐름**에 맞춰 나뉘어 있다. Copilot Studio로 사내 에이전트를 기획할 때 바로 써먹을 만한 관점이다.

다만 수치는 모두 Microsoft 자체 측정이다. 특히 영업 성과는 2024년 상반기 687명 기준이라, 다른 회사에 그대로 옮겨 기대하기는 어렵다. 그래도 "도구 보급 ≠ 전환"이라는 결론은 AI를 도입하는 조직이라면 한 번쯤 짚어 볼 문장이다.

## 참고

- [What we've learned from Microsoft's own AI transformation — Official Microsoft Blog (2026-09-17)](https://blogs.microsoft.com/blog/2026/09/17/what-weve-learned-from-microsofts-own-ai-transformation/)
