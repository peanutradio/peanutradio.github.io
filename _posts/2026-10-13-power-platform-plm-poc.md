---
title: "PLM을 Power Platform으로? 제조사 PoC를 하루 동안 해 본 기록"
date: 2026-10-13 22:30:00 +0900
categories: [Power Platform, PLM]
tags: [power-platform, power-apps, dataverse, power-automate, sharepoint, plm, bom, sap, poc]
description: 전자부품 제조사의 PLM 핵심 기능 4가지(BOM 관리, 설계변경 결재, 문서관리, SAP 전송)를 Power Apps·Dataverse·Power Automate·SharePoint 기본 기능만으로 PoC 했다. 된 것, 안 된 것, 막힌 지점, 본 구축 전에 챙길 라이선스까지 정리했다.
---

> 고객사 보안을 위해 회사명, 제품명, 사내 시스템명, 환경 주소는 지우거나 가렸다. 데이터는 실제 데이터가 아니라 제품 구조를 본뜬 샘플이다.
{: .prompt-info }

## 시작하며 — 왜 이걸 해 봤나

"PLM은 전용 솔루션을 사야 한다"는 말을 현장에서 자주 듣는다. 맞는 말이다. 다만 중견 제조사와 이야기해 보면 정말 필요한 건 생각보다 단순할 때가 많다. **품목과 BOM을 한곳에서 보고, 설계변경을 결재로 돌리고, 도면 버전을 잃어버리지 않고, 확정된 BOM을 SAP로 넘기는 것.** 이미 Microsoft 365를 쓰는 회사라면 이 네 가지를 이미 가진 도구 안에서 할 수 있지 않을까?

말로 설득하는 것보다 직접 만들어 보는 게 빠르다고 생각했다. 그래서 개발자 환경에 앱 1개, 표 4개, 흐름 1개, SharePoint 테스트 사이트 1개를 만들고 하루 동안 끝까지 돌려 봤다. 이 글은 그 기록이다. 잘된 것만이 아니라 막힌 곳과 "이건 기본 기능으로는 안 된다"는 지점까지 같이 적었다.

## 결과부터

| 기능 | 결과 | 숫자로 본 결과 |
|---|---|---|
| ① BOM 관리 | 동작 | 품목 13개, BOM 구성 14건, 역전개 한 화면 |
| ② 설계변경 결재 | 동작 | 승인 요청부터 반영까지 1분 40초 |
| ③ 문서관리 | 동작 | 품목별 SharePoint 폴더, 버전 1.0 → 2.0 기록 |
| ④ SAP 전송 모의 | 동작 | 승인 1건당 SAP 형식 JSON 1건 |

네 가지 모두 **Power Platform 기본 기능만으로** 돌아갔다. 본 구축으로 가려면 풀어야 할 것은 세 가지다. BOM 트리 화면(맞춤 개발), 실제 SAP 연결(게이트웨이·SAP 권한), 사용자별 Power Apps Premium 라이선스.

## 구성 — 전부 Microsoft 365 테넌트 안에서

<figure class="sketch">
<svg viewBox="0 0 720 300" role="img" aria-label="사용자가 Entra ID로 로그인해 Power Apps 앱을 쓰고, 데이터는 Dataverse, 문서는 SharePoint, 결재는 Power Automate와 Teams로 돌며, SAP 연결은 본 구축 때 게이트웨이로 붙인다">
  <defs>
    <marker id="plm-m1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/></marker>
    <marker id="plm-m2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/></marker>
  </defs>
  <text x="0" y="14" class="sk-title">① PoC 구성 — 실선은 이번에 만든 것, 점선은 본 구축 때 붙일 것</text>
  <rect x="96" y="30" width="440" height="262" rx="10" class="sk-box"/>
  <text x="108" y="50" class="sk-sub">Microsoft 365 테넌트 · Power Apps 개발자 환경</text>
  <rect x="4" y="112" width="76" height="52" rx="8" class="sk-box-accent"/>
  <text x="42" y="135" text-anchor="middle" class="sk-label">사용자</text>
  <text x="42" y="152" text-anchor="middle" class="sk-sub">설계·개발</text>
  <path d="M82,138 L122,138" class="sk-line" marker-end="url(#plm-m1)"/>
  <rect x="126" y="108" width="120" height="60" rx="8" class="sk-box-accent"/>
  <text x="186" y="132" text-anchor="middle" class="sk-label">Power Apps</text>
  <text x="186" y="150" text-anchor="middle" class="sk-sub">모델 기반 앱</text>
  <path d="M248,138 L288,138" class="sk-line" marker-end="url(#plm-m1)"/>
  <rect x="292" y="108" width="100" height="60" rx="8" class="sk-box-accent"/>
  <text x="342" y="132" text-anchor="middle" class="sk-label">Dataverse</text>
  <text x="342" y="150" text-anchor="middle" class="sk-sub">품목·BOM·ECO</text>
  <path d="M394,128 L424,128" class="sk-line-accent" marker-end="url(#plm-m2)"/>
  <path d="M424,150 L394,150" class="sk-line-accent" marker-end="url(#plm-m2)"/>
  <rect x="428" y="108" width="100" height="60" rx="8" class="sk-box-accent"/>
  <text x="478" y="132" text-anchor="middle" class="sk-label">Power</text>
  <text x="478" y="150" text-anchor="middle" class="sk-label">Automate</text>
  <path d="M478,106 L478,76" class="sk-line" marker-end="url(#plm-m1)"/>
  <rect x="428" y="56" width="100" height="22" rx="6" class="sk-box"/>
  <text x="478" y="71" text-anchor="middle" class="sk-sub">Teams 결재</text>
  <path d="M342,170 L342,212" class="sk-line" marker-end="url(#plm-m1)"/>
  <rect x="282" y="216" width="120" height="52" rx="8" class="sk-box"/>
  <text x="342" y="238" text-anchor="middle" class="sk-label">SharePoint</text>
  <text x="342" y="256" text-anchor="middle" class="sk-sub">품목별 문서·버전</text>
  <path d="M186,212 L186,170" class="sk-line" marker-end="url(#plm-m1)"/>
  <rect x="126" y="216" width="120" height="52" rx="8" class="sk-box"/>
  <text x="186" y="238" text-anchor="middle" class="sk-label">Entra ID</text>
  <text x="186" y="256" text-anchor="middle" class="sk-sub">로그인·권한</text>
  <path d="M530,138 L586,138" class="sk-line" stroke-dasharray="6 5" marker-end="url(#plm-m1)"/>
  <rect x="590" y="70" width="126" height="196" rx="10" class="sk-box" stroke-dasharray="6 5"/>
  <text x="653" y="92" text-anchor="middle" class="sk-sub">사내 시스템 (본 구축)</text>
  <text x="653" y="128" text-anchor="middle" class="sk-label">게이트웨이</text>
  <text x="653" y="160" text-anchor="middle" class="sk-label">SAP ERP</text>
  <text x="653" y="192" text-anchor="middle" class="sk-label">MDM·ERP</text>
  <text x="653" y="224" text-anchor="middle" class="sk-label">기존 PDM</text>
  <text x="653" y="244" text-anchor="middle" class="sk-sub">1회 이관</text>
</svg>
<figcaption>PLM의 뼈대(데이터·화면·결재·문서)는 이미 가진 M365 안에서 다 돈다. 남는 건 바깥 시스템과의 연결이다.</figcaption>
</figure>

데이터는 이렇게 흐른다.

1. 사용자가 Entra ID로 로그인해 모델 기반 앱 「PLM」에 들어간다.
2. 품목·BOM 구성·설계변경 데이터는 Dataverse 표에 저장되고, 앱에서 역전개까지 바로 조회한다.
3. 품목마다 SharePoint 문서 폴더가 연결되고, 도면·사양서는 버전 기록이 남는다.
4. 설계변경을 '승인 요청'으로 바꾸면 Power Automate 흐름이 시작된다.
5. 결재자에게 메일·Teams로 승인 요청이 가고, 결재자가 승인하거나 반려한다.
6. 승인되면 흐름이 BOM을 바꾸고, SAP로 보낼 내용을 JSON으로 만들어 전송 대기열에 쌓는다.
7. 본 구축 때는 이 대기열을 온프레미스 데이터 게이트웨이와 SAP ERP 커넥터로 SAP에 반영한다.

## ① BOM 관리 — 품목, 다단계 BOM, 주·대체 부품, 역전개

전자 디바이스 2종(2.9인치·4.2인치)을 본뜬 품목 13개와 BOM 구성 14건으로 확인했다.

![품목 목록 — 완제품·반제품·부품 13개](/assets/img/posts/power-platform-plm-poc/02-items.png)
_품목 목록. 완제품·반제품·부품 13개 (제품명·제조사는 가림)_

![메인보드 품목의 BOM과 역전개](/assets/img/posts/power-platform-plm-poc/03-pcb-bom.png)
_PCB-29 메인보드. 하위 부품 3개 중 A사 저항(주 부품)과 C사 저항(대체 부품)이 대체 그룹 R1로 묶여 있고, 맨 아래에 상위 품목 역전개가 보인다_

![저항 부품의 역전개](/assets/img/posts/power-platform-plm-poc/04-where-used.png)
_저항 R-10K-A의 역전개. 두 메인보드(PCB-29, PCB-42)에 쓰이는 것을 한 화면에서 확인_

| 확인 항목 | 결과 | 구현 방식 |
|---|---|---|
| 품목 마스터 | 동작 | Dataverse 표 + 목록·상세 화면 |
| 다단계 BOM | 동작 | BOM 구성 표(상위·하위 품목 관계 2개) |
| 주·대체 부품 | 동작 | 주·대체 구분 + 대체 그룹 열 |
| 역전개(사용처) | 동작 | 같은 표를 하위 품목 기준으로 조회 |
| BOM 전체 트리 한 화면 펼치기 | **기본 화면 없음** | 본 구축 때 맞춤 컨트롤(PCF) 개발 |

역전개는 따로 만들 필요가 없었다. BOM 구성 표 하나에 상위 품목·하위 품목 관계를 둘 다 걸어 두면, 같은 표를 "하위 품목 기준"으로 보여 주는 하위 표만 폼에 추가하면 끝이다. 반대로 **전체 트리를 한 번에 펼치는 화면은 기본 기능에 없다.** 모델 기반 앱은 한 단계씩만 보여 주기 때문에, 설계자가 매일 쓰는 화면이라면 PCF로 따로 만들어야 한다.

## ② 설계변경 결재 — 승인 요청에서 BOM 반영까지 1분 40초

시나리오는 "A사 저항(R-10K-A) 단종 예고에 맞춰 PCB-29의 주 부품을 B사 저항(R-10K-B)으로 바꾸는 건"이다. 설계변경(ECO)의 진행 상태를 '승인 요청'으로 바꾸면 흐름이 결재를 요청하고, 결재자가 Teams에서 승인하면 BOM 변경과 SAP 전송 데이터 적재까지 사람 손 없이 이어졌다. 1분 40초 중 1분 38초는 결재자가 Teams에서 내용을 읽고 버튼을 누른 시간이다.

**1. 설계변경 작성**

![ECO-1000 작성 화면](/assets/img/posts/power-platform-plm-poc/05-eco-draft.png)
_대상 품목, 바꿀 BOM 구성, 변경 후 부품, 변경 사유를 고르고 진행 상태를 '승인 요청'으로 바꿔 저장_

**2. Teams 결재**

![Teams 승인 카드](/assets/img/posts/power-platform-plm-poc/06-teams-card.png)
_결재자 Teams에 도착한 승인 카드. 대상 품목, 변경 후 부품, 변경 사유와 ECO 바로가기 링크가 자동으로 담긴다_

![Teams 승인 앱](/assets/img/posts/power-platform-plm-poc/07-teams-approved.png)
_Teams 승인 앱에서 ECO-1000이 '승인됨'으로 바뀐 화면_

**3. 결과 자동 반영**

![승인 후 ECO-1000](/assets/img/posts/power-platform-plm-poc/08-eco-approved.png)
_승인 후 ECO-1000. 진행 상태 '승인', 결재 의견과 승인 일시가 채워지고 아래 SAP 전송 대기열에 1건이 생겼다_

![변경 후 BOM](/assets/img/posts/power-platform-plm-poc/09-bom-after.png)
_변경 후 PCB-29 BOM. 순번 20의 주 부품이 A사에서 B사 저항으로 바뀌었고, 대체 부품(C사)은 그대로다_

**4. Power Automate 흐름**

흐름은 하나다. 트리거 1개에 단계 11개, 연결은 Dataverse와 표준 승인(Standard approvals) 두 개만 쓴다.

![흐름 개요](/assets/img/posts/power-platform-plm-poc/10-flow-overview.png)
_흐름 개요. 상태 '켜짐', 연결 2개, 실행 기록에 성공 1건과 결재 대기 1건_

![흐름 실행 기록](/assets/img/posts/power-platform-plm-poc/11-flow-run.png)
_ECO-1000 실행 기록. 결재 대기 1분 38초를 빼면 나머지 단계는 다 합쳐 2초가 안 걸렸다_

![트리거 설정](/assets/img/posts/power-platform-plm-poc/12-flow-trigger.png)
_트리거는 설계변경 표의 '진행 상태' 열만 지켜보다가 값이 '승인 요청'이 될 때만 실행된다_

![결재 단계 설정](/assets/img/posts/power-platform-plm-poc/13-flow-approval.png)
_결재 단계는 '먼저 응답한 사람이 결정'하는 승인/거부 방식. 제목과 본문에 ECO 번호·품목·변경 사유가 자동으로 들어간다_

트리거에서 **Select columns**와 **Filter rows**를 같이 건 게 포인트다. 설계변경 행이 수정될 때마다 흐름이 도는 게 아니라, 진행 상태 열이 바뀌고 그 값이 '승인 요청'일 때만 돈다. 이걸 안 걸면 결재 결과를 쓰는 흐름 자신의 업데이트에 다시 반응해 실행 기록이 지저분해진다.

| 확인 항목 | 결과 | 구현 방식 |
|---|---|---|
| 승인 요청 시 자동 시작 | 동작 | Dataverse 트리거(진행 상태 = 승인 요청) |
| Teams 결재 | 동작 | 표준 승인 커넥터, Teams 승인 앱 |
| 승인 시 BOM 자동 변경 | 동작 | Dataverse 행 업데이트 |
| 반려 처리 | 구성만 함 | 반려 분기는 만들었고 실행 테스트는 안 함 |
| 다단계 결재선(팀장 → 임원) | PoC 범위 밖 | 본 구축 때 순차 승인 유형으로 확장 |

## ③ 문서관리 — 품목마다 SharePoint 폴더, 버전은 자동

품목마다 SharePoint 폴더가 붙고, 앱의 품목 화면 '문서' 탭에서 바로 올리고 열 수 있다. 같은 이름으로 다시 올리면 SharePoint가 이전 버전을 자동으로 남긴다. 테스트용 SharePoint 사이트를 새로 만들어 Dataverse와 서버 기반으로 연결했다.

![품목 화면의 문서 탭](/assets/img/posts/power-platform-plm-poc/14-docs-tab.png)
_품목 화면의 문서 탭. SharePoint 폴더에 있는 회로도 PDF와 BOM 파일이 그대로 보이고, 여기서 새 문서 만들기와 업로드가 된다_

![SharePoint의 같은 폴더](/assets/img/posts/power-platform-plm-poc/15-sp-folder.png)
_SharePoint의 같은 폴더. '품목' 라이브러리 아래 '품목명_ID' 폴더로 정리되고, 회로도는 버전 2.0이다_

![회로도 PDF 버전 기록](/assets/img/posts/power-platform-plm-poc/16-sp-versions.png)
_회로도 PDF 버전 기록. Rev.A(1.0) 위에 Rev.B(2.0)를 올려도 1.0이 그대로 남는다_

| 확인 항목 | 결과 | 구현 방식 |
|---|---|---|
| 품목별 폴더 자동 연결 | 동작 | 서버 기반 SharePoint 통합, 품목 표 문서 관리 사용 |
| 앱에서 업로드·열기 | 동작 | 품목 화면 문서 탭 |
| 버전 기록 | 동작 | SharePoint 라이브러리 버전 관리 |
| 도면 배포 통제(체크아웃, 열람 권한) | PoC 범위 밖 | 본 구축 때 권한 설계와 함께 |
| CAD 파일 연동·미리보기 | 제외 | 별도 CAD 연동 필요 |

## ④ SAP 전송 모의 — 대기열에 JSON으로 쌓기

승인이 나면 흐름이 바뀐 BOM을 다시 읽어 SAP BOM 형식(헤더 + 품목 목록)으로 바꾸고, 'SAP 전송 대기열' 표에 JSON 1건으로 쌓는다. 실제 SAP에는 연결하지 않았다. 본 구축 때 이 대기열을 온프레미스 데이터 게이트웨이와 SAP ERP 커넥터가 가져가 SAP에 반영하는 구조다.

- 품목 매핑: BOM 구성의 순번·하위 품목·수량·대체 그룹 → SAP 품목 필드 `POSNR`·`IDNRK`·`MENGE`·`ALPGR`
- 헤더: `MATNR`(자재), `WERKS`(플랜트), `REVLV`(리비전), `AENNR`(변경번호)
- 대기열 상태: 대기 → 전송 완료 또는 오류

대기열을 한 번 거치게 한 이유는 단순하다. SAP가 잠깐 안 받아도 결재 결과가 사라지지 않고, 어떤 ECO가 언제 SAP에 들어갔는지 PLM 쪽에서 추적할 수 있기 때문이다. 플랜트와 BOM 용도 값은 PoC에서는 임시값을 넣었고, 실제 값은 고객사 SAP 기준으로 다시 맞춰야 한다.

## 막힌 곳과 푼 방법

매끄럽기만 했던 건 아니다.

- **승인자(approvedBy)가 비어서 나왔다.** 첫 실행에서 SAP 전송 데이터의 승인자 값이 빈칸이었다. 실행 기록을 열어 보니 승인 결과의 승인자 이름이 생각한 위치가 아니라 `responder` 쪽에 들어 있었다. 식을 그 경로로 고쳤고, 고친 흐름은 두 번째 건(ECO-1001) 결재로 다시 확인하는 중이다.
- **SharePoint '문서 관리 설정' 마법사가 마지막 단계에서 멈췄다.** 브라우저에서 계속 진행이 안 돼서, 같은 결과(품목 라이브러리와 연결 정보)를 API로 만들었다. PCB-42는 문서 탭을 처음 여는 순간 폴더와 연결이 자동으로 생기는 것까지 확인했다. 본 구축 때는 관리자 화면에서 정상 절차로 다시 하면 된다.
- **시간이 9시간씩 어긋나 보였다.** 개발자 환경의 앱과 SharePoint 사이트 시간대가 기본값(UTC)이라 캡처 속 시간이 한국 시간과 다르다. 운영 환경은 처음부터 시간대를 맞춰 두자.
- **BOM 트리가 한 화면에 안 펼쳐진다.** 위에서 말한 대로 기본 기능의 한계다. 설계자 입장에서는 가장 먼저 아쉬워할 부분이라 본 구축 견적에 PCF 개발을 꼭 넣어야 한다.

## 본 구축 전에 챙길 것

| 항목 | PoC에서 본 것 | 본 구축 때 할 일 |
|---|---|---|
| BOM 트리 한 화면 보기 | 기본 화면은 한 단계씩만 보임 | 맞춤 컨트롤(PCF) 개발 |
| 결재선 | 1인 결재 | 결재선 표 설계, 순차 승인·대결 규칙 |
| 리비전 | ECO 승인 때 리비전을 올리지 않음 | 리비전 규칙 확정 후 흐름에 추가 |
| SAP 실제 반영 | 대기열 적재까지만 | 게이트웨이 서버, SAP .NET Connector, SAP 계정의 RFC·BAPI 권한 |
| 라이선스·환경 | 무료 개발자 환경(개발·테스트 전용) | 사용자별 Power Apps Premium, 운영 환경 따로 |

**라이선스는 가장 먼저 계산해야 한다.** 모델 기반 앱은 Dataverse를 쓰기 때문에 기본이 프리미엄 앱이고, 쓰는 사람 모두에게 Power Apps 프리미엄 사용 권한이 필요하다. Microsoft는 **2027년 2월부터** 이 요건을 엄격하게 확인해, 라이선스가 없는 사용자는 앱을 열 수 없게 한다고 [라이선스 FAQ](https://learn.microsoft.com/power-platform/admin/powerapps-licensing-faq)에 밝혀 두었다. PoC는 무료 [개발자 플랜](https://learn.microsoft.com/power-platform/developer/plan)으로 충분하지만, 운영은 다른 이야기다.

## 마치며 — 개인적인 생각

해 보고 나니 생각이 정리됐다. **PLM의 뼈대는 Power Platform 기본 기능으로 충분히 선다.** 품목·BOM·결재·문서·전송 대기열까지 코드 한 줄 없이 하루 만에 끝까지 이어졌고, 결재자는 새 시스템을 배울 필요 없이 매일 쓰는 Teams에서 버튼 하나만 누르면 된다. 현업 입장에서는 이게 제일 큰 장점이라고 본다.

반대로 이번 PoC가 보여 준 건 "다 된다"가 아니라 **"어디까지 되고 어디부터 돈이 드는지"**다. BOM 트리 화면, SAP 실제 연결, 사용자 수만큼의 Premium 라이선스. 본 구축 비용은 결국 이 세 가지가 정한다. 고객에게 "로우코드라 싸다"고 말하기 전에 이 셈부터 같이 해 보는 게 맞다고 생각한다.

본 구축으로 가기 전에 고객과 먼저 맞출 것은 세 가지다. SAP 기준값(플랜트, BOM 용도)과 연결 방식, 설계변경 결재선, 도면·문서 분류와 열람 권한. 이 셋이 정해지면 앱·흐름·문서관리는 바로 상세 설계로 넘어갈 수 있다.

## 참고한 공식 문서

- [Power Apps licensing FAQs](https://learn.microsoft.com/power-platform/admin/powerapps-licensing-faq) — 모델 기반 앱 프리미엄 라이선스, 2027년 2월 확인 강화
- [Power Apps developer plan](https://learn.microsoft.com/power-platform/developer/plan) — 무료 개발자 환경
- [Create, manage, and publish model-driven apps using code](https://learn.microsoft.com/power-apps/developer/model-driven-apps/create-manage-model-driven-apps-using-code)
- [Work with cloud flows using code](https://learn.microsoft.com/power-automate/manage-flows-with-code)
- [Get started with approvals](https://learn.microsoft.com/power-automate/get-started-approvals) — 승인 유형(먼저 응답, 전원 승인, 순차 승인)
- [Set up SharePoint Online integration](https://learn.microsoft.com/power-platform/admin/set-up-dynamics-365-online-to-use-sharepoint-online)
- [Add the SharePoint documents tab to the main form](https://learn.microsoft.com/power-apps/maker/model-driven-apps/add-documents-tab-entity-main-form)
- [Power Platform and SAP connection options](https://learn.microsoft.com/power-platform/sap/connect/connect-power-platform-and-sap) — SAP ERP 커넥터(RFC·BAPI)와 온프레미스 데이터 게이트웨이
