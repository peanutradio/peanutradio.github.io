---
title: "GitHub 보안 권고 권한 GA — 관리자 권한 없이 권고를 다루는 법"
date: 2026-11-22 06:51:00 +0900
categories: [IT News]
tags: [github, security, governance, devops]
description: 커스텀 저장소 역할에 보안 권고 권한 4종이 GA로 추가됐다. 저장소 관리자 없이 권고를 다루게 되는 대신 남는 한계와 점검할 것을 정리했다.
---

> 이 글은 GitHub Changelog의 "Security advisory permissions are now generally available"(2026-11-20) 발표를 읽고 정리·재구성한 것이다.
{: .prompt-info }

## 결론 먼저

- 커스텀 저장소 역할(custom repository role)에 보안 권고 관련 권한 4종이 추가됐고, 상태는 GA다.
- 보안 권고를 다루려고 저장소 관리자(admin)나 조직 전체의 보안 관리자(security manager) 역할을 줄 필요가 줄어든다.
- 다만 모든 작업이 위임되는 건 아니다. 임시 비공개 포크에서 병합하는 일은 여전히 관리자나 보안 관리자가 해야 한다.

## 발표 내용 요약

원문이 밝힌 권한은 네 가지다. 아래는 원문 기준이며, 메뉴 이름은 원문에서 확인한 범위에서만 적는다.

### 1. 보안 권고 보기

비공개 보안 권고를 읽을 수 있는 가장 낮은 단계다. 다른 권한 두 가지가 이 권한이나 다음 권한을 전제로 한다.

### 2. 보안 권고 만들기·수정·관리

권고를 작성하고 고치고 관리하는 권한이다. 이름이 "Create, Edit, and Manage"로 묶여 있어서 보기보다 한 단계 위로 생각해야 한다.

### 3. 비공개 취약점 신고 수락, 권고 닫기·다시 열기

외부에서 들어온 비공개 취약점 신고를 받아들이고, 권고를 닫거나 다시 여는 권한이다. 1번 또는 2번 권한이 먼저 있어야 한다.

### 4. 권고 게시와 CVE 요청

권고를 공개하고 CVE를 요청하는 권한이다. 이것도 1번 또는 2번이 전제다.

권한은 저장소 단위로 역할에 붙는다. 원문이 밝힌 한계는 세 가지다. 첫째, 임시 비공개 포크에서 병합하려면 여전히 저장소 관리자나 보안 관리자가 필요하다. 둘째, 권한 보유자가 임시 비공개 포크를 삭제할 수는 없다. 셋째, 비공개 코멘트를 쓰려면 2번 권한에 더해 저장소 쓰기(write) 권한이 필요하다. 대상은 커스텀 저장소 역할을 지원하는 플랜이고, GitHub Enterprise Cloud가 포함된다고 한다.

<figure class="sketch">
<svg viewBox="0 0 720 280" role="img" aria-label="보안 권고 권한 네 가지의 의존 관계와 관리자 전용으로 남는 작업을 보여주는 그림">
  <defs>
    <marker id="gsa-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 권한 4종의 의존 관계와, 여전히 관리자 몫인 일</text>
  <rect x="8" y="50" width="190" height="56" rx="8" class="sk-box"/>
  <text x="103" y="74" text-anchor="middle" class="sk-label">보기</text>
  <text x="103" y="94" text-anchor="middle" class="sk-sub">View security advisories</text>
  <rect x="8" y="130" width="190" height="56" rx="8" class="sk-box"/>
  <text x="103" y="154" text-anchor="middle" class="sk-label">만들기·수정·관리</text>
  <text x="103" y="174" text-anchor="middle" class="sk-sub">Create, Edit, and Manage</text>
  <rect x="270" y="50" width="200" height="56" rx="8" class="sk-box-accent"/>
  <text x="370" y="74" text-anchor="middle" class="sk-label">신고 수락·닫기·재오픈</text>
  <text x="370" y="94" text-anchor="middle" class="sk-sub">위 둘 중 하나 필요</text>
  <rect x="270" y="130" width="200" height="56" rx="8" class="sk-box-accent"/>
  <text x="370" y="154" text-anchor="middle" class="sk-label">게시·CVE 요청</text>
  <text x="370" y="174" text-anchor="middle" class="sk-sub">위 둘 중 하나 필요</text>
  <path d="M200,78 L266,78" class="sk-line" marker-end="url(#gsa-m1)"/>
  <path d="M200,158 L266,158" class="sk-line" marker-end="url(#gsa-m1)"/>
  <rect x="520" y="50" width="192" height="136" rx="8" class="sk-box-muted"/>
  <text x="616" y="80" text-anchor="middle" class="sk-label">관리자·보안 관리자 전용</text>
  <text x="616" y="108" text-anchor="middle" class="sk-sub">임시 비공개 포크 병합</text>
  <text x="616" y="128" text-anchor="middle" class="sk-sub">임시 포크 삭제는 권한 보유자도 불가</text>
  <text x="616" y="148" text-anchor="middle" class="sk-sub">비공개 코멘트는 쓰기 권한도 필요</text>
  <text x="360" y="236" text-anchor="middle" class="sk-sub">권한은 저장소 단위 역할에 붙는다. 조직 전체 역할을 줄 필요가 줄어든다.</text>
</svg>
<figcaption>권고 업무의 앞쪽(보기·작성·게시)은 위임할 수 있게 됐지만, 코드 병합 단계는 아직 관리자에게 묶여 있다.</figcaption>
</figure>

## 실무에서 달라지는 점

### 1. 보안 담당에게 주던 광범위한 역할을 다시 점검한다

그동안 보안 권고를 다루게 하려면 저장소 관리자나 조직 단위 보안 관리자를 줘야 하는 경우가 많았다. 이제는 저장소 단위의 좁은 역할로 줄일 수 있다. 조직 운영자는 현재 보안 관리자 역할이 붙은 사람 목록을 뽑아서, 실제로는 권고만 다루는 사람이 있는지 확인해 볼 만하다. 이 판단은 내 해석이며, 현재 권한 구성은 조직마다 다르다.

### 2. 역할 설계는 "읽기 전용 → 작성 → 게시" 순으로 쪼갠다

원문의 의존 관계를 보면 게시와 신고 수락은 보기 또는 작성 권한이 있어야 한다. 그래서 역할을 만들 때는 보기만 있는 역할, 작성까지 하는 역할, 게시·CVE 요청까지 하는 역할로 나누는 방식이 자연스럽다. 게시는 외부에 공개되는 행위라서 작성자와 게시자를 분리하고 싶은 팀이 있을 것이다. 이것도 설계 제안이지 원문의 권고는 아니다.

### 3. 병합은 위임되지 않는다는 점을 업무 흐름에 반영한다

비공개 권고 작업의 마지막은 보통 임시 비공개 포크에서의 수정 병합이다. 원문에 따르면 이 단계는 여전히 저장소 관리자나 보안 관리자가 필요하다. 권고 담당자에게 권한을 나눠 줬어도 병합 시점에 관리자를 호출해야 하므로, 대응 절차 문서에 "누가 병합하나"를 적어 둬야 한다. 임시 포크 삭제도 권한 보유자는 못 한다.

### 4. 플랜과 설정 위치를 먼저 확인한다

대상은 커스텀 저장소 역할을 지원하는 플랜이다. 어느 플랜이 해당하는지, 역할을 만드는 설정 화면이 어디인지는 이번 발표 본문만으로는 내가 확인하지 못했다. 적용 전에 GitHub 문서에서 커스텀 저장소 역할 항목을 열어 확인하는 것이 안전하다.

### 5. 교육할 때는 비공개 권고의 수명주기로 설명한다

IT 강사 입장에서는 "신고 접수 → 권고 작성 → 수정 병합 → 게시·CVE" 순서로 설명하고, 각 단계에 어떤 권한이 필요한지 표로 붙이면 이해가 빠를 것이다. 병합 단계만 관리자 몫이라는 예외를 따로 강조해야 한다. 보안 패치 대응 사례는 [Git 2.56.1 보안 패치](/posts/git-2-56-1-six-vulnerabilities/)에서 다룬 점검 순서와 이어서 보면 좋다.

## 한 줄 정리

기능 자체는 작은 변경이지만, 권한을 쪼개 줄 수 있게 되면서 "보안 권고 담당 = 관리자"라는 습관을 고칠 기회가 생겼다. 단, 병합 단계가 남아 있어서 절차 문서까지 같이 손봐야 효과가 있다.

## 참고

- GitHub Changelog, Security advisory permissions are now generally available (2026-11-20): <https://github.blog/changelog/2026-11-20-security-advisory-permissions-are-now-generally-available>
- 이어 읽기: [Git 2.56.1 보안 패치 — 취약점 6건, 사내 점검 순서](/posts/git-2-56-1-six-vulnerabilities/)
