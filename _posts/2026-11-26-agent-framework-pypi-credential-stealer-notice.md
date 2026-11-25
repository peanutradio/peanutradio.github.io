---
title: "Agent Framework PyPI 사고 — 1.26.1·1.26.2 설치했다면 재구축"
date: 2026-11-26 06:51:00 +0900
categories: [IT News]
tags: [ai-agent, security, devops]
description: agent-framework Python 1.26.1·1.26.2에 자격증명 탈취 코드가 올라갔다. 영향 범위, 지표 파일, 대응 순서와 운영자가 바꿀 것을 정리한다.
---

> 이 글은 Microsoft Agent Framework 팀의 11월 25일 공지("Microsoft Agent Framework repository update")를 읽고 정리·재구성한 것입니다. 사고 경위나 악성 코드의 동작은 공지에 없어서 추측하지 않았고, 기술 세부는 공지가 가리키는 어드바이저리(PYSEC-2026-4388)에 있습니다. 저는 어드바이저리 본문을 열어 보지 못했습니다.
{: .prompt-info }

## 결론 먼저

- 2026-11-17에 PyPI의 `agent-framework` Python 메타 패키지 **1.26.1과 1.26.2**가 무단 업로드됐고, 둘 다 자격증명을 훔치는 코드를 담고 있었다고 공지는 밝힙니다. 지금은 PyPI에서 제거됐습니다.
- 그 버전을 설치한 적이 있다면 해당 머신·컨테이너·CI 작업을 **침해된 것으로 간주**하고, 접근 가능했던 자격증명을 모두 교체한 뒤 깨끗한 이미지에서 다시 만들어야 합니다. 제자리 다운그레이드는 하지 말라고 공지가 명시합니다.
- 설치한 적이 없다면 `agent-framework==1.26.0`으로 고정하고 추가 안내가 나올 때까지 유지하면 됩니다.

## 공지가 말하는 것

공지에서 확인되는 사실은 다음과 같습니다.

- 영향 대상은 `agent-framework` Python **메타 패키지**의 1.26.1, 1.26.2입니다. 다른 Agent Framework Python 패키지는 영향이 없다고 합니다.
- 악성 코드는 저장소가 아니라 **게시된 패키지** 안에 있었습니다. 1.26.0 이하, .NET 패키지, Semantic Kernel은 영향이 없고, 프로젝트가 중단된 것도 아니라고 합니다.
- 두 버전은 11월 17일에 "짧은 시간" 내려받을 수 있는 상태였습니다.
- 저장소는 안전하다고 확신할 수 있을 때까지 접근 불가이고, 복구 시점은 정해지지 않았습니다.

침해 지표는 site-packages 안의 파일입니다.

| 버전 | 지표 파일 |
|---|---|
| 1.26.1 | `agent_framework_meta.pth`, `agent_framework_meta/_telemetry.py` |
| 1.26.2 | `af_runtime_diag.pth`, `af_runtime_diag.py` |

## 왜 "import 안 해도" 위험한가

공지에서 가장 무게가 실리는 문장은 이것입니다. 코드가 "Python이 시작될 때" 실행되므로 애플리케이션이 패키지를 import하지 않아도 영향을 받는다는 설명입니다. 지표 파일에 `.pth`가 들어 있는 것과 맞물려 읽히는데, `.pth`는 인터프리터 시작 시 site-packages에서 처리되는 파일 형식입니다(이 연결은 일반적인 Python 동작에 근거한 제 해석이며, 공지가 메커니즘을 설명하지는 않습니다).

<figure class="sketch">
<svg viewBox="0 0 720 260" role="img" aria-label="패키지 설치 여부에 따라 대응이 갈리는 판단 흐름">
  <defs>
    <marker id="afp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="afp-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 설치 이력 확인 → 대응 분기</text>
  <rect x="8" y="90" width="190" height="64" rx="8" class="sk-box"/>
  <text x="103" y="116" text-anchor="middle" class="sk-label">1.26.1 / 1.26.2</text>
  <text x="103" y="136" text-anchor="middle" class="sk-sub">설치·빌드 이력이 있나?</text>
  <path d="M200,110 L300,60" class="sk-line-accent" marker-end="url(#afp-a1)"/>
  <path d="M200,134 L300,196" class="sk-line" marker-end="url(#afp-m1)"/>
  <text x="236" y="72" class="sk-sub">있음</text>
  <text x="236" y="190" class="sk-sub">없음</text>
  <rect x="302" y="30" width="410" height="64" rx="8" class="sk-box-accent"/>
  <text x="507" y="56" text-anchor="middle" class="sk-label">침해로 간주</text>
  <text x="507" y="76" text-anchor="middle" class="sk-sub">자격증명 교체 → 깨끗한 이미지로 재구축 → 감사 로그 확인</text>
  <rect x="302" y="166" width="410" height="64" rx="8" class="sk-box"/>
  <text x="507" y="192" text-anchor="middle" class="sk-label">1.26.0으로 고정</text>
  <text x="507" y="212" text-anchor="middle" class="sk-sub">추가 안내 전까지 유지</text>
</svg>
<figcaption>같은 패키지라도 "설치한 적이 있느냐"가 대응의 크기를 갈라 놓는다.</figcaption>
</figure>

## 실무에서 달라지는 점

### 1. 먼저 "설치했는지"를 증거로 확인한다

공지가 꼽은 확인 위치는 lock 파일, `pip freeze` 결과, CI 로그, 컨테이너 이미지, 사내 패키지 미러·아티팩트 프록시, pip 캐시입니다. 버전을 고정하지 않고 설치·빌드하는 CI가 있었다면 11월 17일 전후의 로그부터 봐야 합니다. PyPI에서 지워져도 이미 받은 사본은 남기 때문에, 사내 미러에 사본이 남아 있는지도 따로 확인할 대상입니다.

### 2. 자격증명 교체 범위를 넓게 잡는다

공지는 클라우드 ID, CI/CD·패키지 레지스트리 토큰, API 키, Git 자격증명을 예로 듭니다. Azure·M365 운영자라면 해당 환경에서 쓰던 서비스 주체 비밀, PAT, 배포 토큰이 후보입니다. 교체 후에는 11월 17일 이후의 클라우드·CI 감사 로그에서 낯선 활동이 있는지 봅니다.

### 3. 다운그레이드가 아니라 재구축

`pip install agent-framework==1.26.0`으로 덮어쓰는 것은 대응이 아니라고 공지가 못 박았습니다. 이미지와 캐시, 미러 사본까지 지우고 깨끗한 베이스에서 다시 빌드하는 절차를 팀 런북에 넣어 둘 필요가 있습니다.

### 4. 버전 고정과 해시 검증을 기본값으로

이번 사고에서 영향을 받은 경로는 "고정 없이 설치하거나 자동 업데이트를 병합한 경우"로 설명됩니다. 앱 의존성은 lock 파일로 고정하고, 의존성 자동 업데이트 PR은 병합 전에 사람이 보게 하는 편이 안전합니다. pip의 `--require-hashes`는 해시가 맞지 않는 패키지의 설치를 막는 기능이므로, 고정과 함께 검토해 볼 만합니다(정확한 사용법은 pip 문서 기준으로 확인할 것).

### 5. 교육에서 강조할 점

에이전트 개발 실습에서 `pip install agent-framework`를 안내해 온 강사라면 교안의 설치 명령에 버전을 붙이고, "패키지는 import하지 않아도 시작 시점에 실행될 수 있다"는 점과 "공급망 사고 시 환경 단위로 교체한다"는 순서를 함께 가르치는 것이 좋습니다. 공식 저장소가 잠겨 있어 당분간 소스 확인도 어렵다는 점은 실습 환경 준비 시 알려야 합니다.

## 비교해 읽을 글

패키지·확장 배포 경로에서 장기 토큰을 줄이는 접근은 [VS Marketplace Trusted Publishing](/posts/vs-marketplace-trusted-publishing/)에서, 사내 점검 순서를 잡는 방식은 [Git 2.56.1 보안 패치](/posts/git-2-56-1-six-vulnerabilities/)에서 다뤘습니다. 이번 건은 배포자 쪽 계정 보호가 아니라 **소비자 쪽 고정·검증**이 핵심이라는 점이 다릅니다.

## 아직 모르는 것

- 악성 코드가 구체적으로 무엇을 수집했는지는 공지에 없습니다.
- 무단 업로드가 어떻게 가능했는지, 저장소는 언제 복구되는지도 공지에 없습니다.
- 공지는 평가가 바뀌면 글을 갱신하겠다고 했으니, 위 내용이 바뀔 수 있습니다.

## 참고

- [Microsoft Agent Framework repository update (Dan Taylor, 2026-11-25)](https://devblogs.microsoft.com/agent-framework/microsoft-agent-framework-repository-update/)
- [어드바이저리 PYSEC-2026-4388 (OSV)](https://osv.dev/vulnerability/PYSEC-2026-4388) — 공지가 "전체 기술 세부"를 담았다고 안내한 문서
