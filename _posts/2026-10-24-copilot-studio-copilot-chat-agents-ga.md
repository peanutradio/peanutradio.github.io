---
title: "Copilot Studio, Copilot Chat 에이전트 제작 기능 10/26 GA 예정"
date: 2026-10-24 06:51:00 +0900
categories: [IT News]
tags: [copilot-studio, copilot-chat, declarative-agent, alm, microsoft-365]
description: Copilot Studio가 Copilot Chat 에이전트용 토픽·연결된 에이전트·평가·모니터링·환경별 배포를 10월 26일 GA로 낸다. 운영자가 챙길 점을 정리했다.
---

> 이 글은 Microsoft 365 Message Center 공지 MC1518233(2026-10-21 게시)을 읽고 정리·재구성한 것입니다. 공식 Learn 문서가 아니라 Message Center 미러 페이지를 읽었으므로, 세부 동작은 발행 시점의 공식 문서로 다시 확인해 주세요.
{: .prompt-info }

## 결론부터

10월 26일(월) Copilot Studio에서 **Copilot Chat 에이전트(선언형 에이전트) 제작 지원이 GA 예정**입니다. 공지가 말하는 용도는 HR 표준 답변처럼 권위 있고 일관된 답, 브랜드·법무 지침을 따르는 콘텐츠입니다. 관리자 조치는 "필요 없음"이지만, 제작자 교육 자료와 사내 가이드는 손봐야 합니다. 아직 예정일이어서 실제 출시는 확인하지 못했습니다.

## 공지가 나열한 기능

- **토픽**: 정해진 대화 흐름을 결정론적으로 만들 수 있습니다.
- **연결된 에이전트**: 다른 에이전트에 작업을 위임합니다.
- **평가**: 에이전트 품질을 개선하기 위한 기능입니다.
- **모니터링**: 에이전트 사용량을 추적합니다.
- **네이티브 Copilot Chat 미리보기 + Activity Map**: 사용자가 보는 화면 그대로 테스트하고 동작을 확인합니다.
- **수명주기 관리**: 개발·테스트·프로덕션 환경에 에이전트 버전을 나눠 배포합니다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="개발, 테스트, 프로덕션 환경을 거쳐 배포되고 평가와 모니터링이 되먹임되는 흐름">
  <defs>
    <marker id="cs-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="cs-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 제작 → 검증 → 배포 → 관찰</text>
  <rect x="8" y="50" width="150" height="56" rx="8" class="sk-box"/>
  <text x="83" y="74" text-anchor="middle" class="sk-label">개발 환경</text>
  <text x="83" y="92" text-anchor="middle" class="sk-sub">토픽·연결된 에이전트</text>
  <rect x="200" y="50" width="150" height="56" rx="8" class="sk-box"/>
  <text x="275" y="74" text-anchor="middle" class="sk-label">테스트 환경</text>
  <text x="275" y="92" text-anchor="middle" class="sk-sub">평가·Activity Map</text>
  <rect x="392" y="50" width="150" height="56" rx="8" class="sk-box-accent"/>
  <text x="467" y="74" text-anchor="middle" class="sk-label">프로덕션</text>
  <text x="467" y="92" text-anchor="middle" class="sk-sub">승인된 버전만</text>
  <rect x="584" y="50" width="128" height="56" rx="8" class="sk-box-muted"/>
  <text x="648" y="74" text-anchor="middle" class="sk-label">모니터링</text>
  <text x="648" y="92" text-anchor="middle" class="sk-sub">사용량 추적</text>
  <path d="M160,78 L198,78" class="sk-line" marker-end="url(#cs-m1)"/>
  <path d="M352,78 L390,78" class="sk-line" marker-end="url(#cs-m1)"/>
  <path d="M544,78 L582,78" class="sk-line-accent" marker-end="url(#cs-a2)"/>
  <path d="M648,108 L648,160 L83,160 L83,108" class="sk-line" marker-end="url(#cs-m1)"/>
  <text x="365" y="152" text-anchor="middle" class="sk-sub">관찰한 결과를 다음 버전에 반영</text>
</svg>
<figcaption>이번 업데이트의 핵심은 새 기능 하나가 아니라, 에이전트를 '버전이 있는 산출물'로 다룰 수 있게 된다는 점이다.</figcaption>
</figure>

## 이 블로그의 해석

공지 자체는 짧고 기능 이름만 나열합니다. 읽으면서 눈에 띈 건 기능 목록이 **"만들기"보다 "운영하기"에 치우쳐 있다**는 점입니다. 평가, 모니터링, 환경별 버전 배포는 모두 에이전트를 한 번 만들고 끝내지 않고 계속 고치겠다는 전제의 기능입니다. 이전 글에서 [Copilot Studio에 MCP 서버를 붙이는 법](/posts/copilot-studio-mcp-tools/)을 다뤘는데, 연결 방법을 알아도 변경을 안전하게 배포할 수단이 없으면 현업에 내놓기 부담스럽습니다. 이번 기능은 그 빈칸을 채우는 쪽입니다. 반대로 토픽은 모델이 알아서 답하게 두는 대신 "이 질문엔 이 답"을 고정하는 장치라, 표준 답변이 중요한 HR·법무 시나리오에 맞는 선택지로 보입니다. 이 해석은 공지 문구에서 추론한 것이며 공지에 적힌 내용이 아닙니다.

## 실무에서 달라지는 점

1. **Copilot Studio 제작자·센터 오브 엑셀런스**: 10/26 이후 Copilot Chat 에이전트를 만드는 화면에서 토픽·평가·모니터링이 보이는지 먼저 확인하세요. 테넌트·라이선스·리전별 제공 여부는 공지에 없으므로 보인다고 단정하면 안 됩니다.
2. **M365·Power Platform 관리자**: 공지상 조치는 없지만, 개발·테스트·프로덕션 환경을 이미 나눠 쓰는지 점검할 좋은 계기입니다. 환경이 하나뿐이면 수명주기 기능을 써도 배포 단계가 나뉘지 않습니다. 환경 신설은 DLP 정책과 권한 설계가 따라오므로 미리 상의하세요.
3. **사내 문서·교육 담당**: Microsoft도 내부 문서와 교육 자료 갱신을 권장합니다. 기존 "Copilot Studio로 에이전트 만들기" 교재에 평가와 버전 배포 단계가 빠져 있다면 이번에 추가할 항목입니다.
4. **IT 강사**: 수강생에게 "토픽 = 정해진 답, 연결된 에이전트 = 위임"의 차이를 먼저 짚어 주세요. 두 기능을 섞어 쓰면 어떤 경로로 답이 나왔는지 헷갈리는데, 이때 Activity Map이 설명 도구로 쓸 만해 보입니다(실제 화면은 GA 후 확인 필요).
5. **도입 전 확인할 것**: 라이선스·추가 요금, 기능별 세부 동작, 프리뷰 표기가 남는 항목이 있는지는 공지에 없습니다. 공식 문서와 관리 센터 알림에서 확인한 뒤 파일럿부터 시작하는 편이 안전합니다.

## 참고

- Microsoft 365 Message Center MC1518233 미러: [Copilot Studio, Enhanced support for Copilot Chat agents](https://mc.merill.net/message/MC1518233)
- 이전 글: [Copilot Studio에 MCP 서버 붙이기](/posts/copilot-studio-mcp-tools/)
