---
title: "Copilot 코드 리뷰 해결률 — 사용량 지표 API에 생긴 두 가지"
date: 2026-11-07 06:53:00 +0900
categories: [IT News]
tags: [copilot, github, observability, governance]
description: GitHub Copilot 사용량 지표 API에 코드 리뷰 해결률과 텔레메트리 누락 점검 필드가 추가됐다. 숫자를 읽기 전에 알아둘 함정을 정리한다.
---

> 이 글은 GitHub changelog 두 건(2026-11-05)을 읽고 정리·재구성한 것입니다. 원문 링크는 맨 아래 '참고'에 있습니다.
{: .prompt-info }

## 결론부터

Copilot 사용량 지표 API의 저장소 단위 리포트(`repos-1-day`)에 `copilot_code_review` 객체가 붙었습니다. 이제 "Copilot이 리뷰를 몇 번 돌렸나"가 아니라 "개발자가 그 댓글대로 움직였나"를 저장소별로 볼 수 있습니다. 같은 날 나온 둘째 changelog는 사용자별로 텔레메트리가 안 들어오는지, IDE·확장 버전이 최소 기준 아래인지 표시합니다. 두 건은 한 세트로 읽어야 합니다. 해결률이 낮게 나왔을 때 그것이 정말 팀의 반응이 낮은 것인지, 그냥 데이터가 빠진 것인지 구분하는 도구가 둘째 건이기 때문입니다.

원문에는 GA·프리뷰 표기가 없고 분류는 "Improvement"입니다. 기능 상태를 단정하지 않고, 문서에 적힌 범위만 옮깁니다.

## 무엇이 추가됐나

### 코드 리뷰 상세 필드

`copilot_code_review` 아래에 다음이 들어갑니다.

- `total_comments_evaluated`

  개발자가 반응했는지 확인한 Copilot 리뷰 댓글 수입니다.

- `total_comments_addressed`

  그중 개발자가 실제로 대응한 수입니다. 제안 적용, Copilot에게 수정 요청, 직접 코드 수정, 다음 PR에서 고치겠다는 답글까지 포함한다고 합니다. 이 값을 evaluated로 나누면 해결률입니다.

- `review_comments_by_severity` 등 분류별 집계

  심각도(high·medium·low), 리뷰 단계(lite·balanced·max), 댓글 유형(bug·security·custom), 커스터마이징 수단(custom instructions·skills·MCP 서버·없음)으로 나뉩니다.

- `total_merged_no_human_change_request_after_review`

  Copilot이 리뷰한 PR 중 Copilot의 첫 리뷰 뒤에 사람이 변경 요청을 하지 않고 병합된 건수입니다.

### 텔레메트리·버전 점검 필드

- `client_telemetry_present`

  그날 해당 사용자의 클라이언트 텔레메트리가 도착했는지입니다. `false`는 Copilot이 활동은 봤지만 에디터에서 사용 이벤트가 오지 않았다는 뜻입니다.

- `usage_metrics_requirements`

  사용자의 IDE·확장별로 최소 지원 버전을 충족(`met`), 미충족(`unmet`), 알 수 없음(`unknown`)으로 표시합니다.

- `missing_client_telemetry_user_count`, `below_usage_metrics_minimum_user_count`

  하루 단위 영향 사용자 수입니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="해결률 계산 흐름: 평가된 댓글 중 대응된 댓글의 비율, 그리고 텔레메트리 누락 점검이 옆에서 신뢰도를 확인">
  <defs>
    <marker id="cr-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="cr-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 해결률은 두 필드의 나눗셈, 신뢰도는 둘째 changelog가 확인</text>
  <rect x="8" y="50" width="190" height="60" rx="8" class="sk-box"/>
  <text x="103" y="76" text-anchor="middle" class="sk-label">Copilot 리뷰 댓글</text>
  <text x="103" y="96" text-anchor="middle" class="sk-sub">PR 병합일에 집계</text>
  <rect x="265" y="50" width="190" height="60" rx="8" class="sk-box"/>
  <text x="360" y="76" text-anchor="middle" class="sk-mono">evaluated</text>
  <text x="360" y="96" text-anchor="middle" class="sk-sub">반응 여부 확인 대상</text>
  <rect x="522" y="50" width="190" height="60" rx="8" class="sk-box-accent"/>
  <text x="617" y="76" text-anchor="middle" class="sk-mono">addressed ÷ evaluated</text>
  <text x="617" y="96" text-anchor="middle" class="sk-sub">= 해결률</text>
  <path d="M200,80 L262,80" class="sk-line" marker-end="url(#cr-m1)"/>
  <path d="M457,80 L519,80" class="sk-line-accent" marker-end="url(#cr-a1)"/>
  <rect x="265" y="160" width="447" height="60" rx="8" class="sk-box-muted"/>
  <text x="488" y="186" text-anchor="middle" class="sk-label">client_telemetry_present · usage_metrics_requirements</text>
  <text x="488" y="206" text-anchor="middle" class="sk-sub">숫자가 비어 있는 이유가 팀 문제인지 데이터 문제인지 가른다</text>
  <path d="M617,158 L617,112" class="sk-line" marker-end="url(#cr-m1)"/>
</svg>
<figcaption>해결률은 계산이 쉬운 만큼, 데이터가 빠진 사용자를 먼저 걸러야 의미가 생깁니다.</figcaption>
</figure>

## 숫자를 읽을 때 걸리는 점

### 집계 날짜는 댓글이 달린 날이 아니라 병합일

댓글은 PR이 병합될 때 평가되고, 병합한 날에 잡힙니다. 월요일에 달린 댓글이 금요일에 병합되면 금요일 숫자입니다. 일 단위 추이 그래프를 그릴 때 "Copilot을 설정으로 바꾼 날 해결률이 올랐다"는 식으로 읽으면 어긋날 수 있습니다. 주 단위나 병합 기준으로 보는 편이 안전합니다.

### 봇이 연 PR은 보통 평가하지 않는다

Copilot cloud agent 같은 봇이 연 PR의 댓글은 대체로 평가 대상이 아닙니다. 반면 `total_merged_no_human_change_request_after_review`에는 그런 PR도 포함됩니다. 에이전트가 만든 PR이 많은 저장소는 두 지표의 모집단이 달라, 이 둘을 한 문장으로 엮어 해석하면 안 됩니다.

### 기존 제안 적용 수와는 다른 숫자

`pull_requests.total_copilot_applied_suggestions`는 "바로 적용 가능한 제안을 누가 적용했나"만 셉니다. 새 해결률은 직접 고치거나 답글로 미룬 경우까지 포함하므로 보통 더 넓게 잡힙니다. 기존 대시보드와 나란히 놓으면 값이 다른 것이 정상입니다.

### 두 개의 누락 카운트는 더하지 않는다

`missing_client_telemetry_user_count`와 `below_usage_metrics_minimum_user_count`는 한 사용자가 양쪽에 들 수 있어 합산하면 중복입니다. 날짜별로 더해서 고유 사용자 수를 구하는 것도 안 됩니다. 값이 null이거나 없으면 0이 아니라 "데이터 없음"이고, 이번 릴리스 이전 날짜에는 값이 없습니다.

## 실무에서 달라지는 점

### 1. 접근 권한부터 확인

조회는 엔터프라이즈 소유자, 청구 관리자, 조직 소유자, 그리고 `View Copilot Metrics` 권한을 가진 커스텀 역할 보유자가 할 수 있고, Copilot 사용량 지표 정책이 켜져 있어야 합니다. 보고서를 만들어 줄 담당자가 소유자가 아니라면 커스텀 역할 부여가 먼저입니다. 정책이 꺼져 있으면 필드가 아예 안 나오므로, "추가됐다는데 안 보인다"는 문의는 이 설정을 먼저 보면 됩니다.

### 2. 해결률 보고 전에 텔레메트리 누락 사용자를 먼저 줄이기

텔레메트리가 없는 사용자도 활성 사용자로는 잡히지만 코드 라인 수, 기능·모델별 분해 같은 상세가 빠집니다. 원인으로 문서는 낡은 에디터·확장, 꺼진 IDE 텔레메트리, 이벤트를 막는 프록시·방화벽을 듭니다. 사내망에 프록시가 있는 조직이라면 `client_telemetry_present`가 `false`인 사용자 목록을 네트워크 담당자와 같이 보는 것이 첫 작업입니다. 버전이 기준을 충족해도 프록시 뒤에서는 텔레메트리가 없을 수 있다는 점이 문서에 명시돼 있어, 버전 점검만으로는 끝나지 않습니다.

### 3. 지표를 평가 점수로 쓰지 않기

해결률이 낮다고 팀이 리뷰를 무시한다는 뜻은 아닙니다. 댓글이 틀렸거나 중요하지 않을 수도 있습니다. 저는 개인·팀 비교용이 아니라, 심각도별·댓글 유형별로 어떤 종류의 댓글이 먹히는지 보는 용도로 쓰는 쪽이 맞다고 봅니다. 사내에 공유할 때 이 전제를 먼저 적어 두지 않으면 순위표로 소비되기 쉽습니다.

### 4. 설정 변경의 효과를 보려면 커스터마이징 분류를 활용

분류 중에 custom instructions, skills, MCP 서버, 없음이 있습니다. 지침 파일을 추가한 저장소와 그렇지 않은 저장소의 해결률을 같은 기간에 비교할 수 있다는 뜻입니다. 다만 저장소 특성이 다르니 인과를 단정하지 말고, 한두 저장소에 적용한 전후를 병합일 기준 주 단위로 보는 정도로 쓰길 권합니다.

### 5. 교육할 때는 "지표의 정의"를 한 장으로

강의나 사내 교육에서 Copilot 코드 리뷰 효과를 말할 때 '적용된 제안 수'와 '해결률'을 구분해 주는 한 장이 필요합니다. 위 세 가지(병합일 집계, 봇 PR 제외, 제안 적용 수와의 차이)만 정리해도 수치를 잘못 인용하는 일이 줄어듭니다.

## 함께 읽을 글

에이전트가 코드를 만들고 리뷰까지 맡는 흐름은 비용과 승인 정책 문제로 이어집니다. 모델 선택을 효과 수준으로 바꾸는 변화는 [Copilot Fusion GA](/posts/github-copilot-fusion-ga/), 승인 창을 모델이 먼저 거르는 변화는 [Copilot 보조 권한 GA](/posts/copilot-assisted-permissions-ga/)에서 정리했습니다. 이번 지표는 그 변경들이 실제 리뷰 흐름에서 먹히는지 숫자로 확인하는 쪽에 해당합니다. 이 연결은 제 해석입니다.

## 한계

두 changelog 모두 짧아서 API 호출 예시나 응답 JSON이 없습니다. 이 글은 문서에 적힌 필드 설명만 근거로 했고, 실제 응답 형태는 확인하지 못했습니다. 적용 전에는 공식 API 문서에서 응답 스키마를 확인해야 합니다.

## 참고

- [Usage metrics API adds code review details and addressed rate (GitHub changelog, 2026-11-05)](https://github.blog/changelog/2026-11-05-usage-metrics-api-adds-code-review-details-and-addressed-rate)
- [Usage metrics API flags telemetry and version gaps (GitHub changelog, 2026-11-05)](https://github.blog/changelog/2026-11-05-usage-metrics-api-flags-telemetry-and-version-gaps)
- [Copilot usage metrics API 문서](https://docs.github.com/rest/copilot/copilot-usage-metrics)
