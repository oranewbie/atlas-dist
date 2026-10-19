# atlas-react — 업무 화면 UI 프레임워크

> 설치 가능한 최신 버전 **v1.1.5** · 다운로드: [Releases](https://github.com/oranewbie/atlas-dist/releases) 의 `react-v*` 태그
>
> 이어서 읽기: [핵심 개념](concepts.md) · [컴포넌트 안내](components.md)

## 한 줄로 말하면

**"조회 → 표로 보기 → 고치기 → 저장" 같은 업무 화면을 빨리, 일정한 모양으로 만들게 해 주는 React 부품 상자"** 예요.

회사 업무 시스템의 화면은 대부분 비슷하게 생겼어요. 위에 검색 조건이 있고, 버튼 줄이 있고, 아래에
큰 표(그리드)가 있죠. 왼쪽엔 메뉴가 있고 위엔 여러 화면을 탭으로 열어 둬요. atlas-react 는 이런 화면의
**뼈대·부품·규칙**을 미리 만들어 둔 프레임워크예요.

- **앱 셸**: 로그인 → 메뉴 → 여러 화면을 탭으로 열어 두는 MDI 화면이 기본으로 들어 있어요.
- **표준 화면 골격**: `검색 영역 + 버튼 영역 + 결과 영역` 을 태그 몇 개로 짜요.
- **데이터 그리드**: 컬럼 정의만 쓰면 표가 생기고, 바뀐 행 추적·검증·엑셀 내보내기/가져오기가 따라와요.
- **서버 없이도 돌아가요**: 메뉴·언어팩·데모 데이터가 들어 있어서 백엔드가 준비되기 전에도 화면을 만들 수 있어요.
- **AI 가 화면을 쓸 수 있어요**: 화면에 등록한 기능("조회", "저장")이 사람 버튼으로도, AI 에이전트의 도구로도 똑같이 노출돼요.

## 이런 데 써요

| 화면 종류 | 예 |
|---|---|
| 기준정보 관리 | 사용자·코드·품목 관리 — 조회하고 표에서 바로 고쳐 저장 |
| 마스터–디테일 | 위 표에서 고르면 아래 표에 상세가 뜨는 화면 |
| 계획 입력 | 월별 수량을 가로로 펼쳐 입력하는 피벗 표 |
| 대시보드 | 숫자 타일 + 차트를 끌어다 배치하는 화면 |
| 프로세스 보기 | 흐름도, 칸반 보드, 간트 차트 |

## 설치

### 1. 패키지 설치

npm 공개 저장소가 아니라 **Releases 의 파일 주소**로 설치해요.

```sh
npm install https://github.com/oranewbie/atlas-dist/releases/download/react-v1.1.5/atlas-react-1.1.5.tgz
```

### 2. 함께 설치할 패키지 (peerDependencies)

atlas-react 는 MUI·AG Grid 같은 라이브러리를 **내 앱이 설치한 것**을 함께 써요. 아래 패키지를 설치해 주세요.

| 분야 | 패키지 |
|---|---|
| React | `react`, `react-dom` — **18.3.1 이상 또는 19** |
| 디자인 | `@mui/material` ^9, `@mui/icons-material` ^9, `@mui/x-date-pickers` ^9, `@emotion/react`, `@emotion/styled` |
| 그리드 | `ag-grid-community` ^36, `ag-grid-react` ^36 |
| 차트 | `chart.js` ^4, `react-chartjs-2` ^5 |
| 편집기 | `@tiptap/react`, `@tiptap/starter-kit`, `@tiptap/pm`, `@tiptap/extension-image`, `@tiptap/extension-placeholder` (^3) |
| 다이어그램·배치 | `@xyflow/react` ^12, `react-grid-layout` ^2, `react-split` ^2 |
| 폼·라우팅·상태 | `react-hook-form` ^7, `react-router-dom` ^7, `zustand` ^5 |
| 다국어·날짜 | `i18next`, `react-i18next`, `dayjs` |
| 엑셀 | `xlsx` — `https://cdn.sheetjs.com/xlsx-0.20.3/xlsx-0.20.3.tgz` 권장 (npm 판은 오래됐어요) |

정확한 버전 범위는 설치된 패키지의 `package.json` 을 보세요.

### 3. 필요한 부분만 쓰기 (부분 채택)

앱 셸까지 다 쓰지 않고 **화면 부품만** 쓰고 싶다면 하위 경로로 가져오세요. 필요한 의존성도 줄어들어요.

| 가져오는 경로 | 들어 있는 것 |
|---|---|
| `atlas-react` | 전부 (앱 셸 + 화면 부품 + 자동화) |
| `atlas-react/page` | 화면 골격(PageContent·SearchArea·ButtonArea·ResultArea·SplitPanel·FormArea…), 입력 필드, 버튼, 팝업, 로딩 표시 |
| `atlas-react/style` | 테마 (`buildTheme`, `applyThemeVariables`, `getThemeTokens`) |
| `atlas-react/lang` | 다국어 (`initI18n`, `transLangKey`) |

`atlas-react/page` 에는 DataGrid·차트·날짜 입력·파일 업로드가 **들어 있지 않아요**. 그런 부품이 필요하면 `atlas-react` 에서 가져오세요.
그 밖의 내부 경로(`atlas-react/src/...` 등)는 지원하지 않아요.

## 시작하기

### 1단계 — 화면 파일 등록과 시작

```jsx
// main.jsx
import ReactDOM from 'react-dom/client'
import { registerViews, configureAtlas, restProviders } from 'atlas-react'
import App from './App'

// 화면 파일들을 등록해요. 메뉴의 filePath('/sales/OrderList')가 이 폴더 기준 경로예요.
registerViews(import.meta.glob('/src/view/**/*.jsx'))

// 백엔드와 연결해요. 이 줄을 빼면 서버 없이(standalone) 데모 데이터로 돌아가요.
configureAtlas({ providers: restProviders({ baseURL: '/api' }) })

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

### 2단계 — 앱 껍데기

```jsx
// App.jsx
import { useEffect, useMemo } from 'react'
import { HashRouter } from 'react-router-dom'
import { ThemeProvider } from '@mui/material/styles'
import CssBaseline from '@mui/material/CssBaseline'
import { LocalizationProvider } from '@mui/x-date-pickers/LocalizationProvider'
import { AdapterDayjs } from '@mui/x-date-pickers/AdapterDayjs'
import {
  MainLayout, MessageSnackbar, ConfirmDialog, buildTheme, applyThemeVariables,
  initI18n, installBridge, getProvider,
  useThemeStore, useUserStore, useMenuStore, useContentStore,
} from 'atlas-react'
import Login from './Login'   // 로그인 화면은 앱이 직접 만들어요

export default function App() {
  const mode = useThemeStore((s) => s.mode)
  const themeName = useThemeStore((s) => s.themeName)
  const isLogin = useUserStore((s) => s.isLogin)
  const locale = useContentStore((s) => s.locale)
  const initMenu = useMenuStore((s) => s.initMenu)
  const theme = useMemo(() => buildTheme(mode, themeName), [mode, themeName])

  useEffect(() => { applyThemeVariables(mode, themeName) }, [mode, themeName])
  useEffect(() => { installBridge() }, [])            // AI 자동화 연결 (개발 모드 기본 켬)
  useEffect(() => {
    getProvider('i18n').getLangPack(locale)
      .then((p) => initI18n(locale, p ?? undefined)).catch(() => initI18n(locale))
  }, [locale])
  useEffect(() => {
    getProvider('menu').getMenus().then((menus) => {
      initMenu(menus)
      useContentStore.getState().restoreTabs()        // 열어 두었던 탭 복원
    })
  }, [initMenu])

  return (
    <ThemeProvider theme={theme}>
      <LocalizationProvider dateAdapter={AdapterDayjs} adapterLocale={locale}>
        <CssBaseline />
        <HashRouter>{isLogin ? <MainLayout /> : <Login />}</HashRouter>
        <MessageSnackbar />
        <ConfirmDialog />
      </LocalizationProvider>
    </ThemeProvider>
  )
}
```

- 로그인 화면에서는 `useUserStore` 의 `login(정보)` 를 부르면 돼요. 서버 없는 모드에서는 아무 아이디나 통과해요.
- `MainLayout` 은 라우터 안에 있어야 해요. 화면 주소는 `#/view/{화면ID}` 모양이에요.

### 3단계 — 첫 화면

```jsx
// src/view/item/ItemView.jsx
import { useRef } from 'react'
import { PageContent, SearchArea, ButtonArea, LeftButtonArea, RightButtonArea, ResultArea,
  DataGrid, GridAddRowButton, GridSaveButton, SearchButton, InputField,
  apiClient, useViewActions } from 'atlas-react'

const columns = [
  { name: 'itemCd', headerText: 'ITEM_CD', dataType: 'text', width: 140, validRules: [{ criteria: 'required' }] },
  { name: 'itemNm', headerText: 'ITEM_NM', dataType: 'text', editable: true },
]

export default function ItemView({ viewId }) {        // viewId 는 프레임워크가 넣어 줘요
  const grid = useRef(null)

  const onSearch = async () =>
    grid.current.setRows((await apiClient.post('item/search', {})).data.RESULT_DATA)

  const onSave = async () => {
    if (grid.current.validate().length) return        // 필수값 검사
    await apiClient.post('item/save', grid.current.getChangedRows())  // 바뀐 행만 보내요
    grid.current.clearRowStates()
  }

  // 툴바 버튼과 AI 도구로 함께 노출돼요
  useViewActions(viewId, {
    search: { standard: 'search', labelKey: 'SEARCH', description: '품목 조회', run: onSearch },
    save:   { standard: 'save',   labelKey: 'SAVE',   run: onSave },
  })

  return (
    <PageContent viewId={viewId}>
      <SearchArea buttons={<SearchButton onClick={onSearch} />}>
        <InputField langKey="ITEM_NM" />
      </SearchArea>
      <ButtonArea>
        <LeftButtonArea><GridAddRowButton grid={grid} /></LeftButtonArea>
        <RightButtonArea><GridSaveButton onSave={onSave} /></RightButtonArea>
      </ButtonArea>
      <ResultArea>
        <DataGrid ref={grid} id="itemGrid" viewId={viewId} items={columns} />
      </ResultArea>
    </PageContent>
  )
}
```

메뉴에 `{ id: 'ITEM_MGMT', name: 'ITEM_MGMT', filePath: '/item/ItemView' }` 항목을 넣으면 이 화면이 탭으로 열려요.
`headerText`·`langKey` 에 쓴 `ITEM_CD` 같은 키는 언어팩으로 자동 번역돼요(번역이 없으면 키가 그대로 보여요).

## Atlas Studio·에이전트와의 관계

- atlas-react 로 만든 화면은 **Atlas Studio 브라우저 확장**과 바로 연결돼요. 화면이 등록한 기능을 Studio 의
  AI 에이전트(Atlas Bridge Agent)가 읽고 실행할 수 있어서, "이 화면에서 조회해 보고 결과 확인해 줘" 같은
  화면 테스트를 맡길 수 있어요. 자세한 건 [핵심 개념 › 자동화](concepts.md#7-ai-자동화--화면을-도구로) 를 보세요.
- Atlas Studio 의 일부 화면도 atlas-react 로 만들어졌어요.

## 버전 이야기

| 버전 | 상태 | 주요 변화 |
|---|---|---|
| **1.1.5** | **설치 가능 (최신 배포)** | React 18 지원, 부분 채택 경로(`/page`·`/style`·`/lang`) |
| 1.1.4 | 1.1.5 에 포함 | ViewHarness — 모든 화면·팝업이 AI 에게 보이도록 등록, 빈 화면 표시 |
| 1.1.6 · 1.1.7 | 다음 배포 예정 | 직접 셸을 조립하는 부품 공개(`ViewContainer`·`RouterSync`·`ViewToolbar`·`SettingDialog`·`getViewPath`), 화면 골격 중첩 방지 |
| 그 이후 | 실험 중 | 대용량 그리드 엔진(브라우저 내 DuckDB)·집계·피벗 — 아직 공개 API 아님 |

## 아직 없는 것

솔직하게 적어 둘게요.

- 그리드는 AG Grid **무료판(Community)** 기반이라 셀 범위 선택·행 그룹핑·트리 데이터·합계 줄·셀 병합이 없어요.
- REST 연결부(`restProviders`)는 실제 서버 검증이 아직 진행 중이에요.
- 탭 바·사이드 메뉴 같은 셸의 세부 부품은 따로 꺼내 쓸 수 없어요(`MainLayout` 통째로 쓰거나, 다음 버전의 조립 부품으로 직접 만들어요).
- 로그인 화면은 제공하지 않아요. 앱이 직접 만들어요.
- 내 앱에서 새 테마 프리셋을 등록하는 공개 방법은 아직 없어요.

## 문서 더 보기

| 문서 | 내용 |
|---|---|
| [핵심 개념](concepts.md) | 프로바이더(백엔드 연결), 화면 등록, 탭(MDI), 상태, 테마, 다국어, AI 자동화 |
| [컴포넌트 안내](components.md) | 화면 골격·입력·그리드·피벗·차트·대시보드 등 부품 목록과 쓰는 법 |
