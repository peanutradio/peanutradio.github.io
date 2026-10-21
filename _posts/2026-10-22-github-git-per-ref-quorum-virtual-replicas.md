---
title: "GitHub Git 인프라 후속편 — 참조 단위 쿼럼과 가상 복제본"
date: 2026-10-22 06:53:00 +0900
categories: [IT News]
tags: [github, git, infrastructure, azure, agents]
description: GitHub가 Git 쓰기 병목을 참조(ref) 단위 쿼럼으로, 읽기 확장을 가상 복제본으로 푼 방식을 정리한다. 프로덕션 배포 범위는 글에 명시되지 않았다.
---

> 이 글은 GitHub 엔지니어링 블로그의 「The bridge to agent-scale Git infrastructure」(2026-10-21)를 읽고 정리·재구성한 것입니다. 에이전트와의 연결 해석은 제 의견이며 원문 주장이 아닙니다.
{: .prompt-info }

**결론부터.** GitHub는 Git 쓰기가 느려지는 이유를 "저장소 전체 잠금과 가장 느린 복제본"에서 찾고, 쿼럼 단위를 저장소에서 참조(브랜치·태그)로 좁혔습니다. 내부 테스트에서 서로 다른 참조에 대한 쓰기 처리량이 3~4배 올랐고 지연도 낮아졌다고 합니다. 읽기는 쿼럼 밖의 가상 읽기 복제본으로 따로 늘립니다. 이전 글(10/6, 에이전트 규모 Git 인프라)이 "왜 옮겼나"였다면 이번은 "옮긴 뒤 무엇을 고쳤나"입니다.

## 배경: 옮겼더니 다음 병목이 보였다

에이전트 트래픽이 늘면서 온프레미스 데이터센터는 전력과 공급망에 막혔고, GitHub는 저장소의 약 99.99999%를 Azure에도 복제했습니다. 대부분 저장소는 복제본 5개 중 2개가 Azure에 있고 Git 트래픽의 약 절반을 Azure가 처리합니다. 남은 저장소는 가장 바쁜 쪽이고, 데이터센터와 Azure 사이 왕복이 약 25ms라서 쓰기가 병목입니다. 쓰기 한 번에 왕복이 여러 번 필요하니 이 25ms가 쌓입니다.

## 변경 1: 참조 단위 쿼럼

기존 쿼럼은 과반이면 커밋되는데도 건강한 복제본을 모두 기다렸고, 저장소 전체 잠금을 쥔 채였습니다. 그래서 가장 느린 복제본이 지연과 처리량 상한을 정했습니다. 이제 잠금과 합의 범위가 참조 단위라서, 같은 참조를 두고 다투는 쓰기만 직렬화됩니다. 에이전트들이 각자 브랜치에 push하는 상황에 맞는 구조입니다. 참조별 일관성("받아들여진 갱신은 이전 값으로 보이지 않는다")은 유지된다고 합니다.

## 변경 2: 저장소와 컴퓨트 분리

<figure class="sketch">
<svg viewBox="0 0 720 260" role="img" aria-label="쓰기는 가상 쓰기 복제본이 Blob Storage에 기록하고 읽기는 가상 읽기 복제본이 담당하는 구조">
  <defs>
    <marker id="gq-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="gq-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 쓰기와 읽기를 따로 키우는 구조</text>
  <rect x="8" y="50" width="130" height="150" rx="8" class="sk-box"/>
  <text x="73" y="116" text-anchor="middle" class="sk-label">클라이언트</text>
  <text x="73" y="134" text-anchor="middle" class="sk-sub">사람·에이전트</text>
  <rect x="230" y="40" width="190" height="60" rx="8" class="sk-box-accent"/>
  <text x="325" y="66" text-anchor="middle" class="sk-label">가상 쓰기 복제본</text>
  <text x="325" y="84" text-anchor="middle" class="sk-sub">쿼럼 구성원, 쓰기 묶음 처리</text>
  <rect x="230" y="150" width="190" height="60" rx="8" class="sk-box"/>
  <text x="325" y="176" text-anchor="middle" class="sk-label">가상 읽기 복제본</text>
  <text x="325" y="194" text-anchor="middle" class="sk-sub">쿼럼 밖, 필요할 때 확장</text>
  <rect x="520" y="80" width="190" height="90" rx="8" class="sk-box"/>
  <text x="615" y="120" text-anchor="middle" class="sk-label">Azure Blob Storage</text>
  <text x="615" y="138" text-anchor="middle" class="sk-sub">내구성 확보 후 확인 응답</text>
  <path d="M140,70 L228,70" class="sk-line-accent" marker-end="url(#gq-a1)"/>
  <path d="M140,180 L228,180" class="sk-line" marker-end="url(#gq-m1)"/>
  <path d="M422,70 L548,78" class="sk-line-accent" marker-end="url(#gq-a1)"/>
  <path d="M548,172 L422,180" class="sk-line" marker-end="url(#gq-m1)"/>
</svg>
<figcaption>쓰기 경로와 읽기 경로를 오브젝트 스토리지 위에서 분리해, 읽기가 100배 이상 많아도 쓰기 쿼럼을 키우지 않는다.</figcaption>
</figure>

- **가상 쓰기 복제본**: 쿼럼 구성원이면서 쓰기를 Azure Blob Storage로 흘려 보내는 다리입니다. 읽기는 안 합니다. 내구성이 확보돼야 확인 응답을 하므로 쓰기마다 약간의 지연이 붙고, 서로 다른 참조의 동시 쓰기를 묶어 I/O를 줄입니다.
- **가상 읽기 복제본**: 쿼럼 구성원이 아니며 필요할 때 늘어납니다. Blob Storage에서 캐시를 데워 읽기를 제공합니다. 읽기 대 쓰기 비율이 대략 100:1~1,000:1이라 효과가 큽니다.

## 글이 말하지 않은 것

두 변경이 프로덕션에 완전히 배포됐는지는 명시되지 않았고 GA 같은 일정도 없습니다. 3~4배의 기준선, 트래픽 구성, 여러 참조를 한 번에 push할 때의 원자성, 장애 복구, 비용도 다루지 않습니다. 남은 과제는 쿼럼의 쓰기 처리량 한계, 일부 읽기에 필요한 인라인 정합, 쓰기 경로의 오브젝트 스토리지 지연입니다. 끝 상태는 GitHub Universe(10/28~29) 후속 글에서 다룬다고 합니다.

## 실무에서 달라지는 점

이 글은 GitHub 내부 구조 이야기라서 사용자가 바꿀 설정은 없습니다. 그래도 운영자와 강사가 가져갈 것은 있습니다.

1. **Git 서비스 운영 담당: "쓰기는 같은 참조에서 줄을 선다"는 모델을 기억하세요.** 에이전트 여러 개가 같은 브랜치(예: `main`)에 몰리면 직렬화는 피할 수 없다고 읽힙니다. 에이전트별 브랜치로 나누는 설계가 구조와 맞습니다. (제 해석입니다.)
2. **자체 Git 서버·미러를 운영한다면: 복제 간 거리를 지연 예산에 넣으세요.** 왕복 25ms가 쓰기 한 번에 여러 번 곱해진다는 설명은 어느 분산 저장소에나 적용되는 계산입니다.
3. **CI·자동화 담당: 읽기 100배 이상 많은 환경에서는 읽기를 따로 키우는 설계를 점검하세요.** 클론·fetch 폭주를 쓰기 경로와 분리해 둔 구성인지 확인할 만합니다.
4. **도입 검토자: 이 글만으로 성능 약속을 하지 마세요.** 수치는 내부 테스트이고 사용자 체감 개선 일정은 글에 없습니다. 장애 이력은 [GitHub 9월 가용성 보고서](/posts/github-availability-september-2026/) 정리 글과 함께 보면 재시도·백그라운드 작업이 부하를 키우는 맥락이 이어집니다.
5. **강의할 때: "합의는 작게, 데이터는 크게"로 요약하세요.** 불변·내용 기반 이름의 Git 객체는 합의가 필요 없고, 순서가 필요한 소수의 참조 갱신만 합의 대상이라는 원문의 관점이 분산 시스템 수업의 좋은 사례입니다.

## 참고

- [The bridge to agent-scale Git infrastructure — GitHub Blog](https://github.blog/engineering/architecture-optimization/the-bridge-to-agent-scale-git-infrastructure/)
- [Building Git infrastructure for agent-scale development — GitHub Blog](https://github.blog/engineering/architecture-optimization/building-git-infrastructure-for-agent-scale-development/)
