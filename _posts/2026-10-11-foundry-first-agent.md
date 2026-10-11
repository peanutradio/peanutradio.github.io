---
title: "Foundry Agent Service로 첫 에이전트 만들기 — 파일 검색부터 코드 호출까지"
date: 2026-10-11 18:55:00 +0900
categories: [Azure AI]
tags: [foundry, agent-service, file-search, azure-openai, hands-on]
description: 새 Microsoft Foundry 포털에서 노코드로 에이전트를 만들고, FAQ 파일을 올려 파일 검색으로 답하게 한 뒤, 같은 에이전트를 Python 코드에서 부르는 데까지 실제 화면으로 정리했다. 기본으로 켜지는 웹 검색 요금 주의점도 적었다.
---

**결과부터.** Foundry 포털에서 에이전트를 하나 만들고, 사내 IT FAQ PDF를 올려 **파일 검색(File search)** 도구로 답하게 했다. "집에서 사내 시스템에 접속하려면?"이라는 질문에 VPN 앱 이름과 MFA 절차를 문서에서 찾아 답했고, 문장마다 출처 번호가 달렸다. 질문 한 번에 든 토큰은 약 3,400개였다. 마지막으로 "코드에서 계속"을 누르면 같은 에이전트를 부르는 Python 코드가 나온다.

[Copilot Studio에서 같은 FAQ 봇을 만든 글](/posts/copilot-studio-faq-agent/)과 비교해 보면, Foundry는 **모델·도구·버전을 개발자가 직접 쥐는 쪽**이다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="Foundry 프로젝트 안에 모델 배포와 에이전트가 있고, 에이전트는 파일 검색 도구로 벡터 인덱스를 조회하며, 포털과 코드 양쪽에서 같은 에이전트를 부른다">
  <defs>
    <marker id="ffa-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/></marker>
  </defs>
  <text x="0" y="14" class="sk-title">① Foundry 안에서 무엇이 무엇을 부르나</text>
  <rect x="160" y="30" width="400" height="160" rx="10" class="sk-box-muted"/>
  <text x="360" y="50" text-anchor="middle" class="sk-sub">Foundry 프로젝트</text>
  <rect x="180" y="64" width="150" height="52" rx="8" class="sk-box-accent"/>
  <text x="255" y="87" text-anchor="middle" class="sk-label">에이전트</text>
  <text x="255" y="105" text-anchor="middle" class="sk-sub">지침 + 버전</text>
  <rect x="390" y="64" width="150" height="52" rx="8" class="sk-box"/>
  <text x="465" y="87" text-anchor="middle" class="sk-label">모델 배포</text>
  <text x="465" y="105" text-anchor="middle" class="sk-mono">gpt-5-mini</text>
  <rect x="180" y="128" width="150" height="50" rx="8" class="sk-box"/>
  <text x="255" y="150" text-anchor="middle" class="sk-label">파일 검색 도구</text>
  <text x="255" y="167" text-anchor="middle" class="sk-sub">→ 벡터 인덱스</text>
  <path d="M332,90 L386,90" class="sk-line" marker-end="url(#ffa-m1)"/>
  <path d="M255,118 L255,126" class="sk-line" marker-end="url(#ffa-m1)"/>
  <rect x="8" y="64" width="120" height="44" rx="8" class="sk-box"/>
  <text x="68" y="91" text-anchor="middle" class="sk-label">포털 플레이그라운드</text>
  <rect x="8" y="128" width="120" height="44" rx="8" class="sk-box"/>
  <text x="68" y="155" text-anchor="middle" class="sk-label">Python 코드</text>
  <path d="M130,86 L176,88" class="sk-line" marker-end="url(#ffa-m1)"/>
  <path d="M130,150 L176,100" class="sk-line" marker-end="url(#ffa-m1)"/>
  <rect x="590" y="64" width="124" height="112" rx="8" class="sk-box"/>
  <text x="652" y="92" text-anchor="middle" class="sk-label">비용이 나는 곳</text>
  <text x="652" y="114" text-anchor="middle" class="sk-sub">모델 토큰</text>
  <text x="652" y="132" text-anchor="middle" class="sk-sub">벡터 저장 용량</text>
  <text x="652" y="150" text-anchor="middle" class="sk-sub">웹 검색(Bing)</text>
</svg>
<figcaption>에이전트는 "모델 배포를 쓰는 설정 묶음"이다. 돈은 에이전트가 아니라 모델·저장소·도구에서 나간다.</figcaption>
</figure>

## 준비물

- Foundry 프로젝트와 **모델 배포 1개.** 나는 이미 있던 `gpt-5-mini`(글로벌 표준) 배포를 그대로 썼다. 새 리소스는 만들지 않았다
- 답하게 할 문서 1개(가상 회사 FAQ PDF)

## 1. 에이전트 만들기

포털 홈의 ① **에이전트 빌드 → 빌드 시작**을 누른다. 화면 아래쪽의 API 키와 엔드포인트는 가렸다.

![Foundry 홈](/assets/img/posts/foundry-first-agent/01-home.png)
_① 에이전트 빌드 → 빌드 시작. API 키·엔드포인트는 캡처에 남기지 않도록 주의_

① 이름 ② 상호 작용 모드(텍스트/음성)를 정하고 ③ 만들기. **상호 작용 모드는 만든 뒤에 못 바꾼다**고 적혀 있다.

![에이전트 만들기 창](/assets/img/posts/foundry-first-agent/02-create.png)
_① 이름 ② 텍스트 모드 ③ 만들기_

## 2. 플레이그라운드: 모델과 기본 도구

만들자마자 플레이그라운드가 열린다. ① 모델은 프로젝트에 배포된 것 중에서 고른다. 그런데 ② **도구에 "웹 검색"이 기본으로 붙어 있다.** 안내 문구대로 이건 Grounding with Bing을 쓰고 **추가 비용**과 별도 약관이 적용된다. 사내 문서 봇이라 지웠다.

![플레이그라운드 첫 화면](/assets/img/posts/foundry-first-agent/03-playground.png)
_① 모델 선택 ② 기본으로 들어가 있는 웹 검색 — 데모에선 제거_

## 3. 지침과 파일 업로드

① 지침을 쓰고 ② **파일 업로드**를 누른다.

![지침 입력](/assets/img/posts/foundry-first-agent/04-instructions.png)
_① 지침 ② 파일 업로드_

파일을 올리면 **벡터 인덱스**가 새로 만들어진다. ① 인덱스 이름(자동 생성) ② 업로드 상태가 "성공"이 되면 ③ 연결.

![파일 첨부 창](/assets/img/posts/foundry-first-agent/05-upload.png)
_① 새 벡터 인덱스 ② 업로드 성공 ③ 연결_

연결하면 도구에 ① **파일 검색**이 붙고, 저장하면 ② 버전이 2로 올라간다. 에이전트를 고칠 때마다 버전이 쌓이는 구조라, 운영 중인 버전과 실험 버전을 나눠 관리할 수 있다.

![파일 검색 도구가 붙은 상태](/assets/img/posts/foundry-first-agent/06-file-search.png)
_① 파일 검색 + 벡터 인덱스 ② 저장하면 버전 2_

## 4. 테스트

"집에서 사내 시스템에 접속하려면 어떻게 해요?"라고 물었다. 문서의 VPN 항목을 찾아 답했고 ① 문장마다 출처(FAQ.pdf)가 붙었다. ② 오른쪽 위에 이번 대화의 토큰 수(3,404)가 바로 보인다. 비용 감을 잡기에 좋다.

![파일 검색 답변](/assets/img/posts/foundry-first-agent/07-answer.png)
_① 출처 표시 ② 사용 토큰 수_

## 5. 코드에서 같은 에이전트 부르기

위쪽 **코드에서 계속**을 누르면 호출 코드가 나온다. ① 핵심은 `get_openai_client(agent_name=...)` 한 줄이다. 포털에서 만든 에이전트를 이름으로 불러서, OpenAI Responses API 모양 그대로 쓴다.

![코드에서 계속](/assets/img/posts/foundry-first-agent/08-code.png)
_① 에이전트 이름으로 OpenAI 클라이언트를 받는다. 엔드포인트는 가렸다_

```python
# pip install "azure-ai-projects>=2.1.0"
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

endpoint = "https://<리소스>.services.ai.azure.com/api/projects/<프로젝트>"

with (
    DefaultAzureCredential() as credential,
    AIProjectClient(endpoint=endpoint, credential=credential, allow_preview=True) as project_client,
):
    openai_client = project_client.get_openai_client(agent_name="blog-demo-it-helper")
    conversation = openai_client.conversations.create()
    response = openai_client.responses.create(
        conversation=conversation.id,
        input=[{"role": "user", "content": "VPN은 언제 써요?"}],
    )
    print(response.output_text)
```

키 대신 `DefaultAzureCredential`(Entra ID)을 쓰는 게 기본값으로 나온다. 로컬에선 `az login`만 돼 있으면 된다.

## 막힌 곳 / 주의할 곳

- **웹 검색 기본 탑재.** 모르고 두면 Bing 그라운딩 요금이 따로 나온다. 사내 문서 봇이면 먼저 지우자
- **API 키가 홈 화면에 그대로 있다.** 블로그·발표용 캡처를 할 땐 홈 화면과 모델 상세 패널을 꼭 가리자
- **정리할 때 에이전트만 지우면 끝이 아니다.** 올린 파일은 "데이터 → 데이터 세트"에 따로 남아 있었다. 실습 후 에이전트와 데이터 세트를 둘 다 지웠다

## 비용

- 모델: 테스트 질문 1번에 약 3,400토큰(포털 표시값). 토큰 단가는 [Azure OpenAI 가격 페이지](https://azure.microsoft.com/pricing/details/cognitive-services/openai-service/)에서 확인
- 벡터 인덱스: 이번 문서는 3KB
- 웹 검색: 제거해서 0

## 써 보고 든 생각

- Copilot Studio가 "업무 담당자가 만드는 에이전트"라면, Foundry는 **"개발자가 만들고 앱에 넣는 에이전트"** 다. 버전, 코드 호출, 토큰 표시가 처음부터 보인다
- 포털에서 지침과 도구를 다듬고 → 코드에선 이름으로만 부르는 흐름이 깔끔하다. 지침을 고쳐도 앱 코드를 다시 배포할 필요가 없다
- 다음엔 같은 에이전트에 AI Search 인덱스(Foundry IQ)를 붙여, 파일 몇 개가 아니라 수천 개 문서에서 찾게 해 볼 생각이다

## 참고

- [What is Microsoft Foundry Agent Service?](https://learn.microsoft.com/en-us/azure/foundry/agents/overview)
- [Get started with Microsoft Foundry SDKs and endpoints](https://learn.microsoft.com/en-us/azure/foundry/how-to/develop/sdk-overview)
- [Data, privacy, and security for Foundry Models](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/openai/data-privacy)
- 관련 글: [Copilot Studio로 사내 IT FAQ 에이전트 만들기](/posts/copilot-studio-faq-agent/)
