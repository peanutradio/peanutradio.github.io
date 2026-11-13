---
title: "Copilot 모델 6종 12/14 폐기 — 모델을 고정해 쓰는 팀의 점검표"
date: 2026-11-14 06:51:00 +0900
categories: [IT News]
tags: [copilot, github, governance]
description: GitHub Copilot에서 Claude Fable 5 등 6개 모델이 2026-12-14 폐기된다. 대체 모델, 자동 활성화 조건, 팀이 미리 확인할 것을 정리했다.
---

> 이 글은 GitHub changelog의 "Upcoming deprecation of selected GitHub Copilot models in December"(2026-11-13)와 "Gemini 4 Hydrogen in GitHub Copilot"을 읽고 정리·재구성한 것이다. 두 글 모두 짧은 changelog라서, 원문에 없는 부분은 내 해석이라고 따로 표시했다.
{: .prompt-info }

## 결론

- 2026-12-14에 Copilot의 모든 환경(Copilot Chat, 인라인 편집, ask·agent 모드, 코드 완성)에서 6개 모델이 폐기된다.
- Enterprise·Business에서는 관리자가 기본값을 끄지 않았다면 대체 모델이 자동으로 켜진다. 반대로 기본값을 꺼 둔 조직은 직접 켜야 한다.
- 폐기 후에 모델을 지우는 작업은 필요 없다. 해야 할 일은 "그 모델 이름을 박아 둔 곳"을 찾아 바꾸는 것이다.

## 폐기 대상과 대체 모델

원문 표를 그대로 옮기면 다음과 같다. 폐기일은 여섯 모델 모두 2026-12-14다.

| 폐기 모델 | 제안 대체 모델 |
|---|---|
| Claude Fable 5 | Claude Fable 5.2 |
| Claude Haiku 4.5 | Claude Haiku 5.5 |
| Gemini 3.8 Flash | Gemini 4 Hydrogen |
| Gemini 4 Argon | Gemini 4 Carbon |
| Grok 4.6 | Grok 4.7 |
| Kimi K3 | 발표 예정 |

두 가지 예외를 눈여겨볼 만하다.

### Haiku 4.5는 일부 구독자에게 남는다

원문 각주에 따르면 Claude Haiku 4.5는 개인 연간 플랜 구독자에게는 계속 제공되고, 이 폐기가 적용되지 않는다. 조직(Business·Enterprise) 계정에는 해당 문구가 없으니 폐기 대상으로 보는 편이 안전하다. 이 부분은 원문 표현을 근거로 한 내 읽기다.

### Kimi K3는 갈 곳이 아직 없다

대체 모델이 "발표 예정"이다. Kimi K3를 워크플로에 쓰는 팀은 12월 14일 전에 다른 모델로 옮길 계획을 따로 세워야 한다. 발표를 기다리다 날짜를 넘기면 대체할 모델을 고를 시간이 없다.

## 자동 활성화가 말해 주는 것

원문은 기본 모델 활성화(default model enablement)가 켜져 있으면 제안 대체 모델이 Enterprise·Business에서 자동으로 켜진다고 한다. 단, 관리자가 전역 기본값을 껐거나 해당 모델을 명시적으로 비활성화했다면 예외다. 기본값이 꺼져 있다면 Copilot 설정의 모델 정책에서 직접 켠다. 켜지면 지원되는 환경의 Copilot Chat 모델 선택기에 나타난다.

이 구조에서 생기는 함정은 "보수적으로 기본값을 끈 조직"이다. 신규 모델 승인 절차를 엄격히 두려고 기본값을 꺼 둔 곳일수록 대체 모델이 자동으로 켜지지 않아, 폐기일에 사용자가 쓸 모델이 줄어드는 상황이 생길 수 있다. 이건 원문에 없는 내 추론이지만, 문서가 말한 조건을 그대로 따라가면 나오는 결론이다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="폐기일까지 점검 흐름: 모델 이름 사용처 찾기, 대체 모델 정책 확인, 12월 14일 전 전환">
  <defs>
    <marker id="cp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 12/14 전에 할 일 세 단계</text>
  <rect x="8" y="50" width="200" height="70" rx="8" class="sk-box"/>
  <text x="108" y="80" text-anchor="middle" class="sk-label">사용처 찾기</text>
  <text x="108" y="100" text-anchor="middle" class="sk-sub">모델 이름을 박아 둔 곳</text>
  <rect x="260" y="50" width="200" height="70" rx="8" class="sk-box"/>
  <text x="360" y="80" text-anchor="middle" class="sk-label">정책 확인</text>
  <text x="360" y="100" text-anchor="middle" class="sk-sub">대체 모델이 켜져 있나</text>
  <rect x="512" y="50" width="200" height="70" rx="8" class="sk-box-accent"/>
  <text x="612" y="80" text-anchor="middle" class="sk-label">전환·공지</text>
  <text x="612" y="100" text-anchor="middle" class="sk-sub">12/14 이전에 끝내기</text>
  <path d="M210,85 L256,85" class="sk-line" marker-end="url(#cp-m1)"/>
  <path d="M462,85 L508,85" class="sk-line-accent" marker-end="url(#cp-m1)"/>
  <rect x="260" y="150" width="200" height="50" rx="8" class="sk-box-muted"/>
  <text x="360" y="180" text-anchor="middle" class="sk-sub">Kimi K3: 대체 모델 발표 대기</text>
</svg>
<figcaption>폐기는 지우는 작업이 아니라 "이름이 박힌 곳"과 "대체 모델 허용 여부"를 맞추는 작업이다.</figcaption>
</figure>

## 같이 나온 Gemini 4 Hydrogen

같은 날 changelog에는 Gemini 4 Hydrogen이 Copilot에서 정식 제공(generally available)된다는 글도 올라왔다. 원문은 빠르고 효율적인 에이전트 코딩, 도구 사용, 다단계 워크플로를 겨냥한 Google의 최신 Flash 모델이라고 소개하고, Gemini 3.8 Flash보다 지침을 더 잘 따른다고 한다. 수치나 벤치마크는 글에 없다.

- 대상 플랜은 Pro, Pro+, Max, Business, Enterprise이다.
- 사용량 기반 과금에서 제공사 공시 가격으로 청구된다.
- VS Code, Visual Studio, Copilot CLI, 코딩 에이전트, Copilot 앱, github.com, 모바일, JetBrains, Xcode, Eclipse, Copilot SDK에서 단계적으로 배포된다. 안 보이면 나중에 다시 확인하라는 안내가 있다.

Hydrogen 모델 자체의 가격과 컨텍스트는 이 블로그의 11월 13일자 Gemini 4 Hydrogen 글에서 따로 정리했다. Claude 쪽 대체 모델인 Fable 5.2의 단가는 [Claude Fable 5.2·Mythos 5.2 출시 글](/posts/claude-fable-mythos-5-2-pricing-fast-mode/)에서 볼 수 있다.

## 실무에서 달라지는 점

### 1. 모델 이름을 고정한 설정부터 검색한다

누가: 저장소·워크플로를 관리하는 개발 리드.

무엇을: 사내 문서, 에이전트 설정 파일, 스크립트, 교육 자료에 `Claude Fable 5`, `Gemini 3.8 Flash`, `Grok 4.6` 같은 표기가 있는지 문자열로 찾는다. 폐기 후에도 설정이 남아 있으면 해당 모델을 못 고르는 상태가 되므로, 날짜 전에 바꿔야 한다. 어디에 어떻게 지정되는지는 환경마다 달라서 원문에 구체적 경로는 없다. 확인할 것은 각 환경의 모델 선택 설정이다.

### 2. 관리자는 기본 모델 활성화 상태를 확인한다

누가: Copilot Enterprise·Business 관리자.

무엇을: Copilot 설정의 모델 정책에서 대체 모델(Fable 5.2, Haiku 5.5, Hydrogen, Carbon, Grok 4.7)이 켜져 있는지 본다. 전역 기본값을 끈 조직은 자동으로 켜지지 않으므로 직접 켜야 한다. 변경 이력과 승인 근거는 따로 남겨 두자.

### 3. 비용 구조가 다른 모델로 바뀐다는 점을 알린다

누가: 비용을 책임지는 팀장, FinOps 담당.

무엇을: Hydrogen은 사용량 기반 과금에서 제공사 공시 가격으로 청구된다고 한다. 대체 모델이 기존 모델과 같은 단가라는 보장은 원문에 없다. 전환 전에 Copilot 문서의 "Models and pricing" 표를 열어 단가를 비교한 뒤 전환하는 순서가 안전하다. 모델 선택 자체를 효과 수준으로 맡기는 방식이 궁금하다면 [Copilot Fusion GA 글](/posts/github-copilot-fusion-ga/)도 이어서 읽어 볼 만하다.

### 4. 교육에서는 "모델은 바뀌는 부품"이라고 강조한다

누가: IT 강사, 사내 교육 담당.

무엇을: 실습 자료에 특정 모델 이름과 화면을 박아 두면 12월 14일 이후 그 화면이 맞지 않는다. 교재에는 "모델 선택기에서 현재 제공되는 모델을 고른다"는 식으로 쓰고, 모델별 비교는 날짜를 붙인 부록으로 분리하는 편이 유지보수가 쉽다. 이건 내 운영 제안이다.

### 5. Kimi K3 사용자는 별도 일정으로 관리한다

누가: Kimi K3를 쓰는 팀.

무엇을: 대체 모델이 발표되지 않았으므로 다른 모델 후보를 지금 정해 두고, 발표가 나오면 그 후보와 비교한다. 날짜가 지나면 선택지가 사라진다는 점만 확실하다.

## 정리

폐기 공지 자체는 짧지만, 실제 부담은 "어디에 그 이름을 써 두었는지 모르는 것"에서 온다. 날짜가 12월 14일로 정해져 있으니, 이번 주에 사용처 목록과 모델 정책 상태만 확인해 두어도 대부분의 혼란은 피할 수 있다. 이 글은 원문을 읽고 정리한 것이며, 직접 설정을 바꿔 본 기록이 아니다.

## 참고

- [Upcoming deprecation of selected GitHub Copilot models in December (GitHub changelog, 2026-11-13)](https://github.blog/changelog/2026-11-13-upcoming-deprecation-of-selected-github-copilot-models-in-december)
- [Gemini 4 Hydrogen in GitHub Copilot (GitHub changelog, 2026-11-13)](https://github.blog/changelog/2026-11-13-gemini-4-hydrogen-in-github-copilot)
- [Supported models (GitHub Docs)](https://docs.github.com/copilot/reference/ai-models/supported-models)
- [Models and pricing for GitHub Copilot (GitHub Docs)](https://docs.github.com/copilot/reference/copilot-billing/models-and-pricing)
