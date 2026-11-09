---
title: "Claude Fable 5.2·Mythos 5.2 출시 — 단가, fast mode, 쓸 곳 정리"
date: 2026-11-10 06:53:00 +0900
categories: [IT News]
tags: [claude, llm, cost, foundry]
description: Anthropic이 11월 9일 Fable 5.2와 Mythos 5.2를 내놓았다. 입력 $8·출력 $40, fast mode는 2배 단가. 기본 모델 자리는 여전히 Opus 5.5다.
---

> 이 글은 Anthropic의 "Introducing Claude Fable 5.2 and Claude Mythos 5.2"(2026-11-09) 발표를 읽고 정리·재구성한 것입니다. 벤치마크 수치는 모두 Anthropic이 직접 공개한 값이며, 제가 따로 실행해 검증한 것이 아닙니다.
{: .prompt-info }

## 결론부터

Fable 5.2는 11월 9일부터 모든 플랫폼에서 쓸 수 있는 일반 공개 모델이고, 가격은 100만 토큰당 입력 $8, 출력 $40입니다. 같은 기반 모델에서 사이버·생물 안전장치를 뺀 Mythos 5.2는 검증 프로그램을 통과한 조직에만 열립니다. 그런데 정작 발표문이 "대부분의 코딩·업무는 Opus 5.5로 시작하라"고 권합니다. 즉 이번 발표는 "전부 Fable로 갈아타라"가 아니라, **모델 계층에 맨 위 한 칸이 더 생겼다**는 소식으로 읽는 편이 맞습니다.

## 원문에서 확인되는 것

- 모델 ID는 `claude-fable-5-2`이고, Fable과 Mythos는 같은 기반 모델입니다. 차이는 안전장치입니다. Fable은 고위험 사이버 작업(침투 테스트, 익스플로잇 생성, 바이너리 기반 취약점 작업)과 대부분의 생물학 연구 작업을 분류기로 막습니다.
- 막힌 질문은 사이버는 Opus 4.8, 생물은 Opus 5가 대신 답합니다.
- 속도는 이전 5.1보다 출력이 최대 30% 빠르다고 합니다. fast mode는 토큰 생성을 최대 2.5배 빠르게 하며, 단가는 입력 $16·출력 $80입니다.
- fast mode 제공처는 Claude Code, Claude Platform, Google Cloud의 Claude 모델, Microsoft Foundry(Anthropic 호스팅)입니다. Mythos의 fast mode는 검증 프로그램 조직만 씁니다.
- Fable 5.2의 안전장치는 5.1만큼 강하면서 불필요한 차단은 줄었다고 합니다. 초기 테스트에서 약 55% 감소(사이버 약 60%, 생물 약 40%)라는 수치를 제시했습니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="Fable 5.2, Mythos 5.2, Opus 5.5의 접근 범위와 단가 비교">
  <defs>
    <marker id="fm-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 같은 기반 모델, 열리는 범위가 다르다</text>
  <rect x="8" y="50" width="200" height="90" rx="8" class="sk-box-accent"/>
  <text x="108" y="82" text-anchor="middle" class="sk-label">Fable 5.2</text>
  <text x="108" y="102" text-anchor="middle" class="sk-sub">모든 사용자 · 안전장치 강함</text>
  <text x="108" y="122" text-anchor="middle" class="sk-mono">$8 in / $40 out</text>
  <rect x="260" y="50" width="200" height="90" rx="8" class="sk-box"/>
  <text x="360" y="82" text-anchor="middle" class="sk-label">Mythos 5.2</text>
  <text x="360" y="102" text-anchor="middle" class="sk-sub">검증 프로그램 조직만</text>
  <text x="360" y="122" text-anchor="middle" class="sk-sub">사이버·생명과학 심화</text>
  <rect x="512" y="50" width="200" height="90" rx="8" class="sk-box-muted"/>
  <text x="612" y="82" text-anchor="middle" class="sk-label">Opus 5.5</text>
  <text x="612" y="102" text-anchor="middle" class="sk-sub">발표문이 권하는 기본값</text>
  <text x="612" y="122" text-anchor="middle" class="sk-sub">필요할 때 위·아래로 이동</text>
  <path d="M108,142 L108,190" class="sk-line" marker-end="url(#fm-m1)"/>
  <text x="120" y="212" class="sk-sub">fast mode: 속도 최대 2.5배, 단가 2배 ($16 / $80)</text>
</svg>
<figcaption>Fable과 Mythos는 모델이 아니라 "누가 어떤 안전장치로 쓰는가"가 다르고, 기본값은 여전히 Opus 5.5다.</figcaption>
</figure>

## 가격을 일 단위로 환산하면

단가만 보면 와닿지 않으니 단순 예를 하나 들겠습니다. 입력 100만 토큰, 출력 20만 토큰을 쓰는 작업이면 표준 단가로 $8 + $8 = $16입니다. fast mode는 단가가 정확히 2배이므로 같은 양이 $32입니다. 이 계산은 발표문의 단가를 곱한 제 산수이고, 실제 청구는 캐시·배포 유형에 따라 달라질 수 있습니다.

여기서 눈여겨볼 점은 fast mode가 "공짜로 빨라지는 옵션"이 아니라 **속도를 돈으로 사는 옵션**이라는 것입니다. 사람이 화면 앞에서 기다리는 대화형 코딩에는 값어치가 있어도, 밤새 도는 배치 에이전트에는 대부분 낭비입니다.

또 하나, 발표문은 중간 effort(Claude 앱과 Claude Code의 기본값)에서 Fable 5.2가 5.1보다 적은 토큰으로 작업을 끝낸다고 말합니다. FrontierCode 2.0에서 High effort의 Fable 5.2가 Max effort의 Opus 5.5와 비슷한 점수를 작업당 비용은 대략 절반으로 낸다는 주장도 있습니다. 단가가 높아도 토큰이 줄면 총비용이 낮아질 수 있다는 뜻인데, 이건 Anthropic 자체 평가이므로 내 워크로드에서 재야 합니다.

## 벤치마크는 이렇게 읽는다

발표문 표에서 Fable 5.2는 FrontierSWE v2 69.9%(Opus 5.5 62.3%), OSWorld 2.1 89.3%(87.2%), Remote Labor Index 41.3%(21.3%) 등을 기록했습니다. 눈에 띄는 격차도 있지만, 몇 가지는 걸러서 봐야 합니다.

- 모든 수치가 제조사 자체 측정이고, 모델마다 최고 effort 설정에서 잰 값입니다.
- 일부 항목(RoboDojo Sim)은 내부에서 직접 돌렸다고 밝힙니다.
- AutomationBench처럼 Gemini 4 Carbon(53.9%)이 Fable 5.2(51.4%)보다 높은 항목도 표에 있습니다. 전 영역 1위가 아닙니다.
- 외부 평가 기관(영국 AI Security Institute, METR 등)은 이 글과 시스템 카드를 검토하지 않았다고 각주에 적혀 있습니다.

## 실무에서 달라지는 점

### 1. Foundry 운영자는 제공 범위부터 확인한다

발표문은 Fable 5.2가 Microsoft Azure를 포함한 모든 플랫폼에서 쓸 수 있다고 하고, fast mode는 Microsoft Foundry(Anthropic 호스팅)에서 제공된다고 합니다. 하지만 어떤 리전·배포 유형으로 열리는지, 쿼터가 어떻게 잡히는지는 이 글에 없습니다. 도입 전에 Foundry 카탈로그와 쿼터 화면에서 직접 확인하세요. 이전 [GPT-6 Astra·Sol·Luna의 Foundry 단가·배포 옵션 정리](/posts/gpt-6-astra-sol-luna-foundry-pricing/)와 같은 방식으로 표를 만들어 두면 모델 간 비교가 쉽습니다.

### 2. 기본 모델은 바꾸지 말고, 승격 규칙을 만든다

발표문이 Opus 5.5 시작을 권하므로, 팀 표준 모델을 Fable로 올릴 필요는 없습니다. 대신 "Opus 5.5로 두 번 실패한 작업만 Fable로 올린다"처럼 승격 조건을 문서로 정해 두세요. 비용 상한이 필요한 팀은 Fable을 조언자(Advisor) 또는 오케스트레이터로 두고 작은 모델에 위임하는 구성도 발표문이 언급하는 선택지입니다.

### 3. 보안 팀은 Fable로 취약점 작업을 하려 하지 않는다

Fable은 침투 테스트·익스플로잇 생성류를 막고 Opus 4.8로 우회 응답합니다. 보안 점검 자동화에 Fable을 붙이면 "답이 이상하다"로 보일 수 있습니다. 해당 업무는 Cyber Verification Program을 거쳐 Mythos를 쓰는 경로가 따로 있고, Claude Enterprise 고객은 Claude Security를 쓸 수 있다고 합니다. 승인 절차에 시간이 걸리니 필요하면 일찍 신청하세요.

### 4. 데이터 보존 정책을 확인한다

발표문에 따르면 Enterprise Frontier Safeguards(EFS)는 고객 인프라에 저장된 프롬프트·응답을 모니터링하고 ZDR(제로 데이터 보존)과 함께 쓸 수 있으며, 현재 비공개 베타입니다. 그 밖의 고객은 안전 모니터링을 위해 30일 보존이 적용된다고 되어 있습니다. ZDR 계약이 있는 조직은 도입 전에 법무·보안 팀과 이 문장을 같이 읽어야 합니다.

### 5. 강의·교육에서는 "최신=정답" 오해를 짚는다

IT 강사 관점에서 이번 발표의 좋은 교재는 "제조사가 기본값으로 한 단계 아래 모델을 권한다"는 점입니다. 모델 선택은 성능표가 아니라 작업당 비용·지연·차단 정책의 조합이라는 걸 수강생에게 강조하기 좋습니다. 비슷한 비용 관점은 [Anthropic 증류 공격 방어 글](/posts/anthropic-distillation-defenses-api-changes/)에서 다룬 API 변경과도 이어집니다. 이번 발표에도 preserved thinking이 모든 계정에 적용된다는 변경이 있으니, 기존 연동 코드가 이전 맥락을 수정하는지 점검 항목에 넣으세요.

## 아직 모르는 것

- Foundry의 리전, 배포 유형, 쿼터와 실제 청구 방식(발표문에 없음)
- Mythos 5.2의 API 모델 ID(발표문에 없음)
- 내 워크로드에서의 토큰 사용량 변화(자체 측정 필요)

## 참고

- Anthropic, Introducing Claude Fable 5.2 and Claude Mythos 5.2: https://www.anthropic.com/claude-fable-and-mythos-5-2
- 이전 글: [GPT-6 Astra·Sol·Luna, Foundry에서 단가와 배포 옵션 정리](/posts/gpt-6-astra-sol-luna-foundry-pricing/)
- 이전 글: [Anthropic의 증류 공격 방어 — API 사용자에게 달라진 점](/posts/anthropic-distillation-defenses-api-changes/)
