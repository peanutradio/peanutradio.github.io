---
title: "Azure DevOps Autofix가 에이전트로 바뀐다 — 알림에서 수정 PR까지"
date: 2026-10-22 06:51:00 +0900
categories: [IT News]
tags: [azure-devops, copilot, autofix, security, ai-agent]
description: Azure DevOps의 Copilot Autofix가 코딩 에이전트 기반으로 바뀌어 의존성 취약점과 서드파티 스캔 알림까지 수정 PR을 만든다. 제한 프리뷰 단계의 변화를 정리했다.
---

> 이 글은 Azure DevOps 블로그의 「Introducing agentic Autofix for code and dependencies」(2026-10-21)를 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록이 아니라 발표 내용을 읽은 정리입니다.
{: .prompt-info }

**결론부터.** Azure DevOps의 GitHub Advanced Security Copilot Autofix가 "CodeQL 규칙 일부에 대한 제안"에서 "Copilot 코딩 에이전트가 저장소를 탐색해 만드는 수정"으로 바뀝니다. 대상도 CodeQL 알림에서 의존성 취약점 알림과 서드파티 코드 스캔 알림까지 넓어졌습니다. 다만 지금은 제한된 퍼블릭 프리뷰에 등록된 조직만 쓸 수 있고, 퍼블릭 프리뷰는 11월 예정입니다. GA 일정은 글에 없습니다.

## 무엇이 달라졌나

- **방식**: 알림을 고르면 Autofix가 백그라운드에서 Copilot 코딩 에이전트로 저장소 맥락을 살핍니다. 제안을 스스로 검토·다듬은 뒤 돌려주고, 병합 결정은 사람이 합니다.
- **범위**: 이전에는 CodeQL 규칙 일부만 다뤘습니다. 이제는 의존성 알림에서 "Generate fix"를 눌러 설명이 붙은 PR을 열 수 있고, 다른 도구가 올린 코드 스캔 알림도 대상입니다.
- **지침·스킬**: `.github` 또는 `.agents`의 지원 위치에 둔 사용자 지침과 스킬을 읽습니다. 글의 예시는 "승인된 입력 검증 헬퍼를 쓰라"는 지침, 그리고 패키지 업데이트 절차와 호환성 점검을 적은 의존성 업데이트 스킬입니다.
- **분류 도구**: 의존성 알림에 EPSS 백분위와 개발 전용 의존성 표시(Development 배지, Scope 필터)가 붙습니다. 글은 "런타임 범위 + 높은 EPSS 구간"으로 거른 뒤 심각도와 노출 정도를 보라고 제안합니다. EPSS는 공격 확률이 아니라 상대 순위라고 분명히 적었습니다.
- **켜는 법**: 조직·프로젝트·저장소 수준에서 버튼 하나로 켜고, 그다음 알림별로 Autofix를 요청합니다. 켠다고 수정이 자동 생성되지는 않습니다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="알림 선택부터 사람의 병합 결정까지 Autofix의 흐름">
  <defs>
    <marker id="af-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="af-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① Autofix 요청이 PR이 되기까지</text>
  <rect x="8" y="50" width="120" height="60" rx="8" class="sk-box"/>
  <text x="68" y="76" text-anchor="middle" class="sk-label">알림 선택</text>
  <text x="68" y="94" text-anchor="middle" class="sk-sub">코드·의존성</text>
  <rect x="168" y="50" width="150" height="60" rx="8" class="sk-box-accent"/>
  <text x="243" y="76" text-anchor="middle" class="sk-label">코딩 에이전트</text>
  <text x="243" y="94" text-anchor="middle" class="sk-sub">저장소 탐색·자체 검토</text>
  <rect x="358" y="50" width="130" height="60" rx="8" class="sk-box"/>
  <text x="423" y="76" text-anchor="middle" class="sk-label">수정 PR</text>
  <text x="423" y="94" text-anchor="middle" class="sk-sub">설명 포함</text>
  <rect x="528" y="50" width="180" height="60" rx="8" class="sk-box"/>
  <text x="618" y="76" text-anchor="middle" class="sk-label">사람의 검토</text>
  <text x="618" y="94" text-anchor="middle" class="sk-sub">diff·빌드·테스트 후 병합</text>
  <path d="M130,80 L166,80" class="sk-line" marker-end="url(#af-m1)"/>
  <path d="M320,80 L356,80" class="sk-line" marker-end="url(#af-m1)"/>
  <path d="M490,80 L526,80" class="sk-line-accent" marker-end="url(#af-a1)"/>
  <rect x="168" y="140" width="150" height="40" rx="8" class="sk-box-muted"/>
  <text x="243" y="164" text-anchor="middle" class="sk-mono">.github / .agents</text>
  <path d="M243,138 L243,112" class="sk-line" marker-end="url(#af-m1)"/>
</svg>
<figcaption>에이전트가 만드는 건 PR까지이고, 병합 책임은 여전히 사람에게 있다.</figcaption>
</figure>

## 글이 말하지 않은 것

글에는 지원 생태계 목록, 서드파티 도구 목록, 지침 파일의 구체 경로, AI 크레딧 금액, 수정 성공률 같은 정량 결과가 없습니다. 수정 생성은 AI 크레딧을 쓰고, 모든 요청이 수정을 만들지는 않는다고 글이 직접 밝힙니다. 이 글은 발표 내용만 정리한 것이어서 실제 품질은 확인하지 못했습니다.

## 실무에서 달라지는 점

1. **보안 담당자: 알림 큐의 정렬 기준을 정하세요.** EPSS는 "상대 순위"입니다. 런타임 범위와 높은 백분위를 함께 걸고, 개발 전용 의존성은 뒤로 미루는 기준을 문서로 먼저 정해 두면 Autofix를 어떤 알림에 쓸지 고르기 쉽습니다.
2. **저장소 관리자: 지침 파일이 곧 수정 품질입니다.** 에이전트가 팀 관례를 지침과 스킬로 읽는다고 하니, 입력 검증 헬퍼나 패키지 업데이트 절차를 지금부터 글로 남겨 두는 것이 준비입니다. 위치(`.github`, `.agents`)는 문서로 다시 확인하세요.
3. **조직 관리자: 켜기 전에 비용과 대상 범위를 확인하세요.** 조직·프로젝트·저장소 어느 단위로 켤지 정해야 하고, 크레딧 단가가 글에 없으니 프리뷰 등록 후 실제 소비량부터 보는 편이 안전합니다.
4. **개발 리드: 리뷰 규칙은 그대로입니다.** 병합 전 diff 검토와 빌드·테스트가 필요하다고 글이 명시합니다. "AI가 만든 보안 PR"을 예외로 두지 않는 규칙을 팀에 공지해야 합니다.
5. **강의·교육할 때: 자동 수정이 아니라 "수정 제안 자동화"로 설명하세요.** 수정이 안 나오는 경우도 있다는 점을 먼저 알려 주면 기대치 문제가 줄어듭니다.

같은 제품군의 이전 소식은 [Azure DevOps 플러그인이 Microsoft Copilot에 들어왔다](/posts/azure-devops-in-microsoft-copilot/)에서 정리했습니다. 거기서는 Copilot에서 Azure DevOps를 다루는 입구였고, 이번 글은 보안 알림 쪽에서 같은 에이전트가 일하는 방식입니다.

## 참고

- [Introducing agentic Autofix for code and dependencies — Azure DevOps Blog](https://devblogs.microsoft.com/devops/introducing-agentic-autofix-for-code-and-dependencies/)
