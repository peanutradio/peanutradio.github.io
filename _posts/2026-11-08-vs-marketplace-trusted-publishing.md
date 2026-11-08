---
title: "VS Marketplace Trusted Publishing — 확장 배포에서 PAT를 치운다"
date: 2026-11-08 16:22:00 +0900
categories: [IT News]
tags: [security, identity, github, devops]
description: Visual Studio Marketplace 확장을 GitHub Actions·Azure Pipelines에서 PAT 없이 OIDC로 게시할 수 있게 됐다. 달라지는 점과 전환 순서를 정리한다.
---

> 이 글은 Microsoft for Developers 블로그의 "Publish Visual Studio Marketplace extensions securely with Trusted Publishing"(2026-10-28) 발표를 읽고 정리·재구성한 것입니다.
{: .prompt-info }

## 결론부터

VS Code·Visual Studio 확장을 Marketplace에 올리는 CI/CD에서, 이제 Azure DevOps 개인 액세스 토큰(PAT)을 비밀값으로 들고 있을 필요가 없다. GitHub Actions와 Azure Pipelines가 자기 신원(OIDC / 워크로드 ID 페더레이션)을 Marketplace에 제시하면, Marketplace가 그 신원을 검증하고 게시 권한을 내준다. 기존 PAT 방식도 계속 동작하므로 강제 전환은 아니다. 다만 "만료일마다 갱신하고, 누가 어디에 복사해 뒀는지 모르는 토큰"을 정리할 수 있는 길이 열렸다는 점이 핵심이다.

## 발표 내용 요약

원문이 밝힌 사실은 다음과 같다.

- 장기 비밀값이 없다. CI/CD 시스템에 저장하거나 돌려야 할 게시용 자격 증명이 사라진다.
- GitHub Actions에서는 Marketplace가 임시 자격 증명을 발급하며, 15분 뒤 만료된다.
- 지원 CI/CD는 두 가지다. GitHub Actions는 게시자(publisher)의 Trusted Publishing 페이지에서 정책을 만들고 GitHub OIDC로 게시한다. Azure Pipelines는 Microsoft Entra 기반 WIF(Workload Identity Federation) 서비스 연결을 만들고, 그 ID를 게시자의 Contributor로 추가한다. 이쪽은 Marketplace 정책을 만들지 않는다.
- GitHub 정책 필드는 정책 이름, 저장소 소유자, 저장소 이름, 워크플로 파일명(`.github/workflows` 아래), 선택 항목인 환경(environment) 이름이다.
- 비공개 저장소는 정책 생성 후 48시간 안에 워크플로를 한 번 성공시켜야 하며, 첫 토큰 교환이 성공하면 정책이 영구 활성화된다. 쓰지 않으면 만료되고 다시 열 수 있다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="PAT 방식과 Trusted Publishing 방식의 비교 다이어그램">
  <defs>
    <marker id="tp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="tp-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 비밀값을 들고 가나, 신원을 보여주나</text>

  <text x="8" y="50" class="sk-sub">기존: PAT 방식</text>
  <rect x="8" y="60" width="150" height="52" rx="8" class="sk-box-muted"/>
  <text x="83" y="82" text-anchor="middle" class="sk-label">CI 시크릿 저장소</text>
  <text x="83" y="100" text-anchor="middle" class="sk-sub">장기 PAT 보관</text>
  <path d="M160,86 L270,86" class="sk-line" marker-end="url(#tp-m1)"/>
  <rect x="272" y="60" width="150" height="52" rx="8" class="sk-box"/>
  <text x="347" y="82" text-anchor="middle" class="sk-label">게시 단계</text>
  <text x="347" y="100" text-anchor="middle" class="sk-sub">토큰을 들고 호출</text>
  <path d="M424,86 L534,86" class="sk-line" marker-end="url(#tp-m1)"/>
  <rect x="536" y="60" width="170" height="52" rx="8" class="sk-box"/>
  <text x="621" y="82" text-anchor="middle" class="sk-label">Marketplace</text>
  <text x="621" y="100" text-anchor="middle" class="sk-sub">토큰만 맞으면 통과</text>

  <text x="8" y="150" class="sk-sub">신규: Trusted Publishing</text>
  <rect x="8" y="160" width="150" height="52" rx="8" class="sk-box"/>
  <text x="83" y="182" text-anchor="middle" class="sk-label">워크플로 신원</text>
  <text x="83" y="200" text-anchor="middle" class="sk-sub">OIDC / WIF</text>
  <path d="M160,186 L270,186" class="sk-line-accent" marker-end="url(#tp-a2)"/>
  <rect x="272" y="160" width="150" height="52" rx="8" class="sk-box-accent"/>
  <text x="347" y="182" text-anchor="middle" class="sk-label">정책·ID 검증</text>
  <text x="347" y="200" text-anchor="middle" class="sk-sub">저장소·워크플로 일치?</text>
  <path d="M424,186 L534,186" class="sk-line-accent" marker-end="url(#tp-a2)"/>
  <rect x="536" y="160" width="170" height="52" rx="8" class="sk-box-accent"/>
  <text x="621" y="182" text-anchor="middle" class="sk-label">15분짜리 자격 증명</text>
  <text x="621" y="200" text-anchor="middle" class="sk-sub">GitHub Actions 기준</text>
</svg>
<figcaption>보관할 비밀값이 사라지면, 유출되어도 쓸 수 있는 시간이 15분으로 줄어든다.</figcaption>
</figure>

## 이 소식을 어떻게 읽을까

### 문제는 토큰 자체보다 '수명'이다

PAT는 만료일을 정할 수는 있지만, 현실에서는 편의상 길게 잡고 잊어버리기 쉽다. 그 사이 로그 출력, 포크한 저장소의 워크플로, 퇴사자의 개인 계정 같은 경로로 새면 공격자는 게시자 권한으로 악성 업데이트를 올릴 수 있다. 확장은 개발자 PC에서 자동 업데이트로 실행되므로, 게시 권한 탈취는 곧 공급망 사고다. Trusted Publishing은 "누가 게시하는가"를 토큰 소지가 아니라 "어느 저장소의 어느 워크플로인가"로 바꾼다. 이 변화는 npm·PyPI 등 다른 레지스트리가 먼저 걸어온 방향과 같은 흐름으로 보인다. 다만 다른 레지스트리와의 세부 비교는 원문에 없으므로 여기서는 방향성만 언급한다.

### 환경(environment) 필드가 사실상 승인 관문이 된다

정책에 GitHub 환경 이름을 넣으면, 해당 환경의 보호 규칙(수동 승인자 지정 등)을 통과한 실행만 게시할 수 있도록 구성할 수 있다. 원문은 이 필드를 "선택 항목"으로만 소개하지만, 운영 관점에서는 `production` 같은 환경을 지정해 승인 절차와 묶는 편이 안전하다. 이 부분은 원문에 명시된 효과가 아니라 GitHub 환경 기능을 결합한 필자의 해석이다.

### 한계도 있다

원문 기준으로 지원 CI/CD는 GitHub Actions와 Azure Pipelines 둘뿐이다. 그 밖의 CI(Jenkins 등 자체 구축 환경)는 이 방식의 대상으로 소개되지 않았으므로 PAT 방식을 유지해야 한다. 또 정책이 가리키는 저장소·워크플로 파일명이 바뀌면 정책과 어긋나 게시가 실패할 수 있다고 추정되므로, 파일명 변경 시 정책도 함께 손봐야 한다(추정이며 원문에는 없다).

## 실무에서 달라지는 점

### 1. 확장을 직접 배포하는 팀: PAT 전환 순서

원문이 권하는 순서는 정책 생성 → 게시 작업에 `id-token: write` 추가 → 게시 명령을 OIDC로 변경 → 워크플로 실행해 성공 확인 → 그 뒤에 시크릿 삭제다. 첫 OIDC 게시가 성공하기 전에는 PAT를 지우지 않는 것이 포인트다. 롤백 수단을 남겨두라는 뜻이다.

### 2. Azure DevOps 운영자: WIF 서비스 연결과 Contributor 부여

Azure Pipelines 사용자는 Marketplace 쪽 정책이 아니라 Entra 앱 등록(또는 관리 ID)에 기반한 WIF 서비스 연결을 만든다. 그 ID를 Marketplace 게시자에 Contributor로 추가해야 한다. 정책 페이지에 만들지 말라는 경고가 원문에 따로 있으니, GitHub 방식과 헷갈리지 않게 체크리스트를 분리해 두자.

### 3. 보안 담당자: 시크릿 인벤토리에서 Marketplace PAT 지우기

사내 확장을 배포한다면 저장소·조직 시크릿 목록에서 Marketplace용 PAT를 찾아 전환 대상 목록에 올린다. 전환 후에도 남은 PAT는 다른 워크플로 단계가 실수로 쓸 수 있다고 원문이 경고한다. 완료 기준을 "OIDC 게시 성공"이 아니라 "시크릿 삭제 확인"까지로 잡는 것을 권한다.

### 4. 비공개 저장소: 48시간 규칙

비공개 저장소에서 정책을 만들었다면 이틀 안에 해당 워크플로를 한 번 실행해야 한다. 금요일 오후에 정책만 만들어 두고 주말을 넘기면 만료될 수 있다. 만료되어도 다시 열 수 있으니 당황할 일은 아니다.

### 5. 교육할 때: "비밀을 잘 숨기기"에서 "비밀을 없애기"로

CI/CD 보안 강의에서 시크릿 관리 설명은 보통 저장 위치와 마스킹에 머문다. 이번 사례는 OIDC 페더레이션이라는 대안을 보여주는 좋은 실습 소재다. 신원 증명 → 정책 대조 → 단기 자격 증명이라는 세 단계를 그림으로 그려 주면 이해가 빠르다. 이 글은 소개·정리이며, 필자가 직접 전환해 본 기록이 아니라는 점은 밝혀 둔다.

## 도입 전 점검 한 줄 요약

- 게시 워크플로가 GitHub Actions 또는 Azure Pipelines인가
- 정책에 넣을 저장소 소유자·이름·워크플로 파일명이 확정되었는가
- 환경 보호 규칙(승인자)을 함께 쓸 것인가
- PAT 삭제 시점과 담당자를 정했는가

## 참고

- [Publish Visual Studio Marketplace extensions securely with Trusted Publishing — Microsoft for Developers](https://devblogs.microsoft.com/blog/trusted-publishing-visual-studio-marketplace/)
- [Secure automated publishing to Visual Studio Marketplace — VS Code 문서](https://code.visualstudio.com/api/working-with-extensions/publishing-extension?from=20423#secure-automated-publishing-to-visual-studio-marketplace)
- [Publish a Visual Studio extension via command line — Microsoft Learn](https://learn.microsoft.com/visualstudio/extensibility/walkthrough-publishing-a-visual-studio-extension-via-command-line?view=visualstudio)
- [Microsoft Entra 기반 서비스 연결 만들기 — Microsoft Learn](https://learn.microsoft.com/azure/devops/pipelines/library/add-devops-entra-service-connection?view=azure-devops#step-2-create-the-service-connection)
- [게시자 계정에 사용자 추가 — Microsoft Learn](https://learn.microsoft.com/visualstudio/extensibility/walkthrough-publishing-a-visual-studio-extension?view=visualstudio#add-additional-users-to-manage-your-publisher-account)
