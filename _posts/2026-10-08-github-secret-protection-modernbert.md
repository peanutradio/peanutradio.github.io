---
title: "에이전트 속도에 맞춘 비밀 정보 보호 — GitHub의 ModernBERT 분류기"
date: 2026-10-08 06:53:00 +0900
categories: [IT News]
tags: [github, secret-protection, security, modernbert, copilot]
description: GitHub이 패턴 없는 비밀 값을 2ms 미만에 가려내는 ModernBERT 분류기를 공개했다. 수치와 배포 일정을 정리했다.
---

> 이 글은 GitHub 블로그의 "Secret protection must scale with software"(2026-10-07)를 읽고 정리·재구성한 것이다. 수치는 모두 GitHub 자체 집계다.
{: .prompt-info }

## 결론부터

AI 에이전트가 코드를 쓰는 비중이 커지면서 키 유출을 사람이 막는 방식은 한계에 닿았고, GitHub은 플랫폼 쪽에서 막는 비중을 늘리겠다고 한다. 그 수단이 새 **ModernBERT 기반 분류기**다. 후보 비밀 값을 주변 코드 맥락과 함께 보고, 후보 묶음 하나를 2밀리초 미만에 판정한다고 한다. GitHub은 막을 수 있는 비밀의 수가 두 배 넘게 늘 수 있다고 본다.

## 왜 지금인가 — 원문이 든 숫자

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="GitHub이 제시한 비밀 정보 관련 핵심 수치 네 가지">
  <text x="0" y="14" class="sk-title">① GitHub 자체 집계 수치</text>
  <rect x="8" y="40" width="170" height="150" rx="8" class="sk-box"/>
  <text x="93" y="95" text-anchor="middle" class="sk-label">PR 3건 중 1건</text>
  <text x="93" y="120" text-anchor="middle" class="sk-sub">AI 에이전트 관여</text>
  <text x="93" y="138" text-anchor="middle" class="sk-sub">(1년 전 10건 중 1건 미만)</text>
  <rect x="190" y="40" width="170" height="150" rx="8" class="sk-box"/>
  <text x="275" y="95" text-anchor="middle" class="sk-label">약 2초마다</text>
  <text x="275" y="120" text-anchor="middle" class="sk-sub">공개 코드에 새 비밀</text>
  <text x="275" y="138" text-anchor="middle" class="sk-sub">3년간 해마다 약 2배</text>
  <rect x="372" y="40" width="170" height="150" rx="8" class="sk-box"/>
  <text x="457" y="95" text-anchor="middle" class="sk-label">약 40일</text>
  <text x="457" y="120" text-anchor="middle" class="sk-sub">수동 폐기 평균</text>
  <text x="457" y="138" text-anchor="middle" class="sk-sub">다섯 중 하나는 90일 초과</text>
  <rect x="554" y="40" width="158" height="150" rx="8" class="sk-box-accent"/>
  <text x="633" y="95" text-anchor="middle" class="sk-label">약 30%</text>
  <text x="633" y="120" text-anchor="middle" class="sk-sub">push protection이</text>
  <text x="633" y="138" text-anchor="middle" class="sk-sub">저장소 진입 전에 차단</text>
</svg>
<figcaption>유출은 빠르게 늘고 수습은 느리니, 들어오기 전에 막는 비중을 키워야 한다는 논리다.</figcaption>
</figure>

2024년 2분기에서 2026년 2분기 사이 검사된 push는 2.84배, 자격 증명이 든 push는 2.59배 늘었다. 흥미로운 대목은 개발자가 부주의해진 것은 아니라는 주장이다. push 단위 유병률에는 아홉 분기 동안 통계적으로 뚜렷한 추세가 없었고, push protection 차단을 개발자가 우회한 비율은 6.63%에서 3.93%로 줄었다. 늘어난 것은 코드의 양이다.

## 분류기가 하는 일

기존 패턴 매칭은 형식이 정해진 키에는 강하지만, 사내 DB 비밀번호처럼 **모양이 없는 비밀**은 놓친다. 이 분류기는 Microsoft Applied Sciences와 함께 만든 미세 조정 모델로, 코드나 글을 생성하지 않고 후보를 맥락 속에서 판정한다. 원문 예시로는 DB URL, Kubernetes Secret 매니페스트, Dockerfile 안의 비밀번호 같은 값은 잡고 `changeme` 같은 자리표시자는 통과시킨다. 원문은 이 문제를 정밀도, 지연, 처리량, 비용이 서로 묶인 "4체 문제"라 부르며, LLM 기반 파이프라인보다 정밀하면서 push 경로에 넣을 만큼 싸다고 주장한다. 이 비교는 GitHub의 주장이고 독립 검증은 아직 보지 못했다.

## 배포 일정

- **지금**: 비공개 프리뷰. AI 비밀 탐지를 쓰는 조직은 자동으로 새 모델로 갱신되고, push 이후 알림은 추가 비용이 없다.
- **10월 중**: Enterprise Cloud와 Teams의 GitHub Secret Protection 조직에서 모델 기반 push protection을 쓸 수 있다. AI 크레딧을 소모한다.
- **GitHub Enterprise Server 3.23**: 공개 프리뷰로 들어가 폐쇄망에서도 AI 탐지 알림을 쓴다.
- **Copilot CLI·Copilot 앱**: `/security-review` 명령에 분류기가 들어가, Secret Protection 플랜 없이도 push 전에 점검할 수 있다. 크레딧 사용량은 AI 사용 인사이트에서 Secret Protection으로 집계된다.

## 내 메모

에이전트에게 코드를 맡기는 입장에서 눈여겨볼 부분은 push 전에 도는 `/security-review`다. 플랫폼 방어는 마지막 방어선이고, 에이전트가 쓰는 `.env`나 설정 파일에 진짜 키를 두지 않는 습관이 먼저다. 다만 이 글은 소개 글이어서 분류기를 실제로 돌려 본 결과는 없다. 정밀도 수치도 원문에 구체적으로 나오지 않는다.

## 참고

- [Secret protection must scale with software — GitHub Blog](https://github.blog/ai-and-ml/github-copilot/secret-protection-must-scale-with-software/)
