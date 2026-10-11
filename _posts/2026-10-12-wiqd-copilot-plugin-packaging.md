---
title: "wiqd로 Copilot 플러그인 배포하기 — 검증 통과가 끝이 아니다"
date: 2026-10-12 06:53:00 +0900
categories: [Copilot Studio]
tags: [wiqd, copilot-plugin, mcp, microsoft-365, agent-plugins]
description: Work IQ Developer Tools(wiqd)로 Copilot 플러그인을 가져와 패키징하고 테넌트에 공유하는 흐름과, 운영자가 따로 챙겨야 할 지점을 정리했다.
---

> 이 글은 Microsoft 365 개발자 블로그의 "Bringing a Plugin to Microsoft Copilot with WIQD" 글을 읽고 정리·재구성한 것이다. 직접 실행해 본 기록이 아니라 원문 기준의 소개·정리다.
{: .prompt-info }

**결론부터.** wiqd는 스킬과 MCP 커넥터로 이루어진 플러그인을 "가져오기 → 검증 → 프로비저닝 → 패키징 → 공유"까지 명령어 몇 줄로 묶어 준다. 다만 두 가지는 도구가 대신해 주지 않는다. 스킬의 지침 문장 작성, 그리고 MCP 서버 인증 설정이다. 게다가 오프라인 검증은 스킬 내용을 보지 않는다. 그래서 "validate 통과 = 배포 가능"이라고 읽으면 안 된다. 명령 표면(`wiqd plugin`)은 공개 프리뷰이고 바뀔 수 있다고 원문이 밝힌다.

앞서 [Canvas authoring 플러그인](/posts/azure-canvas-authoring-plugin/) 글에서는 플러그인을 "쓰는" 쪽을 다뤘다. 이번에는 "만들어서 배포하는" 쪽이다.

## 원문이 말하는 흐름

시작점은 세 가지다. 기존 Agent Plugins 형식 플러그인을 `wiqd plugin import --path ...`로 가져오거나(개인정보·약관 URL 옵션 포함), 코딩 하니스(원문은 GitHub Copilot 권장)에 자연어로 만들어 달라고 하거나, 터미널에서 `wiqd plugin create`, `add skill`, `add connector`를 직접 치는 방법이다. 이후 순서는 이렇다.

<figure class="sketch">
<svg viewBox="0 0 720 150" role="img" aria-label="wiqd 플러그인 배포 단계 흐름도">
  <defs>
    <marker id="wq-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① wiqd 배포 흐름</text>
  <rect x="4" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="64" y="74" text-anchor="middle" class="sk-label">만들기/가져오기</text>
  <text x="64" y="92" text-anchor="middle" class="sk-sub">create · import</text>
  <rect x="148" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="208" y="74" text-anchor="middle" class="sk-label">오프라인 검증</text>
  <text x="208" y="92" text-anchor="middle" class="sk-sub">내용은 안 봄</text>
  <rect x="292" y="50" width="130" height="56" rx="8" class="sk-box-accent"/>
  <text x="357" y="74" text-anchor="middle" class="sk-label">provision</text>
  <text x="357" y="92" text-anchor="middle" class="sk-sub">M365 환경 변경</text>
  <rect x="446" y="50" width="120" height="56" rx="8" class="sk-box"/>
  <text x="506" y="74" text-anchor="middle" class="sk-label">package</text>
  <text x="506" y="92" text-anchor="middle" class="sk-sub">+ deep 검증</text>
  <rect x="590" y="50" width="124" height="56" rx="8" class="sk-box"/>
  <text x="652" y="74" text-anchor="middle" class="sk-label">share</text>
  <text x="652" y="92" text-anchor="middle" class="sk-sub">users / tenant</text>
  <path d="M126,78 L146,78" class="sk-line" marker-end="url(#wq-m1)"/>
  <path d="M270,78 L290,78" class="sk-line" marker-end="url(#wq-m1)"/>
  <path d="M424,78 L444,78" class="sk-line" marker-end="url(#wq-m1)"/>
  <path d="M568,78 L588,78" class="sk-line" marker-end="url(#wq-m1)"/>
</svg>
<figcaption>강조된 provision 단계만 실제 Microsoft 365 환경을 바꾼다. 나머지는 로컬 작업이다.</figcaption>
</figure>

로그인은 `wiqd auth login --interactive`, 내부 공유는 `wiqd plugin share --scope users --email ...` 또는 `--scope tenant`다. 외부 고객에게 파는 ISV라면 패키지를 Partner Center에 제출해 인증을 받는다. wiqd는 패키지를 준비할 뿐 공개 제출과 인증은 Partner Center 몫이다.

## 이 글의 해석: 검증이 두 겹인 이유

원문을 읽으며 가장 눈에 띈 건 검증 단계가 둘로 나뉜다는 점이다. 처음의 `validate`는 오프라인이며 스킬 내용과 최상위 앱 매니페스트를 검토하지 않는다. 패키징 뒤의 `validate --mode deep`이 만들어진 패키지를 본다. 정리하면 "형식 점검"과 "패키지 점검"일 뿐, "스킬이 의도대로 동작하는지"는 어디서도 검증되지 않는다. 그 부분은 사람이 Copilot에서 써 보고 확인해야 한다.

<figure class="sketch">
<svg viewBox="0 0 720 150" role="img" aria-label="도구가 해 주는 일과 사람이 해야 하는 일의 구분">
  <defs>
    <marker id="wq-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 도구와 사람의 경계</text>
  <rect x="8" y="40" width="330" height="90" rx="8" class="sk-box"/>
  <text x="173" y="64" text-anchor="middle" class="sk-label">wiqd가 해 주는 것</text>
  <text x="173" y="88" text-anchor="middle" class="sk-sub">프로젝트 뼈대 · 스킬/커넥터 등록</text>
  <text x="173" y="106" text-anchor="middle" class="sk-sub">패키지 생성 · 공유 범위 지정</text>
  <rect x="382" y="40" width="330" height="90" rx="8" class="sk-box-accent"/>
  <text x="547" y="64" text-anchor="middle" class="sk-label">사람이 해야 하는 것</text>
  <text x="547" y="88" text-anchor="middle" class="sk-sub">스킬 지침 문장 · MCP 인증 설정</text>
  <text x="547" y="106" text-anchor="middle" class="sk-sub">Copilot 화면별 동작 확인</text>
  <path d="M340,85 L380,85" class="sk-line-accent" marker-end="url(#wq-m2)"/>
</svg>
<figcaption>자동화되는 부분은 껍데기이고, 품질을 좌우하는 부분은 여전히 사람 몫이다.</figcaption>
</figure>

## 실무에서 달라지는 점

- **M365 관리자: `provision` 전에 어떤 계정으로 돌리는지 정한다.** 원문은 이 단계가 Microsoft 365 환경을 변경한다고만 밝히고 필요한 권한은 명시하지 않았다. 개발용 테넌트나 테스트 계정에서 먼저 돌리고, 운영 테넌트에서는 변경 내용을 확인한 뒤 실행하는 편이 안전하다.
- **`--scope tenant`는 승인 절차 뒤에 둔다.** 테넌트 전체 공유는 한 줄이라 쉽지만 되돌리기는 별개 문제다. 먼저 `--scope users`로 소수 검증 인원에게 풀고, 이후 전사로 넓히는 2단계 운영을 권한다. 이건 원문이 아니라 이 글의 제안이다.
- **MCP 커넥터를 붙이는 팀: 인증을 먼저 설계한다.** 커넥터 등록은 이름·설명·URL이면 끝나지만, 서버 쪽 인증 설정은 직접 해야 한다. 어떤 사용자 권한으로 호출되는지 보안 팀과 맞추지 않으면 공유 후에 막힌다. 관련해서는 곧 올라올 「Copilot Studio에 MCP 서버 붙이기」 글도 참고할 만하다.
- **검증 통과를 승인 기준으로 쓰지 않는다.** 스킬 내용은 오프라인 검증 대상이 아니므로, 대표 질문 몇 개로 Copilot에서 결과를 보는 체크리스트가 필요하다. 원문은 같은 패키지라도 Copilot 화면마다 경험이 다를 수 있다고 경고한다. 사용할 화면별로 지원 여부를 확인한 뒤 공유한다.
- **교육할 때는 "프리뷰"를 강조한다.** 명령과 옵션이 바뀔 수 있고 GA 시점은 원문에 없다. 교육 자료에 명령어를 박제하기보다 흐름(검증 → 프로비저닝 → 패키징 → 공유)을 가르치고, 실습 직전에 명령 레퍼런스를 다시 확인하는 쪽이 낫다.

## 정리

wiqd는 플러그인 배포의 반복 작업을 줄여 주지만, 스킬 품질과 인증은 그대로 사람의 숙제다. 도입을 검토한다면 소규모 사용자 공유로 시작해 화면별 동작을 확인하고, 그다음 테넌트로 넓히자. 프리뷰 단계인 만큼 운영 반영 전에 명령 레퍼런스를 다시 읽는 것도 잊지 말자.

## 참고

- [Bringing a Plugin to Microsoft Copilot with WIQD — Microsoft 365 Developer Blog](https://devblogs.microsoft.com/microsoft365dev/bring-your-plugin-to-microsoft-copilot-with-wiqd/)
