---
title: "OpenAI 모델과 Codex, Google Cloud Model Garden에 — 프리뷰 단계"
date: 2026-10-24 06:53:00 +0900
categories: [IT News]
tags: [openai, google-cloud, model-garden, codex, multi-cloud, azure]
description: OpenAI가 GPT-6 계열과 Codex를 Google Cloud에 낸다. 비공개 프리뷰이고 GA는 수주 내 예고다. Azure 사용 조직이 짚을 점을 정리했다.
---

> 이 글은 2026-10-21 OpenAI·Google Cloud의 공동 발표를 다룬 보도를 읽고 정리·재구성한 것입니다. OpenAI 원문 페이지(openai.com)는 접근이 막혀 직접 읽지 못했고, 본문은 The Next Web 기사와 검색 요약에 기대고 있습니다. 가격·리전 같은 세부는 원문에서 다시 확인해 주세요.
{: .prompt-warning }

## 결론부터

OpenAI의 프런티어 모델과 Codex가 **Google Cloud의 Model Garden에 들어옵니다.** 현재는 일부 기업 고객 대상 **비공개 프리뷰**이고, OpenAI는 GA를 "수주 내"로 예고했지만 날짜는 없습니다. Azure에서 OpenAI 모델을 쓰는 조직에게 당장 바뀌는 건 없습니다. 다만 "OpenAI 모델은 Azure에서만"이라는 전제로 짠 설계와 교육 자료는 점검할 때가 됐습니다.

## 발표 내용 (확인된 것만)

- **모델**: GPT-6 Astra, GPT-6.1 Sol, GPT-6 Luna가 Model Garden에 올라갑니다. 기사 본문과 이미지 캡션이 Sol의 이름을 다르게 적었고(6.1 / 6), 기사는 이를 정리하지 않았습니다.
- **구매 방식**: Google Cloud Marketplace에서 사고, 기존 Google Cloud 사용 약정을 적용하며, 청구서는 하나로 통합됩니다.
- **데이터 취급(Google 설명)**: 프롬프트와 출력은 고객 데이터 경계 안에 머물고 학습에 쓰이지 않으며, OpenAI 모델은 Google Cloud의 전용 인프라에서 돕니다.
- **Codex**: 개발자는 이미 Codex를 Google 플랫폼에 연결할 수 있고, Google Antigravity와 Gemini Agent 지원은 "곧"이며 날짜는 없습니다.
- **맥락**: OpenAI는 Microsoft가 여전히 주 클라우드 파트너라고 밝혔습니다. 기사에 따르면 Amazon은 4월부터 OpenAI 모델을 호스팅했습니다.

**기사에 없는 것**: 가격, 호출 한도, 모델별 차이와 벤치마크, GA 날짜, 리전, SLA, Google–OpenAI 간 계약 조건.

<figure class="sketch">
<svg viewBox="0 0 720 240" role="img" aria-label="한 모델 제공자가 여러 클라우드 마켓플레이스로 유통되고 고객은 기존 클라우드 약정으로 결제하는 구조">
  <defs>
    <marker id="og-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="og-a2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 모델은 한 곳, 살 수 있는 길은 여러 갈래</text>
  <rect x="8" y="90" width="150" height="60" rx="8" class="sk-box"/>
  <text x="83" y="116" text-anchor="middle" class="sk-label">OpenAI 모델</text>
  <text x="83" y="134" text-anchor="middle" class="sk-sub">GPT-6 계열 · Codex</text>
  <rect x="290" y="36" width="190" height="50" rx="8" class="sk-box"/>
  <text x="385" y="58" text-anchor="middle" class="sk-label">Azure (주 파트너)</text>
  <text x="385" y="75" text-anchor="middle" class="sk-sub">기존 경로</text>
  <rect x="290" y="96" width="190" height="50" rx="8" class="sk-box-accent"/>
  <text x="385" y="118" text-anchor="middle" class="sk-label">Google Model Garden</text>
  <text x="385" y="135" text-anchor="middle" class="sk-sub">비공개 프리뷰 (신규)</text>
  <rect x="290" y="156" width="190" height="50" rx="8" class="sk-box-muted"/>
  <text x="385" y="178" text-anchor="middle" class="sk-label">Amazon (4월부터)</text>
  <text x="385" y="195" text-anchor="middle" class="sk-sub">기사 기준</text>
  <rect x="572" y="90" width="140" height="60" rx="8" class="sk-box"/>
  <text x="642" y="116" text-anchor="middle" class="sk-label">고객 약정</text>
  <text x="642" y="134" text-anchor="middle" class="sk-sub">클라우드별 청구</text>
  <path d="M160,112 L288,62" class="sk-line" marker-end="url(#og-m1)"/>
  <path d="M160,120 L288,121" class="sk-line-accent" marker-end="url(#og-a2)"/>
  <path d="M160,128 L288,181" class="sk-line" marker-end="url(#og-m1)"/>
  <path d="M482,121 L570,121" class="sk-line-accent" marker-end="url(#og-a2)"/>
</svg>
<figcaption>모델 선택과 클라우드 선택이 분리되면서, 무엇을 쓸지는 성능이 아니라 어느 약정으로 살지로 갈리기 쉽다.</figcaption>
</figure>

## 이 블로그의 해석

이번 소식의 무게중심은 모델 성능이 아니라 **유통 경로**입니다. 기사 기준으로 Google은 "세 곳의 주요 비공개 모델(Claude, Gemini, OpenAI)을 모두 파는 유일한 하이퍼스케일러"를 내세웁니다. 고객 입장에서는 이미 가진 클라우드 약정으로 여러 모델을 사는 길이 넓어진다는 뜻입니다. 이는 보도 내용에서 제가 읽어 낸 방향이고, 실제 구매 패턴이 바뀔지는 GA 이후에야 보일 것입니다. Azure 쪽 비용 설계는 [Azure OpenAI 비용 줄이는 세 가지](/posts/azure-openai-cost-guide/)에서 정리한 적이 있는데, 비교할 대안이 생기면 같은 모델이라도 약정·캐싱·배치 조건을 따지는 협상력이 달라질 수 있습니다. 추론 비용 측면은 [Azure Maia 소식](/posts/azure-maia-multi-gigawatt/)과 함께 보면 흐름이 이어집니다.

## 실무에서 달라지는 점

1. **Azure로 OpenAI를 쓰는 운영자**: 당장 이전할 이유는 없습니다. 다만 모델명·버전을 코드에 박지 말고 설정으로 빼 두면 나중에 경로를 바꾸기 쉽습니다. 대상은 이번 프리뷰가 아니라 GA 이후입니다.
2. **멀티 클라우드 조직의 구매·FinOps 담당**: 기존 Google Cloud 약정을 OpenAI 모델에 쓸 수 있다는 점이 핵심입니다. 단가·한도·리전은 공개되지 않았으니 GA 때 Azure 조건과 같은 표로 비교할 항목(단가, 한도, 데이터 보관 옵션)만 미리 정해 두세요.
3. **보안·컴플라이언스 담당**: "데이터 경계 안, 학습 미사용"은 Google 쪽 설명입니다. 기사에 따르면 Zero Data Retention과 Confidential Compute는 나중으로 미뤄진 것으로 보이므로(검색 요약 기준, 원문 미확인), 규제 데이터는 이 항목이 확정되기 전에 올리지 마세요.
4. **IT 강사**: "OpenAI 모델 = Azure"라고 가르쳤다면 "주 파트너는 Azure, 판매 경로는 늘어나는 중"으로 표현을 고치는 편이 정확합니다. 한국 리전 제공 여부는 확인된 바 없다고 덧붙이세요.
5. **지금 할 일 없음도 결론**: 프리뷰는 초청제이고 GA 날짜가 없습니다. 팀 일정에 넣기보다 GA 공지가 나오면 그때 가격표부터 확인하는 정도가 적당합니다.

## 참고

- OpenAI 발표(접근 제한으로 직접 읽지 못함): [OpenAI models are coming to Google Cloud](https://openai.com/index/openai-models-google-cloud)
- 보도: [OpenAI brings GPT-6 Astra and Codex to Google Cloud — The Next Web](https://thenextweb.com/news/openai-models-codex-google-cloud)
- 이전 글: [Azure OpenAI 비용 줄이는 세 가지](/posts/azure-openai-cost-guide/), [Azure Maia, 멀티 기가와트로](/posts/azure-maia-multi-gigawatt/)
