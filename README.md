# Atlas

**사람과 AI 가 한 프로젝트 안에서 같은 방법으로 함께 일하게 해 주는 도구 모음** — Atlas 제품군의 공개 배포 창구예요.

> 지금은 **프리뷰(0.x)** 단계예요. **2026년 12월 정식 오픈**과 함께 소스 코드를 공개할 예정이고,
> 그 전까지 이 저장소에는 **설치 파일([Releases](../../releases))과 사용자 문서([docs](docs/README.md))** 만 있어요.
> 소스(atlas-studio · atlas-agent · atlas-engine · atlas-react)는 private 저장소에 있고, AI 를 활용한 바이브 코딩으로 만들어지고 있어요.

## 제품

| 제품 | 한 줄 소개 | 최신 | 문서 |
|---|---|---|---|
| **Atlas Studio** | 일하는 방법을 담아 AI 와 함께 수행하는 데스크톱 작업대 — 문서·보고서·전환 프로젝트·웹 테스트·개발 | v0.1.29 | [시작하기](docs/studio/README.md) |
| **atlas-agent** | 파일·명령·웹·데이터 도구를 쓰는 AI 에이전트 서버 (Claude·OpenAI·Azure·사내 모델) | v0.1.4 | [시작하기](docs/agent/README.md) |
| **atlas-engine** | 노드와 선으로 이은 그래프를 실행하는 엔진 (Rust, gRPC·MCP) | v0.1.3 | [시작하기](docs/engine/README.md) |
| **atlas-react** | 조회·편집·저장형 업무 화면을 만드는 React UI 프레임워크 | v1.1.5 | [시작하기](docs/react/README.md) |

처음이라면 → **[Atlas Studio 소개](docs/studio/README.md)** · **[활용 사례](docs/studio/use-cases.md)** · **[30분 따라하기](docs/studio/tutorial.md)**

## 다운로드

[Releases](../../releases) 에서 제품별 태그를 찾으세요. 한 저장소에 네 제품의 릴리스가 섞여 있으니 **태그로 구분**해요.
공개 저장소라 로그인 없이 브라우저·curl 로 받을 수 있어요.

| 태그 | 제품 | 파일 |
|---|---|---|
| `studio-v<버전>` | Atlas Studio | `atlas-studio_<버전>_x64-setup.exe` (Windows 설치 파일) · `atlas-browser-bridge.zip` (브라우저 확장) · `atlas-java-debug.tar.gz` (Java 디버그 어댑터) |
| `agent-v<버전>` | atlas-agent | `atlas-agent-bundle-<버전>-<os>-<arch>-py<minor>.zip` (Windows) / `.tar.gz` (Linux) |
| `engine-v<버전>` | atlas-engine | `atlas-engine-bundle-<버전>-<os>-<arch>.zip` (Windows) / `.tar.gz` (Linux) |
| `react-v<버전>` | atlas-react | `atlas-react-<버전>.tgz` (npm 패키지) |

> 예전 `forge-v*` 태그(atlas-forge)는 atlas-agent 와 atlas-engine 으로 나뉘어 더 이상 나오지 않아요.

### Atlas Studio

`atlas-studio_<버전>_x64-setup.exe` 를 받아 실행하세요. 필요한 런타임이 모두 들어 있어요.
서명되지 않은 설치 파일이라 SmartScreen 경고가 뜨면 **추가 정보 → 실행**. 이후 업데이트는 앱이 알려 줘요.
같은 릴리스의 브라우저 확장·Java 디버그 어댑터, 그리고 엔진·에이전트는 Studio 가 필요할 때 알아서 받아요 — 직접 받을 필요는 없어요.

### atlas-agent / atlas-engine

```sh
gh release download agent-v0.1.4  -R oranewbie/atlas-dist
gh release download engine-v0.1.3 -R oranewbie/atlas-dist
```

압축을 풀고 `install.ps1`(Windows) / `install.sh`(Linux) 를 실행해요.

- **atlas-agent** 는 인터넷·uv·pip 없이 설치되는 오프라인 번들이에요. 번들 안에 grpcio·pandas·pyarrow 같은 네이티브 패키지가
  들어 있어서 **(OS, Python 버전)에 묶여요** — 파일명의 `py3.12` 와 PC 의 Python 버전이 **정확히** 같아야 해요
  (지원 대상은 3.10 / 3.11 / 3.12, 현재 배포 파일은 3.12).
- **atlas-engine** 은 실행 파일이라 따로 필요한 게 없어요(Python 노드를 쓸 때만 Python 3.10+). 그래서 파일명에 Python 버전이 없어요.

자세한 내용: [atlas-agent 설치](docs/agent/README.md#설치) · [atlas-engine 설치](docs/engine/README.md#설치)

### atlas-react

```sh
npm install https://github.com/oranewbie/atlas-dist/releases/download/react-v1.1.5/atlas-react-1.1.5.tgz
```

MUI · AG Grid · chart.js 등 함께 쓰는 패키지(peerDependencies)는 앱이 설치해요 → [설치 안내](docs/react/README.md#설치)

## 문서

**[docs/README.md](docs/README.md)** 에서 전체 문서를 볼 수 있어요. 버전별 변화는 [변경 이력](docs/changelog.md).

---

## 관리자용

### Studio 가 이 릴리스를 직접 받아 써요

이 저장소는 사람만 내려받는 곳이 아니에요. Atlas Studio 가 실행 중에 여기서 파일을 받아 설치해요.

| Studio 동작 | 받는 릴리스 | 설치 위치 |
|---|---|---|
| `atlas engine install` | `engine-v*` 최신 번들에서 실행 파일만 | `~/.atlas/engine` |
| `atlas agent install` | `agent-v*` 번들 전체 + 동봉 `install.py` 로 venv 생성 (PC 의 Python 에 맞는 py 태그, 최신 태그 우선) | `~/.atlas/atlas-agent` |
| 브라우저 확장 설치 | `studio-v*` 의 `atlas-browser-bridge.zip` | — |
| Java 디버그 어댑터 설치 | `studio-v*` 의 `atlas-java-debug.tar.gz` | — |
| 앱 자동 업데이트 | `studio-v*` 의 `-setup.exe` | — |

⚠️ **태그 앞부분과 파일 이름은 Studio 코드와의 약속이에요.** 워크플로에서 이름을 바꾸면 atlas-studio 의
`sidecar/engine-install.ts`(`ENGINE_TAG_PREFIX`·`pickEngineAsset`), `sidecar/agent-install.ts`(`AGENT_TAG_PREFIX`·`pickAgentAssets`·`pyTagFromBundleName`),
`sidecar/jdtls.ts`, `sidecar/browser-ext-install.ts` 도 함께 고쳐야 해요 — 한쪽만 바꾸면 빌드는 통과하고 설치가 조용히 깨져요.

### 배포 파이프라인

각 소스 저장소(private)의 `.github/workflows/release.yml` 이 담당해요.

1. 소스 저장소에서 버전 파일을 올린 뒤 `v<버전>` 태그를 push 해요 (`git tag v0.2.0 && git push origin v0.2.0`).
2. Actions 가 빌드해서 이 저장소에 `<제품>-v<버전>` Release 를 만들어요.
3. 각 소스 저장소에는 `DIST_TOKEN` secret 이 필요해요 — 이 저장소에 **Contents: Read/Write** 권한이 있는 fine-grained PAT.

**버전의 기준은 태그 이름이 아니라 저장소 안의 버전 파일**이에요. 태그를 밀기 전에 버전 파일을 먼저 올리세요.

| 제품 | 버전 파일 |
|---|---|
| engine · agent | `setup/VERSION` |
| studio | 루트 `package.json` |
| react | `package.json` |

Studio 설정 ⚙ › Atlas 개발자의 배포 버튼(내부적으로 `atlas engine|agent|studio release [ver]`)이 버전 갱신 · 커밋 · 브랜치 push · 태그 push 를
한 번에 처리해요. 로컬 빌드는 없고 CI 가 빌드해요. 릴리스는 OS(× Python) 매트릭스 잡들이 **같은 태그에 파일을 덧붙이는** 구조라,
Windows 잡의 `.zip` 과 Linux 잡의 `.tar.gz` 가 한 릴리스에 모여요.

### 문서 관리

`docs/` 는 **이 저장소가 원본**이에요(네 제품 공통 공개 문서). 예전에는 atlas-studio 의 `scripts/publish-docs.mjs` 가
Studio 문서를 `docs/` 바로 아래로 덮어썼는데, 지금 구조(`docs/studio/` 등)와 맞지 않으니 **그 스크립트로 발행하지 마세요**
(쓰려면 대상 경로를 `docs/studio/` 로 바꿔야 해요). Studio 앱 안의 도움말 문서는 atlas-studio 의 `docs/` 를 그대로 동봉해요.
