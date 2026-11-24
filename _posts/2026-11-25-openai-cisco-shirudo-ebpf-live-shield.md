---
title: "OpenAI·Cisco Shirudo — 패치 전 서버를 재시작 없이 막는 eBPF 방패"
date: 2026-11-25 06:53:00 +0900
categories: [IT News]
tags: [security, openai, ai-agent]
description: OpenAI와 Cisco가 공개한 오픈소스 런타임 Shirudo가 무엇을 하는지, 아직 확인되지 않은 부분은 무엇인지, 리눅스 서버 운영자가 볼 지점을 정리했다.
---

> 이 글은 OpenAI가 2026-11-23에 올린 "OpenAI and Cisco introduce Shirudo" 발표를 읽고 정리·재구성한 것이다. 수치와 시험 결과는 모두 발표문의 주장이며 직접 실행해 확인한 내용이 아니다.
{: .prompt-info }

## 결론부터

Shirudo는 취약점 패치가 나오기 전이나 적용되기 전에, 실행 중인 리눅스 서버에 "임시 방패(shield)"를 얹어 공격 경로만 막는 오픈소스 런타임이다. 서비스를 내리지 않고 막는다는 점이 핵심이다. 방패 코드를 AI 에이전트(Codex)가 만들 수 있다는 점이 이번 발표의 새로운 부분이다.

다만 발표문에는 버전 번호도, 프리뷰나 GA 같은 상태 표기도 없다. 운영 서버에 바로 넣을 수 있는 완성품이라기보다 "첫걸음"이라고 스스로 부르는 프로젝트다. 이 글은 그 전제로 읽는다.

## 무엇을 하는 도구인가

### 문제 설정

발표문이 짚는 문제는 단순하다. 패치에는 며칠에서 몇 주가 걸릴 수 있다. 그렇다고 취약한 리눅스 동작을 통째로 막으면 정상 요청까지 끊겨 서비스가 멈출 수 있다. 공격과 정상 요청이 같은 동작을 공유할 때 특히 그렇다.

### 방패라는 해법

Shirudo는 일본어로 "방패"라는 뜻이다. 이름 그대로 문제가 되는 동작만 골라 막는 임시 방패를 만든다. 운영자가 승인하면 서비스가 도는 채로 방패를 배포한다. 적용 전에 방패가 애플리케이션에 어떤 영향을 줄지 먼저 관찰하는 단계도 있다고 한다.

### 기술 기반은 eBPF

eBPF는 커널의 특정 지점에 작은 프로그램을 끼워 넣는 리눅스 기능이다. Shirudo는 이것으로 의심스러운 활동을 관찰하고 필요한 곳만 막는다. 발표문은 신뢰 장치를 세 겹으로 설명한다.

- 방패는 암호학적으로 서명되고 검증된 뒤에만 로드된다.
- 리눅스 eBPF 검증기가 안전 규칙을 검사한다.
- TPM을 써서 방패가 로드되거나 제거된 기록을 검증할 수 있게 한다.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="취약점 발견부터 방패 배포까지의 흐름과 패치 경로 비교">
  <defs>
    <marker id="sh-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="sh-a1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 패치를 기다리는 동안의 두 갈래</text>
  <rect x="8" y="90" width="130" height="52" rx="8" class="sk-box"/>
  <text x="73" y="112" text-anchor="middle" class="sk-label">취약점 공개</text>
  <text x="73" y="130" text-anchor="middle" class="sk-sub">서버는 가동 중</text>
  <rect x="200" y="40" width="170" height="52" rx="8" class="sk-box-muted"/>
  <text x="285" y="62" text-anchor="middle" class="sk-label">패치·재시작 대기</text>
  <text x="285" y="80" text-anchor="middle" class="sk-sub">며칠~몇 주 노출</text>
  <rect x="200" y="150" width="170" height="52" rx="8" class="sk-box-accent"/>
  <text x="285" y="172" text-anchor="middle" class="sk-label">방패 시험·승인</text>
  <text x="285" y="190" text-anchor="middle" class="sk-sub">서명·검증 후 로드</text>
  <rect x="440" y="150" width="150" height="52" rx="8" class="sk-box-accent"/>
  <text x="515" y="172" text-anchor="middle" class="sk-label">서비스 유지</text>
  <text x="515" y="190" text-anchor="middle" class="sk-sub">공격 경로만 차단</text>
  <rect x="440" y="40" width="150" height="52" rx="8" class="sk-box"/>
  <text x="515" y="62" text-anchor="middle" class="sk-label">정식 패치</text>
  <text x="515" y="80" text-anchor="middle" class="sk-sub">영구 해결</text>
  <path d="M140,108 L198,70" class="sk-line" marker-end="url(#sh-m1)"/>
  <path d="M140,124 L198,170" class="sk-line-accent" marker-end="url(#sh-a1)"/>
  <path d="M372,66 L438,66" class="sk-line" marker-end="url(#sh-m1)"/>
  <path d="M372,176 L438,176" class="sk-line-accent" marker-end="url(#sh-a1)"/>
  <path d="M515,148 L515,94" class="sk-line" marker-end="url(#sh-m1)"/>
</svg>
<figcaption>방패는 패치를 대체하는 것이 아니라, 패치가 도착할 때까지의 공백을 메우는 임시 수단이다.</figcaption>
</figure>

## 발표문이 내놓은 시험 결과

통제된 시험에서 Codex가 만든 방패가 실행 중인 Elasticsearch에 대한 Java4Shell 공격 시도를 재시작 없이 막았다고 한다. 방패는 Java Base64 디코더에 입력 검사를 더하는 방식이었고, 악성 입력은 거부되고 Elasticsearch는 계속 돌았다는 설명이다.

여기서 구분할 점이 있다. 이 결과는 OpenAI와 Cisco가 자기 환경에서 보고한 사례 한 건이다. 다른 취약점, 다른 커널 버전, 실제 트래픽 아래에서도 같은 결과가 나온다는 근거는 발표문에 없다. 오탐으로 정상 요청이 막힐 확률도 수치로 나와 있지 않다.

## 에이전트는 어디까지 하나

발표문은 에이전트가 취약점을 조사하고, 표적형 방패를 만들고, 공격과 실제 워크로드에 대해 검증하도록 설계했다고 쓴다. 다만 무엇을 배포할지는 보안팀이 통제한다고 못 박는다. 서명 검증과 운영자 승인이 들어 있는 것은 에이전트가 만든 코드를 커널 가까이에 바로 올리지 않겠다는 설계다.

에이전트의 권한을 어떻게 나눌지는 앞서 쓴 [에이전트 코드 실행 샌드박스 글](/posts/aca-dynamic-sessions-agent-sandbox-guide/)과 같은 맥락이다. 여기서는 격리 대상이 에이전트가 짠 코드가 아니라 에이전트가 짠 방어 코드이고, 그 코드를 올리는 자리가 커널이라는 점이 다르다. 에이전트의 이상 행동을 탐지하는 쪽 이야기는 [Google 이상 탐지 벤치마크 글](/posts/google-agent-anomaly-detection-benchmarks/)에서 다뤘다. 이 둘이 에이전트를 감시하는 쪽이라면 Shirudo는 에이전트가 방어를 만드는 쪽이다.

## 실무에서 달라지는 점

### 1. 당장 운영 서버에 넣을 도구는 아니다

발표문은 버전, 지원 배포판, 커널 요건을 밝히지 않는다. 시작점은 공식 VM 퀵스타트(docs.shirudo.dev)이고, 서명·로드·검증 시험 방어를 돌려 보는 수준이다. 운영 서버 담당자는 별도 테스트 VM에서 먼저 확인한다고 계획을 잡는 편이 맞다.

### 2. 패치 관리 절차의 "예외 경로"를 미리 정해 둔다

방패는 패치를 대신하지 못한다. 방패를 붙이면 취약점 대응이 끝난 것처럼 보이는 위험이 오히려 생긴다. 방패를 적용하더라도 정식 패치 일정과 방패 제거 시점을 변경 관리 티켓에 함께 남기는 규칙을 먼저 정해야 한다. [Git 2.56.1 보안 패치 글](/posts/git-2-56-1-six-vulnerabilities/)처럼 패치가 나오는 취약점과는 쓰임이 다르다. 패치가 없거나 늦는 구간만 대상이다.

### 3. 승인 권한과 감사 기록을 누가 갖는지 정한다

방패는 운영자가 승인해야 배포되고, TPM으로 로드·제거 기록을 남길 수 있다고 한다. 그렇다면 승인자, 승인 기준, 기록 보관 위치를 도입 전에 정해야 한다. 커널 수준 기능이라 서버 관리팀과 보안팀 중 어디가 책임지는지 모호하면 사고가 났을 때 가장 먼저 문제가 된다.

### 4. 오탐 영향 시험을 요구한다

방패 적용 전에 영향을 관찰하는 단계가 있다고 하니, 이 관찰 기간을 필수 절차로 삼아야 한다. 정상 트래픽이 막혔을 때 되돌리는 방법과 담당자도 같이 정한다. 발표문은 오탐률을 공개하지 않았다.

### 5. 교육할 때는 "무엇이 새로운가"를 가르친다

eBPF 자체는 새 기술이 아니다. 새로운 것은 AI가 방패 코드를 만들고 사람이 서명·승인하는 흐름이다. 강의에서는 eBPF 기초, 서명·검증 구조, 승인 책임이라는 세 줄기로 나누면 수강생이 마케팅 문구와 실제 기능을 구분하기 쉽다.

## 라이선스와 공개 범위

저장소는 GitHub의 shirudo-dev/shirudo에 공개돼 있다. 라이선스는 BSD 2-Clause이고, BPF 프로그램은 BSD와 GPL 이중 라이선스다. 사내 배포나 수정에 제약이 큰 편은 아니지만, 커널에 로드되는 BPF 부분이 GPL 이중이라는 점은 법무 검토 때 따로 적어 두는 것이 좋다. 12월 15일에 기술 심층 웨비나가 예정돼 있다고 안내돼 있다. 연도는 페이지에 적혀 있지 않다.

## 아직 모르는 것

- 상태 표기(프리뷰·GA)와 버전
- 지원하는 리눅스 배포판과 커널 버전
- 방패를 만들 때 Codex 사용 조건이나 비용
- 시험 범위 밖의 방어 성공률, 오탐률, 성능 부담

이 항목들이 채워지기 전까지 이 발표는 "방향을 보여 준 오픈소스 첫 공개"로 읽는 것이 정확하다.

## 참고

- OpenAI, "OpenAI and Cisco introduce Shirudo" (2026-11-23): <https://openai.com/index/introducing-shirudo/>
- Isovalent(Cisco) 블로그, 통제된 시험 설명: <https://isovalent.com/blog/post/cisco-openai-shirudo/>
- Shirudo 저장소: <https://github.com/shirudo-dev/shirudo>
- Shirudo 문서·VM 퀵스타트: <https://docs.shirudo.dev/start/quickstart.html>
- 기술 워크스루(Linux Plumbers Conference): <https://lpc.events/event/20/contributions/2437/>
