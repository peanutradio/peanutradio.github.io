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

배포 전 평가는 에이전트를 어느 정도까지 끌어올립니다. 문제는 그 뒤입니다. 트래픽은 평가 세트에서 멀어지고, 모델·도구가 바뀌어도 헬스 체크는 초록색입니다. AQuA는 **운영 중인 에이전트 옆에서 세션을 샘플링해 실패를 묶고, 별도 모델로 검증한 뒤, 원인을 코드 줄 범위까지 짚어 주는** 오픈소스 참조 구현입니다. 요청 경로에 끼지 않고 에이전트에 쓰기도 하지 않습니다. 앞서 정리한 [ReviewBench](/posts/github-early-october-roundup/)가 "평가를 어떻게 재나"였다면, 이 글은 배포 이후 평가입니다.

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

검증은 실행당 최대 50개 클러스터까지 합니다. 원문은 내부 87개 트레이스 세트에서 검증기가 후보 클러스터 24개 중 4개를 기각했다고 밝힙니다.

근본 원인 진단은 필요할 때 따로 돌립니다(대시보드 Chat 또는 `agents-cli aqua run`, Gemini 3.8 Flash 사용). 배포 시점에 저장한 소스 스냅샷을 읽어 `<경로>:<시작줄>-<끝줄>` 형태로 인용하고 수정안을 제안하지만, 스스로 적용하거나 PR을 열지는 않습니다. 결함이 저장소 밖(의존성, 에이전트 간 핸드오프, 검색해 온 데이터)에 있으면 코드 수정안 없이 해당 단계에 원인을 돌립니다.

## 결과를 믿게 만드는 장치

AQuA는 자기 확신도 점수를 내지 않습니다. 대신 인사이트마다 Cloud Trace의 세션 ID와 코드 인용을 증거로 붙이고, 서버가 인용을 스냅샷과 대조해 존재하지 않는 파일·줄을 가리키면 기각합니다. 기각된 클러스터, 검증 한도 초과, 채점 오류, 빈 트레이스 구간은 "문제 없음"으로 세지 않고 실행 기록에 따로 남깁니다. 사람이 **Dismiss**를 누른 발견은 다시 NEW로 올라오지 않습니다.

배치 구조도 정리해 두면, 트레이스는 Cloud Trace·Cloud Logging·BigQuery에서 읽고, 대시보드는 Cloud Run, 인사이트는 BigQuery 데이터셋에 쌓입니다. 배포 시 소스 스냅샷은 Cloud Storage에 배포 리비전별로 저장됩니다. 원본 대화와 스냅샷은 사용자 프로젝트 안에 머물고, 서비스 계정 권한과 Identity-Aware Proxy 뒤에서 접근합니다. 일정에 따라, 배포 직후, 또는 수동으로 돌릴 수 있습니다. 판정 기준은 `goal.md`(모든 리뷰 프롬프트에 붙는 평문 목표)와 `eval_config.yaml`(결정적 Python 지표)로 조정합니다.

## 사례: travel-concierge

- 32세션 중 5개만 깨끗하게 통과했고, 42개 발견이 9개 클러스터로 묶였습니다.
- 검증에서 3개가 거짓 양성으로 기각되어 6개가 남았습니다.
- 가장 큰 문제는 사용자가 말한 좌석을 가용성 확인 없이 저장한 것(15세션)이었고, 채식 조건이 다음 에이전트로 넘어가지 않아 스테이크를 추천한 것(7세션)이 뒤를 이었습니다.
- 세 번째는 도구 없이(`tools=[]`) 검증된 필드를 요구한 프롬프트 때문에 `example.com` URL을 지어낸 문제(5세션)로, 사례에서는 고치지 않고 추적만 했습니다.
- 프롬프트 한 줄씩 두 곳을 고치자 좌석 문제는 15세션에서 2세션으로, 채식 누락은 7에서 0으로 줄었고, 통과가 5/32에서 13/32로 늘었습니다.

## 비용

원문 기준으로 96세션 단일 에이전트 스윕이 약 $0.70(세션당 약 $0.007), 위 사례(32세션, 스팬 1,583개)가 $3.76(세션당 약 $0.12)입니다. 리뷰·클러스터링은 Gemini 3.1 Pro, 검증은 Gemini 3.7 Flash를 썼고 Gemini 플랫폼 표준 단가로 계산했습니다. 근본 원인 분석은 인사이트당 $0.33~$2.47입니다. 세션 길이에 따라 비용이 크게 달라진다는 점이 숫자에서 읽힙니다.

## 한계와 내 해석

원문이 밝힌 한계는 무작위 샘플링만 한다는 점(재시도·지연 급증 같은 사전 필터는 과제), 판정 모델을 전문가 기준에 맞추는 일이 아직 숨은 공정이라는 점, 수백 턴짜리 긴 대화에는 전체 전사 방식이 안 맞는다는 점입니다.

여기서부터는 제 해석입니다. 이 구조는 Google Cloud(Cloud Trace, BigQuery, Gemini)에 묶여 있지만, "샘플링 → 채점 → 클러스터 → **검증** → 추적"이라는 뼈대는 Foundry 트레이스에도 옮길 수 있을 것 같습니다. 특히 검증 단계가 거짓 양성을 걸러내는 장치라는 점이 가져올 만합니다. 로컬 데모는 클라우드 없이 `make demo`로 돌릴 수 있습니다. 라이선스는 원문에 명시가 없지만, 저장소(google/adk-recipes)는 GitHub 기준 Apache-2.0으로 표시되어 있습니다.

원문이 밝힌 한계를 하나 더 보태면, `adk run --replay`는 기록된 사용자 발화를 다시 보낼 뿐 DB 행이나 API 타임아웃 같은 외부 상태는 재현하지 않습니다. 수정 전후 비교 수치를 읽을 때 감안해야 할 부분입니다.

## 실무에서 달라지는 점

사내에서 에이전트를 운영하는 인프라 담당자, Azure 쪽에서 비슷한 구조를 고민하는 사람, 그리고 에이전트 운영을 가르치는 강사 입장에서 정리합니다.

1. **권한 설계가 먼저입니다.** AQuA는 에이전트에 쓰지 않지만, 운영 트레이스 전체를 읽고 BigQuery와 대시보드에 자기 기록을 씁니다. 원본 대화에 개인정보가 섞여 있다면, 진단 에이전트용 서비스 계정에 어떤 데이터셋 읽기 권한을 줄지, 대시보드 접근을 IAP로 누구에게 열지를 보안 담당과 먼저 정해야 합니다.
2. **비용은 세션 길이가 좌우합니다.** 같은 원문 안에서도 세션당 $0.007과 $0.12로 열 배 넘게 차이 납니다. 샘플 상한(최대 1,000세션)과 실행 주기를 정할 때 우리 에이전트의 평균 스팬 수부터 재고 예산을 잡는 게 순서입니다.
3. **Azure로 옮긴다면 '검증 단계'와 '인용 검증'을 빼먹지 않습니다.** Foundry나 Application Insights 트레이스로 같은 뼈대를 짤 수는 있겠지만, 이건 제 해석이고 원문이 다루는 범위는 아닙니다. 핵심은 클러스터를 다른 모델로 한 번 더 확인하는 단계와, 코드 인용이 실제로 존재하는지 기계적으로 검사하는 단계입니다. 이 둘이 빠지면 그럴듯한 오진단이 쌓입니다.
4. **판정 모델 보정 공수를 일정에 넣습니다.** 원문도 전문가 기준에 맞춘 보정을 "실제 엔지니어링 작업"이라고 부릅니다. 도입 계획에 도메인 담당자가 샘플을 직접 채점하는 시간을 따로 잡아 두지 않으면 결과를 믿기 어렵습니다.
5. **강의에서는 '자동 수정 안 함'을 설계 선택으로 설명합니다.** 수정안을 제안만 하고 사람이 적용하는 구조는 [WinDbg MCP](/posts/windbg-mcp-ga/)가 실행 명령을 화면에 보여 주는 것과 같은 원칙입니다. 에이전트 운영 도구는 증거와 함께 사람에게 넘기는 지점이 있어야 한다는 점을 함께 묶어 가르치면 좋습니다.

## 참고

- [The outer loop: insights first — Google Developers Blog](https://developers.googleblog.com/the-outer-loop-insights-first-an-ambient-quality-agent-that-diagnoses-your-production-agent/)
- [google/adk-recipes — ambient-quality-agent](https://github.com/google/adk-recipes/tree/main/core/python/ambient-quality-agent)
