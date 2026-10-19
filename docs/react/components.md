# atlas-react 컴포넌트 안내

> 모든 부품은 `'atlas-react'` 에서 가져와요. 부분 채택 경로(`atlas-react/page`)에 들어 있는 부품은 ⓟ 로 표시했어요.
> 처음이라면 [소개 › 시작하기](README.md#시작하기) 의 예제를 먼저 보세요.

## 1. 화면 골격

업무 화면은 대부분 이 모양이에요.

```
┌ PageContent ───────────────────────────────┐
│ SearchArea   [조건1] [조건2]       [조회]   │
│ ButtonArea   [행 추가] [삭제]      [저장]   │
│ ResultArea                                  │
│   ┌ DataGrid ─────────────────────────────┐ │
│   │                                       │ │
│   └───────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

| 부품 | 쓰임 |
|---|---|
| `PageContent` ⓟ | 화면의 뿌리. `viewId` 를 주면 로딩 표시·AI 연결(ViewHarness)이 자동으로 붙어요 |
| `SearchArea` · `SearchRow` ⓟ | 검색 조건 영역. `buttons` 에 조회 버튼을 넣어요 |
| `ButtonArea` · `LeftButtonArea` · `RightButtonArea` ⓟ | 버튼 줄 (왼쪽/오른쪽 정렬) |
| `ResultArea` ⓟ | 남은 공간을 채우는 결과 영역 |
| `StatusArea` ⓟ | 하단 상태 표시 |
| `SplitPanel` ⓟ | 영역을 좌우·상하로 나누고 경계를 끌어 크기 조절. 경계 더블클릭으로 접기 |
| `FormArea` · `FormRow` · `FormItem` · `GroupBox` ⓟ | 입력 폼 배치. 라벨 자동 번역, 필수 표시 |
| `TabContainer` ⓟ | 화면 안의 탭 (탭을 옮겨도 내용 유지) |
| `PopupDialog` ⓟ | 팝업. 기본으로 끌어 옮기기·크기 조절이 돼요. `componentId` 를 주면 AI 에게도 보여요 |

### 자주 쓰는 화면 패턴

| 패턴 | 모양 |
|---|---|
| 단순 그리드 | 검색 + 표 하나 |
| 마스터–디테일 | 위(또는 왼쪽) 표에서 고르면 아래 표에 상세 |
| 탭 그리드 | 탭마다 다른 표 |
| 차트 복합 | 숫자 타일 + 차트 + 표 |
| 피벗 입력 | 월·주 단위로 가로로 펼친 입력 표 |
| 팝업 | 코드 찾기, 상세 편집 |

## 2. 입력 필드

모두 `langKey`(번역 키) 또는 `label` 을 받고, react-hook-form 의 `control`·`name`·`rules` 를 지원해요.
`control` 없이 쓰면 일반 입력처럼 동작해요.

| 부품 | 쓰임 |
|---|---|
| `InputField` ⓟ | 글자 입력 |
| `SelectField` ⓟ | 하나 고르기 (`options`) |
| `MultiSelectField` ⓟ | 여러 개 고르기 (값은 배열, "전체" 토글 포함) |
| `AutocompleteField` ⓟ | 입력하며 고르기 |
| `CheckboxField` · `SwitchField` · `RadioGroupField` ⓟ | 체크·스위치·라디오 |
| `LookupField` | 코드 찾기 — 여러 열 목록에서 골라 **코드는 저장, 이름은 표시** (`valueField`, `displayField`, `columns`, `onSearch`) |
| `DateField` · `DateRangeField` · `TimeField` | 날짜·기간·시간 (dayjs) |
| `FileUploader` | 파일 첨부 — `uploadFiles()` 로 file 프로바이더를 통해 업로드 |

## 3. 버튼과 알림

| 부품 / 함수 | 쓰임 |
|---|---|
| `CommonButton` · `SearchButton` · `SaveButton` · `RefreshButton` ⓟ | 공통 버튼 (표준 아이콘·문구) |
| `showMessage(문구, 'success'\|'error'…)` | 화면 아래 짧은 알림 |
| `await showConfirm(문구, 제목)` | 예/아니요 확인 → `true`/`false` |
| `formatNumber` · `formatDate` · `formatDateTime` | 숫자·날짜 표시 형식 |
| `WaitPanel` · `WaitOverlay` · `ScopedWaitOverlay` ⓟ | 로딩 표시 |

## 4. DataGrid — 업무용 표

AG Grid(무료판) 위에 업무 화면에 필요한 규칙을 얹은 표예요. **컬럼 정의(`items`)만 쓰면** 표가 만들어져요.

```js
const items = [
  { name: 'itemCd', headerText: 'ITEM_CD', dataType: 'text', width: 140, validRules: [{ criteria: 'required' }] },
  { name: 'qty',    headerText: 'QTY',     dataType: 'number', editable: true, format: '#,##0' },
  { name: 'useYn',  headerText: 'USE_YN',  dataType: 'text', useDropdown: true, codes: 'USE_YN' },
  { name: 'period', headerText: 'PERIOD',  dataType: 'group', childs: [ /* 묶음 헤더 */ ] },
]
<DataGrid ref={grid} id="itemGrid" viewId={viewId} items={items} />
```

| 컬럼 속성 | 뜻 |
|---|---|
| `name` · `headerText` | 데이터 필드 · 헤더(번역 키) |
| `dataType` | `text` · `number` · `date` · `boolean` · `group`(묶음 헤더) |
| `editable` · `visible` · `sortable` · `width` | 편집 가능 · 보이기 · 정렬 · 폭 |
| `format` · `styleName` | 표시 형식 · 셀 스타일 |
| `useDropdown` + `values`/`labels` 또는 `codes` | 드롭다운 셀 (공통 코드 연결) |
| `validRules` | 검증 규칙 (예: 필수) |
| `agProps` | AG Grid 원래 옵션을 그대로 넘기기 |

### 표를 다루는 방법 (ref)

| 메서드 | 하는 일 |
|---|---|
| `setRows(rows)` · `getRows()` | 데이터 넣기 · 꺼내기 |
| `addRow()` · `deleteSelectedRows()` · `getSelectedRows()` | 행 추가 · 선택 삭제 · 선택 행 |
| `getChangedRows()` | **바뀐 것만** `{ added, updated, deleted }` 로 — 저장할 때 이걸 보내요 |
| `isUpdated()` · `clearRowStates()` | 변경 여부 · 저장 후 변경 표시 지우기 |
| `validate()` | 검증 — 오류 목록을 돌려줘요 |
| `saveLayout()` · `restoreLayout()` · `resetLayout()` | 사용자가 바꾼 열 배치 저장·복원 |
| `exportExcel()` | 엑셀(xlsx) 내보내기 |
| `getGridApi()` | AG Grid 원래 API |

### 그리드 버튼

`GridAddRowButton` · `GridDeleteRowButton` · `GridSaveButton` · `GridExcelExportButton` ·
`GridExcelImportButton`(내보낸 엑셀을 고쳐 다시 올리기) · `GridLayoutButton` · `GridCnt`(건수 표시)

**안 되는 것**: 셀 범위 선택, 행 그룹핑, 트리 데이터(유료판 기능), 합계 줄, 셀 병합.

## 5. CrossTabGrid — 피벗 표

세로로 길게 쌓인 데이터를 가로로 펼쳐 보여 줘요. 예: 품목×월 판매량.

```jsx
<CrossTabGrid
  rows={data}
  pivot={{ rowKeys: ['itemCd'], colKey: 'month', valueKeys: ['qty'], aggregate: 'sum' }}
  rowItems={[{ name: 'itemCd', headerText: 'ITEM_CD' }]}
/>
```

- 집계 방식: `sum` · `count` · `avg` · `min` · `max` · `first`
- 화면 없이 계산만 하려면 `pivotRows`, `buildCrossTabItems` 함수를 써요.
- 펼친 표에서 고친 값을 다시 세로 데이터로 되돌리는 기능은 아직 없어요.

## 6. 차트·숫자 타일·대시보드

| 부품 | 쓰임 |
|---|---|
| `Chart` | `type="bar\|line\|doughnut"`, `labels`, `datasets`. 테마 색을 따라가고, 쌓기(`stacked`)·가로(`horizontal`)·값 표시·가운데 글자·드릴다운 지원 |
| `StatTile` | 숫자 타일 — `labelKey`, `value`, `unit`, `delta`(증감) |
| `Dashboard` + `registerWidget` | 위젯을 끌어다 배치하는 대시보드. 편집 모드에서 추가·삭제, 배치는 dashboard 프로바이더에 저장 |

```js
registerWidget('salesTrend', { labelKey: 'SALES_TREND', defaultSize: { w: 6, h: 4 }, component: SalesTrend })
<Dashboard id="main" editable defaultItems={[{ type: 'salesTrend' }]} />
```

## 7. 그 밖의 부품

| 부품 | 쓰임 |
|---|---|
| `TreeList` (+ `toTree`, `moveTreeNode`) | 트리 목록 — 하위까지 같이 체크, 끌어서 옮기기 |
| `FlowDiagram` | 흐름도 (@xyflow/react). flow 프로바이더로 저장 |
| `KanbanBoard` | 칸반 보드 (끌어서 상태 바꾸기) |
| `GanttChart` | 간트 차트 — 일·주·월 눈금, 진행률, 오늘 선 |
| `RichTextEditor` | 서식 있는 글 편집기 (Tiptap). 이미지는 기본으로 문서 안에 넣고, `onUploadImage` 를 주면 업로드 |

## 8. 셸 조립 부품 (다음 배포 v1.1.6 부터)

`MainLayout` 을 통째로 쓰지 않고 내 앱만의 레이아웃을 짤 때 쓰는 부품이에요. **현재 배포본(1.1.5)에는 아직 없어요.**

| 부품 | 쓰임 |
|---|---|
| `ViewContainer` | 열린 화면들을 탭처럼 살려 두며 그려 주는 영역 (필수) |
| `RouterSync` | 주소창과 열린 화면을 맞춰 주는 부품 (라우터 안에 하나, 필수) |
| `ViewToolbar` | 현재 화면 경로 + 화면이 등록한 기능 버튼 |
| `SettingDialog` | 테마·언어·날짜 형식 설정 창 (`hideMenuPosition` 으로 메뉴 위치 항목 숨기기) |
| `getViewPath(viewId)` | 화면 주소 만들기 (공유 링크·새 창) |

```jsx
<HashRouter>
  <RouterSync />
  <MyHeader />
  <MyTabs />          {/* useContentStore 로 직접 만든 탭 바 */}
  <ViewToolbar />
  <ViewContainer />
</HashRouter>
```

탭 바와 사이드 메뉴는 따로 제공하지 않아요. `useContentStore`(열린 탭)와 `useMenuStore`(메뉴)로 직접 만들어 주세요.
