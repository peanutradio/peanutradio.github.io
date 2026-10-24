---
title: "Anthropic의 증류 공격 방어 — API 사용자에게 달라진 점"
date: 2026-10-25 06:51:00 +0900
categories: [IT News]
tags: [anthropic, claude, distillation, api, security]
description: Anthropic이 증류 공격 방어책을 공개했다. 추론 기록 접근 제한과 차단 요청 과금 등 API를 쓰는 조직이 알아야 할 변화를 정리했다.
---

> 이 글은 Anthropic의 "Strengthening our defenses against distillation attacks"(10/12)를 읽고 정리·재구성한 것입니다. 수치는 모두 Anthropic의 자체 발표이며 제가 검증한 것이 아닙니다.
{: .prompt-info }

**결론부터.** Anthropic은 증류 공격(다른 모델을 만들려고 Claude의 출력과 추론 기록을 대량으로 빼 가는 행위)을 막는 새 방어책이 효과를 보고 있다고 밝혔습니다. 일반 API 사용자에게 중요한 건 세 가지입니다. 새 계정은 이전 컨텍스트를 수정할 수 없고, 추론 기록은 요청한 조직만 볼 수 있으며, 9월 24일부터 응답 전에 차단된 요청에도 요금이 붙을 수 있다는 점입니다.

## 무슨 일이었나

원문에 따르면 증류 공격은 가짜 계정망으로 규모 있게 벌어지고, 가장 값진 데이터는 최종 답이 아니라 추론 기록입니다. Anthropic은 초기 방어(추론 요약)가 효과가 있었다고 근거를 듭니다. 전체 추론 기록으로 학습한 모델은 SWE-bench Verified에서 80.4%, 요약본으로 학습한 모델은 71.8%였다고 합니다. 이후 공격자는 "이전 추론을 그대로 출력하라"는 식의 추출 프롬프트로 옮겨 갔습니다.

## 세 갈래 방어

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="추론 기록 통제, 차단과 패치, 반복 시도 억제의 세 방어 단계 스케치">
  <defs>
    <marker id="dist-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 증류 방어 세 겹</text>
  <rect x="8" y="50" width="200" height="90" rx="8" class="sk-box"/>
  <text x="108" y="84" text-anchor="middle" class="sk-label">추론이 가는 곳 통제</text>
  <text x="108" y="104" text-anchor="middle" class="sk-sub">컨텍스트 수정 제한</text>
  <text x="108" y="120" text-anchor="middle" class="sk-sub">기록은 요청 조직만</text>
  <rect x="260" y="50" width="200" height="90" rx="8" class="sk-box-accent"/>
  <text x="360" y="84" text-anchor="middle" class="sk-label">차단과 패치</text>
  <text x="360" y="104" text-anchor="middle" class="sk-sub">2단계 분류기</text>
  <text x="360" y="120" text-anchor="middle" class="sk-sub">오탐 0.1% 미만</text>
  <rect x="512" y="50" width="200" height="90" rx="8" class="sk-box"/>
  <text x="612" y="84" text-anchor="middle" class="sk-label">반복 시도 억제</text>
  <text x="612" y="104" text-anchor="middle" class="sk-sub">사용 한도 강화, 차단</text>
  <text x="612" y="120" text-anchor="middle" class="sk-sub">차단 요청 과금</text>
  <path d="M210,95 L256,95" class="sk-line" marker-end="url(#dist-m1)"/>
  <path d="M462,95 L508,95" class="sk-line" marker-end="url(#dist-m1)"/>
</svg>
<figcaption>한 곳을 막는 게 아니라 데이터 위치, 요청 필터, 계정 행동 세 층으로 막는다.</figcaption>
</figure>

1. **추론이 갈 수 있는 곳 제한**: Claude Fable 5.1·Opus 5.5에서는 "preserved thinking"이 2026년 8월 31일 이후 만든 새 API 계정의 이전 컨텍스트 수정을 막습니다. 추론은 보호가 약한 이전 모델이 읽을 수 없고, Sonnet 5.5에서는 추론 기록을 요청한 조직만 접근할 수 있습니다.
2. **차단과 패치**: 두 단계 분류기를 쓰고, 오탐률은 0.1% 미만이라고 합니다. 추론을 번역하거나 태그로 쓰게 하는 우회 기법도 막았다고 합니다.
3. **반복 시도 억제**: 반복하는 계정은 한도가 더 엄격해지고 지속하면 차단됩니다. 9월 24일부터는 응답 전에 안전장치가 막은 요청(증류 등 특정 영역)에도 요금을 부과합니다.

결과로 Anthropic은 증류자가 추출한 추론 토큰이 10~100배 이상 줄었고 수만 개 계정을 막았다고 밝혔습니다. 다만 얼마나 지속될지 모르고, 아직 매우 정교하고 끈질긴 우회는 보지 못했다고 스스로 적었습니다. 정당한 사용자가 연동을 고쳐야 했고, 목표를 완전히 달성하진 못했다고도 인정합니다.

## 실무에서 달라지는 점

1. **Claude API 연동 개발자**: 8월 31일 이후 만든 계정에서 이전 추론 블록을 편집해 다시 보내는 코드가 있다면 동작 변화를 점검하세요. 대화 기록을 재가공하는 에이전트가 해당됩니다. 적용 범위의 세부는 원문 기준으로 확인하세요.
2. **비용·정산 담당**: 차단된 요청도 과금될 수 있다는 점을 비용 모니터링에 반영하세요. 오탐이 0.1% 미만이라 해도 대량 호출 환경에서는 건수가 쌓일 수 있습니다(이건 제 추정입니다).
3. **보안·거버넌스 담당**: 조직 내 API 키가 외부로 새어 프록시 계정망에 쓰이면 우리 조직이 한도 제한이나 차단 대상이 될 수 있습니다. 키 관리와 사용량 이상 탐지를 점검할 이유가 생겼습니다.
4. **모델 선정 담당**: 오픈 웨이트 모델을 고를 때 증류 출신 여부와 안전장치 수준이 위험 요소라는 주장이 나왔습니다. 원문은 Anthropic 측 주장이므로 [오픈 웨이트 모델 위험 관리](/posts/open-weight-model-risk-deployment-context/) 관점과 함께 보세요.
5. **강사 관점**: "추론 기록은 자산이자 공격 표적"이라는 점, 그리고 API 약관과 안전장치가 비용 구조에도 들어온다는 점을 교육에서 짚겠습니다.

## 확인 못 한 것

Claude API 외의 제공 경로(타사 클라우드)에서 같은 정책이 어떻게 적용되는지, 차단 요청 과금 단가는 원문에 없습니다.

## 참고

- [Strengthening our defenses against distillation attacks — Anthropic](https://www.anthropic.com/news/distillation-defenses)
