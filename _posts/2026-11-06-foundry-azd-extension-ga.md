---
title: "Foundry용 azd 확장 1.0 GA — 에이전트 배포가 명령어 여섯 줄이 됐다"
date: 2026-11-06 06:53:00 +0900
categories: [IT News]
tags: [foundry, azure, devops, ai-agent]
description: Azure Developer CLI의 microsoft.foundry 확장이 1.0 GA가 됐다. 호스티드 에이전트를 azd로 만들고 로컬 테스트하고 배포하는 흐름과 운영자가 챙길 점을 정리했다.
---

> 이 글은 Azure SDK 블로그의 "Microsoft Foundry extension for Azure Developer CLI GA"(2026-11-05, John Miller) 발표를 읽고 정리·재구성한 것입니다. 직접 실행해 본 기록이 아니라, 원문에 적힌 내용과 그에 대한 제 해석입니다.
{: .prompt-info }

## 결론부터

Azure Developer CLI(`azd`)용 Foundry 확장 번들 `microsoft.foundry`가 1.0이 되면서 정식 출시(GA)됐습니다. 의미는 단순합니다. 호스티드 에이전트의 "만들기 → 로컬 테스트 → 배포 → 평가"를 포털 클릭 대신 `azd` 명령과 `azure.yaml` 한 파일로 반복할 수 있게 됐다는 것입니다. 원문 표현으로는 `azd`가 Azure 리소스 프로비저닝과 코드 배포를 맡고, Foundry 확장이 에이전트 설정과 로컬 테스트를 더합니다.

이번 소식에서 제가 가장 주목한 부분은 기능 목록이 아니라 **에이전트 배포가 코드 저장소 안의 설정 파일이 됐다**는 점입니다. 설정이 파일이 되면 리뷰하고, 되돌리고, CI에서 돌릴 수 있습니다.

## 원문이 말하는 것

요구 사항은 azd 1.37.0 이상과 최신 uv입니다. 원문의 샘플은 Python 3.13 이상을 씁니다(샘플 기준이라고 읽는 편이 안전합니다). 그 밖에 Azure 구독, Foundry 권한, 선택한 리전의 모델 쿼터가 필요합니다.

흐름은 아래 그림과 같습니다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="azd 확장으로 에이전트를 만들고 배포하는 다섯 단계 흐름">
  <defs>
    <marker id="fa-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① azd + Foundry 확장의 작업 흐름</text>
  <rect x="4" y="50" width="124" height="60" rx="8" class="sk-box"/>
  <text x="66" y="76" text-anchor="middle" class="sk-label">init</text>
  <text x="66" y="96" text-anchor="middle" class="sk-mono">azd ai agent init</text>
  <rect x="150" y="50" width="124" height="60" rx="8" class="sk-box"/>
  <text x="212" y="76" text-anchor="middle" class="sk-label">provision</text>
  <text x="212" y="96" text-anchor="middle" class="sk-mono">azd provision</text>
  <rect x="296" y="50" width="124" height="60" rx="8" class="sk-box-accent"/>
  <text x="358" y="76" text-anchor="middle" class="sk-label">로컬 실행</text>
  <text x="358" y="96" text-anchor="middle" class="sk-mono">azd ai agent run</text>
  <rect x="442" y="50" width="124" height="60" rx="8" class="sk-box"/>
  <text x="504" y="76" text-anchor="middle" class="sk-label">배포</text>
  <text x="504" y="96" text-anchor="middle" class="sk-mono">azd deploy</text>
  <rect x="588" y="50" width="124" height="60" rx="8" class="sk-box"/>
  <text x="650" y="76" text-anchor="middle" class="sk-label">평가</text>
  <text x="650" y="96" text-anchor="middle" class="sk-mono">azd ai eval run</text>
  <path d="M130,80 L148,80" class="sk-line" marker-end="url(#fa-m1)"/>
  <path d="M276,80 L294,80" class="sk-line" marker-end="url(#fa-m1)"/>
  <path d="M422,80 L440,80" class="sk-line" marker-end="url(#fa-m1)"/>
  <path d="M568,80 L586,80" class="sk-line" marker-end="url(#fa-m1)"/>
  <text x="358" y="150" text-anchor="middle" class="sk-sub">모든 단계의 설정은 azure.yaml 한 파일에 모인다</text>
  <text x="358" y="170" text-anchor="middle" class="sk-sub">로컬 실행에서도 모델 호출은 Azure의 배포를 쓴다</text>
</svg>
<figcaption>에이전트 배포의 기준점이 포털 화면에서 저장소 안의 azure.yaml로 옮겨왔다.</figcaption>
</figure>

원문이 설명하는 각 단계의 요점은 이렇습니다.

### 시작과 설정

`azd auth login`으로 로그인하고 `azd ext install microsoft.foundry`로 확장을 설치합니다. 빈 폴더에서 `azd ai agent init`을 실행하면 Foundry 프로젝트(새로 만들거나 기존 것)를 고르고 샘플 템플릿을 가져옵니다. 인프라 파일을 직접 쓰지 않고 시작하며, 커스터마이즈가 필요할 때만 Bicep 또는 Terraform으로 꺼내 쓰는(eject) 구조입니다. `azure.yaml`에는 모델 배포(원문 예시는 GlobalStandard, 용량 10), 에이전트 런타임과 진입점, 컨테이너 CPU 0.5·메모리 1Gi 같은 값이 들어갑니다. 이 값들은 원문의 발췌 예시이니 그대로 복사할 값은 아닙니다.

### 로컬 테스트와 배포

`azd ai agent run`은 로컬에서 에이전트를 띄우고 Agent Inspector로 요청·응답을 보여 줍니다. 두 번째 터미널에서 `azd ai agent invoke --local`로 호출합니다. `azd deploy`는 소스를 Foundry에 올려 원격 빌드하며, 로컬 컨테이너 빌드가 필요 없다고 합니다. 배포 전 변경 검토용 `azd deploy --preview`, 로그용 `azd ai agent monitor --follow`, 진단용 `azd ai agent doctor`도 언급됩니다.

### 평가와 CI/CD

`azd ext install azure.ai.evaluations` 뒤 `azd ai eval generate`로 테스트 데이터와 채점 기준을 만들고, `azd ai eval init`, `azd ai eval run start --eval quality`로 실행합니다. 평가 설정이 코드와 함께 저장소에 남는다는 점이 핵심입니다. CI에서는 `azd pipeline config`로 GitHub Actions 워크플로를 만들고, 원문 발췌는 `azd up --no-prompt` 이후 평가를 돌리는 구성을 보여 줍니다.

## 이 소식을 어떻게 읽을까

### 같은 이름의 두 확장을 구분하자

이 글의 GA 대상은 `microsoft.foundry` 번들입니다. 제가 11/4자 Foundry 월간 정리 글([What's new in Microsoft Foundry, October 2026](https://devblogs.microsoft.com/foundry/whats-new-in-microsoft-foundry-october-2026/))에서 확인한 바로는 별도의 agents 확장이 베타(1.0.0-beta.23)로 적혀 있습니다. 이번 GA 글 본문에는 프리뷰·베타 문구가 없어서, 두 확장의 관계는 원문만으로 판단할 수 없습니다. 도입 전 `azd ext list`로 설치된 확장과 버전을 눈으로 확인하는 편이 안전합니다.

### "GA"가 보장하는 범위

GA는 확장 번들의 상태입니다. 그 아래에서 쓰는 모델 배포 유형, 리전별 쿼터, 호스티드 에이전트 기능의 상태는 각각 따로 봐야 합니다. 원문도 옵션과 쿼터가 구독과 리전에 따라 다르고, 프로비저닝된 리소스와 모델 사용에는 Azure 요금이 발생한다고 밝힙니다.

### 앞선 흐름과의 연결

[Azure SDK 10월 릴리스](/posts/azure-sdk-october-2026-release-auth-cosmos-rust/)에서 인증 관련 수정이 이어졌고, [Azure 에이전트 플러그인 7종](/posts/azure-agent-plugins-seven/)에서는 에이전트가 쓰는 도구 쪽을 다뤘습니다. 이번 GA는 반대편, 즉 에이전트 자체를 배포하는 쪽의 표준 도구가 생겼다는 의미로 읽힙니다. 이 연결은 제 해석입니다.

## 실무에서 달라지는 점

### 1. 인프라·플랫폼 운영자: 배포 권한 설계를 먼저

`azd provision`은 리소스를 만들고 `azd deploy`는 에이전트를 올립니다. 개발자 개인 계정으로 돌리게 둘지, 파이프라인용 서비스 주체로 돌릴지 정해야 합니다. 원문이 Foundry 권한 문서를 따로 링크하므로, 역할 부여 범위를 도입 전에 확인하세요. 개발 구독과 운영 구독을 나눈다면 `azure.yaml`의 모델 배포 값(용량 등)이 환경마다 달라질 수 있어 리뷰 대상에 넣는 것이 좋습니다.

### 2. 비용 담당자: 정리 명령을 교육에 포함

새 Foundry 프로젝트에서 `azd down`은 리소스 그룹 전체를 삭제하고, 기존 프로젝트를 재사용한 경우에는 프로젝트와 호스티드 에이전트가 남습니다. 연습 후 정리하지 않으면 요금이 계속 나갈 수 있고, 반대로 잘못 지우면 공유 리소스가 사라질 수 있으니, 어느 쪽인지 확인한 뒤 실행하도록 안내해야 합니다.

### 3. IT 강사: 실습 환경 사양과 사전 점검표

azd 1.37.0 이상, uv, Python 3.13 이상(샘플 기준), 모델 쿼터가 수강생 전원에게 있는지가 실습의 성패를 가릅니다. 강의 전날 `azd version`과 쿼터를 확인하는 사전 점검표를 배포하는 것이 좋겠습니다. 로컬 실행도 모델 호출은 Azure 배포를 쓰므로 오프라인 실습은 안 됩니다.

### 4. 개발 팀: 평가를 배포 게이트로 옮길지 결정

평가 설정이 저장소에 남으므로 "배포 후 평가가 통과해야 머지"를 파이프라인에 넣을 수 있습니다. 다만 원문은 임계값 기반 품질 게이트를 CI/CD 가이드 링크로만 언급합니다. 합격선은 팀이 직접 정해야 하며, 자동 생성된 테스트 데이터와 채점 기준을 사람이 한 번 검토하는 단계를 두기를 권합니다.

### 5. 도입 시기: 시범 프로젝트부터

GA라도 기존 에이전트를 한꺼번에 옮기기보다, 새 에이전트 하나를 이 흐름으로 만들어 보며 권한·쿼터·CI 구성을 확인하는 순서가 현실적입니다. 저는 이 흐름을 직접 실행해 보지 않았으므로, 막히는 지점은 실제로 돌려 본 뒤에야 알 수 있습니다.

## 참고

- [Microsoft Foundry extension for Azure Developer CLI GA — Azure SDK Blog, 2026-11-05](https://devblogs.microsoft.com/azure-sdk/microsoft-foundry-azure-developer-cli-ga/)
- [What's new in Microsoft Foundry, October 2026 — Foundry Blog, 2026-11-04](https://devblogs.microsoft.com/foundry/whats-new-in-microsoft-foundry-october-2026/)
- [Hosted agents 개념 — Microsoft Learn](https://learn.microsoft.com/azure/foundry/agents/concepts/hosted-agents)
- [Foundry 권한 — Microsoft Learn](https://learn.microsoft.com/azure/foundry/agents/concepts/hosted-agent-permissions)
- [CI/CD 설정 가이드 — Microsoft Learn](https://learn.microsoft.com/azure/foundry/agents/how-to/set-up-ci-cd-cli)
- [Hosted agent 퀵스타트(azd) — Microsoft Learn](https://learn.microsoft.com/azure/foundry/agents/quickstarts/quickstart-hosted-agent?pivots=azd)
