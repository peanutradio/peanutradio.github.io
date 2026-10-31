---
title: "Azure SDK 10월 릴리스 — 인증 수정 3건과 Cosmos DB Rust 1.0"
date: 2026-11-01 06:53:00 +0900
categories: [IT News]
tags: [azure, identity, security, devops]
description: Azure SDK 2026년 10월 릴리스에서 업그레이드 전에 챙길 것만 추렸다. Cosmos DB Rust 1.0 이름 변경, 인증 라이브러리 수정, AI 관련 베타 패키지를 정리한다.
---

> 이 글은 Azure SDK 블로그의 "Azure SDK Release (October 2026)"(2026-10-28, Sandeep Sen)를 읽고 정리·재구성한 것입니다. 패키지 목록은 원문 그대로 옮기지 않고 운영 관점에서 골랐습니다.
{: .prompt-info }

## 결론부터

이번 릴리스에서 코드나 설정을 건드려야 하는 항목은 두 가지입니다. 하나는 Cosmos DB for Rust가 프리뷰에서 1.0.0 안정 버전으로 올라가며 생기는 이름 변경이고, 다른 하나는 .NET의 실험적 managed identity 옵션 이름 변경입니다. 나머지 인증 관련 항목은 "올리면 좋아지는" 버그 수정에 가깝습니다. 그 가운데 Python 공유 토큰 캐시 수정은 보안 성격이라 우선순위를 높게 잡을 만합니다.

AI 관련 신규 베타 패키지도 몇 개 나왔지만, 원문은 이름과 버전만 나열할 뿐 용도를 설명하지 않습니다. 그래서 이 글에서는 "무엇이 나왔는지"까지만 적고 기능은 추측하지 않습니다.

## 원문에서 확인된 내용

### Cosmos DB for Rust 1.0.0

첫 안정 버전입니다. Cosmos DB Client Driver, SDK Macros 크레이트도 같이 1.0.0이 됐습니다. 프리뷰에서 올릴 때 원문이 안내한 변경은 다음과 같습니다.

- 기능 플래그 `preview_dtx`, `preview_patch`, `preview_opentelemetry`를 `unstable_` 접두사가 붙은 이름으로 바꿉니다.
- `CosmosCredential::MasterKey`를 `CosmosCredential::AccountKey`로 바꿉니다.
- `User-Agent` 형식이 바뀝니다.
- 특정 최종 서비스 오류가 합성 503 오류로 매핑되고, 원래 오류는 `source()`에 남습니다.

### 인증 라이브러리 3건

- Azure Core 1.63.0(.NET): 클라이언트 인증서 인증에 mTLS proof-of-possession을 지원합니다. IMDS를 쓸 수 없을 때 `DefaultAzureCredential`이 지연되던 문제와, 쓸 수 없는 managed identity 소스가 자격 증명 체인을 일찍 끊던 문제를 고쳤습니다. 실험적 PoP 옵션을 쓰고 있다면 `DisableMtlsProofOfPossession = true`를 `EnableMtlsProofOfPossession = false`로 바꿉니다. 의미는 같고(베어러 인증 강제) 값의 극성이 반대입니다.
- Identity 1.26.0(Python): Azure Arc 서버의 사용자 할당 관리 ID를 클라이언트 ID·개체 ID·리소스 ID로 지정할 수 있습니다. 요청한 ID를 확인해 주지 않는 토큰 응답은 거부합니다. 또 `AuthorizationCodeCredential`과 비동기 `OnBehalfOfCredential`이 공유 토큰 캐시에서 다른 계정의 토큰을 돌려줄 수 있던 문제를 고쳤습니다.
- Identity 1.18.7(Java): `ManagedIdentityCredential`과 `DefaultAzureCredential`에 Arc 사용자 할당 ID 지원이 들어왔고, 네이티브 이미지 호환성과 IMDS 탐색 타임아웃을 고쳤습니다.

### AI 관련 신규 베타

Azure Data AI(.NET 1.0.0-beta.1, Python 1.0.0b1), AI Agent Server – Optimization(.NET 1.0.0-beta.1), Azure AI Fine-Tuning Sessions(Python 1.0.0b1), Azure AI Content Safety(JavaScript 1.0.0-beta.1)입니다. 안정 버전이 아니므로 API가 바뀔 수 있다는 전제로 봐야 합니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="이번 릴리스의 항목을 코드 수정 필요, 업그레이드 권장, 관찰 대상 세 묶음으로 나눈 그림">
  <defs>
    <marker id="az-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 10월 릴리스를 세 묶음으로 읽기</text>
  <rect x="8" y="40" width="220" height="150" rx="8" class="sk-box-accent"/>
  <text x="118" y="64" text-anchor="middle" class="sk-label">코드 수정 필요</text>
  <text x="118" y="94" text-anchor="middle" class="sk-mono">MasterKey → AccountKey</text>
  <text x="118" y="114" text-anchor="middle" class="sk-mono">preview_* → unstable_*</text>
  <text x="118" y="134" text-anchor="middle" class="sk-mono">DisableMtls… → EnableMtls…</text>
  <text x="118" y="164" text-anchor="middle" class="sk-sub">안 바꾸면 빌드·동작 영향</text>
  <rect x="250" y="40" width="220" height="150" rx="8" class="sk-box"/>
  <text x="360" y="64" text-anchor="middle" class="sk-label">업그레이드 권장</text>
  <text x="360" y="94" text-anchor="middle" class="sk-sub">Python 공유 토큰 캐시 수정</text>
  <text x="360" y="114" text-anchor="middle" class="sk-sub">IMDS 불가 시 지연 수정</text>
  <text x="360" y="134" text-anchor="middle" class="sk-sub">Arc 사용자 할당 ID 지원</text>
  <text x="360" y="164" text-anchor="middle" class="sk-sub">올리면 좋아지는 수정</text>
  <rect x="492" y="40" width="220" height="150" rx="8" class="sk-box-muted"/>
  <text x="602" y="64" text-anchor="middle" class="sk-label">관찰 대상</text>
  <text x="602" y="94" text-anchor="middle" class="sk-sub">Data AI · Fine-Tuning Sessions</text>
  <text x="602" y="114" text-anchor="middle" class="sk-sub">Agent Server – Optimization</text>
  <text x="602" y="134" text-anchor="middle" class="sk-sub">Content Safety (JS)</text>
  <text x="602" y="164" text-anchor="middle" class="sk-sub">베타라 운영 도입은 보류</text>
  <path d="M230,115 L248,115" class="sk-line" marker-end="url(#az-m1)"/>
  <path d="M472,115 L490,115" class="sk-line" marker-end="url(#az-m1)"/>
</svg>
<figcaption>월간 릴리스는 전부 읽지 말고, 코드를 바꿔야 하는 것부터 가려내면 된다.</figcaption>
</figure>

## 실무에서 달라지는 점

### 1. Rust로 Cosmos DB를 쓰는 팀: 업그레이드 전에 검색부터

프리뷰 크레이트를 쓰고 있다면 `Cargo.toml`의 기능 플래그와 소스의 `CosmosCredential::MasterKey` 사용처를 먼저 검색해 두는 편이 안전합니다. 이름만 바꾸면 되는 변경이라 작업량은 작지만, 안 바꾸면 컴파일 단계에서 걸립니다. 더 조심할 곳은 오류 처리입니다. 일부 오류가 합성 503으로 보이게 되므로, 503을 재시도 대상으로 취급하는 코드가 있다면 어떤 오류가 거기에 들어오는지 릴리스 노트로 확인해야 합니다. 원문에는 어떤 오류가 매핑되는지까지는 나오지 않습니다.

### 2. 사내 .NET 서비스 운영자: 실험적 PoP 옵션 사용 여부 확인

`DisableMtlsProofOfPossession`을 설정에 넣어 둔 서비스가 있는지 먼저 찾아보세요. 있다면 이름과 값이 같이 뒤집힙니다. 실험적 옵션이라 쓰는 곳이 많지는 않겠지만, 값만 보고 `true`를 그대로 두면 의도와 반대가 될 수 있어서 코드 리뷰에서 한 번 더 보라고 권하고 싶습니다. 같은 릴리스에서 `DefaultAzureCredential` 지연이 고쳐졌으니, 온프레미스·개발 PC처럼 IMDS가 없는 환경에서 로그인 시간이 길었던 앱은 업그레이드 효과를 비교하기 좋은 후보입니다.

### 3. Python으로 OAuth 흐름을 쓰는 곳: 보안 수정이라 먼저 올릴 후보

`AuthorizationCodeCredential`이나 비동기 `OnBehalfOfCredential`을 쓰는 다중 사용자 웹앱·API라면 1.26.0 업그레이드 우선순위를 높게 봐야 합니다. 원문 표현은 "다른 계정의 캐시 토큰을 돌려줄 수 있었다"입니다. 실제로 어떤 조건에서 재현되는지는 원문에 없으므로, 영향 범위는 릴리스 노트와 이슈를 확인해서 판단해야 합니다. 에이전트가 사용자 대신 호출하는 구조(On-Behalf-Of)가 늘어나는 만큼, 비슷한 경계 문제는 [Agent 365와 Agent ID 정리](/posts/agent-365-agent-id/)에서 다룬 "누구의 권한으로 실행되는가"와 같은 맥락입니다.

### 4. Arc 서버를 운영하는 인프라 담당: 코드 없이 관리 ID로 인증

Python 1.26.0과 Java 1.18.7로 Arc 서버의 사용자 할당 관리 ID를 SDK에서 지정할 수 있게 됐습니다. 서버에 비밀값을 두던 방식을 줄이고 싶은 팀에게 의미가 있습니다. 다만 두 언어의 지원 범위(예: `DefaultAzureCredential` 포함 여부)가 완전히 같다고 단정할 수는 없으므로 사용 중인 언어의 릴리스 노트를 확인하세요.

### 5. 강의·교육: 베타 패키지는 "이름이 나왔다"까지만 말하기

AI Agent Server – Optimization 같은 신규 베타를 수업에서 소개할 때는 기능을 추정해서 설명하지 않는 편이 좋습니다. 같은 주에 정리한 [Copilot 하네스가 Foundry Agent Service에 들어온 소식](/posts/copilot-harness-foundry-agent-service-preview/)의 최적화 기능과 연결될지 궁금해지지만, 이번 원문에는 그 연결을 보여 주는 내용이 없습니다. 수강생에게는 "릴리스 노트와 패키지 문서에서 확인하기 전까지는 관계를 모른다"고 말하는 것이 정직합니다.

## 정리

월간 릴리스 글은 길지만 실제 조치 항목은 적습니다. 이번 달은 Rust·.NET의 이름 변경 3개를 찾아 바꾸고, Python OAuth 흐름을 쓰는 곳은 1.26.0 업그레이드 일정을 잡는 것으로 충분합니다. 신규 베타는 지켜보되 도입 결정은 안정 버전 이후로 미루는 쪽이 무난합니다.

## 참고

- [Azure SDK Release (October 2026) — Azure SDK Blog](https://devblogs.microsoft.com/azure-sdk/azure-sdk-release-october-2026/)
- [Azure SDK 2026년 10월 릴리스 노트(전체 언어)](https://azure.github.io/azure-sdk/releases/2026-10/index.html)
