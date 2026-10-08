---
title: "MCP 서버를 처음 만들어봤다 — Python·TypeScript로 메모 도구 붙이기"
date: 2026-10-04 23:30:00 +0900
categories: [Dev Notes]
tags: [mcp, python, typescript, vscode, copilot]
description: 메모 폴더를 읽는 MCP 서버를 Python과 TypeScript로 각각 만들어 VS Code + GitHub Copilot 에이전트 모드에서 호출해봤다. 막혔던 에러와 고객사 구축 때 달라지는 점까지 정리했다.
---

MCP 이야기를 몇 달째 들었다. 발표 자료에도, 고객 미팅에도 빠지지 않고 나왔다. 그런데 정작 내가 직접 서버를 만들어 본 적은 없었다. 남이 만든 서버를 붙여 쓰는 것과 내가 만든 함수를 AI가 골라 쓰게 만드는 것은 다른 경험일 것 같았다.

그래서 제일 단순한 것부터 해봤다. 내 PC의 `notes` 폴더에 있는 메모를 읽는 서버다. 도구는 세 개, 메모 목록 보기와 메모 읽기, 키워드 찾기뿐이다. 이걸 Python으로 한 번, TypeScript로 한 번 만들어서 VS Code의 GitHub Copilot 에이전트 모드에 붙였다.

결론부터 말하면 코드는 금방 썼고, 시간은 대부분 에러 로그를 읽는 데 들었다. 그 과정을 그대로 적는다.

## MCP를 한 문장으로

**AI가 내 함수를 골라서 호출하게 해주는 연결 규격**이다.

나는 이렇게 이해했다. Copilot은 머리다. MCP 서버는 손이다. `notes` 폴더는 손이 만지는 물건이다. 그리고 `mcp.json`은 손이 어디 있는지 적어둔 쪽지다. 머리는 쪽지를 보고 손을 찾고, 손에 붙은 설명서를 읽고, 어떤 손을 쓸지 스스로 고른다.

<figure class="sketch">
<svg viewBox="0 0 720 260" role="img" aria-label="전체 구조 — VS Code가 mcp.json을 읽어 MCP 서버를 띄우고, Copilot이 서버의 도구로 notes 폴더를 읽는다">
  <defs>
    <marker id="ar-mm1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ar-ma1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">① 전체 구조 — 머리 · 손 · 물건 · 쪽지</text>

  <rect x="8" y="50" width="250" height="150" rx="10" class="sk-box"/>
  <text x="133" y="74" text-anchor="middle" class="sk-label">VS Code (호스트)</text>
  <rect x="28" y="88" width="210" height="46" rx="8" class="sk-box-accent"/>
  <text x="133" y="110" text-anchor="middle" class="sk-label">Copilot = 머리</text>
  <text x="133" y="126" text-anchor="middle" class="sk-sub">어떤 도구를 쓸지 판단</text>
  <rect x="28" y="146" width="210" height="40" rx="8" class="sk-box-muted"/>
  <text x="133" y="171" text-anchor="middle" class="sk-mono">mcp.json = 쪽지</text>

  <path d="M258,125 L340,125" class="sk-line-accent" marker-end="url(#ar-ma1)"/>
  <text x="299" y="116" text-anchor="middle" class="sk-sub">stdio</text>

  <rect x="346" y="70" width="190" height="110" rx="10" class="sk-box-accent"/>
  <text x="441" y="96" text-anchor="middle" class="sk-label">MCP 서버 = 손</text>
  <text x="441" y="122" text-anchor="middle" class="sk-mono">list_notes</text>
  <text x="441" y="142" text-anchor="middle" class="sk-mono">read_note</text>
  <text x="441" y="162" text-anchor="middle" class="sk-mono">search_notes</text>

  <path d="M536,125 L590,125" class="sk-line" marker-end="url(#ar-mm1)"/>

  <rect x="596" y="95" width="116" height="60" rx="8" class="sk-box"/>
  <text x="654" y="121" text-anchor="middle" class="sk-label">notes/</text>
  <text x="654" y="139" text-anchor="middle" class="sk-sub">= 물건</text>

  <text x="360" y="240" text-anchor="middle" class="sk-sub">서버는 시키는 일만 한다 — 언제 무엇을 시킬지는 머리(LLM)가 정한다</text>
</svg>
<figcaption>도구를 고르는 건 Copilot이고, 서버는 정해진 일을 하는 손일 뿐이다</figcaption>
</figure>

처음에 헷갈렸던 게 파일 두 개의 역할이었다. 정리하면 이렇다.

| 파일 | 역할 |
|---|---|
| mcp.json | 서버가 아니라 사용하는 쪽(VS Code) 설정. 도구함이 어디 있고 어떻게 여는지 |
| memo_server.py | 도구함과 도구가 한 파일에 같이 들어 있음 |

## 준비물

Python 3.10 이상, Node 20 이상, VS Code, 그리고 GitHub Copilot이다. Copilot은 채팅 모드를 **Agent**로 바꿔야 MCP 도구를 쓴다.

## Python 버전

폴더는 이렇게 잡았다. 메모 세 개를 미리 넣어뒀다. 회의 메모, 장보기 목록, 읽을거리다.

```text
C:\dev\memo-mcp
├─ .vscode\mcp.json
├─ notes\
│   ├─ meeting.md
│   ├─ reading.md
│   └─ shopping.md
├─ memo_server.py
└─ ts\            ← 나중에 만든 TypeScript 버전
```

가상환경을 만들고 MCP SDK를 설치했다. 버전을 `<2`로 묶은 이유는 아래 에러 이야기에서 설명한다.

```powershell
cd C:\dev\memo-mcp
uv venv .venv
uv pip install "mcp[cli]<2"
```

서버 코드는 이게 전부다.

```python
from pathlib import Path

from mcp.server.fastmcp import FastMCP

# 1) 준비: 서버 이름과 메모 폴더 위치
mcp = FastMCP("memo")
NOTES_DIR = Path(__file__).parent / "notes"


# 2) 보조 함수: 폴더 밖 파일을 열지 못하게 막는다
def _note_path(name: str) -> Path:
    path = (NOTES_DIR / name).resolve()
    if path.parent != NOTES_DIR.resolve() or path.suffix != ".md":
        raise ValueError(f"notes 폴더 안의 .md 파일만 열 수 있습니다: {name}")
    return path


# 3) 도구 3개: docstring 이 곧 LLM 이 읽는 도구 설명서다
@mcp.tool()
def list_notes() -> list[str]:
    """메모 목록을 보여준다. 어떤 메모가 있는지 모를 때 가장 먼저 쓴다."""
    return sorted(p.name for p in NOTES_DIR.glob("*.md"))


@mcp.tool()
def read_note(name: str) -> str:
    """메모 하나의 전체 내용을 읽는다. name 은 list_notes 가 돌려준 파일 이름(예: meeting.md)."""
    return _note_path(name).read_text(encoding="utf-8")


@mcp.tool()
def search_notes(keyword: str) -> list[str]:
    """모든 메모에서 keyword 가 들어간 줄을 찾아 '파일명: 줄' 형태로 돌려준다."""
    hits = []
    for p in sorted(NOTES_DIR.glob("*.md")):
        for line in p.read_text(encoding="utf-8").splitlines():
            if keyword in line:
                hits.append(f"{p.name}: {line.strip()}")
    return hits


# 4) 실행: VS Code 가 이 파일을 자식 프로세스로 띄우고 stdin/stdout 으로 대화한다
if __name__ == "__main__":
    mcp.run()  # 기본값 transport="stdio"
```

네 덩어리로 읽으면 쉽다.

첫째는 준비다. `FastMCP("memo")`가 도구함을 만든다. 이름은 Copilot 화면에 그대로 보인다.

둘째는 보조 함수다. `read_note`에 `../memo_server.py` 같은 이름이 들어오면 폴더 밖 파일이 열린다. AI가 일부러 그럴 리는 없겠지만, 도구는 결국 입력을 그대로 믿으면 안 된다. 실제로 이 이름을 넣어 호출해보니 에러가 돌아오고 파일은 열리지 않았다.

셋째가 핵심인 `@mcp.tool()`이다. 함수에 이 데코레이터를 붙이면 도구가 된다. 함수 이름이 도구 이름, 타입 힌트가 입력 형식, 그리고 docstring이 도구 설명이 된다. 이 설명이 얼마나 중요한지는 뒤에서 다시 쓴다.

넷째는 실행이다. `mcp.run()`은 기본으로 stdio, 그러니까 표준 입출력으로 통신한다. 서버가 포트를 여는 게 아니라 VS Code가 이 파일을 실행하고 파이프로 말을 주고받는다.

### mcp.json으로 VS Code에 알려주기

`.vscode\mcp.json`에 서버를 등록한다. 이게 아까 말한 쪽지다.

```json
{
  "servers": {
    "memo": {
      "type": "stdio",
      "command": "${workspaceFolder}\\.venv\\Scripts\\python.exe",
      "args": ["${workspaceFolder}\\memo_server.py"]
    }
  }
}
```

파일을 열면 `"memo"` 위에 **Start** 버튼이 생긴다. 누르고 잠시 뒤 **Running · 3 tools**가 뜨면 연결된 것이다.

### Copilot에게 시켜보기

Copilot Chat을 Agent 모드로 바꾸고 이렇게 물었다.

> 메모 목록 보여주고, 회의 메모에서 다음 할 일 찾아줘

Copilot은 먼저 `list_notes`를 불러 메모 목록을 확인했다. 그다음 회의 메모를 골라 `read_note`를 불렀고, 메모에 적힌 다음 할 일을 정리해 답했다. 나는 어떤 도구를 어떤 순서로 쓰라고 말한 적이 없다.

<figure class="shot">
  <div class="shot-icon">🖼</div>
  <div class="shot-what">Copilot Chat(Agent 모드)에서 list_notes → read_note 가 차례로 호출되는 화면</div>
  <figcaption>계정·이메일·세션 목록은 가리고 넣을 것</figcaption>
</figure>

## 내가 겪은 에러: FastMCP를 찾을 수 없다

처음에는 버전을 고정하지 않고 `uv pip install "mcp[cli]"`로 설치했다. 그러자 최신 2.x가 깔렸다. Start를 누르자 서버가 바로 죽었다.

VS Code에서 원인을 찾는 방법은 이랬다. 서버 이름 옆에 뜬 **Error**를 클릭하면 **Output** 패널이 열리고, 서버가 남긴 로그가 그대로 보인다. 거기에 이런 줄이 있었다.

```text
ModuleNotFoundError: No module named 'mcp.server.fastmcp'.
This is mcp 2.x, where FastMCP was renamed to MCPServer
(from mcp.server.mcpserver import MCPServer) and other APIs changed;
see the migration guide ... or pin 'mcp<2' to keep running v1 code.
```

2.x에서 `FastMCP`가 `MCPServer`로 이름이 바뀌었다는 뜻이다. 인터넷에 있는 예제와 튜토리얼은 대부분 1.x 기준이라 `from mcp.server.fastmcp import FastMCP`로 시작한다. 예제를 그대로 따라 하면 여기서 막힌다.

해결은 두 가지다. 2.x 마이그레이션 가이드를 따라 코드를 고치거나, 버전을 1.x로 묶는 것이다. 처음 배우는 단계라 예제와 맞추는 게 우선이라고 보고 `mcp<2`로 고정했다. 다시 설치하니 1.x가 깔렸고 서버가 떴다.

에러 메시지가 해결책까지 알려주는데도, 로그를 열어보기 전까지는 그냥 "서버가 안 뜬다"로만 보였다. Output 패널을 여는 습관이 이번 실습에서 얻은 제일 실용적인 교훈이었다.

## TypeScript 버전

같은 서버를 TypeScript로도 만들었다. `ts` 폴더에서 패키지를 설치했다.

```powershell
cd C:\dev\memo-mcp\ts
npm init -y
npm install @modelcontextprotocol/sdk zod tsx
```

`index.ts`는 Python과 모양이 거의 같다. 다른 점은 데코레이터 대신 `registerTool`로 등록하고, 입력 형식을 타입 힌트 대신 zod로 적는다는 것이다.

```typescript
import { readdir, readFile } from "node:fs/promises";
import path from "node:path";
import { fileURLToPath } from "node:url";
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// 1) 준비: 서버 이름과 메모 폴더 위치 (Python 버전과 같은 notes 폴더를 쓴다)
const server = new McpServer({ name: "memo-ts", version: "1.0.0" });
const NOTES_DIR = path.resolve(path.dirname(fileURLToPath(import.meta.url)), "..", "notes");

// 2) 보조 함수: 폴더 밖 파일을 열지 못하게 막는다
function notePath(name: string): string {
  const p = path.resolve(NOTES_DIR, name);
  if (path.dirname(p) !== NOTES_DIR || path.extname(p) !== ".md") {
    throw new Error(`notes 폴더 안의 .md 파일만 열 수 있습니다: ${name}`);
  }
  return p;
}

async function mdFiles(): Promise<string[]> {
  return (await readdir(NOTES_DIR)).filter((f) => f.endsWith(".md")).sort();
}

const text = (t: string) => ({ content: [{ type: "text" as const, text: t }] });

// 3) 도구 3개: description 이 Python 의 docstring 역할
server.registerTool(
  "list_notes",
  { description: "메모 목록을 보여준다. 어떤 메모가 있는지 모를 때 가장 먼저 쓴다." },
  async () => text((await mdFiles()).join("\n")),
);

server.registerTool(
  "read_note",
  {
    description: "메모 하나의 전체 내용을 읽는다. name 은 list_notes 가 돌려준 파일 이름(예: meeting.md).",
    inputSchema: { name: z.string() },
  },
  async ({ name }) => text(await readFile(notePath(name), "utf-8")),
);

server.registerTool(
  "search_notes",
  {
    description: "모든 메모에서 keyword 가 들어간 줄을 찾아 '파일명: 줄' 형태로 돌려준다.",
    inputSchema: { keyword: z.string() },
  },
  async ({ keyword }) => {
    const hits: string[] = [];
    for (const f of await mdFiles()) {
      for (const line of (await readFile(path.join(NOTES_DIR, f), "utf-8")).split("\n")) {
        if (line.includes(keyword)) hits.push(`${f}: ${line.trim()}`);
      }
    }
    return text(hits.join("\n"));
  },
);

// 4) 실행: stdio 로 연결 (stdout 은 프로토콜 전용이라 로그는 console.error 로)
await server.connect(new StdioServerTransport());
console.error("memo-ts 서버 시작");
```

나란히 놓으면 대응 관계가 바로 보인다.

| Python | TypeScript |
|---|---|
| `FastMCP("memo")` | `new McpServer({ name: "memo-ts", ... })` |
| `@mcp.tool()` + 함수 | `server.registerTool("이름", { ... }, 함수)` |
| docstring | `description` |
| 타입 힌트 `name: str` | `inputSchema: { name: z.string() }` |
| `mcp.run()` | `server.connect(new StdioServerTransport())` |

`mcp.json`에는 서버를 하나 더 적었다.

```json
"memo-ts": {
  "type": "stdio",
  "command": "npx",
  "args": ["tsx", "${workspaceFolder}\\ts\\index.ts"]
}
```

TypeScript에서도 두 번 막혔다. 둘 다 실제로 실행해서 확인한 에러다.

첫 번째는 `package.json`에 `"type": "module"`이 없을 때였다. 마지막 줄의 `await`(최상위 await) 때문에 tsx가 이렇게 멈췄다.

```text
ERROR: Top-level await is currently not supported with the "cjs" output format
```

`npm pkg set type=module`로 한 줄 추가하니 해결됐다.

두 번째는 디버깅하려고 `console.log`를 넣었을 때였다. stdio 방식에서 표준 출력은 MCP 메시지가 오가는 통로다. 거기에 로그를 섞으니 클라이언트 쪽에 `Failed to parse JSONRPC message from server` 경고가 찍혔다. 로그는 반드시 `console.error`로 표준 에러에 써야 한다.

이 둘을 고치자 `memo-ts`도 도구 세 개가 잡혔다. 같은 `notes` 폴더를 보기 때문에 호출 결과도 Python 버전과 똑같았다.

## 도구는 어떤 순서로 불리나

<figure class="sketch">
<svg viewBox="0 0 720 220" role="img" aria-label="도구 호출 흐름 — 질문, Copilot의 도구 선택, 서버 실행, 결과 반환">
  <defs>
    <marker id="ar-mm2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ar-ma2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">② 도구 호출 흐름</text>

  <rect x="8" y="60" width="150" height="70" rx="8" class="sk-box"/>
  <text x="83" y="90" text-anchor="middle" class="sk-label">① 질문</text>
  <text x="83" y="110" text-anchor="middle" class="sk-sub">"다음 할 일 찾아줘"</text>

  <path d="M158,95 L190,95" class="sk-line" marker-end="url(#ar-mm2)"/>

  <rect x="196" y="60" width="160" height="70" rx="8" class="sk-box-accent"/>
  <text x="276" y="90" text-anchor="middle" class="sk-label">② Copilot 판단</text>
  <text x="276" y="110" text-anchor="middle" class="sk-sub">설명서 보고 도구 선택</text>

  <path d="M356,95 L388,95" class="sk-line-accent" marker-end="url(#ar-ma2)"/>

  <rect x="394" y="60" width="150" height="70" rx="8" class="sk-box"/>
  <text x="469" y="90" text-anchor="middle" class="sk-label">③ 서버 실행</text>
  <text x="469" y="110" text-anchor="middle" class="sk-mono">read_note()</text>

  <path d="M544,95 L576,95" class="sk-line" marker-end="url(#ar-mm2)"/>

  <rect x="582" y="60" width="130" height="70" rx="8" class="sk-box"/>
  <text x="647" y="90" text-anchor="middle" class="sk-label">④ 결과 반환</text>
  <text x="647" y="110" text-anchor="middle" class="sk-sub">메모 내용</text>

  <path d="M647,130 C647,180 276,180 276,134" class="sk-line" stroke-dasharray="4 4" marker-end="url(#ar-mm2)"/>
  <text x="461" y="196" text-anchor="middle" class="sk-sub">부족하면 ②로 돌아가 다른 도구를 또 부른다</text>
</svg>
<figcaption>순서는 코드에 없다 — 결과를 보고 다음 도구를 고르는 것도 Copilot이다</figcaption>
</figure>

## 내가 처음에 오해했던 것

MCP 서버를 일종의 지침서라고 생각했다. "이런 질문이 오면 이 함수를 써라"를 서버가 정해두는 줄 알았다. 실제로는 반대였다. 서버는 **설명서가 붙은 도구함**일 뿐이다. 언제 어떤 도구를 꺼낼지는 LLM이 설명서를 읽고 판단한다.

LLM이 서버와 직접 통신한다는 것도 오해였다. 서버를 실행하고 도구를 호출하는 건 VS Code 같은 **호스트**다. LLM은 "이 도구를 이 인자로 불러줘"라고 말할 뿐이고, 실제 호출은 호스트가 한다. Claude Desktop이나 Copilot Studio도 같은 자리에 있다. 그래서 같은 서버를 어떤 LLM에도 붙일 수 있다. 내가 만든 `memo_server.py`는 Copilot 전용이 아니다.

그러다 보니 docstring이 코드만큼 중요하다는 것도 알게 됐다. LLM이 보는 건 함수 본문이 아니라 이름과 설명뿐이다. `list_notes`의 설명에 "어떤 메모가 있는지 모를 때 가장 먼저 쓴다"는 문장을 넣은 이유가 그거다. 설명이 모호하면 도구가 있어도 안 쓰거나 엉뚱한 때 쓴다. 도구 설명은 사람이 읽는 주석이 아니라 AI에게 주는 사용 설명서다.

## 로컬 실습과 고객사 구축은 무엇이 다른가

이번 실습은 전부 내 노트북 안에서 끝났다. 고객사에 같은 걸 만든다면 거의 모든 칸이 바뀐다.

| | 로컬 실습 | 고객사 구축 |
|---|---|---|
| 통신 | stdio (내 PC 안) | Streamable HTTP (원격 URL) |
| 위치 | 내 노트북 | Azure Container Apps 등 |
| 인증 | 없음 | API Key 또는 OAuth 2.0 (Entra ID) |
| 데이터 | notes 폴더 | 사내 DB, SharePoint, ERP API |

Copilot Studio에 붙인다면 통신 방식은 선택지가 없다. MS Learn 문서에 따르면 Copilot Studio는 현재 Streamable 방식만 지원하고, SSE는 2025년 8월 이후 지원하지 않는다. 인증은 없음, API Key, OAuth 2.0 중에서 고른다. 그리고 Copilot Studio의 MCP 연결은 Power Platform 커넥터를 통하기 때문에, 커넥터에 걸린 데이터 정책(DLP)이 MCP 서버에도 그대로 적용된다. 고객사에서는 이 부분을 관리자와 먼저 맞춰야 할 것 같다. ([MS Learn: Connect your agent to an existing MCP server](https://learn.microsoft.com/microsoft-copilot-studio/mcp-add-existing-server-to-agent))

서버를 어디에 올릴지도 정해야 한다. 후보로 본 게 Azure Container Apps다. Kubernetes 기반의 서버리스 컨테이너 플랫폼이고, 요청이 없을 때 0개까지 줄어드는 scale to zero를 지원한다. 호출이 드문 사내 도구 서버에는 잘 맞아 보인다. ([MS Learn: Comparing Container Apps with other Azure container options](https://learn.microsoft.com/azure/container-apps/compare-options))

VM과 비교하면 차이가 확실하다.

| | 비유 | 직접 관리하는 것 |
|---|---|---|
| VM | 빈 사무실 통째로 임대 | OS, 패치, 보안, 런타임, 상시 가동 |
| Container Apps | 짐만 들고 가는 공유오피스 | 컨테이너(코드 상자)만 |

<figure class="sketch">
<svg viewBox="0 0 720 270" role="img" aria-label="로컬 stdio 구조와 원격 HTTP 구조 비교">
  <defs>
    <marker id="ar-mm3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-muted"/>
    </marker>
    <marker id="ar-ma3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="sk-fill-accent"/>
    </marker>
  </defs>
  <text x="0" y="14" class="sk-title">③ 로컬(stdio) vs 원격(HTTP)</text>

  <rect x="8" y="34" width="704" height="92" rx="10" class="sk-box-muted"/>
  <text x="24" y="56" class="sk-sub">내 노트북 한 대</text>
  <rect x="40" y="66" width="200" height="46" rx="8" class="sk-box"/>
  <text x="140" y="94" text-anchor="middle" class="sk-label">VS Code</text>
  <path d="M240,89 L330,89" class="sk-line" marker-end="url(#ar-mm3)"/>
  <text x="285" y="81" text-anchor="middle" class="sk-mono">stdio</text>
  <rect x="336" y="66" width="200" height="46" rx="8" class="sk-box"/>
  <text x="436" y="94" text-anchor="middle" class="sk-label">memo_server.py</text>
  <text x="620" y="94" text-anchor="middle" class="sk-sub">인증 없음</text>

  <rect x="40" y="166" width="200" height="56" rx="8" class="sk-box"/>
  <text x="140" y="190" text-anchor="middle" class="sk-label">Copilot Studio</text>
  <text x="140" y="208" text-anchor="middle" class="sk-sub">사내 에이전트</text>
  <path d="M240,194 L330,194" class="sk-line-accent" marker-end="url(#ar-ma3)"/>
  <text x="285" y="186" text-anchor="middle" class="sk-mono">HTTPS</text>
  <text x="285" y="214" text-anchor="middle" class="sk-sub">OAuth / Key</text>
  <rect x="336" y="160" width="200" height="68" rx="10" class="sk-box-accent"/>
  <text x="436" y="188" text-anchor="middle" class="sk-label">MCP 서버</text>
  <text x="436" y="208" text-anchor="middle" class="sk-sub">Container Apps</text>
  <path d="M536,194 L592,194" class="sk-line" marker-end="url(#ar-mm3)"/>
  <rect x="598" y="166" width="114" height="56" rx="8" class="sk-box"/>
  <text x="655" y="190" text-anchor="middle" class="sk-label">사내 데이터</text>
  <text x="655" y="208" text-anchor="middle" class="sk-sub">DB · SharePoint</text>

  <text x="360" y="258" text-anchor="middle" class="sk-sub">도구 코드는 거의 그대로 — 바뀌는 건 통신 · 위치 · 인증 · 데이터</text>
</svg>
<figcaption>로컬에서 만든 도구는 그대로 두고, 바깥 껍데기만 갈아 끼우면 된다</figcaption>
</figure>

## 다음에 해볼 것

이번 서버는 읽기만 한다. 다음에는 `write_note` 도구를 추가해서 Copilot이 메모를 직접 쓰게 해볼 생각이다. 읽기와 달리 쓰기는 사고가 날 수 있는 도구라서, 호스트가 실행 전에 확인을 받는지 보는 게 관심사다.

그다음은 같은 서버를 stdio가 아니라 HTTP 방식으로 바꿔 띄워보는 것이다. 표에 적은 고객사 구성으로 가려면 결국 이 단계를 거쳐야 한다.

MCP가 기존 API 호출과 무엇이 다른지는 앞서 [MCP와 API는 무엇이 다른가](/posts/mcp-vs-api/)에 정리해뒀다.

## 참고

- [MS Learn — Connect your agent to an existing Model Context Protocol (MCP) server](https://learn.microsoft.com/microsoft-copilot-studio/mcp-add-existing-server-to-agent)
- [MS Learn — Comparing Container Apps with other Azure container options](https://learn.microsoft.com/azure/container-apps/compare-options)
- [MCP Python SDK v2 마이그레이션 가이드 — FastMCP renamed to MCPServer](https://py.sdk.modelcontextprotocol.io/v2/migration/#fastmcp-renamed-to-mcpserver)
