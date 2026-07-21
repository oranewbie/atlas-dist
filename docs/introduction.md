# Atlas Studio 소개서

> Atlas Studio 리포에서 배포된 문서 사본입니다 (v0.1.5 기준) — 직접 수정하지 마세요.

> AI 와 사람이 **한 프로젝트 안에서 함께 일하는** 데스크톱 AI IDE / Workbench Host.
> 이 문서는 처음 접하는 분을 위한 소개서다. 사용법은 [사용자 매뉴얼](manual.md),
> 아키텍처 상세는 atlas-studio.md 참조.

## 1. Atlas Studio 란

Atlas Studio 는 **작업 방법론을 실행 가능한 형태로 담아 AI 에이전트와 함께 수행하는
데스크톱 IDE** 다. 코드 편집기·터미널·디버거 같은 IDE 의 기본기 위에, 프로젝트를
단계(Stage)와 작업(Activity)으로 구조화하고 각 화면마다 그 문맥을 아는 AI 에이전트가
붙는다. 에이전트는 화면을 보고(정찰), 파일과 CLI 로 일하고, 결과를 다시 화면으로
보여준다 — 사람은 방향을 정하고 게이트를 판정한다.

한 줄로: **"방법론이 실행되는 곳"**. 문서로만 존재하던 작업 절차(전환 방법론, 검증
파이프라인, 운영 체크리스트…)가 Stage 파이프라인 + Activity 스킬 + 도구 레지스트리로
프로젝트 안에 살아 움직이고, 잘 자란 구성은 템플릿으로 추출해 다음 프로젝트의 자산이 된다.

## 2. 왜 필요한가

- **AI 협업은 "채팅창"만으로 부족하다.** 실제 프로젝트는 단계가 있고, 산출물이 있고,
  통과 기준이 있다. Atlas Studio 는 이 구조(Stage·게이트·산출물 폴더)를 1급 개념으로
  제공하고, 에이전트가 그 구조를 읽고 쓰게 한다.
- **반복되는 작업은 자산이 되어야 한다.** 에이전트에게 매번 같은 지시를 반복하는 대신,
  절차를 Activity(skill 패키지)로, 도구를 툴 레지스트리로 굳힌다. 검증된 구성 전체는
  워크벤치 템플릿으로 배포된다.
- **서버 없이, 프로젝트 폴더가 진실.** 모든 상태는 프로젝트의 `.atlas/` 폴더(파일)에
  있다 — 별도 서버·DB 가 없고, 공유·협업은 쓰던 VCS(git/svn) 그대로다.

## 3. 핵심 개념

### 워크벤치 (Workbench)

도메인 작업 단위의 애플리케이션이다. 호스트(Studio)는 프로젝트 열기·라이선스·전역
설정만 담당하고, "무슨 일을 어떤 단계·도구로 하는가"는 전부 워크벤치가 소유한다.
워크벤치는 코드가 아니라 **선언(스키마·스킬·툴·가이드)의 묶음**으로 자라며,
빈 워크벤치에서 시작해 → 프로젝트에서 키우고 → 템플릿으로 추출(export)해 → 다른
프로젝트에 설치(install/apply)하는 생애주기를 가진다.

### Stage · Activity · Tool

```
프로젝트
└─ Stage            단계 — 목표와 게이트(체크리스트·판정)를 가진 탭
   ├─ Activity      실행 단위(= Task) — skill 패키지(절차서 + 리소스), run/eval/improve 분류
   ├─ Todo          사람 백로그
   └─ CheckList     통과 조건
Tool                에이전트가 호출하는 도구 — 실행 프로그램(code) 또는 폼/화면(form·render)
Hook                activity 전/후 자동 동작
```

운영 루프는 **run → eval(완성도 평가) → 사람 보강 → improve(가이드·도구 개선) →
게이트 → stage 판정**의 반복이다. 이 루프에서 나온 피드백이 다음 개선의 연료가 된다.

### 에이전트 협업

에이전트는 역할별로 나뉘어 같은 프로젝트 홈과 도구를 공유한다 — 한 에이전트가 만든
것이 다른 에이전트의 입력이 된다(상세: agents.md):

- **실행**: Atlas Agent(범용 기본) · Stage Agent(단계 책임 실행자) ·
  Atlas Bridge Agent(브라우저 화면 테스트) · Graph Agent(그래프 화면 전용)
- **Builder**: Tool Builder(도구 제작) · Skill Builder(절차 제작) ·
  Workbench Builder(방법론 설계·템플릿화)

에이전트 백엔드는 교체 가능하다 — 기본 claude-code(Claude Agent SDK), 또는
Atlas Forge 의 atlas-agent(gRPC).

## 4. 주요 기능

**IDE 기본기** — VS Code 의 부품(라이브러리·프로토콜)을 이식해 익숙한 UX 를 제공한다:
- 파일 탐색기 + Monaco 에디터, ripgrep 고속 검색, 파일 감시(자동 갱신)
- 통합 터미널(xterm + PTY, 프로젝트당 상주)
- 디버거(DAP) — Node/TS(js-debug)·Python(debugpy)·**Java(JDT LS 내장 경로)** + 커스텀
  어댑터 등록, 멀티 세션·조건부 중단점·디버그 콘솔·`launch.json` 임포트
- 명령 팔레트(Ctrl+Shift+P)·파일 열기(Ctrl+P)·VS Code 관례 단축키(F5 디버그 등 —
  [shortcuts.md](shortcuts.md))

**AI 세션** — 화면마다 문맥에 맞는 에이전트 챗 도크. 세션은 상주형이라 턴 사이에도
백그라운드 알림·권한 승인이 살아 있고, 화면 상태가 턴마다 함께 전달된다. 프로젝트
메모리(`.atlas/agent/memory`)·작업 위키(`.atlas/wiki`)·챗 아카이브(`.atlas/agent/chats`)로
지식이 프로젝트에 축적된다. 외부 MCP 서버(`.atlas/mcp.json`)도 연결할 수 있다.

**그래프 편집기** — Atlas Forge 그래프 엔진(atlas-engine)의 시각 편집·실행·디버그.
프로젝트 노드 클래스(Python 워커)를 만들고, 그래프를 저장·실행하고, 노드 소스에
중단점을 걸어 라이브/재현 디버그를 한다.

**확장성**
- **플러그인** — manifest + `activate(AtlasApi)` 2계층(VS Code 대응). Activity Bar
  화면·커맨드·키바인딩·패널 탭을 기여하고, zip 으로 배포한다(plugins.md)
- **VS Code 브리지 확장** — 에이전트가 같은 폴더의 VS Code 창을 제어(태스크·진단·디버그)
- **브라우저 확장** — side panel 챗 + 페이지 DOM 조작·화면 계약 저작(extensions.md)
- **interactive/render tool** — 코드 없이 선언(FormSpec/RenderSpec)만으로 에이전트가
  띄우는 폼·화면을 만든다

## 5. 제품군 구성

```
Atlas Forge (플랫폼 — 별도 리포)
├── atlas-engine    그래프 실행 엔진 (Rust)
└── atlas-agent     AI 에이전트 프레임워크 (Python gRPC)
atlas-react         UI 프레임워크 (별도 리포)
Atlas Studio (이 제품)
├── apps/studio             Tauri v2 데스크톱 호스트
├── packages/atlas-runtime  Node 사이드카 — headless 두뇌 + `atlas` CLI
├── packages/workbench-sdk  범용 셸·UI 계약 (WorkbenchShell)
├── packages/graph-editor   그래프 편집기 (플러그인 1호)
└── clients/                VS Code 확장 · 브라우저 확장
```

UI(Studio·확장)는 얇고, 로직은 atlas-runtime(사이드카)에 있다 — 같은 두뇌를 데스크톱·
브라우저·VS Code 세 클라이언트가 공유한다. Forge 의 엔진·에이전트 서버는 Studio 설정
화면에서 설치·기동까지 관리된다.

## 6. 설치와 라이선스

- **설치**: NSIS 인스톨러(`-setup.exe`) 하나로 끝 — Node 런타임과 사이드카가 동봉된
  자급자족 번들이다. 인앱 업데이트 알림(하단 상태바) 제공.
- **시작**: 첫 실행 시 사용자 등록(체험 30일) 또는 라이선스 키 등록.
- **프로젝트**: 아무 폴더나 열어 새 프로젝트 위저드로 시작한다 — 빈 워크벤치(스타터),
  방법론 인터뷰로 파이프라인을 설계해 주는 스타터+빌더, 또는 등록된 워크벤치 템플릿.

## 7. 더 읽을거리

| 문서 | 내용 |
|---|---|
| [manual.md](manual.md) | 사용자 매뉴얼 — 설치부터 화면별 사용법까지 |
| atlas-studio.md | 아키텍처·운영 모델(현재 실체) |
| agents.md | 에이전트 카탈로그 |
| tools.md | 툴 지형(내장 capability vs 프로젝트 레지스트리) |
| plugins.md | 플러그인 호스트 명세 |
| extensions.md | VS Code·브라우저 확장 |
| [shortcuts.md](shortcuts.md) | 단축키 카탈로그 |
