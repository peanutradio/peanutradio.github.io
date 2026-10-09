---
title: "10월 첫째 주 GitHub 소식 — 에이전트 규모 git 인프라·ReviewBench·시크릿 보호"
date: 2026-10-08 07:10:00 +0900
categories: [IT News]
tags: [github, git, code-review, secret-protection, ai-agent, copilot]
description: GitHub이 10월 첫째 주에 낸 글 세 편을 묶었다. 에이전트 규모 git 인프라 재설계, AI 코드 리뷰 벤치마크 ReviewBench, ModernBERT 시크릿 분류기와 관리자가 챙길 점.
---

> 이 글은 GitHub Blog에 10월 6~7일 올라온 글 세 편을 읽고 정리·재구성한 것입니다. 수치는 모두 GitHub 자체 집계이며, 원문 링크는 맨 아래 참고에 있습니다.
{: .prompt-info }

## 결론부터

세 글은 주제가 달라 보이지만 같은 이야기를 한다. **에이전트가 사람보다 훨씬 많은 코드를 쓰기 시작했고, GitHub은 그 양을 감당하도록 플랫폼을 고치고 있다.** 저장소 쪽에서는 git 인프라를 저장과 연산이 분리된 구조로 다시 짓고 있고, 품질 쪽에서는 AI 코드 리뷰를 재는 공개 벤치마크 ReviewBench를 냈고, 보안 쪽에서는 모양이 정해지지 않은 비밀 값까지 push 단계에서 잡는 ModernBERT 분류기를 내놨다.

관리자 입장에서 당장 손댈 것은 세 번째, 시크릿 보호다. 10월 안에 Enterprise Cloud와 Teams에서 모델 기반 push protection이 열리는데, 이게 **AI 크레딧을 소모한다.** 나머지 두 개는 바로 설정할 것은 없지만 에이전트 도입 계획을 세울 때 판단 근거로 쓸 만하다.

## ① git 인프라 — 에이전트 규모로 다시 짓는다

10월 6일 Brian Celenza가 쓴 글이다. 먼저 숫자부터 보면 왜 이 작업을 하는지 바로 보인다.

- 월간 git 이벤트: 2025년 9월 2,182억 건 → 2026년 8월 4,733억 건, 두 배 넘게 증가
- 2026년 9월 커밋 73.8억 건, 1년 전의 다섯 배 이상
- 푸시: 월 6.9억 건 → 33.5억 건, 4.9배
- PR 머지는 1년 전의 거의 4배, GitHub Actions 실행은 9월 32.6억 회로 4배 이상
- 가장 바쁜 저장소 하나가 2026년 8월에 약 10억 요청을 받았다

사람의 타이핑 속도를 전제로 설계한 시스템에 에이전트 여러 개가 동시에 달라붙으면, 저장소 크기보다 **동시에 쓰는 주체의 수**가 병목이 된다. GitHub이 내건 원칙은 두 가지다. 하나는 조율 최소화다. 브랜치 ref를 갱신하는 임계 경로만 줄을 세우고, 오브젝트 저장이나 시크릿 스캔처럼 병렬로 돌려도 되는 일은 그 경로 밖으로 뺀다. 다른 하나는 저장과 연산의 분리다. 원본 저장소 데이터는 Azure Blob Storage에 두고, 읽기는 가볍게 늘릴 수 있는 컴퓨트가 맡는다.

<figure class="sketch">
<svg viewBox="0 0 720 240" role="img" aria-label="쓰기 경로는 ref 갱신만 조율하고, 오브젝트 저장과 스캔은 병렬로, 원본은 Azure Blob Storage, 읽기는 독립 워커가 맡는 구조">
  <defs>
    <marker id="ro-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 저장과 연산을 분리한 git 인프라</text>
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
  <path d="M160,86 L248,66" class="sk-line" marker-end="url(#ro-m1)"/>
  <path d="M160,92 L248,150" class="sk-line" marker-end="url(#ro-m1)"/>
  <path d="M442,66 L528,82" class="sk-line" marker-end="url(#ro-m1)"/>
  <path d="M442,156 L528,100" class="sk-line" marker-end="url(#ro-m1)"/>
  <path d="M620,114 L620,148" class="sk-line" marker-end="url(#ro-m1)"/>
</svg>
<figcaption>쓰기는 꼭 줄 세워야 하는 곳에서만 줄 세우고, 나머지는 읽기와 따로 늘린다.</figcaption>
</figure>

새 구조는 내부 벤치마크에서 쓰기 처리량이 최대 35배 높았다고 한다. 측정 조건은 글에 자세히 나오지 않으니 우리 저장소의 체감 속도로 읽으면 안 된다. 브랜치, 리뷰, 머지 큐, 브랜치 보호, 감사 로그 같은 기존 워크플로는 그대로 두는 것이 전제이고, 유지보수 시간 없이 서비스를 돌리면서 교체한다고 한다. 출시 일정은 없다. 더 자세한 구조는 후속 글에서 다룬다고 했다.

## ② ReviewBench — AI 코드 리뷰를 재는 공개 벤치마크

같은 날 나온 ReviewBench는 코드 리뷰 에이전트의 품질을 숫자로 비교하려는 시도다. 오픈소스 라이선스 저장소 187곳의 공개 PR 219건, 19개 언어로 구성했다. GitHub 전체 PR 1억 390만 건의 분포를 참고해 언어와 저장소 크기를 맞추되, 한두 줄 수정보다는 여러 파일을 건드리는 PR 쪽으로 일부러 가중했다.

정답은 사람 리뷰, 분석 도구, 프런티어 모델, 후속 커밋에서 지적 사항을 모으고, 의미가 같은 것을 합친 뒤, 공통 루브릭으로 LLM 판정자(Claude Sonnet 5)가 걸러서 만든다. 지적 사항은 심각도(Critical·Medium·Low)와 범주(정확성·보안·신뢰성·유지보수성·테스트 등)로 나뉜다. 독립된 시니어 엔지니어들이 정답을 다시 라벨링했을 때 일치율은 96.6%였다고 한다.

내가 보기에 가장 잘 설계한 부분은 지표다. 고정 정답 기준의 Grounded 정밀도·재현율·F1과 함께, 정답지에 없던 새 지적까지 판정자가 진위를 가려 반영하는 Augmented 계열을 따로 둔다. 잘하는 리뷰어일수록 정답지에 없는 걸 찾아내기 때문이다. 다만 Augmented 재현율은 에이전트마다 분모가 달라지므로 대표 비교는 Grounded 재현율로 하라고 원문이 직접 적어 두었다. 정밀도와 재현율 가중치를 조정하는 Fβ도 있다.

벤치마크 예측이 실서비스와 맞았는지도 확인했다. Copilot 코드 리뷰의 멀티 모델 앙상블 실험에서 온라인 재현율은 13.6%, 정밀도(지적이 실제로 반영된 비율)는 8.0% 올랐고, 리뷰당 비용은 8.0% 내렸다. 코멘트 수는 61% 늘었고, Critical 코멘트는 벤치마크가 227% 증가를 예측했는데 온라인은 262%였다. 방향은 모두 맞았지만 크기까지 맞은 것은 아니고, 실험 수도 제한적이라고 원문이 인정한다.

자체 에이전트를 올리려면 review-bench.ai에서 GitHub 로그인 → 컨테이너 이미지·설정·자기 모델 키로 에이전트 등록 → 25건 테스트셋 → 219건 전체 3회 실행 → 메인테이너 승인 순서를 밟는다. 판정자가 LLM 하나라는 점, 정답이 본질적으로 불완전하다는 점, 연구 프리뷰라 벤치마크가 바뀔 수 있다는 점은 감안해야 한다.

## ③ 시크릿 보호 — 에이전트 속도에 맞춘 ModernBERT 분류기

10월 7일 글은 비밀 정보 유출 문제를 다룬다. 지금 GitHub PR의 약 3건 중 1건에 AI 에이전트가 관여하고, 1년 전에는 10건 중 1건이 안 됐다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="GitHub이 제시한 비밀 정보 관련 핵심 수치 네 가지">
  <text x="0" y="14" class="sk-title">② 시크릿 유출 — GitHub 자체 집계 수치</text>
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
<figcaption>유출은 빠르게 늘고 수습은 느리니, 들어오기 전에 막는 비중을 키워야 한다.</figcaption>
</figure>

2024년 2분기부터 2026년 2분기까지 검사된 push는 2.84배, 자격 증명이 든 push는 2.59배 늘었다. 그런데 push 단위 유출 비율에는 아홉 분기 동안 통계적으로 뚜렷한 추세가 없었고, 개발자가 push protection 차단을 우회한 비율은 6.63%에서 3.93%로 줄었다. 사람이 부주의해진 것이 아니라 코드의 양이 늘었다는 해석이다. 현재 push protection이 새로 탐지된 비밀의 약 30%를 저장소 기록에 들어가기 전에 막고, 나머지 70%는 노출된 뒤에 발견된다.

새 분류기는 Microsoft Applied Sciences와 함께 미세 조정한 ModernBERT 모델이다. 정해진 형식의 키는 기존 패턴 매칭이 잘 잡지만, 사내 DB 비밀번호처럼 모양이 없는 비밀은 놓친다. 분류기는 후보 값을 주변 코드 맥락과 함께 보고 판정하며, 후보 묶음 하나를 2밀리초 미만에 처리한다. DB URL이나 Kubernetes Secret 매니페스트, Dockerfile 속 비밀번호는 잡고 `changeme` 같은 자리표시자는 통과시키는 식이다. GitHub은 막을 수 있는 비밀이 두 배 넘게 늘 수 있다고 추정하고, 기존 LLM 기반 파이프라인보다 정밀하다고 주장한다. 다만 정밀도나 오탐률 수치는 원문에 없다.

배포 일정은 이렇다. 지금은 push protection이 비공개 프리뷰이고, AI 비밀 탐지를 쓰던 조직은 발표 시점부터 새 모델로 자동 전환됐다. push 이후 스캔 알림은 추가 비용이 없다. 10월 중 Enterprise Cloud와 Teams의 GitHub Secret Protection 조직에 모델 기반 push protection이 열리며 AI 크레딧을 소모한다. GitHub Enterprise Server 3.23에는 폐쇄망 환경을 포함해 공개 프리뷰로 들어간다. Copilot CLI와 Copilot 앱의 `/security-review` 명령에도 들어가서, 조직에 Secret Protection 플랜이 없어도 push 전에 점검할 수 있고, 이때 크레딧은 AI 사용 인사이트에서 Secret Protection 항목으로 잡힌다.

## 실무에서 달라지는 점

사내 인프라 담당, M365·Azure·GitHub 관리자, 그리고 이 내용을 가르치는 강사 입장에서 정리하면 이렇다.

1. **Secret Protection 조직이라면 AI 크레딧 예산부터 본다.** 모델 기반 push protection은 크레딧을 소모하고, push 이후 알림만 무료다. 켜기 전에 Enterprise 설정의 AI 사용 인사이트에서 Secret Protection 항목이 얼마나 잡히는지 지켜볼 기준을 정하고, 크레딧 한도·알림을 미리 걸어두는 게 순서다.
2. **Copilot CLI 사용자는 플랜 없이도 크레딧을 쓴다.** `/security-review`는 Secret Protection 플랜이 없어도 동작하지만 사용량은 Secret Protection으로 집계된다. 청구서에 예상 못 한 항목이 보일 수 있으니 Copilot 정책 담당과 비용 담당이 이 사실을 같이 알고 있어야 한다.
3. **폐쇄망 GHES라면 3.23 업그레이드 계획에 넣는다.** 공개 프리뷰지만 air-gapped 환경에서 AI 탐지가 된다는 점은 그동안 클라우드 전용 기능을 못 쓰던 조직에 의미가 크다. 프리뷰 기능이므로 운영 반영 전에 테스트 조직에서 알림 품질을 먼저 확인한다.
4. **분류기는 마지막 방어선일 뿐이다.** 에이전트가 읽고 쓰는 `.env`, 설정 파일, 매니페스트에 진짜 키를 두지 않는 것이 먼저다. Azure라면 Key Vault나 관리 ID로 키 자체를 코드에서 빼는 구성을 기본으로 가르치고, push protection 우회 사유 기록을 감사 로그로 주기적으로 본다.
5. **리뷰 에이전트를 고를 때 ReviewBench를 질문지로 쓴다.** 벤더에 Grounded 재현율과 정밀도를 묻고, 판정자가 LLM이라는 한계를 감안해 사내 PR 몇십 건으로 따로 비교해 보는 것이 맞다. 강의에서는 "코멘트가 많다 = 좋은 리뷰"가 아니라는 점, 정밀도와 재현율의 균형을 먼저 짚는다.
6. **git 인프라 재설계는 당장 설정할 것이 없다.** 일정도 플랜별 적용도 아직 없다. 다만 에이전트 여러 개가 한 저장소에 동시에 커밋하는 구성을 설계 중이라면, 지연의 원인이 내 쪽이 아닐 수도 있다는 점과 브랜치 보호·머지 큐 같은 기존 통제는 그대로 유지된다는 점을 기억해 두면 된다.

## 참고

- [Building Git infrastructure for agent-scale development — GitHub Blog (2026-10-06)](https://github.blog/engineering/architecture-optimization/building-git-infrastructure-for-agent-scale-development/)
- [ReviewBench: An open benchmark for AI code review — GitHub Blog](https://github.blog/ai-and-ml/github-copilot/reviewbench-an-open-benchmark-for-ai-code-review/)
- [ReviewBench 사이트](https://review-bench.ai)
- [Secret protection must scale with software — GitHub Blog (2026-10-07)](https://github.blog/ai-and-ml/github-copilot/secret-protection-must-scale-with-software/)
