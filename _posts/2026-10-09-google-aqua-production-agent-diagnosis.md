---
title: "AQuA — 운영 중인 에이전트의 실패를 묶고 코드 줄까지 짚는 진단 에이전트"
date: 2026-10-09 06:51:00 +0900
categories: [AI Agent]
tags: [ai-agent, evaluation, observability, google-adk, aqua]
description: Google이 공개한 AQuA는 운영 세션을 샘플링해 실패를 묶고 검증한 뒤 근본 원인을 코드 줄 범위로 짚는 오픈소스 참조 구현이다.
---

> 이 글은 Google Developers 블로그의 AQuA 소개 글을 읽고 정리·재구성한 것입니다. 수치는 원문 저자의 사례 측정이고, 제 해석은 별도로 표시했습니다.
{: .prompt-info }

## 결론부터

배포 전 평가는 에이전트를 어느 정도까지 끌어올립니다. 문제는 그 뒤입니다. 트래픽은 평가 세트에서 멀어지고, 모델·도구가 바뀌어도 헬스 체크는 초록색입니다. AQuA는 **운영 중인 에이전트 옆에서 세션을 샘플링해 실패를 묶고, 별도 모델로 검증한 뒤, 원인을 코드 줄 범위까지 짚어 주는** 오픈소스 참조 구현입니다. 요청 경로에 끼지 않고 에이전트에 쓰기도 하지 않습니다. 앞서 정리한 [ReviewBench](/posts/github-reviewbench-ai-code-review/)가 "평가를 어떻게 재나"였다면, 이 글은 배포 이후 평가입니다.

<figure class="sketch">
<svg viewBox="0 0 720 150" role="img" aria-label="AQuA의 다섯 단계: 샘플, 리뷰, 클러스터, 검증, 추적">
  <defs>
    <marker id="aq-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① AQuA 스윕 5단계</text>
  <rect x="4" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="64" y="74" text-anchor="middle" class="sk-label">샘플</text>
  <text x="64" y="92" text-anchor="middle" class="sk-sub">최대 1,000 세션</text>
  <rect x="150" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="210" y="74" text-anchor="middle" class="sk-label">리뷰</text>
  <text x="210" y="92" text-anchor="middle" class="sk-sub">9항목 체크리스트</text>
  <rect x="296" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="356" y="74" text-anchor="middle" class="sk-label">클러스터</text>
  <text x="356" y="92" text-anchor="middle" class="sk-sub">같은 실패끼리</text>
  <rect x="442" y="50" width="120" height="56" rx="8" class="sk-box-accent"/>
  <text x="502" y="74" text-anchor="middle" class="sk-label">검증</text>
  <text x="502" y="92" text-anchor="middle" class="sk-sub">별도 모델이 기각</text>
  <rect x="588" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="648" y="74" text-anchor="middle" class="sk-label">추적</text>
  <text x="648" y="92" text-anchor="middle" class="sk-sub">NEW / RECURRING</text>
  <path d="M126,78 L148,78" class="sk-line" marker-end="url(#aq-m1)"/>
  <path d="M272,78 L294,78" class="sk-line" marker-end="url(#aq-m1)"/>
  <path d="M418,78 L440,78" class="sk-line" marker-end="url(#aq-m1)"/>
  <path d="M564,78 L586,78" class="sk-line" marker-end="url(#aq-m1)"/>
</svg>
<figcaption>묶은 결과를 곧바로 믿지 않고, 한 번 더 검증하는 단계가 파이프라인의 중심이다.</figcaption>
</figure>

## 파이프라인

1. **샘플**: 최근 세션을 무작위로 최대 1,000개 뽑습니다.
2. **리뷰**: 9개 항목 체크리스트로 세션마다 채점하고, 실패는 실제/기대 형태의 발견으로 남깁니다.
3. **클러스터**: 같은 실패 메커니즘끼리 묶습니다.
4. **검증**: 별도 모델이 클러스터를 최대 3개의 전체 대화와 대조해 근거 없는 것을 버립니다.
5. **추적**: 열려 있는 인사이트와 비교해 NEW, RECURRING으로 표시하고, 14일간 안 보이면 RESOLVED로 처리합니다.

근본 원인 진단은 필요할 때 따로 돌립니다. 배포 시점에 저장한 소스 스냅샷을 읽어 `<경로>:<시작줄>-<끝줄>` 형태로 인용하고 수정안을 제안하지만, 스스로 적용하거나 PR을 열지는 않습니다.

## 사례: travel-concierge

- 32세션 중 5개만 깨끗하게 통과했고, 42개 발견이 9개 클러스터로 묶였습니다.
- 검증에서 3개가 거짓 양성으로 기각되어 6개가 남았습니다.
- 가장 큰 문제는 사용자가 말한 좌석을 가용성 확인 없이 저장한 것(15세션)이었고, 채식 조건이 다음 에이전트로 넘어가지 않아 스테이크를 추천한 것(7세션)이 뒤를 이었습니다.
- 프롬프트 한 줄씩 두 곳을 고치자 통과가 5/32에서 13/32로 늘었습니다.

## 비용

원문 기준으로 96세션 단일 에이전트 스윕이 약 $0.70, 위 사례(32세션, 스팬 1,583개)가 $3.76입니다. 근본 원인 분석은 인사이트당 $0.33~$2.47입니다. 세션 길이에 따라 비용이 크게 달라진다는 점이 숫자에서 읽힙니다.

## 한계와 내 해석

원문이 밝힌 한계는 무작위 샘플링만 한다는 점(재시도·지연 급증 같은 사전 필터는 과제), 판정 모델을 전문가 기준에 맞추는 일이 아직 숨은 공정이라는 점, 수백 턴짜리 긴 대화에는 전체 전사 방식이 안 맞는다는 점입니다.

여기서부터는 제 해석입니다. 이 구조는 Google Cloud(Cloud Trace, BigQuery, Gemini)에 묶여 있지만, "샘플링 → 채점 → 클러스터 → **검증** → 추적"이라는 뼈대는 Foundry 트레이스에도 옮길 수 있을 것 같습니다. 특히 검증 단계가 거짓 양성을 걸러내는 장치라는 점이 가져올 만합니다. 로컬 데모는 클라우드 없이 `make demo`로 돌릴 수 있습니다. 라이선스는 원문에 명시가 없어 저장소(google/adk-recipes)에서 확인이 필요합니다.

## 참고

- [The outer loop: insights first — Google Developers Blog](https://developers.googleblog.com/the-outer-loop-insights-first-an-ambient-quality-agent-that-diagnoses-your-production-agent/)
- [google/adk-recipes — ambient-quality-agent](https://github.com/google/adk-recipes/tree/main/core/python/ambient-quality-agent)
