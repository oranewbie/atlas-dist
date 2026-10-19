# atlas-engine 그래프 만들기 가이드

> [atlas-engine 소개](README.md) 를 먼저 읽고 오시면 좋아요. 이 문서는 그래프 JSON 을 직접 쓰거나,
> Python 노드를 만들거나, 프로그램에서 엔진을 부르려는 분을 위한 실습 안내서예요.
>
> **Atlas Studio 사용자라면** 그래프 편집기가 이 JSON 을 대신 만들어 줘요. 원리가 궁금할 때 읽어 보세요.

## 1. 첫 그래프 — 세 상자 잇기

가장 단순한 그래프부터 볼게요. 숫자 목록을 만들고 → 예측하고 → 결과를 그대로 보여 주는 3단계예요.

```json
{
  "id": "forecast-demo",
  "name": "forecast",
  "nodes": [
    { "id": "history", "node_type": "py",       "params": { "op": "value", "value": [10, 12, 11, 13, 12, 14] } },
    { "id": "fc",      "node_type": "forecast", "params": { "window": 3, "horizon": 3 } },
    { "id": "out",     "node_type": "echo" }
  ],
  "edges": [
    { "from": "history", "to": "fc" },
    { "from": "fc",      "to": "out" }
  ]
}
```

읽는 법:

- `nodes` — 상자 목록이에요. `id` 는 이 그래프 안에서의 이름, `node_type` 은 **어떤 실행기로 돌릴지**를 가리켜요.
- `params` — 그 상자에 주는 설정값이에요. 아무 JSON 이나 넣을 수 있어요.
- `edges` — 화살표 목록이에요. 키 이름은 꼭 **`from`, `to`** 여야 해요.

`node_type` 이름(`py`, `forecast`, `echo`)은 미리 **런타임으로 등록**해 둔 이름이에요(5장).
등록 안 된 이름을 쓰면 실행이 바로 실패해요.

### 그래프 문서의 필드

| 위치 | 필수 | 선택 |
|---|---|---|
| 그래프 | `id`, `name` | `namespace`, `max_supersteps`(기본 30), `description`, `nodes`, `edges` |
| 노드 | `id`, `node_type` | `name`, `params`, `runtime`, `namespace`, 서브그래프용 `nodes`·`edges` |
| 엣지 | `from`, `to` | `link_type`(기본 `DATA`), `condition` |

화면 편집기가 넣는 `position`, `react_flow_node` 같은 필드는 엔진이 무시해요. 그래서 편집기가 저장한
파일을 그대로 엔진에 넘겨도 돼요.

## 2. 노드는 무엇을 받나

노드는 **"누가 무엇을 보냈는지"** 를 담은 맵을 받아요.

```json
{ "history": [10, 12, 11, 13, 12, 14] }
```

- 키는 보낸 노드의 `id`, 값은 그 노드의 결과예요.
- 시작 노드(들어오는 화살표가 없는 노드)는 `{"__entry__": null}` 을 받아요. 정상이에요.
- **실행할 때 바깥에서 입력값을 따로 넣는 기능은 아직 없어요.** 실행마다 바뀌는 값은 시작 노드의
  `params` 에 넣어 주세요.

노드가 `null` 을 돌려주면 그 뒤로는 아무것도 보내지 않아요("여기서 멈춤").

## 3. 연결 방식 6가지

엣지의 `link_type` 으로 흐름의 성격을 정해요.

| link_type | 하는 일 | 언제 쓰나 |
|---|---|---|
| `DATA` (기본) | 결과를 그대로 다음 노드에 넘겨요 | 대부분의 경우 |
| `CONDITION` | 결과가 `condition` 값과 **같을 때만** 흐름이 이어져요 | 분기, 루프 탈출 |
| `ITERATE` | 배열을 받아 **원소마다 하나씩 차례로** 다음 노드를 실행해요 | 목록 항목별 처리 |
| `ITERATE_PARALLEL` | 원소마다 **동시에** 실행해요 | 서로 독립인 항목의 빠른 처리 |
| `JOIN` | 앞선 노드들이 **모두 도착할 때까지** 기다렸다가 실행해요 | 여러 갈래 결과 합치기 |
| `CONFIG` | 지금은 `DATA` 와 똑같이 동작해요 | — |

알아 두면 좋은 규칙:

- **CONDITION 은 결과가 아니라 "입력"을 넘겨요.** 게이트 노드는 "통과/불통과"만 판정하고, 원래 데이터는
  그대로 다음으로 흘러가요.
- CONDITION 비교는 **타입까지** 같아야 해요. `"1"`(문자)과 `1`(숫자)은 달라요. 결과가 배열이면
  "포함되어 있는지"로 판정해요. `condition` 이 없는 CONDITION 엣지는 절대 발화하지 않아요.
- ITERATE 계열은 **그 노드로 들어오는 엣지가 딱 하나**이고 입력이 배열일 때 동작해요.

### 예: 될 때까지 돌기 (루프)

```json
{
  "id": "cycle-demo",
  "name": "cycle",
  "nodes": [
    { "id": "kick", "node_type": "py",      "params": { "op": "value", "value": "go" } },
    { "id": "loop", "node_type": "counter", "params": { "threshold": 3 } },
    { "id": "out",  "node_type": "echo" }
  ],
  "edges": [
    { "from": "kick", "to": "loop" },
    { "from": "loop", "to": "loop", "link_type": "CONDITION", "condition": "again" },
    { "from": "loop", "to": "out",  "link_type": "CONDITION", "condition": "done" }
  ]
}
```

`counter` 는 세 번째까지 `"again"` 을 내보내서 자기 자신에게 돌아가고, 그다음 `"done"` 을 내보내
`out` 으로 빠져나가요. 실수로 끝나지 않는 루프를 만들어도 `max_supersteps`(기본 30)에서 멈춰요.

### 서브그래프 (그래프 안의 그래프)

노드에 `nodes` 와 `edges` 를 넣으면 그 노드는 작은 그래프가 돼요. 안쪽 그래프의 마지막 노드
(나가는 화살표가 없는 노드) 결과가 바깥으로 나와요. 마지막 노드가 여러 개면
`{노드id: 결과}` 모양으로 나와요. 몇 겹이든 겹쳐 넣을 수 있어요.

## 4. Python 노드 만들기

대부분의 실제 작업은 Python 노드로 만들어요. 엔진이 Python 프로세스를 미리 띄워 두고 재사용하기 때문에,
노드를 부를 때마다 Python 을 새로 켜지 않아 빨라요.

### 최소 예제

```python
# my_node.py
import sys, os
sys.path.insert(0, os.path.dirname(__file__))
import worker_io          # 번들의 workers/ 폴더에 있어요. 이 파일 옆에 복사해 두세요.

def handle(req):
    vertex = req["vertex"]                 # 이 노드의 정의 (id, params …)
    inputs = req.get("inputs") or {}       # 앞 노드들이 보낸 값
    params = vertex.get("params") or {}

    worker_io.node_trace(vertex["id"], "start")       # 진행 상황 알리기
    result = {"seen": list(inputs.values()), "params": params}
    worker_io.send_stream("partial", result)           # 중간 결과 흘려보내기
    return result                                      # None 을 돌려주면 뒤로 흐르지 않아요

if __name__ == "__main__":
    worker_io.serve(handle)
```

규칙은 세 가지예요.

1. `handle(req)` 함수 하나만 만들면 돼요. 돌려준 값이 이 노드의 결과예요.
2. **`print()` 로 화면에 쓰지 마세요.** 표준 출력은 엔진과 대화하는 통로라서 섞이면 통신이 깨져요.
   로그는 `sys.stderr` 로 쓰세요.
3. `handle` 안에서 오류가 나면 **그 노드만 실패**하고, Python 프로세스는 살아남아 다음 일을 받아요.

### 실행 중에 소식 보내기

| 함수 | 용도 |
|---|---|
| `worker_io.send_stream(이름, 내용)` | 중간 결과를 흘려보내요 (`kind="stream"`) |
| `worker_io.node_trace(이름, 상태, 메시지)` | 진행 상태를 알려요 (`kind="trace"`) |
| `worker_io.send(kind, data)` | 원하는 이름의 채널로 아무 데이터나 보내요 |

이 소식들은 실행 이벤트(`NodeMessage`)로 실시간 전달돼요.

### 상태를 기억하는 노드

취소 후 다시 불러올 때 노드 내부 상태도 이어 가고 싶다면 저장/복원 함수를 넘겨요.

```python
worker_io.serve(handle, dump_state=my_dump, load_state=my_load)
```

이런 노드는 등록할 때 `pool_size: 1` 로 두세요(프로세스가 하나여야 상태가 하나로 유지돼요).

### 노드 소스 디버깅

등록할 때 환경 변수 `ATLAS_DEBUGPY_PORT=5678` 을 주면, 그 노드의 Python 이 debugpy 로 해당 포트에서
디버거 연결을 기다려요(`ATLAS_DEBUGPY_WAIT=1` 이면 연결될 때까지 실행을 멈추고 기다려요).
워커 환경에 `debugpy` 가 설치되어 있어야 해요. Atlas Studio 그래프 편집기의 🐞 버튼이 이걸 대신 해 줘요.

## 5. 런타임 등록 — "이 이름은 이 스크립트로 돌려라"

그래프에 쓴 `node_type` 이름을 실제 실행기와 연결하는 일이에요. 한 번 등록하면 디스크에 저장되어
엔진을 다시 켜도 남아 있어요(`~/.atlas/registry/`).

### Python 노드 등록

```json
{
  "script": "C:/work/nodes/my_node.py",
  "python": "C:/work/.venv/Scripts/python.exe",
  "pool_size": 4
}
```

| 키 | 뜻 |
|---|---|
| `script` | 노드 스크립트 경로 (필수) |
| `python` | 쓸 Python. 생략하면 설정 파일의 `python` |
| `pool_size` | 미리 띄울 프로세스 수 (기본 4) |
| `env` | 노드에 넘길 환경 변수 `{ "KEY": "VALUE" }` |
| `env_path` | 가상환경 경로. 없으면 `uv` 로 만들어 줘요 |
| `requirements` | `requirements.txt` 경로. 빠진 패키지만 설치해요 |
| `wheelhouse` | 미리 받아 둔 패키지 폴더 — **인터넷 없이** 설치할 때 |

### 내장 노드 등록

```json
{ "name": "demand_forecast" }
```

| 내장 노드 | 하는 일 | params 예 |
|---|---|---|
| `demand_forecast` | 이동평균 수요 예측 (숫자 배열 → 예측값) | `{"window":3,"horizon":3}` |
| `rust_echo` | 받은 입력을 그대로 돌려줘요 (확인용) | `{}` |
| `counter` | 정한 횟수까지 `"again"`, 그다음 `"done"` (루프용) | `{"threshold":3}` |

### namespace — 프로젝트끼리 이름이 겹치지 않게

등록할 때 namespace 를 주면 이름이 `"프로젝트A/my_node"` 처럼 저장돼서, 다른 프로젝트의 같은 이름과
섞이지 않아요. Atlas Studio 는 프로젝트마다 자동으로 namespace 를 붙여요.

### AI 노드는?

따로 종류가 있는 게 아니에요. **atlas-agent 가 설치된 가상환경을 `python` 으로 지정한 Python 노드**가
곧 AI 노드예요. 노드 안에서 에이전트를 한 번씩 호출하는 식으로 써요.

## 6. 실행하고 지켜보기

| 하고 싶은 일 | gRPC | MCP 도구 |
|---|---|---|
| 그래프 제출 | `SubmitGraph` → `run_id` | `submit_graph` |
| 제출하고 끝날 때까지 기다리기 | — | `run_graph` |
| 실시간 이벤트 보기 | `StreamRun` | `get_run_events` |
| 현재 상태 | `GetState` | `get_run_state` |
| 일시정지 / 이어 하기 | `Pause` / `ResumeRun` | `pause_run` / `resume_run` |
| 취소 (상태 저장) | `Cancel` | `cancel_run` |
| 저장한 실행 다시 불러오기 | `LoadRun` | `load_run` |
| 중단점 | `SetBreakpoints` | `set_breakpoints` |

- **일시정지**는 박자(super-step) 사이에서 걸려요. 돌고 있는 노드를 중간에 끊지는 않아요.
- **취소**하면 그 시점 상태가 `~/.atlas/runs/…` 에 저장되고, 나중에 `LoadRun` 으로 이어서 돌릴 수 있어요.
  성공·실패로 끝난 실행은 기록(이벤트 로그)만 남아요.
- **중단점**을 건 노드는 실행되기 **직전**에 일시정지돼요.
- 엔진을 다시 켜면 진행 중이거나 일시정지된 실행은 사라져요. 오래 멈춰 둘 실행은 취소해서 저장해 두세요.

실행 상태 값: `PENDING`(대기) · `RUNNING`(실행 중) · `PAUSED`(일시정지) · `SUCCEEDED`(성공) ·
`FAILED`(실패) · `CANCELLED`(취소).

## 7. 자주 겪는 문제

| 증상 | 원인과 해결 |
|---|---|
| `graph_json: …` 오류 | JSON 문법 오류이거나 필수 필드가 없어요. 엣지 키가 `from`/`to` 인지 확인하세요 |
| 바로 FAILED, 이벤트 없음 | 대부분 **등록 안 된 `node_type`** 이에요. `list_runtimes`/`ListRuntimes` 로 확인하세요 (namespace 도 맞아야 해요) |
| `unknown builtin runtime` | 내장 노드 이름 오타예요. `search_nodes` 로 목록을 보세요 |
| `python_worker runtime needs config.script` | 등록 설정에 `script` 가 없거나 설정이 JSON 객체가 아니에요 |
| 아무것도 안 돌고 바로 끝남 | 시작 노드가 없어요(그래프 전체가 원으로 이어져 있음) |
| 노드가 `{"__entry__": null}` 을 받음 | 정상이에요. 데이터는 `params` 로 넣으세요 |
| 노드가 실행되지 않음 | JOIN 의 앞 노드 중 하나가 안 왔거나, 앞 노드가 `null` 을 돌려줬어요 |
| ITERATE 가 한 번만 돎 | 들어오는 엣지가 하나인지, 입력이 배열인지 확인하세요 |
| CONDITION 분기가 안 탐 | 값과 **타입**이 정확히 같은지 확인하세요 (`"1"` ≠ `1`) |
| 루프가 안 끝남 | `max_supersteps`(30)에서 멈춰요. 탈출 조건을 확인하세요 |
| `run_graph` 가 `timed_out: true` | 기다리기만 끝난 거예요. 실행은 계속되고 있어요 |
| 다시 불러온 뒤 Python 노드 상태가 없음 | `dump_state`/`load_state` 를 넘기고 `pool_size: 1` 로 등록하세요 |
| Python 노드 통신 오류 | 노드에서 `print()` 를 쓰지 않았는지 확인하세요 (로그는 stderr 로) |

더 자세한 API 와 설정은 [API·설정 레퍼런스](reference.md) 를 보세요.
