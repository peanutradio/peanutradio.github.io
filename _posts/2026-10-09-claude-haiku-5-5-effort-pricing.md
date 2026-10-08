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

## 성능

| 벤치마크 | Haiku 5.5 | Haiku 4.5 | Sonnet 5.5 |
|---|---|---|---|
| OSWorld 2.1 (오프라인 부분집합) | 72.4% | 15.7% | 83.9% |
| Terminal-Bench 4.0 | 39.2% | 0.0% | 70.6% |
| Humanity's Last Exam (도구 사용) | 57.4% | 18.7% | 64.5% |

Haiku 4.5와의 격차는 큽니다. 그래도 터미널 기반 코딩에서는 Sonnet 5.5와 차이가 분명합니다. 고객 사례로 Asana는 작업 완료 지연이 30% 넘게 줄었다고, HubSpot은 CRM 평가에서 92.8%를 얻었다고 전했습니다. 이것 역시 Anthropic 페이지에 실린 고객 보고입니다.

## 어디에 쓰라고 하나

원문은 대량·비용 민감·범위가 좁은 작업(요약, 컴팩션, DB 쿼리, 분류, 빠른 조회, 서브에이전트)과 응답 속도가 중요한 고객 지원·브라우저 사용을 꼽습니다. 코딩은 Opus 5.5나 Sonnet 5.5를 메인으로 두고 Haiku를 서브에이전트로 붙이는 구성을 권합니다.

## 내가 확인할 것

- 페이지는 AWS, Google Cloud, Microsoft Azure에서 쓸 수 있다고 하지만, **Microsoft Foundry 모델 카탈로그에 실제로 올라와 있는지**는 이 페이지로 확인되지 않습니다. MS 쪽에서 쓰려면 카탈로그부터 봐야 합니다.
- 컨텍스트 윈도우 크기는 페이지에 나와 있지 않습니다.
- 벤치마크는 자체 발표라, 내 워크로드에서의 effort별 품질·비용은 따로 재봐야 합니다.

## 참고

- [Claude Haiku 5.5 — Anthropic](https://www.anthropic.com/claude-haiku-5-5)
