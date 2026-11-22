---
title: "Azure Cobalt 200 VM GA — Arm 전환 전에 볼 리전·모드·호환성"
date: 2026-11-23 06:53:00 +0900
categories: [IT News]
tags: [azure, cost, devops]
description: Azure Cobalt 200 기반 Arm VM이 GA됐다. 14개 리전, 클럭 모드 두 가지, 새 시리즈를 정리하고 도입 전 확인할 점을 짚었다.
---

> 이 글은 Microsoft Azure 블로그의 "Azure Cobalt 200 VMs now available" 발표(2026-11-17, Ignite 2026)를 읽고 정리·재구성한 것입니다. 수치는 모두 원문 표기이며, 대부분 "최대(up to)" 조건과 Microsoft 자체 비교입니다.
{: .prompt-info }

## 결론부터

Cobalt 200 기반 VM이 **GA**가 됐습니다. 지금 쓸 수 있는 리전은 14곳이고, 원문은 2026년 말까지 23곳으로 늘린다고 합니다. 한국 쪽 운영자에게는 **Korea Central이 GA 목록에 들어 있다**는 점이 가장 직접적인 소식입니다.

다만 이 글을 읽고 바로 "Arm으로 갈아타자"고 결론 내리기는 이릅니다. 원문에는 **가격이 없고**, 마이그레이션 절차 상세도 없습니다. 아래에서는 원문이 말한 것, 원문이 말하지 않은 것, 그래서 운영자가 먼저 확인할 것을 나눠 적습니다.

## 발표 내용 요약

### 칩과 시리즈

Cobalt 200은 Microsoft가 설계한 Arm 기반 SoC입니다. 원문 기준으로 TSMC 3nm 공정, Arm Neoverse V3 코어 132개, 코어당 3MB L2 캐시, 시스템 캐시 192MB, DDR5 12채널 메모리 구성이며 메모리 암호화가 기본 적용됩니다. 이 숫자는 SoC 기준이고, VM 한 대의 vCPU는 표 기준 최대 128입니다.

VM 시리즈는 다음과 같이 나뉩니다.

- 범용: Dplsv7·Dpldsv7(vCPU당 2GiB), Dpsv7·Dpdsv7(vCPU당 4GiB)
- 메모리 최적화: Epsv7·Epdsv7(vCPU당 8GiB, 최대 1,024GiB)
- 고메모리(신규): Mpsv4·Mpdsv4(vCPU당 16GiB, 최대 1,388GiB)
- 스토리지 최적화(신규): Lpsv5(로컬 NVMe 최대 23TB)

### 성능 수치

Cobalt 100 대비 "최대 50% 향상"이 헤드라인입니다. 세부 항목은 웹 서빙 1.5배, Java 1.6배, 에이전트 도구 실행 1.5배, 데이터베이스 1.9배(모두 최대치)입니다. 네트워크 대역폭 최대 15%, 원격 스토리지 IOPS 20%, 처리량 10% 향상도 같은 비교 기준입니다.

고객 사례로는 Amadeus가 항공편 검색 처리량을 Cobalt 100 대비 30% 이상 높였다고 하는데, 벤치마크 랩 결과로 표기돼 있습니다. OpenAI, Databricks, Snowflake의 코멘트는 정성적 평가 위주였습니다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="Cobalt 200 VM 도입 전 확인 순서: 리전, 호환성, 클럭 모드, 가격">
  <defs>
    <marker id="cb-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="16" class="sk-title">① Cobalt 200 도입 전 확인 순서</text>
  <rect x="8" y="60" width="150" height="70" rx="8" class="sk-box-accent"/>
  <text x="83" y="90" text-anchor="middle" class="sk-label">리전 확인</text>
  <text x="83" y="110" text-anchor="middle" class="sk-sub">14곳 GA, 연말 23곳</text>
  <rect x="198" y="60" width="150" height="70" rx="8" class="sk-box"/>
  <text x="273" y="90" text-anchor="middle" class="sk-label">Arm 호환성</text>
  <text x="273" y="110" text-anchor="middle" class="sk-sub">OS·이미지·바이너리</text>
  <rect x="388" y="60" width="150" height="70" rx="8" class="sk-box"/>
  <text x="463" y="90" text-anchor="middle" class="sk-label">클럭 모드 선택</text>
  <text x="463" y="110" text-anchor="middle" class="sk-sub">Deterministic / Opportunistic</text>
  <rect x="578" y="60" width="134" height="70" rx="8" class="sk-box-muted"/>
  <text x="645" y="90" text-anchor="middle" class="sk-label">가격 비교</text>
  <text x="645" y="110" text-anchor="middle" class="sk-sub">원문에 수치 없음</text>
  <path d="M160,95 L194,95" class="sk-line" marker-end="url(#cb-m1)"/>
  <path d="M350,95 L384,95" class="sk-line" marker-end="url(#cb-m1)"/>
  <path d="M540,95 L574,95" class="sk-line" marker-end="url(#cb-m1)"/>
  <text x="8" y="190" class="sk-sub">성능 수치는 모두 Microsoft 자체 비교 기준이므로, 마지막 판단은 우리 워크로드로 하는 쪽이 안전하다.</text>
</svg>
<figcaption>성능 숫자보다 먼저 리전과 호환성이 되는지부터 확인해야 한다.</figcaption>
</figure>

## 원문에 있는 것 중 놓치기 쉬운 두 가지

### 클럭 모드를 고르게 돼 있다

ProcessorMode 설정으로 두 모드 중 하나를 고릅니다. Deterministic Mode는 코어를 3.2GHz로 고정하고, Opportunistic Mode는 최대 3.6GHz까지 올라가 최대 12% 더 높은 성능을 내는 대신 **주파수 변동이 생깁니다.** 원문은 분산 데이터·분석 클러스터처럼 노드 간 속도 편차가 문제 되는 경우에는 Deterministic을 권합니다. 설정은 포털, CLI, PowerShell, ARM/Bicep에서 가능하다고 합니다.

### 물량은 2027년부터 본격적

원문은 Cobalt 200 VM이 "2027년부터 신규 Azure 컴퓨트 용량의 상당 비중"이 될 것이라고 적었습니다. 지금은 시작 단계라는 뜻이고, 특정 리전·사이즈에서 용량이 빡빡할 가능성도 열어 두는 편이 좋습니다. 이 부분은 원문이 직접 말한 것이 아니라 제 추정입니다.

## 실무에서 달라지는 점

### 1. Azure 운영자 — 리전부터 대조

배포 리전이 14곳 목록(Korea Central 포함, 그 외 East US, Sweden Central, UK South 등)에 있는지 먼저 봅니다. 목록 밖이라면 연말 확장을 기다려야 합니다. 최신 현황은 원문이 안내한 "Products Available by Region"에서 확인합니다.

### 2. 비용 담당 — 가격은 직접 계산

원문에는 시간당 단가가 없습니다. "가성비가 좋다"는 문구만 있으므로, 현재 쓰는 x86 사이즈·Cobalt 100과 같은 vCPU·메모리 기준으로 가격 페이지에서 직접 비교해야 합니다. 성능이 최대 1.5배라 해도 그 수치는 우리 워크로드에서 재현된다는 보장이 아닙니다.

### 3. 앱·플랫폼 팀 — Arm 호환성이 진짜 비용

원문이 지원 OS로 든 것은 Ubuntu, Debian, RHEL, SUSE, Azure Linux, AlmaLinux, Rocky, Oracle Linux, Windows 11 클라이언트입니다. **Windows Server는 본문에 언급이 없습니다.** 컨테이너 이미지의 arm64 빌드 여부, 네이티브 라이브러리, 에이전트·백신·모니터링 소프트웨어의 Arm 지원을 먼저 목록으로 만들어야 합니다. AKS 노드풀과 VM Scale Set에서도 쓸 수 있다고 하니, 컨테이너 중심 워크로드가 시험하기 쉬운 쪽입니다.

### 4. 개발·CI 담당 — 멀티 아키텍처 빌드

Arm VM으로 옮기려면 빌드 파이프라인이 amd64와 arm64를 둘 다 내는지가 관건입니다. 원문은 GitHub Actions의 더 큰 Arm 러너가 Cobalt 100보다 작업을 눈에 띄게 빨리 끝냈다고 적었는데, 구체 수치는 없습니다.

### 5. IT 강사 — 교육에서 강조할 점

"Arm이면 무조건 싸고 빠르다"가 아니라 "호환성 확인 → 소규모 시험 → 비교 후 이전"의 순서를 강조하는 편이 좋겠습니다. 벤치마크의 "up to"와 비교 대상(Cobalt 100)을 읽는 연습도 같이 시키면 도움이 됩니다.

## 이어 읽기

Ignite 주간의 Azure·AI 소식 전체 흐름은 [Azure·AI 주간 업데이트 — 11월 셋째 주](/posts/azure-ai-weekly-2026-11-20/)에 정리해 두었습니다.

## 참고

- [Azure Cobalt 200 VMs now available (Azure Blog, 2026-11-17)](https://azure.microsoft.com/en-us/blog/azure-cobalt-200-vms-now-available-proven-on-real-workloads-and-built-for-the-agentic-era/)
- [Cobalt 200 VM 사이즈 및 배포 가이드 (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/cobalt-overview)
- [Cobalt 100 대비 50% 성능 향상 (Azure Blog)](https://azure.microsoft.com/en-us/blog/new-azure-cobalt-200-vms-deliver-50-performance-improvement-fully-optimized-for-modern-agentic-ai-workloads/)
- [Ignite 2026: Building the frontier ecosystem (Official Microsoft Blog)](https://blogs.microsoft.com/blog/2026/11/17/ignite-2026-building-the-frontier-ecosystem/)
- [리전별 제품 가용성](https://aka.ms/product-by-region)
