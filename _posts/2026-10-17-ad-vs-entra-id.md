---
title: "Active Directory vs Entra ID — 'AD의 클라우드 버전'이 아닌 이유"
date: 2026-10-17 14:15:00 +0900
categories: [Dev Notes]
tags: [entra-id, active-directory, conditional-access, hybrid-identity, zero-trust]
description: ITProGuides의 5분 영상을 바탕으로 Active Directory와 Entra ID의 프로토콜·구조·정책·보안 경계 차이와, 하이브리드 환경에서 동기화 방향 때문에 자주 생기는 실수를 정리했다.
---

> 이 글은 ITProGuides의 "Active Directory vs Entra ID | What's the Difference? Explained in 5 Minutes"(2026-05-10 공개)를 보고 정리·재구성한 것이다. 그림 캡처는 영상에서 가져왔다.
{: .prompt-info }

{% include embed/youtube.html id='J65i3Ba68T0' %}

**결론부터.** Entra ID(구 Azure AD)는 "클라우드에 올린 Active Directory"가 아니다. 이름만 비슷한 **다른 제품**이다. AD는 "회사 건물 안"을 전제로 만들었고, Entra ID는 "어디서든"을 전제로 만들었다. 그래서 AD를 다루던 방식 그대로 Entra를 운영하면, 나중에 메우기 어려운 보안 구멍이 생긴다.

앞서 정리한 [SSO·SAML·OIDC 글](/posts/sso-saml-oidc/)과 [Agent 365 글](/posts/agent-365-agent-id/)이 모두 Entra ID를 전제로 한다. 그래서 그 바닥에 있는 "AD와 뭐가 다른가"를 한 번 짚어 둔다.

## 태어난 시대가 다르다

- **Active Directory (1999~)**: 모두가 사무실에 있고, 모든 게 사내망에 있던 시절의 설계다. 기본 가정은 **"네트워크 안에 들어왔으면 믿는다"**이다. 인증은 Kerberos와 NTLM을 쓴다. 사내망에서는 빠르고 안정적이지만, 인터넷을 건너가도록 만든 프로토콜은 아니다.
- **Entra ID**: 사용자는 집이나 출장지에 있고, 개인 기기를 쓰고, 앱은 회사 네트워크 밖(M365, Salesforce, ServiceNow…)에 있는 세상을 위한 설계다. 인증은 **OAuth 2.0, OpenID Connect, SAML** 같은 웹 프로토콜을 쓴다.

영상이 강조하는 한 줄은 이렇다. **Entra ID에는 도메인 컨트롤러가 없다. OU도, GPO도 없다.**

## 나란히 놓고 보면

![Active Directory와 Entra ID 비교 — 인증 프로토콜, 구조, 정책 도구, 카페에서 접속하는 사용자 예시](/assets/img/posts/ad-vs-entra-id/01-compare.jpg)
_왼쪽 AD, 오른쪽 Entra ID. 카페에 있는 같은 사용자가 Teams에는 HTTPS 토큰으로 바로 들어가고, 사내 파일 서버에는 VPN 없이는 못 들어간다. (출처: ITProGuides 영상)_

| 구분 | Active Directory | Entra ID |
|---|---|---|
| 인증 | Kerberos, NTLM | OAuth 2.0, OpenID Connect, SAML |
| 구조 | 계층형: 도메인 → OU → 하위 컨테이너 | 평면형: 보안 그룹, M365 그룹, 관리 단위(AU) |
| 설정 배포 | 그룹 정책(GPO) | Intune(기기 관리) |
| 접근 통제 | 네트워크 경계 + 권한 | 조건부 액세스 |
| MFA | 별도 도구가 필요 | 기본 내장(보안 기본값으로 테넌트 전체 적용 가능) |
| 보안 경계 | 네트워크 | 신원(Identity) |

영상의 카페 예시가 이 차이를 잘 보여 준다. 같은 사람, 같은 노트북이라도 Teams는 HTTPS 토큰으로 VPN 없이 열린다. 하지만 사내 파일 공유는 도메인 컨트롤러에 닿지 못해 열리지 않는다. Kerberos는 인터넷을 건너가지 못하기 때문이다. **같은 사용자에게 인증 경로가 두 개**인 셈이다.

**흔한 실수: OU 구조를 Entra에 그대로 옮기려는 것.** Entra는 평면 구조라 OU가 없다. 관리 위임 단위가 필요하면 관리 단위(Administrative Unit)를, 정책 대상은 그룹을 쓴다. 도구가 다르면 설계 방식도 달라야 한다.

## 보안 경계가 네트워크에서 신원으로

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="AD는 네트워크 경계 안이면 신뢰, Entra ID는 요청마다 신원과 맥락을 평가">
  <defs>
    <marker id="ave-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/></marker>
  </defs>
  <text x="0" y="16" class="sk-title">AD: 성벽 모델 vs Entra ID: 매번 검문 모델</text>
  <rect x="10" y="36" width="320" height="180" rx="12" class="sk-box-muted"/>
  <text x="170" y="58" text-anchor="middle" class="sk-label">사내망 (경계 = 네트워크)</text>
  <rect x="40" y="76" width="110" height="44" rx="8" class="sk-box"/>
  <text x="95" y="103" text-anchor="middle" class="sk-sub">사용자 PC</text>
  <rect x="190" y="76" width="110" height="44" rx="8" class="sk-box"/>
  <text x="245" y="103" text-anchor="middle" class="sk-sub">파일 서버</text>
  <path d="M152,98 L186,98" class="sk-line" marker-end="url(#ave-m1)"/>
  <rect x="40" y="146" width="260" height="52" rx="8" class="sk-box-accent"/>
  <text x="170" y="168" text-anchor="middle" class="sk-label">안에 들어오면 신뢰</text>
  <text x="170" y="186" text-anchor="middle" class="sk-sub">→ 침입 후 내부 이동(lateral movement)에 약함</text>
  <rect x="380" y="36" width="330" height="180" rx="12" class="sk-box-muted"/>
  <text x="545" y="58" text-anchor="middle" class="sk-label">어디서든 (경계 = 신원)</text>
  <rect x="400" y="76" width="90" height="44" rx="8" class="sk-box"/>
  <text x="445" y="103" text-anchor="middle" class="sk-sub">사용자</text>
  <rect x="510" y="70" width="80" height="56" rx="8" class="sk-box-accent"/>
  <text x="550" y="94" text-anchor="middle" class="sk-label">조건부</text>
  <text x="550" y="112" text-anchor="middle" class="sk-label">액세스</text>
  <rect x="610" y="76" width="80" height="44" rx="8" class="sk-box"/>
  <text x="650" y="103" text-anchor="middle" class="sk-sub">앱</text>
  <path d="M492,98 L506,98" class="sk-line" marker-end="url(#ave-m1)"/>
  <path d="M592,98 L606,98" class="sk-line" marker-end="url(#ave-m1)"/>
  <text x="545" y="152" text-anchor="middle" class="sk-sub">요청마다 묻는다: 누구? 어떤 기기?</text>
  <text x="545" y="170" text-anchor="middle" class="sk-sub">무엇에 접근? 평소와 같은 행동?</text>
  <text x="545" y="196" text-anchor="middle" class="sk-label">→ 허용 / MFA 요구 / 차단</text>
</svg>
<figcaption>AD는 성문을 지키는 모델, Entra ID는 문마다 검문하는 제로 트러스트 모델이다.</figcaption>
</figure>

AD에서는 일단 사내 LAN에 들어오면 할 수 있는 게 많다. 그래서 공격자가 한 대만 뚫어도 옆으로 퍼지는 **내부 이동 공격**에 특히 약하다.

Entra ID는 **제로 트러스트**를 전제로 한다. 네트워크가 아니라 신원이 경계다. 접근 요청이 올 때마다 누구인지, 어떤 기기인지, 무엇에 접근하려는지, 평소와 같은 행동인지를 평가한다. 이 판단을 정책으로 만드는 도구가 **조건부 액세스**다.

![조건부 액세스 — 누구인지, 어떤 기기인지, 무엇에 접근하는지, 정상 행동인지를 보고 직접 정책을 만든다](/assets/img/posts/ad-vs-entra-id/02-conditional-access.jpg)
_조건부 액세스는 비밀번호 하나가 아니라 '맥락'으로 접근을 결정한다._

영상의 정책 예시는 이렇다.
- 해외에서, 관리되지 않는 기기로 로그인 → **차단하거나 MFA 요구**
- Intune 규정을 지키는 회사 기기로, 회사 네트워크에서 로그인 → **추가 확인 없이 통과**

이런 맥락 기반 판단은 AD와 GPO만으로는 할 수 없다. 영상이 "지금 Microsoft 환경에서 가장 중요한 기술 중 하나"로 조건부 액세스를 꼽는 이유다.

## 하이브리드 — 대부분의 회사는 둘 다 쓴다

![하이브리드 — AD에서 Entra ID Connect로 Entra ID에 동기화, AD가 원본(Source of Truth)](/assets/img/posts/ad-vs-entra-id/03-hybrid.jpg)
_하이브리드에서는 AD가 원본이고, 변경은 AD → Entra 방향으로 흐른다._

현실에서는 둘 중 하나를 고르는 게 아니라 둘 다 돌린다. **Microsoft Entra Connect**(영상에서는 'Entra ID Connect')가 사내 AD 계정을 Entra로 동기화한다. 그래서 아이디·비밀번호 하나로 양쪽에 다 들어간다.

여기서 꼭 알아야 할 점이 있다.
- **원본(Source of Truth)은 AD다.** 기본 설정에서 변경은 AD → Entra 방향으로만 흐른다.
- **Entra에서만 만든 사용자는 사내 AD에 없다.** 그 사람은 클라우드 앱에는 들어가도 사내 파일 서버나 도메인 PC에는 못 들어간다.
- 동기화된 사용자의 속성을 Entra 쪽에서 고치면, 다음 동기화 때 AD 값으로 덮어쓰이거나 아예 수정이 막힌다.

영상은 이걸 하이브리드 환경에서 **가장 흔한 실수 중 하나**로 꼽는다. 바꾸기 전에 동기화 방향부터 확인하라는 것이다.

## 실무 메모

아래는 영상 내용 위에 운영자 관점으로 덧붙인 것이다.

- **"계정이 어디서 왔나"부터 본다.** Entra 관리 센터 사용자 목록에서 '온-프레미스 동기화' 여부를 먼저 확인하면, 수정 위치(AD냐 Entra냐)를 헷갈리지 않는다.
- **GPO를 Intune으로 옮길 때는 1:1 이식이 아니다.** 원래 GPO로 하던 일 중 무엇이 기기 설정(Intune)이고 무엇이 접근 통제(조건부 액세스)인지 먼저 나눈다.
- **조건부 액세스는 '보고 전용'으로 먼저 걸어 본다.** 바로 켜면 관리자까지 잠기는 사고가 난다. 비상 접근용 계정(break-glass)은 정책에서 꼭 제외해 둔다.
- **MFA는 보안 기본값으로라도 지금 켠다.** 라이선스가 넉넉하지 않은 작은 조직이라면 보안 기본값만으로도 테넌트 전체에 MFA를 바로 적용할 수 있다.
- **교육할 때는 '카페 예시'가 가장 잘 통한다.** 같은 노트북인데 Teams는 열리고 사내 파일 서버는 안 열리는 상황 하나로 두 인증 체계의 차이가 설명된다.

AD는 건물을 위해, Entra ID는 어디서든을 위해 만들어졌다. 경쟁 관계가 아니라 **서로 다른 시대의 두 신원 플랫폼**이고, 지금 현장에서는 둘 다 알아야 한다.

## 참고

- [ITProGuides — Active Directory vs Entra ID (YouTube)](https://www.youtube.com/watch?v=J65i3Ba68T0)
- [Microsoft Entra ID란? (Microsoft Learn)](https://learn.microsoft.com/entra/fundamentals/whatis)
- [Microsoft Entra Connect 개요 (Microsoft Learn)](https://learn.microsoft.com/entra/identity/hybrid/connect/whatis-azure-ad-connect)
- [조건부 액세스 개요 (Microsoft Learn)](https://learn.microsoft.com/entra/identity/conditional-access/overview)
