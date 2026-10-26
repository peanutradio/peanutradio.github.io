---
title: "Azure MCP Server 3.0 안정판 — 2.0에서 올리기 전 확인할 것들"
date: 2026-10-15 06:53:00 +0900
categories: [IT News]
tags: [azure, mcp, ai-agent, security]
description: Azure MCP Server 3.0이 안정판이 됐다. 새 도구보다 DB 쿼리 도구 분류 변경, 설정 위치 이동, 새 리소스 보안 기본값이 운영자에게 더 중요하다.
---

> 이 글은 Azure SDK 블로그의 "Azure MCP Server 3.0 stable release"(2026-10-14) 발표를 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록은 아니며, 수치와 동작은 모두 원문 기준입니다.
{: .prompt-info }

## 결론부터

3.0 안정판(10/12 릴리스, 베타 50회)에서 눈길을 끄는 건 Backup·SRE Agent·Terraform 같은 새 도구 목록이지만, 운영자가 먼저 볼 것은 **"기존 설정이 어디서 깨지는가"와 "에이전트에게 쓰기 권한이 얼마나 열리는가"**입니다. 특히 MySQL·PostgreSQL 쿼리 도구가 이제 파괴적·비멱등·비읽기 전용으로 분류되고, SQL 동사 필터가 사라졌습니다. 2.0에서 올리는 팀은 이 한 줄부터 읽어야 합니다.

## 원문에서 추린 핵심

- **새 도구**: Azure Backup(보관소·정책·복구 지점, 소프트 삭제 복원), Resilience Management(복구 계획·장애 조치 검증·훈련), SRE Agent(55개 명령), Terraform(AzureRM·AzAPI 문서, Azure Verified Modules, `aztfexport`, `conftest`), Advisor, 비용 최적화 순위 등.
- **클라이언트 쪽**: `--learn`으로 실행 없이 명령과 파라미터 탐색, 알 수 없는 파라미터는 오류, 도구마다 `operationPlane`(data/control/both) 메타데이터, `--disable-proxy-tools`, 로그는 stderr 분리.
- **새 리소스 보안 기본값**: Storage는 Shared Key 비활성화, Event Hubs는 로컬 인증 비활성화, Recovery Services vault는 퍼블릭 네트워킹 비활성화, NFS 공유는 RootSquash와 전송 중 암호화. 서버가 *새로 만드는* 리소스에만 적용되고 기존 리소스는 바뀌지 않습니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="Azure MCP Server 3.0에서 쓰기 권한을 여는 순서: 읽기 전용, 도구 탐색, 쓰기 도구 하나씩 승인">
  <defs>
    <marker id="amcp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 권한을 여는 순서</text>
  <rect x="8" y="60" width="200" height="70" rx="8" class="sk-box"/>
  <text x="108" y="88" text-anchor="middle" class="sk-label">읽기 전용으로 시작</text>
  <text x="108" y="108" text-anchor="middle" class="sk-mono">--read-only</text>
  <rect x="260" y="60" width="200" height="70" rx="8" class="sk-box"/>
  <text x="360" y="88" text-anchor="middle" class="sk-label">도구·파라미터 파악</text>
  <text x="360" y="108" text-anchor="middle" class="sk-mono">--learn, operationPlane</text>
  <rect x="512" y="60" width="200" height="70" rx="8" class="sk-box-accent"/>
  <text x="612" y="88" text-anchor="middle" class="sk-label">쓰기 도구 하나씩</text>
  <text x="612" y="108" text-anchor="middle" class="sk-sub">개발 구독 + 최소 권한 신원</text>
  <path d="M210,95 L256,95" class="sk-line" marker-end="url(#amcp-m1)"/>
  <path d="M462,95 L508,95" class="sk-line-accent" marker-end="url(#amcp-m1)"/>
  <text x="360" y="180" text-anchor="middle" class="sk-sub">읽기 전용 모드와 자연어 프롬프트는 접근 통제의 대체물이 아니다 (원문 경고)</text>
</svg>
<figcaption>안정판이 됐다고 바로 쓰기를 열 이유는 없다. 읽기 전용에서 시작해 한 단계씩 연다.</figcaption>
</figure>

## 실무에서 달라지는 점

1. **DB 쿼리 도구를 쓰는 팀**: 2.0에서 SQL 동사 필터에 기대어 "SELECT만 나가겠지" 하고 있었다면 3.0에서는 그 안전장치가 없습니다. 도구 자체가 파괴적으로 분류되므로 에이전트에 연결하는 DB 계정을 읽기 전용 최소 권한으로 따로 만드는 것이 사실상 필수입니다. 통합(consolidated) 모드에서는 도구 이름도 `get_azure_databases_details`에서 `execute_azure_database_query`로 옮겨 가니, 도구 이름으로 승인 정책을 걸어 둔 곳은 같이 고쳐야 합니다.
2. **설정·스크립트를 가진 운영자**: 서버 설정이 `appsettings.json`의 `MicrosoftMcp` 아래로 이동하고, 네임스페이스가 `Azure.Mcp.Core.*`에서 `Microsoft.Mcp.Core.*`로 바뀌어 로그 레벨 키(`Logging:LogLevel:Microsoft.Mcp.Core`)도 달라집니다. 저장된 호출에서는 사라진 파라미터(구독·리소스 그룹·테넌트 등)와 `sql server delete --force`를 걷어내야 합니다. 알 수 없는 파라미터가 이제 오류라서 조용히 무시되던 옵션이 한꺼번에 드러날 수 있습니다.
3. **프리뷰부터 써 온 사람**: `azmcp resilience`는 `resiliency`로, `resilience_*` 도구 접두사는 `resiliency_*`로 바뀌었습니다. VM 생성은 `--image`가 필수이고 `Win2019Datacenter`·`Ubuntu2004` 별칭은 삭제됐습니다. 응답 파싱 코드(Advisor는 ARM 스타일 `properties`, Monitor 로그와 AI Search 쿼리는 `results` 래퍼)도 점검 대상입니다.
4. **보안 담당자**: 새 기본값은 에이전트가 *새로 만드는* 리소스에만 적용됩니다. 기존 Storage 계정의 Shared Key가 꺼지는 것이 아니므로 "3.0으로 올렸으니 안전해졌다"는 식의 보고는 틀립니다. 반대로 에이전트가 만든 Storage를 Shared Key 방식으로 쓰던 스크립트는 새 리소스에서 인증 오류가 날 수 있습니다. 또 SRE Agent 도구는 프리뷰 리소스 공급자(`Microsoft.App/SREAgentPreview`)를 쓰므로 운영 구독에 바로 붙이기 전에 프리뷰 범위를 확인하세요.
5. **교육할 때 강조할 점**: 시연은 `npx -y @azure/mcp@3.0.0 server start --read-only`로 개발 구독에 붙여 Advisor 비용 권고를 설명시키는 정도로 시작하는 구성이 안전합니다. 버전을 `3.0.0`으로 고정해 두는 것도 수강생 환경이 흔들리지 않게 하는 방법입니다. Advisor의 apply 도구는 실제 리소스를 바꾸지 않고 IaC 준비 규칙만 돌려준다는 점, 시작 속도 39% 개선은 특정 CLI 명령 호출에만 해당한다는 점을 함께 알려 주면 오해가 줄어듭니다.

## 내가 보는 의미

"에이전트에게 어디까지 쓰기를 줄 것인가"에 대해 3.0은 도구 쪽에서 답을 주기 시작했습니다. `operationPlane` 메타데이터는 데이터 평면과 제어 평면 호출을 구분해 승인 정책을 짜는 재료가 되고, `--learn`은 실제 호출 없이 명령 표면을 파악하게 합니다. 다만 이 둘이 실제로 통제에 얼마나 쓸모 있는지는 써 보며 확인할 부분이고, 원문도 읽기 전용 모드가 접근 통제를 대신하지 않는다고 선을 긋습니다. 결국 권한 설계는 신원과 RBAC에서 끝납니다. 이 관점은 [Durable Task가 해 주는 일, 해 주지 않는 일](/posts/durable-task-real-world-boundaries/)에서 정리한 "승인은 설계에서 나온다"와 같은 이야기이고, MCP 서버를 에이전트에 붙이는 방법 자체는 [Copilot Studio에 MCP 서버 붙이기](/posts/copilot-studio-mcp-tools/)와 [MCP와 API는 무엇이 다른가](/posts/mcp-vs-api/)에서 다뤘습니다.

> 원격 MCP 서버 배포는 아직 프리뷰이고, 커스텀 HTTP 클라이언트는 프로토콜 `2026-07-28`에서 모든 POST에 `Mcp-Method`·`Mcp-Name` 헤더를 보내야 합니다(`2025-11-25` 클라이언트는 호환). 자체 클라이언트를 가진 팀은 이 부분도 확인하세요.
{: .prompt-warning }

## 참고

- [Azure MCP Server 3.0 stable release — Azure SDK Blog](https://devblogs.microsoft.com/azure-sdk/azure-mcp-server-3-0-stable-release/)
