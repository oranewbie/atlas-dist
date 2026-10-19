# atlas-react 핵심 개념

> [atlas-react 소개](README.md) 의 "시작하기"를 한 번 따라 해 본 뒤 읽으면 이해가 빨라요.
> 모든 import 는 `'atlas-react'` 에서 해요.

## 1. 전체 그림

```
내 앱
├─ main.jsx      registerViews(...)   ← 화면 파일 등록
│                configureAtlas(...)  ← 백엔드 연결(프로바이더)
└─ App.jsx       MainLayout           ← 메뉴 + 탭 + 화면 영역 (앱 셸)
                   └─ 화면 (예: ItemView)
                        PageContent → SearchArea / ButtonArea / ResultArea
                        useViewActions(...)  ← "이 화면이 할 수 있는 일" 등록
```

이 문서는 위 그림의 각 부분을 차례로 설명해요.

## 2. 프로바이더 — 백엔드와 연결하는 콘센트

로그인, 메뉴, 권한, 언어팩, 코드 목록, 파일 업로드처럼 **서버가 필요한 기능**은 모두 "프로바이더"라는
교체 가능한 부품 뒤에 숨어 있어요. 화면 코드는 프로바이더만 부르고, 그 뒤에 진짜 서버가 있는지
가짜 데이터가 있는지는 몰라요. 그래서 **서버 없이 먼저 화면을 만들고, 나중에 서버를 꽂을 수 있어요.**

| 프로바이더 | 하는 일 |
|---|---|
| `auth` | 로그인·로그아웃·사용자 정보·세션 확인 |
| `menu` | 메뉴 목록·즐겨찾기 |
| `permission` | 화면별 권한 (`READ`·`UPDATE`·`DELETE`) |
| `i18n` | 언어팩 |
| `code` | 공통 코드 목록 (`[{value, label}]`) |
| `settings` | 앱 설정 |
| `file` *(선택)* | 파일 업로드·다운로드·삭제 |
| `preference` *(선택)* | 그리드 레이아웃 저장 |
| `dashboard`, `flow` *(선택)* | 대시보드·흐름도 배치 저장 |
| `events` *(선택)* | 서버 이벤트(SSE) 구독 |
| `scaffold` *(선택)* | 아직 내용이 없는 화면에 보여 줄 안내 화면 |

```js
import { configureAtlas, standaloneProviders, restProviders } from 'atlas-react'

// 서버 없이 — 기본값이라 생략해도 같아요
configureAtlas({ providers: standaloneProviders() })

// REST 서버와 연결 (엔드포인트 이름을 바꿀 수도 있어요)
configureAtlas({ providers: restProviders({ baseURL: '/api', endpoints: { MENUS: 'api/v2/menus' } }) })

// 일부만 바꾸기 — 로그인만 우리 회사 방식으로
configureAtlas({ providers: { ...standaloneProviders(), auth: myAuthProvider } })
```

`configureAtlas` 는 **넘긴 것만** 바꾸고 나머지는 그대로 둬요. 코드에서 꺼내 쓸 땐 `getProvider('menu')`
(없으면 오류) 또는 `findProvider('file')`(없으면 `null`) 을 써요.

## 3. 화면 등록과 메뉴

화면 파일은 `registerViews` 로 한꺼번에 등록하고, 메뉴 데이터의 `filePath` 로 찾아가요.

```js
registerViews(import.meta.glob('/src/view/**/*.jsx'))
// 다른 폴더라면
registerViews(import.meta.glob('/src/views/**/*.jsx'), { prefix: '/src/views' })
```

메뉴 한 줄은 이렇게 생겼어요.

```js
{ id: 'ITEM_MGMT', parentId: 'MASTER', name: 'ITEM_MGMT', filePath: '/item/ItemView', items: [] }
```

- `id` 가 곧 **화면 ID(viewId)** 예요. 화면 컴포넌트는 `viewId` 를 prop 으로 받아요.
- `name` 은 번역 키예요.
- `filePath` 가 없으면(null) 폴더 메뉴예요.
- 화면 파일을 못 찾거나 불러오다 실패하면 "화면 없음" 안내가 대신 떠요.

## 4. 탭(MDI)과 주소

- 열어 둔 탭은 **화면을 떠나도 살아 있어요**(keep-alive). 다른 탭에 다녀와도 표의 스크롤·입력값이 그대로예요.
- 한 화면이 오류로 죽어도 그 탭에만 "다시 시도" 버튼이 뜨고 나머지 화면은 멀쩡해요.
- 주소창은 `#/view/{화면ID}` 를 따라가요. 링크로 바로 들어오기, 뒤로 가기가 돼요.
- 탭은 끌어서 순서를 바꾸고, 한꺼번에 닫고, 새 창으로 열 수 있어요. 저장 안 한 변경이 있으면 닫기 전에 물어봐요.
- 열어 둔 탭은 새로 고침해도 복원돼요(`restoreTabs`).

코드에서 탭을 다룰 땐 `useContentStore` 를 써요: `openView(메뉴)`, `activateView`, `closeView`.

## 5. 화면이 할 수 있는 일 등록 — View Registry

각 화면은 "나는 이런 일을 할 수 있어요"를 등록해요. 이 등록 하나로 **사람용 툴바 버튼**과
**AI 에이전트용 도구**가 함께 만들어져요.

```js
useViewActions(viewId, {
  search: { standard: 'search', labelKey: 'SEARCH', description: '품목 조회', run: onSearch },
  save:   { standard: 'save', labelKey: 'SAVE', run: onSave, disabled: () => !canUpdate },
  approve:{ labelKey: 'APPROVE', description: '선택 행 승인', params: {...}, run: onApprove },
})
```

| 필드 | 뜻 |
|---|---|
| `labelKey` | 버튼 이름(번역 키) |
| `run` | 실제로 할 일 |
| `standard` | `search` · `save` · `refresh` 중 하나면 표준 아이콘 버튼이 돼요 |
| `disabled` | 지금 못 하는 상황이면 `true` — 사람 버튼도 AI 호출도 같이 막혀요 |
| `description` | **이게 있어야 AI 도구로 노출돼요.** AI 가 읽는 설명이에요 |
| `params` | AI 가 넘길 인자의 모양 |

화면이 사라지면 등록도 자동으로 풀려요. 폼은 `useViewForm(viewId, formId, form, onSubmit)` 으로 등록하고,
DataGrid 는 `viewId` 와 `id` 만 주면 알아서 등록돼요.

권한은 `useViewRegistry` 의 `fetchPermissions(viewId)` 로 불러와 `hasPermission(viewId, PERMISSION_TYPE_UPDATE)` 로 확인해요.

## 6. 상태·테마·다국어

### 상태 저장소 (zustand)

| 저장소 | 담는 것 |
|---|---|
| `useUserStore` | 로그인 여부, 사용자 정보, `login`/`logout` |
| `useMenuStore` | 메뉴 트리, 즐겨찾기 |
| `useContentStore` | 열린 탭, 언어, 날짜 형식, 메뉴 위치(`side`·`top`·`float`) |
| `useViewRegistry` | 화면별 기능·권한 |
| `useThemeStore` | 밝은/어두운 모드, 테마 이름 |
| `useLoadingStore` | 로딩 표시 |

화면은 "지금 열린 화면이 뭐지?"를 전역 상태에서 추측하지 말고, 항상 **받은 `viewId`** 를 쓰는 게 규칙이에요.
그래야 탭이 여러 개 떠 있어도 헷갈리지 않아요.

### 테마

- 테마 프리셋 하나가 MUI 테마(`buildTheme`)와 CSS 변수(`--atlas-*`, `applyThemeVariables`)에 동시에 반영돼요.
- 내장 프리셋: `default`, `t3`, `atlasStudio`(Atlas Studio 와 같은 색).
- 밝은/어두운 모드를 바꾸면 그리드와 차트도 따라 바뀌어요.

### 다국어

- 화면에는 문장 대신 **번역 키**(`ITEM_NM`, `MSG_SAVED` 같은 대문자 키)를 써요.
  그리드 헤더, 입력 라벨, 버튼, 메뉴 이름, 서버 오류 키가 자동으로 번역돼요.
- 코드에서 직접 번역할 땐 `transLangKey('MSG_COUNT', { count: 3 })` — `{{count}}` 자리가 채워져요.
- 번역이 없으면 키가 그대로 보여요. 화면이 깨지지는 않아요.
- 언어는 설정 창(SettingDialog)이나 `useContentStore` 의 `setLocale` 로 바꿔요. 이미 그려진 그리드 헤더는
  새로 고침해야 바뀔 수 있어요.

## 7. AI 자동화 — 화면을 도구로

5장에서 등록한 기능은 브라우저의 `window.__ATLAS_BRIDGE__` 로도 열려요. AI 에이전트나 테스트 스크립트가
이걸 통해 화면을 열고, 기능을 실행하고, 표의 값을 읽고 쓸 수 있어요.

```js
installBridge()   // 개발 모드에선 기본으로 켜져요
// 운영 빌드에서 켜려면 먼저:
setAppSettings('automationBridge', true)
```

| 할 수 있는 일 | 예 |
|---|---|
| 화면 목록·열기·닫기 | `listViews()`, `openView(id)` |
| 화면 설명 보기 | `describeView(id)` — 등록된 기능·그리드·폼 |
| 기능 실행 | `invoke(viewId, 'search', args)` → `{ ok, result }` |
| 그리드 다루기 | `grid.getRows`, `grid.setCell`, `grid.selectRows`, `grid.addRow`, `grid.validate` |
| 폼 다루기 | `form.getValues`, `form.setValues`, `form.submit` |
| 시나리오 실행 | `scenario.run(steps)` — 단계별 실행 + 결과 확인(assert) |

AI 에게는 이것이 LLM 도구 정의 모양으로 보여요. 기본 도구 7종(`atlas_list_views`, `atlas_open_view`,
`atlas_describe_view`, `atlas_get_grid_rows`, `atlas_set_grid_cell`, `atlas_get_form_values`, `atlas_set_form_values`)과
화면별 도구 `view__{화면ID}__{기능}` 이에요.

**안전장치**: 화면이 등록한 것만 노출돼요. `disabled` 와 권한 검사가 AI 호출에도 똑같이 걸려요.

**Atlas Studio 와 함께 쓰면**: Studio 브라우저 확장의 Atlas Bridge Agent 가 이 다리를 통해 화면을 정찰하고
조작해요. "저장 버튼이 안 먹어요" 같은 문제를 화면·네트워크·콘솔을 함께 보며 진단할 수 있어요.

### ViewHarness — 빈 화면도 AI 에게 보이게

`PageContent` 와 (`componentId` 를 준) `PopupDialog` 는 자동으로 `ViewHarness` 로 감싸져서, 아직 기능을
하나도 등록하지 않은 화면이라도 "이런 화면이 있다"는 사실이 AI 에게 보여요. 내용이 비어 있으면
`scaffold` 프로바이더가 준 안내 화면을 대신 그려요.

## 8. 서버 호출 — apiClient

`apiClient` 는 axios 와 비슷하게 쓰는 호출 도구예요(`get`·`post`·`put`·`delete`).

- 현재 화면 ID(`menu-cd`)와 로그인 토큰을 자동으로 붙여요.
- 호출하는 동안 그 화면에 로딩 표시를 자동으로 띄워요(`waitOn` 으로 대상 지정, `false` 면 끔).
- 응답이 200 이어도 `errCode` 나 `success: false` 가 있으면 오류로 처리해요. 401 이면 로그아웃해요.
- 파일 업로드: `apiClient.upload(url, formData, { onUploadProgress })`
- 서버 이벤트: `connectSSE`, `useServerEvents`
- 기본 주소는 `/api`, 기본 제한 시간은 60초예요.
