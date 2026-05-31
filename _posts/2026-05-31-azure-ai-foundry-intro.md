---
title: "Azure AI Foundry란 무엇인가 — RAG 구현 첫 인상"
date: 2026-05-31 11:00:00 +0900
categories: [Azure & Cloud]
tags: [azure, ai-foundry, rag, openai, microsoft]
---

## Azure AI Foundry란?

Azure AI Foundry(구 Azure AI Studio)는 Microsoft의 통합 AI 개발 플랫폼입니다. 기업이 Azure OpenAI, 오픈소스 모델, 자체 데이터를 결합해 엔터프라이즈급 AI 솔루션을 구축할 수 있도록 합니다.

## 핵심 구성 요소

```
Azure AI Foundry
├── Hub — 조직 단위 관리 (리소스, 보안, 비용)
├── Project — 실제 AI 앱 개발 공간
│   ├── Playground (GPT-4o, Phi-4 등 모델 테스트)
│   ├── Prompt Flow (파이프라인 시각화)
│   └── Evaluation (모델 품질 평가)
└── Model Catalog (700+ 모델)
```

## RAG (Retrieval-Augmented Generation) 구현

Pre-Sales 업무에서 고객사 내부 문서를 기반으로 답변하는 챗봇을 PoC로 구현했습니다.

### 아키텍처

```
사용자 질문 
    → Azure AI Search (인덱싱된 문서 검색)
    → 관련 청크 추출
    → GPT-4o에 컨텍스트와 함께 전달
    → 근거 있는 답변 생성
```

### 설정 포인트

1. **Azure AI Search 인덱스 생성** — PDF, DOCX 등 문서 청크 분할
2. **Embedding 모델** — `text-embedding-3-large` 권장 (한국어 포함)
3. **Prompt Flow** — 검색 → 증강 → 생성 파이프라인 시각화

## 실전에서 느낀 점

- **장점**: Azure 생태계(AAD, Key Vault, Monitor)와 자연스러운 통합
- **주의**: 한국어 청크 분할 시 의미 단위 분리 필요 (기본 설정으로는 어색하게 잘림)
- **비용**: GPT-4o 사용량 모니터링 필수 — Prompt Flow 테스트만으로도 토큰 빠르게 소모

다음 포스트에서 Azure AI Foundry + RAG 구현 단계별 가이드를 올리겠습니다.

---

> 본 내용은 실무 경험을 바탕으로 한 개인 기록이며, Microsoft 공식 문서와 함께 참고하세요.
> 공식 문서: [Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
