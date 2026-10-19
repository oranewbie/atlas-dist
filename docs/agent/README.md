# atlas-agent — AI 에이전트 서버

> 현재 버전 **v0.1.4** (프리뷰) · 다운로드: [Releases](https://github.com/oranewbie/atlas-dist/releases) 의 `agent-v*` 태그
>
> 이어서 읽기: [설정 가이드](configuration.md) · [기능·API 레퍼런스](reference.md)

## 한 줄로 말하면

**"파일을 읽고, 명령을 실행하고, 고치고, 확인하면서 스스로 일을 끝내는 AI 일꾼을 내 PC·내 서버에 띄우는 프로그램"** 이에요.

채팅 AI 는 답을 "말"로만 줘요. atlas-agent 는 **도구**를 써요. 폴더의 파일을 읽고, 명령을 실행하고,
결과를 보고 다시 고치는 일을 한 턴 안에서 반복해요. 위험한 일을 하려 할 땐 먼저 사람에게 물어봐요.

- **여러 AI 모델을 골라 써요** — Claude, OpenAI, Azure OpenAI, 그리고 vLLM·ollama 같은
  **사내·로컬 모델**(OpenAI 호환 서버)도 같은 방식으로 써요. 인터넷이 막힌 폐쇄망에서도 돼요.
- **서버로 떠서 여러 프로그램이 함께 써요** — Atlas Studio, 사내 앱(Java·Python·TypeScript SDK), 그래프 엔진이 붙어요.
- **반복 작업은 "스킬"로 굳혀요** — 절차를 문서(SKILL.md) 하나로 남기면 다음부터 이름만 불러 같은 품질로 수행해요.
- **사람 없이도 돌려요** — 스킬을 잡(job)으로 실행하거나, 정해진 시각·주기마다 자동 실행(트리거)해요.
- **오프라인 설치** — 설치 파일 안에 필요한 패키지가 다 들어 있어서 인터넷 없이 설치돼요.

## 이런 데 써요

| 활용 | 예 |
|---|---|
| **문서 작업** | 여러 문서 요약, 보고서 초안 작성, 일괄 문구 치환, 템플릿대로 문서 만들기 |
| **데이터 분석** | 폴더의 CSV·엑셀·Parquet·JSON 을 복사 없이 바로 SQL 로 집계 (DuckDB 내장) |
| **조사·정리** | 웹 검색 + 페이지 읽기로 자료를 모아 정리 (폐쇄망은 사내 검색 서버 연결) |
| **파일 정리** | 많은 파일을 기준에 따라 분류하고 결과를 표로 |
| **정기 작업** | 매일 아침 리포트 생성, 폴더 감시, 외부 알림을 받으면 스킬 실행 |
| **사내 앱에 AI 붙이기** | SDK 로 우리 앱의 기능을 AI 도구로 열어 주고, 사내 정책(금지 명령 등)을 강제 |
| **개발** | 테스트 실행 → 실패 수정, 리팩터링, 코드 점검 |

> **Atlas Studio 를 쓰신다면** 설정 ⚙ › 환경 설정 › 에이전트에서 백엔드를 `atlas-agent` 로 바꾸기만 하면 돼요.
> 설치·실행은 Studio 가 알아서 해요. 이 문서는 서버를 따로 띄우거나 내 앱에 붙이려는 분을 위한 거예요.

## 핵심 개념

| 개념 | 뜻 |
|---|---|
| **세션** | 대화 하나. 세션마다 작업 폴더·모델·권한이 따로 있어요. 기록은 작업 폴더의 `.atlas/private/agent/chats/` 에 남아 이어서 할 수 있어요 |
| **도구** | 에이전트가 쓰는 능력 — 명령 실행(Bash), 파일 읽기·쓰기·고치기, 검색, 웹, 데이터 SQL, 할 일 목록, 서브에이전트 |
| **권한** | 도구를 쓸 때 "허용·물어보기·거부" 규칙. 기본은 위험한 명령과 작업 폴더 밖 쓰기만 물어봐요 |
| **AGENT.md** | 에이전트가 항상 지킬 규칙 파일 (`.atlas/AGENT.md`) |
| **메모리** | 대화하며 알게 된 사실을 `.atlas/memory/` 에 적어 두고 다음 세션에 참고해요 |
| **스킬** | 반복 절차를 담은 `SKILL.md`. `/이름` 으로 부르거나, 에이전트가 알아서 골라 써요 |
| **서브에이전트** | 큰 일을 쪼개 다른 에이전트에게 맡기고 요약만 받아요 (탐색·계획·검증 등) |
| **잡 · 트리거** | 사람 없이 스킬 실행 · 주기/시각/외부 이벤트로 자동 실행 |
| **훅** | 도구 실행 전후 등에 내 스크립트를 끼워 넣어 규칙을 강제해요 |
| **MCP** | 외부 도구 서버(그래프 엔진 등)를 연결해요 |

## 설치

### 1. 받기

| OS | 파일 |
|---|---|
| Windows (x64) | `atlas-agent-bundle-<버전>-win-amd64-py3.12.zip` |
| Linux (x64) | `atlas-agent-bundle-<버전>-linux-amd64-py3.12.tar.gz` |

파일 이름의 `py3.12` 는 **필요한 Python 버전**이에요. macOS·ARM 번들은 아직 없어요.

### 2. 준비물

- **Python 3.12** — 번들 이름과 **같은 버전**이어야 해요(다르면 설치가 멈춰요).
- 인터넷·pip·uv 는 필요 없어요.
- (선택) Docker — 명령을 격리된 컨테이너에서 실행하고 싶을 때.

### 3. 설치

압축을 풀고 그 폴더에서 실행해요. **압축을 푼 폴더가 곧 설치 위치**예요.

```powershell
# Windows
.\install.ps1
```

```sh
# Linux
./install.sh
```

설치 스크립트는 전용 가상환경(`venv/atlas-agent/`)을 만들고, 설정 파일 견본을
`conf/default/atlas-agent.yaml`, `conf/default/secrets.toml` 로 복사해 줘요.

### 4. 모델 설정

`conf/default/atlas-agent.yaml` 에 어떤 AI 모델을 쓸지 적어요. 가장 간단한 예(Claude):

```yaml
llm_config:
  prefer_model_config:
    SYSTEM:
      model_provider: "anthropic"
      model_name: "claude-opus-4-8"
```

API 키는 `conf/default/secrets.toml` 에 넣거나 환경 변수(`ANTHROPIC_API_KEY` 등)로 줘요.

```toml
anthropic_api_key = "sk-ant-..."
```

사내 모델, OpenAI, Azure 설정은 [설정 가이드](configuration.md) 를 보세요.

> 설정 파일을 못 찾아도 서버는 켜지지만 **기본값(Claude)으로 조용히** 동작해요. 의도한 모델이 아닌 것 같으면
> CLI 의 `/config` 로 지금 설정을 확인하세요.

### 5. 실행

```powershell
.\run\agent.ps1                 # Windows
```

```sh
./run/agent.sh                  # Linux
```

`atlas-agent gRPC server listening on 127.0.0.1:50061` 이 보이면 준비 끝이에요.

| 옵션 | 기본 | 뜻 |
|---|---|---|
| `--host` | `127.0.0.1` | 접속 주소 |
| `--port` | `50061` | 포트 |
| `--max-jobs` | `4` | 동시에 돌릴 잡 수 |

**끄기**: 실행 창에서 Ctrl+C 를 누르거나, gRPC `Shutdown` 을 호출해요.

> 지금 버전은 **인증·암호화(TLS)가 없어요.** 기본값처럼 내 PC(127.0.0.1)에서만 쓰거나, 믿을 수 있는 네트워크에서만 여세요.

### 서버 없이 바로 써 보기 (CLI)

```sh
venv/atlas-agent/Scripts/atlas-agent --cwd C:/work/my-project     # Windows
venv/atlas-agent/bin/atlas-agent --cwd ~/work/my-project          # Linux
```

터미널에서 바로 대화해요. 도구 사용 승인은 `[y/N, a=이 세션 동안 항상]` 으로 답해요.

| 명령 | 뜻 |
|---|---|
| `/config` | 지금 설정 보기 (키는 가려져요) |
| `/model <provider> <model>` | 메인 모델 바꾸기 (설정 파일에 저장) |
| `/llm <역할> <provider> <model>` | 역할별 모델 바꾸기 (`/llm help`) |

### Atlas Studio 로 설치하기 (가장 쉬운 방법)

Studio 의 **설정 ⚙ › 환경 설정 › 구성 요소 › 에이전트 서버** 카드에서 **[설치]** 를 누르거나 `atlas agent install` 을
실행해요. 설치 위치는 `~/.atlas/atlas-agent` 예요. 이후로는 챗을 시작할 때 Studio 가 프로젝트마다 서버를 띄우고,
Studio 를 닫으면 같이 꺼요. 설정 파일은 `~/.atlas/atlas-agent.yaml` 을 쓰고, 에이전트 서브탭의 **Config** 버튼으로 고쳐요.

## 지금 할 수 있는 것 / 아직 못 하는 것

**할 수 있어요**
- Claude·OpenAI·Azure OpenAI·OpenAI 호환 사내 모델, 역할별·세션별·서브에이전트별 모델 지정
- 내장 도구: 명령 실행, 파일 읽기·쓰기·고치기·찾기, 웹 페이지 읽기·검색, 데이터 SQL(DuckDB), 할 일 목록, 서브에이전트, 스킬
- 권한 규칙(허용·물어보기·거부)과 4가지 모드, 훅, MCP 연결, 플러그인
- 대화가 길어지면 자동 요약(압축), 메모리, 계층형 AGENT.md
- 잡·트리거로 사람 없이 실행, Java·Python·TypeScript SDK
- Docker 샌드박스(선택), 오프라인 설치

**아직 안 돼요 / 주의할 점**
- 인증·TLS 가 없어요 — 내 PC 나 믿을 수 있는 네트워크에서만 쓰세요.
- Docker 를 쓰지 않으면 **OS 수준 격리가 없어요.** 권한 규칙이 위험한 명령을 묻긴 하지만, 명령 안에 숨은 경로까지 모두 잡지는 못해요.
- 비용 상한(예산) 설정이 없어요.
- 이미지·PDF 를 파일 도구로 직접 읽지 못해요(텍스트 위주).
- 작은 로컬 모델은 AGENT.md·스킬 지시를 잘 따르지 않을 수 있어요.
- 번들 플랫폼은 Windows·Linux(x64) × Python 3.12 만 있어요. SDK 는 소스 공개(2026년 12월) 때 함께 제공돼요.

## 문서 더 보기

| 문서 | 내용 |
|---|---|
| [설정 가이드](configuration.md) | 모델 연결(Claude·OpenAI·Azure·사내 모델), 옵션, 권한, 샌드박스, 웹 검색 |
| [기능·API 레퍼런스](reference.md) | 도구 목록, 스킬·서브에이전트·훅·MCP, gRPC API, SDK, 파일 위치, 문제 해결 |
