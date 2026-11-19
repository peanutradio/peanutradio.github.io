---
title: "Entra 에이전트 ID 운영 가이드 — 유형 고르기와 관리 위치"
date: 2026-11-20 06:53:00 +0900
categories: [AI Agent]
tags: [ai-agent, identity, governance, security]
description: 에이전트에 어떤 신원을 줄지(에이전트 ID만 vs 사용자 계정까지) 고르는 기준과 Entra 관리 위치, 역할, 막히는 지점을 Microsoft Learn 문서 기준으로 정리한다.
---

> 이 글은 Microsoft Learn의 Entra Agent ID 문서들과 Foundry 블로그의 자율 에이전트 글을 읽고 정리·재구성한 가이드다. 직접 구성해 본 기록이 아니라 문서 기준 순서이며, 문서에 없는 설정 경로는 쓰지 않았다.
{: .prompt-info }

## 결론 3줄

- 에이전트가 사용자 대신 API만 부르면 **에이전트 ID**로 충분하고, 메일함·Teams 같은 Microsoft 365 작업이 필요할 때만 **에이전트 사용자 계정**을 추가한다.
- 에이전트 사용자 계정은 자동으로 만들어지지 않고, Graph API로만 만들면 신원만 생기고 Microsoft 365 기능은 붙지 않는다.
- 운영 시작점은 사람 **스폰서** 지정이다. 관리 화면은 Entra 관리 센터의 **Agents** 메뉴다.

<figure class="sketch">
<svg viewBox="0 0 720 280" role="img" aria-label="블루프린트 하나에서 인스턴스마다 에이전트 ID와 사용자 계정이 생기는 구조">
  <defs>
    <marker id="eai-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="eai-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 블루프린트 → 인스턴스 → 신원</text>
  <rect x="8" y="100" width="170" height="70" rx="8" class="sk-box-accent"/>
  <text x="93" y="128" text-anchor="middle" class="sk-label">에이전트 블루프린트</text>
  <text x="93" y="148" text-anchor="middle" class="sk-sub">할 수 있는 일의 정의</text>
  <path d="M180,120 L260,70" class="sk-line-accent" marker-end="url(#eai-a1)"/>
  <path d="M180,150 L260,200" class="sk-line-accent" marker-end="url(#eai-a1)"/>
  <rect x="262" y="40" width="200" height="60" rx="8" class="sk-box"/>
  <text x="362" y="66" text-anchor="middle" class="sk-label">인스턴스 A</text>
  <text x="362" y="86" text-anchor="middle" class="sk-sub">에이전트 ID + 사용자 계정</text>
  <rect x="262" y="170" width="200" height="60" rx="8" class="sk-box"/>
  <text x="362" y="196" text-anchor="middle" class="sk-label">인스턴스 B</text>
  <text x="362" y="216" text-anchor="middle" class="sk-sub">에이전트 ID + 사용자 계정</text>
  <path d="M464,70 L530,70" class="sk-line" marker-end="url(#eai-m1)"/>
  <path d="M464,200 L530,200" class="sk-line" marker-end="url(#eai-m1)"/>
  <rect x="532" y="45" width="180" height="50" rx="8" class="sk-box-muted"/>
  <text x="622" y="75" text-anchor="middle" class="sk-label">사람 스폰서</text>
  <rect x="532" y="175" width="180" height="50" rx="8" class="sk-box-muted"/>
  <text x="622" y="205" text-anchor="middle" class="sk-label">사람 스폰서</text>
</svg>
<figcaption>블루프린트는 권한이 아니라 능력의 정의이고, 업무 자원 접근은 인스턴스별 신원에서 갈린다.</figcaption>
</figure>

## 용어부터 — 문서 기준

### 에이전트 ID

Entra ID 안에서 에이전트를 식별하고 인증하는 신원이다. Foundry 문서는 모든 Foundry 에이전트가 만들어지는 순간부터 Entra 에이전트 ID를 가진다고 적는다. 구현상 서비스 주체이고, 에이전트 ID·이름·스폰서·인증에 쓰는 권한을 담는다.

### 에이전트 사용자 계정

Entra의 사용자 신원 하위 유형이다. 선택 사항이고 자동 생성되지 않는다. 에이전트 ID와 1:1로만 연결되고 한번 연결하면 바꿀 수 없다. 암호나 패스키 같은 자격 증명을 가질 수 없고, 연결된 에이전트 ID가 받은 토큰을 제시해야만 인증된다. 권한은 보통 게스트 사용자와 비슷한 수준이다.

### 블루프린트

에이전트가 어떤 API를 부를 수 있는지 같은 능력을 정의한다. 특정 팀의 업무 자원 접근을 기본으로 허용하지는 않는다고 문서가 적었다. 자율 에이전트(autopilot)로 게시할 수 있는 것은 Foundry 호스티드 에이전트뿐이다.

## 단계별 순서

문서에서 확인된 것은 만드는 절차가 아니라 **관리하는 절차**다. 블루프린트 생성의 메뉴 경로는 읽은 문서에서 확인되지 않아 싣지 않는다.

1. 유형을 정한다. 문서의 세 유형은 Assistive(로그인한 사용자 컨텍스트), Background service(앱 전용, Microsoft 365 작업 불가), Autopilot(에이전트 ID + 사용자 계정)이다.
2. 사람 스폰서를 정한다. 스폰서는 에이전트의 목적과 수명 주기에 책임지고, 기술 관리는 오너가 맡는다.
3. [Microsoft Entra 관리 센터](https://entra.microsoft.com)에서 **Entra ID** > **Agents** > **Agent identities**로 에이전트를 확인한다.
4. **Entra ID** > **Agents** > **Agent blueprints**에서 블루프린트를 확인한다. 생성하려면 Agent ID Developer 역할이 필요하고, 만든 사람이 오너가 된다.
5. Microsoft 365 작업이 필요하면 사용자 계정을 만든다. 문서는 완전한 프로비저닝(메일함, Teams 표시, HR 연동)을 하려면 Microsoft Teams를 통해 에이전트를 만들라고 안내한다. 블루프린트에는 사용자 계정 생성 권한이 기본으로 없으며 명시적으로 부여해야 한다.
6. 감사는 **Entra ID** > **Monitoring & health** > **Sign-in logs**에서 "Agent type", "Is Agent"로 거른다. 최소 Reports Reader 역할이 필요하다.

## 판단표

| 질문 | 에이전트 ID만 | 에이전트 ID + 사용자 계정 |
|---|---|---|
| 하는 일 | API 호출, 사용자 위임 접근 | 메일·Teams 등 사용자처럼 활동 |
| 자동 생성 | Foundry 에이전트는 생성 시 부여 | 아니오, 명시적 생성 |
| 자격 증명 | 문서의 인증 경로 | 직접 부여 금지, 토큰 제시로만 |
| Microsoft 365 기능 | 앱 전용은 불가 | Teams 경유 생성 시 가능 |
| 라이선스 | 모든 Entra 고객에게 제공 | M365 작업은 사용자별 Agent 365 라이선스 필요 |

라이선스 항목은 문서 기준으로, Agent 365는 Microsoft 365 E7에 포함되고 E5/A5/Business Premium의 추가 항목으로 제공된다고 적혀 있다. 에이전트 대상 ID Protection은 프리뷰 기간 중 Entra ID P2가 필요하다.

## 역할 요약

| 하려는 일 | 필요한 역할(문서 표기) |
|---|---|
| 에이전트 보기 | Entra 사용자 계정(관리자 역할 불필요) |
| 에이전트 관리 | Agent ID Administrator 또는 Cloud Application Administrator |
| 블루프린트 생성 | Agent ID Developer |
| 조건부 액세스 | Conditional Access Administrator (Entra ID P1) |
| 위험 보고서 | Security Administrator, Operator, Reader |

## 자주 막히는 지점

### 1. Graph로만 사용자 계정을 만들었는데 메일함이 없다

문서가 직접 경고하는 지점이다. Graph API로 만들면 Entra 신원만 생기고 Microsoft 365 기능은 프로비저닝되지 않는다. 디지털 직원처럼 쓸 에이전트는 Teams를 통해 만든다.

### 2. 사용자 계정에 자격 증명을 넣으려 한다

연동 편의 때문에 페더레이션 자격 증명이나 암호, 인증서를 계정에 직접 붙이면 안 된다. 인증은 연결된 에이전트 ID가 받은 토큰으로만 한다.

### 3. 블루프린트를 끄면 기존 에이전트도 멈춘다

블루프린트 비활성화는 새 에이전트 생성뿐 아니라 기존 에이전트도 막는다. 에이전트 ID 전체 비활성화도 기존 에이전트 실패를 부를 수 있다고 한다. 침해 의심 때는 자격 증명을 교체한 뒤 다시 켜거나 에이전트 ID를 폐기하라는 안내가 있다.

### 4. 조건부 액세스가 모든 토큰에 적용된다고 생각한다

적용 대상은 에이전트 ID나 사용자 계정이 리소스 토큰을 요청할 때다. 블루프린트가 에이전트 ID를 만들려고 토큰을 얻을 때는 적용되지 않는다.

### 5. 상속 권한을 선언만 하고 끝낸다

필수 리소스 접근은 동의 검토용 선언일 뿐 접근을 직접 부여하지 않는다. 문서는 애플리케이션 역할 상속은 Graph 지원 여부를 먼저 확인하라고 적었다.

### 6. 스폰서가 퇴사한다

Entra ID가 스폰서를 그 사람의 관리자에게 자동 이전한다. 그래도 이전된 관리자가 새 책임을 아는지 점검 항목에 넣어야 한다.

## 도입 전 점검표

- 에이전트마다 사람 스폰서가 지정돼 있는가
- 사용자 계정이 정말 필요한가(위임 접근으로 충분하지 않은가)
- Agent 365 라이선스 대상 사용자가 정리됐는가
- 조건부 액세스, ID Protection 대상 범위와 라이선스(P1/P2)가 맞는가
- 블루프린트 비활성화 시 영향받는 에이전트 목록을 알고 있는가

## 확인되지 않은 것

블루프린트와 자율 에이전트의 GA·프리뷰 상태, 그리고 생성 절차의 메뉴 경로는 읽은 문서에서 명시되지 않았다. 도입 직전에 Learn 문서와 Foundry 블로그의 최신 상태를 다시 확인해야 한다. 관련 맥락은 [Foundry 호스티드 에이전트 글](/posts/foundry-hosted-agents-ignite-2026-egress-sessions/)에서 볼 수 있다.

## 참고

- What are agent identities?: https://learn.microsoft.com/en-us/entra/agent-id/what-are-agent-identities
- Agent's user account in Microsoft Entra Agent ID: https://learn.microsoft.com/en-us/entra/agent-id/agent-users
- Manage agent identities in your organization: https://learn.microsoft.com/en-us/entra/agent-id/manage-agent-identities-admin
- What is an autopilot in Microsoft Foundry?: https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/autopilot-overview
- Foundry 블로그, Build trusted autonomous agents for your team: https://devblogs.microsoft.com/foundry/build-trusted-autonomous-agents-for-your-team-with-microsoft-foundry/
