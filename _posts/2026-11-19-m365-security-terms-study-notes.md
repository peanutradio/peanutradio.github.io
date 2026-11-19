---
title: "M365 보안 용어 공부 노트 — 반출·SIT·대외비 기본 레이블·Copilot 사이드챗"
date: 2026-11-19 19:50:00 +0900
categories: [Study Notes, M365 보안]
tags: [microsoft-365, security, copilot, governance]
description: 망분리 회사에서 Copilot 도입을 이야기할 때 자주 나오는 용어 다섯 가지를 그림과 함께 정리했다. 반출, 문서반출 경유, SIT, 대외비 기본 레이블, Copilot 사이드챗.
---

망분리 환경의 회사에서 Microsoft 365 Copilot 도입 이야기를 따라가다 보면, 낯선 단어가 한꺼번에 쏟아진다.
나도 처음 들었을 때 헷갈렸던 다섯 개를 공부하면서 정리해 둔다.

결론부터 말하면, 다섯 단어는 전부 한 질문으로 모인다.
**"회사 문서가 회사 밖(클라우드 포함)으로 나갈 때, 무엇으로 찾고 · 막고 · 기록할 것인가."**

---

## 1. 반출, 그리고 '문서반출 경유'

### 반출

'반출'은 회사 안에 있는 문서나 파일을 회사 밖으로 내보내는 것이다. 반대로 밖에서 안으로 들여오는 건 '반입'이다.

- **흔한 반출 경로**

  USB에 담아 가져가기, 외부 메일에 첨부하기, 개인 클라우드나 외부 사이트에 올리기, 망분리 환경에서 업무망 파일을 인터넷망 PC로 옮기기.

### 문서반출 경유

'경유'는 '거쳐서 간다'는 뜻이다. 그래서 **문서반출 경유**는 파일을 바로 내보내지 못하고, 반드시 문서반출 시스템을 거쳐야 나갈 수 있다는 말이다.
국내 대기업·금융권에서는 이런 별도 솔루션을 두는 경우가 많고, 하는 일은 대체로 세 가지다.

- **결재** — 신청을 받고 팀장 결재·보안 검토를 거친다
- **검사** — 파일 안에 민감 정보가 있는지 본다
- **기록** — 누가·언제·어떤 파일을 내보냈는지 남긴다

<figure class="sketch">
<svg viewBox="0 0 720 230" role="img" aria-label="직접 반출과 문서반출 경유 비교 — 경유하면 결재, 검사, 기록 단계를 반드시 거친다">
  <defs>
    <marker id="ex-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ex-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 직접 반출 vs 문서반출 경유</text>

  <rect x="8" y="40" width="120" height="48" rx="8" class="sk-box"/>
  <text x="68" y="62" text-anchor="middle" class="sk-label">내 PC</text>
  <text x="68" y="78" text-anchor="middle" class="sk-sub">업무망</text>
  <path d="M132,64 L574,64" class="sk-line" stroke-dasharray="6 6" marker-end="url(#ex-m1)"/>
  <text x="350" y="56" text-anchor="middle" class="sk-sub">✕ 직접 — 막혀 있음</text>
  <rect x="584" y="40" width="128" height="48" rx="8" class="sk-box-muted"/>
  <text x="648" y="62" text-anchor="middle" class="sk-label">회사 밖</text>
  <text x="648" y="78" text-anchor="middle" class="sk-sub">메일·클라우드·USB</text>

  <rect x="8" y="140" width="120" height="48" rx="8" class="sk-box"/>
  <text x="68" y="162" text-anchor="middle" class="sk-label">내 PC</text>
  <text x="68" y="178" text-anchor="middle" class="sk-sub">반출 신청</text>
  <path d="M132,164 L218,164" class="sk-line-accent" marker-end="url(#ex-a1)"/>
  <rect x="226" y="118" width="268" height="92" rx="10" class="sk-box-accent"/>
  <text x="360" y="142" text-anchor="middle" class="sk-label">문서반출 시스템</text>
  <text x="360" y="164" text-anchor="middle" class="sk-sub">① 결재·승인</text>
  <text x="360" y="182" text-anchor="middle" class="sk-sub">② 민감 정보 검사</text>
  <text x="360" y="200" text-anchor="middle" class="sk-sub">③ 누가·언제·무엇을 기록</text>
  <path d="M498,164 L576,164" class="sk-line-accent" marker-end="url(#ex-a1)"/>
  <text x="537" y="154" text-anchor="middle" class="sk-sub">○ 승인분만</text>
  <rect x="584" y="140" width="128" height="48" rx="8" class="sk-box"/>
  <text x="648" y="162" text-anchor="middle" class="sk-label">회사 밖</text>
  <text x="648" y="178" text-anchor="middle" class="sk-sub">기록이 남은 채로</text>
</svg>
<figcaption>반출을 '못 하게' 막는 게 아니라, 반드시 한 관문을 지나가게 만들어 통제와 기록을 남기는 것이다.</figcaption>
</figure>

---

## 2. SIT — 민감 정보를 '찾는 패턴'

SIT는 Sensitive Information Type(민감 정보 유형)의 줄임말이다.
Microsoft Purview 문서는 SIT를 **패턴 기반 분류기**라고 설명한다. 주민등록번호·신용카드 번호·계좌번호처럼 형식이 정해진 정보를 문서 안에서 찾아낸다.
Microsoft가 미리 만들어 둔 SIT가 많고, 회사가 직접 만들 수도 있다.

처음에 내가 헷갈렸던 건 SIT와 레이블의 차이였다. 역할을 동사로 나누면 깔끔해진다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="SIT는 찾고, DLP는 막고, 민감도 레이블은 붙인다 — 세 기능의 순서">
  <defs>
    <marker id="sit-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 찾고 → 막고 → 붙인다</text>

  <rect x="8" y="50" width="200" height="96" rx="10" class="sk-box-accent"/>
  <text x="108" y="78" text-anchor="middle" class="sk-label">SIT · 찾는다</text>
  <text x="108" y="100" text-anchor="middle" class="sk-sub">"주민번호 형식이 있네"</text>
  <text x="108" y="122" text-anchor="middle" class="sk-mono">패턴 + 키워드</text>

  <path d="M212,98 L254,98" class="sk-line" marker-end="url(#sit-m1)"/>

  <rect x="260" y="50" width="200" height="96" rx="10" class="sk-box"/>
  <text x="360" y="78" text-anchor="middle" class="sk-label">DLP · 막는다</text>
  <text x="360" y="100" text-anchor="middle" class="sk-sub">"외부 메일 발송 차단"</text>
  <text x="360" y="122" text-anchor="middle" class="sk-sub">차단 · 경고 · 기록</text>

  <path d="M464,98 L506,98" class="sk-line" marker-end="url(#sit-m1)"/>

  <rect x="512" y="50" width="200" height="96" rx="10" class="sk-box"/>
  <text x="612" y="78" text-anchor="middle" class="sk-label">레이블 · 붙인다</text>
  <text x="612" y="100" text-anchor="middle" class="sk-sub">"이 문서는 기밀"</text>
  <text x="612" y="122" text-anchor="middle" class="sk-sub">꼬리표 + 암호화·권한</text>

  <text x="360" y="184" text-anchor="middle" class="sk-sub">비유: X-ray가 칼 모양을 알아보는 기준(SIT) → 검색대에서 멈춤(DLP) → 짐에 '주의' 스티커(레이블)</text>
</svg>
<figcaption>SIT는 혼자서 아무것도 막지 않는다. DLP·자동 레이블 같은 정책이 SIT의 결과를 받아 행동한다.</figcaption>
</figure>

Purview 문서에 따르면 SIT는 DLP 정책, 민감도 레이블, 보존 레이블, 자동 레이블 정책 등 여러 기능에서 공통으로 쓰인다.
그래서 SIT를 잘 만들어 두면 여러 정책이 같은 기준으로 움직인다.

---

## 3. '대외비 기본 레이블'

처음엔 나도 "밖으로 나가는 문서에 기본으로 붙는 레이블인가?"라고 이해했다. **반대다.**

'대외비'는 **외부에 공개하면 안 되는 문서**라는 뜻이다(대외 = 바깥, 비 = 비밀).
'대외비 기본 레이블'은 직원이 문서나 메일을 새로 만들면, 따로 고르지 않아도 처음부터 '대외비'가 붙도록 설정했다는 뜻이다.

Purview의 레이블 정책에는 실제로 이런 옵션이 있다.

- **기본 레이블 지정**

  레이블이 없는 문서·메일·회의 초대 등에 자동으로 붙일 레이블을 고른다. 사용자는 나중에 바꿀 수 있다.

- **레이블을 낮출 때 사유 입력**

  기밀 → 일반처럼 우선순위가 낮은 레이블로 바꾸거나 레이블을 지우면 사유를 적게 할 수 있다. 관리자는 활동 탐색기에서 그 사유를 볼 수 있다.

<figure class="sketch">
<svg viewBox="0 0 720 210" role="img" aria-label="기본 레이블이 대외비일 때 문서 흐름 — 새 문서는 대외비로 시작하고, 밖으로 보내려면 사유를 남기고 일반으로 낮춘다">
  <defs>
    <marker id="lb-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="lb-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">③ 모든 문서는 '대외비'로 시작한다</text>

  <rect x="8" y="60" width="140" height="56" rx="8" class="sk-box"/>
  <text x="78" y="84" text-anchor="middle" class="sk-label">새 문서·메일</text>
  <text x="78" y="102" text-anchor="middle" class="sk-sub">레이블 안 고름</text>
  <path d="M152,88 L198,88" class="sk-line-accent" marker-end="url(#lb-a1)"/>

  <rect x="206" y="52" width="170" height="72" rx="10" class="sk-box-accent"/>
  <text x="291" y="80" text-anchor="middle" class="sk-label">대외비 (기본값)</text>
  <text x="291" y="100" text-anchor="middle" class="sk-sub">깜빡해도 보호된 상태</text>

  <path d="M380,88 L426,88" class="sk-line" marker-end="url(#lb-m1)"/>
  <rect x="434" y="52" width="140" height="72" rx="8" class="sk-box"/>
  <text x="504" y="78" text-anchor="middle" class="sk-label">일반으로 낮춤</text>
  <text x="504" y="98" text-anchor="middle" class="sk-sub">사유 입력 → 기록</text>

  <path d="M578,88 L620,88" class="sk-line" marker-end="url(#lb-m1)"/>
  <rect x="628" y="60" width="84" height="56" rx="8" class="sk-box"/>
  <text x="670" y="84" text-anchor="middle" class="sk-label">외부</text>
  <text x="670" y="102" text-anchor="middle" class="sk-sub">반출 절차</text>

  <rect x="206" y="150" width="368" height="44" rx="8" class="sk-box-muted"/>
  <text x="390" y="170" text-anchor="middle" class="sk-sub">기본값이 '일반'이면? 깜빡한 문서가 무방비로 나갈 수 있다</text>
  <text x="390" y="186" text-anchor="middle" class="sk-sub">→ 그래서 '보호된 쪽'을 기본값으로 둔다</text>
</svg>
<figcaption>대외비 기본 레이블은 '나가는 문서 표시'가 아니라 '모든 문서는 일단 외부 공개 금지'라는 출발점이다.</figcaption>
</figure>

한 가지 주의할 점도 문서에 나온다. **암호화가 걸린 레이블을 기본값으로 두는 건 보통 권하지 않는다.**
외부와 문서를 주고받아야 하는 일이 많으면, 상대방이 암호화된 문서를 못 여는 문제가 생기기 때문이다.
그래서 기본값은 '표시만 하는 레이블'로 두고, 암호화는 '기밀' 같은 상위 레이블에 거는 구성을 흔히 쓴다.

---

## 4. Copilot 사이드챗

### 사이드챗이란

Word·Excel·PowerPoint·Outlook 같은 데스크톱 앱에서 리본의 Copilot 버튼을 누르면 오른쪽에 열리는 채팅 창이다.
"이 문서 요약해 줘", "이 표로 차트 만들어 줘"처럼 **지금 열린 파일**을 바로 다룬다.
Power Automate Desktop 같은 다른 앱에도 Copilot 창이 붙어 있지만, 보통 '사이드챗'이라고 하면 Office 앱 안의 Copilot을 가리킨다.

### 망분리 회사에서 자주 빠지는 이유

Microsoft 문서는 데스크톱 앱에서 Copilot을 쓰려면 **Microsoft 365 Apps**(구독형 Office)를 배포하라고 안내한다.
설치형 영구 버전(LTSC)을 쓰는 회사라면 앱 안의 Copilot은 처음부터 선택지에 없다.
그러면 남는 건 웹 브라우저나 Copilot 앱에서 쓰는 **Copilot Chat**이다.

### 그리고 다시 OneDrive 문제로

Copilot Chat에 파일을 올리면, 그 파일은 **사용자의 OneDrive 안 폴더**에 저장된다(Microsoft 문서 기준).
그래서 망분리 회사에서는 "Copilot을 열어 줄까?"라는 질문이 곧 **"OneDrive를 열어 줄까?"**라는 질문으로 바뀐다.
보안 담당자 입장에서는 이것도 일종의 반출이기 때문이다.

<figure class="sketch">
<svg viewBox="0 0 720 200" role="img" aria-label="Copilot Chat에 파일을 올리면 OneDrive에 저장되므로 OneDrive 정책이 쟁점이 된다">
  <defs>
    <marker id="cp-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="cp-a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">④ Copilot 질문이 OneDrive 질문이 되는 이유</text>

  <rect x="8" y="50" width="150" height="60" rx="8" class="sk-box-muted"/>
  <text x="83" y="76" text-anchor="middle" class="sk-label">Office 앱 사이드챗</text>
  <text x="83" y="94" text-anchor="middle" class="sk-sub">설치형 Office면 없음</text>

  <rect x="8" y="126" width="150" height="60" rx="8" class="sk-box"/>
  <text x="83" y="152" text-anchor="middle" class="sk-label">Copilot Chat</text>
  <text x="83" y="170" text-anchor="middle" class="sk-sub">웹·Copilot 앱</text>

  <path d="M162,156 L228,156" class="sk-line" marker-end="url(#cp-m1)"/>
  <text x="195" y="146" text-anchor="middle" class="sk-sub">파일 업로드</text>

  <rect x="236" y="120" width="190" height="72" rx="10" class="sk-box-accent"/>
  <text x="331" y="148" text-anchor="middle" class="sk-label">사용자 OneDrive</text>
  <text x="331" y="168" text-anchor="middle" class="sk-sub">업로드 파일 저장 폴더</text>

  <path d="M430,156 L496,156" class="sk-line-accent" marker-end="url(#cp-a1)"/>

  <rect x="504" y="104" width="208" height="88" rx="10" class="sk-box"/>
  <text x="608" y="130" text-anchor="middle" class="sk-label">보안팀의 질문</text>
  <text x="608" y="150" text-anchor="middle" class="sk-sub">OneDrive를 어디까지 열까?</text>
  <text x="608" y="168" text-anchor="middle" class="sk-sub">감사 로그는 어디서 보나?</text>
  <text x="608" y="184" text-anchor="middle" class="sk-sub">레이블·DLP로 막을 수 있나?</text>
</svg>
<figcaption>Copilot 도입 논의의 상당 부분은 사실 'Copilot'이 아니라 '클라우드 저장소를 얼마나 열 것인가'에 대한 논의다.</figcaption>
</figure>

참고로 Microsoft는 최근 이름을 정리했다. 문서에서는 Microsoft 365 Copilot을 **Microsoft Copilot**, Microsoft 365 Copilot Chat을 **Microsoft Copilot Chat**으로 부른다.
예전 자료와 새 자료를 같이 볼 때 같은 제품이라는 걸 알아 두면 덜 헷갈린다.

---

## 한 번에 정리

| 용어 | 한 줄 뜻 | 기억할 동사 |
|---|---|---|
| 반출 | 회사 밖으로 파일을 내보내는 것 | 나간다 |
| 문서반출 경유 | 반드시 반출 시스템을 거쳐서만 나감 | 거친다 |
| SIT | 민감 정보를 찾아내는 패턴 | 찾는다 |
| DLP | 찾은 결과로 공유·발송을 막는 정책 | 막는다 |
| 대외비 기본 레이블 | 모든 새 문서가 '외부 공개 금지'로 시작 | 시작한다 |
| Copilot 사이드챗 | Office 앱 안 오른쪽 Copilot 창 | 앱 안에서 |

다음에는 같은 흐름에서 자주 나오는 **조건부 액세스(CA)**와 **감사(Audit) 로그**를 이어서 정리해 볼 생각이다.

---

## 참고

- [Learn about sensitive information types — Microsoft Learn](https://learn.microsoft.com/purview/sit-sensitive-information-type-learn-about)
- [Learn about sensitivity labels: What label policies can do — Microsoft Learn](https://learn.microsoft.com/purview/sensitivity-labels#what-label-policies-can-do)
- [Default sensitivity labels and policies — Microsoft Learn](https://learn.microsoft.com/purview/default-sensitivity-labels-policies)
- [Microsoft Copilot requirements — Microsoft Learn](https://learn.microsoft.com/copilot/plan/microsoft-copilot-requirements)
- [Data protection and auditing for Microsoft Copilot — Microsoft Learn](https://learn.microsoft.com/copilot/plan/architecture-data-protection-auditing)
