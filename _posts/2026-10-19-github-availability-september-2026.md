---
title: "GitHub 9월 가용성 보고서 — 백그라운드 작업과 재시도가 장애를 키운 방식"
date: 2026-10-19 06:51:00 +0900
categories: [IT News]
tags: [github, availability, incident, copilot, azure]
description: GitHub이 9월 장애 두 건과 Azure 이전 진행 상황을 공개했다. 원인과 재발 방지책, 그리고 팀이 가져갈 점검 항목을 정리했다.
---

> 이 글은 GitHub의 "GitHub availability report: September 2026"(2026-10-14)을 읽고 정리·재구성한 것입니다. 수치는 원문 값이고, 해석은 따로 표시했습니다.
{: .prompt-info }

**결론부터.** 9월 장애는 두 건이고, 둘 다 "외부 공격"이 아니라 **내부 변경이 보호 장치를 우회한 경우**였습니다. 하나는 백그라운드 정리 작업이 DB 주 서버를 포화시킨 사건, 다른 하나는 Copilot 코드 리뷰 결과가 일부 고객에게만 전달된 사건입니다. GitHub에 의존하는 팀은 이 두 사건에서 자기 파이프라인의 점검 항목을 뽑을 수 있습니다. 앞선 [10월 첫째 주 GitHub 소식](/posts/github-early-october-roundup/)과 같이 읽으면 인프라 이전 맥락이 이어집니다.

## 사건 1 — 9월 13일, DB 과부하

- 영향: 이슈·PR 생성과 수정, API 호출, 토큰 생성, Actions 워크플로 시작, 가입. 이슈 생성 웹 요청의 약 96%가 실패했고 신규 가입 약 90%, Actions 워크플로 약 4%가 간헐적으로 실패했습니다. 지속 시간은 헤더에 1시간 28분, 영향 요약에는 약 2시간으로 둘 다 적혀 있습니다.
- 원인: 데이터 정리 작업이 공유 DB에 부하를 더했습니다. 안전장치는 복제본이 뒤처지면 쓰기를 늦추지만 **주 서버 부하는 보지 않았고**, 작업은 주 서버의 연결 한도까지 계속 돌았습니다.
- 증폭 요인: DB 호출에 빠른 타임아웃이 없어 요청 핸들러가 멈춘 DB를 기다렸고, 토큰 생성의 재시도 루프가 실패하는 쓰기를 계속 보내 포화를 유지했습니다.
- 재발 방지: 주 서버 여유 용량 기준의 배치 제어, 불필요한 재시도 축소, 느린 요청 영향 제한, 서비스 시작 개선.

## 사건 2 — 9월 28일, Copilot 코드 리뷰 미완료

- 영향: github.com과 GHEC 데이터 상주 리전에서 리뷰가 "reviewing" 상태로 멈췄습니다. 지연은 30분에서 최대 2시간. 지속 시간은 헤더 52분, 요약 약 2시간 45분으로 두 값이 적혀 있습니다.
- 원인: 리뷰 서비스는 코멘트가 없는 제한본과 완성본 두 결과를 돌려주고, Copilot은 완성본으로만 게시합니다. 변경 때문에 일부 고객에게 제한본만 갔습니다. 그런데 작업은 **"성공"으로 보고**되었고 모니터링은 결과 누락을 잡지 못했습니다. 30분 넘게 미완료일 때의 경보가 감지했습니다.
- 조치: 변경을 되돌렸고, 고객에게 영향받은 리뷰를 다시 요청하라고 안내했습니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="두 장애에서 공통된 구조: 성공 신호가 있지만 실제 결과 확인이 없다">
  <defs>
    <marker id="ga-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 두 사건의 공통 구조</text>
  <rect x="8" y="46" width="200" height="60" rx="8" class="sk-box"/>
  <text x="108" y="72" text-anchor="middle" class="sk-label">내부 변경·배치 작업</text>
  <text x="108" y="90" text-anchor="middle" class="sk-sub">정리 작업 / 결과 전달 변경</text>
  <rect x="260" y="46" width="200" height="60" rx="8" class="sk-box-muted"/>
  <text x="360" y="72" text-anchor="middle" class="sk-label">보호 장치가 한쪽만 봄</text>
  <text x="360" y="90" text-anchor="middle" class="sk-sub">복제 지연만 / "성공" 신호만</text>
  <rect x="512" y="46" width="200" height="60" rx="8" class="sk-box-accent"/>
  <text x="612" y="72" text-anchor="middle" class="sk-label">장시간 영향</text>
  <text x="612" y="90" text-anchor="middle" class="sk-sub">DB 포화 / 리뷰 미게시</text>
  <path d="M210,76 L256,76" class="sk-line" marker-end="url(#ga-m1)"/>
  <path d="M462,76 L508,76" class="sk-line-accent" marker-end="url(#ga-m1)"/>
  <text x="8" y="150" class="sk-sub">원문 사실에서 이 블로그가 뽑은 공통점 (해석)</text>
</svg>
<figcaption>"작업이 성공했다"는 신호와 "결과가 실제로 전달됐다"는 확인은 다른 것이다.</figcaption>
</figure>

## 같이 공개된 진행 상황

원문은 Azure 이전 현황도 적었습니다(10월 7일 기준). Central US에 프로덕션 서비스 160개 중 120개가 배포됐고, 주 MySQL DB 51개 중 13개가 이전됐습니다. 9월 30일까지 전부 배포한다는 목표는 **달성하지 못했다**고 밝혔고, 용량 불균형으로 9월 11일 확장을 멈췄다가 9월 18일 재개했습니다. 계획된 DB 페일오버 중 자동 비상 절차가 Central US의 DB 복제본을 끊어 고객에 영향을 준 일도 적혀 있습니다. 그 절차는 계획된 전환 중에는 꺼 두었습니다.

## 실무에서 달라지는 점

해석이 섞인 항목입니다. 직접 적용해 본 결과가 아닙니다.

- **GitHub Actions·API에 의존하는 CI 담당자**: 사건 1처럼 토큰 생성과 워크플로 시작이 함께 흔들릴 수 있습니다. 자체 스크립트의 재시도에 **상한과 백오프**가 있는지 점검하세요. 원문이 재시도 루프를 증폭 요인으로 꼽았습니다.
- **Copilot 코드 리뷰를 PR 게이트로 쓰는 팀**: 리뷰가 "reviewing"에서 멈출 수 있으니, 일정 시간 넘으면 사람 리뷰어로 넘기는 규칙을 문서에 넣어 두세요. 원문 권고도 영향받은 리뷰를 다시 요청하는 것이었습니다.
- **GHEC 데이터 상주 사용 조직**: 사건 2는 상주 리전에서 더 심했습니다. 상주 리전 사용 시 장애 공지 구독(GitHub Status)을 담당자에게 연결해 두는 편이 안전합니다.
- **자체 배치 작업을 운영하는 관리자**: "복제 지연 하나만 보는 안전장치"를 점검하세요. 주 서버 부하나 연결 수 같은 지표를 같이 보는지가 이번 사건의 교훈입니다.
- **IT 강사**: 자동화 교육에서 "성공 코드 반환 ≠ 결과 도착"을 사례로 쓸 수 있습니다. [에이전트 승인 설계](/posts/durable-task-real-world-boundaries/)를 다룬 글과도 연결됩니다.

## 참고

- [GitHub availability report: September 2026](https://github.blog/news-insights/company-news/github-availability-report-september-2026/) (GitHub Blog, 2026-10-14)
