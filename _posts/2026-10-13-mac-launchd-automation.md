---
title: "맥미니에서 launchd로 매일 도는 자동화 만들기 — cron 대신 쓰는 이유와 함정"
date: 2026-10-13 11:55:00 +0900
categories: [Dev Notes]
tags: [automation, hands-on]
description: 블로그 강의 글을 매일 아침 8시에 자동 발행하는 잡을 launchd로 걸었다. 원격 스케줄러를 포기한 이유, plist 구성, 로그 남기는 법, 그리고 실제로 겪은 StartInterval·RunAtLoad·PATH 함정을 정리했다.
---

**결과부터.** 맥미니에 launchd 잡을 20개 넘게 돌리고 있다. 오늘 아침에도 08:00:02에 시작해서 08:01:14에 끝난 잡이 이 블로그에 [네트워크 기초 3편](/posts/network-ep03-ip-address/)을 올렸다. 글을 큐에서 꺼내 날짜를 박고, 영상과 포스터를 복사하고, 검사 스크립트를 돌리고, push한 뒤 페이지가 200을 돌려줄 때까지 기다렸다가 텔레그램으로 알려 준다.

이 글은 그 잡 하나를 기준으로, **로컬 파일을 다루는 자동화를 왜 launchd로 거는지**와 실제로 밟은 함정을 적는다.

## 왜 원격 스케줄러가 아니라 launchd인가

처음엔 클라우드 쪽 스케줄러(원격 트리거)에 루틴을 걸었다. 루틴은 "활성" 상태였는데 결과 폴더가 몇 주째 **완전히 비어 있었다.** 원격에서 도니 내 맥의 파일에 쓸 수도, 로컬 스크립트를 부를 수도, 크롬 쿠키를 쓸 수도 없었던 것이다. 당연한 얘기인데 "활성"이라는 표시에 속았다.

<figure class="sketch">
<svg viewBox="0 0 720 190" role="img" aria-label="원격 스케줄러는 클라우드에서 돌아 로컬 디스크와 스크립트에 닿지 않고, launchd는 맥 안에서 돌아 로컬 파일, 스크립트, git, 브라우저 쿠키에 모두 닿는다">
  <defs>
    <marker id="mla-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/></marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 루틴이 어디서 도느냐가 먼저다</text>
  <rect x="8" y="40" width="200" height="120" rx="8" class="sk-box-muted"/>
  <text x="108" y="66" text-anchor="middle" class="sk-label">원격 스케줄러</text>
  <text x="108" y="90" text-anchor="middle" class="sk-sub">클라우드에서 실행</text>
  <text x="108" y="110" text-anchor="middle" class="sk-sub">"활성"인데 결과 폴더 비어 있음</text>
  <text x="108" y="130" text-anchor="middle" class="sk-sub">웹·API 호출만 하는 일에 적합</text>
  <rect x="250" y="30" width="462" height="140" rx="10" class="sk-box-muted"/>
  <text x="481" y="50" text-anchor="middle" class="sk-sub">맥미니</text>
  <rect x="270" y="64" width="150" height="84" rx="8" class="sk-box-accent"/>
  <text x="345" y="92" text-anchor="middle" class="sk-label">launchd</text>
  <text x="345" y="112" text-anchor="middle" class="sk-mono">08:00</text>
  <text x="345" y="130" text-anchor="middle" class="sk-sub">잠자도 깨면 실행</text>
  <path d="M422,106 L462,106" class="sk-line-accent" marker-end="url(#mla-m1)"/>
  <rect x="466" y="64" width="226" height="84" rx="8" class="sk-box"/>
  <text x="579" y="88" text-anchor="middle" class="sk-label">publish_lecture.py</text>
  <text x="579" y="108" text-anchor="middle" class="sk-sub">로컬 영상·글 · git push</text>
  <text x="579" y="126" text-anchor="middle" class="sk-sub">URL 200 확인 · 텔레그램 보고</text>
</svg>
<figcaption>로컬 파일을 만지는 일은 로컬 스케줄러에 건다. 원격은 웹만 보는 일에 쓴다.</figcaption>
</figure>

## cron이 아니라 launchd인 이유

맥미니도 밤에 잠든다. `man launchd.plist`에 이렇게 적혀 있다.

> Unlike cron which skips job invocations when the computer is asleep, launchd will start the job the next time the computer wakes up.

cron은 자는 동안의 실행을 **건너뛰고**, launchd의 `StartCalendarInterval`은 **깨어나면 실행**한다. 여러 번 놓쳤으면 한 번으로 합쳐서 돈다. "매일 1편 발행" 같은 일에는 이게 맞다.

## plist는 이렇게 생겼다

`~/Library/LaunchAgents/com.clara.blog.lecture.plist` (경로는 일부 줄였다)

```xml
<plist version="1.0">
<dict>
  <key>Label</key><string>com.clara.blog.lecture</string>
  <key>ProgramArguments</key>
  <array>
    <string>/bin/bash</string><string>-c</string>
    <string>export PATH=/opt/homebrew/bin:$HOME/.local/bin:/usr/bin:/bin;
      echo "[$(date '+%F %T')] 시작" &gt;&gt; lecture.log;
      /usr/bin/python3 publish_lecture.py &gt;&gt; lecture.log 2&gt;&amp;1;
      echo "[$(date '+%F %T')] 종료 rc=$?" &gt;&gt; lecture.log</string>
  </array>
  <key>StartCalendarInterval</key>
  <dict><key>Hour</key><integer>8</integer><key>Minute</key><integer>0</integer></dict>
  <key>RunAtLoad</key><false/>
</dict>
</plist>
```

```bash
launchctl load ~/Library/LaunchAgents/com.clara.blog.lecture.plist
launchctl list | grep clara        # 등록 확인 (PID, 마지막 종료 코드, Label)
```

오늘 아침 로그는 이 세 줄이 전부였다.

```text
[2026-10-11 08:00:02] 시작
발행 대상: EP.03 network-ep03-ip-address
[2026-10-11 08:01:14] 종료 rc=0
```

## 밟은 함정 세 가지

**1. PATH가 거의 비어 있다.** launchd는 내 셸 설정(.zshrc)을 읽지 않는다. 터미널에선 되던 `ffmpeg`, `gh`, `uvx`가 "command not found"로 죽었다. 그래서 plist에서 `PATH`를 직접 export하고, 파이썬 스크립트 안의 `subprocess`에도 같은 PATH를 넘긴다.

**2. StartInterval이 안 돌았다.** "60초마다"로 건 감시 잡이 설치 직후 1번만 돌고 나흘 동안 `runs = 1`이었다. 같은 맥에서 `StartCalendarInterval` 잡들은 매일 잘 돈다. 원인을 끝까지 파진 못했고, 자주 도는 일은 `StartCalendarInterval`에 분 단위 배열을 넣거나 crontab으로 돌렸다. 참고로 man 페이지에도 StartInterval은 잠든 사이 실행을 놓친다고 적혀 있다.

**3. RunAtLoad를 믿지 말 것.** `RunAtLoad true`로 등록해도 바로 안 뜬 적이 있다. 등록 직후 한 번 돌려 보려면 이렇게 직접 깨운다.

```bash
launchctl kickstart gui/$(id -u)/com.clara.blog.lecture
```

<figure class="sketch">
<svg viewBox="0 0 720 150" role="img" aria-label="잡 스크립트는 실행, 검증, 보고, 정리 네 단계로 나누고 검증에 실패하면 되돌린다">
  <defs>
    <marker id="mla-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/></marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 무인 잡은 "확인하고 알리는 데"까지가 한 단위</text>
  <rect x="8" y="44" width="160" height="70" rx="8" class="sk-box"/>
  <text x="88" y="72" text-anchor="middle" class="sk-label">실행</text>
  <text x="88" y="92" text-anchor="middle" class="sk-sub">큐에서 1편 꺼내기</text>
  <path d="M170,79 L190,79" class="sk-line" marker-end="url(#mla-m2)"/>
  <rect x="194" y="44" width="160" height="70" rx="8" class="sk-box-accent"/>
  <text x="274" y="72" text-anchor="middle" class="sk-label">검증</text>
  <text x="274" y="92" text-anchor="middle" class="sk-sub">검사 실패 → 되돌림</text>
  <path d="M356,79 L376,79" class="sk-line" marker-end="url(#mla-m2)"/>
  <rect x="380" y="44" width="160" height="70" rx="8" class="sk-box-accent"/>
  <text x="460" y="72" text-anchor="middle" class="sk-label">확인·보고</text>
  <text x="460" y="92" text-anchor="middle" class="sk-sub">URL 200 → 텔레그램</text>
  <path d="M542,79 L562,79" class="sk-line" marker-end="url(#mla-m2)"/>
  <rect x="566" y="44" width="146" height="70" rx="8" class="sk-box"/>
  <text x="639" y="72" text-anchor="middle" class="sk-label">정리</text>
  <text x="639" y="92" text-anchor="middle" class="sk-sub">큐 → done 폴더</text>
</svg>
<figcaption>아무도 안 보는 시간에 도는 잡은, 성공했는지를 스스로 확인하고 알려야 한다.</figcaption>
</figure>

## 스크립트 쪽에서 지키는 것

- **검증 실패 시 되돌린다.** 글 검사(check_posts)가 실패하면 만든 파일을 지우고 예외를 던진다. 반쯤 올라간 상태를 남기지 않는다
- **"push 성공"이 아니라 "페이지 200"을 성공으로 본다.** GitHub Pages는 push 뒤 빌드에 1~2분 걸린다. 20초 간격으로 최대 10분까지 확인하고, 실패하면 "Actions 확인 필요"로 따로 알린다
- **같은 걸 두 번 하지 않게.** 발행한 큐 파일은 `done/`으로 옮긴다. 잡이 두 번 돌아도 다음 편이 나갈 뿐 같은 글이 두 번 올라가지 않는다
- **`--dry-run`을 만든다.** 무엇을 올릴지 출력만 하는 옵션이 있으면 큐 순서를 바꾼 뒤 확인하기 쉽다. 실제로 어제 강의 순서를 바꾸고 이걸로 확인했다

## 정리

- 로컬 파일·스크립트·브라우저 쿠키가 필요하면 **launchd**, 웹만 보면 원격 스케줄러
- 정해진 시각은 `StartCalendarInterval`. 잠든 사이 놓친 실행을 깨어나서 해 준다
- PATH는 plist에서 직접, 로그는 시작·종료·rc 세 줄은 꼭
- 등록 후엔 `launchctl list`와 `kickstart`로 한 번 돌려 확인

## 참고

- `man launchd.plist` (macOS 내장 매뉴얼) — StartInterval, StartCalendarInterval 설명
- `man launchctl` — load, list, kickstart
- 관련 글: [MCP 서버를 처음 만들어봤다](/posts/memo-mcp-first-server/)
