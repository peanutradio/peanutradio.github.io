---
title: "Copilot Fusion GA — 모델 대신 '효과 수준'을 고르는 시대의 비용·정책 점검"
date: 2026-10-31 06:53:00 +0900
categories: [IT News]
tags: [copilot, github, cost, governance]
description: GitHub Copilot Fusion이 GA됐다. 모델 선택기에서 Standard·High·Max만 고르면 Single·Cascade·Critique 중 하나로 처리되는 방식과 비용·관리자 설정을 정리한다.
---

> 이 글은 GitHub Changelog의 "Route and orchestrate models with GitHub Copilot Fusion"(2026-10-28) 발표를 읽고 정리·재구성한 것입니다. 벤치마크 수치는 모두 GitHub가 밝힌 값이며 직접 실행해 확인한 것이 아닙니다.
{: .prompt-info }

## 결론부터

Copilot Fusion은 "이 작업에 어느 모델을 쓸까"를 사람이 아니라 Copilot이 정하게 하는 옵션이고, 이번에 정식 공개(GA)됐다. 사용자가 고르는 것은 모델이 아니라 효과 수준(Standard, High, Max) 하나다. 이 변화에서 눈여겨볼 점은 성능보다 **통제 지점이 옮겨 간다**는 것이다. 모델 선택이라는 개발자의 손잡이가 사라지는 대신, 비용은 "실제로 쓰인 모델의 표준 요금"으로 청구되고, 허용 범위는 관리자의 모델 정책이 결정한다. 도입한다면 성능 비교보다 먼저 청구 방식과 정책을 확인하는 것이 순서라고 본다.

## 원문이 말하는 것

Fusion은 리서치 프리뷰였던 Project HydraFusion의 다중 모델 오케스트레이션과 Copilot의 auto 모델 선택을 합친 것이다. 효과 수준은 세 단계다.

- **Standard**

  일상적인 코딩에 맞는 범용 모델을 우선한다.

- **High**

  까다로운 작업과 깊은 추론에 맞는 모델을 우선한다.

- **Max**

  가장 어려운 작업과 복잡한 추론에 가장 강력한 모델을 우선한다.

프롬프트마다 Fusion이 세 가지 흐름 중 하나를 고른다. Single은 모델 하나가 바로 답하고, Cascade는 효율적인 모델이 초안을 쓴 뒤 품질 검사를 통과하면 채택하고 아니면 더 강한 모델로 넘긴다. Critique는 한 모델이 초안을 쓰고 다른 모델이 검토하면 처음 모델이 고치는 방식이다.

<figure class="sketch">
<svg viewBox="0 0 720 330" role="img" aria-label="Fusion이 프롬프트마다 Single, Cascade, Critique 세 흐름 중 하나를 고르는 구조">
  <defs>
    <marker id="fu-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 효과 수준 하나 → 작업마다 다른 흐름</text>
  <rect x="8" y="110" width="150" height="70" rx="8" class="sk-box-accent"/>
  <text x="83" y="140" text-anchor="middle" class="sk-label">프롬프트 + 효과 수준</text>
  <text x="83" y="160" text-anchor="middle" class="sk-sub">Standard / High / Max</text>
  <path d="M160,145 L215,145" class="sk-line" marker-end="url(#fu-m1)"/>
  <rect x="217" y="110" width="130" height="70" rx="8" class="sk-box"/>
  <text x="282" y="140" text-anchor="middle" class="sk-label">Fusion 라우터</text>
  <text x="282" y="160" text-anchor="middle" class="sk-sub">흐름 선택</text>
  <path d="M349,130 L420,60" class="sk-line" marker-end="url(#fu-m1)"/>
  <path d="M349,145 L420,145" class="sk-line" marker-end="url(#fu-m1)"/>
  <path d="M349,160 L420,235" class="sk-line" marker-end="url(#fu-m1)"/>
  <rect x="422" y="30" width="290" height="60" rx="8" class="sk-box"/>
  <text x="567" y="56" text-anchor="middle" class="sk-label">Single</text>
  <text x="567" y="76" text-anchor="middle" class="sk-sub">모델 하나가 바로 답한다</text>
  <rect x="422" y="115" width="290" height="60" rx="8" class="sk-box"/>
  <text x="567" y="141" text-anchor="middle" class="sk-label">Cascade</text>
  <text x="567" y="161" text-anchor="middle" class="sk-sub">효율 모델 초안 → 품질 검사 → 통과 또는 상위 모델</text>
  <rect x="422" y="205" width="290" height="60" rx="8" class="sk-box"/>
  <text x="567" y="231" text-anchor="middle" class="sk-label">Critique</text>
  <text x="567" y="251" text-anchor="middle" class="sk-sub">초안 → 다른 모델 검토 → 처음 모델 수정</text>
  <text x="8" y="310" class="sk-sub">청구: 흐름에 동원된 모델 각각의 표준 요금 (유료 요금제는 한시적 30% 할인)</text>
</svg>
<figcaption>사용자는 효과 수준만 고르고, 어떤 흐름을 탈지는 작업마다 Fusion이 정한다.</figcaption>
</figure>

제공 범위는 모든 Copilot 요금제이고, VS Code, GitHub Copilot 앱, Copilot CLI, JetBrains에서 쓸 수 있다. 다른 클라이언트는 "곧"이라고만 적혀 있다. 켜는 방법은 클라이언트를 업데이트하고 모델 선택기에서 Copilot Fusion과 효과 수준을 고르는 것이다. 프리뷰에서 auto나 HydraFusion 계열을 쓰던 사용자는 대응하는 Fusion 수준으로 자동 이동하는데, 정확한 대응표는 원문에 없다.

요금은 쓰인 모델의 표준 요금이며, 유료 요금제는 한시적으로 모델 비용 30% 할인이 붙는다. 관리자는 조직·엔터프라이즈 모델 정책에서 허용된 모델만 Fusion이 쓰게 되고, Fusion에서만 특정 모델을 끄거나 Fusion을 기본 모델로 지정할 수 있다.

성능 주장도 있다. Artificial Analysis Coding Agent Index에서 Fusion Max가 60점으로, Opus 5.5 Max를 쓴 Claude Code와 2점 이내이고, 작업당 평균 API 비용과 토큰은 약 절반, 시간은 약 40% 적다고 한다. 비교 조건과 과제 구성은 원문에 자세히 나오지 않는다.

## 이 발표를 어떻게 읽을까

### 모델 선택이 "예산 단위"로 바뀐다

지금까지 모델 선택기는 개발자에게 "비싸고 강한 것 vs 싸고 빠른 것"을 매번 판단하게 했다. Fusion은 그 판단을 효과 수준 세 칸으로 줄인다. 편해지는 대신 **같은 프롬프트의 비용이 호출마다 달라질 수 있다**. Single로 끝나는 날과 Critique로 모델 두 개를 부르는 날의 청구액은 같을 수 없기 때문이다. 원문도 흐름별 가격을 따로 제시하지 않고, 어떤 흐름이 쓰였는지 확인하는 방법도 밝히지 않았다. 이 부분이 도입 전 가장 큰 불확실성이다.

### 할인은 한시적이다

30% 할인은 "한시적"이고 종료 시점이 적혀 있지 않다. 파일럿 기간의 체감 비용을 그대로 연간 예산에 곱하면 어긋날 수 있다. 할인 없는 기준으로 한 번 더 계산해 두는 편이 안전하다.

### 벤치마크는 방향 참고용

점수 60과 "비용 절반"은 GitHub 자체 비교다. 내부 코드베이스의 언어, 테스트 체계, 작업 크기에 따라 결과는 다르게 나올 수 있어서, 이 수치는 "시도해 볼 이유" 정도로만 받아들이는 것이 적절하다.

이 블로그에서 다뤄 온 흐름과도 맞닿아 있다. [Copilot 보조 권한 GA](/posts/copilot-assisted-permissions-ga/)는 승인 단계를 모델이 먼저 거르는 이야기였고, [Copilot 하네스의 Foundry 확장](/posts/copilot-harness-foundry-agent-service-preview/)은 모델 바깥의 소프트웨어가 결과를 좌우한다는 이야기였다. Fusion도 같은 방향이다. 모델 한 개의 성능보다 **모델을 어떻게 묶고 고르느냐**가 제품의 경쟁력이 되고 있다.

## 실무에서 달라지는 점

### 1. 조직·엔터프라이즈 관리자: 모델 정책부터 열어 본다

Fusion은 정책에서 허용된 모델만 쓴다. 따라서 Fusion을 켜기 전에 현재 어떤 모델이 허용돼 있는지 확인해야 한다. 규정상 쓰면 안 되는 모델이 허용 상태로 남아 있었다면, 그동안은 개발자가 안 골라서 문제가 없었을 뿐 Fusion에서는 후보가 될 수 있다. 원문에 따르면 Fusion에서만 특정 모델을 끄는 설정이 있으므로, 전체 정책을 건드리지 않고 Fusion 후보만 좁힐 수 있다.

### 2. 비용 담당자: 청구 단위가 달라진다

요금은 쓰인 모델 각각의 표준 요금이다. 그래서 월 청구서에서 모델별 사용량이 어떻게 보이는지, Fusion 사용분을 따로 구분할 수 있는지를 첫 달 안에 확인하는 것이 좋다. 원문에는 이 보고 화면에 대한 설명이 없다. 한시적 할인이 끝난 뒤의 단가도 따로 가정해 두자.

### 3. 기본값 지정은 파일럿 뒤에

관리자는 Fusion을 어느 효과 수준으로든 기본 모델로 지정할 수 있다. 다만 전사 기본값으로 바로 지정하기보다, 한 팀에서 Standard로 시작해 비용과 만족도를 본 뒤 High·Max로 넓히는 순서가 합리적이라고 생각한다. 리서치 프리뷰에서 auto를 쓰던 사용자는 이미 자동 이동되므로, 본인 환경이 어느 수준에 놓였는지부터 확인할 필요가 있다.

### 4. 개발자·IT 강사: "모델 이름"이 아닌 "수준"으로 가르친다

교육 자료에서 특정 모델 이름을 전제로 한 스크린샷과 팁은 Fusion에서는 맞지 않을 수 있다. 어떤 모델이 답했는지가 눈에 안 보일 수 있기 때문이다. 수강생에게는 "결과가 이상하면 효과 수준을 올려 보고, 그래도 안 되면 프롬프트와 컨텍스트를 점검한다"처럼 수준 중심으로 가르치는 편이 낫다. 또한 Critique 흐름처럼 여러 모델이 개입하는 경우, 코드나 데이터가 어느 모델까지 전달되는지 정책 문서로 확인하는 습관도 함께 강조하고 싶다.

### 5. 도입 전 비교 시나리오 준비

같은 작업 묶음을 Standard, High, Max와 단일 모델로 나눠 돌려 비용, 소요 시간, 결과 품질을 기록하는 비교 틀을 미리 만들어 두면 좋다. 다만 이 글은 그렇게 해 본 기록이 아니라 계획 제안일 뿐이며, Fusion이 어떤 흐름을 골랐는지 확인할 방법이 원문에 없다는 한계는 감안해야 한다.

## 아직 확인되지 않은 것

- 효과 수준별 구체적인 가격과 흐름별 평균 비용
- 어떤 프롬프트가 Single, Cascade, Critique로 가는지에 대한 기준과 확인 방법
- 작업별로 모델을 직접 고정하는 방법 (원문 기준으로는 Fusion에서 모델을 고르지 않는다)
- 할인 종료 시점
- 추가 클라이언트 지원 일정("곧")

## 참고

- [Route and orchestrate models with GitHub Copilot Fusion — GitHub Changelog (2026-10-28)](https://github.blog/changelog/2026-10-28-route-and-orchestrate-models-with-github-copilot-fusion)
