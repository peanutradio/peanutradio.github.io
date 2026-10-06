---
title: "GitHub, 에이전트 시대에 맞춰 Git 인프라를 다시 짓는다"
date: 2026-10-07 06:53:00 +0900
categories: [IT News]
tags: [github, git, ai-agent, azure, infrastructure]
description: GitHub가 에이전트가 쏟아내는 커밋 폭증에 대응해 Git 저장소 인프라를 재설계 중이다. 저장은 Azure Blob, 읽기는 가벼운 컴퓨트로 분리한다.
---

> 이 글은 GitHub Blog의 「Building Git infrastructure for agent-scale development」(2026-10-06, Brian Celenza)를 읽고 정리·재구성한 것입니다.
{: .prompt-info }

## 결론부터

GitHub가 "에이전트가 하루에 수백만 커밋을 밀어 넣는 시대"에 맞춰 Git 인프라를 근본부터 다시 만들고 있다. 핵심은 두 가지다. 쓰기 경로에서 **서로 기다리는 구간을 최소화**하고, **저장(storage)과 연산(compute)을 분리**한다. 출시일이나 GA 일정은 글에 없다. "기반을 이미 깔고 있다"는 단계이고, 어떻게 여기까지 왔는지는 후속 글로 풀겠다고 한다.

## 얼마나 늘었길래

글에 나온 수치(모두 GitHub 자체 집계)를 옮긴다.

- Git 이벤트: 월 2,182억 → 4,733억 건 (전년 대비)
- 9월 한 달 커밋: 14억 → 73.8억 건, 약 5배
- 푸시: 월 6.9억 → 33.5억 건 (4.9배)
- PR 머지: 전년 대비 거의 4배
- GitHub Actions 실행: 9월 32.6억 회, 연 4배
- 가장 바쁜 저장소 하나가 2026년 8월에 약 10억 요청

사람이 치던 속도를 전제로 설계된 시스템에 에이전트가 동시에 달라붙으면 이렇게 된다는 이야기다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="지난해 대비 9월 커밋 수와 푸시 수가 각각 약 5배 늘어난 것을 보여주는 비교 막대">
  <defs>
    <marker id="gi-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 월간 커밋 — 1년 만에 약 5배</text>
  <text x="8" y="68" class="sk-label">작년 9월</text>
  <rect x="100" y="48" width="95" height="28" rx="6" class="sk-box-muted"/>
  <text x="205" y="67" class="sk-sub">14억</text>
  <text x="8" y="118" class="sk-label">올해 9월</text>
  <rect x="100" y="98" width="500" height="28" rx="6" class="sk-box-accent"/>
  <text x="610" y="117" class="sk-sub">73.8억</text>
  <path d="M150,150 L560,150" class="sk-line-accent" marker-end="url(#gi-m1)"/>
  <text x="355" y="178" text-anchor="middle" class="sk-sub">막대 길이는 비율 스케치 (출처: GitHub Blog)</text>
</svg>
<figcaption>병목은 저장소 하나의 크기가 아니라, 동시에 쓰는 주체의 수가 바뀌었다는 데 있다.</figcaption>
</figure>

## 설계 원칙 세 가지

1. **조율(coordination) 최소화** — 브랜치 ref를 갱신하는 임계 경로와, 오브젝트 저장·시크릿 스캔처럼 병렬로 돌려도 되는 작업을 갈라놓는다.
2. **저장과 연산 분리** — 원본 데이터는 Azure Blob Storage로 옮기고, 읽기는 가볍고 늘리기 쉬운 컴퓨트 워커가 맡는다. 읽기와 쓰기를 따로 확장할 수 있다.
3. **기존 워크플로 보존** — 브랜치, 리뷰, 머지 큐, 브랜치 보호, 감사 로그는 그대로 유지한다.

내부 벤치마크에서 "최대 35배 높은 쓰기 처리량"이 나왔다고 한다. 어디까지나 GitHub 내부 측정이고 조건은 글에 자세히 나오지 않아서, 우리 저장소에서 체감할 수치로 읽으면 안 된다.

<figure class="sketch">
<svg viewBox="0 0 720 240" role="img" aria-label="기존에는 한 덩어리였던 저장과 연산이 분리되어, 쓰기 경로는 ref 갱신만 맡고 읽기는 독립 워커가 맡는 구조">
  <defs>
    <marker id="gi-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 저장과 연산을 분리</text>
  <rect x="8" y="60" width="150" height="52" rx="8" class="sk-box"/>
  <text x="83" y="84" text-anchor="middle" class="sk-label">에이전트·개발자</text>
  <text x="83" y="102" text-anchor="middle" class="sk-sub">동시 push</text>
  <rect x="250" y="40" width="190" height="52" rx="8" class="sk-box-accent"/>
  <text x="345" y="64" text-anchor="middle" class="sk-label">ref 갱신 (임계 경로)</text>
  <text x="345" y="82" text-anchor="middle" class="sk-sub">여기만 조율한다</text>
  <rect x="250" y="130" width="190" height="52" rx="8" class="sk-box"/>
  <text x="345" y="154" text-anchor="middle" class="sk-label">오브젝트 저장·스캔</text>
  <text x="345" y="172" text-anchor="middle" class="sk-sub">병렬로 처리</text>
  <rect x="530" y="60" width="180" height="52" rx="8" class="sk-box"/>
  <text x="620" y="84" text-anchor="middle" class="sk-label">Azure Blob Storage</text>
  <text x="620" y="102" text-anchor="middle" class="sk-sub">원본 데이터</text>
  <rect x="530" y="150" width="180" height="52" rx="8" class="sk-box-muted"/>
  <text x="620" y="174" text-anchor="middle" class="sk-label">읽기 워커</text>
  <text x="620" y="192" text-anchor="middle" class="sk-sub">가볍게 수평 확장</text>
  <path d="M160,86 L248,66" class="sk-line" marker-end="url(#gi-m2)"/>
  <path d="M160,92 L248,150" class="sk-line" marker-end="url(#gi-m2)"/>
  <path d="M442,66 L528,82" class="sk-line" marker-end="url(#gi-m2)"/>
  <path d="M442,156 L528,100" class="sk-line" marker-end="url(#gi-m2)"/>
  <path d="M620,114 L620,148" class="sk-line" marker-end="url(#gi-m2)"/>
</svg>
<figcaption>쓰기는 꼭 줄 세워야 하는 곳에서만 줄 세우고, 나머지는 읽기와 따로 늘린다.</figcaption>
</figure>

## 에이전트를 만드는 입장에서

- 에이전트 여러 개가 같은 저장소에 동시에 커밋하는 구성을 짜고 있다면, 병목이 내 코드가 아니라 플랫폼 쪽 쓰기 경로일 수 있다는 걸 알아둘 만하다. 이 글은 그 병목을 GitHub가 인정하고 손보고 있다는 신호다.
- 사용자 입장에서 바뀌는 것은 없다고 한다. 브랜치 보호·머지 큐·감사 로그 같은 워크플로는 보존하는 것이 전제다.
- 아직 로드맵 수준이다. 언제 어떤 플랜에 적용되는지는 후속 글을 기다려야 한다.

## 참고

- [Building Git infrastructure for agent-scale development — GitHub Blog (2026-10-06)](https://github.blog/engineering/architecture-optimization/building-git-infrastructure-for-agent-scale-development/)
