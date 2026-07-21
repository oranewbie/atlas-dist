# Atlas Studio 단축키

> Atlas Studio 리포에서 배포된 문서 사본입니다 (v0.1.5 기준) — 직접 수정하지 마세요.

Studio 셸의 키보드 단축키 카탈로그. VS Code 관례를 기본값으로 따른다(부품 이식 원칙과 동일).
표시 규칙: 타이틀바 메뉴(파일·프로젝트)와 명령 팔레트가 각 항목 옆에 단축키를 보여준다
(`Command.shortcut` 표시 힌트 + 키바인딩 레지스트리 역조회 — `shell/CommandPalette.tsx`),
Run & Debug 사이드바는 버튼 툴팁에 표기한다.

## 1. 전역(셸 빌트인)

`WorkbenchShell` 의 window capture keydown 이 소유하는 **예약 조합** — 플러그인 키바인딩보다
먼저 처리된다.

| 키 | 동작 |
|---|---|
| `Ctrl+Shift+P` | 명령 팔레트 |
| `Ctrl+P` | 파일 열기(quick open) |
| `` Ctrl+` `` | 하단 패널 접기/펼치기 |

mac 의 `Cmd` 는 `Ctrl` 로 취급한다. `F1` 은 Monaco 자체 팔레트와 충돌해 쓰지 않는다.

## 2. 파일(워크스페이스 서피스)

활성 메인 패널 컴포넌트가 `registerWorkspaceSurface` 로 **구현한 액션만** 동작하고
타이틀바 파일 메뉴·팔레트 '파일' 그룹에 나타난다. 등록이 없으면 이벤트를 삼키지 않아
Monaco 등 하위 핸들러가 받는다(진실: `shell/workspace-surface.ts` WORKSPACE_ACTION_META).

| 키 | 동작 |
|---|---|
| `Ctrl+N` | 새로 만들기 |
| `Ctrl+O` | 열기 |
| `Ctrl+S` | 저장 |
| `Ctrl+Shift+S` | 다른 이름으로 저장 |
| `Ctrl+W` | 닫기 |

## 3. 디버그(VS Code F5 패밀리)

`WorkbenchShell` 이 키바인딩 레지스트리에 등록하는 `shell.debug` 세트 — 디버거 가용
전송(Studio Tauri)에서만 등록된다(HTTP 전송(브라우저 확장)은 키·팔레트 항목이 아예 없다).
`when` 조건의 `debugStatus` 는 셸이 게시하는 컨텍스트 키다.

| 키 | 동작 | 조건(when) |
|---|---|---|
| `F5` | 디버그 시작/계속 — 포커스 세션이 정지 중이면 계속, 아니면 선택 실행 구성으로 새 세션 시작(+Run & Debug 뷰 전환) | 항상 |
| `Shift+F5` | 디버그 중지(포커스 세션) | 세션 활성(starting/running/stopped) |
| `Ctrl+Shift+F5` | 디버그 재시작(선택 구성) | 세션 활성 |
| `F6` | 일시정지 | running |
| `F10` | 한 단계 넘기기(step over) | stopped |
| `F11` | 단계 안으로(step into) | stopped |
| `Shift+F11` | 단계 밖으로(step out) | stopped |

- "선택 실행 구성"은 Run & Debug 사이드바 셀렉트의 선택(`debug-state.selectedConfig` 로
  동기화 — 사이드바가 안 떠 있어도 유지)이고, 미선택이면 `.atlas/launch.yaml` +
  `.vscode/launch.json` 병합 목록의 첫 구성이다. 구성이 하나도 없으면 F5 는 Run & Debug
  뷰를 열어 템플릿 생성을 안내한다(전역 로직: `shell/debug-actions.ts`).
- 중단점 토글은 에디터 거터 클릭(전용 키 없음 — Monaco 는 자체 단축키 체계).

## 4. 새로 고침

| 키 | 동작 |
|---|---|
| `Ctrl+R` | 활성 패널의 ⟳ 새로 고침(`setRefreshHandler` 등록분) — 등록이 없으면 워크스페이스 다시 읽기 폴백. 타이틀바 프로젝트 메뉴 '다시 읽기'와 같은 계열 |

- 호스트(`App.tsx`)가 처리하며 **WebView 전체 리로드를 막는다**(챗 세션·패널 상태 보호).
- **F5 는 새로 고침이 아니다**(2026-07-21 변경 — 디버그 시작으로 이관). 셸 밖 화면
  (게이트·위저드)에서는 F5 도 WebView 리로드 차단만 한다.
- **통합 터미널 포커스는 예외** — `Ctrl+R` 은 가로채지 않고 셸(reverse-i-search)로 보낸다.

## 5. 우선순위와 입력 보호

키 하나가 처리되는 순서(자세한 구현: `WorkbenchShell` capture keydown):

1. **셸 빌트인**(§1·§2 예약 조합) — 워크스페이스 액션은 등록된 것만 삼킨다.
2. **키바인딩 레지스트리**(`shell/keybindings.ts`) — `shell.debug` 세트 + 플러그인 기여.
   나중 등록이 우선(재정의 허용), `when` 불통과는 하위로 통과.
3. **하위 핸들러** — Monaco·xterm 등 컴포넌트 자체 단축키.
4. **호스트 폴백**(`App.tsx`) — `Ctrl+R` 새로 고침, F5/Ctrl+R 의 WebView 리로드 차단.

입력 보호: 수식어(`Ctrl`/`Alt`/`Meta`) 없는 조합은 input·textarea·contentEditable 포커스
중에는 발화하지 않는다 — **F-키는 예외**(F5 등은 어디서나 동작).

## 6. 확장(플러그인)

플러그인은 `api.registerKeybindings([{ key, command, when?, args? }])` 로 추가한다 —
커맨드 본체는 `registerCommands` 가 소유하고 키는 id 참조만 한다(VS Code
contributes.keybindings 대응). 예약 조합(§1·§2)은 빌트인이 먼저 삼키므로 재정의할 수
없다. `when` 문법과 컨텍스트 키 카탈로그는 `docs/plugins.md` 참조. 팔레트 표시 힌트는
키바인딩 등록만으로 자동으로 붙는다.
