# List component props: columns, handler, actions reference

`List` is a batteries-included data grid component. You supply a `handler` function that receives the current filter state, pagination, sort model, chip toggles, and search string, then returns a page of rows. `List` handles the rest: rendering columns, paginating results, collecting filter form values via a `One`-compatible `filters` schema, and wiring up toolbar actions and per-row menus. The component is generic over `FilterData` (filter form shape), `RowData` (row object shape), and `Payload` (extra context).

```tsx
import { List, FieldType, ColumnType } from "react-declarative";

interface IFilters {
  status: string;
}

interface IRow {
  id: string;
  name: string;
  status: string;
  createdAt: string;
}

export default function UserList() {
  return (
    <List<IFilters, IRow>
      handler={async (filterData, pagination, sort, chips, search) => {
        const res = await fetch(
          `/api/users?page=${pagination.offset / pagination.limit}&q=${search}`
        );
        const { rows, total } = await res.json();
        return { rows, total };
      }}
      columns={[
        { type: ColumnType.Text, field: "name", headerName: "Name", width: "auto" },
        { type: ColumnType.Text, field: "status", headerName: "Status", width: "120px" },
        { type: ColumnType.Text, field: "createdAt", headerName: "Created", width: "180px" },
      ]}
      filters={[
        { type: FieldType.Text, name: "status", title: "Status" },
      ]}
      withSearch
    />
  );
}
```

## Data

**`handler`** `ListHandler<FilterData, RowData, Payload>` (required)

The data-fetching function. Receives `(filterData, pagination, sort, chips, search, payload)` and must return either `RowData[]` (total assumed unknown) or `{ rows: RowData[], total: number | null }`. Can be async. You can also pass a static `RowData[]` array for client-side-only lists.

---

**`columns`** `IColumn<FilterData, RowData, Payload>[]` (required)

Column definitions. Each entry describes one visible column in the grid. See the [IColumn fields](#icolumn-fields) section below.

---

**`filters`** `Field[]`

Field schema (same format as `One`'s `fields` prop) used to render the filter panel. When the user submits the filter form, `handler` is re-called with the new `filterData`.

---

**`payload`** `Payload | (() => Payload)`

Arbitrary context forwarded to `handler`, column `compute` callbacks, action visibility checks, and filter field callbacks. Use it for user identity, tenant IDs, or other cross-cutting values.

---

**`filterData`** `Partial<FilterData>`

Pre-populate the filter form with specific values on first render.

---

**`selectedRows`** `RowId[]`

Controlled selection state. Pass an array of row IDs to externally control which rows appear selected.

---

**`sortModel`** `IListSortItem<RowData>[]`

Initial sort model applied when the list first renders. Each item is `{ field: keyof RowData, sort: 'asc' | 'desc' }`.

---

**`chipData`** `Partial<Record<keyof RowData, boolean>>`

Initial enabled/disabled state of each chip toggle. Keys are `RowData` field names; values are booleans.

---

**`search`** `string`

Pre-populate the search box with a string value on first render.

## Columns (`IColumn`)

**`type`** `ColumnType` (required)

Rendering strategy for the column. Common values: `ColumnType.Text`, `ColumnType.Action`, `ColumnType.CheckBox`, `ColumnType.Component`.

---

**`field`** `string`

Key in `RowData` whose value this column displays. Omit when the column content is fully derived via `compute` or `element`.

---

**`headerName`** `string`

Column header label shown in the grid header row.

---

**`width`** `string | ((containerWidth: number) => string | number)` (required)

Column width. Pass a CSS string such as `"200px"` or `"auto"`, or a function that receives the grid container width and returns a value.

---

**`sortable`** `boolean`

When `true`, clicking the column header cycles through ascending / descending sort. The active sort is passed to `handler` on every re-fetch.

---

**`compute`** `(row: RowData & { _payload: Payload }, payload: Payload) => Promise<Value> | Value`

Derive the displayed value from the row instead of reading `field` directly. Runs for every visible row on each render. Can be async.

---

**`element`** `React.ComponentType<RowData & { _payload: Payload }>`

Replace the default cell renderer with your own React component. The component receives the full row object plus `_payload`.

---

**`columnMenu`** `IListActionOption[]`

Per-column action menu items. Each item has `label`, `action`, optional `icon`, and optional `isVisible`/`isDisabled` callbacks that receive `selectedRows` and `payload`.

---

**`isVisible`** `(params: { filterData, pagination, sortModel, chips, search, payload }) => boolean`

Conditionally hide the entire column based on the current list state. Called with the same params as `handler`.

---

**`primary`** `boolean`

Marks the column as the primary identifier column, used for mobile layout prioritisation.

---

**`phoneHidden`** `boolean`

When `true`, this column is hidden on phone-sized viewports.

---

**`tabletHidden`** `boolean`

When `true`, this column is hidden on tablet-sized viewports.

## Actions and operations

**`actions`** `IListAction<RowData, Payload>[]`

Toolbar-level action buttons rendered above the grid. Each action has a `type` (`ActionType`), optional `label`, optional `icon`, and optional `options` array for dropdown sub-actions. `isVisible` and `isDisabled` receive the current `selectedRows` and `payload`.

---

**`operations`** `IListOperation<RowData, Payload>[]`

Bulk operations shown when one or more rows are selected. Rendered in the selection toolbar.

---

**`rowActions`** `IListRowAction[]`

Per-row action menu items accessible through a context menu icon on each row. Each item specifies a `label`, `action` string, and optional visibility/disabled predicates.

---

**`chips`** `IListChip<RowData>[]`

Quick-filter chip toggles rendered above the grid. Each chip has `name` (a `keyof RowData`), `label`, optional `color`, and an `enabled` default. Active chips are passed to `handler` as a `Record<keyof RowData, boolean>`.

## Callbacks

**`onRowClick`** `(row: RowData, reload: (keepPagination?: boolean) => Promise<void>) => void`

Called when the user clicks anywhere on a row. The second argument is a `reload` function you can call to refresh the list without navigating away.

---

**`onAction`** `(action: string, selectedRows: RowData[], reload: ReloadFn) => void`

Called when a toolbar action is triggered. `action` is the string identifier from the action definition.

---

**`onRowAction`** `(action: string, row: RowData, reload: ReloadFn) => void`

Called when a per-row action menu item is activated. `action` is the item's `action` string.

---

**`onOperation`** `(action: string, selectedRows: RowData[], isAll: boolean, reload: ReloadFn) => void`

Called when a bulk operation is triggered. `isAll` is `true` when the user selected all rows across all pages.

---

**`onSelectedRows`** `(rowIds: RowId[], initialChange: boolean) => void`

Called when the selection changes. `initialChange` is `true` for the event fired on mount.

---

**`onFilterChange`** `(data: FilterData) => void`

Called whenever the filter form values change.

---

**`onSearchChange`** `(search: string) => void`

Called whenever the search box value changes.

---

**`onSortModelChange`** `(sort: IListSortItem<RowData>[]) => void`

Called when the sort model changes (e.g. the user clicks a sortable column header).

---

**`onRows`** `(rows: RowData[]) => void`

Called after each successful data load with the full array of currently displayed rows.

---

**`fallback`** `(e: Error) => void`

Called if `handler` rejects. Use it to display an error message.

---

**`onLoadStart`** `(source: string) => void`

Called when a data fetch begins.

---

**`onLoadEnd`** `(isOk: boolean, source: string) => void`

Called when a data fetch completes. `isOk` is `false` on error.

## Flags

**`withSearch`** `boolean`

Show a search input in the toolbar. The current search string is passed to `handler` on every re-fetch.

---

**`withMobile`** `boolean`

Enable the responsive mobile layout, which reorders and hides columns according to each column's `phoneOrder` and `phoneHidden` settings.

---

**`withArrowPagination`** `boolean`

Replace the default page-number pagination controls with simple previous/next arrow buttons.

---

**`withRangePagination`** `boolean`

Show a page range selector alongside the arrow controls.

---

**`withSingleSort`** `boolean`

Restrict sorting to a single column at a time, clearing any existing sort when the user clicks a new column header.

---

**`withSelectOnRowClick`** `boolean`

Toggle row selection when the user clicks a row body (in addition to clicking the checkbox).

---

**`withToggledFilters`** `boolean`

Start with the filter panel collapsed. The user can expand it via a toolbar toggle button.

---

**`selectionMode`** `SelectionMode`

Controls how rows can be selected. Typical values: `SelectionMode.None`, `SelectionMode.Single`, `SelectionMode.Multiple`.

## Pagination and limits

**`limit`** `number`

Initial page size (number of rows per page). Defaults to the first entry in `rowsPerPage`.

---

**`page`** `number`

Initial page index (zero-based).

---

**`rowsPerPage`** `Array<number | { value: number; label: string }>`

Options shown in the rows-per-page selector. Pass plain numbers or `{ value, label }` objects for custom labels.

## Subjects (reactive control)

**`reloadSubject`** `TSubject<void>`

Emit on this subject to trigger a fresh `handler` call programmatically without unmounting the component.

---

**`rerenderSubject`** `TSubject<void>`

Emit to force a visual re-render without re-fetching data.

---

**`setFilterDataSubject`** `TSubject<FilterData>`

Emit a `FilterData` object to programmatically update the filter form values and re-fetch.
