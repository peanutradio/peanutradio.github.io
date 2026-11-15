---
title: "Git 2.56.1 보안 패치 — 취약점 6건, 사내 점검 순서"
date: 2026-11-16 06:51:00 +0900
categories: [IT News]
tags: [security, github, devops]
description: Git이 취약점 6건을 고친 2.56.1과 시리즈별 백포트를 내놨다. GHES 패치 버전, 번들 Git 점검, 못 올릴 때의 임시 대응을 정리했다.
---

> 이 글은 GitHub 블로그의 "Git security vulnerabilities announced" 발표(2026-11-10)를 읽고 정리·재구성한 것이다. 원문에 없는 심각도·CVSS 점수는 쓰지 않았고, 아래 해석과 점검 순서는 이 블로그의 의견이다.
{: .prompt-info }

## 결론부터

- Git이 취약점 6건을 고친 2.56.1을 냈고, 2.55.1·2.54.1·2.53.1·2.52.1·2.51.3·2.50.2로 백포트도 나왔다.
- GitHub.com은 이미 패치됐고, GHES는 3.19.15·3.20.11·3.21.9·3.22.4로 올려야 한다.
- 사내에서 손이 가는 곳은 개발자 PC의 Git보다 **Git을 번들한 도구와 자동화 서버**다. 못 올리는 곳은 "신뢰할 수 없는 저장소를 건드리지 않는다"로 버틴다.

## 무슨 취약점인가

원문은 6건을 나열하지만 심각도 등급이나 CVSS는 적지 않았다. 그래서 여기서는 "어떤 동작이 위험해지는가"만 정리한다.

### 1. 재귀 클론 중 코드 실행 (CVE-2026-111760)

인덱스 처리에서 정수가 잘리는 문제다. 아주 긴 경로가 검증은 한 이름으로 통과하고 저장은 더 짧은 이름으로 된다. 공격자는 서브모듈의 `.git` 파일을 심볼릭 링크로 바꿔 악성 설정이나 훅을 실행시킬 수 있다. 조건은 악성 저장소를 `--recursive`로 클론하는 것이다.

### 2. 서버가 알려주는 HEAD로 경로 탈출 (CVE-2026-113408)

프로토콜 v2로 빈 저장소를 클론할 때, 서버가 아직 없는 브랜치를 가리키는 HEAD를 알려준다. 이 대상을 검증하지 않아서, 악성 서버가 경로 탈출 문자열을 넣으면 이후 브랜치 갱신 때 참조 디렉터리 밖에 쓸 수 있었다. files 참조 백엔드 기준이다.

### 3. 서명 제거 중 스택 손상 (CVE-2026-114467)

메시지에서 GPG 서명을 뗄 때 서명이 두 개 이하라고 가정했다. 서명이 세 개 이상인 태그를 머지하면 고정 크기 배열 밖에 쓴다. 원문은 메모리 손상과 서비스 거부 가능성을 언급하고, 사용자 대신 머지해 주는 호스팅 서비스도 영향권이라고 적었다.

### 4. 바이너리 패치 적용 중 범위 밖 읽기 (CVE-2026-114469)

약 1GiB 넘는 패치를 막는 검사가 "전달받은 크기"만 봤다. 바이너리 패치는 압축·인코딩돼 있어 작은 패치가 한도를 넘게 풀릴 수 있다. 조작된 패치로 검사를 피하면 Git이 충돌한다.

### 5. 큰 머지에서 힙 손상 (CVE-2026-105437)

머지 결과가 매우 크면 메모리 계산이 넘쳐서 너무 작은 버퍼를 잡고 그 밖에 쓴다. 충돌이 많고 conflict-marker-size 속성이 크면 조건이 맞는다. 신뢰할 수 없는 내용을 머지하는 앱·서비스가 특히 해당한다.

### 6. fast-import 스트림으로 파일 접근 (CVE-2026-114466)

`rewrite-submodules-from`, `rewrite-submodules-to`, `export-pack-edges` 세 기능이 신뢰할 수 없는 스트림에 대한 보호를 받지 못했다. 앞의 둘은 임의 파일을 읽을 수 있고, 뒤의 하나는 파일을 만들거나 덧붙일 수 있다. 이제는 스트림 안에서 쓰면 거부되고, 믿는 프로그램이 만든 스트림일 때만 `--allow-unsafe-features`를 줘야 한다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="Git 취약점 6건이 닿는 지점을 클론, 머지, 패치·가져오기 세 입구로 묶은 그림">
  <defs>
    <marker id="gv-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 취약점이 닿는 세 입구</text>
  <rect x="8" y="50" width="200" height="56" rx="8" class="sk-box"/>
  <text x="108" y="74" text-anchor="middle" class="sk-label">클론</text>
  <text x="108" y="94" text-anchor="middle" class="sk-sub">재귀 클론 · 빈 저장소 HEAD</text>
  <rect x="260" y="50" width="200" height="56" rx="8" class="sk-box"/>
  <text x="360" y="74" text-anchor="middle" class="sk-label">머지</text>
  <text x="360" y="94" text-anchor="middle" class="sk-sub">다중 서명 태그 · 거대한 결과</text>
  <rect x="512" y="50" width="200" height="56" rx="8" class="sk-box"/>
  <text x="612" y="74" text-anchor="middle" class="sk-label">패치·스트림</text>
  <text x="612" y="94" text-anchor="middle" class="sk-sub">바이너리 패치 · fast-import</text>
  <path d="M108,108 L108,150" class="sk-line" marker-end="url(#gv-m1)"/>
  <path d="M360,108 L360,150" class="sk-line" marker-end="url(#gv-m1)"/>
  <path d="M612,108 L612,150" class="sk-line" marker-end="url(#gv-m1)"/>
  <rect x="8" y="152" width="704" height="64" rx="8" class="sk-box-accent"/>
  <text x="360" y="178" text-anchor="middle" class="sk-label">공통 조건: 신뢰할 수 없는 저장소·서버·패치·스트림</text>
  <text x="360" y="200" text-anchor="middle" class="sk-sub">내가 만든 저장소만 만지는 환경이면 위험이 훨씬 작다</text>
</svg>
<figcaption>여섯 건 모두 "남이 만든 입력"을 Git에 먹일 때 열리는 문이다.</figcaption>
</figure>

## 원문이 말한 제품별 영향

- **GitHub.com**: github.com, Enterprise Cloud, 데이터 레지던시 포함 Enterprise Cloud에 수정이 배포됐다.
- **GHES**: 3.19.15, 3.20.11, 3.21.9, 3.22.4. 관리자는 가능한 빨리 올리라고 안내한다.
- **GitHub Desktop·Copilot 앱**: 패치된 Git을 담은 릴리스를 조율 중이라고만 했고 버전과 날짜는 없다.
- **Codespaces·Actions**: 패치된 Git을 받을 예정이라고 했고 날짜는 없다.

## 실무에서 달라지는 점

### 1. GHES 관리자: 이번 주 안에 패치 계획을

사내 GHES를 쓰면 위 네 버전 중 현재 시리즈에 맞는 것으로 올리는 일이 1순위다. 어제 정리한 [Copilot 앱 GHES 지원 GA](/posts/github-copilot-app-ghes-ga/) 때문에 GHES 버전을 올릴 계획이 이미 있다면, 이 패치를 같은 작업창에 묶는 편이 낫다.

### 2. CI·빌드 서버 운영자: Git이 어디에 들어 있는지 목록부터

OS 패키지 Git, 컨테이너 이미지에 박힌 Git, 에이전트 도구가 번들한 Git은 각각 따로 올려야 한다. 원문도 "자체 Git을 번들한 애플리케이션을 업데이트하라"고 했다. 외부 포크의 PR을 받아 클론·머지하는 파이프라인이 있다면 그곳이 우선순위가 높다.

### 3. 코딩 에이전트를 도입한 팀: 외부 저장소를 읽히는 곳을 확인

에이전트가 낯선 저장소를 재귀 클론하는 구조라면 첫 번째 취약점의 조건에 닿는다. 에이전트 실행 환경의 Git 버전과, 격리된 작업 디렉터리인지부터 확인한다. 이 연결은 원문이 직접 말한 것이 아니라 이 블로그의 해석이다.

### 4. 바로 못 올리는 곳: 원문이 준 임시 대응

신뢰하는 서버에서만 클론하고, 신뢰할 수 없는 저장소는 재귀 클론하지 않고, 신뢰할 수 없는 태그·내용을 머지하지 않고, 출처 불명 바이너리 패치와 fast-import 스트림을 처리하지 않는다. 원문도 이것이 패치를 대신하지 못한다고 못 박았다.

### 5. IT 강사: 교육 자료의 한 줄

"`git clone --recursive`는 편의 옵션이 아니라 남의 설정을 내려받는 행위"라고 짚을 만하다. 훅과 서브모듈이 코드 실행 경로가 될 수 있다는 점은 이번 건과 상관없이 오래 쓸 수 있는 설명이다.

## 아직 확인되지 않은 것

원문에는 CVE별 심각도, 영향받는 버전 범위, 데스크톱 앱의 패치 일정이 없다. 정확한 범위는 링크된 보안 권고와 2.56.1 릴리스 노트에서 확인해야 한다. 이 글은 그 문서들을 읽은 것이 아니다.

## 참고

- GitHub Blog, Git security vulnerabilities announced: https://github.blog/open-source/git/git-security-vulnerabilities-announced-7/
- Git 2.56.1 릴리스 노트: https://github.com/git/git/blob/v2.56.1/Documentation/RelNotes/2.56.1.adoc
- Git 다운로드: https://git-scm.com/downloads
