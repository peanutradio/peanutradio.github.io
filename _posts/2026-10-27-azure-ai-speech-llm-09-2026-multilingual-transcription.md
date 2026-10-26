---
title: "Azure AI Speech LLM 09-2026 — 섞인 언어도 한 번에, 힌트는 언제 줄까"
date: 2026-10-27 06:53:00 +0900
categories: [IT News]
tags: [azure, foundry, llm]
description: Fast Transcription API의 LLM Speech가 09-2026으로 올라 로캘이 늘고 다국어 혼합 오디오 처리가 기본이 됐다. 고객이 할 일은 없지만 확인할 것은 있다.
---

> 이 글은 Microsoft Foundry Blog의 "Announcing Azure AI Speech LLM 09-2026" 발표를 읽고 정리·재구성한 것입니다. 수치는 모두 Microsoft 내부 평가 기준이며, 직접 실행해 확인한 값이 아닙니다.
{: .prompt-info }

## 결론부터

Azure AI Speech의 LLM Speech(Fast Transcription API)가 09-2026 버전으로 올라갔습니다. 핵심은 세 가지입니다. 지원 로캘이 늘었고, 여러 언어가 섞인 오디오는 로캘 힌트 없이 기본 다국어 모드로 처리하며, 이전 버전(2607) 대비 정확도와 속도가 좋아졌다고 합니다. 서비스 쪽에서 자동 배포되므로 고객이 따로 할 일은 없다고 합니다. 다만 "자동으로 바뀐다"는 말은 곧 내 앱의 전사 결과가 어느 날 달라질 수 있다는 뜻이기도 해서, 운영자는 확인할 항목이 생깁니다.

## 원문에서 확인된 것

- 새로 이름이 나온 로캘은 인도계 11개(아삼어, 벵골어, 구자라트어, 칸나다어, 말라얄람어, 마라티어, 오디아어, 펀자브어, 타밀어, 텔루구어, 우르두어)와 카탈루냐어, 말레이어, 웨일스어입니다.
- 모델은 50개 언어, 60개 이상의 로캘을 다룬다고 합니다.
- 내부 평가에서 토큰 오류율(TER)은 최대 25%, 엔티티 오류율(EER)은 최대 37% 개선됐다고 합니다. 벤치마크로는 GigaSpeech와 VoxPopuli가 언급되고, 이득이 가장 큰 곳은 실적 발표 도메인의 엔티티라고 합니다.
- 5~10초 짧은 클립을 높은 동시성으로 처리할 때 2607보다 최대 2배 빠르다고 합니다.
- 무음·빈 오디오를 내용으로 전사하는 환각을 줄이는 개선도 포함됩니다.
- 호출은 `transcriptions:transcribe` 엔드포인트에 `multipart/form-data`로 오디오와 `definition`을 보내는 방식이고, 예시 API 버전은 `2025-10-15`입니다. `definition`에는 `enhancedMode`, `diarization`(`enabled`, `maxSpeakers`), `phraseList`(요청당 최대 2,000개), `locales`, `profanityFilterMode`를 넣습니다. Foundry 플레이그라운드의 "Azure Speech – Speech to Text"에서도 시험할 수 있습니다.

원문에 없는 것도 분명히 해 둡니다. GA인지 프리뷰인지, 가격, 제공 리전, 오디오 길이·파일 크기·속도 제한은 글에 나오지 않습니다. 도입부는 "25개 언어 추가"라고 하는데 본문에 이름이 나온 건 14개뿐이라 숫자가 맞지 않고, 한국어가 좋아졌다는 언급도 없습니다.

## 이 업데이트를 어떻게 읽을까

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="오디오 종류에 따라 로캘 힌트를 생략하거나 지정하는 선택 흐름">
  <defs>
    <marker id="sp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="sp-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 로캘 힌트, 주느냐 마느냐</text>
  <rect x="8" y="80" width="150" height="56" rx="8" class="sk-box"/>
  <text x="83" y="104" text-anchor="middle" class="sk-label">오디오 입력</text>
  <text x="83" y="122" text-anchor="middle" class="sk-sub">언어가 몇 개인가</text>
  <path d="M160,100 L300,56" class="sk-line" marker-end="url(#sp-m1)"/>
  <path d="M160,116 L300,170" class="sk-line-accent" marker-end="url(#sp-a1)"/>
  <rect x="302" y="30" width="200" height="56" rx="8" class="sk-box"/>
  <text x="402" y="54" text-anchor="middle" class="sk-label">여러 언어 섞임</text>
  <text x="402" y="72" text-anchor="middle" class="sk-sub">locales 생략 → 다국어 기본 모드</text>
  <rect x="302" y="144" width="200" height="56" rx="8" class="sk-box-accent"/>
  <text x="402" y="168" text-anchor="middle" class="sk-label">한 언어만</text>
  <text x="402" y="186" text-anchor="middle" class="sk-sub">locales 지정 → 정확도·지연 개선 가능</text>
  <path d="M504,58 L560,58" class="sk-line" marker-end="url(#sp-m1)"/>
  <path d="M504,172 L560,172" class="sk-line-accent" marker-end="url(#sp-a1)"/>
  <rect x="562" y="30" width="150" height="170" rx="8" class="sk-box-muted"/>
  <text x="637" y="108" text-anchor="middle" class="sk-label">전사 결과</text>
  <text x="637" y="126" text-anchor="middle" class="sk-sub">phraseList로 용어 보정</text>
</svg>
<figcaption>힌트는 "언어가 하나라고 확신할 때만" 주고, 모르거나 섞여 있으면 비워 두는 것이 이 버전의 기본 사용법이다.</figcaption>
</figure>

이번 발표는 새 기능이라기보다 "다국어를 기본값으로 삼는 쪽으로 무게가 이동했다"는 소식으로 읽힙니다. 예전에는 언어를 먼저 정하고 그 언어의 모델에 오디오를 넣는 설계가 자연스러웠다면, 이제는 섞여 있는 채로 넣고 필요할 때만 좁히는 설계가 권장되는 셈입니다. 콜센터, 다국적 회의, 한국어 사이에 영어 제품명이 끼는 강의 녹음처럼 현실의 오디오는 대부분 섞여 있으니 방향은 이해가 됩니다.

지난번에 정리한 [Foundry 음성 에이전트 글](/posts/foundry-voice-agents-long-running-resilience/)과 이어 보면, 음성 에이전트의 앞단에는 결국 "소리를 글자로 바꾸는 계층"이 있습니다. 이 글이 그 계층을 직접 다룬다고 밝힌 것은 아니고, 둘을 연결하는 것은 제 해석입니다. 그래도 에이전트의 응답 품질은 전사 품질 위에 얹히므로, 전사 계층의 변화는 따로 챙길 가치가 있습니다.

## 실무에서 달라지는 점

### 1. 자동 배포라서 "전과 후"를 비교할 기준부터 만들 것

서비스 쪽에서 모델이 바뀌면 같은 오디오의 전사 결과도 바뀔 수 있습니다. 이미 Fast Transcription을 쓰는 곳이라면 대표 오디오 몇 개와 현재 결과를 기준 파일로 보관해 두고, 변화가 생겼을 때 비교할 수 있게 하는 것이 먼저입니다. 원문은 버전을 고정하는 방법을 말하지 않으므로, 고정이 가능한지는 공식 문서(migrate-2025-10-15 등)에서 확인해야 합니다.

### 2. 로캘 힌트 정책을 앱 단위로 다시 정할 것

한 언어만 나오는 오디오라면 힌트가 정확도와 지연에 도움이 될 수 있다고 하고, 섞인 오디오라면 힌트를 비우라고 합니다. 그래서 "무조건 `ko-kr` 고정"이나 "무조건 비움" 중 하나로 정하기보다, 입력 유형(단일 언어 회의, 혼합 강의 등)별로 정책을 나누는 편이 맞아 보입니다. 어느 쪽이 나은지는 원문이 한국어 수치를 주지 않으므로, 자기 오디오로 재 보고 판단해야 합니다.

### 3. 한국어 효과는 가정하지 말 것

새로 늘어난 로캘에 한국어는 없고, 개선 수치가 한국어에도 해당하는지 글에 나오지 않습니다. 고객 제안서나 사내 공지에 "정확도 25% 향상"이라고 옮겨 적는 것은 위험합니다. 인용하려면 "Microsoft 내부 평가, 영어 위주 벤치마크 기준, 최대치"라는 조건을 함께 써야 합니다.

### 4. 용어 사전과 화자 분리는 `definition`에서 설정

제품명·약어가 많은 업무라면 `phraseList`(요청당 최대 2,000개)가 가장 먼저 손댈 곳입니다. 화자 분리는 `diarization`의 `enabled`와 `maxSpeakers`로 켭니다. 다만 예시 JSON의 `locales` 값에 따옴표가 빠져 있다고 하니 그대로 복사하지 말고 각 로캘 코드를 문자열로 감싸야 합니다.

### 5. 교육할 때 강조할 점

수강생에게는 "다국어를 지원한다"와 "내 언어가 잘 된다"가 다른 말이라는 점을 짚겠습니다. 무음 구간 환각이 줄었다는 개선이 있어도, 결과를 사람이 검수하는 단계는 남겨야 합니다. 법무·의료·회의록처럼 틀리면 곤란한 용도라면 더욱 그렇습니다.

## 남는 질문

GA 여부, 가격, 리전, 입력 제한이 모두 비어 있습니다. 도입 검토 단계라면 이 네 가지는 공식 문서에서 먼저 확인해야 하고, 확인 전에는 계획서에 확정 사항처럼 적지 않는 편이 안전합니다.

## 참고

- [Announcing Azure AI Speech LLM 09-2026 (Microsoft Foundry Blog)](https://devblogs.microsoft.com/foundry/announcing-azure-ai-speech-llm-09-2026/)
- [LLM Speech 개요 (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/llm-speech)
- [Fast transcription API (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/fast-transcription-create)
- [2025-10-15 마이그레이션과 요청 구성 (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/migrate-2025-10-15)
- [Phrase list로 정확도 높이기 (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/improve-accuracy-phrase-list)
- [Announcing Azure AI Speech LLM 2607 (이전 버전)](https://devblogs.microsoft.com/foundry/announcing-azure-ai-speech-llm-2607/)
