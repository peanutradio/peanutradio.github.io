---
title: "2,500번 요청을 24번으로 — 응답이 작아도 작업은 싸지 않다는 API 설계"
date: 2026-10-19 06:53:00 +0900
categories: [AI Agent]
tags: [api, tool-design, github, agent, pagination]
description: GitHub 별 기록 API 재설계 사례를 읽고, 에이전트가 호출하는 도구의 API를 어떻게 설계할지 정리했다.
---

> 이 글은 GitHub 엔지니어링 블로그의 "From 2,500 requests to 24"(2026-10-14)를 읽고 정리·재구성한 것입니다. 원문은 에이전트를 다루지 않습니다. 에이전트 도구 설계로의 연결은 이 블로그의 해석이며, 직접 실행해 본 것이 아닙니다.
{: .prompt-info }

**결론부터.** 원문의 핵심은 "응답을 작게 만드는 것"과 "DB가 하는 일을 줄이는 것"이 다르다는 점입니다. 에이전트가 호출하는 도구도 같습니다. 도구는 사람보다 훨씬 많이, 끈질기게 호출되므로 **신원 없는 집계 응답**과 **범위 기반 페이지 나누기**가 비용을 좌우합니다.

## 무슨 일이 있었나

별 기록 도구들은 stargazer 목록을 페이지로 훑어 시각만 쓰고 이름은 버렸습니다. 문제가 셋이었습니다.

- 이름이 스팸·피싱 수집에 악용됐습니다.
- 페이지당 최대 100명이라 react/react(약 25.1만 별)는 전체 곡선에 약 2,511번 요청이 필요했습니다.
- 페이지 수 계산용 count 쿼리가 요청마다 전체 별을 셌습니다. 제한 전 일주일에 느린 쿼리로 약 37,000건 기록됐고 평균 약 3초, 1,000건 넘게 강제 종료됐습니다.

먼저 stargazer·watcher 목록을 관리자·협력자에게만 열자 stargazer 요청의 DB 처리가 약 60%, count 쿼리 시간이 약 80% 줄었습니다. 이어 새 엔드포인트를 냈습니다. **달력상 주 단위**로 `page`를 색인된 `created_at` 범위로 바꿔 먼저 거른 뒤, 일별 개수로 묶어 한 번에 최대 30주치를 돌려줍니다. 결과는 약 2,511번 → 약 24번, 각 별을 대략 한 번씩(약 25.1만 행) 검사합니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="집계 후 페이지 나누기와 범위로 거른 뒤 집계하는 방식의 작업량 비교">
  <defs>
    <marker id="sh-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="sh-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 같은 24번 요청, 다른 작업량</text>
  <rect x="8" y="40" width="130" height="46" rx="8" class="sk-box-muted"/>
  <text x="73" y="68" text-anchor="middle" class="sk-label">전체 별 조회</text>
  <rect x="190" y="40" width="130" height="46" rx="8" class="sk-box-muted"/>
  <text x="255" y="68" text-anchor="middle" class="sk-label">묶고 정렬</text>
  <rect x="372" y="40" width="130" height="46" rx="8" class="sk-box-muted"/>
  <text x="437" y="68" text-anchor="middle" class="sk-label">LIMIT 적용</text>
  <path d="M140,63 L186,63" class="sk-line" marker-end="url(#sh-m1)"/>
  <path d="M322,63 L368,63" class="sk-line" marker-end="url(#sh-m1)"/>
  <text x="520" y="68" class="sk-sub">집계 후 페이지: 약 600만 행 검사</text>
  <rect x="8" y="130" width="130" height="46" rx="8" class="sk-box-accent"/>
  <text x="73" y="158" text-anchor="middle" class="sk-label">날짜 범위 거름</text>
  <rect x="190" y="130" width="130" height="46" rx="8" class="sk-box-accent"/>
  <text x="255" y="158" text-anchor="middle" class="sk-label">일별로 묶음</text>
  <rect x="372" y="130" width="130" height="46" rx="8" class="sk-box-accent"/>
  <text x="437" y="158" text-anchor="middle" class="sk-label">응답</text>
  <path d="M140,153 L186,153" class="sk-line-accent" marker-end="url(#sh-a1)"/>
  <path d="M322,153 L368,153" class="sk-line-accent" marker-end="url(#sh-a1)"/>
  <text x="520" y="158" class="sk-sub">범위 먼저: 약 25.1만 행</text>
</svg>
<figcaption>응답 크기는 같아도 "무엇을 먼저 줄이느냐"에 따라 DB 작업량이 20배 넘게 갈린다.</figcaption>
</figure>

## 에이전트 도구에 옮겨 보면 (해석)

에이전트는 도구 응답이 길면 컨텍스트를 쓰고, 호출이 잦으면 속도·한도를 씁니다. 원문의 교훈을 도구 정의에 옮기면 이렇습니다.

1. **필요한 것만 준다.** 유지보수자에게 필요한 것은 추세였지 신원이 아니었습니다. 도구도 "목록 조회" 대신 "집계 조회"가 목적에 맞는지 먼저 묻습니다.
2. **페이지는 범위로 나눈다.** 오프셋 페이지는 매번 처음부터 계산하게 만들기 쉽습니다. 시간·ID 범위로 나누면 호출 횟수가 예측됩니다.
3. **한계를 계약에 적는다.** 원문도 별이 몰린 주는 여전히 느릴 수 있고, 별 취소는 기록에서 사라져 현재 별 기준 근사치라고 밝혔습니다. 도구 설명에도 이런 한계를 적어야 에이전트가 결과를 과신하지 않습니다.

[에이전트 규모 Git 인프라](/posts/github-early-october-roundup/) 글에서 본 "에이전트가 늘수록 인프라 호출이 달라진다"는 흐름과 맞닿는 사례입니다.

## 실무에서 달라지는 점

- **GitHub 별 기록 도구·대시보드를 쓰는 팀**: 원문은 이번 변경으로 일부 기존 도구가 먼저 깨졌다고 인정합니다. stargazer 목록에 의존한 스크립트가 있으면 403 또는 빈 응답이 올 수 있으니 확인하세요. 새 엔드포인트의 정확한 경로는 원문이 본문에 적지 않았고 "Get repository star history" 문서 링크로 안내하므로, 그 문서에서 확인해야 합니다.
- **MCP·사내 도구를 만드는 개발자**: 도구 하나를 정의할 때 "응답 크기"만이 아니라 "서버가 한 번에 훑는 행 수"를 같이 적어 두세요. 원문의 600만 행 대 25만 행 비교가 기준점이 됩니다.
- **Azure·M365 운영자**: 에이전트에 연결할 사내 API(예: 인사·재고 조회)에서 개인 식별 목록 대신 집계 엔드포인트를 따로 열면 노출 범위가 줄어듭니다. 도입 전 권한 모델과 함께 검토할 항목입니다.
- **IT 강사**: "응답이 작으면 빠르다"는 직관이 틀릴 수 있다는 사례로 쓸 수 있습니다. 호출 횟수 비교 실습은 소개 수준으로만 안내하고, 실제 수치는 각자 환경에서 측정하도록 하는 것이 정확합니다.

## 영상으로 보기

기초부터 짚고 싶다면 이 영상을 먼저 보세요. (AI 에이전트 기초 EP.04 — 도구 사용)

{% include embed/youtube.html id='IRzqRJRRDFw' %}

> 📺 [Peanut Radio AI 유튜브 채널](https://www.youtube.com/@PeanutRadioAI) — AI 시대의 IT, 기초부터 실무까지
{: .prompt-tip }

## 참고

- Taylor Cuilty, Sarita Iyer, [From 2,500 requests to 24: Rebuilding star history without the stargazers](https://github.blog/engineering/architecture-optimization/from-2500-requests-to-24-rebuilding-star-history-without-the-stargazers/) (GitHub Blog, 2026-10-14)
