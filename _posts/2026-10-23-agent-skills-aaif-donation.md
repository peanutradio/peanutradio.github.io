---
title: "Agent Skills, Linux Foundation 산하 AAIF로 — SKILL.md가 한 회사 것이 아니게 됐다"
date: 2026-10-23 06:49:00 +0900
categories: [IT News]
tags: [agent-skills, skill-md, aaif, linux-foundation, ai-agent]
description: Anthropic이 Agent Skills 표준을 Agentic AI Foundation에 기증했다. SKILL.md 폴더 형식이 도구를 넘나드는 공용 포맷이 된다는 뜻과 운영자가 확인할 점을 정리했다.
---

> 이 글은 Anthropic의 "Donating Agent Skills to the Agentic AI Foundation"(2026-10-22) 발표를 읽고 정리·재구성한 것입니다. 수치는 모두 Anthropic이 밝힌 값이며 제가 따로 검증하지 않았습니다.
{: .prompt-info }

## 결론부터

스킬 표준의 소유권이 Anthropic에서 Linux Foundation 산하 Agentic AI Foundation(AAIF)으로 넘어갔습니다. 스킬 작성자 입장에서 **당장 바뀌는 코드는 없습니다.** 바뀌는 것은 "이 형식에 투자해도 되는가"라는 판단 근거입니다. 특정 회사 제품의 포맷이 아니라 중립 재단이 관리하는 포맷이 됐기 때문에, 팀 내부 지침을 `SKILL.md`로 쌓아 두는 선택이 조금 더 안전해졌습니다. 다만 중립성이 실제로 어떻게 작동할지는 아직 글로 확인되는 것이 없습니다.

## 무엇이 넘어갔나

원문에 따르면 기증 대상은 네 가지입니다.

- Agent Skills 표준 자체
- Agent Skills라는 이름과 로고
- GitHub 조직
- agentskills.io 도메인

스킬은 `SKILL.md` 파일이 든 폴더이고, 필요하면 스크립트나 참고 자료를 같이 넣습니다. 2025년 10월에 공개됐고 12월에 오픈 표준으로 발행됐다고 합니다. AAIF는 Anthropic이 Block, OpenAI와 함께 설립했고 MCP도 이미 여기에 기증돼 있습니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="SKILL.md 폴더 하나를 여러 에이전트 도구가 읽고, 표준은 AAIF가 관리하는 구조">
  <defs>
    <marker id="as-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 같은 폴더, 여러 도구 — 표준은 재단이 관리</text>
  <rect x="8" y="50" width="200" height="70" rx="8" class="sk-box-accent"/>
  <text x="108" y="80" text-anchor="middle" class="sk-label">my-skill/</text>
  <text x="108" y="100" text-anchor="middle" class="sk-mono">SKILL.md + scripts</text>
  <rect x="300" y="36" width="170" height="40" rx="8" class="sk-box"/>
  <text x="385" y="61" text-anchor="middle" class="sk-label">에이전트 도구 A</text>
  <rect x="300" y="94" width="170" height="40" rx="8" class="sk-box"/>
  <text x="385" y="119" text-anchor="middle" class="sk-label">에이전트 도구 B</text>
  <rect x="300" y="152" width="170" height="40" rx="8" class="sk-box"/>
  <text x="385" y="177" text-anchor="middle" class="sk-label">에이전트 도구 C</text>
  <path d="M210,85 L296,56" class="sk-line" marker-end="url(#as-m1)"/>
  <path d="M210,85 L296,114" class="sk-line" marker-end="url(#as-m1)"/>
  <path d="M210,90 L296,170" class="sk-line" marker-end="url(#as-m1)"/>
  <rect x="540" y="70" width="170" height="70" rx="8" class="sk-box-muted"/>
  <text x="625" y="100" text-anchor="middle" class="sk-label">AAIF</text>
  <text x="625" y="120" text-anchor="middle" class="sk-sub">Linux Foundation 산하</text>
  <path d="M538,105 L474,105" class="sk-line" marker-end="url(#as-m1)"/>
</svg>
<figcaption>스킬은 폴더 하나로 이동하고, 그 폴더의 규칙은 재단이 관리한다.</figcaption>
</figure>

## 원문이 말하는 것과 말하지 않는 것

유지보수는 Anthropic, Google, OpenAI 등의 엔지니어가 회사 대표가 아닌 개인 자격으로 맡고, 리드 메인테이너는 David Soria Parra와 Mahesh Murag입니다. 제안된 로드맵은 버전 릴리스, 공식 변경 제안 절차, 스킬 작성·설치자를 위한 보안 가이드 세 가지입니다.

수치로는 70개 넘는 에이전트 플랫폼이 통합했고(ChatGPT, Cursor, GitHub Copilot, VS Code 포함), 게시된 스킬이 약 100만 개, 설치가 약 2억 8천만 건이라고 합니다. 이 숫자는 외부 검증 없는 발표자 주장입니다.

반대로 글에 **없는** 것도 분명합니다. 로드맵에는 일정이 없습니다. 메인테이너를 어떻게 뽑는지, 의견이 갈리면 누가 결정하는지도 적혀 있지 않습니다. 기술 사양은 폴더와 `SKILL.md` 구조 이상으로 나오지 않아서, 세부 필드는 agentskills.io 원문을 따로 봐야 합니다.

## 실무에서 달라지는 점

- **사내 지침 문서를 쌓는 팀(운영자):** Copilot이든 다른 에이전트든 "에이전트용 작업 절차"를 도구별 전용 파일로 흩어 두고 있다면, 폴더 단위 `SKILL.md`로 모으는 방안을 검토할 시점입니다. 단, 도구마다 스킬을 찾는 위치와 지원 범위는 다를 수 있으니 실제 인식 여부는 도구별로 확인해야 합니다.
- **보안 담당:** 스킬에는 스크립트가 들어갈 수 있습니다. 100만 개가 공개돼 있다는 말은 곧 출처 불명 스크립트를 내려받을 길도 넓다는 뜻입니다. 보안 가이드가 "제안된 로드맵"인 지금은 사내 승인된 저장소의 스킬만 설치하도록 정책부터 잡는 편이 안전합니다. 저는 이것을 제 해석으로 덧붙입니다.
- **도구 선정 담당:** 도입 기준에 "SKILL.md 지원 여부"를 한 줄 넣어 두면 이동 비용을 가늠하기 쉽습니다. 다만 70개 이상이라는 통합 수는 자체 주장이므로, 실제로 쓸 도구는 직접 확인하세요.
- **IT 강사:** 스킬을 "프롬프트 모음"이 아니라 "폴더 단위 업무 매뉴얼 + 스크립트"로 설명하면 학습자가 구조를 잡기 쉽습니다. 표준이 바뀔 수 있다는 점(버전 릴리스 예정)도 함께 언급해 두면 좋습니다.
- **MS 생태계 운영자:** GitHub Copilot과 VS Code가 통합 목록에 있다는 점이 핵심입니다. 같은 폴더를 Copilot과 다른 에이전트가 함께 읽는 구성이 가능한지는, 이 글만으로는 단정할 수 없습니다. 파일럿 환경에서 먼저 확인하시길 권합니다. 저는 직접 실행해 보지 않았습니다.

관련해서 이 블로그에서 다룬 에이전트 배포·거버넌스 글도 함께 읽어 보세요. [wiqd로 Copilot 플러그인 배포하기](/posts/wiqd-copilot-plugin-packaging/)는 배포 단위를 어떻게 다룰지, [Agent 365와 Agent ID 정리](/posts/agent-365-agent-id/)는 에이전트 신원을 어떻게 다룰지에 대한 글입니다.

## 참고

- Anthropic, Donating Agent Skills to the Agentic AI Foundation (2026-10-22): https://www.anthropic.com/news/donating-agent-skills-to-the-agentic-ai-foundation
- 사양 문서: https://agentskills.io
- GitHub: https://github.com/agentskills/agentskills
