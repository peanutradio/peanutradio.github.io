---
title: "SKILL.md 첫 스킬 만들기 — 위치·필드·막히는 지점 (GitHub Copilot 문서 기준)"
date: 2026-11-14 06:53:00 +0900
categories: [AI Agent]
tags: [ai-agent, copilot, github]
description: GitHub Copilot의 Agent Skills를 SKILL.md로 만드는 순서와 위치 선택 기준, 자주 막히는 지점을 공식 문서 기준으로 정리한 실무 가이드.
---

> 이 글은 GitHub Docs의 Agent skills 문서와 GitHub 블로그 "Build once, run on any agent"(2026-11-12)를 읽고 정리한 것이다. 직접 실행한 기록이 아니라 "문서 기준 순서"이며, 환경별 화면은 달라질 수 있다.
{: .prompt-info }

## 결론

- 스킬은 `SKILL.md`라는 이름의 파일 하나와 그 폴더다. 필수 front matter는 `name`과 `description` 두 개다.
- 프로젝트용은 `.github/skills/<이름>/`, 개인용은 `~/.copilot/skills/<이름>/`에 둔다. 클라우드 에이전트에서 개인 스킬이 자동으로 된다고 가정하면 안 된다.
- 스킬이 언제 쓰일지는 거의 `description` 문장이 결정한다. 이 한 줄을 가장 공들여 써야 한다.

배경 설명은 [Agent Skills가 AAIF로 간 소식](/posts/agent-skills-aaif-donation/)에서 다뤘으니 여기서는 만드는 순서에 집중한다.

## 단계별 순서

### 1. 둘 곳을 정한다

문서가 안내하는 프로젝트 스킬 위치는 `.github/skills`, `.claude/skills`, `.agents/skills`다. 개인 스킬은 `~/.copilot/skills` 또는 `~/.agents/skills`다. 각 스킬은 자기 하위 폴더를 가지며, 폴더 이름은 소문자와 하이픈을 쓴다.

### 2. 폴더와 파일을 만든다

예를 들어 `.github/skills/github-actions-failure-debugging/SKILL.md`처럼 만든다. 파일 이름은 정확히 `SKILL.md`여야 한다고 문서가 명시한다.

### 3. front matter를 쓴다

문서가 요구하는 필드는 다음과 같다.

- `name`(필수): 소문자와 하이픈으로 된 고유 식별자. 보통 폴더 이름과 맞춘다.
- `description`(필수): 스킬이 무엇을 하고 Copilot이 언제 써야 하는지.
- `license`(선택): 적용되는 라이선스 설명.
- `allowed-tools`(선택): 스크립트를 쓸 때 예로 `shell`이 나온다. 목록에 없는 도구는 권한을 묻는다.

문서 예시는 아래와 같다.

```markdown
---
name: github-actions-failure-debugging
description: Guide for debugging failing GitHub Actions workflows. Use this when asked to debug failing GitHub Actions workflows.
---
```

### 4. 본문에 절차를 적는다

front matter 아래 Markdown 본문에 지침을 쓴다. 스킬이 호출되면 폴더 안의 다른 파일(스크립트, 참고 자료)도 지침과 함께 쓸 수 있으므로, 긴 참고는 별도 파일로 두고 본문에서 가리키면 된다.

### 5. 호출과 목록을 확인한다

문서 기준으로 Copilot은 프롬프트와 description을 보고 스킬 사용 여부를 스스로 정하며, 선택되면 SKILL.md가 에이전트 컨텍스트에 주입된다. Copilot CLI에서는 `/스킬이름`으로 명시 호출하고, `/skills list`, `/skills info SKILL-NAME`, `/skills reload`로 상태를 본다. 터미널 명령으로는 `copilot skill list`, `copilot skill add`, `copilot skill enable`, `copilot skill disable`이 문서에 나온다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="스킬 폴더 구조와 로드 흐름: 폴더, SKILL.md, 에이전트 컨텍스트">
  <defs>
    <marker id="sk-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 스킬은 폴더이고, description이 입구다</text>
  <rect x="8" y="50" width="210" height="120" rx="8" class="sk-box"/>
  <text x="113" y="76" text-anchor="middle" class="sk-label">.github/skills/이름/</text>
  <text x="113" y="104" text-anchor="middle" class="sk-mono">SKILL.md</text>
  <text x="113" y="126" text-anchor="middle" class="sk-sub">스크립트·참고 파일(선택)</text>
  <rect x="270" y="50" width="180" height="120" rx="8" class="sk-box-accent"/>
  <text x="360" y="84" text-anchor="middle" class="sk-label">description 판단</text>
  <text x="360" y="108" text-anchor="middle" class="sk-sub">프롬프트와 맞으면 선택</text>
  <rect x="502" y="50" width="210" height="120" rx="8" class="sk-box"/>
  <text x="607" y="84" text-anchor="middle" class="sk-label">에이전트 컨텍스트</text>
  <text x="607" y="108" text-anchor="middle" class="sk-sub">SKILL.md 주입</text>
  <text x="607" y="128" text-anchor="middle" class="sk-sub">폴더 파일 접근 가능</text>
  <path d="M220,110 L266,110" class="sk-line" marker-end="url(#sk-m1)"/>
  <path d="M452,110 L498,110" class="sk-line-accent" marker-end="url(#sk-m1)"/>
</svg>
<figcaption>description이 모호하면 스킬은 만들어 놓고도 선택되지 않는다.</figcaption>
</figure>

## 위치를 고르는 판단표

| 구분 | 프로젝트 스킬 | 개인 스킬 |
|---|---|---|
| 위치 | `.github/skills`, `.claude/skills`, `.agents/skills` | `~/.copilot/skills`, `~/.agents/skills` |
| 공유 범위 | 저장소를 쓰는 팀 전체 | 내 여러 프로젝트 |
| 적합한 내용 | 저장소 전용 절차, 팀 규칙 | 개인 습관, 범용 도구 사용법 |
| 클라우드 에이전트 | 저장소에 있으므로 대상이 됨 | 문서상 자동 보장 아님(아래 참조) |

판단 기준은 단순하다. 팀이 같이 써야 하면 프로젝트, 혼자만 쓰면 개인이다. 그리고 두 번 이상 같은 설명을 에이전트에게 반복했다면 스킬로 올릴 후보다. 이 기준은 내 제안이다.

## 자주 막히는 지점

### 1. 스킬이 선택되지 않는다

원인은 대개 `description`이다. 문서는 "무엇을 하는지"와 "언제 쓸지"를 함께 쓰라고 안내한다. 예시처럼 "Use this when asked to …" 문형을 넣고, 목록에서 `/skills info`로 읽히는 내용을 확인한다. 그래도 안 보이면 `/skills reload`를 시도한다.

### 2. 개인 스킬이 클라우드 에이전트에서 안 보인다

문서는 지원 위치가 환경마다 다르니 개인 스킬이 클라우드에서 쓰일 것이라 가정하지 말라고 한다. 클라우드 샌드박스 세션에서는 dotfiles 저장소의 `.copilot/skills`에 넣는 방법이 소개되는데, GitHub Actions 기반 세션에는 적용되지 않는다고 한다. 팀 공용이면 저장소의 프로젝트 스킬로 두는 쪽이 확실하다.

### 3. IDE에서 기능이 없다

기능 매트릭스 문서는 해당 페이지가 공개 프리뷰이고 변경될 수 있다고 하며, 버전별 지원 표를 싣는다. 예를 들어 VS Code는 1.108.0 열에서 지원, 1.104.0~1.107.0은 미지원으로 표시되고, Visual Studio는 18.6.0에서 지원, JetBrains는 일부 확장 버전에서 프리뷰(P)로 표시된다. 안 보이면 IDE와 확장 버전부터 확인한다. 수치는 문서 작성 시점 표 기준이다.

### 4. 쉘 스크립트 권한 질문이 반복된다

`allowed-tools`에 나열되지 않은 도구는 권한을 묻는다. 그렇다고 `shell`을 미리 허용하는 건 신중해야 한다. 문서는 스킬과 스크립트를 검토한 뒤에만 쉘 계열을 미리 승인하라고 경고한다.

### 5. 가져온 스킬을 그대로 믿는다

문서는 스킬이 GitHub에 의해 검증되지 않고 프롬프트 인젝션이나 숨은 지시, 악성 스크립트를 담을 수 있다고 경고한다. 외부 스킬은 설치 전에 SKILL.md와 스크립트를 읽어야 한다. 문서에는 공개 프리뷰 상태의 `gh skill`(GitHub CLI 2.90.0 이상)로 `gh skill preview`, `gh skill install`을 쓸 수 있고 `--pin`으로 버전을 고정할 수 있다고 나온다.

### 6. 패키징 스크립트가 SKILL.md를 만든다고 오해한다

블로그는 `skills/*/SKILL.md`를 원본으로 두고, 패키징 스크립트가 Claude Code용 `.claude-plugin/plugin.json`, Cursor용 `.cursor-plugin/plugin.json`, Copilot용 `.github/agents/*.agent.md`를 만든다고 설명한다. 보여 준 명령은 CI용 검사 모드인 `node scripts/package.mjs --check` 하나뿐이다. 이 스크립트는 해당 데모 저장소의 것이지 표준 도구가 아니므로, 내 저장소에 같은 방식을 쓰려면 직접 만들어야 한다.

## 도입 전 점검표

- 폴더 이름과 `name`이 같고 소문자·하이픈인가
- `description`에 "무엇을"과 "언제"가 둘 다 있는가
- 스크립트를 쓴다면 `allowed-tools` 범위를 최소로 했는가
- 쓰려는 환경(CLI, VS Code, 클라우드 에이전트)의 지원 버전을 확인했는가
- 외부에서 가져온 스킬은 본문과 스크립트를 읽었는가

## 참고

- [Adding agent skills for GitHub Copilot (GitHub Docs)](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)
- [Agent skills (GitHub Docs)](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [Adding agent skills for GitHub Copilot CLI (GitHub Docs)](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
- [Copilot feature matrix (GitHub Docs)](https://docs.github.com/en/copilot/reference/copilot-feature-matrix)
- [Build once, run on any agent: a practical guide to Agent Skills (GitHub Blog)](https://github.blog/ai-and-ml/github-copilot/build-once-run-on-any-agent-a-practical-guide-to-agent-skills/)
