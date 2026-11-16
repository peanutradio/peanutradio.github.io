---
title: "M365 개발자 샌드박스 개편 — 레거시는 만료 후 복구 불가 삭제"
date: 2026-11-17 06:53:00 +0900
categories: [IT News]
tags: [microsoft-365, copilot, governance, cost]
description: Microsoft 365 Developer Program이 60일 갱신 새 샌드박스로 바뀌고 레거시 샌드박스는 퇴역한다. 만료 후 삭제 전에 점검할 것을 정리했다.
---

> 이 글은 Microsoft 365 Developer Blog의 2026-11-16 발표 "Microsoft 365 Developer Program updates: new sandbox options and retirement of legacy sandboxes"를 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록이 아니라 발표 내용을 해석한 글입니다.
{: .prompt-info }

## 결론부터

개발자 계정에서 "Legacy sandbox" 라벨이 붙은 구독이 있다면 이번 주 안에 대시보드부터 열어 보는 것이 좋습니다. 이 구독은 퇴역 대상이고, 만료되면 읽기 전용이 된 뒤 데이터와 함께 영구 삭제되며 복구되지 않는다고 원문이 밝힙니다.

반대로 무료 E5 개발 샌드박스가 없어지는 것은 아닙니다. 무료라는 점, 적격 요건, 개발·테스트 전용이라는 조건은 그대로입니다. 바뀌는 것은 샌드박스의 "모델"입니다. 새 샌드박스는 새 테넌트로 만들어지고, 60일마다 개발 활동을 기준으로 갱신되며, 유료 add-on(Microsoft 365 Copilot, Copilot Credits)을 붙일 수 있습니다.

## 발표 내용 요약

원문에서 확인되는 사실만 추리면 다음과 같습니다.

### 바뀌지 않는 것

- 적격 개발자에게 샌드박스는 계속 무료입니다.
- 적격 요건은 바뀌지 않습니다.
- 샌드박스는 개발·테스트 용도로만 씁니다. 원문 인용 상자는 개발 외 용도로 쓰면 E5 개발자 구독이 회수될 수 있다고 경고하고, add-on을 샀다고 이 약관이 달라지지는 않는다고 덧붙입니다.

### 바뀌는 것

- 새 샌드박스는 적격 개발 활동을 기준으로 60일마다 갱신 대상이 됩니다.
- 유료 add-on(Microsoft 365 Copilot, Copilot Credits)을 선택해서 쓸 수 있습니다.
- 설정 때 검증용 청구 계정이 필요하지만, 샌드박스 자체에는 요금이 붙지 않고 add-on이나 구독을 샀을 때만 청구됩니다.
- 레거시 샌드박스는 Microsoft 전반의 구독 정책에 맞춰 퇴역합니다. Visual Studio에 연결된 구독도 포함됩니다.

### 레거시 구독의 두 갈래

<figure class="sketch">
<svg viewBox="0 0 720 280" role="img" aria-label="레거시 샌드박스가 적격 여부에 따라 마지막 60일 갱신 또는 만료 후 삭제로 나뉘는 흐름도">
  <defs>
    <marker id="m365-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="m365-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 레거시 구독의 갈림길</text>
  <rect x="8" y="104" width="150" height="56" rx="8" class="sk-box-accent"/>
  <text x="83" y="128" text-anchor="middle" class="sk-label">Legacy sandbox</text>
  <text x="83" y="146" text-anchor="middle" class="sk-sub">대시보드 라벨</text>
  <path d="M160,120 L240,70" class="sk-line-accent" marker-end="url(#m365-a1)"/>
  <path d="M160,144 L240,200" class="sk-line" marker-end="url(#m365-m1)"/>
  <rect x="242" y="40" width="190" height="56" rx="8" class="sk-box"/>
  <text x="337" y="64" text-anchor="middle" class="sk-label">여전히 적격</text>
  <text x="337" y="82" text-anchor="middle" class="sk-sub">마지막 60일 갱신 1회</text>
  <rect x="242" y="172" width="190" height="56" rx="8" class="sk-box-muted"/>
  <text x="337" y="196" text-anchor="middle" class="sk-label">적격 아님</text>
  <text x="337" y="214" text-anchor="middle" class="sk-sub">현재 기간 끝나면 만료</text>
  <path d="M434,68 L494,68" class="sk-line-accent" marker-end="url(#m365-a1)"/>
  <path d="M434,200 L494,200" class="sk-line" marker-end="url(#m365-m1)"/>
  <rect x="496" y="40" width="216" height="56" rx="8" class="sk-box-accent"/>
  <text x="604" y="64" text-anchor="middle" class="sk-label">새 샌드박스로 이전</text>
  <text x="604" y="82" text-anchor="middle" class="sk-sub">새 테넌트·도메인·테넌트 ID</text>
  <rect x="496" y="172" width="216" height="56" rx="8" class="sk-box-muted"/>
  <text x="604" y="196" text-anchor="middle" class="sk-label">읽기 전용 → 영구 삭제</text>
  <text x="604" y="214" text-anchor="middle" class="sk-sub">관리자만 접근, 복구 불가</text>
</svg>
<figcaption>같은 레거시 구독이어도 적격 여부에 따라 이전할 시간이 생기거나, 만료 후 삭제로 바로 이어진다.</figcaption>
</figure>

여전히 적격이면 마지막 60일 갱신이 1회 적용되고 안내 메일이 옵니다. 적격이 아니면 갱신 없이 현재 기간이 끝날 때 만료됩니다. 원문은 레거시 구독을 다시 변환하거나 연장할 수 없다고 못 박습니다. 구체적인 만료·삭제 날짜는 글에 없고, 대시보드와 알림 메일에서 확인하라고만 안내합니다.

## 이 블로그의 해석

### "무료 유지"보다 "새 테넌트"가 핵심이다

발표 제목은 새 옵션을 앞세우지만, 운영 관점에서 무게가 실리는 문장은 "새 샌드박스는 새 테넌트, 새 도메인, 새 테넌트 ID"라는 부분입니다. 내용이 자동으로 옮겨지지 않으므로, 사실상 환경을 한 번 새로 지어야 합니다. 앱 등록, 커넥터 연결, 사용자·라이선스 배정, 데모 데이터가 모두 대상입니다.

### 60일 갱신은 "방치하면 사라지는 환경"이 된다는 뜻이다

갱신 기준이 적격 개발 활동이므로, 한동안 손대지 않은 실습 테넌트는 갱신이 안 될 수 있다고 읽는 것이 안전합니다. 어떤 활동이 적격으로 인정되는지의 세부 기준은 이번 글에 없어서 Developer Program FAQ를 따로 확인해야 합니다. 강의나 시연용으로 샌드박스를 "만들어 두고 쓰는" 방식은 이 갱신 주기와 잘 안 맞을 수 있습니다.

### Copilot add-on은 선택지이지 기본값이 아니다

Microsoft 365 Copilot과 Copilot Credits를 새 샌드박스에서 쓸 수 있게 된 점은 Copilot Studio나 M365 에이전트를 실습하려는 쪽에 의미가 있습니다. 다만 무료 E5 샌드박스에 Copilot이 포함되는 것은 아닙니다. 붙이는 순간부터 청구가 시작되므로, 비용 상한을 어떻게 둘지는 add-on을 켜기 전에 정해야 합니다. 에이전트 도구 연결 방식은 [Copilot Studio에 MCP 서버 연결하기](/posts/copilot-studio-connect-mcp-server-guide/)에서 정리했는데, 이런 실습을 새 테넌트에서 다시 해야 할 수도 있습니다.

## 실무에서 달라지는 점

### 1. 지금 확인할 곳: 대시보드의 라벨

developer.microsoft.com/microsoft-365/profile에 가입 때 쓴 계정으로 로그인해 "Legacy sandbox" 라벨이 있는지 봅니다. 팀원마다 계정이 다를 수 있으니 사내 개발자 몇 명이 각자 쓰는지부터 파악해야 합니다. Visual Studio 연결 구독도 대상이라, 개발자 PC의 VS 혜택으로 만든 테넌트도 놓치면 안 됩니다. add-on 구매가 활성화된 샌드박스는 영향이 없습니다.

### 2. 이전 작업 목록 만들기

새 테넌트는 도메인과 테넌트 ID가 다릅니다. 따라서 Entra 앱 등록의 리디렉션 URI, 코드나 설정 파일에 박아 둔 테넌트 ID, Teams 앱 매니페스트, Copilot Studio 환경의 연결 참조는 이전 대상이 됩니다. 원문은 백업, 앱 재배포, 설정 재구성을 만료 전에 하라고 안내합니다. 이전 가능한 것은 코드로 관리(IaC·스크립트·매니페스트)해 두었는지에 따라 작업량이 크게 갈립니다.

### 3. 적격이 아닌 경우의 대안 비용

적격 요건을 못 맞추는 개발자는 Microsoft 365 trial이나 business 플랜으로 직접 라이선스와 add-on을 골라야 한다는 것이 원문 안내입니다. 사내에서 개발용 테넌트를 쓰던 팀이라면 이 경우 라이선스 비용이 새로 생기므로, 예산 반영 여부를 담당 부서와 먼저 맞춰야 합니다.

### 4. 교육·시연 환경은 갱신 주기를 달력에 넣기

강의나 워크숍용 테넌트가 있다면 60일 단위로 갱신 확인을 일정에 넣는 편이 안전합니다. 수강생 실습용으로 한 번 만든 환경을 오래 두는 방식은 위험합니다. 강의 자료에는 테넌트 ID나 도메인을 하드코딩하지 말고 변수로 빼 두라는 점을 교육에서 강조할 만합니다.

### 5. 개발 전용 약관 환기

add-on이 붙은 샌드박스에 실제 업무 데이터를 올리고 싶은 유혹이 생길 수 있습니다. 원문은 개발·테스트 전용 정책이 add-on 구매와 무관하게 유지된다고 하므로, 운영 데이터를 넣지 않는다는 원칙을 사내 가이드에 한 줄로 명시해 두는 것이 좋습니다.

## 아직 확인되지 않은 것

- 레거시 구독의 구체적인 만료일·삭제일은 원문에 없습니다. 대시보드와 메일로만 알 수 있습니다.
- 어떤 개발 활동이 갱신 기준을 충족하는지의 세부 정의는 이번 글에 없습니다.
- add-on 가격은 원문에 나오지 않습니다. 이 글에서도 금액은 다루지 않았습니다.

## 참고

- [Microsoft 365 Developer Program updates: new sandbox options and retirement of legacy sandboxes](https://devblogs.microsoft.com/microsoft365dev/microsoft-365-developer-program-updates-new-sandbox-options-and-retirement-of-legacy-sandboxes/) — Microsoft 365 Developer Blog, 2026-11-16
- [Microsoft 365 Developer Program 소개](https://learn.microsoft.com/en-us/office/developer-program/microsoft-365-developer-program)
- [Developer Program FAQ](https://learn.microsoft.com/en-us/office/developer-program/microsoft-365-developer-program-faq)
- [Legacy subscription retirement FAQ](https://learn.microsoft.com/en-us/office/developer-program/legacy-subscription-retirement-faq)
- [Developer Program 약관](https://learn.microsoft.com/en-us/office/developer-program/terms-and-conditions)
- [개발자 대시보드](https://developer.microsoft.com/microsoft-365/profile)
