---
title: "Copilot 코드 리뷰 설정이 API로 — 정책 자동화 전 토큰 종류부터"
date: 2026-11-09 06:53:00 +0900
categories: [IT News]
tags: [copilot, github, governance, automation]
description: Copilot 코드 리뷰 설정을 엔터프라이즈·조직·저장소 단위로 REST·GraphQL API에서 관리할 수 있게 됐다. 토큰 지원 범위와 운영 시 확인할 점을 정리했다.
---

> 이 글은 GitHub Changelog의 「Copilot code review: API management and review overview updates」(2026-11-06)를 읽고 정리·재구성한 것입니다. 원문에는 GA·프리뷰 표기가 없고, 저는 직접 실행해 보지 않았습니다.
{: .prompt-info }

## 결론부터

Copilot 코드 리뷰의 설정을 이제 웹 화면이 아니라 API로 다룰 수 있습니다. 엔터프라이즈 정책, 조직 기본값, 저장소별 옵션까지 세 층 모두 대상입니다. 다만 **엔터프라이즈·조직 단위 엔드포인트는 GitHub App과 fine-grained PAT를 지원하지 않습니다.** 자동화를 설계할 때 가장 먼저 걸리는 부분이 여기라서, 이 글은 이 제약을 중심으로 읽습니다.

적용 대상은 Copilot Pro, Pro+, Max, Business, Enterprise 요금제입니다. 원문은 구체적인 엔드포인트 경로를 적지 않고 docs의 "Configuring Copilot code review with the API" 섹션으로 안내합니다.

## 무엇이 추가됐나

### 설정 관리 — 역할별로 나뉜다

원문 기준으로 역할마다 만질 수 있는 범위가 다릅니다.

- 엔터프라이즈 관리자: 모든 조직의 코드 리뷰 사용 여부를 켜고 끄거나, 결정을 조직 관리자에게 맡길 수 있습니다.
- 조직 관리자: 사용 여부, 기본 리뷰 노력 수준(effort), 러너 구성을 관리합니다. 엔터프라이즈가 잠가 둔 설정은 덮어쓸 수 없습니다.
- 저장소 관리자: 리뷰 노력 수준, 그리고 리뷰에 커스텀 지침과 MCP 도구를 쓸지 여부를 관리합니다.

리뷰 노력 수준은 웹 화면과 같은 값을 REST API가 받고, 조직·저장소 설정 응답에 내가 쓸 수 있는 수준 목록이 함께 나옵니다.

### 출력 식별 — 누가 남긴 리뷰인지

Copilot이 남긴 리뷰와 댓글을 명시적인 attribution 필드로 구분할 수 있습니다. 리뷰에 쓰인 노력 수준(메타데이터가 있을 때), 그리고 댓글 근거로 Copilot이 보고한 스킬·MCP 서버·커스텀 지침도 조회됩니다. 이 필드들은 뒤에서 말할 "감사"에 쓸 만합니다.

<figure class="sketch">
<svg viewBox="0 0 720 300" role="img" aria-label="엔터프라이즈, 조직, 저장소 세 층의 설정 API와 지원 토큰 종류를 비교한 그림">
  <defs>
    <marker id="cra-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 설정 층과 쓸 수 있는 토큰</text>
  <rect x="8" y="40" width="220" height="64" rx="8" class="sk-box"/>
  <text x="118" y="66" text-anchor="middle" class="sk-label">엔터프라이즈 설정</text>
  <text x="118" y="86" text-anchor="middle" class="sk-sub">전체 조직 사용 여부 / 위임</text>
  <rect x="8" y="124" width="220" height="64" rx="8" class="sk-box"/>
  <text x="118" y="150" text-anchor="middle" class="sk-label">조직 설정 (러너 포함)</text>
  <text x="118" y="170" text-anchor="middle" class="sk-sub">기본 effort / 러너 구성</text>
  <rect x="8" y="208" width="220" height="64" rx="8" class="sk-box-accent"/>
  <text x="118" y="234" text-anchor="middle" class="sk-label">저장소 설정</text>
  <text x="118" y="254" text-anchor="middle" class="sk-sub">effort / 지침 / MCP 도구</text>
  <path d="M232,72 L330,72" class="sk-line" marker-end="url(#cra-m1)"/>
  <path d="M232,156 L330,156" class="sk-line" marker-end="url(#cra-m1)"/>
  <path d="M232,240 L330,240" class="sk-line-accent" marker-end="url(#cra-m1)"/>
  <rect x="336" y="40" width="370" height="148" rx="8" class="sk-box-muted"/>
  <text x="521" y="92" text-anchor="middle" class="sk-label">GitHub App · fine-grained PAT 미지원</text>
  <text x="521" y="116" text-anchor="middle" class="sk-sub">원문 경고: 엔터프라이즈·조직 엔드포인트</text>
  <text x="521" y="140" text-anchor="middle" class="sk-sub">클래식 PAT 등 사람 계정 토큰 기반으로 설계해야 함</text>
  <rect x="336" y="208" width="370" height="64" rx="8" class="sk-box-accent"/>
  <text x="521" y="234" text-anchor="middle" class="sk-label">GitHub App · fine-grained PAT 지원</text>
  <text x="521" y="254" text-anchor="middle" class="sk-sub">Administration read / write 권한</text>
</svg>
<figcaption>저장소 층은 앱으로 자동화할 수 있지만, 위의 두 층은 토큰 선택부터 제약이 있다.</figcaption>
</figure>

## 토큰 제약, 어떻게 읽어야 하나

원문 경고를 그대로 옮기면, 엔터프라이즈·조직 설정 엔드포인트(조직 러너 엔드포인트 포함)는 GitHub App과 fine-grained PAT를 지원하지 않습니다. 저장소 설정 엔드포인트는 둘 다 지원하며, 읽기에는 Administration: read, 수정에는 Administration: write 권한이 필요합니다. 클래식 PAT는 저장소 관리자 권한이 필요합니다.

여기서 제 해석을 덧붙입니다. 상위 두 층을 자동화하려면 결국 클래식 PAT 같은 사람 계정 기반 토큰이 필요하다는 뜻으로 읽힙니다(원문이 클래식 PAT의 엔터프라이즈·조직 요건을 따로 설명하지는 않으므로, 실제 필요 권한은 docs에서 확인해야 합니다). 퇴사나 권한 변경에 따라 끊길 수 있는 토큰에 정책 전체를 맡기는 구조는 좋지 않습니다. 반대로 저장소 층은 앱 기반으로 만들 수 있어 설계가 깔끔합니다.

## 리뷰 결과물도 바뀐다

설정 API가 이번 소식의 중심이지만, 같은 글에 리뷰 화면 쪽 변경도 실려 있습니다.

### 더 짧아진 리뷰 overview

발견 사항이 맨 위로 올라가고, 사용한 리뷰 노력 수준과 설문은 작은 footer로 내려갑니다. 건너뛰었거나 제외된 파일은 "Files not reviewed" 한 섹션에 모이고 제외 사유 링크가 붙습니다. 비어 있는 섹션은 빠지고 긴 목록은 초과 개수만 표시합니다. 👎를 누른 뒤 "Add a reason"에서 승인 판단에 동의하지 않음, 너무 장황함, 부정확하거나 사소한 지적 중 주된 문제를 고를 수 있습니다.

### 기타 변경

- 코드 리뷰 에이전트 세션에 리뷰에 쓴 AI credits가 표시됩니다.
- "Automatically request Copilot code review" 저장소 규칙이 bypass 목록을 지원해, 목록에 든 주체는 자동 리뷰를 받지 않습니다.
- 같은 PR의 이후 리뷰에서는 최근 변경과 무관한 발견 사항이 접히고 이어서 숨겨집니다.
- Balanced 노력 수준은 리뷰어 모델 3개를 쓰고 다른 모델이 결과를 합칩니다.

버그 수정도 있습니다. 코드 리뷰를 껐다 켜도 과금·프리뷰 설정이 유지되고, 실패하거나 시간 초과된 리뷰는 한 번 자동 재시도됩니다. 선호 모델을 쓸 수 없으면 다른 적격 모델로 전환될 수 있습니다. 외부 Copilot 라이선스로 요청한 리뷰를 막는 설정이 켜져 있어도, 조직이 비용을 내는 경우 Actions·Dependabot·조직 소유 앱이 봇 작성 PR에 리뷰를 요청할 수 있게 복구됐습니다.

## 실무에서 달라지는 점

### 1. 자동화 계정 설계를 먼저 정한다

상위 두 층은 앱·fine-grained PAT가 안 되므로, 어떤 토큰으로 돌릴지, 누가 보관하고 만료를 어떻게 알릴지부터 정해야 합니다. 서비스 전용 계정을 쓸지 여부는 사내 보안 정책과 맞춰야 하고, 이 글에서 정답을 단정할 수는 없습니다. 저장소 층만 먼저 앱으로 자동화하는 단계적 접근이 현실적으로 보입니다.

### 2. 엔터프라이즈 잠금과 조직 위임의 경계를 문서화한다

엔터프라이즈가 관리하는 설정은 조직이 덮어쓸 수 없습니다. API로 바꾸기 전에, 어떤 항목을 엔터프라이즈에서 고정하고 어떤 항목을 조직에 맡길지 표로 정리해 두면 "왜 내 설정이 안 바뀌는가"라는 문의를 줄일 수 있습니다.

### 3. 비용 지표를 함께 본다

리뷰 세션에 AI credits가 표시되고 Balanced가 모델 3개를 쓴다는 점을 같이 보면, 기본 노력 수준을 조직 전체에 올릴 때 비용이 어떻게 달라지는지는 확인이 필요한 항목입니다. 원문에 credits 단가나 노력 수준별 소비량은 없으므로 수치는 말할 수 없습니다. [Fusion 글](/posts/github-copilot-fusion-ga/)에서 다룬 효과 수준 선택과 같은 맥락에서 점검하길 권합니다.

### 4. 봇·자동화 PR의 예외를 점검한다

bypass 목록으로 특정 주체를 자동 리뷰에서 뺄 수 있습니다. 반면 외부 라이선스 차단 설정을 쓰는 조직은 이번 수정으로 Actions·Dependabot 등의 PR에 리뷰가 다시 요청될 수 있으니, 의도한 동작인지 확인합니다. 비용을 조직이 부담하는 경우라는 조건이 붙습니다.

### 5. 교육할 때는 "설정 변경 이력과 근거"를 강조한다

강의에서 코드 리뷰 도입을 다룬다면, 설정을 API로 바꿀 수 있다는 점과 함께 리뷰가 어떤 스킬·MCP 서버·지침을 근거로 삼았는지 조회된다는 점을 짚을 만합니다. 효과를 숫자로 보는 방법은 [코드 리뷰 해결률 글](/posts/copilot-code-review-usage-metrics-addressed-rate/)과 이어서 보면 좋습니다. 설정은 이 글, 측정은 그 글로 나뉩니다.

## 아직 모르는 것

원문에는 엔드포인트 경로, 요청 예시, GA·프리뷰 상태, 요금제별 세부 차이가 없습니다. 기능이 어느 단계인지 표기가 없으니, 운영 환경에 반영하기 전에 docs 페이지와 사내 테스트 조직에서 직접 확인하는 것이 안전합니다.

## 참고

- GitHub Changelog, Copilot code review: API management and review overview updates (2026-11-06): https://github.blog/changelog/2026-11-06-copilot-code-review-api-management-and-review-overview-updates
- GitHub Docs, Configuring Copilot code review with the API: https://docs.github.com/copilot/how-tos/copilot-on-github/set-up-copilot/configure-code-review#configuring-copilot-code-review-with-the-api
- 이전 글: [Copilot 코드 리뷰 해결률 — 사용량 지표 API에 생긴 두 가지](/posts/copilot-code-review-usage-metrics-addressed-rate/)
- 이전 글: [Copilot Fusion GA](/posts/github-copilot-fusion-ga/)
