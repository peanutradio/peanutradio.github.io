---
title: "GPT-6 Astra·Sol·Luna, Foundry에서 단가와 배포 옵션 정리"
date: 2026-10-25 06:49:00 +0900
categories: [IT News]
tags: [gpt-6, foundry, azure-openai, pricing, model-selection]
description: Microsoft Foundry에 올라온 GPT-6 Astra, Sol, Luna의 역할과 100만 토큰당 단가, 배포 옵션을 정리하고 모델을 나눠 쓰는 기준을 해석했다.
---

> 이 글은 Microsoft Azure 블로그의 "GPT-6 Astra, Sol, and Luna for production agents in Microsoft Foundry"(9/22)를 읽고 정리·재구성한 것입니다. 해석 부분은 제 의견이며 직접 배포해 본 기록이 아닙니다.
{: .prompt-info }

**결론부터.** GPT-6는 Astra(어려운 작업), Sol(범용), Luna(대량 처리) 세 단계로 나뉘고, 단가 차이는 한 단계마다 5배, 20배 수준입니다. 그래서 "가장 좋은 모델 하나"를 고르는 글이 아니라, 에이전트의 단계별로 모델을 나누는 글로 읽는 게 맞습니다. 가격은 바뀔 수 있으니 도입 전에 Azure OpenAI 가격 페이지로 다시 확인하세요.

## 세 모델의 역할

원문이 설명하는 역할은 이렇습니다.

- **Astra**: 고급 추론, 소프트웨어 엔지니어링, 컴퓨터 사용 같은 어려운 작업용. 원문은 어려운 작업이면 Astra부터 시작하라고 권합니다.
- **Sol**: 엔터프라이즈 에이전트, 코딩, 복합 지식 작업, 도구를 쓰는 워크플로를 위한 범용 모델. 원문은 신규 프로덕션 작업이나 레거시 모델 이전의 출발점으로 제시합니다.
- **Luna**: Sol보다 작고 빠른 모델. 추출, 요약, 요청 라우팅, 일상 응대처럼 물량이 많은 작업용.

## 100만 토큰당 단가 (Global Standard, 단기 컨텍스트)

| 모델 | 입력 | 캐시된 입력 | 출력 |
|---|---|---|---|
| Astra | $10 | $1 | $50 |
| Sol | $2 | $0.20 | $10 |
| Luna | $0.10 | $0.01 | $0.50 |

장기 컨텍스트는 단가가 더 높습니다. 예를 들어 Sol은 입력 $4, 출력 $15입니다. 다만 원문에는 단기와 장기를 가르는 토큰 수 기준이 없습니다. 캐시된 입력은 표준 입력의 10분의 1입니다.

Data Zone은 지역 처리 요건을 위한 옵션입니다. 원문 각주는 US Data Zone이 Global보다 10% 비싸다고 하는데, 표를 보면 EU Data Zone은 Astra·Sol 기준 20% 비쌉니다(Sol 입력 $2.40). 원문이 이 차이를 설명하지 않으니, EU를 고려한다면 표를 직접 보세요.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="Astra, Sol, Luna의 출력 토큰 단가를 막대 길이로 비교한 스케치">
  <defs>
    <marker id="gpt6-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 출력 토큰 단가 비교 (100만 토큰, Global 단기)</text>
  <text x="8" y="68" class="sk-label">Astra</text>
  <rect x="80" y="48" width="600" height="28" rx="6" class="sk-box-accent"/>
  <text x="90" y="67" class="sk-mono">$50</text>
  <text x="8" y="116" class="sk-label">Sol</text>
  <rect x="80" y="96" width="120" height="28" rx="6" class="sk-box"/>
  <text x="210" y="115" class="sk-mono">$10</text>
  <text x="8" y="164" class="sk-label">Luna</text>
  <rect x="80" y="144" width="6" height="28" rx="2" class="sk-box-muted"/>
  <text x="96" y="163" class="sk-mono">$0.50</text>
  <text x="80" y="204" class="sk-sub">막대 길이는 단가에 비례 (Astra = 600)</text>
</svg>
<figcaption>Astra 한 번 호출할 비용이면 Sol은 5번, Luna는 100번 부를 수 있다.</figcaption>
</figure>

## 배포 옵션

- **Standard**: Astra, Sol, Luna 모두. Global 28개 리전과 US·EU Data Zone.
- **Provisioned Throughput**: Astra와 Sol. 예약 용량과 낮은 지연이 필요한 핵심 서비스용.
- **Priority Processing**: Sol만. 응답성이 중요한 사용자 화면용 종량제 우선 처리 경로.

Provisioned와 Priority의 요금은 배포 유형별로 달라서 원문도 가격 페이지를 가리킵니다.

## 실무에서 달라지는 점

1. **에이전트 설계자**: 모델을 에이전트 단위가 아니라 단계 단위로 정하세요. 원문도 Luna는 대량 단계, Sol은 범용, Astra는 어려운 추론에 쓰라고 합니다. "라우터는 Luna, 본 작업은 Sol" 같은 구체적 조합은 제 해석이고 원문의 예시가 아닙니다.
2. **비용 담당자**: 토큰 단가가 아니라 작업당 비용으로 비교하세요. 원문의 권고입니다. 단가가 5배 비싸도 재시도가 줄면 싸질 수 있고, 반대도 가능합니다. 다만 원문에 작업당 비용 수치는 없습니다. 이 수치는 직접 평가를 돌려야 나옵니다. 캐시 입력이 10분의 1이니 시스템 프롬프트가 긴 에이전트는 [프롬프트 캐싱](/posts/azure-openai-cost-guide/)부터 확인할 만합니다.
3. **M365·Azure 운영자**: 데이터 처리 위치 요건이 있으면 Data Zone을 쓰되 할증(US 10%, EU는 표 기준 20%)을 예산에 넣으세요. 한국 리전 제공 여부는 원문에서 확인되지 않습니다.
4. **레거시 모델 사용 조직**: 원문은 GPT-5.6 Sol 이상으로의 평가를 권합니다. 이전 일정을 잡기 전에 평가셋부터 만드는 편이 안전합니다.
5. **강사 관점**: 교육에서는 "큰 모델이 정답"이 아니라 "평가로 고른다"를 강조하겠습니다. 성능 비교 차트는 원문에서 이미지라 수치를 확인할 수 없고, 성능 주장은 벤더 자료입니다.

## 확인 못 한 것

컨텍스트 윈도 크기, 장기 컨텍스트 구분 기준, 한국 리전, 작업당 비용 실측은 원문에 없습니다. 이 글의 수치는 모두 원문 표 기준입니다.

관련해서 Foundry의 에이전트 도구 변화는 [Foundry 9월 업데이트 정리](/posts/foundry-sept-2026-two-agent-changelog/)에서 다뤘습니다.

## 참고

- [GPT-6 Astra, Sol, and Luna for production agents in Microsoft Foundry — Azure Blog](https://azure.microsoft.com/en-us/blog/gpt-6-astra-sol-and-luna-for-production-agents-in-microsoft-foundry/)
