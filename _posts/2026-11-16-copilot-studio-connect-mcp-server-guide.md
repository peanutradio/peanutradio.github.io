---
title: "Copilot Studio에 MCP 서버 연결하기 — 인증 방식 고르는 기준"
date: 2026-11-16 06:53:00 +0900
categories: [Copilot Studio]
tags: [copilot-studio, mcp, governance]
description: Copilot Studio 에이전트에 기존 MCP 서버를 붙이는 순서와, None·API key·OAuth 2.0 세 방식 중 고르는 기준을 문서 기준으로 정리했다.
---

> 이 글은 Microsoft Learn의 Copilot Studio MCP 문서(연결 문서 ms.date 2026-10-30)를 읽고 정리·재구성한 것이다. 직접 실행한 기록이 아니라 "문서 기준 순서"이며, 문서에 없는 항목은 없다고 적었다.
{: .prompt-info }

## 결론 3줄

- 기존 MCP 서버는 **Tools → Add a tool → New tool → Model Context Protocol** 마법사로 붙이는 것이 문서의 권장 경로다.
- 전송 방식은 Streamable만 지원하고, SSE는 2025년 8월 이후 지원이 끝났다. 서버가 SSE뿐이면 연결 전에 서버부터 손봐야 한다.
- 인증은 서버가 DCR을 지원하는지에 따라 갈린다. 지원하면 Dynamic discovery, 아니면 Dynamic이나 Manual로 내려간다.

## 시작 전 점검

문서는 준비물 목록을 따로 주지 않고, 이미 MCP 서버가 있다는 전제에서 시작한다. 그래서 아래는 문서 내용에서 뽑은 "확인할 것"이다.

- 서버가 Streamable 전송을 쓰는가
- 서버 URL을 알고 있는가
- 서버의 인증 방식이 무엇인가 (없음 / API 키 / OAuth 2.0)
- OAuth라면 우리 쪽 ID 공급자 앱 등록을 만질 권한이 있는가
- 테넌트에 Power Platform 커넥터 데이터 정책이 걸려 있는가 (아래 '막히는 지점' 참고)

## 단계별 순서 (마법사 방식)

### 1. 도구 추가 화면 열기

에이전트의 **Tools** 페이지에서 **Add a tool**을 누르고 **New tool**을 고른다.

### 2. 종류 선택

**Create new** 영역에서 **Model Context Protocol**을 고르면 MCP 온보딩 마법사가 열린다.

### 3. 기본 필드 입력

**Server name**, **Server description**, **Server URL**을 채운다. 세 칸 모두 필수다.

### 4. 인증 유형 선택

**None**, **API key**, **OAuth 2.0** 중 하나를 고른다. 세부는 아래 표를 본다.

### 5. 연결 만들기

**Create**를 누르면 **Add tool** 대화상자가 나온다. **Create a new connection**을 하거나 기존 연결을 고른 뒤 **Add to agent**를 누른다.

<figure class="sketch">
<svg viewBox="0 0 720 270" role="img" aria-label="Copilot Studio에서 MCP 서버를 연결하는 인증 분기 흐름도">
  <defs>
    <marker id="cs-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① MCP 서버 인증 분기</text>
  <rect x="8" y="100" width="150" height="56" rx="8" class="sk-box-accent"/>
  <text x="83" y="124" text-anchor="middle" class="sk-label">서버 인증은?</text>
  <text x="83" y="142" text-anchor="middle" class="sk-sub">마법사 4단계</text>
  <path d="M160,115 L230,55" class="sk-line" marker-end="url(#cs-m1)"/>
  <path d="M160,128 L230,128" class="sk-line" marker-end="url(#cs-m1)"/>
  <path d="M160,142 L230,205" class="sk-line" marker-end="url(#cs-m1)"/>
  <rect x="232" y="30" width="180" height="50" rx="8" class="sk-box"/>
  <text x="322" y="52" text-anchor="middle" class="sk-label">None</text>
  <text x="322" y="68" text-anchor="middle" class="sk-sub">Create로 끝</text>
  <rect x="232" y="103" width="180" height="50" rx="8" class="sk-box"/>
  <text x="322" y="125" text-anchor="middle" class="sk-label">API key</text>
  <text x="322" y="141" text-anchor="middle" class="sk-sub">Header 또는 Query + 이름</text>
  <rect x="232" y="180" width="180" height="50" rx="8" class="sk-box"/>
  <text x="322" y="202" text-anchor="middle" class="sk-label">OAuth 2.0</text>
  <text x="322" y="218" text-anchor="middle" class="sk-sub">세 갈래로 다시 분기</text>
  <path d="M414,205 L470,205" class="sk-line" marker-end="url(#cs-m1)"/>
  <rect x="472" y="150" width="240" height="40" rx="8" class="sk-box"/>
  <text x="592" y="174" text-anchor="middle" class="sk-sub">Dynamic discovery (DCR+탐색)</text>
  <rect x="472" y="196" width="240" height="40" rx="8" class="sk-box"/>
  <text x="592" y="220" text-anchor="middle" class="sk-sub">Dynamic (DCR, 탐색 없음)</text>
  <rect x="472" y="242" width="240" height="26" rx="8" class="sk-box-muted"/>
  <text x="592" y="260" text-anchor="middle" class="sk-sub">Manual (전부 직접 입력)</text>
</svg>
<figcaption>서버가 지원하는 것이 많을수록 입력할 칸이 줄어든다.</figcaption>
</figure>

## 인증 방식 비교표

| 방식 | 언제 쓰나 | 필요한 입력 | 이후 할 일 |
|---|---|---|---|
| None | 사내 읽기 전용 등 인증이 없는 서버 | 없음 | **Create** |
| API key | 서버가 키를 요구 | **Type**(Header 또는 Query), 헤더·쿼리 이름 | 에이전트 사용자가 키를 제공, 에이전트가 요청에 실어 보냄 |
| OAuth 2.0 – Dynamic discovery | 서버가 DCR과 탐색을 모두 지원 | 추가 입력 없음 | **Create** → **Next** |
| OAuth 2.0 – Dynamic | DCR은 되지만 탐색은 안 됨 | **Authorization URL**, **Token URL template** | 콜백 URL이 보이면 ID 공급자 앱 등록에 추가 |
| OAuth 2.0 – Manual | DCR 미지원 | **Client ID**, **Client secret**, **Authorization URL**, **Token URL template**, **Refresh URL** (Scopes는 선택, 공백으로 구분) | 나타난 콜백 URL을 ID 공급자 등록에 복사 |

고르는 기준은 간단하다. 사용자별 권한이 중요하면 OAuth 2.0, 서버 하나에 공용 키면 충분하면 API key, 인증이 필요 없는 서버만 None이다. 문서는 OAuth를 "사용자가 자격 증명을 에이전트와 공유하지 않고 자기 데이터 접근을 허용하게 하는 방식"으로 설명한다.

## 마법사 말고 커스텀 커넥터로 붙이는 길

문서는 두 번째 방법도 준다. OpenAPI(Swagger 2.0 YAML) 스키마 파일이 있어야 하고, 경로는 **Tools → Add a tool → New tool → Custom connector**로 Power Apps를 연 뒤 **New custom connector → Import OpenAPI file** 순서다. 문서의 샘플 스키마에는 `x-ms-agentic-protocol: mcp-streamable-1.0`과 `operationId: InvokeMCP`가 들어 있다. 수정은 Power Apps나 Power Automate 포털의 **Custom connectors**에서 한다. 문서가 권장하는 쪽은 마법사다.

## 자주 막히는 지점

### 1. 서버가 SSE만 지원한다

문서는 현재 Streamable 전송만 지원하고, SSE는 2025년 8월 이후 지원하지 않는다고 적었다. 오래된 서버 예제를 그대로 가져왔다면 여기서 막힌다. 연결 설정이 아니라 서버 쪽 전송 방식을 바꿔야 한다.

### 2. OAuth에서 콜백 URL을 등록하지 않았다

Dynamic과 Manual은 **Create** 뒤에 콜백 URL이 나타나고, 이를 ID 공급자의 앱 등록에 넣어야 한다. 이 단계를 건너뛰면 연결 단계에서 인증이 끝나지 않을 것이다. 이 증상 자체는 문서가 기술한 것이 아니라 단계 누락에 대한 추정이다.

### 3. 데이터 정책이 MCP 서버까지 막는다

문서에 따르면 Copilot Studio의 MCP 서버 접근은 Power Platform 커넥터에 기대고 있어서, 커넥터를 규제하는 데이터 정책이 있으면 MCP 서버와 그 도구 접근도 함께 규제된다. 테넌트 관리자에게 정책을 먼저 물어보는 편이 빠르다.

### 4. 인증 방식을 잘못 짚었다

서버가 DCR을 지원하는지 모르는 채 Manual로 시작하면 입력칸 다섯 개를 서버 문서에서 찾아야 한다. 먼저 서버 쪽 문서에서 DCR과 탐색 지원 여부를 보고 Dynamic discovery부터 내려가는 순서가 낫다.

### 5. 도구 선택·한도는 문서에 없다

연결 후 도구가 어떻게 나열되고 고르는지, 호출 한도가 얼마인지는 이 연결 문서에 적혀 있지 않았다. 운영 전에 별도 확인이 필요한 항목으로 남겨 둔다.

## 이어서 볼 것

- 서버를 직접 만들 때: 서버 생성 문서(ms.date 2026-08-25)는 MCP SDK를 쓰라고만 하고 메뉴 경로나 지원 언어 목록은 주지 않는다. 인증은 선택이고 API key나 OAuth 2.0을 쓸 수 있다.
- 배포 쪽 비용·구성은 [Foundry azd 확장 GA](/posts/foundry-azd-extension-ga/) 글과 같이 읽으면 좋다.
- 문서는 대안으로 Agents 365 CLI와 Microsoft 365 관리 센터로 서버를 등록하는 BYO 방식도 링크한다.

## 참고

- Connect your agent to an existing MCP server: https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-add-existing-server-to-agent
- Create a new MCP server: https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-create-new-server
