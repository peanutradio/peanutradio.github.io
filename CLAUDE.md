# 이 블로그에 글을 쓸 때 지켜야 할 것

> peanutradio.github.io — 박찬후(Daniel Park)의 기술 블로그.
> **이 파일 하나만 읽으면 어느 기기에서든 글을 쓰고 발행할 수 있다.**
> 어떤 환경(Claude Code / Claude Desktop / 웹)에서 작업하든 이 규칙을 따른다.

---

## 0. 블로그 정체성

Microsoft 생태계 위에서 **AI 에이전트를 직접 만드는** 사람의 기록.
잘 정리된 문서가 아니라 **직접 부딪혀서 알게 된 것**을 쓴다.

| 항목 | 값 |
|---|---|
| URL | https://peanutradio.github.io/ |
| 엔진 | Jekyll + `jekyll-theme-chirpy` ~> 7.5 |
| 배포 | `main` push → GitHub Actions → GitHub Pages (약 1~2분) |
| 글 경로 | `_posts/YYYY-MM-DD-{영문-slug}.md` |
| 초안 | `_drafts/` (빌드에서 제외됨) |
| 이미지 | `assets/img/posts/` |

---

## 1. 카테고리 — 5개. 이 밖으로 나가지 않는다

| 카테고리 | 범위 |
|---|---|
| `AI Agent` | MCP, A2A, 멀티에이전트 오케스트레이션, 설계 패턴 — **플랫폼 무관 개념** |
| `Azure AI` | Microsoft Foundry(구 Azure AI Foundry), Azure OpenAI, AI Search, RAG |
| `Copilot Studio` | 로우코드 에이전트, M365 Copilot 확장, 커넥터, 거버넌스 |
| `IT News` | 신기능·GA·릴리스·라이선스·비용 변화 등 **업계 소식 요약·재구성** |
| `Dev Notes` | 트러블슈팅, 환경 설정, 짧은 실무 메모, **직접 한 핸즈온 기록** |

**어디에 넣을지 헷갈릴 때**

1. 특정 플랫폼에 묶인 심층 내용 → `Azure AI` / `Copilot Studio`
2. 플랫폼 무관한 개념·프로토콜 → `AI Agent`
3. 릴리스 노트·주간 요약·GA 발표 같은 **소식 모양** → `IT News`
   - "agent"라는 단어가 있어도 소식 모양이면 `AI Agent`가 아니라 `IT News`다
4. 내가 직접 해보고 남기는 기록 → `Dev Notes`

> `IT News`는 **요약·재구성 글만.** 외부 글을 그대로 옮기지 않는다. 원문을 읽고 한국어로 다시 쓴다.
>
> **새 최상위 카테고리는 사용자 승인 없이 만들지 않는다.**
> 한 카테고리에 12편 넘게 쌓이면 하위 분리를 제안한다.

---

## 2. Front matter

```yaml
---
title: "제목 — 60자 이내"
date: 2026-09-07 08:10:00 +0900
categories: [AI Agent]
tags: [mcp, api, ai-agent]
description: 한 문장 요약. 160자 이내.
---
```

- `categories`는 위 5개 중 **하나만**. 대소문자·띄어쓰기까지 정확히.
- `tags`는 소문자 영문 3~6개.

### 🚨 `date`를 반드시 현재 시각 이전으로

**Jekyll은 미래 날짜 글을 조용히 빌드에서 제외한다.** 에러도 경고도 없이 그냥 404가 난다.

반드시 이 순서로 한다.

```bash
date "+%Y-%m-%d %H:%M"      # 1) 지금 몇 시인지 확인
# 2) 그보다 이전 시각을 date에 넣는다. 절대 반올림하지 말 것.
```

> 2026-09-07 하루에 이 실수를 **두 번** 했다.
> 08:26에 08:30으로 넣어서 글이 사라졌다. 5분 뒤로 미루는 것도 미래다.

---

## 3. 발행 절차

```bash
git pull --rebase origin main

# 글 작성: _posts/YYYY-MM-DD-slug.md

# 발행 전 검사 (Claude Code / 로컬 맥에서만 가능)
bash ~/.claude/clara/agents/blog/skills/blog-publisher/scripts/check_posts.sh .

git add _posts/... && git commit && git push origin main
```

**발행 후 반드시 확인한다. 확인 전에는 링크를 알리지 않는다.**

```bash
gh run list --limit 1                                  # Actions 성공?
curl -sI https://peanutradio.github.io/posts/{slug}/   # 200?
```

Actions를 못 쓰는 환경이라면 브라우저로 저장소 Actions 탭과 실제 URL을 직접 확인한다.

---

## 4. 글쓰기 규칙

**분량·톤**
- 본문 1,200~2,500자 한국어
- **1인칭.** 사용자(박찬후)가 쓴 글이다. AI가 썼다는 티를 내지 않는다
- 결론을 앞에 둔다. 서론에서 뜸들이지 않는다
- 공식 문서 번역체·클릭베이트 금지

**출처 — 예외 없다**
- 수치·인용·통계는 **반드시 출처 명시.** 없으면 쓰지 않는다
- 글 마지막에 `## 참고` 섹션. 원본 링크를 전부 남긴다
- 영상·기사 요약글이면 "이 글은 아래 내용을 정리하고 재구성한 것"을 명시한다
- **제목·설명만 보고 쓰지 않는다.** 자막 전문 / 원문 전문을 확보한 뒤 쓴다

**핸즈온 글**
- 튜토리얼 번역이 아니라 **실습 기록**이다
- 순서: 무엇을 만들었나(결과 먼저) → 사전 준비 → 실제 단계 → 비용 → 써보고 든 생각
- **막힌 지점과 해결 방법을 반드시 포함한다.** 매끄럽기만 한 글은 공식 문서로 충분하다

---

## 5. 다이어그램 — 인라인 SVG로 그린다

**중요한 개념·아키텍처는 그림으로 그린다.** 이미지 파일·외부 서비스·mermaid를 쓰지 않는다.

```html
<figure class="sketch">
<svg viewBox="0 0 720 300" role="img" aria-label="스크린리더용 설명">
  <defs>
    <marker id="xx-m" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 다이어그램 제목</text>
  <rect x="8" y="60" width="160" height="52" rx="8" class="sk-box"/>
  <text x="88" y="84" text-anchor="middle" class="sk-label">라벨</text>
  <text x="88" y="102" text-anchor="middle" class="sk-sub">보조 설명</text>
  <path d="M170,86 L210,86" class="sk-line" marker-end="url(#xx-m)"/>
</svg>
<figcaption>이 그림이 말하는 핵심 한 문장.</figcaption>
</figure>
```

| 클래스 | 용도 |
|---|---|
| `.sketch` | `<figure>` 래퍼 — 카드 배경·테두리·모바일 가로 스크롤 |
| `.sk-box` | 기본 박스 |
| `.sk-box-accent` | 강조 박스 (cyan 테두리 + 옅은 배경) |
| `.sk-box-muted` | 점선 박스 — 대체되는 것, 사라질 것 |
| `.sk-label` | 13px 굵은 라벨 |
| `.sk-sub` | 11px 보조 텍스트 |
| `.sk-mono` | 코드·식별자 (cyan, JetBrains Mono) |
| `.sk-title` | 다이어그램 번호·제목 |
| `.sk-line` / `.sk-line-accent` | 연결선 / 강조 연결선 |
| `.sk-fill-muted` / `.sk-fill-accent` | 화살촉 마커 채우기 |

**반드시 지킬 것**
- `viewBox` 폭은 **720 고정**. 높이만 내용에 맞춘다
- **색을 직접 쓰지 않는다.** `fill="#0ea5e9"` 같은 건 금지 — 다크/라이트 대응이 깨진다. `sk-*` 클래스만 쓴다
- marker `id`는 **다이어그램마다 다르게** (`xx-m1`, `yy-a2`…). 같은 페이지에서 충돌한다
- `role="img"` + `aria-label` 필수
- `<figcaption>`에는 그림 설명이 아니라 **그림이 말하는 주장**을 쓴다
- 글 하나에 2~4개. 억지로 넣지 않는다

**참고 구현**: `_posts/2026-09-07-mcp-vs-api.md` (스케치 4종),
`_posts/2026-09-07-copilot-studio-workflows-email.md` (스케치 2종 + 스크린샷 슬롯)

### 스크린샷 자리표시자

당장 이미지가 없으면 슬롯을 남긴다. 나중에 교체하기 쉽다.

```html
<figure class="shot">
  <div class="shot-icon">🖼</div>
  <div class="shot-what">어떤 화면이 들어갈 자리인지</div>
  <div class="shot-time">영상 09:43</div>
  <figcaption>보충 설명 (선택)</figcaption>
</figure>
```

교체할 때는 이 블록 전체를 `![설명](/assets/img/posts/파일명.png)`으로 바꾼다.

---

## 5.5 사진 — IT News 글에는 가급적 넣는다

사용자 요청 사항이다. **`IT News` 글에는 사진을 한 장 넣는 것을 기본으로 한다.**
글 맨 위, 제목 바로 아래에 대표 이미지로 배치한다.

### 🚨 기사 사진을 그대로 가져오지 않는다

뉴스 기사에 붙은 사진은 대부분 **Getty / AP / Reuters / 매체 자체 촬영본**이고
전부 라이선스가 걸려 있다. Getty는 무단 사용에 실제로 청구서를 보낸다.
기사에 있다고 해서 쓸 수 있는 사진이 아니다.

### 쓸 수 있는 것 — 이 순서로 찾는다

**① 오픈 라이선스 (권장)**
Wikimedia Commons에서 CC BY / CC BY-SA / CC0 / Public domain 을 찾는다.

```bash
curl -s "https://commons.wikimedia.org/w/api.php?action=query&generator=search\
&gsrsearch=<검색어>&gsrnamespace=6&gsrlimit=5&prop=imageinfo\
&iiprop=url|extmetadata&format=json"
```
응답의 `LicenseShortName`, `Artist`, `LicenseUrl`, `descriptionurl` 을 반드시 기록한다.

> 매체 자체 행사 사진이 Commons에 CC로 올라와 있는 경우가 많다.
> 예: TechCrunch가 자사 Disrupt 행사 사진을 CC BY 2.0으로 공개.

**② 회사 공식 프레스·브랜드 자산** — 사용 조건을 확인한 경우만

**③ 직접 그린 SVG** — 위 둘이 없으면 이게 답이다. 스케치가 사진보다 나은 경우도 많다

### 저장과 표기

```bash
# 폭 1200, 품질 82로 재인코딩해서 저장
sips -Z 1200 -s format jpeg -s formatOptions 82 원본.jpg \
  --out assets/img/posts/{slug}.jpg
```

본문에는 이렇게 넣는다. **크레딧은 예외 없이 붙인다.**

```markdown
![대체 텍스트](/assets/img/posts/{slug}.jpg)
_사진: [작가명](커먼즈 파일 페이지 URL), [라이선스명](라이선스 URL)_
```

**촬영 시점이 기사 내용과 다르면 캡션에 명시한다.** 독자가 현장 사진으로 오해하면 안 된다.

```markdown
_사진: [TechCrunch](...), [CC BY 2.0](...) — 2018년 Disrupt 현장 자료 사진_
```

---

## 6. Chirpy 문법 몇 가지

```markdown
> 팁 상자
{: .prompt-tip }

> 정보 상자
{: .prompt-info }

> 경고 상자
{: .prompt-warning }

> 위험·주의 상자
{: .prompt-danger }
```

내부 글 링크는 `/posts/{slug}/` 형태로 건다. 예: `[MCP와 API](/posts/mcp-vs-api/)`

---

## 7. 하지 말 것

- ❌ 위 5개 밖의 카테고리
- ❌ 미래 날짜 `date`
- ❌ 출처 없는 수치·인용
- ❌ 자막·원문 없이 제목만 보고 요약
- ❌ 기사에 실린 라이선스 사진(Getty·AP·Reuters·매체 촬영본)을 가져다 쓰기
- ❌ 크레딧 없는 이미지
- ❌ SVG에 색상 직접 지정
- ❌ 배포 성공 확인 전에 링크 알리기
- ❌ 외부 글을 그대로 옮기기 (요약·재구성만)
- ❌ 사용자 승인 없이 새 최상위 카테고리 만들기

---

## 8. 더 자세한 운영 규칙

뉴스 감지 → 추천 → 승인 → 발행 파이프라인, 소스 목록, 점수화 기준 등
에이전트 운영 전반은 사용자의 맥에 있는 파일에 있다.

`~/.claude/clara/agents/blog/AGENT.md`
