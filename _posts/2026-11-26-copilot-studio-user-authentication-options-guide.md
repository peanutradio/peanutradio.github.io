---
title: "Copilot Studio 사용자 인증 3종 — 고르는 기준과 설정 순서"
date: 2026-11-26 06:53:00 +0900
categories: [Copilot Studio]
tags: [copilot-studio, identity, governance]
description: Copilot Studio 에이전트의 인증 옵션(인증 없음·Microsoft 인증·수동 인증)을 고르는 기준과 설정 순서, 막히는 지점을 공식 문서 기준으로 정리한다.
---

> 이 글은 Microsoft Learn의 "Configure user authentication in Copilot Studio" 문서(2026-10-21 갱신본)를 읽고 정리·재구성한 것입니다. 메뉴 이름과 설정값은 이 문서에서 확인된 것만 적었고, 직접 실행한 기록이 아니라 문서 기준 순서입니다.
{: .prompt-info }

## 결론 3줄

- Teams·SharePoint·Power Apps·Microsoft Copilot에 붙일 사내용 에이전트는 **Authenticate with Microsoft**가 기본 선택입니다. 설정이 거의 없습니다.
- 사용자 토큰(`User.AccessToken`)이 필요하거나 웹 등 다른 채널에서 로그인이 필요하면 **Authenticate manually**로 가야 합니다.
- **No authentication**은 링크를 아는 누구나 대화할 수 있는 상태입니다. 공개 정보 전용이 아니면 고르지 않는 편이 맞습니다.

## 설정 순서 (문서 기준)

### 1. 인증 화면 열기

에이전트의 **Settings**에서 **Security**를 선택하고 **Authentication**을 엽니다.

### 2. 옵션 선택

세 가지 중 하나를 고릅니다. 고르는 기준은 아래 판단표를 봅니다.

### 3. 수동 인증이면 서비스 공급자 선택

문서가 지원 대상으로 든 공급자는 다음과 같습니다.

- Microsoft Entra ID V2 with federated credentials
- Microsoft Entra ID V2 with certificates
- Microsoft Entra ID V2 with client secrets
- Microsoft Entra ID
- Generic OAuth 2 (OAuth 2.0 표준을 따르는 모든 공급자)

선택한 공급자에 따라 화면에 보이는 필드가 달라집니다. 문서에 나온 필드는 Client ID, Client secret, Tenant ID, Scopes, Scope list delimiter, Authorization URL template, Token URL template, Refresh URL template, Token exchange URL(SSO에 필수) 등입니다. 인증서 방식에서는 Client Certificate KeyVault URL과 Is x5c Claim required가 추가됩니다.

### 4. Require users to sign in 결정

수동 인증에서만 쓰는 토글입니다. 켜면 로그인하지 않은 사용자에게 시스템 토픽 **Require users to sign in**이 자동으로 실행되고, 로그인에 실패하면 **Escalate** 시스템 토픽으로 넘어갑니다. 끄면 로그인이 필요한 토픽에 도달했을 때만 로그인을 요청합니다.

### 5. 저장 후 게시

**Save**를 누릅니다. 문서가 강조하듯 인증 설정 변경은 **에이전트를 게시한 뒤에야** 적용됩니다.

<figure class="sketch">
<svg viewBox="0 0 720 280" role="img" aria-label="Copilot Studio 인증 옵션을 고르는 분기 흐름">
  <defs>
    <marker id="csa-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="csa-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 인증 옵션 고르기</text>
  <rect x="8" y="100" width="170" height="64" rx="8" class="sk-box"/>
  <text x="93" y="126" text-anchor="middle" class="sk-label">누가 쓰나?</text>
  <text x="93" y="146" text-anchor="middle" class="sk-sub">채널·토큰 필요 여부</text>
  <path d="M180,118 L260,50" class="sk-line" marker-end="url(#csa-m1)"/>
  <path d="M180,132 L260,132" class="sk-line-accent" marker-end="url(#csa-a1)"/>
  <path d="M180,150 L260,216" class="sk-line" marker-end="url(#csa-m1)"/>
  <rect x="262" y="22" width="450" height="56" rx="8" class="sk-box-muted"/>
  <text x="487" y="46" text-anchor="middle" class="sk-label">No authentication</text>
  <text x="487" y="64" text-anchor="middle" class="sk-sub">공개 정보만 · 링크를 아는 누구나 대화</text>
  <rect x="262" y="104" width="450" height="56" rx="8" class="sk-box-accent"/>
  <text x="487" y="128" text-anchor="middle" class="sk-label">Authenticate with Microsoft</text>
  <text x="487" y="146" text-anchor="middle" class="sk-sub">Teams·SharePoint·Power Apps·Copilot · 수동 설정 없음</text>
  <rect x="262" y="188" width="450" height="56" rx="8" class="sk-box"/>
  <text x="487" y="212" text-anchor="middle" class="sk-label">Authenticate manually</text>
  <text x="487" y="230" text-anchor="middle" class="sk-sub">User.AccessToken · 다른 채널 · 타사 OAuth</text>
</svg>
<figcaption>사내 Microsoft 채널이면 자동 인증, 토큰이나 다른 채널이 필요할 때만 수동으로 간다.</figcaption>
</figure>

## 판단표

| 항목 | No authentication | Authenticate with Microsoft | Authenticate manually |
|---|---|---|---|
| 로그인 | 없음 | Teams 등에서는 자동 | 공급자 설정에 따름 |
| 대화 가능한 사람 | 링크를 아는 누구나 | 공유 설정으로 조직 내 대상 제어 | Entra ID는 Require users to sign in을 켜면 공유로 제어, Generic OAuth 2는 특정 조직 사용자 제어 불가 |
| `User.Id`, `User.DisplayName` | 문서에 언급 없음 | 사용 가능 | 사용 가능 |
| `User.AccessToken` | 해당 없음 | 사용 불가 | 사용 가능 |
| `User.IsLoggedIn` | 해당 없음 | 웹 앱에서만 가능, Teams 플랜에서는 불가 | 사용 가능 |
| Require users to sign in | 해당 없음 | 해당 없음 | 사용 가능 |
| 못 쓰는 경우 | 관리자 데이터 정책이 인증을 요구할 때 | Dynamics 365 Customer Service 연동 에이전트 | 관리자 컨트롤이 설정을 고정한 경우 |

## 자주 막히는 지점

### 1. 설정을 바꿨는데 반영이 안 된다

인증 설정은 게시 후에야 적용됩니다. 저장만 하고 테스트하면 이전 설정으로 동작하는 것처럼 보입니다. 변경 후에는 게시 여부부터 확인합니다.

### 2. Microsoft 인증에서 수동으로 바꾸거나 반대로 바꿨더니 토픽이 깨진다

수동에서 Microsoft 인증으로 바꾸면 `User.AccessToken`을 쓰던 곳이 **Unknown** 변수가 됩니다. Teams 플랜에서는 `User.IsLoggedIn`도 마찬가지입니다. 변경 전에 토픽에서 해당 변수 사용처를 찾아 두고, 게시 전에 Topics 페이지에서 고칩니다.

### 3. 인증을 끄자 도구가 멈춘다

사용자 자격증명이 필요하도록 설정한 도구가 있다면, 에이전트 수준에서 인증을 끄지 말라고 문서가 경고합니다. 끄면 그 도구들이 동작하지 않습니다. MCP 서버 연결의 인증 방식은 [Copilot Studio에 MCP 서버 연결하기](/posts/copilot-studio-connect-mcp-server-guide/)에서 따로 정리했습니다.

### 4. Teams에 올리려는데 수동 인증이라 막힌다

내장 **Teams + Microsoft 365** 채널은 Authenticate with Microsoft를 쓰라고 안내됩니다. 수동 인증으로 Teams에 배포하려면 별도의 Teams SSO와 앱 매니페스트 요건을 따라야 합니다. 채널이 막히면 에이전트 상태의 Blocked status를 확인하라고 문서가 안내합니다.

### 5. 옵션이 회색이라 못 고른다

관리자가 Power Platform 관리 센터의 데이터 정책으로 인증을 요구하면 No authentication을 선택할 수 없고 Require users to sign in도 끌 수 없습니다. 관리자 컨트롤이 켜져 있으면 Authenticate manually 옵션을 Copilot Studio에서 바꿀 수 없습니다. 이 경우는 설정 문제가 아니라 정책이므로 관리자와 먼저 얘기합니다.

### 6. Scopes에 공백이 안 들어간다

Scope list delimiter에는 공백을 쓸 수 없습니다. 공급자가 공백 구분을 요구하면 delimiter에 쉼표(`,`)를 넣고 Scopes 필드에 공백을 넣으라고 문서가 안내합니다.

## 도입 전 점검표

- 에이전트가 올라갈 채널이 Teams·SharePoint·Power Apps·Copilot 중심인가, 웹 등 다른 채널도 포함하는가
- 토픽이나 도구에서 사용자 토큰이 필요한가
- 대화 가능한 조직 사용자를 공유 설정으로 제어해야 하는가(제어해야 하면 Generic OAuth 2는 맞지 않음)
- 관리 센터의 데이터 정책과 관리자 컨트롤 상태
- Scopes는 필요한 것만 최소 권한으로 잡았는가(문서의 권고)
- 에이전트 ID 쪽 관리 위치는 [Entra 에이전트 ID 운영 가이드](/posts/entra-agent-identity-admin-guide/)와 함께 볼 것

## 참고

- [Configure user authentication in Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/configuration-end-user-authentication)
- [Microsoft Entra ID authentication (Copilot Studio)](https://learn.microsoft.com/en-us/microsoft-copilot-studio/configuration-authentication-azure-ad)
- [Configure SSO with Microsoft Entra ID for agents in Teams](https://learn.microsoft.com/en-us/microsoft-copilot-studio/configure-sso-teams)
- [Add user authentication to topics](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-end-user-authentication)
- [Configure data policies for agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/admin-data-loss-prevention)
