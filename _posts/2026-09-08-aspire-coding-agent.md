---
title: "Aspire가 코딩 에이전트에게 앱 전체를 넘긴다 — aspire agent init"
date: 2026-09-08 19:30:00 +0900
categories: [IT News]
tags: [aspire, copilot, coding-agent, opentelemetry, claude-code, dotnet]
description: 명령 한 줄로 컨테이너·프로세스·디버거를 다 띄우고, 그 통제권을 코딩 에이전트에게 넘긴다. aspire agent init이 실제로 하는 일.
---

Microsoft Developer 채널에 짧은 영상이 하나 올라왔습니다.
Aspire PM인 Maddy Montaquila가 **Aspire로 코딩 에이전트를 강화하는 방법**을 70초 만에 설명합니다.

핵심 문장은 이겁니다.

> "Aspire화된 앱은 코딩 에이전트에게 앱 전체에 대한 **훨씬 많은 맥락과 통제권**을 준다."

짧지만 짚고 갈 게 여러 개라 정리했습니다.

## Aspire가 뭐였더라

이름이 `.NET Aspire`에서 그냥 **Aspire**로 바뀌면서 성격도 달라졌습니다.
지금은 **폴리글랏**입니다.

- C#, TypeScript, JavaScript, Python, Java, Go, Rust
- Docker 컨테이너, React/Vite 프론트, 백엔드, 데이터베이스

스스로를 **"agent-ready, code-first tool for composing, debugging, and deploying distributed applications"**
이라고 소개합니다. 중심에 **AppHost**가 있고, 여기가 서비스·의존성·리소스가 어떻게 연결되는지에 대한
**단일 진실 공급원**입니다.

| 기능 | 내용 |
|---|---|
| `aspire run` | 서비스·DB·컨테이너를 **한 번에** 기동, 서비스 디스커버리와 엔드포인트 연결까지 |
| 대시보드 | 리소스 토폴로지 실시간 확인, OpenTelemetry 로그·트레이스·메트릭 통합 |
| 배포 | Docker, Kubernetes, Azure Container Apps, AWS용 산출물 생성 |

현재 최신은 **Aspire 13.5**입니다.

## 에이전트 입장에서 뭐가 달라지나

분산 앱을 코딩 에이전트에게 맡겨본 적 있으면 아실 겁니다.
프론트 따로, 백엔드 따로, DB 컨테이너 따로 띄우고,
로그는 터미널 세 개에 흩어져 있고, 프론트가 백엔드랑 제대로 붙었는지 확인하려면
백그라운드 태스크를 여러 개 돌려야 합니다.

<figure class="sketch">
<svg viewBox="0 0 720 320" role="img" aria-label="코딩 에이전트가 분산 앱을 다룰 때 — Aspire 이전에는 프로세스와 로그가 흩어져 있고, 이후에는 AppHost 한 곳으로 모인다">
  <defs>
    <marker id="as-m" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="as-a" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>

  <text x="0" y="14" class="sk-title">코딩 에이전트가 분산 앱을 다루는 두 가지 방식</text>

  <text x="172" y="42" text-anchor="middle" class="sk-title">이전</text>
  <text x="548" y="42" text-anchor="middle" class="sk-title">Aspire</text>

  <rect x="112" y="58" width="120" height="46" rx="8" class="sk-box"/>
  <text x="172" y="86" text-anchor="middle" class="sk-label">코딩 에이전트</text>

  <rect x="16" y="150" width="96" height="40" rx="6" class="sk-box-muted"/>
  <text x="64" y="175" text-anchor="middle" class="sk-sub">프론트엔드</text>
  <rect x="124" y="150" width="96" height="40" rx="6" class="sk-box-muted"/>
  <text x="172" y="175" text-anchor="middle" class="sk-sub">백엔드</text>
  <rect x="232" y="150" width="96" height="40" rx="6" class="sk-box-muted"/>
  <text x="280" y="175" text-anchor="middle" class="sk-sub">DB 컨테이너</text>

  <path d="M150,106 L74,146"  class="sk-line" marker-end="url(#as-m)"/>
  <path d="M172,106 L172,146" class="sk-line" marker-end="url(#as-m)"/>
  <path d="M196,106 L270,146" class="sk-line" marker-end="url(#as-m)"/>

  <rect x="16" y="214" width="96" height="36" rx="6" class="sk-box-muted"/>
  <text x="64" y="237" text-anchor="middle" class="sk-sub">로그 1</text>
  <rect x="124" y="214" width="96" height="36" rx="6" class="sk-box-muted"/>
  <text x="172" y="237" text-anchor="middle" class="sk-sub">로그 2</text>
  <rect x="232" y="214" width="96" height="36" rx="6" class="sk-box-muted"/>
  <text x="280" y="237" text-anchor="middle" class="sk-sub">로그 3</text>

  <text x="172" y="284" text-anchor="middle" class="sk-sub">따로 띄우고 · 따로 보고</text>
  <text x="172" y="302" text-anchor="middle" class="sk-sub">붙었는지 확인하려 태스크를 돌린다</text>

  <path d="M352,170 L368,170" class="sk-line" stroke-dasharray="3 4"/>

  <rect x="488" y="58" width="120" height="46" rx="8" class="sk-box"/>
  <text x="548" y="86" text-anchor="middle" class="sk-label">코딩 에이전트</text>

  <path d="M548,106 L548,140" class="sk-line-accent" marker-end="url(#as-a)"/>

  <rect x="392" y="144" width="312" height="60" rx="8" class="sk-box-accent"/>
  <text x="548" y="168" text-anchor="middle" class="sk-label">AppHost — 단일 진실 공급원</text>
  <text x="548" y="188" text-anchor="middle" class="sk-mono">aspire run · CLI · skills</text>

  <path d="M470,206 L470,224" class="sk-line-accent" marker-end="url(#as-a)"/>
  <path d="M626,206 L626,224" class="sk-line-accent" marker-end="url(#as-a)"/>

  <rect x="392" y="228" width="150" height="40" rx="6" class="sk-box"/>
  <text x="467" y="253" text-anchor="middle" class="sk-sub">모든 리소스 기동·정지</text>
  <rect x="554" y="228" width="150" height="40" rx="6" class="sk-box"/>
  <text x="629" y="253" text-anchor="middle" class="sk-sub">로그·트레이스 한 곳</text>

  <text x="548" y="292" text-anchor="middle" class="sk-sub">명령 하나로 전부 · 대시보드에 보이는 건 에이전트도 본다</text>
</svg>
<figcaption>흩어진 것을 에이전트가 각자 챙기게 하는 대신, <strong>AppHost 하나를 보게 만드는</strong> 구조입니다.</figcaption>
</figure>

영상에서 말한 달라지는 점은 이렇습니다.

- **명령 하나로** 컨테이너·프로세스·코드·디버거를 전부 기동
- 에이전트가 **개별 리소스를 껐다 켤 수 있음**
- **모든 로그를 한 곳에서** 봄 → 프론트가 백엔드랑 잘 붙었는지 보려고 백그라운드 태스크를 여러 개 돌릴 필요가 없음
- **Aspire 대시보드에서 보이는 것은 에이전트에게도 노출됨** (CLI + 함께 배포되는 스킬 세트를 통해)

## `aspire agent init`이 실제로 하는 일

영상에서는 "명령 한 줄이면 스킬이 여러 개 깔린다"고만 넘어가는데,
문서를 보니 좀 더 구체적입니다.

이 명령은 **감지된 에이전트 환경에 맞춰 AI 에이전트 설정을 초기화**합니다.

스킬 번들은 **6개**입니다.

| 스킬 | 역할 |
|---|---|
| `aspire` | 기본 |
| `aspire-init` | 초기 설정 |
| `aspire-orchestration` | 리소스 기동·정지, 커스텀 리소스 명령 (DB 시드·초기화 등) |
| `aspire-monitoring` | 앱 모니터링, OpenTelemetry로 프로파일링 |
| `aspire-deployment` | 배포 API로 테스트·운영 환경 배포 |
| `aspireify` | 리소스 탐색, 의존성 연결, OpenTelemetry 설정, 검증 |

`aspire agent init`을 단독으로 실행하면 **기존 워크스페이스에 추가해도 안전한 것들만** 골라 넣습니다.
앞의 다섯 개고, `aspireify`는 빠집니다. 남의 프로젝트 구조를 건드리는 스킬이라 그런 것으로 보입니다.

그리고 설치 대상이 눈에 띕니다. **감지된 각 에이전트 클라이언트의 사용자 레벨 설정**에
스킬과 **PostToolUse 훅**을 넣습니다. 문서가 예로 든 게
**GitHub Copilot CLI와 Claude Code**입니다.

> Microsoft 도구가 Copilot만이 아니라 **Claude Code까지 대상으로 명시**하고 있다는 점.
> 에이전트 도구 계층이 특정 벤더에 묶이지 않는 방향으로 가고 있다는 신호로 읽힙니다.
{: .prompt-info }

## 스킬은 데이터를 노출하지 않는다

여기가 제일 짚고 싶은 부분입니다.

Aspire 스킬은 **마크다운 지시문 묶음**입니다.
각 스킬이 폴더 하나에 `SKILL.md`로 들어가고, 그 안에
**언제 이 스킬이 적용되는지, 에이전트가 어떤 순서로 일해야 하는지**가 적혀 있습니다.

문서가 명확히 선을 긋습니다.

> 스킬은 서비스를 실행하지도, 애플리케이션 데이터를 노출하지도 않는다.
> **Aspire 도구를 올바르게 쓰는 법을 에이전트에게 가르칠 뿐이다.**

이게 [MCP와 결이 다른 지점](/posts/mcp-vs-api/)입니다.

- **MCP 서버** — 능력을 *노출*한다. 도구 목록과 스키마를 주고, 모델이 골라 호출한다
- **스킬** — 사용법을 *가르친다*. 실행도 데이터 제공도 안 한다. "이럴 땐 이렇게 해라"는 문서다

실제로 일이 되게 하는 건 여전히 **Aspire CLI와 대시보드**입니다.
스킬은 그 위에 얹힌 **설명서**고요. 둘은 대체재가 아니라 다른 층입니다.

## 짚고 갈 것

**좋아 보이는 점**
분산 앱에서 에이전트가 겪는 문제는 대부분 **맥락이 흩어져 있어서** 생깁니다.
로그가 세 군데면 에이전트는 세 번 확인해야 하고, 그만큼 토큰과 왕복이 늘어납니다.
AppHost 하나로 모으는 접근은 이 비용을 직접 줄입니다.

**확인이 필요한 점**
- 이 글은 **70초 영상과 공식 문서를 근거로 정리한 것**이고, 저는 아직 직접 돌려보지 않았습니다
- 기존 프로젝트를 Aspire화하는 비용이 얼마나 되는지는 별개 문제입니다.
  `aspireify`가 기본에서 빠져 있다는 건 그만큼 침습적이라는 뜻으로 보입니다
- PostToolUse 훅을 **사용자 레벨 설정에 넣는다**는 점은 확인이 필요합니다.
  프로젝트 단위가 아니라 클라이언트 전역에 영향을 준다는 뜻이니까요

직접 붙여보고 실습 기록으로 따로 남기겠습니다.

---

## 참고

- [How does Aspire make Copilot more efficient when I'm building an app?](https://www.youtube.com/watch?v=K_TD4d1VvwE) — Microsoft Developer (Maddy Montaquila), 2026-09-07, 70초
- [aspire agent init command](https://aspire.dev/reference/cli/commands/aspire-agent-init/) — Aspire 공식 문서
- [Aspire skills for AI coding agents and AppHost projects](https://aspire.dev/get-started/aspire-skills/)
- [Use AI coding agents with Aspire AppHost projects today](https://aspire.dev/get-started/ai-coding-agents/)
- [Agentic development aspirations: build, run, observe — without more Markdown](https://devblogs.microsoft.com/aspire/agentic-dev-aspirations/) — Aspire Blog
- [aspire.dev](https://aspire.dev)

이 글은 위 영상과 문서의 내용을 정리하고 재구성한 것입니다. 다이어그램은 직접 그렸습니다.
