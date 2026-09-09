# atlas-dist

Atlas 제품군의 **배포 전용 채널**입니다. 소스는 private 리포(atlas-engine · atlas-agent · atlas-studio · atlas-react)에 있고,
각 리포의 GitHub Actions 가 태그 push 시 빌드 산출물만 이 리포의 [Releases](../../releases) 로 올립니다.
이 리포에는 코드가 없습니다 — Release asset 과 [사용자 문서](docs/)가 전부입니다.

버그 픽스가 어느 정도 되면 소스 리포들도 오픈 소스로 전환할 예정입니다.
이 레포들은 AI를 활용한 바이브 코딩으로 만들어지고 있습니다.

## Release 태그 규칙

제품마다 태그 프리픽스가 다릅니다. 하나의 리포에 네 제품의 릴리스가 섞여 있으니 태그로 구분하세요.

| 태그 | 제품 | 산출물 |
|---|---|---|
| `engine-v<ver>` | atlas-engine — Rust 그래프 런타임 | `atlas-engine-bundle-<ver>-<os>-<arch>.zip` (Windows) / `.tar.gz` (Linux) |
| `agent-v<ver>` | atlas-agent — Python gRPC 에이전트 | `atlas-agent-bundle-<ver>-<os>-<arch>-py<minor>.zip` / `.tar.gz` |
| `studio-v<ver>` | atlas-studio — 데스크톱 IDE | `atlas-studio_<ver>_x64-setup.exe` (+ `atlas-browser-bridge.zip`, `atlas-java-debug.tar.gz`) |
| `react-v<ver>` | atlas-react — UI 프레임워크 | `atlas-react-<ver>.tgz` — npm 패키지 tarball |

## 다운로드 / 설치

Releases 는 public 이라 인증 없이 브라우저·curl 로 바로 받을 수 있습니다. `gh` 를 쓴다면:

```sh
gh release download engine-v0.1.1 -R oranewbie/atlas-dist
```

### atlas-engine (설치 번들)

압축 해제 후 `install.ps1`(Windows) / `install.sh`(Linux) 실행.
**전제조건 없음** — 산출물이 네이티브 바이너리입니다 (Python 노드 워커를 쓸 때만 Python ≥ 3.10).
그래서 자산명에 py 태그가 붙지 않고 OS/arch 만 구분합니다.

### atlas-agent (설치 번들)

압축 해제 후 `install.ps1` / `install.sh` 실행. 인터넷·uv·pip 추가 설치가 필요 없는 오프라인 wheelhouse 설치입니다.
⚠️ **번들은 (OS, Python minor) 종속**입니다 — wheelhouse 가 grpcio·pandas·pyarrow 같은 네이티브 wheel 을 담기 때문에
파일명의 `-py3.12` 가 대상 머신의 Python minor 와 **정확히** 일치해야 합니다.
지원 대상은 3.10 / 3.11 / 3.12 이며, 현재 배포되는 자산은 py3.12 입니다.

### atlas-studio (데스크톱 앱)

Releases 에서 `atlas-studio_<ver>_x64-setup.exe`(NSIS 인스톨러)를 받아 실행합니다.
서명되지 않은 인스톨러라 Windows SmartScreen 경고가 뜰 수 있습니다 — "추가 정보 → 실행".

같은 릴리스의 `atlas-browser-bridge.zip`(브라우저 확장)과 `atlas-java-debug.tar.gz`(Java 디버그 어댑터)는
Studio 가 필요할 때 알아서 내려받는 자산입니다 — 직접 받을 필요는 없습니다.

### atlas-react (npm 패키지)

```sh
npm install https://github.com/oranewbie/atlas-dist/releases/download/react-v1.1.5/atlas-react-1.1.5.tgz
```

peerDependencies(MUI · AG Grid · chart.js 등)는 소비 앱이 공급합니다 — 패키지의 `package.json` 참고.

## Studio 가 이 릴리스를 자동 소비합니다

이 리포는 사람만 내려받는 곳이 아닙니다. Atlas Studio 가 런타임에 여기서 직접 자산을 받아 설치합니다:

| Studio 동작 | 소비하는 릴리스 | 설치 위치 |
|---|---|---|
| `atlas engine install` | `engine-v*` 최신 번들에서 바이너리만 추출 | `~/.atlas/engine` |
| `atlas agent install` | `agent-v*` 번들 전체 + 동봉 `install.py` 로 venv 생성 | `~/.atlas/atlas-agent` |
| 브라우저 확장 설치 | `studio-v*` 의 `atlas-browser-bridge.zip` | — |
| Java 디버그 어댑터 설치 | `studio-v*` 의 `atlas-java-debug.tar.gz` | — |
| 앱 자동 업데이트 | `studio-v*` 의 `-setup.exe` | — |

agent 설치 시 Studio 는 머신에 있는 Python 을 스캔해 py 태그가 맞는 자산을 고릅니다(최신 태그 우선).

⚠️ **태그 프리픽스와 자산명은 Studio 코드와의 계약입니다.** 워크플로에서 이름을 바꾸면
Studio 의 `sidecar/engine-install.ts`(`ENGINE_TAG_PREFIX`·`pickEngineAsset`),
`sidecar/agent-install.ts`(`AGENT_TAG_PREFIX`·`pickAgentAssets`·`pyTagFromBundleName`),
`sidecar/jdtls.ts`, `sidecar/browser-ext-install.ts` 도 함께 고쳐야 합니다 —
한쪽만 바꾸면 빌드는 통과하고 설치가 조용히 깨집니다.

## 문서

[docs/](docs/) 의 사용자 문서(소개서 · 용어집 · 튜토리얼 · 매뉴얼 · 단축키)는 atlas-studio 리포가 원본입니다.
릴리스 파이프라인과 무관하게 `node scripts/publish-docs.mjs` 가 이 리포로 직접 push 하므로,
**여기서 직접 수정하지 마세요** — 다음 발행 때 덮어써집니다. 원본은 atlas-studio 의 `docs/` 입니다.

## 배포 파이프라인 (관리자용)

각 소스 리포의 `.github/workflows/release.yml` 이 담당합니다:

1. 소스 리포에서 버전 태그 push: `git tag v0.2.0 && git push origin v0.2.0`
2. Actions 가 빌드 후 이 리포에 `<product>-v0.2.0` Release 를 생성/업데이트
3. 각 소스 리포에는 `DIST_TOKEN` secret 필요 — 이 리포에 **Contents: Read/Write** 권한을 가진 fine-grained PAT

**버전의 진실은 태그명이 아니라 리포 안의 버전 파일입니다** — 릴리스 태그는 이 파일들을 읽어 붙습니다.
태그를 밀기 전에 버전 파일을 먼저 올리세요.

| 제품 | 버전 파일 |
|---|---|
| engine · agent | `setup/VERSION` |
| studio | 루트 `package.json` |
| react | `package.json` |

Studio 설정 ⚙ > Atlas 개발자의 배포 버튼(내부적으로 `atlas engine release [ver]` / `atlas agent release [ver]`)이
버전 갱신 · 커밋 · 브랜치 push · 태그 push 를 한 번에 처리합니다. 로컬 빌드는 없고 CI 가 빌드합니다.

각 릴리스는 OS(× Python) 매트릭스 잡이 **같은 태그에 자산을 덧붙이는** 구조입니다 —
Windows 잡의 `.zip` 과 Linux 잡의 `.tar.gz` 가 한 릴리스에 모입니다.
