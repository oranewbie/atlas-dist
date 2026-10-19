# atlas-agent 기능·API 레퍼런스

> 필요한 항목을 찾아보는 문서예요. 기준 버전 v0.1.4.

## 1. 내장 도구

| 도구 | 하는 일 | 알아 둘 점 |
|---|---|---|
| **Bash** | 명령 실행 | 작업 위치(cwd)가 다음 명령까지 유지돼요. 기본 제한 120초(최대 600초). Windows 에선 cmd 문법 |
| **Read** · **Write** · **Edit** | 파일 읽기 · 쓰기 · 부분 고치기 | 이미 있는 파일은 **먼저 읽어야** 고칠 수 있어요. 읽은 뒤 밖에서 바뀌었으면 고치기 전에 알려 줘요. 줄바꿈(CRLF)을 보존해요 |
| **Glob** · **Grep** | 파일 이름 찾기 · 내용 찾기 | 결과가 잘리면 잘렸다고 알려 줘요 |
| **WebFetch** | 웹 페이지 읽기 | `prompt` 를 주면 필요한 내용만 뽑아 줘요. 5MB 까지. 다른 사이트로 넘기는 링크는 따라가지 않고 알려 줘요 |
| **WebSearch** | 웹 검색 | `web.search.provider` 를 설정해야 생겨요 ([설정](configuration.md#7-웹-검색-연결)) |
| **DuckDBQuery** | 데이터 파일을 SQL 로 분석 | 작업 폴더의 csv·json·parquet·xlsx 를 복사 없이 바로 조회. 만든 테이블은 그 대화 동안 유지. 결과는 500행까지 |
| **TodoWrite** | 할 일 목록 관리 | 긴 작업의 진행 상황 표시 |
| **Agent** · **AgentOutput** | 서브에이전트에게 맡기기 · 백그라운드 결과 받기 | |
| **Skill** | 스킬 실행 | |
| **ExitPlanMode** | 계획(읽기 전용) 모드 끝내기 | 계획을 보여 주고 승인받은 뒤 실행으로 넘어가요 |
| **load_tools** | 숨은 도구 꺼내기 | 작은 모델은 문맥이 좁아 기본 도구만 보여요. 필요하면 이걸로 나머지를 켜요 |

MCP 서버의 도구는 `mcp__<서버>__<도구>` 이름으로 붙고, 내장 도구와 같은 권한 검사를 받아요.

### DuckDBQuery 예

```sql
SELECT region, SUM(amount) AS total
FROM read_xlsx('data/sales_2026.xlsx')
GROUP BY region ORDER BY total DESC
```

`read_csv`, `read_json_auto`, `read_parquet`, `read_xlsx` 로 파일을 그 자리에서 읽어요.
`COPY ... TO 'result.parquet'` 처럼 내보내면 대화 데이터 폴더(`chats/<대화>/data/`)에 저장돼요.

## 2. 스킬

반복 절차를 `SKILL.md` 하나로 굳혀 두는 기능이에요. Anthropic Agent Skills 형식을 그대로 쓸 수 있어요.

```
<작업 폴더>/.atlas/skills/weekly-report/SKILL.md
```

```markdown
---
description: 한 주의 작업 메모를 모아 주간 보고서 초안을 만든다
argument-hint: <주차 예: 2026-W42>
user-invocable: true
---
1. notes/ 폴더에서 $ARGUMENTS 주차의 메모를 모두 읽는다.
2. 성과 / 다음 주 계획 / 리스크 세 절로 정리한다.
3. reports/$ARGUMENTS.md 로 저장한다.
```

| 위치 | 범위 |
|---|---|
| `<작업 폴더>/.atlas/skills/<이름>/SKILL.md` | 프로젝트 — 팀 공유 |
| `<작업 폴더>/.atlas/private/agent/users/<사용자>/skills/<이름>/SKILL.md` | 나만 |
| `~/.atlas/agent/skills/<이름>/SKILL.md` | 작업 폴더가 없을 때 |

- **`description` 은 꼭 쓰세요.** 에이전트가 "이 스킬을 쓸 때인지" 판단하는 근거예요.
- `/weekly-report 2026-W42` 처럼 부르면 `$ARGUMENTS` 에 인자가 들어가요.
- 주요 머리말: `description` · `when-to-use` · `argument-hint` · `user-invocable`(사람이 `/` 로 부를 수 있게) ·
  `disable-model-invocation`(에이전트가 알아서 쓰지 않게) · `context: fork`(별도 세션에서 실행) · `background`
- 스킬 안에 하위 스킬을 한 겹 둘 수 있어요(`부모:자식`).
- 아직 무시되는 머리말: `allowed-tools`, `model`, `hooks`.

## 3. 서브에이전트

큰 일을 쪼개 다른 에이전트에게 맡기고, **최종 보고만** 돌려받아요. 메인 대화가 세부 과정으로 어지러워지지 않아요.

| 내장 종류 | 역할 |
|---|---|
| `general-purpose` | 범용 |
| `explore` | 조사·탐색 (읽기 위주) |
| `plan` | 계획 세우기 |
| `bash` | 명령 실행 위주 작업 |
| `verify` | 결과 검증 — `PASS` / `FAIL` / `PARTIAL` 로 판정 |

직접 만들려면 `.atlas/agents/<이름>.md`(팀), `.atlas/private/agent/agents/<이름>.md`(나만),
`~/.atlas/agent/agents/<이름>.md`(전역)에 정의를 두세요. 서브에이전트마다 다른 모델을 쓸 수 있어요.

## 4. 지시·기억 파일

| 파일 | 뜻 |
|---|---|
| `~/.atlas/AGENT.md` | 모든 프로젝트에서 지킬 규칙 |
| `<작업 폴더>/.atlas/AGENT.md` | 이 프로젝트의 규칙 (항상 시스템 프롬프트에 실려요) |
| `<작업 폴더>/<하위 폴더>/.atlas/AGENT.md` | 그 폴더 파일을 처음 다룰 때 함께 읽혀요 |
| `<작업 폴더>/.atlas/memory/*.md` | 대화 중 알게 된 사실(팀 공유). 다음 세션엔 목록만 보이고 필요할 때 읽어요 |

AGENT.md 안에서 `@경로` 로 다른 파일을 끌어올 수 있어요.

## 5. MCP 연결

외부 도구 서버를 붙여요. 형식은 Claude Code 의 `.mcp.json` 과 같아요.

```json
{
  "mcpServers": {
    "atlas-engine": { "command": "atlas-engine-mcp", "args": ["--addr", "127.0.0.1:50051"] }
  }
}
```

| 위치 | 범위 |
|---|---|
| `~/.atlas/mcp.json` | 이 PC 전체 |
| `<작업 폴더>/.atlas/mcp.json` | 이 프로젝트 (같은 이름이면 이쪽이 이겨요) |

- 연결 방식: stdio · sse · streamable-http
- 바꾼 내용은 **다음 세션부터** 적용돼요. 접속에 실패한 서버는 경고만 남기고 건너뛰어요.

## 6. 플러그인

스킬·서브에이전트·훅·MCP 설정을 한 폴더로 묶어 나눠 쓸 수 있어요.

```
<작업 폴더>/.atlas/plugins/<이름>/   (또는 ~/.atlas/plugins/<이름>/)
├─ plugin.json
├─ skills/  agents/
├─ hooks.json
└─ mcp.json
```

## 7. 잡과 트리거 — 사람 없이 돌리기

| 기능 | 뜻 |
|---|---|
| **잡** (`RunSkill`) | 스킬을 바로 실행하고 결과를 기록. 승인할 사람이 없으니 **`allowed_tools` 에 허락한 도구만** 써요 (나머지는 즉시 거부되고 결과에 표시) |
| **트리거** (`CreateTrigger`) | 주기(초) · cron(5필드, 로컬 시간) · 특정 시각에 스킬을 자동 실행. 이전 실행이 아직 돌면 건너뛰기, 최대 실행 횟수 지정 |
| **외부 이벤트** (`FireTrigger`) | 웹훅·파일 감시 같은 바깥 신호로 트리거를 당겨요. 인자는 `$ARGUMENTS` 로 들어가요 |

잡 기록은 서버를 다시 켜도 남아요(최근 500개).

## 8. gRPC API

서비스 `atlas.agent.v1.AgentService`, 기본 주소 `127.0.0.1:50061`.

| 묶음 | RPC |
|---|---|
| 세션 | `CreateSession` → `session_id` · `Converse`(양방향 스트림) · `Cancel` · `GetHistory` · `Shutdown` |
| 잡 | `RunSkill` · `AttachJob`(지난 이벤트부터 다시 보기) · `ListJobs` · `CancelJob` |
| 트리거 | `CreateTrigger` · `ListTriggers` · `SetTriggerEnabled` · `DeleteTrigger` · `FireTrigger` |

### 대화(Converse)의 흐름

```
클라이언트 → user_turn("이 폴더 CSV 요약해 줘")
서버       → MESSAGE_START, CONTENT_BLOCK_DELTA(글자), TOOL_USE, TOOL_RESULT ...
서버       → PERMISSION_REQUEST   ← 위험한 도구면 승인 요청
클라이언트 → permission(허용/거부, 기억 범위)
서버       → ... MESSAGE_STOP ... TERMINAL("end_turn")   ← 턴은 반드시 TERMINAL 로 끝나요
```

- `MESSAGE_STOP` 은 AI 호출 한 번의 끝이라 한 턴에 여러 번 와요. **턴의 끝은 `TERMINAL`** 이에요.
- 클라이언트가 보낼 수 있는 것: `user_turn`(이미지 첨부 가능) · `permission` · `tool_result` · `register_tools`(내 앱 기능을 도구로 등록) · `set_permissions` · `register_hooks` · `hook_response`
- 서버 이벤트: `MESSAGE_START` · `CONTENT_BLOCK_DELTA` · `TOOL_USE` · `TOOL_RESULT` · `LLM_TURN_DONE` · `MESSAGE_STOP` · `TERMINAL` · `AGENT_PROGRESS` · `TOOL_PROGRESS` · `PERMISSION_REQUEST` · `TOOL_INVOKE` · `HOOK_REQUEST`

## 9. SDK

| SDK | 핵심 사용법 |
|---|---|
| **Java** (JDK 17+) | `AtlasAgentClient` → `AgentConversation.sendUserTurn(...)`, `respondPermission(...)`, `registerTools(...)` |
| **Python** (`atlas-agent-client`, asyncio) | `await conv.run_turn(text, on_text=..., can_use_tool=..., tools=...)` |
| **TypeScript** (`@atlas/agent-client`) | `await conv.runTurn(text, { onText, canUseTool, tools })` |

- Python·TypeScript 는 `can_use_tool`/`canUseTool` 을 주지 않으면 **승인 요청을 모두 거부**해요. 안전한 기본값이에요.
- SDK 는 소스 저장소와 함께 2026년 12월 공개될 예정이에요.

## 10. 파일 위치

| 무엇 | 위치 |
|---|---|
| 대화 기록 | `<작업 폴더>/.atlas/private/agent/chats/<대화>/<세션>.jsonl` |
| 서브에이전트·도구 결과·데이터 | 같은 `chats/<대화>/` 아래 `subagents/` · `tool-results/` · `data/` · `duckdb/` |
| 잡 · 트리거 | `.atlas/private/agent/jobs/` · `triggers/` |
| 토큰 사용량 | `.atlas/private/agent/usage/<연-월>.jsonl` |
| LLM 호출 상세 (켰을 때) | `.atlas/private/agent/llm-call-tracker/` |
| 팀 공유 | `.atlas/memory/` · `.atlas/skills/` · `.atlas/agents/` · `.atlas/permission.json` · `.atlas/hooks.json` · `.atlas/mcp.json` · `.atlas/plugins/` |
| 작업 폴더 없이 쓸 때 | `~/.atlas/agent/` |
| 서버 로그 | 콘솔 출력. Atlas Studio 가 띄운 경우 `~/.atlas/agent-server.log` |

> 대화 기록에는 에이전트가 읽은 파일 내용과 명령 결과가 그대로 들어가요. `private/` 폴더는 git 에 올리지 마세요
> (Atlas Studio 는 자동으로 제외해 줘요).

## 11. 문제 해결

| 증상 | 확인할 것 |
|---|---|
| 생각한 모델이 아닌 것 같다 | 설정 파일을 못 찾은 거예요. `/config` 로 확인하고 `ATLAS_AGENT_CONFIG`·`ATLAS_PROFILE` 을 점검 |
| 설치 중 "python 버전 불일치" | 번들 이름의 Python 버전(3.12)으로 다시 설치 |
| 턴이 멈춘 것 같다 | 승인 요청에 답하지 않았을 수 있어요. 서버 기본은 600초 뒤 거부 |
| 잡이 계속 실패·거부 | `allowed_tools` 에 필요한 도구가 들어 있는지 |
| 웹 검색 도구가 없다 | `web.search.provider` 가 `none`(기본)이에요. SearXNG 는 `format=json` 도 켜야 해요 |
| 웹 페이지가 403 | `web.user_agent` 조정 |
| 샌드박스에서 명령이 실패 | Docker 데몬·이미지 확인. 폐쇄망은 이미지를 미리 받고 `sandbox.pull: never` |
| 작은 모델에서 Skill·Agent 도구가 안 보인다 | 도구 예산 때문이에요. 에이전트가 `load_tools` 로 켜거나 `tools.bind_budget` 을 올리세요 |
| Windows 에서 글자 깨짐·인코딩 오류 | `PYTHONUTF8=1` 환경 변수 |
| MCP 서버가 안 붙는다 | 새 세션에서 다시 시도. 로그의 경고 확인 |
