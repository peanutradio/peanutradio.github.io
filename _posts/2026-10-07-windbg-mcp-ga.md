---
title: "WinDbg MCP 정식 출시 — 자연어 디버깅, 실행한 명령은 눈에 보인다"
date: 2026-10-07 07:15:00 +0900
categories: [AI Agent]
tags: [mcp, windbg, debugging, microsoft, ai-agent]
description: WinDbg 1.2610.1001.0부터 GA된 MCP 서버. AI가 호출한 디버거 동작을 WinDbg에서 그대로 확인할 수 있게 설계한 점이 핵심이다.
---

> 이 글은 Microsoft Performance & Diagnostics 블로그의 WinDbg MCP 발표(2026-10-06)를 읽고 정리·재구성한 것입니다. 직접 설치해 써 본 기록이 아니라 발표 내용의 정리입니다.
{: .prompt-info }

## 결론부터

WinDbg MCP는 AI 클라이언트가 **지금 열려 있는 WinDbg 세션**에 붙어 자연어로 디버깅을 돕게 하는 공식 MCP 서버다. WinDbg 1.2610.1001.0부터 정식(GA) 제공이고, Microsoft Store와 WinDbg 다운로드 페이지에서 받는다. 내가 흥미롭게 본 건 기능 자체보다 설계다. AI가 한 일을 사람이 같은 도구 화면에서 곧바로 검증할 수 있게 했다. MCP 서버를 "도구를 열어주는 쪽"에서 어떻게 설계해야 하는지 보여 주는 사례다.

## 구조: 로컬 프록시가 사이를 잇는다

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="AI 클라이언트, 로컬 프록시, WinDbg 세션이 이어진 구조">
  <defs>
    <marker id="wd-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① WinDbg MCP 연결 구조</text>
  <rect x="8" y="60" width="180" height="60" rx="8" class="sk-box"/>
  <text x="98" y="86" text-anchor="middle" class="sk-label">AI 클라이언트</text>
  <text x="98" y="104" text-anchor="middle" class="sk-sub">Copilot (VS Code / CLI)</text>
  <rect x="270" y="60" width="180" height="60" rx="8" class="sk-box-accent"/>
  <text x="360" y="86" text-anchor="middle" class="sk-label">로컬 프록시</text>
  <text x="360" y="104" text-anchor="middle" class="sk-sub">MCP 서버</text>
  <rect x="532" y="60" width="180" height="60" rx="8" class="sk-box"/>
  <text x="622" y="86" text-anchor="middle" class="sk-label">WinDbg 세션</text>
  <text x="622" y="104" text-anchor="middle" class="sk-sub">동작이 화면에 그대로 표시</text>
  <path d="M190,90 L268,90" class="sk-line" marker-end="url(#wd-m1)"/>
  <path d="M452,90 L530,90" class="sk-line" marker-end="url(#wd-m1)"/>
  <text x="360" y="160" text-anchor="middle" class="sk-sub">디버거 연결은 로컬에 머문다 · 세션당 MCP 클라이언트 하나</text>
</svg>
<figcaption>AI가 건드리는 모든 것이 사람이 보는 WinDbg 세션 안에서 일어난다.</figcaption>
</figure>

대상은 사용자 모드와 커널 모드의 크래시·행(hang)이고, 라이브 대상, 크래시 덤프, Time Travel Debugging(TTD) 트레이스를 모두 다룬다. AI 클라이언트는 현재 대상을 살펴보고, 디버거 명령을 실행하고, 진단 정보와 소스 코드를 검토하고, 디버거 데이터를 시각화하고, 반복 분석용 스크립트를 만들 수 있다고 한다.

발표에 따르면 디버거 연결은 로컬에 머물고, AI 클라이언트는 자기가 설정된 모델 서비스와 통신한다. 한 WinDbg 세션에는 활성 MCP 클라이언트가 하나만 붙을 수 있다.

## 지원 범위와 설정 순서

- 공식 지원 클라이언트: VS Code의 GitHub Copilot, GitHub Copilot CLI
- 그 밖의 MCP 클라이언트(글은 Codex와 Claude Code를 예로 든다): 별도 설정으로 best-effort. 이 클라이언트들은 현재 MCP sampling을 지원하지 않아서, 쓰려면 아래의 교차 프롬프트 인젝션 보호를 꺼야 한다. 글은 이를 권장하지 않는다

설정은 WinDbg의 MCP 서비스 설정에서 시작한다.

1. **Enable Server**를 선택하고 클라이언트(VS Code 또는 GitHub Copilot CLI)를 고른다
2. **Install MCP**로 클라이언트에 WinDbg를 등록한다
3. **MCP Service**에서 개인정보·보안 경고와 Secure Mode 옵션을 확인하고 시작한다
4. 디버깅 대상을 로드하거나 연결한다
5. **Open Chat**으로 조사를 시작한다

순서가 중요하다. Secure Mode를 완전하게 적용하려면 대상을 연결하기 **전에** MCP를 시작해야 한다. 이미 대상이 붙어 있으면 부분 Secure Mode로 들어가고 경고가 뜬다. Secure Mode는 WinDbg를 재시작할 때까지 유지된다.

함께 공개된 **Diagnostician 스킬**은 앱, 서비스, UMDF 드라이버, 커널 모드 드라이버를 대상으로 5단계 근본 원인 조사를 돌린다. 패턴이 맞아도 가설로 취급하고 대안을 검증한 뒤, 후보 원인과 검증 단계, 아직 비어 있는 증거를 함께 보고한다. 공개 카탈로그 `microsoft/win-dev-skills`의 windbg 플러그인으로 배포되고, Copilot CLI에서 `copilot plugin install windbg@win-dev-skills`로 설치한다. 파일 쓰기가 허용되면 작업 폴더의 `.diagnoses\` 아래에 Markdown 보고서를 남긴다.

## 보안 장치

디버거는 AI에게 넘기기엔 위험한 도구다. 그래서 글이 강조하는 보호 장치가 설계의 절반이다.

- **Secure Mode**: 기본 선택. 신뢰할 수 없는 코드 로드·실행, 프로세스 실행, 안전하지 않은 파일 작업 같은 고위험 동작을 제한한다
- **교차 프롬프트 인젝션 보호**: 디버거 출력(덤프 안의 문자열 등)이 AI를 속여 위험한 행동을 하게 만들지 않도록, 보호 대상 동작에서는 클라이언트의 모델 서비스로 출력을 분류하고 위험 판정이 난 내용을 막는다. MCP sampling을 지원하는 클라이언트가 필요하고 AI 토큰을 추가로 쓸 수 있다
- AI가 시작한 동작은 감사할 수 있게 기록된다

대상 프로세스의 메모리와 출력은 신뢰할 수 없는 입력이다. 크래시 덤프에 심어진 문자열이 에이전트를 조종할 수 있다는 점을 제품 수준에서 다룬 것이다. 이 보호를 끄면 방어가 약해지므로 Secure Mode를 쓰라는 게 글의 권고다.

## 내가 가져갈 점

<figure class="sketch">
<svg viewBox="0 0 720 170" role="img" aria-label="AI의 결론을 증거와 대조해 검증하는 흐름">
  <defs>
    <marker id="wd-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 결론이 아니라 증거를 확인한다</text>
  <rect x="8" y="50" width="200" height="60" rx="8" class="sk-box"/>
  <text x="108" y="76" text-anchor="middle" class="sk-label">AI의 결론</text>
  <text x="108" y="94" text-anchor="middle" class="sk-sub">불완전하거나 틀릴 수 있음</text>
  <rect x="260" y="50" width="200" height="60" rx="8" class="sk-box-accent"/>
  <text x="360" y="76" text-anchor="middle" class="sk-label">실행된 명령 확인</text>
  <text x="360" y="94" text-anchor="middle" class="sk-sub">WinDbg 화면에서 그대로</text>
  <rect x="512" y="50" width="200" height="60" rx="8" class="sk-box"/>
  <text x="612" y="76" text-anchor="middle" class="sk-label">대상 상태와 대조</text>
  <text x="612" y="94" text-anchor="middle" class="sk-sub">사람이 최종 판단</text>
  <path d="M210,80 L258,80" class="sk-line-accent" marker-end="url(#wd-m2)"/>
  <path d="M462,80 L510,80" class="sk-line-accent" marker-end="url(#wd-m2)"/>
</svg>
<figcaption>에이전트 도구는 답보다 검증 경로를 함께 내놓아야 한다.</figcaption>
</figure>

글도 AI 결과가 불완전하거나 틀릴 수 있으니 중요한 결론은 WinDbg에서 보이는 대상 상태와 증거로 확인하라고 적는다. 내가 만드는 MCP 서버에도 그대로 적용할 원칙이다. 도구 호출을 사람이 읽을 수 있게 남기고, 위험 동작은 기본으로 막고, 도구가 돌려주는 데이터는 믿지 않는다. 이 제품이 실제로 얼마나 잘 맞히는지는 글에 수치가 없어서 알 수 없다. 사용하려면 승인된 대상에서, 조직의 데이터 취급 정책을 지켜야 한다.

## 실무에서 달라지는 점

사내 개발 도구를 배포·관리하거나 Windows 트러블슈팅을 가르치는 입장에서 볼 항목이다.

1. **클라이언트 표준을 정한다.** 보호 장치가 온전히 동작하는 건 VS Code의 GitHub Copilot과 Copilot CLI뿐이다. 다른 MCP 클라이언트를 쓰면 교차 프롬프트 인젝션 보호를 꺼야 하므로, 사내 표준 클라이언트를 이 둘로 정하고 나머지는 정책상 막는 편이 단순하다.
2. **데이터 흐름을 정책 문서에 반영한다.** 디버거 연결은 로컬이지만, 디버깅 컨텍스트는 AI 클라이언트에 설정된 모델 서비스로 나갈 수 있다. 크래시 덤프에는 메모리 내용, 즉 자격 증명이나 고객 데이터가 섞여 있을 수 있다. 어떤 덤프를 AI 세션에 올려도 되는지 분류 기준을 먼저 정해야 한다.
3. **토큰 비용이 늘어난다는 점을 감안한다.** 교차 프롬프트 인젝션 보호는 출력 분류에 모델을 추가로 호출한다. Copilot 사용량이 플랜 한도나 프리미엄 요청으로 관리되는 조직이라면, 덤프 분석이 잦은 팀의 사용량을 따로 지켜볼 필요가 있다. 구체적인 증가량은 글에 없다.
4. **Secure Mode 순서를 교육 포인트로 삼는다.** "MCP 먼저 시작, 대상은 나중에 연결"을 지키지 않으면 부분 Secure Mode로 동작한다. 실습 가이드에 이 순서를 체크 항목으로 넣고, AI가 실행한 명령을 WinDbg 창에서 직접 읽어 보는 과정을 반드시 포함한다. 결론을 그대로 복사하는 습관을 막는 게 이 도구 교육의 핵심이다.
5. **감사 로그 보관 위치를 확인한다.** AI가 시작한 동작은 로그에 남는다고 하지만, 저장 위치와 보관 기간은 발표에 없다. 사고 분석 증빙으로 쓰려면 Learn 문서에서 확인한 뒤 수집 방식을 정한다.

MCP 서버를 직접 만들어 보는 쪽이라면 [첫 MCP 서버 기록](/posts/memo-mcp-first-server/)과 [MCP와 API의 차이](/posts/mcp-vs-api/)도 함께 보면 이 설계의 의미가 더 잘 보인다.

## 참고

- [Introducing WinDbg MCP: Debug with natural language, grounded in evidence](https://devblogs.microsoft.com/performance-diagnostics/introducing-windbg-mcp-debug-with-natural-language-grounded-in-evidence/) — Microsoft Performance & Diagnostics Blog, 2026-10-06
- [Set up WinDbg MCP — Microsoft Learn](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/set-up-windbg-mcp)
- [microsoft/win-dev-skills — GitHub](https://github.com/microsoft/win-dev-skills)
