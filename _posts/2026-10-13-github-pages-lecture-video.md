---
title: "GitHub Pages 블로그에 강의 영상 올리기 — HTML 임베드를 버리고 mp4로 간 이유"
date: 2026-10-13 18:55:00 +0900
categories: [Dev Notes]
tags: [github, lecture, automation]
description: 3D 인터랙티브 강의를 iframe으로 넣었더니 소리만 나고 화면이 넘어가지 않았다. mp4와 video 태그로 바꾸고, 포스터·챕터·용량까지 정리한 과정을 남긴다.
---

**결과부터.** [네트워크 기초](/categories/네트워크-기초/) 강의를 이 블로그에 올리면서, 처음엔 강의 HTML 파일을 iframe으로 넣었다가 **mp4 + `<video>` 태그로 갈아탔다.** 지금은 편마다 25~32MB짜리 mp4 하나와 포스터 이미지 하나만 올린다. 휴대폰에서도 잘 나오고, 글 본문의 챕터 표와 영상 시간이 맞는다.

## 처음 시도: 강의 HTML을 iframe으로

강의는 3D 장면과 내레이션이 한 HTML 파일(편당 4~6MB, 음성까지 파일 안에 들어 있다)에 담겨 있다. 브라우저에서 열면 바로 재생되니, 블로그에도 그대로 넣으면 되겠다고 생각했다.

```html
<iframe src="/assets/lectures/network-ep02/" allow="autoplay; fullscreen" allowfullscreen></iframe>
```

배포까지는 잘 됐다. 그런데 블로그에서 재생해 보니 **내레이션 소리는 나오는데 화면이 다음 장면으로 넘어가지 않았다.** 원인을 끝까지 파진 않았다. 로컬에서 파일을 직접 열면 정상이고, 블로그 안 iframe에서만 그랬다. 3D 렌더링과 소리 타이밍을 브라우저가 iframe 안에서 다르게 다루는 쪽을 의심하고 있다.

여기서 고민했다. iframe 문제를 파고들지, 아니면 이미 같이 만들어 둔 mp4를 쓸지. **보는 사람에게 필요한 건 "잘 나오는 영상"이지 인터랙티브 HTML이 아니다.** mp4로 갔다.

<figure class="sketch">
<svg viewBox="0 0 720 190" role="img" aria-label="HTML 임베드는 브라우저와 기기마다 동작이 달라 위험하고, mp4는 모든 기기의 기본 플레이어가 재생해 안정적이다">
  <text x="0" y="14" class="sk-title">① 무엇을 올릴 것인가</text>
  <rect x="8" y="36" width="340" height="140" rx="8" class="sk-box-muted"/>
  <text x="178" y="60" text-anchor="middle" class="sk-label">강의 HTML (iframe)</text>
  <text x="178" y="86" text-anchor="middle" class="sk-sub">인터랙티브, 파일은 작음(4~6MB)</text>
  <text x="178" y="106" text-anchor="middle" class="sk-sub">브라우저·기기마다 동작이 다를 수 있음</text>
  <text x="178" y="126" text-anchor="middle" class="sk-sub">실제로: 소리만 나고 화면 정지</text>
  <text x="178" y="152" text-anchor="middle" class="sk-label">→ 디버깅 비용이 큼</text>
  <rect x="372" y="36" width="340" height="140" rx="8" class="sk-box-accent"/>
  <text x="542" y="60" text-anchor="middle" class="sk-label">mp4 (video 태그)</text>
  <text x="542" y="86" text-anchor="middle" class="sk-sub">파일은 큼(25~32MB)</text>
  <text x="542" y="106" text-anchor="middle" class="sk-sub">기본 플레이어라 어디서나 같음</text>
  <text x="542" y="126" text-anchor="middle" class="sk-sub">유튜브에도 같은 파일을 올림</text>
  <text x="542" y="152" text-anchor="middle" class="sk-label">→ 한 번 만들면 끝</text>
</svg>
<figcaption>블로그에 올리는 건 "보는 사람 기기에서 확실히 재생되는 것"이어야 한다.</figcaption>
</figure>

## 지금 쓰는 마크업

```html
<video controls preload="metadata" playsinline
       poster="/assets/lectures/network-ep03/poster.jpg"
       style="width:100%;border-radius:12px">
  <source src="/assets/lectures/network-ep03/network-ep03.mp4" type="video/mp4">
</video>
```

- `preload="metadata"`: 페이지를 열자마자 30MB를 받지 않는다. 길이와 첫 정보만 받는다
- `playsinline`: 아이폰에서 누르자마자 전체 화면으로 튀지 않게
- `poster`: 재생 전 화면. 이게 없으면 검은 네모가 보인다

![블로그 글 안의 강의 영상](/assets/img/posts/github-pages-lecture-video/01-player.png)
_① 포스터(영상 8초 지점 캡처) ② 브라우저 기본 컨트롤. 3분 21초짜리 3편_

포스터는 ffmpeg로 영상 8초 지점을 뽑는다. 0초는 대부분 빈 화면이라 8초쯤이 제목 카드가 가장 잘 보였다.

```bash
ffmpeg -ss 8 -i network-ep03.mp4 -frames:v 1 -vf scale=1280:-1 -q:v 4 poster.jpg
```

## 챕터 표는 영상 타임스탬프에서

영상과 같이 나오는 `*.mp4.log.json`에 내레이션 문장마다 시작 시각이 들어 있다. 이걸로 "몇 분 몇 초에 어느 챕터가 시작하는지"를 계산해서, 글 본문의 챕터 표와 유튜브 설명란 챕터, 자막(SRT)을 한 번에 만든다. 손으로 시간을 재지 않으니 영상을 다시 렌더링해도 스크립트만 다시 돌리면 된다. 실제로 어제 3~6편이 다시 렌더링됐을 때 이걸로 몇 분 만에 맞췄다.

## 용량은 미리 계산해 둔다

GitHub에는 크기 제한이 있다([About large files on GitHub](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github), [GitHub Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits)).

- 파일 하나가 **50MiB를 넘으면 경고, 100MiB를 넘으면 push가 막힌다**
- Pages 사이트는 **1GB 이하**, 대역폭은 월 100GB 정도가 기준이다

<figure class="sketch">
<svg viewBox="0 0 720 140" role="img" aria-label="편당 30MB로 12편이면 약 360MB로 1GB 한도의 3분의 1을 쓰며, 파일 하나는 50MB 경고선 아래에 있다">
  <text x="0" y="14" class="sk-title">② 12편을 다 올리면</text>
  <rect x="8" y="50" width="704" height="34" rx="6" class="sk-box"/>
  <rect x="8" y="50" width="253" height="34" rx="6" class="sk-box-accent"/>
  <text x="134" y="72" text-anchor="middle" class="sk-label">강의 12편 ≈ 360MB</text>
  <text x="486" y="72" text-anchor="middle" class="sk-sub">나머지 글·이미지 여유</text>
  <text x="712" y="108" text-anchor="end" class="sk-sub">Pages 사이트 한도 1GB</text>
  <text x="8" y="108" class="sk-sub">편당 25~32MB — 파일 하나 50MiB 경고선 아래</text>
</svg>
<figcaption>지금 속도면 괜찮지만, 시리즈가 30편을 넘으면 영상은 다른 곳으로 옮겨야 한다.</figcaption>
</figure>

지금 강의 폴더는 3편까지 올린 상태로 95MB다. 시리즈가 늘면 영상은 유튜브에만 두고 블로그엔 유튜브 임베드를 쓰는 쪽으로 옮길 계획이다. 같은 mp4를 유튜브에도 올리니까 옮기기는 어렵지 않다.

## 정리

- 블로그에 넣을 영상은 **mp4 + `<video controls preload="metadata" playsinline poster>`**
- 인터랙티브 HTML은 "직접 열어 보는 링크"로만 두고, 본문에는 넣지 않는다
- 챕터·자막은 영상 로그에서 자동으로 만들고, 용량은 GitHub 한도 기준으로 미리 계산한다

## 참고

- [About large files on GitHub](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github)
- [GitHub Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits)
- [MDN — The Video Embed element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)
- 관련 글: [맥미니에서 launchd로 매일 도는 자동화 만들기](/posts/mac-launchd-automation/)
