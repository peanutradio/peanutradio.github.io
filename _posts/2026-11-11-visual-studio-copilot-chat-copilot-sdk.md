---
title: "Visual Studio Copilot Chat, 엔진이 Copilot SDK로 바뀐다 — 18.12 전에 볼 것"
date: 2026-11-11 06:53:00 +0900
categories: [IT News]
tags: [copilot, github, ai-agent, devops]
description: Visual Studio 2026 18.12부터 Copilot Chat이 Copilot SDK 위에서 기본으로 동작한다. 달라지는 기능, 제조사 벤치마크의 한계, 팀이 점검할 항목을 정리했다.
---

> 이 글은 Visual Studio 블로그의 "Copilot in Visual Studio Just Got a Major Upgrade"(2026-11-10, Dalia Abo Sheasha)를 읽고 정리·재구성한 것이다. 직접 실행해 본 내용은 아니며, 수치는 모두 원문(제조사) 발표 기준이다.
{: .prompt-info }

## 결론부터

Visual Studio 2026 **18.12부터 Copilot Chat의 엔진이 바뀐다.** GitHub의 Copilot SDK 위에서 동작하는 새 방식이 **기본으로 켜져** 있고, 끄려면 옵션에서 직접 해제해야 한다. 모델은 그대로이고 달라지는 것은 모델을 감싼 하네스(작업 루프)다. 원문에는 GA나 프리뷰라는 표기가 없다. "기본 켜짐"이라는 사실만 확인되므로, 팀 단위로는 18.12 업데이트가 곧 전환 시점이라고 보는 편이 안전하다.

## 원문이 말하는 것

### 엔진 교체

새 Copilot Chat은 GitHub Copilot CLI, GitHub Copilot 앱, VS Code가 쓰는 것과 같은 Copilot SDK(하네스) 위에서 돈다. Visual Studio만 따로 가던 채팅 구현이 다른 Copilot 제품과 같은 엔진으로 합쳐진다는 뜻이다. 하네스가 무엇인지는 [Copilot 하네스가 Foundry Agent Service에 들어온 소식](/posts/copilot-harness-foundry-agent-service-preview/)에서 먼저 다뤘다.

### 벤치마크 수치

원문은 .NET 팀이 실제 C# 프로젝트의 이슈로 만든 벤치마크 157개 과제를 두 방식에 모두 돌렸다고 밝힌다. 과제는 실패하던 테스트가 통과하고, 원래 통과하던 테스트는 깨지지 않을 때만 해결로 센다. 같은 모델(GPT-5.6 Sol)로 기존 방식은 103개, 새 방식은 124개를 해결했다. 그 차이가 21개다. 본문 서두에는 100개 넘는 코딩 과제에서 테스트한 모든 모델에서 새 방식이 앞섰고, 최대 21개를 더 풀었다고 적혀 있다. 두 수치가 같은 실행인지는 원문에 설명이 없다.

내가 계산해 보면 103/157은 약 66%, 124/157은 약 79%다. 이 비율은 원문에 없는 내 계산이다.

### 새 기능

- 작업 도중 새 지시를 입력하면 에이전트가 받아 간다. 반영되기 전에는 메시지가 Pending으로 표시되고, X로 철회할 수 있다.
- 각 단계가 시도한 것뿐 아니라 돌아온 결과까지 보여 준다. 펼쳐서 결과를 보거나, 실행한 명령을 복사할 수 있다.
- 일을 위임받은 서브에이전트가 자기 카드를 갖는다. 카드에는 사용 중인 모델과 단계가 실시간으로 나온다.
- 작업 디렉터리를 솔루션 밖의 임의 폴더로 바꾸거나 폴더째 컨텍스트로 붙일 수 있다.
- Agent 모드에서 `/`를 입력해 내장 에이전트를 부른다. 원문이 이름을 밝힌 것은 `/security-review`(변경분 보안 점검)와 `/rubber-duck`(다른 모델이 계획을 비평) 둘이다.

### 아직 빠진 것

계획 모드와 채팅 내보내기는 이식 중이라고 한다. 일정은 없다. 옵션 해제 경로는 Tools > Options > GitHub > Copilot > Copilot Chat의 "Use new Copilot chat experience"이고, 해제 후 재시작이 필요하다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="Visual Studio Copilot Chat의 기존 엔진과 Copilot SDK 기반 새 엔진, 그리고 같은 SDK를 쓰는 다른 제품들">
  <defs>
    <marker id="vsc-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 18.12에서 바뀌는 것 — 채팅 화면 아래의 엔진</text>
  <rect x="8" y="50" width="170" height="56" rx="8" class="sk-box-muted"/>
  <text x="93" y="74" text-anchor="middle" class="sk-label">기존 채팅 엔진</text>
  <text x="93" y="94" text-anchor="middle" class="sk-sub">옵션 해제 시 유지</text>
  <rect x="8" y="140" width="170" height="56" rx="8" class="sk-box-accent"/>
  <text x="93" y="164" text-anchor="middle" class="sk-label">Copilot SDK 엔진</text>
  <text x="93" y="184" text-anchor="middle" class="sk-sub">18.12 기본값</text>
  <path d="M180,168 L300,168" class="sk-line-accent" marker-end="url(#vsc-m1)"/>
  <rect x="304" y="140" width="190" height="56" rx="8" class="sk-box"/>
  <text x="399" y="164" text-anchor="middle" class="sk-label">Visual Studio 채팅</text>
  <text x="399" y="184" text-anchor="middle" class="sk-sub">서브에이전트 카드·Pending</text>
  <rect x="540" y="50" width="170" height="32" rx="8" class="sk-box"/>
  <text x="625" y="71" text-anchor="middle" class="sk-label">VS Code</text>
  <rect x="540" y="92" width="170" height="32" rx="8" class="sk-box"/>
  <text x="625" y="113" text-anchor="middle" class="sk-label">Copilot CLI</text>
  <rect x="540" y="134" width="170" height="32" rx="8" class="sk-box"/>
  <text x="625" y="155" text-anchor="middle" class="sk-label">GitHub Copilot 앱</text>
  <text x="625" y="196" text-anchor="middle" class="sk-sub">같은 SDK 계열 — 스킬 공유</text>
</svg>
<figcaption>채팅 UI는 그대로 Visual Studio인데, 그 아래 엔진이 다른 Copilot 제품과 같아진다.</figcaption>
</figure>

## 내가 읽은 포인트

### 같은 모델에서 21개 차이는 모델이 아니라 하네스의 차이다

모델 이름을 바꾸지 않고도 결과가 달라진다는 점이 이번 소식의 핵심이다. 지금까지 "모델을 바꾸면 결과가 달라진다"는 이야기가 많았는데, 이번에는 같은 GPT-5.6 Sol에서 하네스만 바꿔 해결 수가 올랐다는 주장이다. 다만 이건 Microsoft .NET 팀이 고른 C# 과제 157개에서 나온 제조사 자체 수치이고, 과제 목록과 실행 조건은 원문에 없다. 우리 코드베이스에서 같은 폭이 나온다는 보장은 없다.

### 하네스 통합의 실익은 스킬 재사용이다

원문은 에이전트 스킬을 한 번 쓰면 Visual Studio, VS Code, Copilot CLI, GitHub Copilot 앱 네 곳에서 수정 없이 돈다고 말한다. 팀 표준 스킬을 만든 조직에는 이쪽이 벤치마크보다 실질적인 이득일 수 있다. SKILL.md 형식의 흐름은 [Agent Skills가 AAIF로 넘어간 글](/posts/agent-skills-aaif-donation/)과 이어서 읽으면 좋다.

### 승인·권한 모델은 원문에 없다

Pending 메시지로 작업 중 방향을 바꾸는 것은 편하지만, 작업 디렉터리를 솔루션 밖으로 바꿀 수 있다는 점은 보안 쪽에서 따져 볼 대목이다. 원문은 이 경우의 권한 확인이나 승인 흐름을 설명하지 않는다. 권한 승인 쪽은 [Copilot 보조 권한 GA](/posts/copilot-assisted-permissions-ga/)에서 정리한 것과 별개로 확인해야 한다.

## 실무에서 달라지는 점

### 1. 18.12 배포 전에 "기본 켜짐"을 공지한다

Visual Studio를 자동 업데이트로 굴리는 환경이라면 어느 날 갑자기 채팅 화면이 바뀐다. 교육이나 사내 가이드 화면 캡처가 있다면 18.12 이후 것으로 갱신할 계획을 잡아 둔다. 강의에서는 Pending 표시와 서브에이전트 카드가 첫 화면의 차이라고 짚어 주면 학습자가 덜 헤맨다.

### 2. 계획 모드·내보내기를 쓰는 팀은 옵션 위치를 알려 둔다

계획 모드나 채팅 내보내기에 의존하는 워크플로가 있다면 Tools > Options > GitHub > Copilot > Copilot Chat에서 새 방식을 끄고 재시작하는 경로를 안내한다. 이식 일정이 원문에 없으니, 끄는 것은 임시 조치라고 전제하는 게 맞다. 설정을 중앙에서 강제할 수 있는지는 원문에서 확인되지 않아, 관리자는 별도로 문서를 찾아봐야 한다.

### 3. 도입 판단은 우리 과제로 A/B를 돌려서

원문 수치는 참고용이다. 옵션 하나로 두 방식을 오갈 수 있으니, 팀의 대표 과제 몇 개를 양쪽에서 같은 모델로 돌려 단계 수와 결과를 비교하는 평가를 권한다. 이 글에서는 그 평가를 해 본 것이 아니라 방법만 제안한다.

### 4. 비용 화면과 모델 정책을 다시 본다

원문 스크린샷에는 크레딧 표시가 보이지만 요금 체계 변화에 대한 설명은 없다. 서브에이전트가 병렬로 여러 개 돌면 호출이 늘 수 있으므로, 사용량 기반 과금을 쓰는 조직은 업데이트 직후 몇 주간 사용량 지표를 먼저 본다. 이는 내 추정이며 원문이 확인해 준 내용은 아니다.

### 5. 팀 스킬과 MCP 구성을 한 곳으로 모은다

네 가지 도구가 같은 엔진을 쓴다면 스킬을 제품별로 복제할 이유가 줄어든다. VS 전용으로 만든 지침이 있다면 공용 스킬로 옮길 수 있는지 점검한다. 다만 실제로 모든 도구에서 동작하는지는 각 환경에서 확인해야 한다.

## 참고

- [Copilot in Visual Studio Just Got a Major Upgrade — Visual Studio Blog (2026-11-10)](https://devblogs.microsoft.com/visualstudio/copilot-in-visual-studio-just-got-a-major-upgrade/)
- 관련 글: [Copilot 하네스가 Foundry Agent Service에](/posts/copilot-harness-foundry-agent-service-preview/), [Copilot 보조 권한 GA](/posts/copilot-assisted-permissions-ga/), [Agent Skills, AAIF로](/posts/agent-skills-aaif-donation/)
