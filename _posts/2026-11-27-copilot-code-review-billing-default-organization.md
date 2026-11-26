---
title: "Copilot 코드 리뷰 과금, 12/30부터 조직 청구가 기본값"
date: 2026-11-27 06:53:00 +0900
categories: [IT News]
tags: [copilot, github, cost, governance]
description: 12월 30일부터 과금 선택이 저장되지 않은 조직은 Copilot 코드 리뷰 비용이 조직으로 청구됩니다. 바뀌는 조건과 점검 순서를 정리했습니다.
---

> 이 글은 GitHub changelog의 "Copilot code review billing will default to the organization"(2026-11-25) 공지를 읽고 정리·재구성한 것입니다. 공지에 없는 메뉴 경로나 금액은 적지 않았고, 설정 화면은 직접 열어 보지 않았습니다.
{: .prompt-info }

## 결론부터

2026년 12월 30일부터, Copilot 코드 리뷰의 과금 방식을 한 번도 저장하지 않은 조직은 리뷰 비용이 **리포지토리를 소유한 조직**으로 청구됩니다. 지금은 리뷰를 요청한 멤버의 Copilot 권한(entitlement)으로 처리되는 구조입니다. 대상은 Copilot Business 또는 Copilot Enterprise를 쓰는 조직과 엔터프라이즈입니다.

지금과 똑같이 멤버 권한으로 계속 처리하고 싶다면 그 전에 정책 설정에서 **Member**를 직접 선택해 두면 됩니다. 아무것도 하지 않아도 장애가 나는 건 아닙니다. 다만 조직 과금은 AI Credits 유료 사용이 켜져 있어야 하고, 공지도 지출이 늘 수 있다고 명시합니다. 그래서 "기본값이 바뀌는데 아무도 몰랐다"가 가장 흔한 사고 경로가 될 가능성이 큽니다.

## 공지가 말하는 것

### 바뀌는 조건

변경은 "선택이 저장되지 않은 경우"에만 적용됩니다. 공지 내용을 표로 풀면 이렇습니다.

| 상황 | 12월 30일 이후 |
|---|---|
| Member를 이미 선택함 | 그대로 유지 |
| Organization을 이미 선택함 | 그대로 유지 |
| 선택 없음 + AI Credits 유료 사용 켜짐 | 조직 청구로 전환 |
| 선택 없음 + AI Credits 유료 사용 꺼짐 / 과금 잠금 / 과금 대상 고객 없음 | Member로 설정 |
| 라이선스 없는 멤버용 별도 설정 | 변경 없음 |

변경 전까지는 미설정 상태가 "Default (Member)"로 표시되고, 곧 바뀐다는 안내가 함께 뜬다고 합니다. 즉 지금 화면에서 Default라고 적혀 있다면 그게 바로 12월 30일에 움직일 설정입니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="12월 30일 전후로 미설정 조직의 Copilot 코드 리뷰 과금 대상이 멤버 권한에서 조직 청구로 바뀌는 흐름">
  <defs>
    <marker id="cb-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="cb-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 선택을 저장하지 않으면 청구 대상이 바뀐다</text>
  <rect x="8" y="50" width="150" height="52" rx="8" class="sk-box"/>
  <text x="83" y="74" text-anchor="middle" class="sk-label">코드 리뷰 요청</text>
  <text x="83" y="92" text-anchor="middle" class="sk-sub">라이선스 있는 멤버</text>
  <rect x="250" y="30" width="200" height="52" rx="8" class="sk-box-muted"/>
  <text x="350" y="54" text-anchor="middle" class="sk-label">~12/29 미설정</text>
  <text x="350" y="72" text-anchor="middle" class="sk-sub">멤버의 Copilot 권한으로 처리</text>
  <rect x="250" y="130" width="200" height="52" rx="8" class="sk-box-accent"/>
  <text x="350" y="154" text-anchor="middle" class="sk-label">12/30~ 미설정</text>
  <text x="350" y="172" text-anchor="middle" class="sk-sub">리포 소유 조직으로 청구</text>
  <rect x="540" y="130" width="170" height="52" rx="8" class="sk-box"/>
  <text x="625" y="154" text-anchor="middle" class="sk-label">AI Credits</text>
  <text x="625" y="172" text-anchor="middle" class="sk-sub">유료 사용 필요</text>
  <path d="M160,70 L246,58" class="sk-line" marker-end="url(#cb-m1)"/>
  <path d="M160,86 L246,150" class="sk-line-accent" marker-end="url(#cb-a2)"/>
  <path d="M452,156 L536,156" class="sk-line-accent" marker-end="url(#cb-a2)"/>
  <text x="0" y="226" class="sk-sub">Member 또는 Organization을 직접 저장했다면 양쪽 경로 모두 해당 없음</text>
</svg>
<figcaption>바뀌는 것은 설정값이 아니라 "저장하지 않았을 때 따라가는 기본값"입니다.</figcaption>
</figure>

### 멤버 과금을 유지하는 방법

공지가 안내하는 방법은 하나입니다. enterprise 또는 organization의 Copilot 정책 설정에서 Member를 선택합니다. 엔터프라이즈가 이 설정을 관리한다면 선택 권한은 엔터프라이즈 owner에게 있습니다. 한 번 선택하면 기본값이 바뀐 뒤에도 그 선택이 유지됩니다. 정확한 메뉴 경로는 공지에 없고 문서 링크만 있어서, 이 글에서도 경로를 적지 않았습니다. 설정하기 전에 공식 문서 "Configuring code review by GitHub Copilot"을 먼저 확인하세요.

## 내 해석: 이 변경이 까다로운 이유

### 기본값 변경은 알림이 늦게 닿는다

기능 폐기는 "쓰던 게 멈춘다"는 증상이 있어서 눈에 띕니다. 이번 건은 리뷰가 평소처럼 돌아가고, 다음 달 청구서 항목만 달라집니다. 조직 소유자는 설정을 만진 적이 없으니 공지를 놓치기 쉽습니다. 이건 제 추정이지만, 청구 주체가 바뀌는 변경은 "누가 이 설정의 주인인가"가 불분명한 조직일수록 늦게 발견됩니다.

### 어느 쪽이 정답인지는 조직 구조에 달렸다

멤버 과금은 개인 라이선스 안에서 처리되니 예측이 쉽고 부서별 분담이 단순합니다. 조직 과금은 리포지토리 단위로 비용이 모여서 팀별 비용 추적에는 유리할 수 있지만, 사용량이 늘면 한 곳으로 청구가 몰립니다. 어느 쪽이 낫다고 단정할 수 없으니, 이번 기회에 어느 쪽으로 갈지 의도적으로 결정해 두는 편이 안전합니다.

## 실무에서 달라지는 점

### 1. GitHub 조직 owner: 12월 30일 전에 설정을 저장

정책 설정에서 지금 "Default (Member)"로 보인다면, 의도한 값(Member 또는 Organization)을 명시적으로 저장하세요. 어느 쪽이든 저장만 하면 이번 기본값 변경의 영향에서 빠집니다. 연말 휴가철과 겹치니 12월 중순까지는 끝내는 게 좋습니다.

### 2. 엔터프라이즈 관리자: 하위 조직 설정 소유자를 정리

엔터프라이즈가 관리하는 설정이면 선택은 엔터프라이즈 owner의 몫입니다. 조직 owner가 바꿀 수 있다고 가정하고 기다리면 아무도 저장하지 않는 상황이 생깁니다. 누가 저장하는지 담당자를 먼저 정하세요.

### 3. 비용 담당자: Organization을 고를 거면 AI Credits부터

조직 과금은 AI Credits 유료 사용이 필요하다고 공지에 적혀 있습니다. 예산 승인 없이 켜는 것은 피하고, 켜기 전에 월 한도나 알림 설정이 가능한지 문서에서 확인하세요. 한도 기능의 세부 사항은 이번 공지에 없어 확인하지 못했습니다.

### 4. 사용량을 보고 싶은 팀: 지표와 API를 같이 본다

전환 전후로 리뷰가 얼마나 쓰이는지 보려면 이전에 정리한 [Copilot 코드 리뷰 사용량 지표 API](/posts/copilot-code-review-usage-metrics-addressed-rate/)와, 정책을 코드로 관리하는 [코드 리뷰 설정 API 글](/posts/copilot-code-review-api-management/)이 출발점이 됩니다.

### 5. IT 강사·교육 담당: "켜는 것"과 "누가 내는지"를 분리해서 설명

Azure DevOps 쪽은 Azure 구독으로 청구되는 구조였습니다([Azure Repos 코드 리뷰 GA 글](/posts/azure-repos-copilot-code-review-ga/) 참고). 같은 Copilot 코드 리뷰라도 GitHub는 멤버 권한 또는 조직, Azure Repos는 Azure 구독으로 청구 경로가 다릅니다. 교육에서는 기능 시연 뒤에 청구 경로를 반드시 한 장으로 짚어 주세요.

## 정리

- 12월 30일부터 미설정 조직의 기본 과금이 멤버 권한에서 조직 청구로 바뀝니다. 단, AI Credits 유료 사용이 꺼져 있거나 과금 잠금이 있으면 Member로 설정됩니다.
- 이미 Member나 Organization을 골랐다면 아무 일도 없습니다.
- 가장 싼 대응은 지금 의도한 값을 한 번 저장하는 것입니다.

## 참고

- [GitHub changelog — Copilot code review billing will default to the organization (2026-11-25)](https://github.blog/changelog/2026-11-25-copilot-code-review-billing-will-default-to-the-organization)
- [GitHub Docs — Configuring code review by GitHub Copilot](https://docs.github.com/copilot/how-tos/copilot-on-github/set-up-copilot/configure-code-review)
- 관련 글: [Azure Repos Copilot 코드 리뷰 GA](/posts/azure-repos-copilot-code-review-ga/), [Copilot 코드 리뷰 설정이 API로](/posts/copilot-code-review-api-management/), [Copilot 코드 리뷰 해결률 지표](/posts/copilot-code-review-usage-metrics-addressed-rate/)
