---
title: "Spec Kit 확장 구조 — 프리셋·확장·번들·워크플로로 키우는 조직용 스펙 주도 개발"
date: 2026-10-08 06:49:00 +0900
categories: [AI Agent]
tags: [spec-kit, spec-driven-development, ai-agent, github, governance]
description: GitHub Spec Kit을 포크하지 않고 조직 기준에 맞추는 네 겹의 확장 구조와 두 층 카탈로그, 도입 순서를 정리했다.
---

> 이 글은 Microsoft 개발자 블로그의 "From spec-first to enterprise-ready: extending GitHub Spec Kit"을 읽고 정리·재구성한 것이다. 명령어와 설치 절차는 원문에 없어서 다루지 않는다.
{: .prompt-info }

## 결론부터

스펙을 먼저 주고 코딩 에이전트가 따라가게 하는 방식을 팀 단위로 키우려면, Spec Kit 코어는 건드리지 않고 **확장 지점만 쌓으라**는 이야기다. 원문의 핵심 문장은 "안정된 코어, 명시적인 확장 지점"이다. 코어를 고쳐 쓰기 시작하면 업그레이드가 막히고 로컬 수정이 오래가는 포크가 되기 때문이다.

배경을 짚으면, Spec Kit은 프로젝트 원칙 → 기능 스펙 → 계획 → 작업 → 코드 → 테스트로 이어지는 추적 가능한 산출물 사슬을 만든다. 이 글은 그 사슬을 조직 표준에 맞추는 방법을 다룬 후속편이다.

## 네 겹의 확장

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="Spec Kit 코어 위에 프리셋, 확장, 번들, 워크플로가 쌓이는 구조">
  <defs>
    <marker id="sk-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 코어는 그대로, 위에 네 겹</text>
  <rect x="8" y="190" width="704" height="44" rx="8" class="sk-box-muted"/>
  <text x="360" y="217" text-anchor="middle" class="sk-label">Spec Kit 코어 (수정하지 않음)</text>
  <rect x="8" y="60" width="160" height="100" rx="8" class="sk-box"/>
  <text x="88" y="92" text-anchor="middle" class="sk-label">프리셋</text>
  <text x="88" y="112" text-anchor="middle" class="sk-sub">있는 것을 바꾼다</text>
  <text x="88" y="130" text-anchor="middle" class="sk-sub">템플릿·명령·용어</text>
  <rect x="192" y="60" width="160" height="100" rx="8" class="sk-box"/>
  <text x="272" y="92" text-anchor="middle" class="sk-label">확장</text>
  <text x="272" y="112" text-anchor="middle" class="sk-sub">없는 것을 더한다</text>
  <text x="272" y="130" text-anchor="middle" class="sk-sub">명령·게이트·연동</text>
  <rect x="376" y="60" width="160" height="100" rx="8" class="sk-box"/>
  <text x="456" y="92" text-anchor="middle" class="sk-label">번들</text>
  <text x="456" y="112" text-anchor="middle" class="sk-sub">버전 붙은 설치 단위</text>
  <text x="456" y="130" text-anchor="middle" class="sk-sub">프리셋+확장 묶음</text>
  <rect x="560" y="60" width="152" height="100" rx="8" class="sk-box-accent"/>
  <text x="636" y="92" text-anchor="middle" class="sk-label">워크플로</text>
  <text x="636" y="112" text-anchor="middle" class="sk-sub">사람 확인 지점이</text>
  <text x="636" y="130" text-anchor="middle" class="sk-sub">있는 반복 절차</text>
  <path d="M170,110 L190,110" class="sk-line" marker-end="url(#sk-m1)"/>
  <path d="M354,110 L374,110" class="sk-line" marker-end="url(#sk-m1)"/>
  <path d="M538,110 L558,110" class="sk-line-accent" marker-end="url(#sk-m1)"/>
</svg>
<figcaption>바꾸는 것(프리셋)과 더하는 것(확장)을 나누고, 검증된 조합만 번들로 묶는다.</figcaption>
</figure>

- **프리셋**: 템플릿, 명령, 프롬프트, 용어를 우선순위 순서로 덮어쓰는 층이다. 위협 모델링 프롬프트를 강화하는 보안 프리셋, 인수 기준을 필수로 넣는 도메인 프리셋 같은 예가 나온다.
- **확장**: 기본 수명주기에 없던 기능을 더한다. 보안 리뷰 명령, 접근성 게이트, 사내 작업 관리 시스템 연동, 버그 분류 흐름 등이다.
- **번들**: 프리셋과 확장을 버전이 붙은 설치 단위 하나로 묶는다. 예로 조직 기준선, 서비스별 품질 게이트, 버그 처리 워크플로, 필수 연동을 한데 묶은 개발자 번들이 나온다. 설치 단계 여러 개 대신 관리되는 진입점 하나가 생기고, 버전 고정과 출처 추적 덕에 갱신·제거도 쉬워진다.
- **워크플로**: 명령, 프롬프트, 스크립트를 사람 확인 지점과 함께 잇는다. 조건 분기, 반복, 일시 정지 후 재개를 지원하고, 버그 수정·릴리스 준비·보안 리뷰·아키텍처 검증·장애 후속 조치처럼 증빙이 필요한 작업에 맞는다. 원문은 목적이 사람의 판단을 없애는 게 아니라 순서·산출물·확인 지점을 명시적으로 만드는 것이라고 못 박는다.

확장의 예로 든 것 중 하나가 **토큰 사용량 가시화**다. 실행이 끝날 때마다 사용량을 모으고, 추세를 보여 주고, 임계값을 넘으면 최적화 안내를 띄운다. 코어 수명주기는 그대로 둔 채 붙였다 뗄 수 있다. 원문은 같은 패턴을 접근성 검사, 보안 개발 리뷰, 의존성 관리, 문서 품질, 서비스 상태 검증에도 쓸 수 있다고 본다.

## 카탈로그는 보안 경계다

카탈로그는 두 층이다. **조직 카탈로그**는 검토를 거친 설치 가능한 소스로 해석 우선순위가 가장 높다. **커뮤니티 카탈로그**는 기본적으로 탐색 전용이고, 목록에 올라 있다는 것은 형식과 메타데이터가 맞다는 뜻이지 보안 검토나 보증이 아니다. 원문은 이 분리를 보안 경계로 본다. 마음껏 둘러보되 설치는 검증된 소스로만 하라는 뜻이다. 조직 카탈로그 쪽에서는 기여 기준, 출처, 프롬프트 인젝션 방어, 소유자 명시를 강제할 수 있다고 한다.

## constitution도 겹쳐 쓴다

프로젝트마다 원칙을 새로 쓰는 대신 세 겹으로 쌓는다.

1. 조직 기준: 어디서나 지켜야 하는 항목
2. 도메인·기술 덧씌우기: 플랫폼, 언어, 아키텍처별 규칙
3. 팀 맞춤: 꼭 필요한 곳에만 허용하는 예외와 추가

원문은 이것이 즉흥적인 프롬프트 지시보다 에이전트에게 오래가는 가드레일이 된다고 본다. [앞서 정리한 문서 구조](/posts/memo-mcp-first-server/)와 같은 결로, 기준을 위에서 아래로 내려보내는 방식이다.

## 도입 순서

원문이 제시하는 여섯 단계를 줄이면 이렇다. 조직 constitution과 기본 통제를 먼저 정하고, 정책과 용어는 프리셋으로, 새 기능은 확장으로 만든다. 그다음 조직 카탈로그를 큐레이션하는데, 설치를 허용하기 전에 소유자, 출처, 권한, 프롬프트, 스크립트, 업데이트 관행을 검토하라고 한다. 이어서 검증된 조합을 번들로 묶고, 워크플로는 **절차와 산출물, 확인 지점, 예외 경로가 분명해진 뒤에** 얹는다. 순서가 핵심이다. 흐름이 불분명한 일을 먼저 워크플로로 만들면 불분명함이 자동화될 뿐이다.

## 읽고 나서 남은 것

내가 보기엔 이 글의 값어치는 도구 소개보다 "바꾸는 것과 더하는 것을 나누라"는 설계 원칙에 있다. 다만 원문은 개념 설명 위주라 명령어, 파일 구조, 구현 방법은 없고 Spec Kit 저장소 문서를 보라고 한다. 프리셋과 확장을 실제로 만들어 보는 일은 문서를 확인한 뒤의 숙제로 남겨 둔다.

## 실무에서 달라지는 점

개발 도구를 사내에 배포하는 관리자, 그리고 스펙 주도 개발을 가르치는 강사 입장에서 정리하면 이렇다.

1. **조직 카탈로그는 소프트웨어 반입 절차로 다룬다.** 확장에는 스크립트와 훅이 들어갈 수 있다. 원문이 꼽은 검토 항목(소유자, 출처, 권한, 프롬프트, 스크립트, 업데이트 관행)은 사내 오픈소스 반입 심사 체크리스트와 거의 같다. 기존 심사 절차에 "프롬프트 검토" 한 줄을 더하는 방식으로 시작하면 새 조직을 만들 필요가 없다.
2. **커뮤니티 카탈로그 설치를 기본 차단한다.** 커뮤니티 목록에 올라 있다는 건 형식 검사를 통과했다는 뜻일 뿐이다. 개발자 PC에서 커뮤니티 소스 설치를 막을 수 있는지, 막을 수 없다면 어떤 방식으로 탐지할지는 Spec Kit 문서에서 확인해야 할 부분이다. 원문은 구체적인 설정 방법을 다루지 않는다.
3. **번들 버전을 변경 관리 대상으로 올린다.** 번들은 버전이 고정되고 출처가 추적된다. 즉 "이번 분기 개발자 번들 v2"처럼 릴리스 노트를 쓰고 롤백 경로를 두는 변경 관리가 가능해진다. 조직 constitution이 바뀌면 번들 버전도 올라가야 한다.
4. **비용 가시화 확장을 초기에 넣는다.** 코딩 에이전트의 토큰 사용량은 팀마다 편차가 크다. 원문의 토큰 사용량 확장 예처럼, 비용 측정은 나중에 붙이기보다 첫 번들에 포함하는 편이 예산 협의에 유리하다.
5. **강의에서는 프리셋과 확장의 경계를 먼저 연습시킨다.** "이건 바꾸는 것인가, 더하는 것인가"를 판단하는 연습이 이 구조의 핵심이다. 사내 보안 기준 하나를 예로 들어 프리셋(위협 모델링 프롬프트 강화)과 확장(보안 리뷰 명령)으로 각각 나눠 보게 하면 개념이 빨리 잡힌다.

같은 시기 나온 [Canvas authoring 플러그인](/posts/azure-canvas-authoring-plugin/) 역시 마켓플레이스에서 플러그인을 설치하는 구조라, 카탈로그 통제 원칙을 함께 적용해 볼 만하다.

## 참고

- [From spec-first to enterprise-ready: extending GitHub Spec Kit — Microsoft Developer Blog](https://developer.microsoft.com/blog/from-spec-first-to-enterprise-ready-extending-github-spec-kit/) — 2026-10-07
- [github/spec-kit — GitHub](https://github.com/github/spec-kit)
