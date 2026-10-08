---
title: "TCP/IP 3D 강의 — 웹페이지 하나가 열리기까지, 데이터가 지나가는 길"
date: 2026-10-09 08:00:00 +0900
categories: [Video Lectures, 네트워크 기초]
tags: [tcp-ip, networking, osi-model, lecture]
description: 내 PC의 요청이 TCP/IP 4계층을 내려가 라우터를 지나 웹 서버에서 다시 올라가는 과정을 3D로 따라가는 약 3분짜리 인터랙티브 강의.
---

**웹 주소를 치고 엔터를 누른 뒤 1초 안에 무슨 일이 벌어질까.** 그 과정을 3D로 직접 따라가 보는 강의를 만들었다. 내 PC에서 보낸 요청이 TCP/IP 4계층을 내려가고, 라우터를 지나, 웹 서버에서 다시 올라가는 길을 약 3분, 5개 챕터로 보여 준다. 음성 설명이 같이 나온다.

> 화면을 드래그하면 시점이 돌아가고, 휠로 확대·축소할 수 있다. 소리를 켜고 보면 좋다.
{: .prompt-tip }

## 강의 보기

<div style="position:relative;width:100%;aspect-ratio:16/10;border-radius:12px;overflow:hidden">
  <iframe src="/assets/lectures/tcpip-3d/" title="TCP/IP 3D 강의" loading="lazy" style="position:absolute;inset:0;width:100%;height:100%;border:0" allow="autoplay; fullscreen" allowfullscreen></iframe>
</div>

[🔎 전체 화면으로 보기](/assets/lectures/tcpip-3d/){: target="_blank" }

## 이번 강의에서 다루는 것

| 챕터 | 내용 |
|---|---|
| 1. TCP/IP 4계층 한눈에 | 응용 · 전송 · 인터넷 · 네트워크 접근, 각 층이 맡는 일 |
| 2. 연결 맺기: 3-way 핸드셰이크 | SYN → SYN+ACK → ACK, 대화를 시작하기 전에 서로 확인하는 순서 |
| 3. 캡슐화: 헤더를 하나씩 붙이기 | 데이터가 아래 계층으로 내려가며 헤더라는 '상자'에 겹겹이 담기는 과정 |
| 4. 라우터를 지나며 | 라우터가 IP 주소를 보고 다음 길을 고르는 방식 |
| 5. 역캡슐화: 서버가 상자를 여는 순서 | 서버가 바깥 헤더부터 하나씩 벗겨 원래 데이터를 꺼내는 과정 |

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="TCP/IP 4계층을 내려가며 헤더가 붙고, 서버에서 올라가며 헤더가 벗겨지는 캡슐화와 역캡슐화">
  <defs>
    <marker id="tcp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 내려갈 때 붙이고, 올라갈 때 벗긴다</text>
  <text x="120" y="40" text-anchor="middle" class="sk-label">내 PC</text>
  <text x="600" y="40" text-anchor="middle" class="sk-label">웹 서버</text>
  <rect x="40" y="52" width="160" height="34" rx="6" class="sk-box"/><text x="120" y="74" text-anchor="middle" class="sk-sub">응용 · 데이터</text>
  <rect x="40" y="92" width="160" height="34" rx="6" class="sk-box"/><text x="120" y="114" text-anchor="middle" class="sk-sub">전송 · +TCP 헤더</text>
  <rect x="40" y="132" width="160" height="34" rx="6" class="sk-box"/><text x="120" y="154" text-anchor="middle" class="sk-sub">인터넷 · +IP 헤더</text>
  <rect x="40" y="172" width="160" height="34" rx="6" class="sk-box-accent"/><text x="120" y="194" text-anchor="middle" class="sk-sub">네트워크 접근 · +프레임</text>
  <rect x="520" y="52" width="160" height="34" rx="6" class="sk-box"/><text x="600" y="74" text-anchor="middle" class="sk-sub">응용 · 데이터</text>
  <rect x="520" y="92" width="160" height="34" rx="6" class="sk-box"/><text x="600" y="114" text-anchor="middle" class="sk-sub">전송 · TCP 헤더 제거</text>
  <rect x="520" y="132" width="160" height="34" rx="6" class="sk-box"/><text x="600" y="154" text-anchor="middle" class="sk-sub">인터넷 · IP 헤더 제거</text>
  <rect x="520" y="172" width="160" height="34" rx="6" class="sk-box-accent"/><text x="600" y="194" text-anchor="middle" class="sk-sub">네트워크 접근 · 프레임 제거</text>
  <rect x="300" y="172" width="120" height="34" rx="6" class="sk-box-muted"/><text x="360" y="194" text-anchor="middle" class="sk-sub">라우터</text>
  <path d="M202,189 L298,189" class="sk-line" marker-end="url(#tcp-m1)"/>
  <path d="M422,189 L518,189" class="sk-line" marker-end="url(#tcp-m1)"/>
  <path d="M30,60 L30,190" class="sk-line-accent" marker-end="url(#tcp-m1)"/>
  <path d="M690,190 L690,60" class="sk-line-accent" marker-end="url(#tcp-m1)"/>
</svg>
<figcaption>보내는 쪽은 아래로 내려가며 헤더를 붙이고, 받는 쪽은 위로 올라가며 같은 순서를 거꾸로 밟는다.</figcaption>
</figure>

## 핵심 정리

- **계층은 역할 분담이다.** 응용 계층은 '무엇을' 보낼지, 전송 계층은 '빠짐없이' 보낼지, 인터넷 계층은 '어디로' 보낼지, 네트워크 접근 계층은 '바로 옆 장비까지' 어떻게 보낼지를 맡는다.
- **TCP는 말 걸기 전에 확인부터 한다.** 3-way 핸드셰이크로 양쪽이 준비됐는지 확인한 뒤에 데이터를 보낸다.
- **캡슐화와 역캡슐화는 거울 관계다.** 붙인 순서의 반대로 벗긴다. 이 그림 하나만 기억하면 OSI 7계층 그림도 훨씬 쉽게 읽힌다.

## 시리즈

**영상 강의 › 네트워크 기초** 시리즈의 첫 편이다.
