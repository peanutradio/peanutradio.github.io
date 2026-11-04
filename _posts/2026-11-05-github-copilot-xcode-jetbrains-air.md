---
title: "Copilot, Xcode 코딩 어시스턴트와 JetBrains Air로 — 두 곳의 차이"
date: 2026-11-05 06:53:00 +0900
categories: [IT News]
tags: [copilot, github, ai-agent]
description: GitHub Copilot이 11/3 JetBrains Air, 11/4 Xcode 코딩 어시스턴트에 들어왔다. 상태와 빠진 기능, 도입 전 점검 항목을 정리했다.
---

> 이 글은 GitHub Changelog(11/4)와 Microsoft Java 블로그(11/3) 발표를 읽고 정리·재구성한 것입니다. 직접 설치해 써 본 기록이 아닙니다.
{: .prompt-info }

## 결론부터

이틀 사이에 Copilot이 두 개의 다른 에이전트 환경에 들어왔습니다. 하나는 Xcode의 내장 코딩 어시스턴트이고, 다른 하나는 JetBrains Air입니다. 둘 다 "Copilot을 쓰려고 특정 IDE 플러그인을 깔던 방식"에서 "그 환경이 제공하는 에이전트 선택기에서 Copilot을 고르는 방식"으로 넘어간 사례입니다.

다만 두 소식은 성숙도가 다릅니다. Xcode 쪽은 **퍼블릭 프리뷰**이고 베타 도구가 필요합니다. Air 쪽은 원문에 프리뷰·GA 표기가 없습니다. 그리고 Air 쪽 원문은 플러그인과 Air 통합의 기능 차이를 표로 직접 보여 주는데, 이 표가 오늘 소식에서 실무적으로 가장 쓸모 있는 부분입니다.

## 무엇이 발표됐나

### Xcode 코딩 어시스턴트 (11/4, 퍼블릭 프리뷰)

Xcode에 내장된 코딩 어시스턴트 안에서 Copilot을 직접 쓸 수 있게 됐습니다. 요구 조건은 Xcode 27.3 베타와 macOS Tahoe 26.6 이상입니다. 설정은 Xcode > Settings > Intelligence에서 하고, 원문은 모든 Copilot 요금제를 지원한다고 적었습니다.

기존 GitHub Copilot for Xcode 확장은 없어지지 않고 별도 선택지로 남습니다. 즉 이번 발표는 대체가 아니라 경로 추가입니다. 지원 모델이나 프리미엄 요청 처리 방식은 원문에 없습니다.

### JetBrains Air (11/3)

Air의 기본 에이전트 선택기에서 Copilot을 고르고, 로그인한 뒤 프롬프트를 보내는 흐름입니다. JetBrains와 협력해 만들었고 Agent Client Protocol(ACP) 기반이며, 별도의 Copilot 플러그인을 설치·관리할 필요가 없다고 합니다. 11/3부터는 기존 Local Run에 더해 Air의 Cloud Run에서도 쓸 수 있습니다. 필요한 것은 Copilot 구독입니다.

별개로, JetBrains IDE 2026.2.4 이상에서는 Next Edit Suggestions의 제공자로 Copilot을 고를 수 있다고 합니다.

## 플러그인과 Air는 같은 Copilot이 아니다

원문 비교표를 제 식으로 다시 묶으면 아래와 같습니다.

<figure class="sketch">
<svg viewBox="0 0 720 280" role="img" aria-label="JetBrains에서 Copilot을 쓰는 두 경로, 플러그인과 Air의 기능 차이">
  <defs>
    <marker id="gcx-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① JetBrains에서 Copilot을 쓰는 두 경로</text>
  <rect x="8" y="40" width="200" height="52" rx="8" class="sk-box-accent"/>
  <text x="108" y="62" text-anchor="middle" class="sk-label">Copilot 구독</text>
  <text x="108" y="80" text-anchor="middle" class="sk-sub">공통 전제</text>
  <path d="M210,56 L270,56 L270,36 L300,36" class="sk-line" marker-end="url(#gcx-m1)"/>
  <path d="M210,76 L270,76 L270,176 L300,176" class="sk-line" marker-end="url(#gcx-m1)"/>
  <rect x="302" y="22" width="400" height="104" rx="8" class="sk-box"/>
  <text x="502" y="44" text-anchor="middle" class="sk-label">Copilot 플러그인 (원문 권장)</text>
  <text x="502" y="64" text-anchor="middle" class="sk-sub">인라인 에이전트 채팅 · 코드 리뷰 · 커밋 메시지 생성</text>
  <text x="502" y="82" text-anchor="middle" class="sk-sub">스킬·훅·지침 · Agent Debug · Local Sandbox</text>
  <text x="502" y="100" text-anchor="middle" class="sk-sub">Enterprise Managed Settings · 정책 진단</text>
  <rect x="302" y="146" width="400" height="104" rx="8" class="sk-box-muted"/>
  <text x="502" y="168" text-anchor="middle" class="sk-label">Air의 Copilot 에이전트 (ACP)</text>
  <text x="502" y="188" text-anchor="middle" class="sk-sub">에이전트 선택기 · 병렬 세션 · 모델 선택 · IDE 도구</text>
  <text x="502" y="206" text-anchor="middle" class="sk-sub">클라우드 작업은 Cloud Run (JetBrains Teams 포함)</text>
  <text x="502" y="224" text-anchor="middle" class="sk-sub">위 고급 기능과 엔터프라이즈 제어는 "포함 안 됨"</text>
</svg>
<figcaption>Air는 설치가 가벼운 대신, 조직이 Copilot을 통제하는 기능은 플러그인 쪽에 남아 있다.</figcaption>
</figure>

Air에서 "포함되지 않음"으로 적힌 항목은 스킬·훅·지침 같은 에이전트 커스터마이징, Agent Debug 로그, 인라인 에이전틱 채팅, Local Sandbox, Copilot 코드 리뷰와 커밋 메시지 생성, 그리고 GitHub 엔터프라이즈 제어(Managed Settings, 정책 진단)입니다. 클라우드 작업도 Copilot 코딩 에이전트가 아니라 Air의 Cloud Run을 통해 돌아갑니다. 원문은 완전한 경험으로 플러그인을 권장한다고 밝혔습니다.

제 해석을 덧붙이면, Air 통합은 "Copilot 기능 전체를 옮긴 것"이 아니라 "Copilot의 에이전트 부분을 다른 환경의 틀에 맞춘 것"에 가깝습니다. 앞서 정리한 [Copilot 보조 권한 GA](/posts/copilot-assisted-permissions-ga/)나 [Copilot Fusion](/posts/github-copilot-fusion-ga/)처럼 Copilot 쪽 제어 기능이 계속 늘고 있는데, 선택기로 붙는 외부 환경에서 이 제어가 어디까지 따라오는지는 환경마다 따로 확인해야 합니다.

## 실무에서 달라지는 점

### 1. 조직 정책을 쓰는 팀은 Air를 먼저 허용하지 않는다

Enterprise Managed Settings와 정책 진단이 Air 쪽에 "포함 안 됨"으로 표시돼 있습니다. 관리자가 중앙에서 Copilot 설정을 강제하는 조직이라면, 개발자가 Air에서 Copilot을 고르는 경로가 그 통제 밖일 수 있다는 뜻입니다. 허용 전에 사내 정책이 Air에서도 적용되는지 GitHub 문서나 담당자에게 확인해야 합니다. 원문만으로는 적용 여부를 단정할 수 없습니다.

### 2. 요금제 영향은 아직 원문으로 확인되지 않는다

두 원문 모두 프리미엄 요청 소모나 모델별 배수에 관한 언급이 없습니다. "모든 요금제 지원"이 곧 "추가 비용 없음"은 아닙니다. 시범 도입 때는 에이전트 세션을 돌린 뒤 사용량 화면에서 소모량을 직접 확인하는 단계를 넣는 편이 안전합니다.

### 3. Xcode는 베타 조합이라 업무 머신에 얹지 않는다

Xcode 27.3 베타와 macOS 26.6 이상이 모두 필요합니다. 앱 출시용 빌드 환경과 분리된 테스트 맥에서 먼저 보는 것이 맞습니다. 기존 확장이 계속 제공되므로, 지금 쓰는 사람은 서두를 이유가 없습니다.

### 4. 교육할 때는 "어디서 고르느냐"가 아니라 "무엇이 빠지느냐"를 강조한다

수강생이 가장 헷갈릴 부분은 같은 Copilot인데 환경마다 되는 기능이 다르다는 점입니다. 교육 자료에는 위 비교표 같은 기능 대조를 넣고, 스킬·훅·지침을 가르치는 과정이라면 실습 환경을 플러그인 쪽으로 못 박아 두는 것이 좋습니다. Air에서는 그 단원이 그대로 재현되지 않을 수 있습니다.

### 5. 프로토콜 관점에서 ACP를 눈여겨본다

Air 통합의 바탕은 ACP입니다. 에이전트가 특정 IDE 플러그인이 아니라 프로토콜로 연결되는 구조라, 앞으로 비슷한 방식의 환경이 더 나올 가능성이 있습니다. 다만 이는 제 추정이고, 원문이 약속한 로드맵은 아닙니다. 지금은 "에이전트를 고르는 선택기" 개념을 교육 자료에 한 줄 추가하는 정도면 충분합니다.

## 아직 확인되지 않은 것

- Air 통합의 프리뷰·GA 여부와 필요한 Air 버전은 원문에 없습니다.
- Xcode 쪽 지원 모델과 프리미엄 요청 처리 방식도 원문에 없습니다.
- 두 환경에서 실제 응답 품질이나 속도가 어떤지는 원문이 다루지 않았고, 저도 직접 측정하지 않았습니다.

## 참고

- [GitHub Copilot is now available in Xcode's coding assistant — GitHub Changelog, 2026-11-04](https://github.blog/changelog/2026-11-04-github-copilot-is-now-available-in-xcodes-coding-assistant)
- [GitHub Copilot is now a default agent in JetBrains Air — Microsoft Java Blog, 2026-11-03](https://devblogs.microsoft.com/java/github-copilot-jetbrains-air/)
- [GitHub Copilot is now available in JetBrains Air — GitHub Changelog, 2026-11-03](https://github.blog/changelog/2026-11-03-github-copilot-is-now-available-in-jetbrains-air)
- [Setting up coding intelligence — Apple Developer Documentation](https://developer.apple.com/documentation/xcode/setting-up-coding-intelligence)
- [Agent Client Protocol in JetBrains](https://www.jetbrains.com/acp/)
