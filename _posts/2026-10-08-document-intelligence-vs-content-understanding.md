---
title: "Document Intelligence vs Content Understanding — 문서 추출기 고르는 기준"
date: 2026-10-08 06:51:00 +0900
categories: [Azure AI]
tags: [azure, document-intelligence, content-understanding, rag, foundry]
description: 잘 도는 Document Intelligence는 두고, 변동 큰 문서·RAG·추론이 필요한 곳에 Content Understanding을 쓰라는 선택 가이드를 정리했다.
---

> 이 글은 Microsoft Foundry 블로그의 "Choosing Azure Document Intelligence and Content Understanding"을 읽고 정리·재구성한 것이다. 원문에 수치 벤치마크는 없고, 권고는 Microsoft 내부 벤치마크에 근거한다고 밝힌다.
{: .prompt-info }

## 결론부터

원문의 답은 "승자는 없다"이다. 잘 돌아가는 Document Intelligence(ADI)는 옮기지 말고, 변동이 크거나 추론이 필요하거나 RAG 입력을 만들어야 하는 곳에 Content Understanding(ACU)을 **선택적으로** 쓰라는 것이다. 두 서비스는 API, 엔드포인트, SDK, 과금이 따로여서 마이그레이션도 필수가 아니고, 같은 업무 안에서 문서 종류별로 다르게 골라도 된다.

## 무엇이 다른가

- **ADI**는 목적별로 학습된 모델이다. 라벨, 값, 행, 열 같은 시각적 관계를 배운다. 양식이 안정적이고, 값이 문서에 명시돼 있고, 라벨 달린 샘플이 있고, 결과의 반복성이 중요할 때 맞는다.
- **ACU**는 필드 스키마를 따라 생성형 모델이 추출한다. 레이아웃이 다양하거나 의미로 정의되는 필드, 여러 구절을 종합해야 하는 답, 도표에 맞는다. 라벨 없이 시작할 수 있고(zero-shot), 신뢰도 점수와 근거 위치를 주며, 날짜·숫자 같은 값을 정규화한다.

## 상황별 선택

<figure class="sketch">
<svg viewBox="0 0 720 270" role="img" aria-label="문서 상황에 따라 ADI와 ACU 중 어느 쪽을 쓰는지 보여주는 표 형태의 그림">
  <defs>
    <marker id="di-m1" viewBox="0 0 10 10" refX="9" refY="5"
            markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 상황 → 쓸 서비스</text>
  <rect x="8" y="40" width="330" height="200" rx="8" class="sk-box"/>
  <text x="173" y="64" text-anchor="middle" class="sk-label">ADI</text>
  <text x="24" y="94" class="sk-sub">· 이미 잘 돌아가는 ADI 워크로드</text>
  <text x="24" y="120" class="sk-sub">· 송장·영수증·신분증 등 성숙한 prebuilt</text>
  <text x="24" y="146" class="sk-sub">· 라벨이 있는 정형 커스텀 양식</text>
  <text x="24" y="172" class="sk-sub">· 온프레미스·폐쇄망 (컨테이너)</text>
  <rect x="382" y="40" width="330" height="200" rx="8" class="sk-box-accent"/>
  <text x="547" y="64" text-anchor="middle" class="sk-label">ACU</text>
  <text x="398" y="94" class="sk-sub">· 새 클라우드 OCR·레이아웃 (prebuilt-read/layout)</text>
  <text x="398" y="120" class="sk-sub">· 라벨 없는 커스텀 추출 (zero-shot)</text>
  <text x="398" y="146" class="sk-sub">· 변동 큰 비정형, 추론·계산이 필요한 필드</text>
  <text x="398" y="172" class="sk-sub">· RAG 전처리, 이미지·오디오·비디오</text>
  <path d="M340,140 L380,140" class="sk-line" marker-end="url(#di-m1)"/>
</svg>
<figcaption>양식이 고정되고 이미 잘 돌면 ADI, 변동·추론·RAG·멀티미디어면 ACU다.</figcaption>
</figure>

세부 항목도 몇 가지 짚어 둘 만하다.

- 새 클라우드 프로젝트의 OCR·레이아웃은 ACU의 `prebuilt-read`, `prebuilt-layout`부터 시작하라고 한다. 원문은 구조 출력이 풍부하고 정확도·지연·페이지당 레이아웃 가격에서 유리하다고 쓴다. 이 분석기들은 언어 모델이나 임베딩 모델 없이 결정적으로 동작한다.
- 성숙한 prebuilt가 있는 표준 양식은 ADI prebuilt가 비슷한 정확도에 비용 이점을 유지한다. ACU의 2026-06-01-preview prebuilt(Advanced Contextualization)도 대안인데 프리뷰다.
- RAG에는 ACU의 RAG 분석기를 쓴다. 구조를 살린 Markdown과 검색용 청크, 도표 분석, 요약을 준다.
- 컨테이너 배포는 ADI만 목록에 있다.

## agentic mode는 기본값이 아니다

ACU 2.0 프리뷰에는 두 가지가 있다. Advanced contextualization은 라벨 예시 여러 개로 필드별 추출 방식을 학습한다. Agentic mode는 반복 추론 루프로 재무 합계 대조, 내부 일관성 검증, 계약서와 개정안의 연결 같은 복잡한 분석에 쓴다. 원문은 단순 필드 추출의 기본값으로 쓰지 말라고 분명히 적는다.

## 평가할 때 조심할 것

같은 대표 샘플로 두 방식을 모두 돌려 비교하라는 권고다. 여기서 함정이 있다. "January 1, 2025", "1/1/2025", "2025-01-01"은 같은 날짜인데 문자열 일치로 채점하면 다르게 센다. ACU는 값을 정규화하니 이 방식으로는 정확도가 왜곡될 수 있다. 원문이 제시하는 지표는 필드 단위 의미 정확도, 정규화·검증 결과, 오류·예외율, 사람 검토 비율과 자동 처리율, 분류·라우팅 정확도, 지연과 총비용, 운영 안정성, 구축·유지 노력이다. 평균 정확도가 특정 문서 종류의 실패를 가릴 수 있다는 경고도 있다.

사례로는 말레이시아 핀테크 FinHero가 나온다. 일반 영수증은 ADI prebuilt와 커스텀 모델로, 할인·중첩 옵션·반올림·서비스 요금이 얽힌 영수증은 ACU 커스텀 분석기로 나눠 둘 다 쓴다.

## 참고할 점

가격, 한도, 지원 요소, 프리뷰 기능은 바뀔 수 있으니 결정 전에 최신 문서를 확인하라고 원문이 거듭 말한다. 나도 숫자 대신 선택 기준만 가져왔다.

## 참고

- [Choosing Azure Document Intelligence and Content Understanding — Microsoft Foundry Blog](https://devblogs.microsoft.com/foundry/choosing-azure-document-intelligence-and-content-understanding/)
