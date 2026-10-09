---
title: "Claude Haiku 5.5 — 소형 모델에 effort가 생기고 가격은 크게 내렸다"
date: 2026-10-09 06:53:00 +0900
categories: [IT News]
tags: [claude, haiku, llm, pricing, ai-agent]
description: Anthropic이 10/7 공개한 Haiku 5.5는 Haiku급 최초로 effort 설정을 넣었고 입력 $0.10, 출력 $0.50(10만 토큰 이하)으로 가격을 낮췄다.
---

> 이 글은 Anthropic의 Claude Haiku 5.5 발표 페이지를 읽고 정리·재구성한 것입니다. 벤치마크는 모두 Anthropic 자체 발표입니다.
{: .prompt-info }

## 결론부터

Haiku 5.5는 에이전트의 **값싼 단계**(분류, 요약, 컴팩션, 서브에이전트)에 어떤 모델을 쓸지 다시 계산하게 만드는 발표입니다. 100만 토큰당 입력 $0.10, 출력 $0.50(요청 10만 토큰 이하 기준)이고, Haiku급으로는 처음으로 effort 설정(Low부터 Max까지)이 들어왔습니다. 다만 복잡한 agentic 코딩은 여전히 Sonnet 5.5나 Opus 5.5를 권장합니다.

<figure class="sketch">
<svg viewBox="0 0 720 210" role="img" aria-label="Haiku 4.5, Haiku 5.5, Sonnet 5.5의 100만 토큰당 입력 가격 비교">
  <text x="0" y="14" class="sk-title">① 100만 토큰당 입력 가격 (10만 토큰 이하 요청)</text>
  <text x="8" y="58" class="sk-label">Haiku 5.5</text>
  <rect x="130" y="42" width="20" height="24" rx="4" class="sk-box-accent"/>
  <text x="162" y="59" class="sk-mono">$0.10</text>
  <text x="8" y="104" class="sk-label">Haiku 4.5</text>
  <rect x="130" y="88" width="200" height="24" rx="4" class="sk-box"/>
  <text x="342" y="105" class="sk-mono">$1.00</text>
  <text x="8" y="150" class="sk-label">Sonnet 5.5</text>
  <rect x="130" y="134" width="400" height="24" rx="4" class="sk-box-muted"/>
  <text x="542" y="151" class="sk-mono">$2.00</text>
  <text x="130" y="192" class="sk-sub">막대 길이는 가격에 비례. 출력 가격은 각각 $0.50 / $5.00 / $10.00</text>
</svg>
<figcaption>같은 소형 모델 라인에서 입력 단가가 10분의 1로 내려갔다.</figcaption>
</figure>

## 가격

원문 표 기준으로 입력은 Haiku 4.5 $1.00에서 5.5는 $0.10(10만 토큰 이하) 또는 $0.50(초과)입니다. 출력은 $5.00에서 $0.50 또는 $2.50입니다. Anthropic은 새 토크나이저 때문에 작업당 토큰이 조금 늘어나는 것까지 반영해 평균 약 75% 저렴하다고 설명합니다. Sonnet 5.5의 캐시 읽기도 100만 토큰당 $0.10으로 절반이 됐고, 대부분의 agentic 작업 비용이 약 20% 줄어든다고 합니다.

캐시 단가도 같은 비율로 내려갔습니다. 10만 토큰 이하 요청 기준 캐시 읽기는 $0.01, 캐시 쓰기는 $0.125이고, 초과 구간은 각각 $0.05, $0.625입니다. Haiku 4.5는 캐시 읽기 $0.10, 쓰기 $1.25였습니다. 원문은 10만 토큰 이하 요청에서는 Haiku 4.5보다 90%, 초과 요청에서는 50% 싸고, Haiku 4.5 요청의 약 90%가 앞쪽 구간에 속했다고 덧붙입니다. "평균 75%"라는 숫자는 이 분포를 깐 결과라는 뜻입니다. 긴 문서를 통째로 넣는 워크로드라면 절감 폭은 50% 쪽에 가깝다고 보는 게 안전합니다.

모델 ID는 `claude-haiku-5-5`입니다. 같은 발표에서 Max 5x 구독자에게 월 $100, Max 20x에 월 $200, Team에는 사용자 합산 최대 $500의 API 크레딧을 준다는 내용과, Python·TypeScript SDK에 computer use·browser use 베타 지원이 추가됐다는 내용도 함께 나왔습니다.

## 성능

| 벤치마크 | Haiku 5.5 | Haiku 4.5 | Sonnet 5.5 |
|---|---|---|---|
| OSWorld 2.1 (오프라인 부분집합) | 72.4% | 15.7% | 83.9% |
| Terminal-Bench 4.0 | 39.2% | 0.0% | 70.6% |
| Humanity's Last Exam (도구 사용) | 57.4% | 18.7% | 64.5% |
| GDPval-AA v2.1 (Elo) | 1620 | 735 | 1840 |

Haiku 4.5와의 격차는 큽니다. 그래도 터미널 기반 코딩에서는 Sonnet 5.5와 차이가 분명합니다. 고객 사례로 Asana는 작업 완료 지연이 30% 넘게 줄었다고, HubSpot은 CRM 평가에서 92.8%를 얻었다고 전했습니다. AlphaSense는 400개 질의에서 0.84(Haiku 4.5는 0.76)를, Box는 Haiku 4.5보다 11점 높은 점수를 약 절반의 지연으로 얻었다고 했습니다. 이것 역시 Anthropic 페이지에 실린 고객 보고입니다.

effort별 결과는 OSWorld 2.1, GDPval-AA, Humanity's Last Exam 세 가지에 대해 그래프로만 제시됩니다. 표의 숫자가 어느 effort에서 나온 값인지는 시스템 카드를 함께 봐야 정확히 알 수 있습니다.

## 안전 장치 수준

보안 쪽 담당자라면 이 부분도 읽어 둘 만합니다. 원문은 사이버 보안 관련 안전 장치가 Haiku 4.5보다는 엄격하고 Sonnet 5.5보다는 느슨하다고 설명합니다. 방어 목적 작업은 더 허용하지만 침투 테스트나 공격자가 쓸 법한 기법은 여전히 막습니다. 더 넓은 범위가 필요한 조직은 Cyber Verification Program에 신청하라고 안내합니다.

## 어디에 쓰라고 하나

원문은 대량·비용 민감·범위가 좁은 작업(요약, 컴팩션, DB 쿼리, 분류, 빠른 조회, 서브에이전트)과 응답 속도가 중요한 고객 지원·브라우저 사용을 꼽습니다. 코딩은 Opus 5.5나 Sonnet 5.5를 메인으로 두고 Haiku를 서브에이전트로 붙이는 구성을 권합니다.

## 내가 확인할 것

- 페이지는 AWS, Google Cloud, Microsoft Azure에서 쓸 수 있다고 하지만, **Microsoft Foundry 모델 카탈로그에 실제로 올라와 있는지**는 이 페이지로 확인되지 않습니다. MS 쪽에서 쓰려면 카탈로그부터 봐야 합니다.
- 컨텍스트 윈도우 크기는 페이지에 나와 있지 않습니다.
- 벤치마크는 자체 발표라, 내 워크로드에서의 effort별 품질·비용은 따로 재봐야 합니다.

## 실무에서 달라지는 점

사내 인프라를 운영하거나 Azure·M365 환경에서 에이전트를 붙이는 입장에서 정리하면 이렇습니다.

1. **모델 라우팅을 다시 설계할 근거가 생겼다.** 메인 추론은 Sonnet·Opus에 두고, 분류·요약·컴팩션·조회 같은 단계를 Haiku 5.5 서브에이전트로 내리는 구성이 원문이 권하는 그림입니다. 단계별로 어떤 모델을 부르는지 설정 파일이나 오케스트레이터 코드에서 분리해 두면, 단가가 바뀔 때 코드 수정 없이 갈아탈 수 있습니다. [Azure Functions Dynamic Workflows](/posts/azure-functions-dynamic-workflows/)처럼 계획과 실행을 나누는 구조와도 잘 맞습니다.
2. **비용 추정은 구간을 나눠서 한다.** 10만 토큰 경계를 넘는 순간 입력 단가가 5배가 됩니다. 사내 문서 전체를 컨텍스트에 넣는 RAG나 긴 로그 분석은 경계를 넘기 쉬우니, 기존 요청 로그에서 프롬프트 길이 분포부터 뽑아 보고 예산을 잡아야 합니다. 토크나이저가 바뀌어 작업당 토큰이 늘어난다는 점도 반영합니다.
3. **Azure에서 쓸 수 있는지 먼저 확인한다.** 발표는 Microsoft Azure를 지원 플랫폼으로 들지만, Foundry 카탈로그 배포 여부·지역·가격이 Anthropic 직접 API와 같은지는 이 페이지로 확인되지 않습니다. 구매 경로(직접 API, 클라우드 마켓플레이스)에 따라 청구 주체와 데이터 처리 조건이 달라지므로 계약 담당과 함께 봅니다.
4. **effort는 기본값을 정해 두고 바꾼다.** effort가 올라갈수록 품질과 비용이 함께 움직입니다. 팀 단위로 "기본 Low, 실패 시 High로 재시도" 같은 규칙을 두고 결과를 기록해야 비용 폭주를 막을 수 있습니다.
5. **강의에서는 '싸졌다'보다 '어디에 쓰나'를 강조한다.** Terminal-Bench 4.0에서 Haiku 5.5(39.2%)와 Sonnet 5.5(70.6%)의 차이는 여전히 큽니다. 소형 모델은 범위가 좁은 반복 작업용이라는 구분을 먼저 가르치는 편이 오해를 줄입니다.

## 참고

- [Claude Haiku 5.5 — Anthropic](https://www.anthropic.com/claude-haiku-5-5) — 2026-10-07
