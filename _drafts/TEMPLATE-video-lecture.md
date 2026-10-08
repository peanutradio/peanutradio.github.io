---
# 영상 강의 글 템플릿 — 복사해서 _posts/YYYY-MM-DD-{영문-slug}.md 로 저장한 뒤 채운다.
# categories 는 반드시 2단계: [Video Lectures, 시리즈명]. 시리즈명은 같은 시리즈끼리 글자 하나까지 똑같이.
# date 는 `date "+%Y-%m-%d %H:%M"` 로 확인한 현재 시각보다 이전으로 (미래 날짜는 빌드에서 빠진다).
title: "시리즈명 N강 — 이번 강의 제목 (60자 이내)"
date: 2026-01-01 00:00:00 +0900
categories: [Video Lectures, 시리즈명]
tags: [lecture, topic-a, topic-b]
description: 이번 강의에서 무엇을 다루는지 한 문장. 160자 이내.
---

**이번 강의의 결론을 한두 문장으로 먼저.**

## 이번 강의에서 다루는 것

- 다루는 내용 1
- 다루는 내용 2
- 다루는 내용 3

## 영상

{% include embed/youtube.html id='VIDEO_ID' %}

> 영상 길이 약 N분. 챕터: 00:00 도입 · 02:10 ○○ · 05:30 ○○
{: .prompt-info }

## 핵심 정리

1. **핵심 1.** 설명
2. **핵심 2.** 설명
3. **핵심 3.** 설명

<!-- 개념 그림이 필요하면 CLAUDE.md §5 의 인라인 SVG(sk-* 클래스) 규칙을 따른다 -->

## 자료·링크

- [슬라이드 / 예제 코드](https://example.com)
- [참고 문서](https://example.com)

## 시리즈 다른 편

- [시리즈명 1강 — 제목](/posts/slug-1/)
- [시리즈명 2강 — 제목](/posts/slug-2/)
- 시리즈 전체 목록: [시리즈명](/categories/시리즈명/) <!-- 공백은 - 로. 예: 네트워크 기초 → /categories/네트워크-기초/ -->
