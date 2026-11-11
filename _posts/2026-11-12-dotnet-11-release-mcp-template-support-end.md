---
title: ".NET 11 출시 — 8·9 지원 종료일과 MCP 템플릿, 실험 기능 구분"
date: 2026-11-12 06:53:00 +0900
categories: [IT News]
tags: [mcp, ai-agent, devops]
description: .NET 11이 11/10 출시됐다. 8·9는 같은 날 지원이 끝났다. 에이전트·MCP 기능은 대부분 실험 단계라 옮길 것과 기다릴 것을 나눠 정리한다.
---

> 이 글은 .NET 팀의 'Announcing .NET 11'(2026-11-10) 발표를 읽고 정리·재구성한 것입니다. 해석과 운영 관점의 판단은 이 블로그의 의견입니다.
{: .prompt-info }

## 결론부터

.NET 11에서 지금 당장 챙길 일은 신기능이 아니라 지원 종료입니다. 발표문에 따르면 .NET 8과 .NET 9는 2026-11-10에 지원이 끝났고, 오늘(11/12)은 이미 이틀이 지났습니다. 그래서 이 글의 순서는 "무엇이 새로 생겼나"가 아니라 "내 서버가 지금 어느 줄에 서 있나"입니다.

두 번째로, 에이전트와 MCP 쪽 기능은 눈에 띄지만 상태가 제각각입니다. `dotnet new mcpserver` 템플릿은 SDK에 들어왔고, Blazor AI 컴포넌트는 실험 단계이며 .NET 11 동안 prerelease로 남습니다. 이 둘을 같은 무게로 보면 안 됩니다.

## 지원 정책: 어디로 옮길 것인가

원문 기준으로 정리하면 이렇습니다.

- .NET 11은 STS이고 지원 기간은 2년입니다. 발표문은 STS와 LTS의 품질·서비스 정책은 같고 지원 기간 길이만 다르다고 설명합니다.
- .NET 10은 LTS로 계속 지원됩니다. 지원 종료일은 발표문에 적혀 있지 않습니다.
- .NET 8과 9는 2026-11-10에 지원이 종료됐습니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="2026-11-10 기준 .NET 8·9는 지원 종료, .NET 10은 LTS 유지, .NET 11은 STS 2년 지원으로 나뉜 그림">
  <defs>
    <marker id="dn11-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="dn11-a1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 2026-11-10 이후 .NET 버전의 위치</text>
  <rect x="8" y="50" width="150" height="60" rx="8" class="sk-box-muted"/>
  <text x="83" y="76" text-anchor="middle" class="sk-label">.NET 8 · .NET 9</text>
  <text x="83" y="96" text-anchor="middle" class="sk-sub">11/10 지원 종료</text>
  <rect x="285" y="50" width="150" height="60" rx="8" class="sk-box"/>
  <text x="360" y="76" text-anchor="middle" class="sk-label">.NET 10</text>
  <text x="360" y="96" text-anchor="middle" class="sk-sub">LTS, 계속 지원</text>
  <rect x="562" y="50" width="150" height="60" rx="8" class="sk-box-accent"/>
  <text x="637" y="76" text-anchor="middle" class="sk-label">.NET 11</text>
  <text x="637" y="96" text-anchor="middle" class="sk-sub">STS, 2년 지원</text>
  <path d="M160,80 L283,80" class="sk-line" marker-end="url(#dn11-m1)"/>
  <path d="M437,80 L560,80" class="sk-line-accent" marker-end="url(#dn11-a1)"/>
  <text x="222" y="70" text-anchor="middle" class="sk-sub">최소 이동</text>
  <text x="498" y="70" text-anchor="middle" class="sk-sub">신기능 필요 시</text>
  <text x="360" y="160" text-anchor="middle" class="sk-sub">SDK만 설치해서는 앱의 대상 프레임워크가 바뀌지 않는다</text>
</svg>
<figcaption>지원이 끝난 8·9에서 갈 곳은 둘이고, 안정 운영이 목적이면 10, 새 기능이 목적이면 11입니다.</figcaption>
</figure>

원문은 "새 SDK를 설치하는 것만으로는 앱의 대상 프레임워크가 바뀌지 않는다"고 분명히 적고 있습니다. 업그레이드 순서도 제시합니다. `global.json`에 SDK가 고정돼 있으면 갱신하고, 대상 프레임워크를 `net11.0`으로 바꾸고, 패키지와 워크로드를 맞추고, breaking changes를 읽고, 실제 게시 방식과 컨테이너 베이스 이미지까지 확인하라는 흐름입니다.

직전에 다룬 [.NET 10월 긴급 보안 업데이트](/posts/dotnet-october-2026-oob-security-updates/)에서 8.0.33·9.0.22로 올렸던 서버라면, 그 패치가 사실상 마지막 안전 지점이었을 가능성이 큽니다. 정확한 마지막 서비스 버전은 이 발표문에 없으니 지원 정책 페이지에서 확인해야 합니다.

## 실무에서 달라지는 점

### 8·9 서버가 남아 있는 운영자

가장 먼저 할 일은 인벤토리입니다. 사내 인프라에 .NET 8이나 9로 올라간 웹앱, 배치, 윈도우 서비스가 몇 개인지 목록부터 만들어야 합니다. 안정성이 우선이면 .NET 10(LTS)이 최소 이동이고, 11은 STS라 2년 안에 다시 옮겨야 한다는 전제로 선택해야 합니다. 이 판단 기준은 제 의견이며, 원문이 10을 권장한다고 쓴 것은 아닙니다.

### 컨테이너와 CI를 쓰는 팀

이미지의 베이스 태그, `global.json`, 빌드 에이전트의 SDK 버전이 따로 놀기 쉽습니다. 원문이 게시 모드·베이스 이미지·네이티브 의존성·배포 호스트를 검증하라고 한 것은 이런 지점을 가리킨다고 읽었습니다. 재현 가능한 이미지 digest를 위한 `SOURCE_DATE_EPOCH` 지원과 NuGet의 최소 패키지 게시 경과 기간 설정도 들어왔는데, 원문은 후자가 취약점 검사를 대체하지 않는다고 못박았습니다. 설정 키 이름은 본문에 없어서 SDK 문서에서 확인해야 합니다.

### MAUI 앱 담당자

`Microsoft.Maui.Controls.Compatibility` 패키지가 더 이상 제공되지 않습니다. 이 패키지를 참조하는 앱은 마이그레이션이 필요합니다. 또 CoreCLR이 모든 플랫폼의 기본 런타임이 되고, MAUI는 런타임 STS/LTS와 별도의 지원 정책을 따릅니다. 업그레이드 일정은 런타임이 아니라 MAUI 정책 기준으로 잡아야 합니다.

### 에이전트를 .NET으로 만들려는 개발자

MCP 서버를 처음 만드는 사람에게는 진입 장벽이 낮아졌습니다. 템플릿이 SDK에 포함되어 별도 설치 없이 `dotnet new mcpserver -o MyMcpServer`로 시작할 수 있고, 공식 MCP C# SDK와 샘플 도구가 따라옵니다. 다만 이 글을 쓴 저는 템플릿을 실행해 보지 않았습니다. 생성되는 구조와 패키지 ID는 원문에 없어서, 도입 전에 직접 열어 확인해야 합니다.

### 강의에서 강조할 점

수강생에게는 "릴리스 이름"과 "기능 상태"를 분리해서 가르치는 편이 낫습니다. .NET 11이 나왔다고 안에 든 기능이 전부 안정은 아닙니다. 아래 그림처럼 같은 릴리스 안에서도 상태가 갈립니다.

## 에이전트 기능, 상태로 나눠 읽기

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label=".NET 11의 AI 관련 기능을 SDK 포함, 프리릴리스, 실험 단계로 나눈 그림">
  <defs>
    <marker id="dn11-m2" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② .NET 11 안의 AI 기능, 상태별 구분</text>
  <rect x="8" y="40" width="220" height="180" rx="8" class="sk-box-accent"/>
  <text x="118" y="64" text-anchor="middle" class="sk-label">SDK·패키지로 제공</text>
  <text x="118" y="96" text-anchor="middle" class="sk-mono">dotnet new mcpserver</text>
  <text x="118" y="122" text-anchor="middle" class="sk-mono">Microsoft.Extensions.AI</text>
  <text x="118" y="148" text-anchor="middle" class="sk-sub">IChatClient 등 추상화</text>
  <text x="118" y="190" text-anchor="middle" class="sk-sub">상태 라벨은 본문에 없음</text>
  <rect x="250" y="40" width="220" height="180" rx="8" class="sk-box"/>
  <text x="360" y="64" text-anchor="middle" class="sk-label">프리뷰·prerelease</text>
  <text x="360" y="96" text-anchor="middle" class="sk-mono">AG-UI hosting</text>
  <text x="360" y="122" text-anchor="middle" class="sk-mono">dotnetup</text>
  <text x="360" y="148" text-anchor="middle" class="sk-mono">Runtime Async</text>
  <text x="360" y="190" text-anchor="middle" class="sk-sub">운영 투입 전 검증 필요</text>
  <rect x="492" y="40" width="220" height="180" rx="8" class="sk-box-muted"/>
  <text x="602" y="64" text-anchor="middle" class="sk-label">실험(experimental)</text>
  <text x="602" y="96" text-anchor="middle" class="sk-mono">Blazor AI components</text>
  <text x="602" y="122" text-anchor="middle" class="sk-mono">MAUI DevFlow</text>
  <text x="602" y="148" text-anchor="middle" class="sk-mono">EF Core 벡터 검색</text>
  <text x="602" y="190" text-anchor="middle" class="sk-sub">API가 바뀔 수 있다</text>
  <path d="M230,130 L248,130" class="sk-line" marker-end="url(#dn11-m2)"/>
  <path d="M472,130 L490,130" class="sk-line" marker-end="url(#dn11-m2)"/>
</svg>
<figcaption>오른쪽으로 갈수록 바뀔 가능성이 크니, 운영 코드는 왼쪽에서 시작하는 것이 안전합니다.</figcaption>
</figure>

원문이 명시한 상태를 모으면 다음과 같습니다. Blazor AI 컴포넌트(`Microsoft.AspNetCore.Components.AI`)는 실험 단계이고 .NET 11 동안 prerelease로 남으며, ASP.NET Core 공유 프레임워크의 안정 부분이 아닙니다. EF Core 11의 SQL Server 벡터 인덱스와 근사 벡터 검색도 실험입니다. MAUI DevFlow는 실험, Agent Framework의 AG-UI 호스팅 통합은 prerelease, dotnetup은 공개 프리뷰입니다. 반면 `Microsoft.Extensions.AI`는 NuGet 패키지라 앱 전체를 새 프레임워크로 옮기지 않아도 쓸 수 있다고 원문이 설명합니다. 즉 "AI 기능을 쓰려고 11로 올릴 필요는 없다"는 뜻으로 읽힙니다.

이 구분은 제 해석입니다. 에이전트용 UI나 벡터 검색을 이번 분기 프로덕션에 넣을 계획이라면, 기능이 실험 상태라는 점을 설계 문서에 먼저 적어 두는 것이 좋겠습니다.

## 그 밖에 눈에 들어온 것

원문에는 AI 외에도 변경이 많습니다. C# 15의 union types와 closed hierarchies, Native AOT 기반 CLI 호스트(기본 활성화), 기본으로 켜진 MSBuild server, System.IO.Compression의 Zstandard, LINQ `FullJoin`, ASP.NET Core의 OpenAPI 3.2 기본값이 있습니다. 한 가지 주의할 점은 OpenAPI 3.2가 생성 문서의 기본값이라는 것입니다. 도구 체인이 3.2를 지원하지 않으면 이전 문서 버전을 명시적으로 고르라고 하니, 클라이언트 코드 생성기를 쓰는 팀은 업그레이드 후 먼저 확인해야 합니다. 또 Runtime Async는 프리뷰이며 `<Features>runtime-async=on</Features>`를 넣어야 켜집니다. 성능 수치는 이 글에서 인용하지 않았고, 원문이 안내하는 별도 성능 글을 보시기 바랍니다.

## 참고

- [Announcing .NET 11 — .NET Blog](https://devblogs.microsoft.com/dotnet/announcing-dotnet-11/)
- [.NET 11 What's new (Microsoft Learn)](https://learn.microsoft.com/dotnet/core/whats-new/dotnet-11/overview)
- [.NET 지원 정책](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)
- [.NET MAUI 지원 정책](https://dotnet.microsoft.com/platform/support/policy/maui)
- [ASP.NET Core 11 릴리스 노트](https://learn.microsoft.com/aspnet/core/release-notes/aspnetcore-11)
- [EF Core 11 What's new](https://learn.microsoft.com/ef/core/what-is-new/ef-core-11.0/whatsnew)
- [Introducing dotnetup](https://devblogs.microsoft.com/dotnet/introducing-dotnetup/)
- [.NET Skills (GitHub)](https://github.com/dotnet/skills)
