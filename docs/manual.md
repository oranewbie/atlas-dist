# Atlas Studio 사용자 매뉴얼

> Atlas Studio 리포에서 배포된 문서 사본입니다 (v0.1.5 기준) — 직접 수정하지 마세요.

> 설치부터 화면별 사용법까지. 제품 개요는 [소개서](introduction.md), 개념·아키텍처는
> atlas-studio.md 참조. 이 매뉴얼은 Windows 기준으로 쓰였다
> (macOS/Linux 는 경로 표기만 다르다).

## 1. 설치와 첫 실행

### 설치

- 배포 채널(atlas-dist 릴리스)에서 받은 **`…-setup.exe`** 를 실행한다. Node 런타임과
  사이드카가 동봉된 자급자족 번들이라 별도 준비물이 없다.
- 업데이트: 새 버전이 나오면 하단 상태바에 1회 알림이 뜬다(✕ 로 끄면 그 버전은 다시
  묻지 않음). 수동 확인은 타이틀바 **도움말 › 업데이트 확인**. 설치를 누르면 인스톨러를
  받아 실행하고 앱이 종료된다.

### 사용자 등록·라이선스

첫 실행 시 게이트 화면이 뜬다 — **등록**(체험 30일) 또는 **라이선스 키** 입력으로
활성화한다. 라이선스는 이후 설정 ⚙ › 라이센스 탭에서 상태 확인·키 변경이 가능하다.

## 2. 프로젝트 시작

### 새 프로젝트

타이틀바 **프로젝트 › 새 프로젝트…** 로 위저드를 연다. 폴더를 고르고 시작 방식을 택한다:

| 방식 | 내용 |
|---|---|
| **스타터**(빈 워크벤치) | S0 골격 + 가이드만 — 백지에서 시작 |
| **스타터 + 빌더** | 위 + 워크벤치 구성 스킬 — 에이전트가 방법론 인터뷰로 stage 파이프라인을 설계·생성 |
| **워크벤치 템플릿** | 등록된 템플릿(`~/.atlas/workbenches/`)을 적용 — 검증된 방법론으로 즉시 시작 |
| **워크벤치 템플릿 등록 (zip)…** | 전달받은 템플릿 zip 을 등록한 뒤 사용 |

기존 프로젝트는 **프로젝트 › 폴더 열기…** 로 연다(최근 프로젝트는 시작 화면에서).
마지막 프로젝트는 다음 실행 때 자동 복원된다.

### 프로젝트 홈(.atlas/) — 폴더가 진실

```
<프로젝트>/.atlas/
├─ project.yaml               앵커 — 프로젝트명·워크벤치 id
├─ application_context.json   개인 UI 상태(VCS 제외 권장)
├─ workbenches/<id>/          워크벤치 인스턴스 — stages/·tools/·hooks/·reports/·guide.md
├─ launch.yaml                디버그 실행 구성 (.vscode/launch.json 도 자동 임포트)
├─ graphs/                    그래프 문서·노드 클래스 카탈로그
├─ mcp.json                   외부 MCP 서버 등록
├─ wiki/                      작업 산출물 문서(에이전트가 축적, atlas wiki grep 으로 검색)
└─ agent/                     챗 아카이브(chats/)·프로젝트 메모리(memory/)
```

모든 상태는 파일이다 — 서버·DB 없음. 팀 공유는 이 폴더를 VCS 로 나누면 된다.
단, `agent/chats` 에는 대화 원문이 통째로 들어가므로 민감 프로젝트는 ignore 를 권장한다.

## 3. 화면 구성

```
┌ 타이틀바     프로젝트/파일/도움말 메뉴 · 프로젝트명(클릭=앵커 편집) · 경로(표시 전용) · 창 컨트롤
├ 본문
│  ├ Activity Bar   좌측 아이콘 — Explorer·찾기·툴·Activity·플러그인·Run & Debug(+플러그인 기여)
│  │                아이콘 드래그로 순서 변경, 하단 ⚙ = 설정
│  ├ 메인 패널      활성 화면(keep-alive — 화면을 오가도 상태 유지)
│  ├ 하단 도크      터미널 · 로그 · 디버그 콘솔 · 문제  (Ctrl+` 토글)
│  └ 챗 도크        AI 세션 — 상/하/좌/우 도킹, 접기/펼치기
└ 상태바       버전 · 업데이트 알림
```

- **명령 팔레트** `Ctrl+Shift+P` — 모든 커맨드(이동·보기·파일·디버그·stage)를 검색 실행.
  `Ctrl+P` 는 파일 quick-open. 전체 단축키는 [shortcuts.md](shortcuts.md).
- **파일 메뉴**는 활성 화면이 지원하는 액션(새로 만들기/열기/저장/닫기)만 나타난다.
- `Ctrl+R` 새로 고침 — 활성 화면의 ⟳(없으면 프로젝트 다시 읽기).

## 4. Explorer · 찾기

- **Explorer**: 프로젝트 트리 + Monaco 에디터. 외부 변경은 파일 감시로 자동 반영된다.
  중단점은 에디터 거터 클릭으로 토글(디버그 참조).
- **찾기**: ripgrep 기반 전체 검색(대소문자 무시·고정 문자열, 숨김 폴더 포함) +
  일괄 치환. include/exclude 글롭 필터 지원.

## 5. Activity 화면 — 대시보드와 Stage

Activity Bar 의 **Activity** 아이콘이 프로젝트 진행의 중심 화면이다.

- **대시보드**: stage 탭바의 고정 첫 탭 — 전체 진행 현황·KPI.
- **Stage 탭**: stage 마다 탭 하나. 상단에 목표·판정(게이트), 본문에 Activity 카드,
  우측 책갈피 탭으로 **Activity / 히스토리 / 피드백 / Hooks / Todo / CheckList** 를 전환한다.
- 탭 드래그로 순서 변경, 숨기기(폴더·산출물은 유지) 가능. **stage 추가**는 팔레트나
  탭바 + 버튼.

### Activity(작업) 실행

Activity 는 실행 단위다(= Task). 카드에서 실행하며, 종류는 생성 시 택한다:

| 종류 | 동작 |
|---|---|
| **Skill 구성**(agent) | 에이전트가 skill.md 절차를 수행 — 챗 세션 턴으로 실행 |
| **Run (터미널)** | 명령을 통합 터미널에서 실행·관찰 |
| **Debug 실행**(launch) | 디버그 구성(launch config)을 참조해 디버거로 실행 |
| **VS Code 실행** | 연동된 VS Code 창에 태스크/디버그를 위임 |
| **Tool call** | 레지스트리 툴 호출 |

- 카드의 체크박스로 **비활성**(목록에서 숨김 — "비활성 보기"로 표시), 🗑 로 삭제.
- **피드백**: 실행 결과 화면에서 gap/error/quality/idea 를 기록한다 — improve 작업이
  최우선 입력으로 읽는 개선 루프의 연료다.
- **Hooks**: activity 전/후 자동 동작(다른 activity·툴 실행)을 선언한다.

## 6. 챗 — AI 세션

챗 도크는 어느 화면에서나 쓸 수 있다. 화면을 옮겨도 **쓰던 세션이 유지**되며, 화면
전용 세션이 필요하면 ＋ 로 명시 생성한다(세션 탭바는 열린 세션 전부를 보여준다).

- **에이전트(ctx)**: 세션은 여는 화면에 따라 문맥이 정해진다 — 일반 화면은
  Atlas Agent, stage 탭은 Stage Agent, 툴 패널은 Tool Builder, 그래프 화면은
  Graph Agent 등(전체: agents.md). 화면 상태는 턴마다 함께 전달되므로
  이월된 세션도 현재 화면을 안다.
- **에이전트가 할 수 있는 일**: 파일 읽기/쓰기·`atlas` CLI 실행·화면 정찰과 제어
  (열린 화면 확인, 파일 열기, 새로 고침)·VS Code 창 제어·프로젝트 메모리 저장.
  폼/화면 툴(form·render)을 호출하면 챗 카드나 팝업으로 뜬다.
- **권한 승인·질문**: 에이전트의 도구 사용 승인과 선택 질문은 챗 안 카드로 온다.
- **지식 축적**: 짧은 사실은 메모리(`.atlas/agent/memory` — 다음 세션에 자동 주입),
  문서 산출물은 위키(`.atlas/wiki` — `atlas wiki grep` 으로 검색), 대화 원문은
  아카이브(`.atlas/agent/chats` — 프로젝트를 옮겨도 이력 복원)로 남는다.
- **외부 MCP 서버**: `.atlas/mcp.json`(Claude Code `.mcp.json` 과 같은 형식) — 툴
  패널의 "MCP 서버" 섹션에서 등록/삭제. 새 챗 세션부터 적용된다.
- **백엔드**: 기본은 claude-code. 설정 ⚙ › 환경 설정에서 atlas-agent(Forge gRPC)로
  전환할 수 있다 — 서버 설치·기동은 자동이다(§12 제품 탭).
- **오류 표면화**: 턴 실패는 하단 **문제** 탭에, 백엔드 진단 로그는 **로그** 탭에 쌓인다.

Claude 계정은 앱이 로그인 대행을 하지 않는다 — 터미널에서 `claude` 로그인(또는
`claude setup-token`) 후, 설정 ⚙ 에서 계정 저장·전환만 관리한다.

## 7. Run & Debug

Activity Bar 의 **Run & Debug** 아이콘. VS Code Run & Debug 뷰와 같은 구성이다 —
상단 툴바(구성 셀렉트 + 제어), 세션 목록, 변수·콜스택·중단점 섹션.

### 실행 구성

- 진실은 **`.atlas/launch.yaml`** (없으면 "기본 템플릿 생성" 버튼). 기존
  **`.vscode/launch.json` 도 자동 임포트**된다(이름 충돌 시 " (vscode)" 접미사).
- `${workspaceFolder}` 변수 지원. `console: integratedTerminal` 이면 통합 터미널에서
  실행된다. 구성 셀렉트에는 Debug 실행방식 Activity 도 별칭으로 올라온다.

### 실행과 조작

- **F5** 시작(정지 중이면 계속) · **Shift+F5** 중지 · **Ctrl+Shift+F5** 재시작 ·
  **F6** 일시정지 · **F10/F11/Shift+F11** 스텝([shortcuts.md](shortcuts.md)).
- **멀티 세션**: ▶ 는 항상 살아 있어 다른 세션이 도는 중에도 새 구성을 추가로 띄운다
  (backend+front 동시 디버그). 세션 목록 행 클릭 = 포커스 전환, ■ = 개별 중지.
- **중단점**: 에디터 거터 클릭. 중단점 목록에서 ✏ 로 **조건식**(참일 때만 정지) 편집.
- **디버그 콘솔**(하단 도크): 프로그램 출력 + REPL 평가.

### 디버그 어댑터

| 스택 | 어댑터 | 준비 |
|---|---|---|
| Node/TS (`pwa-node` 등) | js-debug | 번들 동봉 — 별도 설치 불필요(갱신: `atlas debug-adapter install js-debug`) |
| Python (`debugpy`) | debugpy | 대상 Python 에 `pip install debugpy` |
| Java (`java`) | JDT LS + java-debug 내장 경로 | 설정 ⚙ › Atlas 제품 › **디버그 어댑터** 카드에서 설치(또는 `atlas debug-adapter install java`). JDK 17+ 필요. 첫 실행은 워크스페이스 임포트로 오래 걸리고 이후는 빠르다 |
| 그 외 | 커스텀 프로바이더 | `atlas debug-adapter add <dir|zip|.vsix>` — VS Code 확장의 어댑터 선언을 그대로 수용(예: Rust=CodeLLDB vsix). 챗에서 `atlas debug-adapter skill` 로 구성 에이전트의 도움을 받을 수 있다 |

## 8. 하단 도크 — 터미널 · 로그 · 디버그 콘솔 · 문제

- **터미널**: 통합 터미널(xterm) — 탭 여러 개, 프로젝트 폴더 기준. Run(터미널)
  activity 와 `console: integratedTerminal` 디버그가 여기서 실행된다. 터미널 포커스
  중 `Ctrl+R` 은 셸(reverse-i-search)로 전달된다.
- **로그**: 앱·호스트 진단 로그(워크스페이스 로드, 에이전트 백엔드 진단 등).
- **디버그 콘솔**: 디버그 세션과 그래프 실행의 프로그램 출력 + REPL.
- **문제**: 실패 목록(챗 턴 실패·그래프 제출 실패 등) — 항목에서 상세 확인.

## 9. 그래프 편집기

Activity Bar 의 그래프 아이콘(플러그인 'graphs'). Atlas Forge 그래프 엔진의 문서를
시각 편집·실행·디버그한다. 그래프 파일은 `.atlas/graphs/<id>.json`.

- **엔진 준비**: 실행 시 로컬 엔진(atlas-engine)에 자동 접속하고, 없으면 설치본을
  자동 기동한다. 설치는 설정 ⚙ › Atlas 제품 › 그래프 엔진 카드(또는
  `atlas engine install`).
- **노드 클래스**: 실행 노드는 프로젝트 노드 클래스다 — 그래프 목록의 "새 노드 클래스"
  로 생성하면 Python 소스(`.atlas/graphs/src/…`)와 카탈로그(팔레트·입력 스키마·엔진
  바인딩)가 만들어진다. 노드의 **"코드" 버튼**으로 소스를 열어 고치면 다음 실행에 자동
  반영된다(엔진 등록 동기화 자동).
- **팔레트**: 좌측 드로어 — 클래스와 **프로젝트 그래프 목록**(드롭하면 서브네트워크로
  임베드)을 드래그해 배치한다.
- **실행**: 툴바 ▶ — 이벤트 스트림은 디버그 콘솔로, 노드 상태는 캔버스 하이라이트로.
  중단점(그래프 문서에 저장)은 super-step 경계에서 일시정지한다. 헤드리스는
  `atlas graph run <file>`.
- **노드 소스 디버그**: 툴바 🐞 — 소스에 중단점이 있는 클래스를 debugpy 로 붙여 실행
  (라이브). 노드 우클릭 **"재현 디버그"** 는 지난 실행에서 캡처한 입력으로 그 노드만
  재실행한다(엔진·선행 파이프라인 불필요).
- **Graph Agent**: 그래프 화면의 챗 — 그래프 구성·노드 클래스 생성·실행 진단을 돕는다.

## 10. 툴 패널 (🛠)

에이전트가 호출하는 도구 레지스트리(`tools/*.yaml|json`)의 관리 화면이다.

- **툴 종류**: `code`(실행 프로그램) / `form`(사용자 폼 — 제출값이 에이전트에게 복귀) /
  `render`(화면 — 챗 카드 또는 팝업). form·render 는 코드 없이 선언(FormSpec/RenderSpec)
  만으로 만든다 — 미리보기 제공.
- 제작은 **Tool Builder** 챗(툴 패널에서 열기)에게 맡기는 것이 기본 경로다.
- **MCP 서버 섹션**: 외부 MCP 서버 등록(스펙 JSON 붙여넣기)·삭제(§6 참조).

## 11. 플러그인

Activity Bar 의 **플러그인** 아이콘 — 설치·제작 관리 화면.

- **설치**: zip 설치/제거(반영은 **Studio 재시작** 버튼). 설치는 사용자 전역
  (`~/.atlas/plugins`)이다. 동봉 플러그인(그래프 편집기)은 "내장" 라벨로 제거 불가.
- **제작**: "새 플러그인" — 이름·설명만 넣으면 프로젝트 `.atlas/plugin-dev/<id>` 에
  골격이 생기고, **"시작 — 에이전트와 제작"** 이 저작 킷과 함께 브리핑 챗을 연다.
  빌드(esbuild) → `atlas plugin pack` → 설치 흐름. 명세는 plugins.md.

## 12. 설정 (⚙)

Activity Bar 하단 ⚙. 탭 구성:

| 탭 | 내용 |
|---|---|
| **프로젝트** | 프로젝트 등록 정보 · 작업자(산출물 author 표기) · 라이선스 요약 |
| **환경 설정** | 에이전트 백엔드(claude-code / atlas-agent) · Claude 계정 관리 · 기본 모델 · 표시 언어 등 전역 설정 — 자동 저장 |
| **Atlas 제품** | 설치·상태·실행 관리 — 그래프 엔진(설치/시작/설정 파일) · 에이전트 서버(프로젝트별 상태/시작/중지) · **디버그 어댑터**(java 등) · **VS Code 연동** 버튼 · **브라우저 연동** 버튼. 각 카드의 Config 팝업으로 설정 파일 직접 편집 |
| **라이센스** | 상태 확인 · 키 등록/변경(활성 상태에서도 가능) |

헤더의 ⟳ 는 전체 재조회. (개발자용 탭은 개발자 라이선스에서만 노출된다.)

### 표시 언어

한국어(기본)·English 내장. 언어 팩(json)을 `~/.atlas/locales/` 에 넣으면 재빌드 없이
추가된다 — 포맷은 atlas-studio.md §2 다국어 참조.

## 13. 클라이언트 연동

### VS Code

설정 ⚙ › Atlas 제품 › **"VS Code 연동"** 버튼 — 브리지 확장을 설치하고 같은 폴더로
VS Code 창을 연다. 이후 에이전트가 그 창의 태스크 실행·진단 조회·디버그 시작을
수행할 수 있다(`mcp__vscode__*`). 폴더가 VS Code 의 Workspace Trust 미신뢰 상태면
연동이 되지 않으니 신뢰로 전환한다. 브리지 구버전 안내가 뜨면 확장 업데이트 후
VS Code 에서 "Developer: Reload Window".

### 브라우저 확장

설정 ⚙ › **"브라우저 연동"** — 로드 폴더(`~/.atlas/browser-extension`)를 준비해 주며,
크롬 확장 페이지에서 "압축해제된 확장 프로그램 로드"로 1회 수동 등록한다(크롬 정책).
side panel 에서 프로젝트와 페어링하면 **Atlas Bridge Agent** 챗으로 접속한 탭의 화면
정찰·조작·화면 계약 저작을 수행한다. 상세: extensions.md.

## 14. CLI (`atlas`)

Studio 의 모든 동작은 동봉 CLI 로도 수행된다(에이전트도 같은 CLI 를 쓴다) —
`atlas --help` 로 전체 확인. 자주 쓰는 것:

```
atlas init --workbench <id>       프로젝트 초기화(위저드가 대신 해 준다)
atlas activity list|run <id>      Activity 조회·헤드리스 실행
atlas workbench export|pack|install   워크벤치 인스턴스 → 템플릿 배포
atlas tool list                   에이전트 툴 지형(내장 + 레지스트리)
atlas graph run <file>            그래프 헤드리스 실행
atlas engine install|status       그래프 엔진 설치·상태
atlas agent install|status        atlas-agent 서버 설치·상태
atlas debug-adapter install …     디버그 어댑터 설치(js-debug·java)
atlas mcp list|add|remove         외부 MCP 서버 관리
atlas wiki grep <키워드…>          프로젝트 위키·워크벤치 홈 검색
atlas accounts …                  Claude 계정 저장·전환
```

## 15. 단축키

전체 카탈로그는 [shortcuts.md](shortcuts.md). 핵심:

| 키 | 동작 |
|---|---|
| `Ctrl+Shift+P` / `Ctrl+P` | 명령 팔레트 / 파일 열기 |
| `Ctrl+N/O/S/W`, `Ctrl+Shift+S` | 파일 새로 만들기/열기/저장/닫기/다른 이름 저장 |
| `F5` · `Shift+F5` · `F10/F11` | 디버그 시작·중지·스텝 |
| `Ctrl+R` | 새로 고침 |
| `` Ctrl+` `` | 하단 패널 토글 |

## 16. 문제 해결

| 증상 | 확인 |
|---|---|
| 챗 턴이 실패한다 | 하단 **문제** 탭의 상세 + **로그** 탭의 백엔드 진단. atlas-agent 백엔드면 서버 로그 `~/.atlas/agent-server.log` |
| 그래프 실행이 안 된다 | 설정 ⚙ › 제품 › 그래프 엔진 카드에서 상태·설치 확인. 엔진 로그 `~/.atlas/engine.log`. 자동 기동을 껐다면(`ATLAS_ENGINE_AUTOSTART=0`) 해제 |
| Java 디버그가 안 붙는다 | 어댑터 설치 여부(설정 ⚙ 디버그 어댑터 카드)·JDK 17+·첫 기동은 프로젝트 임포트로 수십 초 소요(디버그 콘솔에 진행 표시). 로그 `~/.atlas/jdtls.log` |
| VS Code 연동이 안 된다 | 대상 폴더 Workspace Trust 신뢰 여부 · 브리지 확장 버전(안내 시 재설치 후 창 리로드) |
| 브라우저 확장이 설치가 안 된다 | 크롬은 외부 설치를 막는다 — 설정 ⚙ "브라우저 연동"으로 폴더 준비 후 확장 페이지에서 수동 로드(1회) |
| 세션 이력이 사라졌다 | 프로젝트를 옮겼어도 `.atlas/agent/chats` 가 있으면 자동 복원된다. 복원 불가한 세션은 새 대화로 시작된다(경고 로그) |
| UI 가 낡은 데이터를 보여준다 | `Ctrl+R`(활성 화면 새로 고침) 또는 프로젝트 메뉴 › 다시 읽기 |
