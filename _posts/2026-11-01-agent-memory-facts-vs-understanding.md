---
title: "에이전트 장기 기억 설계 — 사실은 DB에, 이해는 벡터에, 원본 영상은 밖에"
date: 2026-11-01 20:55:00 +0900
categories: [AI Agent]
tags: [ai-agent, llm, rag]
description: 에이전트가 세션이 끝난 뒤에도 무엇을 기억해야 하는지 공부하며 정리했습니다. 정확히 찾아야 하는 사실과 의미로 떠올려야 하는 이해를 나눠 저장하고, 읽기는 세션 시작에, 쓰기는 두 갈래로 나누는 구조입니다.
---

> 이 글은 Google Cloud Tech의 영상 "AI Agent Memory Architecture: Structured SQL, Semantic Vector Search & Memory Bank"(2026-10-28, Annie Wang, 12분)의 자막 전문을 보며 공부한 내용을 제 말로 정리·재구성한 것입니다. 제 비서 에이전트와 비교한 부분은 제 경험입니다.
{: .prompt-info }

## 결론부터

에이전트 기억을 설계할 때 첫 질문은 "어디에 저장할까"가 아니라 **"이건 어떤 종류의 기억인가"**였습니다.

- **사실(fact)**

  정확합니다. 찾아봅니다(look up). → SQL 같은 구조화 DB
- **이해(understanding)**

  의미입니다. 떠올립니다(recall). → 의미로 검색되는 벡터 저장소
- **원본 지각(raw perception)**

  카메라 프레임, 오디오 파형 자체는 보통 기억이 아닙니다. 거기서 뽑은 의미만 기억으로 갑니다.

그리고 읽기는 세션 시작에 두 저장소를 모두 불러오고, 쓰기는 사실은 즉시·동기로, 이해는 대화 흐름 밖에서 비동기로 처리합니다. 저는 맥미니에서 몇 달째 개인 비서 에이전트를 돌리고 있는데, 공부하고 나니 제 비서의 기억 구조에서 고칠 곳이 바로 보였습니다.

## 단기 기억과 장기 기억

멀티모달 에이전트와 대화하면 모든 걸 기억하는 것처럼 느껴집니다. 5분 전에 한 말을 받아서 바로 씁니다. 하지만 대부분은 **단기 기억**, 즉 RAM 같은 것입니다. 세션이 도는 동안만 쓸모 있고 끝나면 사라집니다.

그래서 진짜 설계 질문은 "다 저장할 수 있나?"가 아니라 **"다음 세션으로 가져갈 가치가 있는 건 무엇인가?"**입니다. 영상의 예시는 개인 스타일링 에이전트입니다. "내일 고객 저녁 자리에 뭘 입을까?"라고 물으면, 위시리스트에 있는 앵클부츠와 지난번에 산 빨간 드레스를 엮어서 추천합니다. 이게 되려면 지난 세션에서 무엇을 남겨 뒀는지가 중요합니다.

## 기억은 두 종류, 저장소도 두 곳

<figure class="sketch">
<svg viewBox="0 0 720 300" role="img" aria-label="세션이 끝날 때 대화에서 사실은 구조화 DB로, 이해는 벡터 저장소로 가고, 원본 오디오와 영상은 기억 밖 별도 저장소로 간다">
  <defs>
    <marker id="am-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="am-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 무엇을 어디에 남기나</text>
  <rect x="8" y="110" width="170" height="80" rx="8" class="sk-box"/>
  <text x="93" y="140" text-anchor="middle" class="sk-label">세션 (단기 기억)</text>
  <text x="93" y="160" text-anchor="middle" class="sk-sub">RAM처럼 — 끝나면 사라짐</text>
  <text x="93" y="176" text-anchor="middle" class="sk-sub">대화 · 음성 · 카메라</text>
  <rect x="300" y="36" width="200" height="66" rx="8" class="sk-box-accent"/>
  <text x="400" y="62" text-anchor="middle" class="sk-label">사실 → 구조화 DB</text>
  <text x="400" y="82" text-anchor="middle" class="sk-mono">Postgres · MySQL · Cloud SQL</text>
  <rect x="300" y="117" width="200" height="66" rx="8" class="sk-box-accent"/>
  <text x="400" y="143" text-anchor="middle" class="sk-label">이해 → 벡터 저장소</text>
  <text x="400" y="163" text-anchor="middle" class="sk-mono">pgvector · AlloyDB · Memory Bank</text>
  <rect x="300" y="210" width="200" height="66" rx="8" class="sk-box-muted"/>
  <text x="400" y="236" text-anchor="middle" class="sk-label">원본 오디오·영상</text>
  <text x="400" y="256" text-anchor="middle" class="sk-sub">기억 아님 · 재생·감사·디버깅용</text>
  <path d="M180,130 L296,72" class="sk-line-accent" marker-end="url(#am-a1)"/>
  <path d="M180,150 L296,150" class="sk-line-accent" marker-end="url(#am-a1)"/>
  <path d="M180,170 L296,240" class="sk-line" marker-end="url(#am-m1)"/>
  <text x="530" y="66" class="sk-sub">"목요일 저녁 약속"</text>
  <text x="530" y="82" class="sk-sub">"파란 재킷 구매" — 찾는다</text>
  <text x="530" y="147" class="sk-sub">"화려한 것보다 단순한 것"</text>
  <text x="530" y="163" class="sk-sub">"자신감엔 어두운 색" — 떠올린다</text>
  <text x="530" y="240" class="sk-sub">뽑아낸 의미만 위로 올린다</text>
  <text x="530" y="256" class="sk-sub">"그 검은 모자를 또 썼다"</text>
</svg>
<figcaption>저장소를 고르기 전에 기억의 종류부터 나눠야 한다. 사실과 이해는 같은 곳에 살면 안 된다.</figcaption>
</figure>

### 1. 사실은 구조화 DB에

"목요일에 저녁 약속이 있다", "파란 재킷을 샀다"는 행과 열에 들어갑니다. ID와 날짜가 있고 생성·수정·삭제가 됩니다. "목요일 일정이 뭐야?"에 비슷한 기억이 돌아오면 안 됩니다. 정확한 그 일정이 와야 합니다. 그래서 Postgres, MySQL, Google Cloud라면 Cloud SQL처럼 스택에 맞는 SQL DB에 둡니다.

### 2. 이해는 벡터 저장소에

"화려한 것보다 단순한 걸 좋아한다", "더위를 탄다", "자신감이 필요할 땐 어두운 색을 고른다"는 표 한 줄로 떨어지지 않는 **패턴**입니다. 영상의 표현을 빌리면 "캘린더를 아는 것"과 "나를 아는 것"의 차이입니다. 이런 건 의미로 검색할 수 있어야 합니다. "이런 자리에 맞는 스타일 취향에 대해 내가 아는 게 뭐지?"라고 물으면 비슷한 의미의 기억이 돌아와야 합니다.

여기서 하나 새로 알게 된 점이 있습니다. **벡터 저장소는 제품명이 아니라 범주**입니다. Postgres 안의 pgvector도, AlloyDB도 됩니다. Google Cloud의 관리형 선택지인 Memory Bank(Vertex AI Agent Engine)는 벡터를 저장하는 데서 그치지 않고 지난 세션을 읽어 쓸 만한 기억을 **추출하고 시간에 따라 갱신**까지 해 줍니다. 일반 벡터 저장소를 쓰면 이 추출 로직은 직접 만들어야 합니다. 그래도 요점은 "Memory Bank를 써라"가 아니라 "이해는 의미로 찾을 수 있는 곳에 둬라"입니다.

### 3. 흔한 실수 — 전부 벡터에 넣기

"시맨틱 메모리"라는 말을 듣고 사실·선호·일정까지 전부 벡터 저장소에 넣는 경우가 많다고 합니다. 단순해 보이지만, 벡터 저장소는 **유사도**를 위해 만든 것입니다. 질문이 모호할 때는 유사도가 좋지만, 답이 정확해야 할 때는 위험합니다. "내가 뭘 샀지?"에 비슷한 기억으로 추측하면 안 되고 DB를 조회해야 합니다. "저녁 자리엔 보통 뭘 좋아하지?"가 의미 기반 질문입니다.

### 4. 원본 지각은 기억 밖에

카메라 프레임, 오디오 파형, 라이브 스트림 자체는 보통 기억에 넣지 않습니다. 에이전트는 모든 픽셀을 영구히 기억할 필요가 없고, 거기서 뽑은 의미("이 저녁 자리는 격식 있는 자리라고 했다")만 있으면 됩니다. 원본을 재생·감사·디버깅·학습 데이터용으로 보관하는 건 **별개의 저장 결정**이지 에이전트 기억이 아닙니다.

## 읽기 — 세션 시작에 두 곳을 다 불러온다

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="세션 시작 때 구조화 DB에서 사실을, 벡터 저장소에서 관련 이해를 불러와 하나의 컨텍스트로 합치고, 에이전트는 두 기억을 섞어 첫 마디를 한다">
  <defs>
    <marker id="am-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 세션 시작 — 읽기</text>
  <rect x="8" y="40" width="190" height="62" rx="8" class="sk-box"/>
  <text x="103" y="66" text-anchor="middle" class="sk-label">구조화 DB</text>
  <text x="103" y="86" text-anchor="middle" class="sk-sub">다가오는 일정 · 위시리스트 · 프로필</text>
  <rect x="8" y="130" width="190" height="62" rx="8" class="sk-box"/>
  <text x="103" y="156" text-anchor="middle" class="sk-label">벡터 저장소</text>
  <text x="103" y="176" text-anchor="middle" class="sk-sub">스타일 취향 · 편안함 · 좋아하는 색</text>
  <rect x="270" y="80" width="160" height="70" rx="8" class="sk-box-accent"/>
  <text x="350" y="108" text-anchor="middle" class="sk-label">컨텍스트</text>
  <text x="350" y="128" text-anchor="middle" class="sk-sub">사실 + 의미</text>
  <rect x="500" y="62" width="212" height="106" rx="8" class="sk-box"/>
  <text x="606" y="88" text-anchor="middle" class="sk-label">에이전트의 첫 마디</text>
  <text x="606" y="112" text-anchor="middle" class="sk-sub">"목요일에 저녁 있으시죠" ← DB</text>
  <text x="606" y="132" text-anchor="middle" class="sk-sub">"단순한 걸 좋아하시니까" ← 벡터</text>
  <text x="606" y="154" text-anchor="middle" class="sk-sub">답 하나, 기억 시스템 둘</text>
  <path d="M200,71 L266,104" class="sk-line" marker-end="url(#am-m2)"/>
  <path d="M200,161 L266,128" class="sk-line" marker-end="url(#am-m2)"/>
  <path d="M432,115 L496,115" class="sk-line" marker-end="url(#am-m2)"/>
</svg>
<figcaption>사용자가 말을 꺼내기 전에 두 기억을 이미 들고 있어야 0에서 시작하지 않는다.</figcaption>
</figure>

대화가 시작되기 전에 DB에서 사실을, 벡터 저장소에서 지금 상황과 관련된 이해를 불러옵니다. 그래서 사용자가 입을 열었을 때 에이전트는 0에서 시작하지 않습니다.

## 쓰기 — 두 갈래로 나눈다

<figure class="sketch">
<svg viewBox="0 0 720 250" role="img" aria-label="쓰기는 두 갈래다. 사실은 대화 중 도구 호출로 즉시 DB에 쓰고, 이해는 실시간 경로 밖에서 대화를 검토해 추출한 뒤 벡터 저장소에 쓴다. 다음 세션에서 둘 다 다시 읽는다">
  <defs>
    <marker id="am-m3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="am-a3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">③ 쓰기 — 빠른 길과 느린 길</text>
  <rect x="8" y="90" width="160" height="70" rx="8" class="sk-box"/>
  <text x="88" y="118" text-anchor="middle" class="sk-label">라이브 대화</text>
  <text x="88" y="138" text-anchor="middle" class="sk-sub">음성 · 지연에 민감</text>
  <rect x="250" y="34" width="200" height="62" rx="8" class="sk-box-accent"/>
  <text x="350" y="60" text-anchor="middle" class="sk-label">동기 · 즉시</text>
  <text x="350" y="80" text-anchor="middle" class="sk-sub">"위시리스트에 블레이저 추가" = 도구 호출</text>
  <rect x="250" y="160" width="200" height="62" rx="8" class="sk-box-muted"/>
  <text x="350" y="186" text-anchor="middle" class="sk-label">비동기 · 핫패스 밖</text>
  <text x="350" y="206" text-anchor="middle" class="sk-sub">모델이 대화를 읽고 의미 추출</text>
  <rect x="540" y="34" width="172" height="62" rx="8" class="sk-box"/>
  <text x="626" y="70" text-anchor="middle" class="sk-label">구조화 DB</text>
  <rect x="540" y="160" width="172" height="62" rx="8" class="sk-box"/>
  <text x="626" y="196" text-anchor="middle" class="sk-label">벡터 저장소</text>
  <path d="M170,110 L246,68" class="sk-line-accent" marker-end="url(#am-a3)"/>
  <path d="M170,140 L246,188" class="sk-line" stroke-dasharray="5 4" marker-end="url(#am-m3)"/>
  <path d="M452,65 L536,65" class="sk-line-accent" marker-end="url(#am-a3)"/>
  <path d="M452,191 L536,191" class="sk-line" marker-end="url(#am-m3)"/>
  <text x="208" y="236" text-anchor="middle" class="sk-sub">턴 뒤 · 구간마다 · 세션 끝</text>
  <text x="626" y="128" text-anchor="middle" class="sk-sub">다음 세션에 둘 다 다시 읽음</text>
</svg>
<figcaption>사실은 지금 바로 쓰고, 이해는 판단할 근거가 쌓인 뒤 대화를 막지 않는 곳에서 쓴다.</figcaption>
</figure>

### 사실은 지금 바로

"그 블레이저 위시리스트에 넣어 줘"는 도구 호출 한 번으로 DB에 바로 씁니다. 구조화돼 있고 정확하며, 사용자도 지금 저장되기를 기대합니다.

### 이해는 판단이 필요하다

"오늘은 노란색이 끌리네" 한 마디로 그 사람의 스타일 전체를 바꾸면 안 됩니다. 이유를 설명했거나 여러 세션에 걸쳐 같은 패턴이 반복되는 등 **더 강한 신호**가 있어야 합니다. 모든 문장을 영구 선호로 저장하지 말고, 기억 서비스가 대화를 검토해서 남길 가치가 있는 것만 뽑게 해야 기억이 시끄러워지지 않습니다.

### 언제 추출하나 — "핫패스 밖"이 규칙

추출은 턴마다, 일정 간격마다, 세션 끝에 할 수 있습니다. Memory Bank는 세션 중간 생성도 지원합니다. 짧은 구간에서 뽑으면 기억이 깔끔하고, 사용자가 중간에 나가도 이미 저장돼 있다는 장점이 있습니다. 하지만 추출은 DB 쓰기보다 무겁습니다. 모델이 대화를 읽고 의미를 추론해야 하니까요. 음성 에이전트는 지연에 민감하므로 **느린 추출이 오디오를 막게 두면 안 됩니다.** 그래서 규칙은 "세션 끝까지 기다려라"가 아니라 "추출을 라이브 경로 밖에 둬라"입니다. 영상의 데모는 사실을 대화 중에 즉시 쓰고, 이해는 세션이 끝난 뒤 따로 추출합니다.

## 제 비서 에이전트에 대어 보니

제 비서는 Claude Code 위에서 도는 개인 에이전트입니다. 공부하고 나서 기억 구조를 다시 보니 맞는 부분과 틀린 부분이 분명했습니다.

- **사실 쪽은 이미 구조화돼 있었다**

  하루 두 번 도는 점검 루틴은 직전 결과를 JSON 파일에 저장하고 "새로 생긴 것"만 비교해서 알려 줍니다. 검색 색인 요청도 큐 파일에서 정확히 10건씩 꺼냅니다. 비슷한 걸 찾는 게 아니라 정확한 걸 찾는 일이라 영상이 말한 "구조화 기억" 자리와 같았습니다.

- **이해 쪽은 한 문장을 바로 규칙으로 저장하고 있었다**

  "번호 항목은 제목 아래 줄에 설명" 같은 제 피드백을 마크다운 파일 하나에 하나씩 바로 저장하고, 세션마다 인덱스를 통째로 불러옵니다. 영상이 경고한 "한 마디를 영구 선호로" 바로 그 패턴입니다. 실제로 제가 한 말을 비서가 좁게 해석해 기억해 두는 바람에 제가 다시 정정한 일이 있었습니다.

- **바꿔 볼 것**

  이해 쪽 기억은 대화 중에 바로 확정하지 말고, 세션이 끝난 뒤 "이번에 반복되거나 이유가 붙은 선호가 있었나"를 따로 검토하는 단계를 두려고 합니다. 인덱스가 더 커지면 그때 이해 쪽만 의미 검색으로 옮기고, 사실 쪽은 지금처럼 파일·DB에서 정확히 조회하는 구조로 가져갈 생각입니다.

## 정리

1. 저장하기 전에 "이건 어떤 종류의 기억인가"부터 묻는다
2. 사실은 찾고(DB), 이해는 떠올린다(벡터). 둘을 한 곳에 섞지 않는다
3. 원본 오디오·영상은 기억이 아니다. 뽑아낸 의미만 남긴다
4. 세션 시작에 둘 다 읽고, 사실은 즉시·이해는 핫패스 밖에서 쓴다

잊어버리는 에이전트와 하던 곳부터 이어 가는 에이전트의 차이는 모델이 아니라 이 기억 설계에서 나온다는 걸 배웠습니다.

## 참고

- Google Cloud Tech, "AI Agent Memory Architecture: Structured SQL, Semantic Vector Search & Memory Bank" (2026-10-28, Annie Wang): <https://www.youtube.com/watch?v=8C4bh59FtsI>
- 예제 코드(영상 설명란): <https://g.dev/ai/fashion-mind>
- Vertex AI Agent Engine Memory Bank: <https://g.dev/ai/scale-your-agents>
- Google Cloud SQL 문서: <https://g.dev/cloud/sql-docs>
- AlloyDB 문서: <https://g.dev/cloud/alloydb-docs>
- 같은 시리즈의 이전 글: [모델 바깥의 소프트웨어, AI 하네스를 설계하는 4가지 축](/posts/ai-harness-four-parts/)
