# atlas-agent 설정 가이드

> 기준 버전 v0.1.4. 설치는 [소개 › 설치](README.md#설치) 를 먼저 보세요.

## 1. 설정 파일은 어디에

| 파일 | 담는 것 |
|---|---|
| `conf/<프로필>/atlas-agent.yaml` | 모델 설정과 동작 옵션(`options`) |
| `conf/<프로필>/secrets.toml` | API 키 |

기본 프로필은 `default` 예요. 어떤 파일을 쓸지는 이 순서로 정해져요.

1. 환경 변수 **`ATLAS_AGENT_CONFIG`** 가 가리키는 파일
2. `<설치 폴더>/conf/<ATLAS_PROFILE 또는 default>/atlas-agent.yaml`

| 환경 변수 | 뜻 |
|---|---|
| `ATLAS_ROOT` | 설치 폴더(번들 루트). 시작 스크립트가 자동으로 정해요 |
| `ATLAS_CONF_DIR` | 설정 폴더 (기본 `<루트>/conf`) |
| `ATLAS_PROFILE` | 프로필 이름 (기본 `default`). 그 프로필에 파일이 없으면 `default` 를 써요 |

**Atlas Studio 로 쓸 때**는 `~/.atlas/atlas-agent.yaml` 이 설정 파일이에요. 설정 ⚙ › 환경 설정 › 에이전트 › **Config** 버튼으로 열어 고쳐요.

## 2. 모델 연결하기

모델은 **역할**별로 정해요. 최소한 `SYSTEM`(메인 대화) 하나만 있으면 돼요.

| 역할 | 쓰이는 곳 |
|---|---|
| `SYSTEM` | 메인 대화 턴 |
| `COMPACT` | 대화가 길어졌을 때 요약(압축) |
| `CODING` · `REASONING` · `NLTOSQL` | 특정 용도로 고를 수 있는 모델 |

### Claude (Anthropic)

```yaml
llm_config:
  prefer_model_config:
    SYSTEM:
      model_provider: "anthropic"
      model_name: "claude-opus-4-8"
```

### OpenAI

```yaml
    SYSTEM:
      model_provider: "openai"
      model_name: "gpt-4.1"
```

### Azure OpenAI

```yaml
    SYSTEM:
      model_provider: "azure_openai"
      model_name: "gpt-4.1"
      model_param:
        base_url: "https://<리소스>.openai.azure.com/"
        deployment_name: "<배포 이름>"
        api_version: "2024-10-21"
```

표준 채팅 모델만 보장해요. 일부 최신 모델은 다른 API 방식이 필요해 안 될 수 있어요.

### 사내·로컬 모델 (vLLM · TensorRT-LLM · ollama)

OpenAI 호환 주소를 가진 서버라면 무엇이든 붙어요. 서버를 이름으로 등록하고, 역할에서 그 이름을 가리켜요.

```yaml
llm_config:
  openai_compatible_servers:
    my_gpu:   { base_url: "http://192.168.0.3:8000/v1", api_key: "dummy" }
    ollama:   { base_url: "http://localhost:11434/v1",  api_key: "ollama" }
  prefer_model_config:
    SYSTEM:
      model_provider: "openai_compatible"
      model_name: "Qwen/Qwen3.6-35B-A3B-FP8"
      model_param: { server_id: "my_gpu", max_tokens: 16000 }
    COMPACT:
      model_provider: "openai_compatible"
      model_name: "Qwen/Qwen3.6-35B-A3B-FP8"
      model_param: { server_id: "my_gpu" }
```

- 도구를 쓰려면 **function calling(도구 호출)을 지원하는 모델**을 고르세요.
- 작은 모델은 긴 지시를 잘 따르지 못해요. 결과가 이상하면 더 큰 모델로 바꿔 보세요.

### `model_param` 에 넣을 수 있는 것

`base_url` · `api_key` · `deployment_name` · `api_version` · `max_tokens`(또는 `max_completion_tokens`) · `temperature` · `server_id`

### 모델 정보 덮어쓰기

모델의 문맥 크기를 엔진이 모르면 작게(8k) 잡아요. 사내 모델은 알려 주는 게 좋아요.

```yaml
models:
  "Qwen/Qwen3.6-35B-A3B-FP8":
    context_window: 131072
    max_output_tokens: 16000
```

## 3. API 키

찾는 순서: **설정에 직접 쓴 값 → 환경 변수 → `secrets.toml`**

| 공급자 | 환경 변수 | `secrets.toml` 키 |
|---|---|---|
| Anthropic | `ANTHROPIC_API_KEY` | `anthropic_api_key` |
| OpenAI | `OPENAI_API_KEY` | `openai_api_key` |
| Azure OpenAI | `AZURE_OPENAI_API_KEY` | `azure_api_key` |
| 웹 검색 | — | `web_search_api_key` |
| OpenAI 호환 서버 | — | `openai_compatible_servers.<이름>.api_key` (yaml) |

## 4. 동작 옵션 (`options`)

yaml 의 `options:` 아래에 써요. 같은 내용을 환경 변수로도 줄 수 있고, **환경 변수가 이겨요.**

```yaml
options:
  workspace:
    dir: "C:/work"
  permissions:
    outside_write: ask
  web:
    search:
      provider: searxng
      base_url: "http://search.internal:8080"
  sandbox:
    mode: auto
```

| 키 | 환경 변수 | 기본 | 뜻 |
|---|---|---|---|
| `workspace.dir` | `ATLAS_AGENT_WORKSPACE` | `~/.atlas/workspace` | 세션이 작업 폴더를 안 줬을 때 쓸 폴더 |
| `memory.disabled` | `ATLAS_DISABLE_MEMORY` | `false` | 자동 메모리 끄기 |
| `memory.bare` | `ATLAS_BARE` | `false` | 디스크에 아무 기록도 남기지 않기 |
| `compact.disabled` | `DISABLE_COMPACT` | `false` | 자동 요약(압축) 끄기 |
| `permissions.outside_read` | `ATLAS_AGENT_OUTSIDE_READ` | `allow` | 작업 폴더 밖 읽기: `allow` / `ask` |
| `permissions.outside_write` | `ATLAS_AGENT_OUTSIDE_WRITE` | `ask` | 작업 폴더 밖 쓰기: `ask` / `allow` |
| `permissions.ask_timeout_sec` | `ATLAS_AGENT_PERMISSION_TIMEOUT` | `600` | 승인 질문에 답이 없으면 몇 초 뒤 거부할지 (0 이하 = 무기한) |
| `permissions.allow_bypass` | `ATLAS_AGENT_ALLOW_BYPASS` | `false` | "전부 허용" 모드를 받아들일지 |
| `hooks.enabled` · `hooks.project_enabled` | — | `true` | 훅 사용 / 프로젝트 폴더의 훅 사용 (모르는 저장소를 열 땐 끄세요) |
| `web.proxy` · `web.timeout_sec` · `web.user_agent` | — | — / 30 / — | 웹 도구 |
| `web.search.provider` | `ATLAS_AGENT_WEB_SEARCH` | `none` | 웹 검색: `searxng` · `brave` · `tavily`. `none` 이면 검색 도구가 없어요 |
| `sandbox.mode` | `ATLAS_AGENT_SANDBOX` | `off` | 명령 격리: `off` · `docker` · `auto` |
| `usage.enabled` | — | `true` | 토큰 사용량 기록 (숫자만) |
| `tracker.enabled` | `LLM_CALL_TRACKER_ENABLED` | `false` | LLM 호출 상세 기록 (문제 분석용) |
| `tools.bind_budget` | — | 모델 문맥의 8% | 도구 설명에 쓸 문맥 예산 |

## 5. 권한 — 무엇을 묻고 무엇을 막을지

### 모드

| 모드 | 동작 |
|---|---|
| `default` | 위험한 명령과 작업 폴더 밖 쓰기만 물어봐요 |
| `edit` | 위 + 작업 폴더 안의 파일 삭제 같은 명령도 묻지 않아요 |
| `readonly` | 읽기만. 고치는 도구는 모두 거부 (계획 세우기용) |
| `bypass` | 거부 규칙 외엔 전부 허용. `permissions.allow_bypass` 가 켜져 있어야 해요 |

### 규칙 파일 (`permission.json`)

```json
{
  "allow": ["Bash(git status*)", "Edit(src/**)", "mcp__atlas-engine__*"],
  "ask":   ["Bash(git push*)"],
  "deny":  ["Read(**/.env)", "WebFetch(domain:*.internal)"],
  "additionalDirectories": ["D:/shared-data"]
}
```

| 위치 | 범위 |
|---|---|
| `~/.atlas/permission.json` | 이 PC 의 모든 프로젝트 |
| `<작업 폴더>/.atlas/permission.json` | 이 프로젝트 — 팀과 공유 (git) |
| `<작업 폴더>/.atlas/private/agent/permission.json` | 이 프로젝트, 나만 |

- 판정 순서: 훅 → **거부** → 물어보기 → 허용 → 작업 폴더 범위 → 모드 기본값
- **거부(deny)는 무엇으로도 풀 수 없어요.** 세션 설정으로도 지워지지 않아요.
- `a && b` 처럼 이어 쓴 명령은 조각마다 따로 판정해요. 허용된 명령 뒤에 위험한 명령을 붙여도 그냥 통과하지 않아요.
- 승인 카드에서 "항상 허용"을 고르면 이 세션 / 이 PC / 프로젝트 중 한 곳에 규칙이 저장돼요.

## 6. 샌드박스 — 명령을 격리해서 실행

| `sandbox.mode` | 동작 |
|---|---|
| `off` (기본) | 내 PC 에서 바로 실행. 파일 도구는 작업 폴더로 제한되지만 **OS 격리는 아니에요** |
| `docker` | 명령마다 일회용 컨테이너에서 실행 (네트워크 없음, 권한 최소화, 작업 폴더만 연결). Docker 가 없으면 **실패**해요(내 PC 로 몰래 돌아가지 않아요) |
| `auto` | Docker 를 쓸 수 있으면 docker, 아니면 off |

| 컨테이너 옵션 | 기본 |
|---|---|
| `sandbox.image` | `python:3.12-slim` |
| `sandbox.pull` | `missing` (폐쇄망은 `never` 로 두고 이미지를 미리 받아 두세요) |
| `sandbox.network` | `none` |
| `sandbox.memory` · `sandbox.cpus` · `sandbox.pids_limit` | `1g` · `2` · `256` |

## 7. 웹 검색 연결

| provider | 필요한 것 |
|---|---|
| `searxng` | 자체 운영 SearXNG 주소(`base_url`). 서버에서 `format=json` 을 켜 두세요. **폐쇄망에 추천** |
| `brave` | API 키 |
| `tavily` | API 키 |

웹 페이지 읽기(WebFetch)는 검색 설정과 상관없이 늘 쓸 수 있어요. 처음 보는 도메인은 승인을 물어봐요.

## 8. 훅 — 내 규칙 끼워 넣기

도구 실행 전후 같은 순간에 내 스크립트를 돌려 검사하거나 막을 수 있어요. 형식은 Claude Code 의 `hooks.json` 과 같아요.

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash|Write", "hooks": [{ "type": "command", "command": "python check.py", "timeout": 30 }] }
    ]
  }
}
```

| 이벤트 | 언제 |
|---|---|
| `SessionStart` | 세션 시작 |
| `UserPromptSubmit` | 사용자가 메시지를 보낼 때 |
| `PreToolUse` · `PostToolUse` | 도구 실행 전 · 후 |
| `Stop` · `SubagentStop` | 턴 종료 · 서브에이전트 종료 |
| `PreCompact` | 대화 요약 직전 |

- 스크립트가 **종료 코드 2** 로 끝나면 차단돼요(stderr 내용이 이유로 전달돼요). 0 이면 통과.
- 시간 초과 시 `PreToolUse` 는 **차단**, 나머지는 통과로 처리해요.
- 위치: `~/.atlas/hooks.json`, `<작업 폴더>/.atlas/hooks.json`, `<작업 폴더>/.atlas/private/agent/hooks.json`

## 9. 세션별로 바꾸기

프로그램에서 세션을 만들 때(`CreateSession`) 일부 설정을 그 세션에만 줄 수 있어요.

| 바꿀 수 있는 것 | 예 |
|---|---|
| 모델 | `model_provider`, `model_name`, `model_param` |
| 권한 | `permissions` (모드·규칙 추가 — 단 거부 규칙은 지울 수 없어요) |
| 샌드박스 | `sandbox` (운영자가 docker 로 정했으면 끌 수 없어요) |
| 시스템 프롬프트 | `system_prompt`, `append_system_prompt` |
| 이어 하기 | `resume_session_id` |
| 한 번만 묻기 | `single_turn: true` — 도구 없이 AI 에게 한 번만 물어보는 모드 (요약·분류·번역 등) |

`options` 와 `models` 는 서버 전체 설정이라 세션별로는 못 바꿔요.
