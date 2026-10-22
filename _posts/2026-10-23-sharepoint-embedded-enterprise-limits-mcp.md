---
title: "SharePoint Embedded 엔터프라이즈 업데이트 — 분당 12,000 RU와 MCP 서버(프리뷰)"
date: 2026-10-23 06:51:00 +0900
categories: [IT News]
tags: [sharepoint-embedded, microsoft-365, graph-api, mcp, throttling]
description: SharePoint Embedded가 앱·테넌트당 분당 12,000 RU 한도, 최소 권한 가이드, 코딩 에이전트용 MCP 서버(프리뷰)를 정리했다. 운영 전 확인할 점을 짚는다.
---

> 이 글은 Microsoft 365 Developer Blog의 "Develop enterprise-grade apps with SharePoint Embedded"(2026-10-22, Steve Pucelik)를 읽고 정리·재구성한 것입니다. 한도 값은 원문이 "게시 시점" 기준이라고 밝힌 것이며, 제가 직접 실행해 확인한 것은 아닙니다.
{: .prompt-info }

## 결론부터

이번 글은 신기능 하나가 아니라 **"SharePoint Embedded로 업무 앱을 만들 때 어디까지 믿고 설계해도 되는가"** 를 숫자로 정리한 문서입니다. 눈에 띄는 것은 세 가지입니다. 앱·테넌트당 기본 API 한도가 분당 12,000 리소스 유닛(RU)이라는 점, 권한을 좁게 쓰라는 가이드, 그리고 코딩 에이전트로 컨테이너 유형을 만들 수 있는 MCP 서버(프리뷰)입니다. 이 중 프리뷰로 명시된 것은 MCP 서버뿐입니다.

## 핵심 숫자

원문이 제시한 값입니다. 모두 게시 시점 기준입니다.

| 항목 | 값 |
|---|---|
| 테넌트당 사이트·컨테이너 합계 상한 | 200만 |
| 컨테이너당 검색 가능 열 | 40개 |
| 앱·테넌트당 API 한도 | 분당 12,000 RU |
| 비교: 라이선스 1,000 이하 SharePoint 테넌트 | 분당 1,250 RU (최대 9.6배 차이) |

요청당 약 2 RU를 가정한 추정이므로, 실제 처리량은 호출하는 API의 비용에 따라 달라집니다. 컨테이너는 SharePoint의 "조직당 사이트 수" 한도에 포함된다는 점도 눈여겨볼 대목입니다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="429 또는 503 응답을 받으면 Retry-After를 기다리고 동시성을 낮춘 뒤 제한된 횟수만 재시도하는 흐름">
  <defs>
    <marker id="sp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="sp-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 429/503은 실패가 아니라 속도 조절 신호</text>
  <rect x="8" y="60" width="130" height="56" rx="8" class="sk-box"/>
  <text x="73" y="84" text-anchor="middle" class="sk-label">API 호출</text>
  <text x="73" y="102" text-anchor="middle" class="sk-mono">429 / 503</text>
  <rect x="190" y="60" width="150" height="56" rx="8" class="sk-box-accent"/>
  <text x="265" y="84" text-anchor="middle" class="sk-label">Retry-After 대기</text>
  <text x="265" y="102" text-anchor="middle" class="sk-sub">즉시 재시도 금지</text>
  <rect x="390" y="60" width="140" height="56" rx="8" class="sk-box"/>
  <text x="460" y="84" text-anchor="middle" class="sk-label">동시성 낮춤</text>
  <text x="460" y="102" text-anchor="middle" class="sk-sub">백오프 적용</text>
  <rect x="580" y="60" width="130" height="56" rx="8" class="sk-box-muted"/>
  <text x="645" y="84" text-anchor="middle" class="sk-label">횟수 제한</text>
  <text x="645" y="102" text-anchor="middle" class="sk-sub">초과하면 중단</text>
  <path d="M140,88 L186,88" class="sk-line" marker-end="url(#sp-m1)"/>
  <path d="M342,88 L386,88" class="sk-line" marker-end="url(#sp-m1)"/>
  <path d="M532,88 L576,88" class="sk-line" marker-end="url(#sp-m1)"/>
  <path d="M460,120 L460,160 L73,160 L73,120" class="sk-line-accent" marker-end="url(#sp-a1)"/>
  <text x="270" y="180" text-anchor="middle" class="sk-sub">제한 안에서만 다시 호출</text>
</svg>
<figcaption>재시도는 대기, 동시성 감소, 횟수 상한이 한 세트여야 한다.</figcaption>
</figure>

## 권한과 협업 기능

원문은 앱이 사용자 대신 동작하는 위임 접근을 앱 전용 접근보다 권장합니다. 행동이 개별 사용자에게 묶이기 때문입니다. 또 `FileStorageContainer.Selected` 권한으로 파일 미리보기 같은 작업에서 `Files.Read.All` 같은 넓은 권한을 대체할 수 있다고 합니다. Word, Excel, PowerPoint는 실시간 공동 편집, 자동 저장, 버전 기록이 기본 제공되며, 공유 접근 수준은 테넌트와 관리자 설정에 달려 있습니다. 컨테이너 유형 설정에는 `UrlTemplate`이 추가됐습니다. 뷰어가 지원하지 않는 콘텐츠에서 다운로드로 보내는 식의 리다이렉트용입니다.

## MCP 서버(프리뷰)

코딩 에이전트에 "내 앱용 컨테이너 유형을 만들어 줘"라고 말하면 리소스를 프로비저닝하는 방식입니다. 클라이언트로는 GitHub Copilot(VS Code, CLI), Claude Desktop, Cursor, Azure AI Foundry가 언급됩니다. 같은 제품군의 Retrieval API와 MCP 서버는 [벡터 DB 없이 RAG](/posts/spe-retrieval-api-mcp-no-vector-db/) 글에서 다뤘습니다. 이번 것은 검색이 아니라 **프로비저닝** 쪽이라는 점이 다릅니다.

## 실무에서 달라지는 점

- **앱 개발팀:** 429/503 처리를 "있으면 좋은 것"에서 설계 필수 항목으로 올려야 합니다. Retry-After를 읽고, 대기 후 동시성을 낮추고, 재시도 횟수를 제한하는 공통 모듈을 먼저 만드세요. 대기 직후 요청이 몰리는 현상도 원문이 피하라고 한 항목입니다.
- **테넌트 관리자:** 컨테이너가 조직 전체 사이트 상한(게시 시점 200만)을 SharePoint 사이트와 나눠 쓴다는 점을 용량 계획에 넣으세요. 컨테이너를 사용자·고객 단위로 마구 만드는 설계는 이 상한과 충돌할 수 있습니다.
- **보안 담당:** 앱 등록 시 요청하는 Graph 권한을 점검하세요. `Files.Read.All`을 쓰는 앱이 있다면 `FileStorageContainer.Selected`로 줄일 수 있는지 검토하고, 가능하면 위임 접근으로 바꾸는 것이 원문 권고입니다.
- **프리뷰 도입 판단:** MCP 서버를 운영 구독에서 에이전트에 연결하기 전에, 에이전트가 프로비저닝할 수 있는 범위를 개발 테넌트에서 먼저 확인하세요. 자연어로 리소스를 만들 수 있다는 것은 잘못된 요청으로도 만들어질 수 있다는 뜻입니다. 이 부분은 제 해석입니다.
- **IT 강사:** 한도 숫자는 "외우는 값"이 아니라 "배포 전 문서에서 다시 확인하는 값"으로 가르치세요. 원문도 한도가 바뀔 수 있다고 반복해서 밝힙니다. 실습에는 개발 테넌트가 필요합니다.

## 참고

- Microsoft 365 Developer Blog, Develop enterprise-grade apps with SharePoint Embedded (2026-10-22): https://devblogs.microsoft.com/microsoft365dev/develop-enterprise-grade-apps-with-sharepoint-embedded/
