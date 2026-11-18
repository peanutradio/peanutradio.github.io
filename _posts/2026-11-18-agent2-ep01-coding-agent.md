---
title: "AI 에이전트 실전 EP.01 — 코딩 에이전트 시대: Claude Code·Codex는 뭘 하나"
date: 2026-11-18 19:51:00 +0900
categories: [Video Lectures, AI 에이전트 실전]
tags: [ai-agent, claude, lecture]
description: 코드를 알려 주는 챗봇과 직접 고치는 코딩 에이전트의 차이, 계획→읽기→고치기→테스트→보고 고리, 사람이 쥐는 권한·승인·되돌리기를 다룬 약 4분 강의와 정리.
---

**코딩 에이전트는 파일을 직접 읽고, 고치고, 실행합니다. 권한과 승인, 되돌리기는 사람이 쥐고 있습니다.** AI 에이전트 실전 시즌의 첫 시간이에요. 코드를 알려 주는 AI에서 직접 고치는 AI로 넘어가면 무엇이 달라지는지, 약 4분 4개 챕터로 정리했습니다.

> 소리를 켜고 보세요. 약 4분짜리 영상입니다.
{: .prompt-tip }

## 강의 보기

{% include embed/youtube.html id='lBP-0LGtMCQ' %}

> 📺 **Peanut Radio AI** 유튜브 채널에서 보기: [채널](https://www.youtube.com/channel/UCvVodonuWF9nFdEJBGZ6vUg) · [AI 에이전트 실전 재생목록](https://www.youtube.com/playlist?list=PLOes6VJrPpAU) — 구독하면 새 강의를 먼저 받아 볼 수 있어요.
{: .prompt-info }

## 오늘의 질문

> 코드를 알려 주는 AI와, 코드를 직접 고치는 AI는 무엇이 다를까요?

답은 **누가 손을 움직이느냐**입니다. 지난 시즌에서는 에이전트, 도구, MCP, 메모리, RAG, 안전까지 개념을 배웠어요. 이번 시즌은 그 개념들이 실제로 일하는 장면을 봅니다.

| 챕터 | 내용 |
|---|---|
| 1. 챗봇에게 묻기 vs 에이전트에게 맡기기 | 옮겨 나르기는 사람 몫 vs 직접 읽고 고치고 실행 |
| 2. 코딩 에이전트의 일하는 고리 | 계획 → 파일 읽기 → 고치기 → 테스트 → 결과 보기 |
| 3. 사람이 쥐고 있는 것 | 권한, 승인, 되돌리기 |
| 4. 맥미니에서 일하는 코딩 에이전트 | 이 채널의 블로그와 강의가 만들어지는 실제 예 |

## 1. 챗봇에게 묻기 vs 에이전트에게 맡기기

챗봇에게 코드를 물어보면 답으로 코드 조각을 받습니다. 그걸 내 파일에 복사해 붙여 넣고, 실행하고, 오류가 나면 또 물어보죠. **옮겨 나르고 실행하는 일은 전부 사람 몫**이에요.

코딩 에이전트는 다릅니다. 내 컴퓨터의 파일을 **직접 읽고, 고치고, 실행까지** 해 봐요. 요리법을 알려 주는 친구와, 부엌에 들어와 직접 요리해 주는 친구의 차이입니다. Claude Code나 Codex 같은 도구가 바로 이런 코딩 에이전트예요.

## 2. 코딩 에이전트의 일하는 고리

코딩 에이전트는 한 번에 끝내지 않고, 고리를 돌며 일합니다.

### 1. 계획과 파일 읽기

먼저 무엇을 할지 계획을 세우고, 관련된 파일을 읽어요.

### 2. 고치고 테스트하기

코드를 고친 다음, 테스트를 실행해 봅니다.

### 3. 실패하면 다시

테스트가 실패하면 오류 메시지를 읽고, 다시 고쳐요. 지난 시즌에 배운 생각 → 행동 → 관찰 고리가 코드 위에서 도는 거예요.

### 4. 통과하면 보고

모두 통과하면, 무엇을 왜 바꿨는지 정리해서 보고합니다. 사람은 그 보고와 바뀐 부분만 확인하면 돼요.

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="코딩 에이전트는 계획, 파일 읽기, 고치기, 테스트 순서로 일하고, 테스트가 실패하면 오류를 읽고 고치기로 돌아가며, 모두 통과하면 무엇을 왜 바꿨는지 사람에게 보고한다">
  <defs>
    <marker id="a1-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="a1-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 고치고, 돌려 보고, 실패하면 다시</text>
  <rect x="10" y="50" width="110" height="54" rx="8" class="sk-box"/>
  <text x="65" y="82" text-anchor="middle" class="sk-label">계획</text>
  <path d="M122,77 L158,77" class="sk-line" marker-end="url(#a1-m1)"/>
  <rect x="160" y="50" width="110" height="54" rx="8" class="sk-box"/>
  <text x="215" y="82" text-anchor="middle" class="sk-label">파일 읽기</text>
  <path d="M272,77 L308,77" class="sk-line" marker-end="url(#a1-m1)"/>
  <rect x="310" y="50" width="110" height="54" rx="8" class="sk-box"/>
  <text x="365" y="82" text-anchor="middle" class="sk-label">고치기</text>
  <path d="M422,77 L458,77" class="sk-line" marker-end="url(#a1-m1)"/>
  <rect x="460" y="50" width="110" height="54" rx="8" class="sk-box-accent"/>
  <text x="515" y="82" text-anchor="middle" class="sk-label">테스트</text>
  <path d="M572,77 L608,77" class="sk-line-accent" marker-end="url(#a1-a1)"/>
  <text x="590" y="68" text-anchor="middle" class="sk-sub">통과</text>
  <rect x="610" y="50" width="100" height="54" rx="8" class="sk-box"/>
  <text x="660" y="74" text-anchor="middle" class="sk-label">결과 보기</text>
  <text x="660" y="92" text-anchor="middle" class="sk-sub">무엇을 · 왜</text>
  <path d="M515,106 L515,150 L365,150 L365,108" class="sk-line" marker-end="url(#a1-m1)"/>
  <text x="440" y="142" text-anchor="middle" class="sk-sub">실패 → 오류를 읽고 다시 고치기</text>
  <path d="M660,106 L660,186" class="sk-line" marker-end="url(#a1-m1)"/>
  <rect x="470" y="190" width="240" height="44" rx="8" class="sk-box"/>
  <text x="590" y="210" text-anchor="middle" class="sk-label">사람</text>
  <text x="590" y="226" text-anchor="middle" class="sk-sub">보고와 바뀐 부분만 확인</text>
  <text x="10" y="200" class="sk-mono">생각 → 행동 → 관찰</text>
  <text x="10" y="218" class="sk-sub">지난 시즌의 고리가 코드 위에서 돈다</text>
</svg>
<figcaption>코딩 에이전트의 진짜 차이는 "다 됐다"고 말하기 전에 테스트로 스스로 확인한다는 데 있다.</figcaption>
</figure>

## 3. 사람이 쥐고 있는 것

손이 빠른 만큼, 사람이 꼭 쥐고 있어야 할 것이 세 가지 있습니다.

### 1. 권한

어느 폴더를 만지고, 어떤 명령을 실행해도 되는지 정해 줍니다.

### 2. 승인

파일을 지우거나 밖으로 내보내기 전에는 먼저 물어보게 하죠.

### 3. 되돌리기

git으로 저장 지점을 남겨 두면, 잘못 고쳐도 그때로 돌아갈 수 있어요.

그럼 무엇을 맡기고 무엇을 직접 챙길까요? 기준은 **확인하기 쉬운가, 되돌릴 수 있는가**입니다.

<figure class="sketch">
<svg viewBox="0 0 720 210" role="img" aria-label="테스트 고치기, 문서 정리, 반복 작업처럼 확인하기 쉬운 일은 코딩 에이전트에게 맡기고, 결제와 돈, 운영 서버, 비밀번호처럼 되돌리기 어려운 일은 사람이 직접 챙긴다">
  <defs>
    <marker id="a1-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 맡길 일과 직접 챙길 일</text>
  <rect x="10" y="36" width="330" height="160" rx="10" class="sk-box-accent"/>
  <text x="175" y="60" text-anchor="middle" class="sk-label">맡겨 볼 일</text>
  <text x="175" y="78" text-anchor="middle" class="sk-sub">확인하기 쉽고, 되돌릴 수 있다</text>
  <rect x="30" y="94" width="290" height="26" rx="6" class="sk-box"/>
  <text x="175" y="112" text-anchor="middle" class="sk-mono">테스트 고치기</text>
  <rect x="30" y="126" width="290" height="26" rx="6" class="sk-box"/>
  <text x="175" y="144" text-anchor="middle" class="sk-mono">문서 정리</text>
  <rect x="30" y="158" width="290" height="26" rx="6" class="sk-box"/>
  <text x="175" y="176" text-anchor="middle" class="sk-mono">반복 작업</text>
  <rect x="380" y="36" width="330" height="160" rx="10" class="sk-box"/>
  <text x="545" y="60" text-anchor="middle" class="sk-label">직접 챙길 일</text>
  <text x="545" y="78" text-anchor="middle" class="sk-sub">되돌리기 어렵다</text>
  <rect x="400" y="94" width="290" height="26" rx="6" class="sk-box-muted"/>
  <text x="545" y="112" text-anchor="middle" class="sk-mono">결제 · 돈</text>
  <rect x="400" y="126" width="290" height="26" rx="6" class="sk-box-muted"/>
  <text x="545" y="144" text-anchor="middle" class="sk-mono">운영 서버</text>
  <rect x="400" y="158" width="290" height="26" rx="6" class="sk-box-muted"/>
  <text x="545" y="176" text-anchor="middle" class="sk-mono">비밀번호</text>
</svg>
<figcaption>에이전트에게 무엇을 맡길지는 실력이 아니라, 잘못됐을 때 되돌릴 수 있느냐로 정한다.</figcaption>
</figure>

## 4. 맥미니에서 일하는 코딩 에이전트

실제 예를 보여 드릴게요. 이 채널의 블로그 글과 강의 영상은 맥미니 한 대에서 만들어집니다. 그 맥미니에서 코딩 에이전트가 일하고 있어요.

- **블로그 초안 쓰기**

  정해진 시간에 자료를 모아 블로그 초안을 씁니다.

- **강의 장면 코드**

  지금 보시는 강의의 장면 코드도 짜고 고쳐요.

- **스스로 점검**

  영상이 만들어지면 화면 겹침과 길이를 스스로 점검합니다.

그래도 **공개 게시와 비용이 드는 일은 제가 확인하고 승인합니다.** 에이전트는 손이 되고, 사람은 방향과 마지막 확인을 맡는 셈이에요. 이번 시즌에서는 이런 실전 장면을 하나씩 열어 볼게요.

## 한 줄 정리

> 코딩 에이전트는 파일을 **직접 읽고·고치고·실행**합니다. **권한 · 승인 · 되돌리기**는 사람이 쥡니다.
{: .prompt-info }

## 퀴즈

1. 코드를 고친 뒤 코딩 에이전트가 하는 일은? (테스트를 실행해 결과를 본다 / 바로 다 됐다고 말한다)
2. 잘못 고쳤을 때 이전 저장 지점으로 돌아가게 해 주는 도구는? (git / 화면 캡처)

<details markdown="1">
<summary>정답 보기</summary>

1. **테스트를 실행해 결과를 본다** — 실패하면 오류를 읽고 다시 고치는 고리를 돌아요.
2. **git(깃)** — 저장 지점을 남겨 두면 언제든 되돌릴 수 있어요.

</details>

## 집에서 해보기: 맡길 일 · 직접 챙길 일 나누기

종이 한 장에 두 칸을 그려 보세요.

1. 왼쪽 칸에는 내 컴퓨터 일 가운데 코딩 에이전트에게 맡겨 볼 일 세 개를 적습니다.
2. 오른쪽 칸에는 끝까지 직접 챙길 일 세 개를 적습니다.

> 확인하기 쉽고 되돌릴 수 있는 일은 맡기고, 돈·삭제·공개는 직접 챙겨요.
{: .prompt-tip }

## 다음 편

- AI 에이전트 실전 EP.02 — 모델 바깥의 소프트웨어, 하네스 (발행 예정)

  같은 AI도 무엇에 담느냐에 따라 달라져요. AI 모델을 감싸서 실제로 일하게 만드는 바깥 소프트웨어를 알아봅니다. 미리 읽어 보고 싶다면 [모델 바깥의 소프트웨어, AI 하네스를 설계하는 4가지 축](/posts/ai-harness-four-parts/)을 먼저 보셔도 좋아요.

- 시리즈 전체 목록: [AI 에이전트 실전](/categories/ai-에이전트-실전/)
