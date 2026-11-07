---
title: "Microsoft AI 월간 정리 — 2026년 10월"
date: 2026-11-07 08:53:00 +0900
categories: [IT News]
tags: [copilot, foundry, microsoft-365, copilot-studio, ai-agent]
description: 2026년 10월 Microsoft AI 변화 12건을 상태별로 정리했다. GPT-6.1 Terra GA, Copilot for legal·finance, Work IQ MCP, Fusion과 코드 리뷰 Max까지 다룬다.
---

> 이번 달 Microsoft 공식 업데이트 중 실무에 영향이 큰 것을 골라 공식 문서를 바탕으로 정리했다. 문서에 가격·라이선스·리전이 없는 항목은 "문서에 없음"이라고 적었고, 문서끼리 상태가 다른 항목은 그 사실을 그대로 적었다.
{: .prompt-info }

## 이번 달 한눈에

- **모델**은 GPT-6.1 Terra가 GA가 됐고, Decisions API가 나왔으며, MAI-Image-2.6은 문서마다 상태 표기가 다르다.
- **업무용 Copilot**은 법무·재무 스킬이 들어왔고, Work IQ를 MCP로 읽는 `retrieve`가 프리뷰로 열렸다.
- **에이전트 운영**은 Copilot Studio의 Copilot Chat 에이전트 제작 기능이 10월 26일 GA가 됐고, API Management와 Agent 365 연동이 프리뷰로 나왔다. GitHub 쪽은 Fusion GA와 코드 리뷰 Max가 눈에 띈다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="10월 변화를 모델, 업무용 Copilot, 에이전트 운영, GitHub 네 줄로 나눈 지도">
  <defs>
    <marker id="mo1026-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 2026년 10월 — 어디가 움직였나</text>
  <rect x="8" y="40" width="160" height="80" rx="8" class="sk-box"/>
  <text x="88" y="64" text-anchor="middle" class="sk-label">Foundry 모델</text>
  <text x="88" y="84" text-anchor="middle" class="sk-sub">Terra · Decisions API</text>
  <text x="88" y="100" text-anchor="middle" class="sk-sub">MAI-Image-2.6 · Multi-LoRA</text>
  <rect x="190" y="40" width="160" height="80" rx="8" class="sk-box"/>
  <text x="270" y="64" text-anchor="middle" class="sk-label">M365 Copilot</text>
  <text x="270" y="84" text-anchor="middle" class="sk-sub">legal · finance</text>
  <text x="270" y="100" text-anchor="middle" class="sk-sub">Work IQ MCP retrieve</text>
  <rect x="372" y="40" width="160" height="80" rx="8" class="sk-box-accent"/>
  <text x="452" y="64" text-anchor="middle" class="sk-label">에이전트 운영</text>
  <text x="452" y="84" text-anchor="middle" class="sk-sub">Studio 제작 기능 GA</text>
  <text x="452" y="100" text-anchor="middle" class="sk-sub">APIM × Agent 365</text>
  <rect x="554" y="40" width="158" height="80" rx="8" class="sk-box"/>
  <text x="633" y="64" text-anchor="middle" class="sk-label">GitHub Copilot</text>
  <text x="633" y="84" text-anchor="middle" class="sk-sub">Fusion GA</text>
  <text x="633" y="100" text-anchor="middle" class="sk-sub">코드 리뷰 Max</text>
  <rect x="190" y="170" width="342" height="52" rx="8" class="sk-box-muted"/>
  <text x="361" y="193" text-anchor="middle" class="sk-label">공통 질문: 누가 쓰고, 얼마 나오고, 어디까지 허용하나</text>
  <text x="361" y="211" text-anchor="middle" class="sk-sub">크레딧 과금 · 정책 · 권한</text>
  <path d="M452,122 L452,166" class="sk-line-accent" marker-end="url(#mo1026-m1)"/>
</svg>
<figcaption>이번 달은 새 기능보다 "누가 얼마까지 쓰게 할지"를 정하는 장치가 같이 늘었다.</figcaption>
</figure>

## Foundry·모델

### GPT-6.1 Terra (GA)

OpenAI의 시각 추론·컴퓨터 사용 모델 GPT-6.1 Terra가 10월 26일 Foundry에서 GA가 됐다. Responses API로 쓰고 입력은 텍스트와 이미지, 컨텍스트는 최대 1M 입력 토큰이다. 배포는 Global Standard와 Data Zone Standard(US/EU/APAC)가 있다.

화면 캡처를 읽고 조작하는 에이전트나 대량 이미지 판독을 만드는 팀이 볼 항목이다. Global Standard 기준 100만 토큰당 입력 $1.00, 캐시 입력 $0.05, 출력 $5.00(Short 컨텍스트)이고, Long 컨텍스트는 입력 $2.00, 출력 $7.50으로 안내돼 있다.

Short와 Long을 가르는 기준 길이는 글에 없다. 배포 전에 확인하자. 접근과 쿼터는 리전마다 다르고, East US 2에서는 보이지 않는다는 독자 댓글에 Microsoft가 다른 US 리전 배포를 안내한 기록이 있다.

### Azure OpenAI Decisions API (GA)

텍스트나 이미지를 평가해 생성 문장 대신 타입이 있는 답을 돌려주는 API다. 질문 유형은 `predicate`(참일 확률 0~1), `choice`(고정 선택지), `score`(순서 있는 단계의 확률 가중 평균)이고 엔드포인트는 `/openai/v1/decisions`다. Foundry 10월 소식에는 10월 22일 출시로 적혀 있다.

분류·라우팅·품질 판정처럼 "문장이 아니라 판정값"이 필요한 파이프라인을 만드는 개발자가 대상이다. 후처리 파싱을 줄일 수 있는지 볼 만하다.

현재 지원 모델은 `gpt-6-luna` 하나이고 요청당 질문 최대 400개, 이미지 최대 128장(합계 32MB)이다. East US 2는 롤아웃에서 제외돼 있다. .NET의 `DecisionsClient`는 experimental이며, 가격은 Learn 문서에 수치가 없다.

### MAI-Image-2.6 / 2.6-Flash (프리뷰)

텍스트→이미지 생성과 이미지→이미지 편집 모델이고, Flash는 더 빠르고 저렴한 변형이다. 2.6 전용으로 모델이 종횡비를 고르는 `auto_aspect_ratio`와 Bing 기반 `web_grounding`이 있다.

상태 표기가 문서마다 다르다. microsoft.ai 글(10월 27일)은 Foundry에서 GA라고 쓰면서 "곧 롤아웃"이라는 문구도 함께 있고, Learn 문서와 Foundry 10월 소식은 프리뷰로 표기한다. 나는 프리뷰로 분류했다.

제약은 프롬프트 최대 32,000 토큰, 출력 항상 PNG, 편집용 참조 이미지 최대 5장이다. 가격은 공식 문서에 수치가 없다. 서비스에 넣기 전에 상태부터 포털에서 재확인하자.

### Multi-LoRA 어댑터 배포, Foundry Managed Compute (프리뷰)

하나의 베이스 모델 배포에 Foundry 파인튜닝으로 만든 LoRA 어댑터 여러 개를 얹어 공유 서빙한다. 어댑터마다 컴퓨트를 따로 만들 필요가 없다.

고객·업무별로 어댑터를 따로 학습해 운영하는 팀이 비용을 줄일 수 있는지 볼 항목이다.

타깃 하나에 어댑터 최대 8개이고 어댑터를 추가해도 서빙 용량은 늘지 않는다. 외부에서 학습한 어댑터는 아직 가져올 수 없다. 초기 지원 모델은 Muse-Glimmer-30B, Qwen3.8-27B, Qwen3.6-35B-A3B, GPT-OSS-120B이고, SDK 배포에는 `azure-mgmt-cognitiveservices` 15.0.0b6 프리릴리스가 필요하다고 안내돼 있다.

## M365 Copilot

### Microsoft Copilot for legal (GA)

10월 15일 게시된 안내에 따르면 법률 스킬이 전 세계 Microsoft Copilot 고객에게 GA다. 변경 추적을 켠 레드라인, SharePoint 선례 기반 초안 같은 20개 이상 스킬과 파트너 플러그인(Bloomberg Law, iManage, Docusign 등)이 소개된다.

계약 검토가 잦은 법무·구매 담당자가 대상이다. Word에서는 사용자가 승인한 뒤에만 편집이 적용되고, 접근 가능한 콘텐츠와 민감도 레이블을 따른다고 설명된다.

라이선스·가격·지원 법역·GCC 지원은 문서에 없다. 파트너 플러그인은 플러그인 레지스트리를 통해 순차 제공 중이며 날짜가 없다. 정의 용어 탐색은 프리뷰 예정이다.

### Microsoft Copilot for finance (GA)

10월 29일 안내 기준으로 Excel의 Copilot Finance 스킬과 Cowork의 Capital Markets·Finance 플러그인이 GA다. Accounting 기능(계정 대사, 분개 준비, 계획 대비 성과 보고)은 Frontier 단계이고, Dynamics 365 ERP나 QuickBooks·Xero·FactSet 같은 파트너 플러그인은 레지스트리로 롤아웃을 시작했다.

재무·회계팀과 이를 지원하는 IT가 대상이다. 변경 내용과 이유를 채팅 기록에 남기는 선택적 감사 추적이 있다고 소개된다.

라이선스 등급은 문서에 없고, 서드파티 서비스는 별도 라이선스가 필요할 수 있다고 적혀 있다. Accounting 쪽 일정을 다룬 메시지센터 글은 내가 본문을 열어 확인하지 못했으니 일정은 쓰지 않는다.

### Work IQ MCP — retrieve (프리뷰)

Work IQ MCP 서버의 `retrieve`는 로그인한 사용자가 접근 가능한 메일·파일·회의·Teams 메시지·사람 정보를 검색해, 최종 답변 없이 인용이 달린 그라운딩 콘텐츠를 에이전트에 돌려준다. 엔드포인트는 `https://workiq.svc.cloud.microsoft/mcp`다.

자체 에이전트에 사내 맥락을 붙이고 싶은 개발자가 대상이다. 답을 만드는 `ask`와 달리 근거 조각만 받아 직접 조합하는 구조다.

전제는 테넌트의 Work IQ 활성화, Copilot Credits 사용량 과금과 Work IQ API가 들어간 spending policy, 위임 권한 `WorkIQAgent.Ask`다. Dataverse와 Power BI는 미지원이다. 기본 `strategy` 값이 Learn 문서(`grounding`)와 개발자 블로그(`copilot`)에서 달라서, 쓰기 전에 `tools/list` 스키마로 확인하는 것이 안전하다.

## Copilot Studio·에이전트 거버넌스

### Copilot Chat 에이전트 제작 기능 (GA)

메시지센터 MC1518233에 따르면 10월 26일 GA다. Copilot Chat 에이전트에 결정론적 흐름용 Topics, 작업 위임용 Connected agents, Evaluations, 사용량 Monitoring, Activity Map 미리보기, 개발·테스트·운영 분리를 위한 수명주기 관리가 추가된다.

Copilot Studio 제작자와 테넌트 관리자가 대상이다. 공지에는 필요한 조치가 없다고 적혀 있다.

라이선스는 명시가 없다. 제작자에게 열리는 범위를 거버넌스 정책과 맞춰 보자.

### Customer conversation harness (프리뷰)

음성·채팅 고객 대응 에이전트를 위한 멀티 에이전트 구조로, 청구·계정 같은 전문 하위 에이전트와 사람 상담원 에스컬레이션, 테스트·평가·배포·모니터링이 내장돼 있다고 Copilot Studio 블로그가 소개한다. 같은 글에서 Triggers와 Business process memory도 언급되지만 상태는 문서에 적혀 있지 않았다.

컨택센터형 에이전트를 준비하는 팀이 대상이다. 사용량은 Copilot Credits로 측정되고 사용자·에이전트 단위 한도와 지출 알림을 설정할 수 있다.

프리뷰이므로 에스컬레이션 경로와 평가 기준을 PoC에서 먼저 따져 보자. 한편 레이더에 있던 "Agent ID 지원"은 10월에 새로 나온 항목을 공식 문서에서 찾지 못해 넣지 않았다.

### API Management × Agent 365 연동 (프리뷰)

Agent 365가 API Management 게이트웨이와 그 안의 MCP 서버·도구를 검색하고, 중앙 인벤토리를 관리하며, MCP 서버를 차단·해제한다. 서버를 차단하면 게이트웨이가 HTTP 403(`AccessBlockedByGateway`)을 돌려준다.

MCP 서버를 APIM 뒤에 모아 운영하는 플랫폼 팀이 대상이다. 도구 단위 차단·관측·위협 보호는 AI Gateway 티어(프리뷰)에서만 되고, v2 티어는 서버 단위 제어까지, v1은 미지원이다.

Agent 365는 에이전트를 게이트웨이로 자동 라우팅하지 않는다. 개발자가 MCP 엔드포인트를 직접 지정해야 하고, 우회 접근을 막으려면 네트워크 제어를 같이 두라고 안내한다. 도구 단위 제어에는 허용 도구의 정적 목록이 필요하다.

## GitHub Copilot

### Copilot Fusion (GA)

10월 28일 GA가 됐다. 사용자는 모델 대신 노력 수준(Standard, High, Max)을 고르고, 워크플로는 Single, Cascade, Critique 중 하나로 동작한다. VS Code, Copilot app, Copilot CLI, JetBrains에서 모든 Copilot 플랜에 제공된다.

비용과 정책을 책임지는 관리자가 볼 항목이다. 사용한 각 모델의 표준 요율로 과금되고, 조직의 모델 정책에서 허용된 모델만 쓰인다. 유료 플랜에는 한시적으로 모델 비용 30% 할인이 적용된다고 한다.

Fusion Max가 Claude Code(Opus 5.5 Max)와 2점 이내이면서 작업당 평균 API 비용이 약 절반이라는 수치는 GitHub 자체 보고다. 우리 코드베이스에서 같은 결과가 나온다는 보장은 아니다.

### Copilot 코드 리뷰 Max (프리뷰)

Lite, Balanced에 이은 세 번째 노력 수준으로, 서로 다른 제공사 모델을 쓰는 독립 리뷰 에이전트 여러 개를 돌린 뒤 집계 에이전트가 결과를 합친다. 인증 로직이나 공유 라이브러리 같은 위험한 변경이 대상이다. 함께 발표된 Approvals는 GA가 됐고, 켜면 Copilot의 승인이 필수 승인 수에 포함된다(기본 꺼짐, 새 커밋이 푸시되면 취소).

리뷰 정책을 설계하는 저장소 관리자가 대상이다. Max는 시간이 더 걸리고 AI 크레딧을 더 쓰며 구체 가격은 문서에 없다.

Approvals를 켜기 전에 필수 승인 규칙에서 Copilot을 사람 승인으로 셀지 팀에서 먼저 정하자. 관리자는 최대 노력 수준의 상한을 설정할 수 있다.

## 참고

- [Microsoft Foundry 10월 소식](https://devblogs.microsoft.com/foundry/whats-new-in-microsoft-foundry-october-2026/)
- [GPT-6.1 Terra GA (Tech Community)](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/gpt-6-1-terra-now-generally-available-in-microsoft-foundry-build-agents-that-see/4567769)
- [Azure OpenAI Decisions API (Learn)](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/decisions)
- [MAI-Image 사용 안내 (Learn)](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/how-to/use-foundry-models-mai-image) · [microsoft.ai 발표](https://microsoft.ai/news/mai-image-2-6-launches-at-no-2-on-arena-ahead-of-google-meta-and-xai/)
- [Multi-LoRA on Foundry Managed Compute](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/from-fine-tuning-to-shared-serving-multi-lora-deployment-on-foundry-managed-comp/4565959)
- [Introducing Microsoft Copilot for legal](https://techcommunity.microsoft.com/blog/microsoft-copilot-blog/introducing-microsoft-copilot-for-legal/4564246)
- [Microsoft Copilot for finance](https://www.microsoft.com/en-us/copilot/blog/2026/10/29/microsoft-copilot-for-finance-purpose-built-for-the-way-finance-teams-work/)
- [Work IQ MCP retrieve (Learn)](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/mcp/retrieve) · [개발자 블로그](https://devblogs.microsoft.com/microsoft365dev/work-iq-retrieve-october-2026/)
- [Copilot Chat 에이전트 제작 기능, MC1518233](https://mc.merill.net/message/MC1518233)
- [Copilot Studio 블로그: 고객 대화용 에이전트와 하네스](https://www.microsoft.com/en-us/copilot/blog/copilot-studio/new-and-improved-agents-that-are-persistent-agents-for-customer-conversations-and-improved-maker-experience-in-microsoft-copilot-studio/)
- [API Management와 Agent 365 (Learn)](https://learn.microsoft.com/en-us/azure/api-management/agent-365)
- [GitHub Copilot Fusion](https://github.blog/changelog/2026-10-28-route-and-orchestrate-models-with-github-copilot-fusion)
- [Copilot 코드 리뷰 Max와 Approvals](https://github.blog/changelog/2026-10-28-copilot-code-review-merge-with-confidence-with-max-and-approvals)
