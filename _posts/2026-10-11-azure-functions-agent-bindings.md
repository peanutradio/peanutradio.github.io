---
title: "Azure Functions Agent bindings — 코드가 주도하고 추론은 한 단계만"
date: 2026-10-11 06:53:00 +0900
categories: [AI Agent]
tags: [azure-functions, ai-agent, agent-framework, durable-functions, python]
description: Azure Functions Agent bindings(프리뷰)는 기존 Python 함수에 경계가 있는 추론 한 단계를 끼운다. Dynamic Workflows와 반대 방향의 설계를 비교한다.
---

> 이 글은 Azure SDK 블로그의 "Azure Functions Agent bindings (preview)" 발표(2026-10-06, Victoria Hall)를 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록이 아니라 원문 기반 소개입니다.
{: .prompt-info }

## 결론부터

Agent bindings는 "에이전트 앱을 새로 만든다"가 아니라 **이미 돌고 있는 함수 안에 추론 한 단계를 끼워 넣는다**는 접근이다. 트리거, 입력 검증, 어떤 데이터를 모델에 보낼지, 언제 실행할지, 응답을 어떻게 쓸지는 전부 내 코드가 쥔다. 모델은 해석·분류·요약 같은 일만 한다. [며칠 전 정리한 Dynamic Workflows](/posts/azure-functions-dynamic-workflows/)가 "모델이 계획하고 Durable이 실행"하는 방향이었다면, 이쪽은 정반대로 **코드가 흐름을 쥐고 모델은 부품**이다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="트리거에서 검증, 최소화, 에이전트 한 단계, 응답 처리로 이어지는 흐름">
  <defs>
    <marker id="afb-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 코드가 흐름을 쥐고, 모델은 한 칸만</text>
  <rect x="8" y="60" width="120" height="60" rx="8" class="sk-box"/>
  <text x="68" y="86" text-anchor="middle" class="sk-label">트리거</text>
  <text x="68" y="104" text-anchor="middle" class="sk-sub">HTTP·큐·타이머</text>
  <rect x="160" y="60" width="140" height="60" rx="8" class="sk-box"/>
  <text x="230" y="86" text-anchor="middle" class="sk-label">검증·권한·최소화</text>
  <text x="230" y="104" text-anchor="middle" class="sk-sub">결정적 코드</text>
  <rect x="332" y="60" width="140" height="60" rx="8" class="sk-box-accent"/>
  <text x="402" y="86" text-anchor="middle" class="sk-label">agent.run()</text>
  <text x="402" y="104" text-anchor="middle" class="sk-sub">추론 한 단계</text>
  <rect x="504" y="60" width="140" height="60" rx="8" class="sk-box"/>
  <text x="574" y="86" text-anchor="middle" class="sk-label">응답 처리</text>
  <text x="574" y="104" text-anchor="middle" class="sk-sub">검사·버리기·반환</text>
  <path d="M130,90 L158,90" class="sk-line" marker-end="url(#afb-m1)"/>
  <path d="M302,90 L330,90" class="sk-line" marker-end="url(#afb-m1)"/>
  <path d="M474,90 L502,90" class="sk-line" marker-end="url(#afb-m1)"/>
  <text x="0" y="170" class="sk-sub">핸들러가 모델 호출 여부와 시점, 결과 처리를 직접 결정한다</text>
</svg>
<figcaption>추론 단계가 코드 사이에 끼어 있으니 따로 테스트하거나 조건부로 건너뛸 수 있다.</figcaption>
</figure>

## 원문이 말하는 구조

- `<name>.agent.md` 마크다운 파일이 지침이 되어 Microsoft Agent Framework의 `Agent`가 만들어지고 핸들러에 주입된다. 핸들러는 `@app.markdown_agent(arg_name=..., agent_name=...)`를 달고 `await agent.run(...)`을 직접 부른다. `agent_name`은 파일명(확장자 앞부분)과, `arg_name`은 핸들러 인자 이름과 일치해야 한다.
- 지침 파일의 YAML 머리말은 파싱되지 않고 통째로 모델에 전달된다. 모델 선택은 클라이언트 팩토리에서, 파이썬 도구는 명시적으로 설정한다. 선택적으로 `skills/` 폴더와 `mcp.json`(원격 MCP 서버, 도구 허용 목록)을 붙일 수 있다.
- Durable 모드에서는 오케스트레이터가 `context.call_agent(name, input)`을 부른다. 모델 호출은 확장이 관리하는 액티비티로 실행되고 결과가 오케스트레이션 기록에 저장되어, 재생(replay) 때 모델을 다시 부르지 않는다. HTTP 시작점은 202와 상태 조회 URL을 돌려준다.
- 요구 사항은 Python 3.13 이상(v2 모델), Core Tools, Azurite, Azure CLI, 그리고 배포된 모델이 있는 Foundry 프로젝트다. 인증은 `DefaultAzureCredential`을 쓴다.

## Dynamic Workflows와 나란히 놓으면

| | Agent bindings | Dynamic Workflows |
|---|---|---|
| 흐름을 쥐는 쪽 | 내 코드 | 모델이 계획, Durable이 실행 |
| 추론의 범위 | 한 단계 | 계획 수립 전체 |
| 테스트 | 추론 단계만 따로 검증·대체 가능 | 계획 자체가 달라질 수 있음 |
| 어울리는 일 | 요약·분류·위험 평가 | 단계가 미리 정해지지 않는 작업 |

이 표의 "어울리는 일" 줄은 두 글을 읽고 내가 붙인 해석이다. 원문이 직접 비교한 것은 아니다. 다만 원문이 deterministic code(검증, 권한, 계산, 분기)와 agent reasoning(해석, 분류, 요약)을 나눠 놓은 것을 보면, 단계가 이미 알려진 업무라면 굳이 모델에게 계획까지 맡길 이유가 없다는 방향이다.

## 실무에서 달라지는 점

- **Python 함수 앱 운영자**: 이미 큐·Service Bus·Event Grid 트리거로 돌아가는 앱에 요약·분류만 얹고 싶을 때 앱 구조를 갈아엎지 않아도 된다. 다만 Python 3.13 이상이 필요하니 런타임 버전부터 확인해야 한다. 3.13 미만이면 업그레이드가 선행 작업이 된다.
- **보안·권한 담당**: 원문의 안전 권고는 모델에 넘기기 전에 스키마 검증, 권한 확인, 데이터 최소화를 코드에서 끝내라는 것이다. 도입 전에 "어떤 필드가 모델로 나가는가"를 `prepare` 함수 한 곳에서 리뷰할 수 있게 설계하면 검토 지점이 하나로 모인다. 원문에 상세한 위협 모델은 없으므로 프롬프트 인젝션과 출력 처리는 팀이 따로 점검해야 한다.
- **비용·한도 담당**: 원문에는 쿼터, 타임아웃, 가격, 지원 리전 정보가 없다. 프리뷰 단계라 Learn 문서에서 확인하기 전에는 운영 앱에 붙이지 않는 편이 안전하다. 호출은 핸들러가 직접 하므로 "이 조건일 때만 모델을 부른다" 같은 비용 통제를 코드에서 걸 수 있다는 점은 장점이다.
- **인증·배포 담당**: 로컬에서는 Azure CLI 로그인 기반 `DefaultAzureCredential`을 쓰고, `local.settings.json`은 커밋하지 않는다. Foundry 프로젝트 접근 권한과 모델 배포 이름(`FOUNDRY_MODEL`)을 환경 설정으로 분리해야 한다. 클라이언트는 호출마다 만들고 닫히므로 연결 재사용을 기대하면 안 된다.
- **IT 강사**: 수업에서는 "에이전트 = 자율 실행"이라는 인상을 먼저 걷어내고, 같은 업무를 (1) 코드 주도 한 단계, (2) 모델 주도 계획으로 나눠 보여주는 구도가 설명하기 좋다. 강조할 점은 Durable 재생에서 모델이 다시 호출되지 않는 이유(결과가 기록에 저장됨)다.

## 아직 모르는 것

- 프리뷰이며 Microsoft Agent Framework만 지원하고 Python 전용이다. API가 바뀔 수 있다.
- 지연 시간과 품질은 원문에서 확인되지 않는다. 실제 성능은 직접 돌려 봐야 안다.
- Durable 모드에서 결과 크기 제한 등 세부 사항은 원문에 없다.

## 참고

- [Bring agentic reasoning to Python function apps with Azure Functions Agent bindings (preview)](https://devblogs.microsoft.com/azure-sdk/azure-functions-agent-binding/) — Azure SDK Blog, 2026-10-06
- 이전 글: [Azure Functions Dynamic Workflows — 모델은 계획만, 실행은 Durable이](/posts/azure-functions-dynamic-workflows/)
- 이전 글 원문: [Dynamic Workflows in Azure Functions](https://devblogs.microsoft.com/azure-sdk/dynamic-workflows-azure-functions-hosted-skills/)
