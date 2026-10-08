---
title: "Azure Functions Dynamic Workflows — 모델은 계획만, 실행은 Durable이"
date: 2026-10-09 06:49:00 +0900
categories: [AI Agent]
tags: [azure-functions, durable-functions, ai-agent, orchestration, hosted-skills]
description: Azure Functions Hosted Skills의 Dynamic Workflows는 LLM이 DAG 계획을 한 번 쓰고 Durable Functions가 추론 루프 밖에서 실행해 토큰과 지연을 줄인다.
---

> 이 글은 Microsoft Azure SDK 블로그의 Dynamic Workflows 발표를 읽고 정리·재구성한 것입니다. 수치는 모두 원문 저자의 자체 측정이며, 기능은 퍼블릭 프리뷰입니다.
{: .prompt-info }

## 결론부터

에이전트가 도구를 열 번 부르면 결과가 열 번 모델로 돌아가 컨텍스트가 불어납니다. Dynamic Workflows는 이걸 뒤집습니다. **LLM은 작업 계획을 DAG로 한 번만 쓰고, 실행은 내장 Durable Functions 엔진이 추론 루프 밖에서 맡습니다.** 중간 도구 결과가 모델로 돌아가지 않으니 토큰과 지연이 크게 줄어듭니다. 이름이 비슷한 10/2의 "Copilot Dynamic workflows"와는 다른 기능으로, Azure Functions에서 직접 에이전트를 만드는 쪽의 이야기입니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="일반 추론 루프는 모델과 도구를 왕복하지만, Dynamic Workflows는 모델이 계획을 한 번 쓰고 Durable 엔진이 실행한다">
  <defs>
    <marker id="dw-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="dw-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 도구 결과가 모델로 돌아오느냐의 차이</text>
  <text x="8" y="52" class="sk-sub">일반 추론 루프</text>
  <rect x="8" y="62" width="120" height="44" rx="8" class="sk-box"/>
  <text x="68" y="89" text-anchor="middle" class="sk-label">LLM</text>
  <rect x="300" y="62" width="120" height="44" rx="8" class="sk-box-muted"/>
  <text x="360" y="89" text-anchor="middle" class="sk-label">도구 ×N</text>
  <path d="M130,76 L298,76" class="sk-line" marker-end="url(#dw-m1)"/>
  <path d="M298,94 L130,94" class="sk-line" marker-end="url(#dw-m1)"/>
  <text x="440" y="89" class="sk-sub">호출 때마다 결과가 컨텍스트에 쌓임</text>
  <text x="8" y="150" class="sk-sub">Dynamic Workflows</text>
  <rect x="8" y="160" width="120" height="44" rx="8" class="sk-box"/>
  <text x="68" y="187" text-anchor="middle" class="sk-label">LLM</text>
  <rect x="200" y="160" width="150" height="44" rx="8" class="sk-box-accent"/>
  <text x="275" y="187" text-anchor="middle" class="sk-label">DAG 계획 1회</text>
  <rect x="430" y="160" width="170" height="44" rx="8" class="sk-box-accent"/>
  <text x="515" y="187" text-anchor="middle" class="sk-label">Durable 엔진 실행</text>
  <path d="M130,182 L198,182" class="sk-line-accent" marker-end="url(#dw-a2)"/>
  <path d="M352,182 L428,182" class="sk-line-accent" marker-end="url(#dw-a2)"/>
  <text x="515" y="228" text-anchor="middle" class="sk-sub">중간 결과는 모델로 안 돌아감</text>
</svg>
<figcaption>모델의 일은 "계획 쓰기"로 줄고, 반복 실행은 엔진이 가져간다.</figcaption>
</figure>

## 어떻게 켜고 쓰나

- `.agent.md` 머리말에 `workflows: enabled: true`를 넣으면 켜집니다.
- 도구는 `@workflow_tool` 데코레이터를 붙인 Python 함수입니다. dict 인자 하나를 받고 JSON 직렬화 가능한 값을 돌려줘야 합니다.
- LLM에게는 `start_workflow`, `get_workflow_status`, `list_workflows`, `cancel_workflow`, `terminate_workflow` 같은 관리 도구가 주어집니다. `start_workflow`는 ID만 돌려주는 fire-and-forget입니다.
- DAG 요소는 `depends_on`(의존 없는 작업은 병렬), `for_each`, `when`, 재시도·타임아웃, 서브 에이전트, 최대 24시간 Durable 타이머입니다.
- 설치는 Copilot의 Customize > Plugins에서 `azure-functions-skills` 플러그인을 쓰는 방법(권장)이나, `azurefunctions-agents-runtime` 패키지로 직접 구성하는 방법이 있습니다. Python 3.13 이상, Functions Core Tools v4, 로컬 실행용 Azurite가 필요합니다.

## 숫자는 이렇게 나왔다

원문 벤치마크는 서비스 N개를 점검해 보고서를 쓰는 합성 시나리오이고, 모델은 Foundry의 gpt-5.4-mini입니다. 세 번 실행의 중앙값입니다.

| 서비스 수 | 토큰(일반 → 워크플로) | 지연(일반 → 워크플로) |
|---|---|---|
| 1 | 12,744 → 5,609 | 38.6초 → 7.1초 |
| 10 | 99,097 → 6,781 (93.1% 감소) | 247.1초 → 12.7초 (95.1% 감소) |

일반 방식은 서비스가 하나 늘 때마다 토큰이 약 1만씩 늘지만, 워크플로 방식은 1개에서 10개로 가도 1,172 늘었을 뿐입니다. 결정적 검사에서 24개 출력이 모두 기대 보고서와 일치했습니다.

## 쓰기 전에 알아둘 제약

- **계획은 고정**: 실행 중 DAG를 바꿀 수 없고, 사용자에게 되묻는 것도 안 됩니다.
- **한도**: 계획당 노드 50개, 병렬 10개, 세션당 활성 워크플로 10개.
- **MCP 도구는 직접 못 씀**: Python `@workflow_tool`로 감싸야 합니다.
- **최소 1회 실행**: 같은 작업이 반복될 수 있어, 원문은 시도 간에 유지되는 `idempotency_key`로 부수 효과를 한 번만 일어나게 하라고 안내합니다.
- 서브 에이전트는 한 단계만 가능하고, 결과는 작게 유지해야 합니다.

## 내 생각

수치가 인상적이지만 구조화된 보고서 작업에 한정된 측정이고, 원문도 자유 형식 작업에서는 다를 수 있다고 인정합니다. 그래서 "모든 에이전트에 쓰라"가 아니라, **도구 호출이 많고 순서를 미리 정할 수 있는 작업**에 후보로 두는 게 맞아 보입니다. 퍼블릭 프리뷰라 API가 바뀔 수 있다는 점도 감안해야 합니다.

## 참고

- [Dynamic Workflows in Azure Functions Hosted Skills — Azure SDK Blog](https://devblogs.microsoft.com/azure-sdk/dynamic-workflows-azure-functions-hosted-skills/)
