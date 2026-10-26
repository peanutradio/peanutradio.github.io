# 태그 정규화 매핑 (2026-10-09)

규칙: 태그는 **소문자 영문 3~6개**. 같은 뜻의 태그는 하나로 합친다.
새 글을 쓸 때 왼쪽 표기를 쓰지 말고 오른쪽 표기를 쓴다.

| 이전 태그 | 바꾼 태그 | 이유 | 적용 글 |
|---|---|---|---|
| `블로그` | `blogging` | 한글 태그 → 영문 | 2026-05-31-welcome |
| `회고` | `retrospective` | 한글 태그 → 영문 | 2026-05-31-welcome |
| `아키텍처` | `architecture` | 한글 태그 → 영문 | 2026-09-07-mcp-vs-api |
| `반도체` | `semiconductor` | 한글 태그 → 영문 | 2026-09-07-samsung-anthropic |
| `foundry` (삼성 파운드리) | `semiconductor` | 반도체 위탁생산 의미 — 위 태그와 중복이라 하나로 합침 | 2026-09-07-samsung-anthropic |
| `foundry` (Microsoft Foundry) | `microsoft-foundry` | 제품명 통일 | 2026-10-08-document-intelligence-vs-content-understanding |
| `ai-foundry` | `microsoft-foundry` | 구 Azure AI Foundry → Microsoft Foundry | 2026-05-31-azure-ai-foundry-intro |
| `agent` | `ai-agent` | 동의어 | 2026-09-07-copilot-studio-workflows-email, 2026-09-08-openai-astra-opaque-recurrence |
| `workflows` | `workflow` | 단수형 통일 | 2026-09-07-copilot-studio-workflows-email |
| `model-context-protocol` | `mcp` | 동의어 (같은 글에 `mcp`가 이미 있어 삭제) | 2026-09-07-mcp-vs-api |
| `m365` | `microsoft-365` | 약어 → 정식 표기 | 2026-09-07-copilot-studio-workflows-email |
| `github-copilot` | `copilot` | Copilot 제품군 태그 하나로 통일 | 2026-10-04-memo-mcp-first-server |
| `claude-code` | `claude` | Claude 제품군 태그 하나로 통일 | 2026-09-08-aspire-coding-agent |
| `microsoft-security` | `security` | 보안 태그 하나로 통일 | 2026-09-27-ms-security-september-2026 |
| `opentelemetry` | `observability` | 상위 개념으로 통일 (Aspire·AQUA 글을 함께 묶음) | 2026-09-08-aspire-coding-agent |
| `monitorability` | `ai-safety` | 같은 글의 `ai-safety`와 중복이라 삭제 | 2026-09-08-openai-astra-opaque-recurrence |

## 결과

| | 이전 | 이후 |
|---|---|---|
| 고유 태그 수 | 88 | 78 |
| 1회만 쓰인 태그 | 74 | 61 |

> 글이 25편뿐이라 1회 태그가 많은 건 자연스럽다. 태그 페이지는 "많이 쓴 태그(2회 이상)"와 "그 외"로 나눠 보여준다.

## 새 글 태그 고를 때

- 먼저 기존 태그에서 고른다: `ai-agent`, `copilot`, `azure`, `mcp`, `github`, `microsoft`, `microsoft-foundry`, `rag`, `claude`, `security`, `observability`, `workflow`, `orchestration`, `google-adk`, `llm`, `openai`
- 제품명은 정식 표기 (`microsoft-365`, `microsoft-foundry`, `microsoft-fabric`)
- 한글 태그 금지, 복수형 금지

---

## 2026-10-26 통합 — 태그 23개로 고정 (애드센스 재심사 대비)

애드센스 불승인('정책 위반 발견', 10/25) 점검에서 사이트맵 325개 중 228개가 태그 페이지였고, 태그 171개가 글 1편에만 쓰였다.
글 하나짜리 목록 화면이 사이트 대부분을 차지해 '가치가 별로 없는 화면'으로 보일 수 있어 정리했다 (사용자 승인 msg 4307).

- 허용 태그 23개: `ai-agent` `mcp` `copilot` `copilot-studio` `microsoft-365` `azure` `foundry` `security` `identity` `governance` `rag` `llm` `openai` `claude` `github` `devops` `power-platform` `automation` `observability` `cost` `networking` `lecture` `hands-on`
- 세부 태그 → 상위 태그로 합침 (예: `tcp`·`dns`·`router` → `networking`, `entra-id`·`sso` → `identity`, `azure-openai`·`document-intelligence` → `foundry`). 어디에도 안 맞는 태그(`samsung`, `techcrunch`, `microsoft` 등)는 뺐다.
- 태그 페이지(/tags/이름/)와 홈 2쪽 이후는 `noindex, follow` (`_includes/metadata-hook.html`), 사이트맵은 직접 만든 `sitemap.xml`(글·카테고리·고정 페이지만).
- 결과: 고유 태그 222 → 23, 사이트맵 325 → 약 90.
