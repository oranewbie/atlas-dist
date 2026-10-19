# atlas-engine API·설정 레퍼런스

> 필요한 항목을 찾아보는 문서예요. 처음이라면 [소개](README.md) → [그래프 만들기 가이드](graph-guide.md) 순서를 권해요.
> 기준 버전: v0.1.3

## 설정 파일

위치는 `conf/<프로필>/atlas-engine.toml` 이고, 기본 프로필은 `default` 예요.
고른 프로필에 파일이 없으면 `default` 의 파일을 써요.

값을 정하는 우선순위는 **기본값 < 설정 파일 < 환경 변수** 예요(뒤로 갈수록 이겨요).

| 키 | 기본값 | 환경 변수 | 뜻 |
|---|---|---|---|
| `listen_addr` | `127.0.0.1:50051` | `LISTEN_ADDR` | gRPC 주소. 다른 PC 에서 접속하려면 `0.0.0.0:50051` |
| `mcp_listen_addr` | `127.0.0.1:50151` | `MCP_LISTEN_ADDR` | MCP 주소. 빈 문자열이면 MCP 를 꺼요 |
| `python` | `python` | `PYTHON` | Python 노드 등록 시 `python` 을 생략하면 쓰는 인터프리터 |

> 주소를 바꿀 때는 **설정 파일이나 환경 변수**를 쓰세요. 시작 스크립트에 `--addr` 옵션을 넘겨도 적용되지 않아요.

### 설정 파일 찾기

| 대상 | 실행 옵션 | 환경 변수 | 기본 |
|---|---|---|---|
| 루트 폴더 | — | `ATLAS_ROOT` | `.atlas-root` 표식 파일이 있는 상위 폴더(번들 루트) |
| 설정 폴더 | `--conf-dir` | `ATLAS_CONF_DIR` | `<루트>/conf` |
| 프로필 | `--profile` | `ATLAS_PROFILE` | `default` |

### 그 밖의 환경 변수

| 환경 변수 | 기본 | 뜻 |
|---|---|---|
| `ATLAS_REGISTRY_DIR` | `~/.atlas/registry` | 등록한 런타임 저장 위치 |
| `ATLAS_RUNS_DIR` | `~/.atlas/runs` | 실행 상태·기록 저장 위치 |
| `DEFAULT_AGENT_SCRIPT` · `DEFAULT_AGENT_PYTHON` | — | 둘 다 주면, 등록 안 된 `node_type` 을 이 에이전트 스크립트로 실행해요 |
| `RUST_LOG` | `info` | 로그 상세도 (`debug`, `warn` …) |

`~` 는 Windows 에서 `%USERPROFILE%`, Linux 에서 `$HOME` 이에요.

## 파일 위치

| 무엇 | 위치 |
|---|---|
| 등록한 런타임 | `~/.atlas/registry/<namespace>/<node_type>.json` |
| 취소한 실행 (다시 불러오기용) | `~/.atlas/runs/<user>/<namespace>/<network>/<instance>/` — `graph.json`·`checkpoint.json`·`meta.json`·`events.pb` |
| 끝난 실행 기록 | 같은 위치의 `meta.json`·`events.pb` (보존 기간 정책은 아직 없어요) |
| 로그 | 엔진은 콘솔(stdout)에 써요. Atlas Studio 로 띄운 경우 `~/.atlas/engine.log` |

## gRPC API

서비스 이름은 `runtime.RuntimeService`, 기본 주소는 `127.0.0.1:50051` 이에요. 그래프는 메시지가 아니라
**JSON 문자열**(`graph_json`)로 주고받아요.

| RPC | 하는 일 |
|---|---|
| `RegisterRuntime` | 노드 종류 등록 (`PYTHON_WORKER` / `BUILTIN`). 같은 이름이면 교체 |
| `ListRuntimes` | 등록 목록 |
| `UnregisterRuntime` | 등록 해제 |
| `SearchNodes` | 이름·설명·태그로 노드 검색 (빈 검색어 = 전체) |
| `SubmitGraph` | 그래프 제출 → `run_id` |
| `StreamRun` | 실행 이벤트 스트림. 이미 지난 이벤트부터 다시 보내 주고 실시간으로 이어져요 |
| `GetState` | 상태·현재 박자·노드별 상태·오류 |
| `Pause` / `ResumeRun` | 일시정지 / 이어 하기 |
| `Cancel` | 취소 (상태를 디스크에 저장) |
| `LoadRun` | 저장된 실행을 불러와 새 `run_id` 로 이어 실행 |
| `SetBreakpoints` | 노드 중단점 설정 |
| `Shutdown` | 엔진 종료 |

**이벤트 종류**: `VertexStarted`, `VertexCompleted`, `VertexFailed`, `SuperstepCompleted`, `NodeMessage`,
`RunCompleted`. 이벤트마다 순서 번호(`seq`)가 붙어요.

정의 파일(`runtime.proto`)로 원하는 언어의 클라이언트를 만들 수 있어요.

## MCP — AI 에이전트용 창구

엔진은 두 가지 방식으로 MCP 를 제공해요. 둘 다 같은 엔진을 보기 때문에 MCP 로 제출한 실행을 gRPC 로
지켜볼 수도 있어요.

| 방식 | 주소 / 실행 | 언제 |
|---|---|---|
| **HTTP** | 엔진이 `http://127.0.0.1:50151/mcp` 를 함께 열어요 | 엔진이 늘 떠 있을 때 |
| **stdio 브리지** | `bin/atlas-engine-mcp` | 클라이언트가 MCP 서버를 직접 띄우는 경우 (Claude Desktop 등). 엔진이 늦게 떠도 괜찮아요 |

### 연결 예

```sh
# Claude Code — HTTP
claude mcp add --transport http atlas-engine http://127.0.0.1:50151/mcp
```

```jsonc
// Claude Desktop — claude_desktop_config.json (stdio)
{
  "mcpServers": {
    "atlas-engine": { "command": "atlas-engine-mcp", "args": ["--addr", "127.0.0.1:50051"] }
  }
}
```

`atlas-engine-mcp` 옵션: `--addr <gRPC 주소>`(없으면 `ATLAS_ENGINE_ADDR` → 설정 파일 → `127.0.0.1:50051`),
`--transport stdio|http`, `--http-addr`.

### MCP 도구 목록

| 도구 | 하는 일 |
|---|---|
| `guide` | 엔진 사용 설명서. 주제: `overview`, `graph_json`, `link_types`, `runtimes`, `python_worker`, `running`, `examples`, `troubleshooting`, `all` |
| `search_nodes` · `list_runtimes` | 쓸 수 있는 노드 찾기 |
| `register_runtime` · `unregister_runtime` | 노드 등록·해제 |
| `submit_graph` | 제출만 |
| `run_graph` | 제출하고 끝날 때까지 기다림 (기본 300초, 넘기면 `timed_out: true` — 실행은 계속돼요) |
| `get_run_state` · `get_run_events` | 상태·이벤트 조회 (`since_seq` 로 이어 받기) |
| `pause_run` · `resume_run` · `cancel_run` · `load_run` · `set_breakpoints` | 실행 제어 |

엔진 종료(`Shutdown`)는 일부러 MCP 에 넣지 않았어요.

## SDK

| SDK | 위치(번들 아님, 소스 공개 시 제공) | 할 수 있는 일 |
|---|---|---|
| **Java** (JDK 17+) | `sdk/java` — `AtlasEngineClient` | 제출·스트림·대기·취소·일시정지·재개·불러오기·중단점·등록·검색·종료 전부, 그래프 빌더(`addNode().connect()`) |
| **Python** | `sdk/python/runtime_client.py` | 제출·상태·취소·대기 (최소 기능) |
| 그 밖의 언어 | `runtime.proto` | gRPC 코드 생성 |

> 소스 저장소는 2026년 12월 공개 예정이에요. 그 전에는 gRPC 정의와 SDK 가 필요하면 문의해 주세요.

## Python 노드 통신 규약 (참고)

엔진과 Python 노드는 표준 입출력으로 `[4바이트 길이][UTF-8 JSON]` 묶음을 주고받아요.
`worker_io.py` 가 이 규약을 대신 처리하니 보통은 신경 쓸 필요가 없어요.

| 방향 | 내용 |
|---|---|
| 엔진 → 노드 | `{"vertex": <노드 정의>, "inputs": {"<앞 노드 id>": <결과>}}` |
| 노드 → 엔진 (0번 이상) | `{"event": {"kind": "...", "data": ...}}` |
| 노드 → 엔진 (마지막 1번) | `{"ok": true, "result": ...}` 또는 `{"ok": false, "error": "..."}` |
