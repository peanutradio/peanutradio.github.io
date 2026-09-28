---
title: "'에이전틱 OS' 논쟁 — Windows는 사람과 에이전트를 함께 섬길 수 있나"
date: 2026-09-27 09:50:00 +0900
categories: [IT News]
tags: [windows, agentic-os, ai-agent, copilot, microsoft]
description: 에이전틱 OS라는 한 문장에 반발이 쏟아진 지 10개월. Windows 수장은 표현을 바꿨지만 방향은 그대로고, OS라는 단어는 이제 Copilot이 가져갔습니다.
---

![Microsoft 레드먼드 캠퍼스 92번 건물](/assets/img/posts/windows-agentic-os-debate.jpg)
_사진: [Coolcaesar](https://commons.wikimedia.org/wiki/File:Building92microsoft.jpg), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0) — 2016년 촬영 자료 사진_

결론부터 적습니다. **Microsoft는 Windows를 에이전트용 OS로 바꾸는 걸 포기하지 않았습니다. 이름만 내려놨습니다.**
Windows 수장 Pavan Davuluri는 이제 "에이전틱 OS"라는 말 대신 "사용자는 계속 사람이고, 여기에 에이전트 워크로드가 더해진다"고 말합니다.
그리고 같은 달, "OS"라는 단어는 Windows가 아니라 **Copilot** 쪽에서 다시 등장했습니다.

## 발단: 한 문장짜리 트윗

2025년 11월 10일(UTC), Davuluri가 Ignite를 앞두고 X에 이렇게 썼습니다.

> "Windows is evolving into an agentic OS, connecting devices, cloud, and AI to unlock intelligent productivity and secure work anywhere."

Windows Central(Zac Bowden, 2025-11-11)에 따르면 답글 대다수가 부정적이었습니다.
"Stop this non-sense. No one wants this", "Bro, straight up, nobody wants this" 같은 반응이 이어졌고,
"압도적으로 부정적인 피드백을 받는데 왜 계속 밀어붙이느냐"는 질문도 나왔습니다.

반발의 핵심은 AI 자체보다 **신뢰**였습니다. 같은 기사는 Microsoft 계정 강제, OneDrive·Copilot 유도,
업데이트 때마다 생기는 안정성 문제를 짚으며 "AI가 Windows의 지금 문제에 대한 답이 아니다"라는 여론을 전했습니다.
Windows Latest(2025-11-14)는 게시물이 70만 뷰를 넘기고 답글이 좋아요보다 많아지자(당시 좋아요 247, 답글 487)
Davuluri가 **답글을 막았다**고 보도했습니다.

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="타임라인: 2025년 11월 에이전틱 OS 트윗과 반발, 2026년 3월 품질 약속, 2026년 9월 10일 사람과 에이전트를 함께 섬긴다는 재정의, 9월 25일 Copilot을 새로운 업무용 OS로 소개">
  <defs>
    <marker id="os-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① "OS"라는 단어가 옮겨 간 10개월</text>
  <path d="M16,110 L704,110" class="sk-line" marker-end="url(#os-m1)"/>

  <rect x="8" y="40" width="160" height="52" rx="8" class="sk-box-muted"/>
  <text x="88" y="62" text-anchor="middle" class="sk-label">"agentic OS"</text>
  <text x="88" y="80" text-anchor="middle" class="sk-sub">Windows 수장 트윗</text>
  <text x="88" y="132" text-anchor="middle" class="sk-mono">2025-11-10</text>
  <text x="88" y="152" text-anchor="middle" class="sk-sub">반발 → 답글 차단</text>

  <rect x="186" y="40" width="160" height="52" rx="8" class="sk-box"/>
  <text x="266" y="62" text-anchor="middle" class="sk-label">품질부터</text>
  <text x="266" y="80" text-anchor="middle" class="sk-sub">EVP 승진 후 약속</text>
  <text x="266" y="132" text-anchor="middle" class="sk-mono">2026-03</text>

  <rect x="364" y="40" width="160" height="52" rx="8" class="sk-box-accent"/>
  <text x="444" y="62" text-anchor="middle" class="sk-label">사람 + 에이전트</text>
  <text x="444" y="80" text-anchor="middle" class="sk-sub">GeekWire 인터뷰</text>
  <text x="444" y="132" text-anchor="middle" class="sk-mono">2026-09-10</text>
  <text x="444" y="152" text-anchor="middle" class="sk-sub">"에이전틱 OS"라 부르지 않음</text>

  <rect x="542" y="40" width="170" height="52" rx="8" class="sk-box-accent"/>
  <text x="627" y="62" text-anchor="middle" class="sk-label">"new OS for work"</text>
  <text x="627" y="80" text-anchor="middle" class="sk-sub">Nadella — Copilot</text>
  <text x="627" y="132" text-anchor="middle" class="sk-mono">2026-09-25</text>
  <text x="627" y="152" text-anchor="middle" class="sk-sub">Windows 언급 없음</text>

  <text x="360" y="200" text-anchor="middle" class="sk-sub">Windows는 표현을 낮추고, "OS"라는 야심은 Copilot 쪽으로 옮겨 갔다</text>
</svg>
<figcaption>방향이 바뀐 게 아니라 <strong>말하는 방식과 말하는 주체</strong>가 바뀌었습니다.</figcaption>
</figure>

## Microsoft의 입장: "그냥 에이전틱 OS라고 부르지 마세요"

10개월 뒤 GeekWire의 Mary Jo Foley가 Davuluri를 인터뷰했습니다(2026-09-10). 기사 소제목이 "Just Don't Call It an 'Agentic OS'"입니다.
기사에 따르면 그는 비판 때문에 유턴한 게 아니라 **말하는 방식을 바꿨습니다.**

> "The user of Windows going forward will continue to be users … but it's also going to add these agentic workloads."

제가 주목한 건 그다음입니다. Davuluri는 UX/UI보다 **시스템 수준을 먼저** 손본다고 했습니다.
에이전트를 네이티브로 만들고 돌리기 위한 저수준 기능, 즉 **보안·ID·거버넌스·관측성·성능** 같은 "프리미티브"입니다.
GeekWire가 전한 구체적인 내용은 이렇습니다.

- 에이전트에게 **로컬 ID** 또는 **Entra 기반 클라우드 ID**를 부여할 수 있음
- 신뢰할 수 없는 코드를 샌드박스·VM에서 돌리는 **Microsoft Execution Containers** 초기 프리뷰
- 파일 시스템, 보안 모델, PowerShell 같은 기반 요소가 영향을 받을 가능성(GeekWire의 전망)

<figure class="sketch">
<svg viewBox="0 0 720 300" role="img" aria-label="Windows 위에 사람 사용자와 에이전트 워크로드가 나란히 있고, 둘 다 보안, ID, 거버넌스, 관측성, 성능이라는 공통 프리미티브 위에서 동작한다">
  <defs>
    <marker id="os-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="os-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 한 OS, 두 종류의 사용자</text>

  <rect x="60" y="36" width="260" height="60" rx="8" class="sk-box"/>
  <text x="190" y="62" text-anchor="middle" class="sk-label">사람 사용자</text>
  <text x="190" y="82" text-anchor="middle" class="sk-sub">개발자 · 게이머 · 기업 · 일반 업무</text>

  <rect x="400" y="36" width="260" height="60" rx="8" class="sk-box-accent"/>
  <text x="530" y="62" text-anchor="middle" class="sk-label">에이전트 워크로드</text>
  <text x="530" y="82" text-anchor="middle" class="sk-sub">새로 더해지는 사용자</text>

  <path d="M190,98 L190,132" class="sk-line" marker-end="url(#os-m2)"/>
  <path d="M530,98 L530,132" class="sk-line-accent" marker-end="url(#os-a2)"/>

  <rect x="20" y="136" width="680" height="96" rx="10" class="sk-box"/>
  <text x="360" y="156" text-anchor="middle" class="sk-title">Windows 플랫폼 프리미티브 — UX보다 먼저 손보는 층</text>
  <rect x="36" y="168" width="120" height="48" rx="6" class="sk-box"/>
  <text x="96" y="190" text-anchor="middle" class="sk-label">보안</text>
  <text x="96" y="206" text-anchor="middle" class="sk-sub">Execution Containers</text>
  <rect x="168" y="168" width="120" height="48" rx="6" class="sk-box"/>
  <text x="228" y="190" text-anchor="middle" class="sk-label">ID</text>
  <text x="228" y="206" text-anchor="middle" class="sk-sub">로컬 ID · Entra</text>
  <rect x="300" y="168" width="120" height="48" rx="6" class="sk-box"/>
  <text x="360" y="196" text-anchor="middle" class="sk-label">거버넌스</text>
  <rect x="432" y="168" width="120" height="48" rx="6" class="sk-box"/>
  <text x="492" y="196" text-anchor="middle" class="sk-label">관측성</text>
  <rect x="564" y="168" width="120" height="48" rx="6" class="sk-box"/>
  <text x="624" y="196" text-anchor="middle" class="sk-label">성능</text>

  <rect x="20" y="244" width="680" height="40" rx="8" class="sk-box-muted"/>
  <text x="360" y="268" text-anchor="middle" class="sk-sub">파일 시스템 · 보안 모델 · PowerShell 등 기반 요소 (GeekWire의 전망)</text>
</svg>
<figcaption>에이전트를 '또 하나의 사용자'로 받아들이려면, 화면보다 <strong>신원과 통제의 층</strong>이 먼저 준비돼야 한다는 얘기입니다.</figcaption>
</figure>

## 그 사이 "OS"는 Copilot으로 갔다

9월 25일, Satya Nadella가 X에 새 Copilot을 소개하며 이렇게 썼습니다.

> "We're building Copilot as a new OS for work that spans every model, every form factor, and every task."

Windows Latest(2026-09-26)는 이 글이 **Windows 11을 한 번도 언급하지 않았다**는 점, 발표 어디에도 Windows 11 UI 목업이 없었다는 점을 짚었습니다.
그러면서 1년 전 "에이전틱 OS" 때와 달리 이번에는 **큰 반발이 보이지 않았다**고 적었습니다.
Home·Code·Autopilot이 무엇인지는 [Copilot 대개편 글](/posts/copilot-home-code-autopilot/)에 따로 정리했습니다.

흥미로운 건 Autopilot의 설명입니다. Windows Latest에 따르면 Autopilot은 **자체 ID, 메모리, 컴퓨터, 작업 공간**을 갖고,
권한·감사·거버넌스는 Agent 365를 통해 조직이 통제합니다.
Davuluri가 Windows에 깔겠다는 프리미티브와 거의 같은 목록입니다. 이건 제 해석이지만,
**Copilot은 클라우드 쪽에서, Windows는 기기 쪽에서** 같은 문제를 풀고 있는 것으로 보입니다.

## 짚고 갈 것

**Microsoft 쪽 논리도 이해는 됩니다.** 에이전트가 사람처럼 앱을 쓰고 파일을 만지려면, 누가 무엇을 했는지 구분하고 막을 수 있는 층이 OS에 있어야 합니다.
화면보다 ID·격리·감사를 먼저 한다는 순서는 에이전트를 만드는 입장에서 옳다고 봅니다.

**반발도 정당합니다.** 2025년 11월의 분노는 "AI가 싫다"보다 "기본부터 고쳐라"에 가까웠습니다.
GeekWire에 따르면 Davuluri는 EVP가 된 뒤 품질·신뢰성 개선을 공개적으로 약속했고, 이후 실제로 결과를 내고 있다는 평가도 있습니다.
결국 이름을 바꾸는 것보다 **그 약속을 계속 지키는지**가 이 논쟁의 답을 정할 것 같습니다.

---

## 참고

- ["We're building Copilot as a new OS," says Satya Nadella, even as Microsoft strips it from Windows 11](https://www.windowslatest.com/2026/09/26/were-building-copilot-as-a-new-os-says-satya-nadella-even-as-microsoft-strips-it-from-windows-11/) — Windows Latest (Abhijith M B), 2026-09-26
- [Windows president says platform is "evolving into an agentic OS," gets cooked in the replies](https://www.windowscentral.com/microsoft/windows-11/windows-president-confirms-os-will-become-ai-agentic-generates-push-back-online) — Windows Central (Zac Bowden), 2025-11-11
- [Microsoft 2.5: EVP Pavan Davuluri wants to remake Windows for both human and agent users](https://www.geekwire.com/2026/microsoft-2-5-evp-pavan-davuluri-wants-to-remake-windows-for-both-human-and-agent-users/) — GeekWire (Mary Jo Foley), 2026-09-10
- [Windows 11 agentic OS AI upgrade faces backlash, Microsoft responds by closing replies](https://www.windowslatest.com/2025/11/14/windows-11-agentic-os-ai-upgrade-faces-backlash-microsoft-responds-by-closing-replies/) — Windows Latest, 2025-11-14
- [Pavan Davuluri의 X 게시물](https://x.com/pavandavuluri/status/1987942909635854336) — 2025-11-10
- [Satya Nadella의 X 게시물](https://x.com/satyanadella/status/2103455884366188544) — 2026-09-25

이 글은 위 기사와 게시물의 내용을 정리하고 재구성한 것입니다. 다이어그램은 직접 그렸습니다.
