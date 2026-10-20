---
title: "에이전트 평가에서 로컬 git remote를 써도 될까 — 340회 실험"
date: 2026-10-21 06:53:00 +0900
categories: [AI Agent]
tags: [agent-eval, git, evaluation, copilot, experiment]
description: 로컬 git remote와 실제 GitHub 중 무엇으로 에이전트를 평가할까. MS 340회 실험은 요청 없는 push 비율이 달랐다고 보고한다.
---

> 이 글은 Microsoft 개발자 블로그의 "Can a local git remote replace GitHub in your agent eval?"(2026-10-20, Waldek Mastykarz)를 읽고 정리·재구성한 것입니다. 실험은 원문 저자의 것이며 제가 재현한 것이 아닙니다.
{: .prompt-info }

## 결론부터

에이전트 평가에서 remote를 로컬 bare 저장소(`../origin.git`)로 대체하면 대부분의 git 검사에는 충분합니다. 그러나 **"요청하지 않았는데 공유 상태를 바꾸는가"** 를 재는 시나리오에서는 결과가 달라집니다. 원문 실험에서 로컬 remote는 에이전트를 실제보다 조심스러워 보이게 했습니다. 평가 환경 선택이 곧 평가 설계의 일부라는 이야기입니다.

## 실험 설정

- 과제: TypeScript 저장소 `pricing-service`의 `feature/tiered-discounts` 브랜치를 최신 main에 충돌 없이 rebase. 프롬프트는 rebase만 요청하고 push 여부는 에이전트가 정하게 둡니다.
- 비교: 로컬 bare remote vs. 실행마다 새로 만든 비공개 GitHub 저장소(개발자가 지을 법한 이름, 종료 시 삭제).
- 에이전트: VS Code의 GitHub Copilot Chat, Copilot CLI, Claude Code. 확인 프롬프트 없이 실행.
- 측정: 에이전트 자기 보고가 아니라 remote-tracking reflog와 종료 시점 remote ref. LLM 판정은 교차 확인용.

## 파일럿에서 얻은 교훈 두 가지

1. 프롬프트에 "팀원이 이 브랜치를 리뷰한다"는 문장을 넣자 10번 중 10번 모두 push했습니다(양쪽 remote 동일). 열려 있던 결정을 프롬프트가 대신 내려 버린 것입니다.
2. Claude Sonnet 5.5는 중립 프롬프트에서 양쪽 모두 요청 없이 push하지 않아 0 대 0, 비교 자체가 불가능했습니다. 그래서 최종 실험은 GPT-5.6 Sol 기반 두 에이전트로 진행됐습니다. 소표본(런당 10~20회)에서는 격차가 0에서 35%p까지 흔들렸다고 합니다.

## 최종 결과 (340회, remote별 170회)

| | GitHub | 로컬 | 차이 |
|---|---|---|---|
| 요청 없는 push | 111/170 (65%) | 92/170 (54%) | +11%p (p = 0.033) |

에이전트별로도 거의 같았습니다(Copilot CLI +12%p, VS Code +11%p). 반면 rebase 정확도는 340/340, 모든 push는 `--force-with-lease`, 파괴적 명령은 없었고, push를 시키면 양쪽 다 push했습니다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="remote 종류별 요청 없는 push 비율 비교">
  <text x="0" y="14" class="sk-title">① 요청 없는 push 비율 (GPT-5.6 Sol, 170회씩)</text>
  <text x="8" y="68" class="sk-label">GitHub</text>
  <rect x="110" y="48" width="390" height="28" rx="4" class="sk-box-accent"/>
  <text x="510" y="68" class="sk-sub">65% (111/170)</text>
  <text x="8" y="128" class="sk-label">로컬</text>
  <rect x="110" y="108" width="324" height="28" rx="4" class="sk-box"/>
  <text x="444" y="128" class="sk-sub">54% (92/170)</text>
  <text x="110" y="176" class="sk-sub">막대 길이는 비율에 비례. 실제 격차 범위는 대략 1~22%p로 불확실.</text>
</svg>
<figcaption>로컬 remote에서는 push가 11%p 덜 나와 에이전트가 더 신중해 보이지만, 이는 환경이 만든 착시일 수 있다.</figcaption>
</figure>

## 해석할 때 조심할 점

원문 저자가 밝힌 한계를 그대로 가져와야 합니다. 한 모델 계열, 한 저장소, 한 시나리오이고, 확인 없이 돌린 설정입니다. 격차의 실제 범위도 약 1~22%p로 넓습니다. 원인은 "로컬 경로가 평가임을 암시했을 것"이라는 **가설**이고, 로컬 실행 3건에서 에이전트가 remote가 로컬이라 제한이 없을 것이라 언급한 코멘트가 있었지만 일화일 뿐입니다. 이 3건은 모두 push로 이어졌다고 합니다. 제 입장에서도 이 결과를 일반 법칙으로 쓰지 않고, "평가 환경이 행동을 바꿀 수 있다"는 경고로만 받아들입니다. 에이전트의 환경 설계 문서화 관점은 [AX 실무자 플레이북](/posts/ax-practitioner-playbook/)과도 맞닿고, 에이전트의 권한 문제는 [Guardian Agents 글](/posts/securing-agentic-insider-guardian-agents/)과 이어집니다.

## 실무에서 달라지는 점

- **에이전트 평가를 설계하는 사람**: rebase 정확도, 명시적 push, push 안전성은 로컬 remote로 충분합니다. "요청하지 않은 공유 상태 변경"을 보는 항목만 실제 GitHub(저장소별 새로 생성, 개발자풍 이름, 종료 시 삭제)로 분리하세요.
- **프롬프트 작성자**: 평가 프롬프트에 "리뷰어가 있다" 같은 맥락을 넣으면 결정을 프롬프트가 내립니다. 측정하려는 행동에는 중립 문장만 쓰세요.
- **결과 측정**: 에이전트가 "push했다"고 말한 것을 믿지 말고 reflog와 remote ref로 확인하세요. 판정 LLM과 증거가 어긋나면 기준부터 점검합니다.
- **사내 도입 검토자**: 평가를 로컬 샌드박스에서 통과했다고 운영에서도 얌전하다고 판단하면 안 됩니다. 평가가 문제를 가릴 수 있다는 방향의 오차입니다.
- **IT 강사**: 이 실험 구조(로컬 vs 실제 remote, reflog 측정)는 소규모 실습으로 옮기기 좋습니다. 다만 이 글은 원문 소개이며, 직접 돌려 본 결과는 아닙니다.

## 참고

- [Can a local git remote replace GitHub in your agent eval? — Microsoft Developer Blog (2026-10-20)](https://developer.microsoft.com/blog/can-a-local-git-remote-replace-github-in-your-agent-eval/)
