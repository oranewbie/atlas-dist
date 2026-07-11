# atlas-dist

Atlas 제품군의 **배포 전용 채널**입니다. 소스는 private 리포(atlas-forge · atlas-react · atlas-studio)에 있고,
각 리포의 GitHub Actions 가 태그 push 시 빌드 산출물만 이 리포의 [Releases](../../releases) 로 올립니다.
이 리포에는 코드가 없습니다 — Release asset 이 전부입니다.

## Release 태그 규칙

| 태그 | 제품 | 산출물 |
|---|---|---|
| `forge-v<ver>` | atlas-forge (엔진 + 에이전트) | `atlas-bundle-<ver>-<os>-<arch>-py<minor>.zip` (Windows) / `.tar.gz` (Linux) — 오프라인 설치 번들 |
| `react-v<ver>` | atlas-react (UI 프레임워크) | `atlas-react-<ver>.tgz` — npm 패키지 tarball |
| `studio-v<ver>` | atlas-studio (데스크톱 IDE) | Windows 인스톨러 (`.msi`, NSIS `.exe`) |

## 다운로드 / 설치

### atlas-forge (설치 번들)

```sh
gh release download forge-v0.1.0 -R oranewbie/atlas-dist
# 또는 브라우저/curl 로 Releases 페이지에서 직접 다운로드 (public — 인증 불필요)
```

압축 해제 후 번들 안의 `install.ps1`(Windows) / `install.sh`(Linux) 실행.
대상 머신 전제조건은 Python ≥ 3.10 뿐입니다 (venv + wheelhouse 오프라인 설치).
⚠️ 번들은 (OS, Python minor) 종속 — 파일명의 `-py3.12` 등이 대상 머신 Python 과 일치해야 합니다.

### atlas-react (npm 패키지)

```sh
npm install https://github.com/oranewbie/atlas-dist/releases/download/react-v1.1.2/atlas-react-1.1.2.tgz
```

peerDependencies(MUI · AG Grid · chart.js 등)는 소비 앱이 공급합니다 — 패키지의 `package.json` 참고.

### atlas-studio (데스크톱 앱)

Releases 에서 `.msi` 또는 NSIS `.exe` 를 받아 실행합니다.
서명되지 않은 인스톨러라 Windows SmartScreen 경고가 뜰 수 있습니다 — "추가 정보 → 실행".

## 배포 파이프라인 (관리자용)

각 소스 리포의 `.github/workflows/release.yml` 이 담당합니다:

1. 소스 리포에서 버전 태그 push: `git tag v0.2.0 && git push origin v0.2.0`
2. Actions 가 빌드 후 이 리포에 `<product>-v0.2.0` Release 를 생성/업데이트
3. 각 소스 리포에는 `DIST_TOKEN` secret 필요 — 이 리포에 **Contents: Read/Write** 권한을 가진
   fine-grained PAT

버전의 진실: forge = `setup/VERSION`, react = `package.json`, studio = 루트 `package.json`
(태그명이 아니라 이 파일들 기준으로 Release 태그가 정해집니다 — 태그 push 전에 버전 파일을 먼저 올리세요).
