---
title: "벡터 DB 없이 RAG — SharePoint Embedded Retrieval API와 spe-mcp"
date: 2026-10-14 06:53:00 +0900
categories: [AI Agent]
tags: [rag, microsoft-365, mcp, copilot]
description: 임베딩·벡터 저장소·권한 동기화를 직접 만들지 않고 Retrieval API로 권한이 반영된 근거 데이터를 가져오는 방식과, 설정을 맡기는 spe-mcp 서버 정리.
---

> 이 글은 Microsoft 365 개발자 블로그의 "Build AI-powered apps without creating a vector database"(2026-10-13)를 읽고 정리·재구성한 것입니다. 직접 실습한 기록이 아닙니다.
{: .prompt-info }

## 결론부터

사내 문서를 근거로 답하는 앱을 만들 때 가장 귀찮은 건 모델 호출이 아니라 그 앞단입니다. 텍스트 추출, 임베딩, 벡터 저장소, 그리고 "이 사용자가 이 파일을 봐도 되는가"의 권한 동기화까지요. 원문은 이 앞단을 **SharePoint Embedded 컨테이너 + Microsoft 365 Copilot Retrieval API**로 대체하자고 말합니다. 앱은 질의를 보내고, 사용자 권한에 맞게 걸러진 텍스트 발췌를 받아, 자기 모델에 넘기면 됩니다. 프롬프트·오케스트레이션·모델은 계속 내 몫입니다.

여기에 설정 작업을 코딩 에이전트에게 맡기는 오픈소스 MCP 서버 `@microsoft/spe-mcp`(퍼블릭 프리뷰)가 함께 나왔습니다.

## 원문이 말하는 구조

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="질의가 Retrieval API를 거쳐 권한이 반영된 발췌로 돌아오고 앱의 모델에 전달되는 흐름">
  <defs>
    <marker id="spe-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 직접 만들던 앞단을 Retrieval API가 대신한다</text>
  <rect x="8" y="60" width="140" height="56" rx="8" class="sk-box"/>
  <text x="78" y="84" text-anchor="middle" class="sk-label">내 앱</text>
  <text x="78" y="102" text-anchor="middle" class="sk-sub">질의 + 사용자</text>
  <rect x="200" y="60" width="200" height="56" rx="8" class="sk-box-accent"/>
  <text x="300" y="84" text-anchor="middle" class="sk-label">Retrieval API</text>
  <text x="300" y="102" text-anchor="middle" class="sk-mono">sharePointEmbedded</text>
  <rect x="452" y="60" width="140" height="56" rx="8" class="sk-box"/>
  <text x="522" y="84" text-anchor="middle" class="sk-label">내 모델</text>
  <text x="522" y="102" text-anchor="middle" class="sk-sub">발췌를 근거로 답</text>
  <rect x="200" y="140" width="200" height="40" rx="8" class="sk-box-muted"/>
  <text x="300" y="165" text-anchor="middle" class="sk-sub">추출·임베딩·벡터DB·권한 동기화</text>
  <path d="M150,88 L198,88" class="sk-line" marker-end="url(#spe-m1)"/>
  <path d="M402,88 L450,88" class="sk-line-accent" marker-end="url(#spe-m1)"/>
</svg>
<figcaption>점선 박스 안의 일은 내가 짓지 않고, 모델과 프롬프트만 내가 정한다.</figcaption>
</figure>

- 파일은 고객 테넌트의 SharePoint Embedded 컨테이너에 있고, 결과는 사용자 권한에 맞춰 걸러집니다.
- 테넌트에 Microsoft 365 Copilot 라이선스 사용자가 최소 1명 있어야 하고, 그 사용자가 시맨틱 인덱스를 초기화합니다. 질의하는 다른 사용자는 Copilot 라이선스가 필요 없다고 합니다.
- 원문은 SharePoint Embedded 앱용 Retrieval API가 GA라고 쓰고, MCP 서버는 퍼블릭 프리뷰라고 구분합니다.
- 과금은 Azure 구독 종량제이고, Retrieval API는 Copilot Studio 메시지 미터로 측정됩니다. 콘텐츠는 Purview(감사, eDiscovery, 보존, DLP, 민감도 레이블, 조건부 액세스) 통제를 그대로 따릅니다.
- 대안으로 Foundry Agent Service의 SharePoint 지식 소스가 있습니다. 검색을 Foundry가 처리하는 쪽입니다. Foundry 에이전트에서 이 API를 직접 부르는 것도 가능하다고 합니다.

## spe-mcp는 무엇을 맡기나

`npx @microsoft/spe-mcp`로 띄우면 GitHub Copilot(VS Code·CLI), Claude, Cursor, Azure AI Foundry 같은 MCP 클라이언트에서 컨테이너 타입·컨테이너 생성, 앱 프로비저닝, 과금 설정, 콘텐츠 업로드·관리, 참조 앱 스캐폴딩, 문서 검색을 자연어로 시킬 수 있습니다. 원문은 포털 클릭과 Graph·Azure CLI 수작업을 대체한다고 설명합니다. 절차는 "서버 실행 → 업로드 샘플 앱 생성 요청 → 앱이 돌면 Retrieval API로 기능 추가"입니다.

## 내 해석: 이 구조의 득과 실

MCP를 "API 대체"가 아니라 "운영 작업을 에이전트에 열어 주는 통로"로 보면 ([MCP와 API](/posts/mcp-vs-api/)) 이 사례는 잘 맞습니다. 다만 편해지는 만큼 따져볼 것도 있습니다.

- **얻는 것**: 권한 동기화 파이프라인이 사라집니다. RAG 사고의 흔한 원인이 "원본 권한은 바뀌었는데 인덱스는 그대로"인데, 그 지점을 플랫폼에 넘깁니다.
- **내주는 것**: 청킹·임베딩 모델·재순위 같은 검색 튜닝 손잡이입니다. 원문에는 검색 품질 수치도 없습니다.
- **원문에 없는 것**: 쿼터, 속도 제한, 파일 크기·형식 제한, 단가, 지역별 제공 여부. 도입 판단에 필요한데 글만으로는 알 수 없습니다.

## 실무에서 달라지는 점

1. **M365 관리자**: Copilot 라이선스 사용자 1명이 선행 조건입니다. 시맨틱 인덱스를 초기화할 계정을 누가 맡을지(퇴사·라이선스 회수 시 영향 포함)를 먼저 정해야 합니다. 이 계정 변동이 인덱스에 미치는 영향은 원문에 없으니 문서로 확인이 필요합니다.
2. **Azure 구독 담당자**: 비용이 Azure 구독의 종량제로 청구되고 Retrieval API는 Copilot Studio 메시지 미터를 씁니다. 기존 Copilot Studio 비용과 같은 미터에 섞이는지, 앱 개발 테넌트와 고객 테넌트 중 누가 내는지(표준 vs 패스스루 과금)를 도입 전에 정리하세요. 모델 쪽 비용 절감은 [Azure OpenAI 비용 가이드](/posts/azure-openai-cost-guide/)와 별개로 봐야 합니다.
3. **보안·컴플라이언스**: 기존 Purview 정책(DLP, 민감도 레이블, 보존)이 이 앱에도 적용된다는 점이 장점입니다. 다만 모델로 넘어간 발췌는 내 앱의 영역이니, 로그·캐시에 남는 발췌의 보관 정책은 따로 정해야 합니다.
4. **개발팀**: spe-mcp는 프리뷰이므로 운영 환경 프로비저닝보다 개발·검증 테넌트의 스캐폴딩 용도로 한정하는 편이 안전합니다. 에이전트가 만든 컨테이너 타입과 과금 설정은 사람이 한 번 검토하세요.
5. **강의·교육 시**: "벡터 DB가 필요 없다"가 아니라 "이 유스케이스에서는 직접 안 만들어도 된다"로 가르치는 편이 정확합니다. 권한이 다른 두 사용자가 같은 질문을 했을 때 응답이 달라지는 시나리오가 핵심 시연 포인트입니다. 단, 실습 환경에 Copilot 라이선스 사용자가 필요해 수강생용 테넌트 준비가 걸림돌입니다.

Foundry로 에이전트를 먼저 만들어 보려면 [Foundry Agent Service로 첫 에이전트 만들기](/posts/foundry-first-agent/)를, MCP 서버를 에이전트에 붙이는 쪽은 [Copilot Studio에 MCP 서버 붙이기](/posts/copilot-studio-mcp-tools/)를 이어서 읽어 보세요.

## 영상으로 보기

기초부터 짚고 싶다면 이 영상을 먼저 보세요. (AI 에이전트 기초 EP.03 — LLM은 어떻게 말을 할까)

{% include embed/youtube.html id='Ay9-NSw1wYI' %}

> 📺 [Peanut Radio AI 유튜브 채널](https://www.youtube.com/@PeanutRadioAI) — AI 시대의 IT, 기초부터 실무까지
{: .prompt-tip }

## 참고

- [Build AI-powered apps without creating a vector database — Microsoft 365 Developer Blog (2026-10-13)](https://devblogs.microsoft.com/microsoft365dev/build-ai-powered-apps-without-creating-a-vector-database/)
- [SharePoint Embedded MCP Server (GitHub, 원문에서 언급)](https://github.com/microsoft/SharePoint-Embedded-MCP-Server)
