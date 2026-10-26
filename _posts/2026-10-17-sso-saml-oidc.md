---
title: "SSO는 어떻게 한 번 로그인으로 여러 앱을 여나 — SAML과 OIDC 흐름 정리"
date: 2026-10-17 11:10:00 +0900
categories: [Dev Notes]
tags: [identity, hands-on]
description: ByteByteGo의 SSO 설명 영상을 바탕으로 SAML 로그인 흐름, 두 번째 앱에서 로그인이 생략되는 이유, SAML과 OIDC의 차이를 정리하고 Entra ID 관점의 실무 포인트를 더했다.
---

결론부터. **SSO는 "비밀번호를 여러 앱이 공유하는 것"이 아니다.** 로그인은 신원 제공자(IdP) 한 곳에서만 하고, 각 앱은 IdP가 서명해 준 "이 사람 맞음" 증명서만 검증한다. 두 번째 앱부터 로그인 창이 안 뜨는 건 IdP가 이미 나를 알고 있어서다.

에이전트를 만들다 보면 Copilot Studio 인증 설정이든 Graph 호출이든 결국 이 구조 위에서 막힌다. 그래서 한 번 제대로 정리해 두기로 했다.

> 이 글은 ByteByteGo의 영상 「What Is Single Sign-on (SSO)? How It Works」를 정리·재구성하고, Microsoft Learn 문서로 Entra ID 관련 내용을 보충한 것이다.
{: .prompt-info }

## 등장인물은 셋

- **사용자(브라우저)**: 앱에 들어가려는 사람
- **서비스 제공자(SP)**: Gmail, Workday, Slack 같은 앱
- **신원 제공자(IdP)**: 로그인을 대신 처리하는 곳. 영상은 Okta, Auth0, OneLogin을 예로 들고, 내가 매일 쓰는 건 Microsoft Entra ID다

이 구조의 바탕 개념이 **연합 신원(federated identity)** 이다. 서로 독립적이지만 신뢰 관계를 맺은 시스템끼리 신원 정보를 주고받는 방식이다.

## SAML 로그인 흐름 — 첫 번째 앱

<figure class="sketch">
<svg viewBox="0 0 720 330" role="img" aria-label="SAML 첫 로그인 흐름: 브라우저가 앱에서 IdP로 리디렉션되고, IdP 로그인 후 서명된 SAML 어설션을 받아 앱에 전달한다">
  <defs>
    <marker id="sso-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="sso-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 첫 앱(Gmail) 로그인 — SAML</text>
  <rect x="20" y="40" width="150" height="50" rx="8" class="sk-box"/>
  <text x="95" y="62" text-anchor="middle" class="sk-label">브라우저</text>
  <text x="95" y="80" text-anchor="middle" class="sk-sub">사용자</text>
  <rect x="285" y="40" width="150" height="50" rx="8" class="sk-box"/>
  <text x="360" y="62" text-anchor="middle" class="sk-label">서비스 제공자</text>
  <text x="360" y="80" text-anchor="middle" class="sk-sub">Gmail (SP)</text>
  <rect x="550" y="40" width="150" height="50" rx="8" class="sk-box-accent"/>
  <text x="625" y="62" text-anchor="middle" class="sk-label">신원 제공자</text>
  <text x="625" y="80" text-anchor="middle" class="sk-sub">IdP (Okta·Entra ID)</text>
  <line x1="95" y1="92" x2="95" y2="320" class="sk-line" stroke-dasharray="3 4"/>
  <line x1="360" y1="92" x2="360" y2="320" class="sk-line" stroke-dasharray="3 4"/>
  <line x1="625" y1="92" x2="625" y2="320" class="sk-line" stroke-dasharray="3 4"/>
  <path d="M97,118 L356,118" class="sk-line" marker-end="url(#sso-m1)"/>
  <text x="226" y="112" text-anchor="middle" class="sk-sub">1. Gmail 접속</text>
  <path d="M358,150 L99,150" class="sk-line" marker-end="url(#sso-m1)"/>
  <text x="226" y="144" text-anchor="middle" class="sk-sub">2. 회사 도메인 감지 → SAML 인증 요청</text>
  <path d="M97,182 L621,182" class="sk-line" marker-end="url(#sso-m1)"/>
  <text x="360" y="176" text-anchor="middle" class="sk-sub">3. IdP로 리디렉션 → 여기서만 아이디·비밀번호 입력</text>
  <path d="M623,222 L99,222" class="sk-line-accent" marker-end="url(#sso-a1)"/>
  <text x="360" y="216" text-anchor="middle" class="sk-mono">4. 서명된 SAML 어설션 (XML)</text>
  <path d="M97,262 L356,262" class="sk-line-accent" marker-end="url(#sso-a1)"/>
  <text x="226" y="256" text-anchor="middle" class="sk-sub">5. 어설션 전달</text>
  <path d="M358,302 L99,302" class="sk-line" marker-end="url(#sso-m1)"/>
  <text x="226" y="296" text-anchor="middle" class="sk-sub">6. 공개키로 서명 검증 → 허용된 범위만 열어 줌</text>
</svg>
<figcaption>비밀번호는 IdP에만 간다. 앱(SP)은 IdP가 서명한 증명서만 검증한다.</figcaption>
</figure>

순서대로 보면 이렇다.

1. 회사 직원이 Gmail에 접속한다. 이 예에서 Gmail이 **서비스 제공자(SP)** 다.
2. Gmail 서버가 회사 도메인 사용자임을 알아채고, 브라우저에 **SAML 인증 요청**을 돌려준다.
3. 브라우저가 요청에 적힌 회사 IdP로 이동한다. 로그인 화면은 **IdP가** 띄운다.
4. 인증이 끝나면 IdP가 **SAML 어설션**을 만들어 브라우저에 준다. 사용자 정보와 "이 앱에서 무엇을 할 수 있는지"를 담은 XML 문서이고, 암호학적으로 서명돼 있다.
5. 브라우저가 이 어설션을 Gmail에 전달한다.
6. Gmail은 어설션이 정말 그 IdP가 서명한 것인지 **공개키 암호로 검증**하고, 어설션에 적힌 권한만큼 열어 준다.

## 두 번째 앱(Workday)에서는 왜 로그인이 안 뜰까

<figure class="sketch">
<svg viewBox="0 0 720 170" role="img" aria-label="두 번째 앱 접속 시 IdP에 이미 로그인돼 있어 로그인 단계를 건너뛰고 새 어설션만 발급한다">
  <defs>
    <marker id="sso-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="sso-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 두 번째 앱(Workday) — 로그인 단계가 빠진다</text>
  <rect x="20" y="50" width="130" height="46" rx="8" class="sk-box"/>
  <text x="85" y="78" text-anchor="middle" class="sk-label">Workday 접속</text>
  <rect x="200" y="50" width="130" height="46" rx="8" class="sk-box"/>
  <text x="265" y="72" text-anchor="middle" class="sk-label">SAML 요청</text>
  <text x="265" y="88" text-anchor="middle" class="sk-sub">→ IdP로 이동</text>
  <rect x="380" y="50" width="150" height="46" rx="8" class="sk-box-muted"/>
  <text x="455" y="72" text-anchor="middle" class="sk-label">로그인 화면</text>
  <text x="455" y="88" text-anchor="middle" class="sk-sub">이미 로그인 → 생략</text>
  <rect x="580" y="50" width="130" height="46" rx="8" class="sk-box-accent"/>
  <text x="645" y="72" text-anchor="middle" class="sk-label">새 어설션</text>
  <text x="645" y="88" text-anchor="middle" class="sk-sub">Workday용 권한</text>
  <path d="M152,73 L196,73" class="sk-line" marker-end="url(#sso-m2)"/>
  <path d="M332,73 L376,73" class="sk-line" marker-end="url(#sso-m2)"/>
  <path d="M532,73 L576,73" class="sk-line-accent" marker-end="url(#sso-a2)"/>
  <text x="360" y="140" text-anchor="middle" class="sk-sub">흐름은 같다. 달라지는 건 IdP가 나를 이미 알고 있다는 것, 그리고 어설션이 앱마다 새로 만들어진다는 것.</text>
</svg>
<figcaption>"한 번 로그인"의 정체는 앱이 나를 기억하는 게 아니라 IdP가 나를 기억하는 것이다.</figcaption>
</figure>

Workday도 Gmail과 똑같이 회사 도메인을 감지해 SAML 요청을 보내고, 브라우저는 다시 IdP로 간다. 차이는 하나다. **IdP에 이미 로그인한 상태라 로그인 단계를 건너뛰고**, Workday용 권한을 담은 새 어설션만 발급한다. 나머지 검증 과정은 같다.

여기서 내가 처음에 헷갈렸던 점은, 어설션이 "통행증 한 장"이 아니라 **앱마다 따로 발급된다는 것**이다. 그래서 앱별로 줄 수 있는 권한을 다르게 관리할 수 있다.

## SAML vs OIDC — 포장만 다르다

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="SAML은 서명된 XML 어설션, OIDC는 서명된 JSON 토큰(JWT)을 쓴다. 인증 판단은 같고 포장만 다르다">
  <text x="0" y="14" class="sk-title">③ 같은 판단, 다른 포장</text>
  <rect x="20" y="40" width="320" height="140" rx="10" class="sk-box"/>
  <text x="180" y="66" text-anchor="middle" class="sk-label">SAML 2.0</text>
  <text x="40" y="94" class="sk-mono">서명된 XML 어설션</text>
  <text x="40" y="118" class="sk-sub">· 오래된 엔터프라이즈 표준</text>
  <text x="40" y="138" class="sk-sub">· 회사 업무 앱(Workday 등)에서 흔함</text>
  <text x="40" y="158" class="sk-sub">· 사용자 속성을 세밀하게 담는다</text>
  <rect x="380" y="40" width="320" height="140" rx="10" class="sk-box-accent"/>
  <text x="540" y="66" text-anchor="middle" class="sk-label">OpenID Connect (OIDC)</text>
  <text x="400" y="94" class="sk-mono">서명된 JSON 토큰 (JWT)</text>
  <text x="400" y="118" class="sk-sub">· OAuth 2.0 위에 얹은 신원 계층</text>
  <text x="400" y="138" class="sk-sub">· "구글로 로그인"이 바로 이것</text>
  <text x="400" y="158" class="sk-sub">· 웹·모바일·API에 다루기 쉽다</text>
</svg>
<figcaption>보안 수준의 차이가 아니라, 어떤 앱에 붙이기 쉬우냐의 차이다.</figcaption>
</figure>

- **SAML**: XML 기반 개방형 표준. 회사 업무 환경에서 주로 본다.
- **OIDC**: 개인 구글 계정으로 YouTube에 로그인할 때 쓰는 그 방식. 서명된 XML 대신 **JWT(JSON Web Token)** 를 주고받는다.

영상의 결론은 **둘 다 안전하다**는 것이다. 신원 관리를 Okta 같은 상용 플랫폼에 맡긴 기업이라면 대부분 둘 다 지원하니, 붙이려는 앱이 어느 쪽을 쉽게 지원하느냐로 고르면 된다. 새 웹 앱을 만든다면 Google, Facebook, GitHub 같은 OIDC 제공자에 붙이는 게 무난하다고 한다.

## Entra ID로 보면 — 에이전트를 만든다면 OIDC 쪽이다

영상에는 없는 부분인데, 내 환경(Microsoft Entra ID)에 대입해 Microsoft Learn 문서를 찾아봤다.

- Entra ID는 SAML이든 OIDC든 **사용자 인증 → 조건부 액세스 같은 정책 검사 → 담을 신원 정보 결정**까지 같은 단계를 거친다. 갈라지는 건 그 뒤 **포장 방식**뿐이다. SAML 앱은 XML 어설션을, OIDC 앱은 JWT를 받는다.
- 실무에서 중요한 차이가 하나 있다. **SAML 토큰으로는 Microsoft Graph를 호출할 수 없다.** Graph는 OIDC/OAuth로 받은 JWT가 필요하다. 그래서 로그인은 SAML로 하는 솔루션이라도 Graph를 쓰려면 OIDC 연동을 따로 붙여야 한다고 문서에 나와 있다.

> 메일·일정·SharePoint를 읽는 에이전트처럼 **Graph를 써야 하는 앱이라면 처음부터 OIDC로 설계하는 게 맞다.** 로그인만 SAML로 붙였다가 나중에 Graph 연동에서 다시 공사하는 경우가 생긴다.
{: .prompt-tip }

## 정리하며

예전에는 SSO를 "회사 계정 하나로 다 들어가는 편한 기능" 정도로만 알았다. 흐름을 따라가 보니 핵심은 편의가 아니라 **신뢰의 위치**였다. 비밀번호는 IdP 한 곳에만 있고, 앱들은 서명만 믿는다. 그래서 IdP 한 곳에 MFA와 조건부 액세스를 걸면 모든 앱이 같이 안전해진다.

다음에 Copilot Studio 에이전트에 사용자 인증을 붙일 때는 "이 에이전트가 Graph를 부르는가?"부터 묻고 프로토콜을 고를 생각이다.

## 참고

- ByteByteGo, 「What Is Single Sign-on (SSO)? How It Works」 (2022-11-17) — <https://www.youtube.com/watch?v=O1cRJWYF-g4>
- Microsoft Learn, 「Understand Microsoft's SSO model」 — <https://learn.microsoft.com/entra/identity/enterprise-apps/understand-microsoft-sso-model>
- Microsoft Learn, 「SAML versus OpenID Connect: Choose the right SSO protocol」 — <https://learn.microsoft.com/entra/identity/enterprise-apps/saml-vs-oidc-decision-guide>
- Microsoft Learn, 「Secure hybrid access with Microsoft Entra integration」 (SAML 토큰과 Graph) — <https://learn.microsoft.com/entra/identity/enterprise-apps/secure-hybrid-access-integrations>
