---
title: ".NET 10월 긴급(OOB) 보안 업데이트 — 8.0.33·9.0.22·10.0.14로 올리기 전 점검"
date: 2026-10-26 06:51:00 +0900
categories: [IT News]
tags: [security]
description: .NET 8·9·10에 Brotli 버퍼 오버플로 등을 고친 긴급 보안 업데이트가 나왔다. 배포 방식(런타임·자체 포함·컨테이너)별로 무엇을 바꿔야 하는지 정리했다.
---

> 이 글은 .NET 블로그의 "[.NET October 2026 Out-of-Band Security Updates](https://devblogs.microsoft.com/dotnet/dotnet-october-2026-oob-security-updates/)"(2026-10-25, Rahul Bhandari)를 읽고 정리·재구성한 것이다.
{: .prompt-info }

## 결론부터

10월 13일 정기 릴리스가 끝이 아니었다. Microsoft가 10/25에 긴급(OOB) 보안 업데이트를 냈고, 수정 버전은 **.NET 8.0.33, 9.0.22, 10.0.14, .NET 11 RC3**이다. 원문은 "가능한 한 빨리 업데이트"를 권한다. 다만 글에는 CVE별 심각도 점수가 없으니, 우선순위를 정할 때는 MSRC 권고문을 직접 열어 확인해야 한다.

가장 먼저 봐야 할 것은 "우리 앱이 Brotli 압축 해제를 쓰는가"와 "런타임이 어떤 방식으로 배포됐는가" 두 가지다.

## 무엇이 고쳐졌나

### CVE-2026-111497 — Brotli 압축 해제 버퍼 오버플로

Brotli 압축 해제에서 버퍼 오버플로가 생겨, 인증 없는 공격자가 프로세스 메모리를 읽거나 손상시킬 수 있다고 한다. 원문이 드는 결과는 정보 노출, 서비스 거부, 권한 상승이다. 모든 플랫폼이 해당하고 .NET Framework는 해당하지 않는다.

영향 조건이 중요하다. 아래 세 경로로 Brotli 데이터를 처리할 때만 해당한다.

- ASP.NET Core 요청 압축 해제
- HttpClient 자동 응답 압축 해제
- `System.IO.Compression`의 Brotli API

요청 압축 해제와 자동 응답 압축 해제는 기본값이 꺼져 있다고 원문이 밝힌다. 즉 기본 설정 그대로인 서비스는 노출이 좁을 수 있지만, 켜 둔 서비스나 Brotli API를 직접 호출하는 코드가 있다면 얘기가 달라진다.

### CVE-2026-114148 — Linux tarball 소유권 메타데이터

Linux용 .NET tarball에 안전하지 않은 소유권 메타데이터가 있어, 로컬 공격자가 파일을 수정하고 권한을 높일 수 있다. .NET 10·11은 소스 빌드 tarball만 해당한다. 반면 .NET 8·9는 소스 빌드를 하지 않아도 **ASP.NET Core tarball로 설치한 경우**가 해당한다.

### 10/13 릴리스의 재배포 2건

CVE-2026-70197(SocketAsyncEventArgs 메모리 손상)과 CVE-2026-102870(Blazor WebAssembly 디버거)은 10/13 릴리스가 완전하지 않아 이번에 다시 수정됐다. 원문에는 제목 외 설명이 없다. 10/13 패치를 이미 적용했다고 안심한 팀이 이번 대상이다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="배포 방식별로 업데이트 방법이 다르다는 흐름도">
  <defs>
    <marker id="dn-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 배포 방식에 따라 "패치"의 의미가 다르다</text>
  <rect x="8" y="50" width="150" height="56" rx="8" class="sk-box-accent"/>
  <text x="83" y="75" text-anchor="middle" class="sk-label">새 수정 버전</text>
  <text x="83" y="93" text-anchor="middle" class="sk-sub">8.0.33 / 9.0.22 / 10.0.14</text>
  <path d="M160,78 L220,78" class="sk-line" marker-end="url(#dn-m1)"/>
  <rect x="222" y="20" width="230" height="52" rx="8" class="sk-box"/>
  <text x="337" y="42" text-anchor="middle" class="sk-label">프레임워크 종속</text>
  <text x="337" y="60" text-anchor="middle" class="sk-sub">호스트 런타임 설치 후 앱 재시작</text>
  <rect x="222" y="84" width="230" height="52" rx="8" class="sk-box"/>
  <text x="337" y="106" text-anchor="middle" class="sk-label">자체 포함(self-contained)</text>
  <text x="337" y="124" text-anchor="middle" class="sk-sub">앱을 다시 빌드·배포</text>
  <rect x="222" y="148" width="230" height="52" rx="8" class="sk-box"/>
  <text x="337" y="170" text-anchor="middle" class="sk-label">컨테이너</text>
  <text x="337" y="188" text-anchor="middle" class="sk-sub">갱신된 이미지로 재빌드·재배포</text>
  <path d="M454,46 L510,46" class="sk-line" marker-end="url(#dn-m1)"/>
  <path d="M454,110 L510,110" class="sk-line" marker-end="url(#dn-m1)"/>
  <path d="M454,174 L510,174" class="sk-line" marker-end="url(#dn-m1)"/>
  <rect x="512" y="26" width="200" height="40" rx="8" class="sk-box"/>
  <text x="612" y="51" text-anchor="middle" class="sk-mono">dotnet --list-runtimes</text>
  <rect x="512" y="90" width="200" height="40" rx="8" class="sk-box-muted"/>
  <text x="612" y="115" text-anchor="middle" class="sk-sub">별도 검증 필요</text>
  <rect x="512" y="154" width="200" height="40" rx="8" class="sk-box-muted"/>
  <text x="612" y="179" text-anchor="middle" class="sk-sub">별도 검증 필요</text>
</svg>
<figcaption>호스트에 런타임을 깔았다고 끝나는 배포 방식은 프레임워크 종속 하나뿐이다.</figcaption>
</figure>

## 실무에서 달라지는 점

### 1. 서버 운영자 — 먼저 Brotli 사용 여부를 가려낸다

소스 코드에서 요청 압축 해제 미들웨어, HttpClient의 자동 압축 해제 설정, `System.IO.Compression`의 Brotli 클래스 사용처를 검색해 보는 것이 우선순위를 가르는 첫 단계다. 사용처가 없어도 업데이트는 해야 하지만, "오늘 밤 당장"과 "정기 점검 창"을 나누는 근거가 된다. 심각도는 원문에 없으므로 MSRC 권고문(CVE-2026-111497)에서 직접 확인한다.

### 2. 컨테이너·자체 포함 앱 담당 — 호스트 패치로는 해결되지 않는다

원문이 분명히 말한다. 호스트에 새 런타임이 있어도 자체 포함 앱에 번들된 런타임은 그대로다. 컨테이너도 마찬가지로 MCR의 갱신된 .NET 이미지로 다시 빌드·배포해야 한다. CI 파이프라인에서 베이스 이미지 태그를 고정해 두었다면, 이번 기회에 어떤 서비스가 어떤 태그를 쓰는지 목록을 만들어 보자.

### 3. 검증 — `dotnet --list-runtimes`는 일부만 보여 준다

이 명령은 프레임워크 종속 배포에서만 의미가 있다. 자체 포함 앱과 컨테이너는 각자 런타임을 들고 있어서, 이미지 내부나 배포 산출물에서 따로 확인해야 한다. "서버에서 명령을 쳐 봤더니 최신이라 끝"이라는 보고는 절반만 맞다.

### 4. Linux 서버 — 설치 경로를 확인한다

.NET 8·9를 ASP.NET Core tarball로 설치했다면 소스 빌드와 무관하게 해당한다. 패키지 관리자로 설치한 서버와 tarball로 푼 서버가 섞여 있는지 파악하고, 후자는 갱신된 ASP.NET Core Runtime tarball로 교체한다.

### 5. IT 강사 — 정기 패치와 OOB를 구분해서 가르친다

10/13 정기 릴리스 이후 보름도 안 돼 수정이 보강된 사례다. 교육에서는 "패치 화요일 한 번이면 끝"이 아니라 재배포 권고가 나올 수 있고, 배포 방식마다 패치 절차가 다르다는 점을 강조하면 좋다.

## 이 글의 한계

심각도, 실제 악용 여부, Brotli 이외 두 건의 상세는 원문에 없어 쓰지 않았다. 직접 설치해 확인한 내용도 아니다. 에이전트 서비스를 .NET으로 운영하는 경우의 일반적인 거버넌스 관점은 [에이전트도 '내부자'다](/posts/securing-agentic-insider-guardian-agents/)에서 이어 읽을 수 있다.

## 참고

- [.NET October 2026 Out-of-Band Security Updates](https://devblogs.microsoft.com/dotnet/dotnet-october-2026-oob-security-updates/) — .NET Blog, 2026-10-25
- [CVE-2026-111497 (MSRC)](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-111497)
- [CVE-2026-114148 (MSRC)](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-114148)
- [CVE-2026-70197 (MSRC)](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-70197)
- [CVE-2026-102870 (MSRC)](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2026-102870)
- [.NET 배포 모델 문서](https://learn.microsoft.com/dotnet/core/deploying/)
