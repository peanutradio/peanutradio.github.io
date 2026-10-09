---
title: "AX 실무자 플레이북 — 코딩 에이전트가 틀리면 모델 말고 문서를 고쳐라"
date: 2026-10-10 06:53:00 +0900
categories: [AI Agent]
tags: [ai-agent, agent-experience, evaluation, mcp, documentation]
description: Microsoft가 공개한 AX 플레이북의 핵심은 에이전트가 낡은 코드를 내놓으면 모델이 아니라 에이전트가 읽는 문서·도구·스킬을 평가하고 고치라는 것이다.
---

> 이 글은 Microsoft 개발자 블로그의 "Agent Experience(AX) Practitioner Playbook" 소개 글을 읽고 정리·재구성한 것입니다. 원문은 방법의 개요만 담고 있고 세부 내용은 PDF에 있어서, PDF 내용은 확인하지 못했습니다.
{: .prompt-info }

## 결론: 틀린 답의 원인을 모델이 아니라 "에이전트가 읽은 자료"에서 찾는다

코딩 에이전트가 폐기된 인증 패턴이나 옛 SDK 버전을 내놓았을 때, 우리는 보통 "학습 데이터가 낡았다"고 넘깁니다. 이 플레이북은 정반대로 말합니다. 에이전트는 자기가 본 자료가 시킨 대로 했을 뿐이고, 모델이 좋아지길 기다릴 게 아니라 에이전트가 참조하는 문서·MCP 도구·스킬·플러그인·지침을 고치고 그 효과를 측정하라는 것입니다. 제품을 만드는 쪽이 "에이전트가 내 제품을 제대로 고르고 쓰는가"를 품질 항목으로 다뤄야 한다는 이야기입니다.

## 원문이 말하는 네 단계

원문은 각 단계를 제목과 한두 문장으로만 소개합니다.

1. **믿을 수 있는 결과의 정의** — 결과를 신뢰하기 전에 필요한 조건을 정합니다. 코드가 실제로 실행된다는 걸 증명하는 관문이 포함됩니다.
2. **판정 기준 작성** — 심사자(사람 또는 모델)가 일관되게 적용할 기준을 쓰고, 쓰기 전에 보정하고, 제품이 바뀌면 버전을 올립니다.
3. **궤적(trajectory)으로 원인 진단** — 결과 점수만 보지 않고 에이전트가 거친 과정을 봅니다.
4. **수정을 가설로 검증** — 고친 내용이 실제로 나아졌는지 확인한 뒤, 근거를 들고 해당 자료의 담당 팀에 전달합니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="궤적 진단의 세 갈래: 로드되지 않음, 로드됐지만 호출되지 않음, 호출됐지만 잘못 적용됨">
  <defs>
    <marker id="ax-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 실패한 궤적을 보면 고칠 곳이 갈린다</text>
  <rect x="8" y="70" width="150" height="56" rx="8" class="sk-box-accent"/>
  <text x="83" y="95" text-anchor="middle" class="sk-label">실패한 시도</text>
  <text x="83" y="113" text-anchor="middle" class="sk-sub">궤적 기록</text>
  <path d="M160,98 L220,50" class="sk-line" marker-end="url(#ax-m1)"/>
  <path d="M160,98 L220,110" class="sk-line" marker-end="url(#ax-m1)"/>
  <path d="M160,98 L220,170" class="sk-line" marker-end="url(#ax-m1)"/>
  <rect x="224" y="26" width="200" height="48" rx="8" class="sk-box"/>
  <text x="324" y="48" text-anchor="middle" class="sk-label">로드되지 않음</text>
  <text x="324" y="65" text-anchor="middle" class="sk-sub">설치·탐색 경로 문제</text>
  <rect x="224" y="86" width="200" height="48" rx="8" class="sk-box"/>
  <text x="324" y="108" text-anchor="middle" class="sk-label">로드됐지만 호출 안 됨</text>
  <text x="324" y="125" text-anchor="middle" class="sk-sub">설명·트리거 문구 문제</text>
  <rect x="224" y="146" width="200" height="48" rx="8" class="sk-box"/>
  <text x="324" y="168" text-anchor="middle" class="sk-label">호출됐지만 잘못 적용</text>
  <text x="324" y="185" text-anchor="middle" class="sk-sub">내용·예제 문제</text>
</svg>
<figcaption>같은 "실패"여도 세 경우의 수정 위치가 다르므로, 점수만 보면 엉뚱한 곳을 고치게 된다.</figcaption>
</figure>

위 그림의 세 갈래는 원문이 구분한 세 경우입니다. 각 칸 아래의 "문제" 설명은 제가 이해를 돕기 위해 붙인 해석이고, 원문은 "경우마다 고치는 곳이 다르다"까지만 밝힙니다. 반복되는 실패 패턴 9가지가 있다고 하지만 목록은 PDF에 있습니다.

## 원문이 경고하는 함정

- 컴파일도 안 되는 코드가 만점을 받는 평가
- "Platform X를 사용했는가" 검사가 실제 사용 여부와 상관없이 통과하는 평가
- 판정 기준을 모델에게 대신 쓰게 하는 것

앞의 둘은 평가가 "그럴듯함"만 재고 "실제로 동작함"을 재지 않을 때 생깁니다. 그래서 1단계에 실행 증명 관문이 들어 있는 것으로 읽힙니다.

## 수치는 이렇게 읽는다

원문에 따르면 이 작업은 2025년 가을부터 Azure, Cosmos DB, SPFx, M365 Copilot 확장에 적용됐고, Azure Cosmos DB Agent Kit에서만 개선 46건이 닫혔습니다. 함께 공개된 AX Practitioner 스킬은 330문항에서 플레이북 기준 평균 95%라고 하는데, 이는 저자가 보고한 값이고 채점 방식은 글에 없습니다. 참고 수치일 뿐 성능 보증으로 읽으면 안 됩니다.

## 실무에서 달라지는 점

- **자사 SDK·API·확장을 배포하는 팀**: "에이전트가 우리 제품을 올바르게 쓰는가"를 릴리스 점검 항목에 넣을지 정해야 합니다. 시나리오 5~10개와 판정 기준을 만들고, 버전이 바뀔 때마다 다시 돌리는 구조가 필요합니다. 이 판정 기준 작성이 가장 어렵다고 원문도 인정하고 지름길은 제시하지 않습니다.
- **사내 인프라·M365·Azure 운영자**: 사내 코딩 에이전트에 연결한 문서, 지침 파일, MCP 도구, 스킬의 "소유자"를 정해 두세요. 이 방법은 근거를 들고 담당 팀에 수정을 요청하는 것으로 끝나는데, 담당이 없으면 고칠 곳이 없습니다.
- **평가를 도입할 때 먼저 확인할 것**: 결과 점수에 "빌드·실행 성공" 관문이 들어 있는지 봅니다. 없으면 컴파일 안 되는 코드가 통과합니다. 점수와 별개로 실패 궤적을 남기도록 로그 보관 설정도 확인하세요.
- **IT 강사**: 수강생에게 "에이전트가 틀리면 프롬프트를 더 세게 쓰라"고 가르치기 전에, 원인을 로드 안 됨/호출 안 됨/잘못 적용의 세 갈래로 나눠 보게 하는 편이 낫습니다. 그래야 문서를 고칠지, 스킬 설명을 고칠지가 정해집니다.
- **비용·라이선스**: 원문에 가격이나 도구 비용 언급은 없습니다. 반복 평가는 에이전트 실행 횟수만큼 토큰을 쓰므로, 시나리오 수를 정할 때 예산을 함께 잡아야 합니다(제 추정이며 원문 내용이 아닙니다).

## 같이 읽으면 좋은 글

운영 중인 에이전트의 실패를 묶어 원인을 짚는 접근은 [AQuA 글](/posts/google-aqua-production-agent-diagnosis/)에서 다뤘습니다. AQuA가 운영 단계의 진단이라면 이 플레이북은 에이전트가 읽는 자료를 배포 전후에 평가하는 쪽입니다. 도구와 API를 에이전트가 어떻게 고르는지의 기본 개념은 [MCP와 API는 무엇이 다른가](/posts/mcp-vs-api/)에 있습니다.

## 참고

- [Introducing the Agent Experience (AX) Practitioner Playbook — Microsoft Developer Blog](https://developer.microsoft.com/blog/introducing-the-agent-experience-ax-practitioner-playbook/)
- 플레이북 PDF: aka.ms/ax-playbook
- AX Practitioner 스킬: aka.ms/ax-playbook/skill
- Agent Experience 시리즈: aka.ms/agent-experience
