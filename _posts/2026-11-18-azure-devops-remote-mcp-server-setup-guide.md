---
title: "Azure DevOps 원격 MCP 서버 연결 — 클라이언트별 순서와 막히는 곳"
date: 2026-11-18 06:53:00 +0900
categories: [AI Agent]
tags: [mcp, devops, identity, github]
description: Azure DevOps 원격 MCP 서버를 VS Code와 Claude Code에 연결하는 문서 기준 순서, 원격·로컬 선택 기준, 헤더 옵션, 자주 막히는 지점을 정리했다.
---

> 이 글은 Microsoft Learn의 Azure DevOps 원격 MCP 서버 문서(2026-11-03 갱신본)를 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록이 아니라 문서 기준 순서이며, 화면과 값은 실제 환경에서 확인해야 합니다.
{: .prompt-info }

## 결론 3줄

1. 원격 서버는 설치 없이 `https://mcp.dev.azure.com/{organization}` 한 줄과 Entra 로그인으로 붙고, 문서도 이쪽을 권장합니다.
2. Claude Desktop·Codex는 Entra 인증 흐름을 지원하지 않아 원격 대신 로컬 서버(Node.js 20+)를 써야 하고, Cursor·Claude Code는 직접 만든 Entra 앱 등록이 필요합니다.
3. 도구를 켠다고 권한이 생기지 않습니다. 앱 스코프와 사용자의 Azure DevOps 권한이 모두 허용해야 동작하므로, 처음에는 읽기 전용 헤더로 시작하는 편이 안전합니다.

## 시작 전 확인

문서가 제시한 전제 조건은 다음과 같습니다.

- Microsoft Entra 테넌트에 연결된 Azure DevOps 조직이 있어야 합니다. 독립 MSA 조직은 원격 서버를 지원하지 않습니다.
- 해당 프로젝트의 구성원이고 조회할 리소스에 접근 권한이 있어야 합니다.
- 지원 클라이언트여야 합니다. 개요 문서는 AI 어시스턴트가 에이전트 모드로 동작해야 한다고 적고 있습니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="MCP 클라이언트가 Entra 로그인 후 mcp.dev.azure.com을 거쳐 Azure DevOps 조직에 접근하는 흐름">
  <defs>
    <marker id="admcp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="admcp-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 원격 MCP 서버 접속 흐름</text>
  <rect x="8" y="50" width="150" height="70" rx="8" class="sk-box"/>
  <text x="83" y="80" text-anchor="middle" class="sk-label">MCP 클라이언트</text>
  <text x="83" y="100" text-anchor="middle" class="sk-sub">VS Code · Copilot CLI 등</text>
  <rect x="210" y="50" width="130" height="70" rx="8" class="sk-box-muted"/>
  <text x="275" y="80" text-anchor="middle" class="sk-label">Entra ID</text>
  <text x="275" y="100" text-anchor="middle" class="sk-sub">OAuth 로그인</text>
  <rect x="392" y="50" width="150" height="70" rx="8" class="sk-box-accent"/>
  <text x="467" y="80" text-anchor="middle" class="sk-label">원격 MCP 서버</text>
  <text x="467" y="100" text-anchor="middle" class="sk-mono">mcp.dev.azure.com</text>
  <rect x="590" y="50" width="122" height="70" rx="8" class="sk-box"/>
  <text x="651" y="80" text-anchor="middle" class="sk-label">Azure DevOps</text>
  <text x="651" y="100" text-anchor="middle" class="sk-sub">조직·프로젝트</text>
  <path d="M160,85 L208,85" class="sk-line" marker-end="url(#admcp-m1)"/>
  <path d="M342,85 L390,85" class="sk-line" marker-end="url(#admcp-m1)"/>
  <path d="M544,85 L588,85" class="sk-line-accent" marker-end="url(#admcp-a1)"/>
  <text x="360" y="170" text-anchor="middle" class="sk-sub">최종 허용 = 앱 스코프 AND 사용자의 Azure DevOps 권한</text>
</svg>
<figcaption>도구 설정은 '보이는 범위'만 정하고, 실제 허용은 스코프와 사용자 권한의 교집합이다.</figcaption>
</figure>

## 단계별 순서

### 1. VS Code (문서에 단계가 명시된 경로)

1. 저장소에 `.vscode/mcp.json`을 만들고 아래 내용을 넣습니다.

```json
{
  "servers": {
    "ado-remote-mcp": {
      "url": "https://mcp.dev.azure.com/{organization}",
      "type": "http"
    }
  },
  "inputs": []
}
```

2. 파일을 저장하고 VS Code에서 GitHub Copilot을 엽니다.
3. 안내가 뜨면 Microsoft Entra 계정으로 인증합니다.
4. 인증이 끝나면 사용 가능한 도구 목록이 나타납니다.
5. "List the projects in my Azure DevOps organization."처럼 물어서 내 조직 결과가 나오면 연결 성공입니다.

조직 이름을 URL에서 뺄 수도 있지만, 그러면 매 호출마다 조직을 맥락으로 알려 줘야 합니다.

### 2. 범위를 좁히는 헤더 (같은 설정에 추가)

`headers` 객체에 다음을 넣을 수 있습니다.

- `"X-MCP-Readonly": "true"` — 쓰기 도구 제외
- `"X-MCP-Toolsets": "repos,wiki,wit"` — 도구 묶음 선택
- `"X-MCP-Tools": "core_list_projects, wit_work_item"` — 개별 도구 선택

Toolsets와 Tools는 함께 쓰지 않습니다. Toolsets와 Readonly는 함께 쓸 수 있습니다.

### 3. Claude Code (직접 만든 Entra 앱 필요)

문서 기준 순서입니다.

1. Entra 관리 센터의 앱 등록에서 앱을 만들고, Authentication에서 `http://localhost:3118/callback`을 모바일·데스크톱 리디렉션 URI로 추가한 뒤 "Allow public client flows"를 켭니다.
2. API permissions에서 "Azure DevOps MCP"(애플리케이션 ID `2a72489c-aab2-4b65-b93a-a91edccf33b8`)의 필요한 위임 권한만 추가하고, 관리자 동의를 받습니다. 테넌트 관리자가 필요할 수 있습니다.
3. 앱의 Client ID를 복사해 아래 명령을 실행합니다.

```bash
claude mcp add --transport http ado https://mcp.dev.azure.com/{organization} \
  --client-id {client-id} --callback-port 3118
```

4. `claude`를 시작해 `/mcp`를 실행하고 브라우저 로그인을 마칩니다.

Cursor는 리디렉션 URI(`http://localhost:8787/callback`)와 설정 위치(Settings > Tools & MCP)가 다릅니다. Visual Studio는 문서에 "MCP 설정에 서버 URL을 추가"라고만 있고 메뉴 경로가 없어서 별도 문서 확인이 필요합니다.

## 원격 vs 로컬, 무엇을 고를까

| 기준 | 원격 서버 | 로컬 서버 |
|---|---|---|
| 설치 | 없음 | Node.js 20.0+ 및 npx |
| 전송 방식 | streamable HTTP | stdio |
| 인증 | Entra ID(OAuth) | PAT 또는 Entra ID |
| 해당 클라이언트 | VS Code, Visual Studio, Copilot CLI·앱, Foundry, Copilot Studio 등 | Claude Desktop, Codex 등 Entra 흐름이 안 되는 곳 |
| 문서의 권장 | 권장 | 원격이 안 될 때 |
| 주의 | MSA 독립 조직 불가 | PAT 보관 책임이 사용자에게 있음 |

기준은 단순합니다. 클라이언트가 Entra 로그인을 지원하면 원격, 아니면 로컬입니다. Copilot Studio에 붙이는 경우의 인증 선택은 [Copilot Studio에 MCP 서버 연결하기](/posts/copilot-studio-connect-mcp-server-guide/)와 같이 보면 비교가 쉽습니다.

## 자주 막히는 지점

### 1. 'Azure DevOps MCP' 엔터프라이즈 앱이 테넌트에 없다

Cursor·Claude Code용 앱 등록은 이 엔터프라이즈 앱이 테넌트에 프로비저닝되어 있다는 전제입니다. 문서에 별도 트러블슈팅 페이지가 링크되어 있으므로, 앱 등록 전에 테넌트 관리자에게 확인을 요청하세요.

### 2. 관리자 동의 대기

위임 권한을 추가해도 Grant admin consent가 끝나기 전에는 로그인이 실패할 수 있습니다. 보안팀과 필요한 스코프 목록을 미리 합의해 두면 대기 시간이 줄어듭니다. 문서는 필요한 권한만 부여하고, 읽기 도구만 쓴다면 읽기 스코프로 충분하다고 안내합니다.

### 3. 연결 거부(Connection Refused)

사내 프록시·방화벽이 `mcp.dev.azure.com`으로의 아웃바운드 HTTPS를 막는 경우입니다. 허용 목록에 추가를 요청하고, 원인 분리를 위해 VPN 없이도 시도해 보라는 것이 문서의 안내입니다.

### 4. 도구가 일부 안 보인다

`X-MCP-Toolsets`로 범위를 제한했다면 필요한 값(예: Enterprise Live Migration 도구는 `elm`)이 빠져 있지 않은지 확인합니다. 문서의 도구 목록은 최신 변경을 항상 반영하지 않을 수 있다는 경고도 있습니다.

### 5. 도구는 보이는데 데이터가 비어 있다

도구 활성화는 권한을 부여하지 않습니다. 해당 프로젝트에 대한 내 Azure DevOps 권한과 앱 스코프를 둘 다 확인하세요.

### 6. 리디렉션 URI 불일치

Cursor Cloud Agents 설정에서 문서가 언급한 AADSTS50011은 리디렉션 URI 불일치, AADSTS7000218은 클라이언트 시크릿 오류입니다.

## 도입 전 점검표

- 조직이 Entra 테넌트에 연결되어 있는가
- 사용할 클라이언트가 Entra 인증을 지원하는가(아니면 로컬 서버)
- 처음에는 `X-MCP-Readonly`와 필요한 Toolsets만 허용했는가
- 앱 등록이 필요한 클라이언트는 관리자 동의 일정을 잡았는가
- 문서에는 원격 서버를 켜는 조직 단위 토글이 나오지 않는다. 사내 정책이 있다면 별도로 확인

## 참고

- Azure DevOps 원격 MCP 서버: https://learn.microsoft.com/en-us/azure/devops/mcp-server/remote-mcp-server
- Azure DevOps MCP 서버 개요: https://learn.microsoft.com/en-us/azure/devops/mcp-server/mcp-server-overview?view=azure-devops
- Azure DevOps 블로그, "Azure DevOps and GitHub: start your agentic journey where you are" (2026-11-17): https://devblogs.microsoft.com/devops/azure-devops-and-github-start-your-agentic-journey-where-you-are/
- 이슈 제보(Remote 템플릿): https://github.com/microsoft/azure-devops-mcp/issues/new?template=remote-mcp-server-issue.md
