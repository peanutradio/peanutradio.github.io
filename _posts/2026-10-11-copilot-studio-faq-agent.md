---
title: "Copilot Studio로 사내 IT FAQ 에이전트 만들기 — 파일 하나를 지식으로"
date: 2026-10-11 11:55:00 +0900
categories: [Copilot Studio]
tags: [copilot-studio, knowledge, faq, ai-agent, hands-on]
description: FAQ PDF 하나를 지식으로 올려 문서 안의 내용으로만 답하는 헬프데스크 에이전트를 만들었다. 실제 화면 순서대로 정리하고, 크레딧 오류로 막혔던 지점과 해결까지 적었다.
---

**결과부터.** 사내 IT FAQ PDF 한 장을 지식으로 붙인 에이전트를 10분 만에 만들었다. 문서에 있는 질문(프린터 출력)에는 3단계 절차로 답했고, 문서에 없는 질문(법인카드 한도)에는 "FAQ에 없으니 헬프데스크로 문의하라"고 답했다. 핵심은 두 가지다. **지침에 "문서 안에서만 답하라"고 못 박는 것**, 그리고 **기본으로 켜져 있는 웹 검색을 끄는 것.**

실습용으로 가상 회사 "피넛컴퍼니"의 FAQ 문서(Wi‑Fi, 비밀번호, 프린터, VPN 등 7문항)를 PDF로 만들어 썼다.

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="질문이 들어오면 에이전트가 지침을 보고 FAQ 지식에서 찾고, 찾으면 절차로 답하고 못 찾으면 헬프데스크로 안내한다">
  <defs>
    <marker id="csf-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/></marker>
    <marker id="csf-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/></marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 이번에 만든 구조</text>
  <rect x="8" y="80" width="120" height="52" rx="8" class="sk-box"/>
  <text x="68" y="103" text-anchor="middle" class="sk-label">직원 질문</text>
  <text x="68" y="121" text-anchor="middle" class="sk-sub">"프린터가 안 돼요"</text>
  <path d="M130,106 L178,106" class="sk-line" marker-end="url(#csf-m1)"/>
  <rect x="182" y="64" width="170" height="84" rx="8" class="sk-box-accent"/>
  <text x="267" y="92" text-anchor="middle" class="sk-label">에이전트</text>
  <text x="267" y="110" text-anchor="middle" class="sk-sub">지침: 문서 안에서만</text>
  <text x="267" y="128" text-anchor="middle" class="sk-sub">웹 검색 끔</text>
  <path d="M354,106 L402,106" class="sk-line" marker-end="url(#csf-m1)"/>
  <rect x="406" y="80" width="120" height="52" rx="8" class="sk-box"/>
  <text x="466" y="103" text-anchor="middle" class="sk-label">지식</text>
  <text x="466" y="121" text-anchor="middle" class="sk-mono">FAQ.pdf</text>
  <path d="M528,96 L586,60" class="sk-line-accent" marker-end="url(#csf-m2)"/>
  <path d="M528,116 L586,160" class="sk-line" marker-end="url(#csf-m1)"/>
  <rect x="590" y="36" width="126" height="46" rx="8" class="sk-box-accent"/>
  <text x="653" y="56" text-anchor="middle" class="sk-label">찾음</text>
  <text x="653" y="72" text-anchor="middle" class="sk-sub">3단계 절차로 답</text>
  <rect x="590" y="138" width="126" height="46" rx="8" class="sk-box-muted"/>
  <text x="653" y="158" text-anchor="middle" class="sk-label">없음</text>
  <text x="653" y="174" text-anchor="middle" class="sk-sub">헬프데스크 안내</text>
</svg>
<figcaption>FAQ 에이전트의 품질은 모델보다 "모르면 모른다고 하라"는 지침에서 갈린다.</figcaption>
</figure>

## 준비물

- Copilot Studio에 접속되는 계정과 **크레딧이 있는 환경** (이게 중요하다. 아래 "막힌 곳" 참고)
- 지식으로 쓸 문서 1개. 파일 업로드는 파일당 512MB, 최대 500개까지 된다([Quotas and limits](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas))

## 1. 에이전트 만들기

홈에서 **Agent**를 누르면 바로 빌드 화면이 열린다. 예전처럼 대화로 설명하는 단계 없이 이름·지침·도구·지식을 한 화면에서 채우는 방식으로 바뀌었다.

![Copilot Studio 홈에서 Agent 카드를 선택](/assets/img/posts/copilot-studio-faq-agent/01-home.png)
_① Agent 카드를 누르면 새 에이전트 빌드 화면으로 간다_

## 2. 이름과 지침

① 이름을 정하고 ② 지침을 쓴다. 내가 넣은 지침은 다섯 줄이다.

```text
너는 피넛컴퍼니 직원의 IT 질문에 답하는 헬프데스크 에이전트야.
반드시 연결된 지식(사내 IT FAQ 문서)에 있는 내용으로만 답해.
문서에 없는 내용은 추측하지 말고 "헬프데스크(내선 1234)로 문의해 주세요"라고 안내해.
답은 한국어로, 3단계 이내의 짧은 절차로 알려 줘.
비밀번호나 계정 정보는 절대 묻지 마.
```

![이름, 지침, Knowledge 추가 버튼 위치](/assets/img/posts/copilot-studio-faq-agent/02-build.png)
_① 이름 ② 지침 ③ Knowledge의 + 버튼. 오른쪽 위에 모델(여기서는 Claude Opus 5.5)도 고를 수 있다_

> 지침 편집기는 줄 앞에 `- `를 치면 자동으로 글머리표로 바뀐다. 처음에 `- `를 붙여 붙여넣었더니 "• - 문장"처럼 두 번 들어갔다. **그냥 문장만 줄바꿈해서 넣는 게 깔끔하다.**
{: .prompt-tip }

## 3. 지식으로 파일 올리기

③ Knowledge 옆 **+** 를 누르면 지식 추가 창이 뜬다. ① 파일을 끌어다 놓거나 ② SharePoint·OneDrive 같은 원본을 고른다. 이번엔 파일 업로드로 갔다.

![Add knowledge 창](/assets/img/posts/copilot-studio-faq-agent/03-add-knowledge.png)
_① 파일 업로드 영역 ② 실무에선 SharePoint를 더 많이 쓴다_

![업로드한 파일 확인 후 Add to agent](/assets/img/posts/copilot-studio-faq-agent/04-upload.png)
_① 올라간 파일 확인 ② Add to agent. 파일은 Dataverse에 저장된다고 안내된다_

추가가 끝나면 Knowledge에 파일이 붙는다. 여기서 하나 더 해야 한다. 새 에이전트에는 **Search all websites**가 기본으로 붙어 있는데, 이걸 X로 지웠다. 켜 두면 FAQ에 없는 질문에 인터넷 답을 섞어 올 수 있다.

![Knowledge에 FAQ PDF만 남긴 상태](/assets/img/posts/copilot-studio-faq-agent/05-knowledge-added.png)
_① FAQ PDF만 남기고 Search all websites는 지웠다_

## 4. 테스트

상단 **Preview**에서 바로 대화해 봤다. "회사 프린터에서 출력이 안 돼요"라고 물으니 ① 지식 검색 도구를 쓰고 ② 문서의 Q3 내용 그대로 3단계로 답했다. 프린터 이름(PRN‑3F, PRN‑5F)과 "사원증 태그"까지 문서에서 가져왔다.

![문서 안 질문에 대한 답변](/assets/img/posts/copilot-studio-faq-agent/06-answer.png)
_① Used tool: Searched knowledge ② 문서 내용 기반 3단계 답변_

다음은 일부러 문서에 없는 질문. "법인카드 한도는 얼마예요?"

![문서 밖 질문에 대한 답변](/assets/img/posts/copilot-studio-faq-agent/07-out-of-scope.png)
_① 지어내지 않고 "FAQ에 없다 → 헬프데스크 문의"로 답했다_

<figure class="sketch">
<svg viewBox="0 0 720 170" role="img" aria-label="웹 검색을 켜 두면 문서 밖 질문에 인터넷 답이 섞이고, 끄고 지침을 주면 모른다고 답한다">
  <text x="0" y="14" class="sk-title">② 문서 밖 질문을 어떻게 처리하나</text>
  <rect x="8" y="40" width="340" height="110" rx="8" class="sk-box-muted"/>
  <text x="178" y="66" text-anchor="middle" class="sk-label">웹 검색 켜짐 + 지침 없음</text>
  <text x="178" y="92" text-anchor="middle" class="sk-sub">"법인카드 한도?" → 인터넷의 일반론</text>
  <text x="178" y="112" text-anchor="middle" class="sk-sub">그럴듯하지만 우리 회사 규정이 아님</text>
  <text x="178" y="132" text-anchor="middle" class="sk-sub">→ 가장 위험한 답</text>
  <rect x="372" y="40" width="340" height="110" rx="8" class="sk-box-accent"/>
  <text x="542" y="66" text-anchor="middle" class="sk-label">웹 검색 끔 + "문서 안에서만"</text>
  <text x="542" y="92" text-anchor="middle" class="sk-sub">"법인카드 한도?" → FAQ에 없음</text>
  <text x="542" y="112" text-anchor="middle" class="sk-sub">헬프데스크(내선 1234)로 안내</text>
  <text x="542" y="132" text-anchor="middle" class="sk-sub">→ 틀린 답보다 나은 "모름"</text>
</svg>
<figcaption>사내 FAQ 봇에서 "모릅니다"는 실패가 아니라 기능이다.</figcaption>
</figure>

## 막힌 곳: "This environment is out of credits"

처음엔 기본(Default) 환경에서 만들었다. 에이전트 생성과 파일 업로드까지는 잘 됐는데, Preview에서 첫 질문을 던지자 이런 메시지가 나왔다.

```text
You need credits to continue. Credits power building and running your agents
and workflows. This environment is out of credits. Contact your admin to add more credits.
Error code: EnforcementUsageCredits
```

**만들기는 크레딧 없이 되지만, 대화(테스트 포함)는 Copilot 크레딧을 쓴다.** 그래서 크레딧이 할당된 개발 환경(dev)으로 바꿔서 같은 에이전트를 다시 만들었고, 그 뒤로는 바로 됐다. 화면 왼쪽 아래 환경 전환 버튼에서 현재 환경 이름을 확인할 수 있다.

## 비용

- 에이전트를 만들고 저장만 해 두는 데는 크레딧이 들지 않았다. 테스트 대화부터 크레딧이 쓰인다
- 이번 실습에서 쓴 건 테스트 대화 몇 번이 전부다. 게시(Publish)는 하지 않았다
- 크레딧 구조는 [Licensing and Copilot Credits](https://learn.microsoft.com/en-us/ai-builder/message-management) 문서를 참고

## 써 보고 든 생각

- 설정 화면이 한 장으로 줄어서, 데모용 FAQ 봇은 정말 10분이면 된다. 오래 걸리는 건 **FAQ 문서를 정리하는 일**이다. 질문-답이 분명하게 나뉜 문서일수록 답이 깔끔했다
- 실무에선 파일 업로드보다 **SharePoint 연결**이 맞다. 문서가 바뀌면 다시 올릴 필요가 없기 때문이다. 다만 권한이 그대로 따라오니, 볼 수 없는 문서를 답하지 않는지 꼭 사용자 계정으로 테스트해야 한다
- 고객사에 만들어 줄 때는 **크레딧이 있는 환경인지부터 확인**하자. 데모 당일에 "out of credits"를 보는 건 꽤 민망하다

## 참고

- [Quotas and limits — Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas)
- [Add SharePoint as a knowledge source — Microsoft Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-sharepoint)
- [Licensing and Copilot Credits](https://learn.microsoft.com/en-us/ai-builder/message-management)
- 지난 글: [Copilot Studio Workflows로 이메일 분류·처리 자동화하기](/posts/copilot-studio-workflows-email/)
