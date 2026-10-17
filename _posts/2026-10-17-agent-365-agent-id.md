---
title: "Agent 365와 Agent ID 정리 — 에이전트에게도 '사원증'이 필요한 이유"
date: 2026-10-17 13:40:00 +0900
categories: [Copilot Studio]
tags: [agent-365, agent-id, entra-id, purview, defender, ai-governance]
description: John Savill의 Agent 365 개요 영상을 바탕으로 Agent ID(블루프린트·에이전트 ID·에이전트 사용자)의 구조, 토큰 흐름, Entra·Purview·Defender가 에이전트를 다루는 방식, 레지스트리와 컬렉션을 정리했다.
---

> 이 글은 John Savill's Technical Training의 "Agent 365 and Agent ID Overview"(2026-02-23 공개, 48분)를 보고 정리·재구성한 것이다. 화면 캡처는 영상에서 가져왔고 데모 테넌트 로고는 가렸다. 영상 속 화면은 대부분 프리뷰 단계라 메뉴 이름과 위치가 지금은 다를 수 있다.
{: .prompt-info }

{% include embed/youtube.html id='WTcyL68qTo8' %}

**결론부터.** Agent 365(A365)는 새 제품이 하나 더 생긴 게 아니다. 사람 직원에게 이미 쓰고 있는 Entra(신원)·Purview(데이터)·Defender(위협)·Microsoft 365(업무 도구)를 **AI 에이전트에게도 그대로 적용하도록 늘린 묶음**이다. 그 출발점은 **에이전트마다 고유한 신원, 즉 Agent ID를 주는 것**이다. 신원이 있어야 권한도 주고, 감사도 하고, 막을 수도 있다.

Copilot Studio로 에이전트를 몇 개 만들다 보면 금방 이런 질문이 나온다. "이 에이전트는 누구 권한으로 돌아가지?" "누가 책임지지?" "몇 개나 있지?" A365는 바로 이 질문들에 답하려고 만든 것이다.

## 왜 지금 이게 필요한가

영상이 드는 숫자는 이렇다. 2028년까지 에이전트가 10억 개를 훨씬 넘을 것으로 본다. Microsoft 한 회사만 해도 매주 5,000개씩 늘고 있고, 2026년 4월쯤이면 에이전트 수가 직원 수를 넘는다고 한다. 조직마다 속도는 달라도 같은 길을 간다.

문제는 **모르는 것은 지킬 수 없다**는 점이다. 어떤 에이전트가 있는지 모르면 활용도 못 하고, 최소 권한으로 도는지, 어떤 데이터를 보고 어디에 공유하는지도 확인할 수 없다.

Savill은 이걸 매슬로의 욕구 단계에 빗대 설명한다. 아래 단계가 채워져야 위 단계로 갈 수 있다.

<figure class="sketch">
<svg viewBox="0 0 640 250" role="img" aria-label="에이전트의 욕구 단계: 신원, 보안과 가드레일, 목적 적합성, 업무 통합">
  <text x="0" y="16" class="sk-title">① 에이전트의 욕구 단계 (아래부터 채운다)</text>
  <path d="M120,230 L520,230 L470,180 L170,180 Z" class="sk-box-accent"/>
  <text x="320" y="211" text-anchor="middle" class="sk-label">1. 신원 — Agent ID</text>
  <path d="M170,176 L470,176 L425,131 L215,131 Z" class="sk-box"/>
  <text x="320" y="159" text-anchor="middle" class="sk-label">2. 보안 · 가드레일</text>
  <path d="M215,127 L425,127 L380,82 L260,82 Z" class="sk-box"/>
  <text x="320" y="110" text-anchor="middle" class="sk-label">3. 목적 적합 · 생산성</text>
  <path d="M260,78 L380,78 L320,32 Z" class="sk-box-muted"/>
  <text x="320" y="68" text-anchor="middle" class="sk-sub">4. 업무 통합</text>
  <text x="540" y="211" class="sk-sub">이게 없으면</text>
  <text x="540" y="227" class="sk-sub">위는 전부 불가</text>
</svg>
<figcaption>신원이 있어야 보안을 걸 수 있고, 그다음에야 "쓸모 있나"와 "업무 흐름에 녹아드나"를 따질 수 있다.</figcaption>
</figure>

## 사람 직원에게 하던 걸 에이전트에게도

영상의 핵심 구도는 단순하다. 먼저 사람 직원에게 무엇을 해 주는지 정리한 뒤, 같은 칸에 에이전트를 넣어 본다.

| 필요한 것 | 사람 직원에게는 | 에이전트에게는 (A365) |
|---|---|---|
| 신원 | Entra 사용자 계정 | **Agent ID** (필요하면 Agent User까지) |
| 권한·통제 | RBAC, 자격 관리, 조건부 액세스, 위험 감지 | 같은 기능의 대상에 에이전트가 추가됨 |
| 찾기 | 전체 주소 목록(GAL) | **에이전트 레지스트리** |
| 데이터 보호 | Purview(레이블, DLP, 내부자 위험) | 에이전트의 행동 패턴을 이해하는 Purview |
| 위협 보호 | Defender | AI 전용 위협 보호(탈옥, 프롬프트 인젝션 등) |
| 생산성 | M365 라이선스(메일·Teams·SharePoint) | **A365 라이선스** + Work IQ |

핵심은 **도구를 따로 만들지 않는다**는 점이다. 별도 콘솔, 별도 프로세스를 새로 만들지 않고, 쓰던 도구를 에이전트의 행동 방식에 맞게 손본다.

에이전트 종류는 세 가지로 나눈다. 내가 말을 걸면 나 대신 일하는 **비서형**, 일정·메일 도착·웹훅 같은 이벤트에 반응해 스스로 도는 **자율형**, 그리고 동료처럼 함께 일하는 **AI 팀원**이다. 오른쪽으로 갈수록 에이전트 자신의 신원이 더 중요해진다.

## 왜 '새로운 종류'의 신원인가

기존 신원을 쓰면 안 되는 이유가 두 가지 있다.

- **앱용 서비스 주체(service principal)**: 할 수 있는 일이 너무 제한적이다. 게다가 앱은 오래 살지만 에이전트는 수시로 생겼다 사라진다.
- **일반 사용자 계정**: 사용자는 실제 사람을 전제로 한다. 에이전트는 휴대폰을 꺼내 MFA 승인을 누를 수 없다. 결국 예외를 잔뜩 만들게 되고, 그 예외가 새로운 구멍이 된다.

그래서 에이전트 전용 신원을 새로 만들었다. **같은 에이전트를 20개 띄우면 신원도 20개다.** 그리고 이 신원들을 묶어서 관리할 계층 구조가 필요하다. 그 구조가 다음 절의 블루프린트다.

## Agent ID의 구조 — 블루프린트에서 에이전트까지

<figure class="sketch">
<svg viewBox="0 0 660 330" role="img" aria-label="블루프린트, 블루프린트 서비스 주체, 에이전트 ID, 에이전트 사용자의 관계">
  <defs>
    <marker id="a365-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/></marker>
  </defs>
  <text x="0" y="16" class="sk-title">② Agent ID 계층 구조</text>
  <rect x="10" y="34" width="250" height="118" rx="10" class="sk-box-muted"/>
  <text x="135" y="54" text-anchor="middle" class="sk-sub">게시 테넌트 (만든 쪽)</text>
  <rect x="30" y="64" width="210" height="78" rx="8" class="sk-box-accent"/>
  <text x="135" y="84" text-anchor="middle" class="sk-label">블루프린트 (템플릿)</text>
  <text x="135" y="102" text-anchor="middle" class="sk-sub">설명 · 앱 권한 · 위임 권한</text>
  <text x="135" y="120" text-anchor="middle" class="sk-sub">인증 수단은 여기에만</text>
  <text x="135" y="135" text-anchor="middle" class="sk-mono">관리 ID / 인증서</text>
  <rect x="330" y="34" width="320" height="286" rx="10" class="sk-box-muted"/>
  <text x="490" y="54" text-anchor="middle" class="sk-sub">사용하는 테넌트 (우리 회사)</text>
  <path d="M262,103 L376,103" class="sk-line" marker-end="url(#a365-m1)"/>
  <text x="318" y="96" text-anchor="middle" class="sk-sub">동의</text>
  <rect x="380" y="76" width="220" height="52" rx="8" class="sk-box"/>
  <text x="490" y="98" text-anchor="middle" class="sk-label">블루프린트 서비스 주체</text>
  <text x="490" y="116" text-anchor="middle" class="sk-sub">에이전트 ID를 만들 권한</text>
  <path d="M440,130 L400,170" class="sk-line" marker-end="url(#a365-m1)"/>
  <path d="M540,130 L580,170" class="sk-line" marker-end="url(#a365-m1)"/>
  <rect x="345" y="172" width="125" height="58" rx="8" class="sk-box-accent"/>
  <text x="407" y="194" text-anchor="middle" class="sk-label">에이전트 ID #1</text>
  <text x="407" y="212" text-anchor="middle" class="sk-sub">자격증명 없음</text>
  <rect x="515" y="172" width="125" height="58" rx="8" class="sk-box-accent"/>
  <text x="577" y="194" text-anchor="middle" class="sk-label">에이전트 ID #2</text>
  <text x="577" y="212" text-anchor="middle" class="sk-sub">+ 추가 권한</text>
  <path d="M407,232 L407,256" class="sk-line-accent" marker-end="url(#a365-m1)"/>
  <rect x="345" y="258" width="190" height="52" rx="8" class="sk-box"/>
  <text x="440" y="279" text-anchor="middle" class="sk-label">에이전트 사용자 (선택)</text>
  <text x="440" y="297" text-anchor="middle" class="sk-sub">메일함 · Teams · SharePoint</text>
  <text x="548" y="270" class="sk-sub">모든 에이전트 ID에</text>
  <text x="548" y="286" class="sk-sub">스폰서(책임자)</text>
  <text x="548" y="302" class="sk-sub">필수</text>
</svg>
<figcaption>앱 등록 → 엔터프라이즈 앱(서비스 주체) 구조를 아는 사람에게는 익숙한 패턴이다. 블루프린트가 앱 등록, 블루프린트 서비스 주체가 엔터프라이즈 앱 자리에 온다.</figcaption>
</figure>

각 단계를 하나씩 보면 이렇다.

1. **블루프린트(Blueprint)**: 에이전트 신원의 템플릿이다. 설명과 기본 권한을 담는다. 권한은 에이전트가 혼자 쓰는 앱 권한(예: 메일 읽기, Azure 리소스 접근)과, 사용자가 동의하면 그 사람 대신 쓰는 위임 권한으로 나뉜다. 여기서 정한 권한은 아래 에이전트 ID들이 물려받는다.
2. **에이전트에게 줄 수 없는 권한이 있다.** 예로 `User.ReadWrite.All`이 나온다. 에이전트가 폭주해서 사용자 계정을 다 지우는 일을 막으려는 것이다. 금지 목록은 문서로 공개돼 있다.
3. **인증은 블루프린트에서만 한다.** Copilot Studio, Agent Builder, Foundry 같은 Microsoft 플랫폼에서는 관리 ID처럼 플랫폼에 기본으로 붙어 있다. 직접 구성한다면 인증서 같은 페더레이션 방식을 쓴다. 기술적으로는 비밀(secret)도 되지만 Savill은 "쓰지 말라"고 강하게 말한다. **에이전트 ID 자체는 자격증명을 갖지 않는다.**
4. **동의(Consent) → 블루프린트 서비스 주체**: 다른 회사가 만든 멀티 테넌트 에이전트라면, 블루프린트 원본은 만든 회사에 하나만 있다. 각 회사가 동의하는 순간 자기 테넌트에 블루프린트 서비스 주체가 생긴다. 이 서비스 주체가 에이전트 ID를 만들 권한을 갖는다.
5. **에이전트 ID 생성**: 블루프린트 권한을 물려받고, 필요하면 개별 권한을 더 붙인다. 이때 **스폰서 지정이 필수**다. 스폰서는 "이 에이전트가 아직 필요한가"를 판단하는 책임자다. 주인 없는 에이전트가 쌓이는 문제(sprawl)를 여기서 막는다. 소유자(기술 관리 권한)와 관리자는 선택이다.

이 과정은 대부분 플랫폼이 알아서 해 준다. 예를 들어 Foundry는 프로젝트마다 블루프린트를 만들고, 에이전트를 게시하면 그 에이전트용 블루프린트가 또 생긴다. 자체 에이전트 플랫폼을 쓰는 경우에는 MSAL, Entra PowerShell, Graph API로 직접 만들 수 있다. **Microsoft 플랫폼이 아닌 에이전트도 Agent ID를 쓸 수 있다**는 점이 중요하다.

## 에이전트 사용자 — 메일함과 Teams가 필요할 때

에이전트 ID는 내부적으로 서비스 주체와 비슷하다. 그래서 Exchange 메일함을 갖거나 Teams 대화에 참여하거나 SharePoint 권한을 받을 수 없다. 이게 필요하면 **에이전트 사용자(Agent User)**를 에이전트 ID의 자식으로 하나 더 만든다.

그러면 일하는 방식이 달라진다. 지금은 Word 문단을 복사해 채팅창에 붙이고 "이 문서의 이 부분을 이렇게 고쳐 줘"라고 길게 설명해야 한다. 에이전트 사용자가 있으면 문서에서 해당 부분을 선택하고 댓글로 @멘션만 하면 된다. 맥락이 이미 거기 있으니까. 메일을 보내거나 Teams에서 부를 수도 있다.

**라이선스는 이 지점에서 들어간다.** 사람이 메일·Teams를 쓰려고 M365 라이선스를 받듯이, 에이전트가 Exchange·SharePoint·Teams와 Work IQ(조직의 사람·문서·회의 관계를 이해하는 맥락 계층)를 쓰려면 **A365 라이선스**가 필요하다.

## 토큰은 어떻게 받나

<figure class="sketch">
<svg viewBox="0 0 660 200" role="img" aria-label="에이전트 플랫폼이 블루프린트로 인증하고 교환용 토큰을 에이전트 ID 토큰으로 바꾸는 흐름">
  <defs>
    <marker id="a365-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/></marker>
  </defs>
  <text x="0" y="16" class="sk-title">③ 토큰 흐름 (페더레이션 교환)</text>
  <rect x="10" y="60" width="140" height="56" rx="8" class="sk-box-muted"/>
  <text x="80" y="84" text-anchor="middle" class="sk-label">에이전트 플랫폼</text>
  <text x="80" y="102" text-anchor="middle" class="sk-sub">Copilot Studio 등</text>
  <path d="M152,88 L216,88" class="sk-line" marker-end="url(#a365-m2)"/>
  <text x="184" y="80" text-anchor="middle" class="sk-sub">①</text>
  <rect x="220" y="60" width="150" height="56" rx="8" class="sk-box"/>
  <text x="295" y="84" text-anchor="middle" class="sk-label">블루프린트로 인증</text>
  <text x="295" y="102" text-anchor="middle" class="sk-sub">→ 교환용 토큰</text>
  <path d="M372,78 L446,52" class="sk-line" marker-end="url(#a365-m2)"/>
  <text x="408" y="56" text-anchor="middle" class="sk-sub">②</text>
  <path d="M372,100 L446,140" class="sk-line" marker-end="url(#a365-m2)"/>
  <text x="408" y="134" text-anchor="middle" class="sk-sub">③</text>
  <rect x="450" y="28" width="200" height="50" rx="8" class="sk-box-accent"/>
  <text x="550" y="50" text-anchor="middle" class="sk-label">에이전트 ID 토큰</text>
  <text x="550" y="67" text-anchor="middle" class="sk-sub">받은 권한 범위 안에서 작업</text>
  <rect x="450" y="118" width="200" height="50" rx="8" class="sk-box"/>
  <text x="550" y="140" text-anchor="middle" class="sk-label">에이전트 사용자 토큰</text>
  <text x="550" y="157" text-anchor="middle" class="sk-sub">교환용 + ID 토큰을 함께 제시</text>
  <text x="10" y="190" class="sk-sub">에이전트 ID에는 비밀번호도 인증서도 없다. 플랫폼이 블루프린트 자격으로 토큰을 바꿔 줄 뿐이다.</text>
</svg>
<figcaption>앞서 정리한 <a href="/posts/sso-saml-oidc/">SSO·OIDC 글</a>의 토큰 개념 위에 "교환" 단계가 하나 더 붙은 형태다.</figcaption>
</figure>

## Entra — 직원용 통제를 에이전트에도

신원이 생기면 직원에게 쓰던 통제를 그대로 걸 수 있다. RBAC, API 권한, Azure 리소스 접근, 자격 관리(access package), 감사 로그가 모두 여기에 해당한다.

**조건부 액세스**에서는 대상에 "에이전트"를 고를 수 있다. 모든 에이전트, 특정 에이전트, 사용자 대신 일하는 경우, 특성(attribute) 기준 중에서 선택한다. Foundry는 프로젝트마다 블루프린트가 생기기 때문에 블루프린트 단위로 대상을 정할 수도 있다.

![Entra 조건부 액세스에서 정책 대상으로 에이전트 ID와 블루프린트를 고르는 화면](/assets/img/posts/agent-365-agent-id/02-ca-select-agents.jpg)
_조건부 액세스 정책 대상 선택. 에이전트 ID와 에이전트 블루프린트가 각각 대상으로 나온다. (출처: John Savill 영상)_

**위험 감지는 에이전트의 '정상'을 따로 이해한다.** 사람이 24시간 내내 일하면 계정을 공유하고 있을 가능성이 높다. 하지만 에이전트가 24시간 일하는 건 당연하다. 대신 위험한 사용자가 에이전트를 썼거나 로그인 빈도가 평소와 다르면 신호를 띄우고, 그 신호를 조건부 액세스의 조건으로 쓸 수 있다.

![에이전트 상세 화면의 애플리케이션 권한과 위임 권한 목록](/assets/img/posts/agent-365-agent-id/03-permissions.jpg)
_에이전트 상세의 권한 탭. 혼자 쓰는 애플리케이션 권한과 사용자 대신 쓰는 위임 권한이 따로 보인다._

![에이전트 보안·규정 준수 탭에서 Entra가 감지한 위험 신호](/assets/img/posts/agent-365-agent-id/04-security-risk.jpg)
_"비정상적인 로그인 빈도", "위험한 사용자가 접근함" 같은 Entra 신호가 에이전트 단위로 표시된다._

여기에 **에이전트 플랫폼 자체의 관측성**이 더해진다. Foundry 컨트롤 플레인은 다음을 본다.
- 프롬프트·첨부·추론 실패
- 환각
- 탈옥·프롬프트 인젝션 시도
- 회귀 평가

Microsoft 플랫폼이 아니어도 API Management 기반 AI 게이트웨이를 거치면 같은 가시성을 얻을 수 있다.

## Purview — 에이전트의 데이터 사용을 이해하는 감시

사람이 5분에 문서 500개를 열면 유출을 의심해야 한다. 하지만 에이전트가 3분에 문서 1,000개를 훑는 건 검색 증강 생성(RAG)이 원래 하는 일이다. Purview는 이 차이를 알도록 조정됐다. 그러면서도 민감 레이블이 붙었거나 AI 근거 자료로 쓰면 안 되는 데이터에 에이전트가 접근하면 정책에 따라 막는다.

![Purview 보호 지표 — 모니터링한 활동 920건, 막은 민감 상호작용 342건](/assets/img/posts/agent-365-agent-id/05-purview.jpg)
_에이전트 하나의 Purview 보호 지표. 감시만 하는 게 아니라 실제로 차단한 건수가 나온다._

## Defender — AI 전용 위협 보호

- **AI 보안 태세 관리**: 에이전트의 취약점, 잘못된 구성, 과도한 권한을 찾아낸다.
- **실시간 런타임 보호**: 직간접 탈옥, 피싱, ASCII 스머글링, Tor·악성 IP 접근, 위험한 도구 호출 같은 악성 행동이나 의도하지 않은 행동을 실시간으로 막는다.
- **헌팅·조사**: 사고가 나면 전체 공격 경로를 보여 준다. 도구 호출, MCP 상호작용, 계획 단계, 에이전트끼리 주고받은 통신까지 추적한다.

## 레지스트리와 컬렉션 — 에이전트용 '주소록'

에이전트 ID와 별개로 **레지스트리**가 있다. 두 가지 역할을 한다.

1. **인벤토리**: 조직에 어떤 에이전트가 있는지 한눈에 본다.
2. **에이전트 카드**: 에이전트의 능력과 다른 에이전트와 소통하는 방법을 적은 명세서다. 다른 에이전트가 이걸 보고 찾아와서 협업한다(A2A).

![Agent 365 개요 대시보드 — 레지스트리 537개, 활성 사용자, 플랫폼별 에이전트 수](/assets/img/posts/agent-365-agent-id/01-overview.jpg)
_M365 관리 센터의 Agent 365 개요. 플랫폼별 집계에 Copilot Studio·Foundry뿐 아니라 Workday·Google·ServiceNow 등 외부 플랫폼도 보인다._

![에이전트 레지스트리 맵 — 플랫폼별로 묶인 에이전트와 그 사이 연결](/assets/img/posts/agent-365-agent-id/06-registry-map.jpg)
_레지스트리 맵 보기. Copilot Studio, Foundry, Google, Amazon Bedrock 등 플랫폼별로 묶이고, 어떤 에이전트가 어떤 에이전트와 이야기하는지 선으로 보인다._

**가장 헷갈리기 쉬운 점은 레지스트리가 '발견' 도구이지 '접근 제어'가 아니라는 것이다.** GAL(전체 주소 목록)이 누구와 대화할 수 있는지를 정하지 않듯이, 레지스트리도 찾게 해 줄 뿐이다. 누가 무엇에 접근하는지는 여전히 역할과 조건부 액세스가 정한다. 또 하나, **Agent ID가 없는 에이전트도 레지스트리에 등록할 수 있다.** 다만 레지스트리를 조회해서 다른 에이전트를 찾는 쪽은 Agent ID가 있어야 한다.

**컬렉션**은 레지스트리를 더 작은 범위로 나눈 묶음이다.
- **Global**: 모두가 볼 수 있다.
- **Quarantined**: 아무도 발견할 수 없다. 문제가 생긴 에이전트를 여기 넣어 격리한다.
- **사용자 정의**: 영업 같은 업무 영역별로 만들어, 그 업무의 에이전트끼리만 찾게 한다.

컬렉션도 발견 범위를 정할 뿐 접근은 막지 않는다. 접근 제어는 Entra 그룹과 조건부 액세스의 몫이다.

![에이전트 컬렉션의 기본 제공 항목 Global과 Quarantined](/assets/img/posts/agent-365-agent-id/07-collections.jpg)
_기본 제공 컬렉션 두 가지. 사용자 정의(Custom) 탭에서 업무별 묶음을 추가로 만든다._

## 정리 — Copilot Studio 쓰는 입장에서 챙길 것

영상 내용을 실무 체크리스트로 바꿔 보면 이렇다. 아래는 영상을 바탕으로 한 내 해석이다.

- **스폰서부터 정한다.** 에이전트를 만들 때 "이걸 누가 책임지나"를 먼저 적는다. 나중에 정리할 때 가장 먼저 막히는 지점이다.
- **권한은 블루프린트 단위로 설계한다.** 공통 권한은 블루프린트에, 특정 에이전트만 필요한 권한은 그 에이전트 ID에 따로 준다.
- **비밀(secret)로 인증하지 않는다.** 자체 플랫폼이라면 인증서나 페더레이션을 쓴다.
- **조건부 액세스는 '보고 전용'으로 먼저 걸어 본다.** 영상 화면에서도 정책이 Report-only로 설정돼 있다. 에이전트를 대상으로 정책을 걸었을 때 무엇이 막히는지 먼저 본다.
- **A365 라이선스가 필요한 에이전트를 구분한다.** 메일함·Teams·SharePoint 협업이 필요한 'AI 팀원' 유형만 에이전트 사용자와 A365 라이선스가 필요하다.

한 줄로 줄이면 이렇다. **에이전트에게 사원증(Agent ID)을 주고, 책임자(스폰서)를 붙이고, 직원과 같은 규칙(Entra·Purview·Defender)으로 관리한다.** 그게 Agent 365다.

## 참고

- [John Savill — Agent 365 and Agent ID Overview (YouTube)](https://www.youtube.com/watch?v=WTcyL68qTo8)
- [영상 화이트보드 원본 (GitHub)](https://github.com/johnthebrit/RandomStuff/raw/master/Whiteboards/Agent365.png)
- [Microsoft Entra Agent ID 문서 (Microsoft Learn)](https://learn.microsoft.com/entra/agent-id/)
