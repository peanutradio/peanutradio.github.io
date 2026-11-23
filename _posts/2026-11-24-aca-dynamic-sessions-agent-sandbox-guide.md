---
title: "에이전트 코드 실행 샌드박스, ACA 세션 풀 고르는 법과 설정 순서"
date: 2026-11-24 06:53:00 +0900
categories: [AI Agent]
tags: [ai-agent, azure, security, cost]
description: LLM이 만든 코드를 격리해 돌리는 Azure Container Apps 동적 세션. 코드 인터프리터와 커스텀 컨테이너 풀을 고르는 기준, 문서 기준 설정 순서, 막히는 지점.
---

> 이 글은 Microsoft Learn의 Azure Container Apps 동적 세션 문서(sessions, sessions-code-interpreter, sessions-custom-container, session-pool, sessions-usage, billing)를 읽고 정리·재구성한 것입니다. 직접 배포해 본 기록이 아니라 문서 기준 순서이며, 명령어와 값은 문서에 나온 것만 옮겼습니다.
{: .prompt-info }

## 결론 3줄

- 에이전트가 만든 코드를 내 서비스 프로세스 안에서 돌리지 말고, **세션 풀(session pool)** 이라는 격리 단위에 맡기는 것이 문서가 제시하는 방식입니다.
- 파이썬·Node·셸 정도면 **코드 인터프리터 풀**, 사내 이미지나 다른 언어·프로토콜이 필요하면 **커스텀 컨테이너 풀**을 고릅니다.
- 보안 기본값은 `EgressDisabled`(외부 송신 차단)이고, 풀 한 개로 시작해 역할(Role)·네트워크·비용 확인 후 늘리는 순서가 안전합니다. 문서에 GA/프리뷰 표기가 명시돼 있지 않으니 아래 "확인할 것"을 먼저 보세요.

## 어떤 문제를 푸는가

에이전트가 데이터 분석 코드를 만들어 실행하거나 사용자가 올린 파일을 처리하게 하면, 그 코드는 신뢰할 수 없는 코드입니다. 문서도 세션 안의 코드를 악의적이라고 가정하라고 경고합니다. 동적 세션은 요청이 올 때 세션을 할당해 주고, 일정 시간 요청이 없으면 정리하는 구조입니다. 호스티드 에이전트의 이그레스 통제와 동시 세션 이야기는 [Foundry 호스티드 에이전트 글](/posts/foundry-hosted-agents-ignite-2026-egress-sessions/)과 이어서 보면 비교가 됩니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="에이전트가 identifier로 세션 풀에 요청하면 격리된 세션이 할당되고 유휴 시간이 지나면 정리되는 구조">
  <defs>
    <marker id="acs-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="acs-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 세션 풀 구조 — identifier 하나가 세션 하나</text>
  <rect x="8" y="60" width="150" height="60" rx="8" class="sk-box"/>
  <text x="83" y="86" text-anchor="middle" class="sk-label">에이전트 앱</text>
  <text x="83" y="104" text-anchor="middle" class="sk-sub">토큰 + identifier</text>
  <rect x="210" y="40" width="300" height="150" rx="10" class="sk-box-accent"/>
  <text x="360" y="62" text-anchor="middle" class="sk-label">세션 풀 (Session Executor 역할 필요)</text>
  <rect x="230" y="80" width="80" height="44" rx="6" class="sk-box"/>
  <text x="270" y="106" text-anchor="middle" class="sk-sub">세션 A</text>
  <rect x="320" y="80" width="80" height="44" rx="6" class="sk-box"/>
  <text x="360" y="106" text-anchor="middle" class="sk-sub">세션 B</text>
  <rect x="410" y="80" width="80" height="44" rx="6" class="sk-box-muted"/>
  <text x="450" y="106" text-anchor="middle" class="sk-sub">대기(ready)</text>
  <text x="360" y="160" text-anchor="middle" class="sk-sub">cooldown 경과 시 삭제 · 기본 외부 송신 차단</text>
  <rect x="560" y="60" width="150" height="60" rx="8" class="sk-box-muted"/>
  <text x="635" y="86" text-anchor="middle" class="sk-label">인터넷·내부망</text>
  <text x="635" y="104" text-anchor="middle" class="sk-sub">EgressEnabled일 때만</text>
  <path d="M160,90 L208,90" class="sk-line-accent" marker-end="url(#acs-a1)"/>
  <path d="M512,90 L558,90" class="sk-line" marker-end="url(#acs-m1)"/>
</svg>
<figcaption>격리의 핵심은 풀이 아니라 "세션별 identifier와 외부 송신 기본 차단"이다.</figcaption>
</figure>

## 단계별 순서 (문서 기준)

### 1. CLI 준비

```bash
az upgrade
az extension add --name containerapp --upgrade --allow-preview true -y
```

문서의 REST 예제가 모두 `-preview` API 버전이고 확장 설치에도 `--allow-preview true`가 붙습니다.

### 2. 풀 유형 결정

아래 비교표를 보고 정합니다. 커스텀 컨테이너는 워크로드 프로필이 켜진 Container Apps 환경이 필요합니다.

### 3. 풀 생성

코드 인터프리터 풀 예시는 문서에 다음과 같이 나옵니다.

```bash
az containerapp sessionpool create --name my-session-pool \
  --resource-group <RESOURCE_GROUP> --location westus2 \
  --container-type PythonLTS --max-sessions 100 \
  --cooldown-period 300 --network-status EgressDisabled
```

커스텀 컨테이너 풀은 `--container-type CustomContainer`에 `--environment`, `--image`, `--registry-server`, `--target-port`, `--cpu`, `--memory`, `--ready-sessions` 등을 추가합니다. 값 예시는 `--cpu 0.25 --memory 0.5Gi --target-port 80`입니다. 인자는 버전에 따라 바뀔 수 있으니 `--help`로 확인하라고 문서가 안내합니다.

### 4. 호출 주체에 역할 부여

```bash
az role assignment create --role "Azure ContainerApps Session Executor" \
  --assignee <PRINCIPAL_ID> --scope <SESSION_POOL_RESOURCE_ID>
```

토큰은 `az account get-access-token --resource https://dynamicsessions.io`로 받고, SDK에서는 스코프 `https://dynamicsessions.io/.default`를 씁니다. 프레임워크 연동(LangChain, LlamaIndex, Semantic Kernel)은 역할이 부여된 관리 ID로 토큰을 처리합니다.

### 5. 관리 엔드포인트 조회와 호출

```bash
az containerapp sessionpool show --name <SESSION_POOL_NAME> \
  --resource-group <RESOURCE_GROUP> \
  --query "properties.poolManagementEndpoint" --output tsv
```

실행 요청은 이 엔드포인트에 `identifier=<세션 ID>`를 쿼리로 붙여 보냅니다. 같은 identifier면 실행 중인 세션을 재사용하고 없으면 새로 할당합니다. 길이는 4~128자입니다.

## 비교표: 두 풀 중 무엇을 고르나

| 기준 | 코드 인터프리터 풀 | 커스텀 컨테이너 풀 |
|---|---|---|
| 런타임 | `PythonLTS`, `NodeLTS`, `Shell` 내장 | 내 이미지(임의 언어·TCP 프로토콜, HTTP 서버 필요) |
| 최대 세션 | `--max-sessions` 최대 600 | 문서에 최대치 명시 없음 |
| 사전 환경 | 없음 | 워크로드 프로필 환경 필요 |
| 로그 | Log Analytics에 세션 로그 없음, 응답에 stdout/stderr | `AppEnvSessionConsoleLogs` 등 테이블 |
| 과금 | 할당된 세션의 실행 시간 기준 | Dedicated 플랜, E16 전용 인스턴스 |
| 적합 | 분석·변환 코드 실행 | 사내 도구·특수 런타임 |

수명은 기본 `Timed`(마지막 요청 후 `cooldownPeriodInSeconds` 경과 시 삭제, 300~3600초)이고, 커스텀 풀만 `OnContainerExit`을 고를 수 있습니다.

## 자주 막히는 지점

### 1. API 버전과 경로가 문서마다 다르다

코드 인터프리터 문서는 실행·파일에 `2025-10-02-preview`, 세션 관리에 `2025-02-02-preview`를 쓰고, 풀 문서는 `2024-02-02-preview`와 다른 경로(`code/execute`)를 씁니다. 복사해서 쓰기 전에 REST 레퍼런스에서 현재 버전을 확인해야 합니다.

### 2. 권한 오류

호출 주체에 풀 범위로 Session Executor 역할이 없거나, 토큰 대상(audience)이 `https://dynamicsessions.io`가 아니면 거부됩니다. 코드 인터프리터 문서는 Contributor도 함께 언급하므로 어떤 작업에 무엇이 필요한지 확인하세요.

### 3. 세션이 갑자기 사라진다

cooldown은 호출마다 초기화되지만, 지나면 삭제되어 다음 요청에서 "session not found"가 날 수 있습니다. 상태를 세션에만 두지 말고, 파일 업로드(`/mnt/data`, 128MB 한도)나 외부 저장소에 두세요. 코드 실행 한 번의 최대 시간은 220초입니다.

### 4. 외부 송신을 켜는 순간 위험이 바뀐다

`EgressEnabled`면 세션 코드가 인터넷에 접근할 수 있고, 문서는 신뢰할 수 없는 코드가 컨테이너의 환경 변수·비밀·파일에 접근한다고 가정하라고 합니다. 커스텀 풀의 `managedIdentitySettings`도 기본 `None`이며 `Main`은 "주의해서 사용"하라고 되어 있습니다.

### 5. identifier를 사용자 입력으로 만들면 세션이 섞인다

identifier는 풀 안에서 유일한 문자열이고 문서는 민감한 값으로 다루라고 합니다. 사용자·테넌트마다 서버에서 생성·검증해, 다른 사용자의 세션에 접근하지 못하게 해야 합니다.

## 확인할 것

- GA/프리뷰 상태: 읽은 페이지에는 명시가 없고 프리뷰 API 버전이 쓰입니다. 운영 도입 전 공식 상태를 확인하세요.
- 과금: 코드 인터프리터는 "1시간 단위"로 적혀 있어 의외이므로 [가격 페이지](https://azure.microsoft.com/pricing/details/container-apps/)와 대조해야 합니다. 문서에 단가는 없습니다. 커스텀 풀은 `properties.nodeCount`로 노드 수를 볼 수 있습니다.
- 리전: 포털의 Location 드롭다운에서 가능 여부를 확인하라고 문서가 안내합니다.

## 참고

- [Azure Container Apps 동적 세션](https://learn.microsoft.com/en-us/azure/container-apps/sessions)
- [코드 인터프리터 세션](https://learn.microsoft.com/en-us/azure/container-apps/sessions-code-interpreter)
- [커스텀 컨테이너 세션](https://learn.microsoft.com/en-us/azure/container-apps/sessions-custom-container)
- [세션 풀 관리](https://learn.microsoft.com/en-us/azure/container-apps/session-pool)
- [세션 사용](https://learn.microsoft.com/en-us/azure/container-apps/sessions-usage)
- [Container Apps 청구](https://learn.microsoft.com/en-us/azure/container-apps/billing)
