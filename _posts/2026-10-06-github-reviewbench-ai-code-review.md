---
title: "GitHub ReviewBench 공개 — AI 코드 리뷰를 재는 공개 벤치마크"
date: 2026-10-06 13:28:00 +0900
categories: [IT News]
tags: [github, copilot, code-review, benchmark, ai-agent]
description: GitHub이 PR 219건으로 AI 코드 리뷰 에이전트를 평가하는 공개 벤치마크 ReviewBench를 내놓았다. 정답 구성 방식과 지표, 직접 평가하는 방법을 정리한다.
---

> 이 글은 GitHub Blog의 ReviewBench 발표를 읽고 정리·재구성한 것입니다. 수치는 모두 원문 기준입니다.
{: .prompt-info }

**결론부터.** GitHub이 코드 리뷰 에이전트를 평가하는 공개 벤치마크 ReviewBench를 냈다. 핵심은 "정답이 불완전하다"는 점을 인정하고 설계했다는 것이다. 정답 목록에 없는 새 지적도 따로 점수로 잡는다. 리뷰 에이전트를 고르거나 직접 만들 때 비교 기준으로 쓸 만하다.

## 무엇으로 만들었나

- 공개 PR **219건**, 저장소 187개, 언어 19종
- GitHub 전체 PR **1억 390만 건(103.9M)** 을 분석해 언어·저장소 크기 분포를 실제와 맞췄다
- 다만 한두 줄짜리 수정보다 여러 파일을 건드리는 의미 있는 PR 쪽으로 일부러 가중했다
- 전부 오픈소스 라이선스 저장소의 공개 PR

## 정답은 어떻게 만드나

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="ReviewBench 정답 구성 3단계: 수집, 중복 제거, 루브릭 검증">
  <defs>
    <marker id="rb-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 정답(ground truth) 구성 3단계</text>
  <rect x="8" y="50" width="200" height="90" rx="8" class="sk-box"/>
  <text x="108" y="80" text-anchor="middle" class="sk-label">1. 수집</text>
  <text x="108" y="102" text-anchor="middle" class="sk-sub">사람 리뷰 · 분석 도구</text>
  <text x="108" y="118" text-anchor="middle" class="sk-sub">프런티어 모델 · 후속 커밋</text>
  <path d="M210,95 L252,95" class="sk-line" marker-end="url(#rb-m1)"/>
  <rect x="260" y="50" width="200" height="90" rx="8" class="sk-box"/>
  <text x="360" y="80" text-anchor="middle" class="sk-label">2. 의미 중복 제거</text>
  <text x="360" y="102" text-anchor="middle" class="sk-sub">같은 문제를 하나로 합침</text>
  <path d="M462,95 L504,95" class="sk-line" marker-end="url(#rb-m1)"/>
  <rect x="512" y="50" width="200" height="90" rx="8" class="sk-box-accent"/>
  <text x="612" y="80" text-anchor="middle" class="sk-label">3. 공통 루브릭 검증</text>
  <text x="612" y="102" text-anchor="middle" class="sk-sub">LLM 판정(Claude Sonnet 5)</text>
  <text x="612" y="118" text-anchor="middle" class="sk-sub">심각도 · 범주 분류</text>
</svg>
<figcaption>한 출처의 지적에만 기대지 않고, 여러 출처를 모아 같은 잣대로 거른다.</figcaption>
</figure>

발견 사항은 심각도(Critical·Medium·Low)와 범주(정확성·보안·신뢰성·유지보수성·테스트 등)로 나뉜다. 출시 전에 독립된 시니어 엔지니어들이 정답을 처음부터 다시 라벨링했고, ReviewBench 라벨과 **96.6%** 일치했다고 한다.

## 지표: 고정 정답 vs 새 발견

| 계열 | 내용 |
|---|---|
| Grounded | 고정된 정답 목록 기준 Precision / Recall / F1 |
| Augmented | 목록에 없던 새 지적까지 판정자가 진짜/가짜를 가려 반영한 같은 세 지표 |

여기에 정밀도와 재현율 가중치를 바꿀 수 있는 Fβ가 더해져 총 여섯 지표다. 내가 보기엔 Augmented가 가장 영리한 부분이다. 잘하는 리뷰어일수록 정답지에 없는 걸 찾아내기 때문이다.

## 벤치마크가 실제와 맞았나

GitHub은 여러 모델을 묶은 앙상블 실험에서 오프라인 예측과 온라인 결과를 비교했다. 재현율은 온라인에서 13.6% 올랐고, 리뷰당 비용은 8.0% 내려 예측과 같은 방향이었다. 코멘트 수는 61% 늘었고, Critical 코멘트는 온라인 262% 증가(벤치마크 예측은 227%)였다. 크기까지 정확히 맞은 건 아니고 방향이 일치했다는 정도로 읽는 게 맞다.

## 직접 써보려면

review-bench.ai 에서 데이터셋과 리더보드를 볼 수 있고, 자체 에이전트도 평가할 수 있다. 절차는 GitHub 인증 → 에이전트(컨테이너 이미지) 등록 → 25개 PR 테스트셋 → 219개 전체 3회 실행 → 메인테이너 검토 후 리더보드 게재다.

## 읽으면서 걸린 점

- 판정자가 LLM 하나(Claude Sonnet 5)라 편향이 있을 수 있다고 원문도 인정한다
- 정답은 본질적으로 불완전하다
- 회사가 만든 벤치마크라는 점은 감안해서 봐야 한다

내 에이전트의 리뷰 품질을 숫자로 재야 할 때, 최소한 "무엇을 정답으로 칠 것인가"를 설계하는 참고 사례로는 충분히 가치가 있다.

## 참고

- [ReviewBench: An open benchmark for AI code review — GitHub Blog](https://github.blog/ai-and-ml/github-copilot/reviewbench-an-open-benchmark-for-ai-code-review/)
- [ReviewBench 사이트](https://review-bench.ai)
