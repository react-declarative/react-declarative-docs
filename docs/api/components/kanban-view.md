---
title: docs/api/components/kanban-view
group: docs
---
# KanbanView props: columns, items, and row definition

`KanbanView` renders a horizontal kanban board made up of typed columns. Each column displays a virtualised list of cards drawn from the `items` array. Users can drag cards between columns; the component fires `onChangeColumn` so you can persist the move. Card content is declarative: each column carries a `rows` array that describes what data to show on every card in that column, and the `value` function on each row is called per-card to asynchronously compute the displayed text or node.

The component is generic over `Data` (the shape of each item's data object), `Payload` (context forwarded to row callbacks), and `ColumnType` (the value used to identify columns, defaults to `IAnything`).

```tsx
import { KanbanView } from "react-declarative";

type Status = "todo" | "in-progress" | "done";

interface ITask {
  id: string;
  title: string;
  assignee: string;
  column: Status;
  data: { title: string; assignee: string };
}

const columns = [
  {
    column: "todo" as Status,
    label: "To Do",
    color: "#e3f2fd",
    rows: [
      {
        label: "Title",
        value: (_id, data) => data.title,
      },
      {
        label: "Assignee",
        value: (_id, data) => data.assignee,
      },
    ],
  },
  { column: "in-progress" as Status, label: "In Progress", color: "#fff3e0", rows: [] },
  { column: "done" as Status, label: "Done", color: "#e8f5e9", rows: [] },
];

export default function TaskBoard({ tasks }: { tasks: ITask[] }) {
  return (
    <KanbanView
      columns={columns}
      items={tasks}
      onChangeColumn={(id, newColumn, data, payload) => {
        console.log(`Card ${id} moved to ${newColumn}`);
      }}
    />
  );
}
```

## Data

**`items`** `IBoardItem<Data, Payload, ColumnType>[]` _(required)_

The flat array of all kanban cards. Each item must have an `id` string and a `column` value matching one of the column identifiers in `columns`. `KanbanView` partitions this array by column on each render.

**`columns`** `IBoardColumn<Data, Payload, ColumnType>[]` _(required)_

Column definitions. Each entry is either an `IBoardColumnInternal` (a real column with cards) or an `IBoardDivider` (a visual spacer between column groups). See the [IBoardColumn shape](#ibboardcolumn-shape) section for details.

**`payload`** `Payload | (() => Payload)`

Context object forwarded to every `value`, `visible`, and `click` function on `IBoardRow`. Use it to pass user data, permissions, or API clients.

## `IBoardItem` shape

Each element in the `items` array must conform to this interface.

**`id`** `string` _(required)_

Unique identifier for the card. Used as the React key and passed to all row callback functions.

**`column`** `ColumnType` _(required)_

The current column this card belongs to. Must match the `column` field of one of your `IBoardColumn` entries.

**`data`** `Data` _(required)_

The payload object for this card. Passed as the second argument to all `IBoardRow` callback functions.

**`label`** `React.ReactNode | ((id: string, data: Data, payload: Payload) => React.ReactNode | Promise<React.ReactNode>)`

Card header label. Can be a static node or an async function. If omitted, falls back to the board-level `cardLabel` prop.

**`updatedAt`** `string`

ISO 8601 timestamp. When `withUpdateOrder` is enabled, cards are sorted by this field in descending order (most recently updated first).

## `IBoardColumn` shape

**`column`** `ColumnType` _(required)_

Unique identifier for this column. Cards whose `column` field equals this value are rendered here.

**`label`** `string`

Column header label shown above the card list.

**`color`** `string`

Background color of the column body and the small color dot in the header. Any valid CSS color string. Defaults to a theme-aware neutral tone.

**`rows`** `IBoardRow<Data, Payload>[]` _(required)_

Declarative row definitions that describe what data to render inside each card belonging to this column. Each row produces one labeled data line on the card.

## `IBoardRow` shape

**`label`** `React.ReactNode` _(required)_

The field label displayed to the left of the value on the card.

**`value`** `(id: string, data: Data, payload: Payload) => React.ReactNode | Promise<React.ReactNode>` _(required)_

Async function that computes the displayed value for this row. Results are cached with a TTL controlled by `rowTtl`.

**`visible`** `boolean | ((id: string, data: Data, payload: Payload) => boolean | Promise<boolean>)`

Controls whether this row is rendered on the card. Omit or set to `true` to always show the row. When a function is provided it is evaluated async and the row is hidden until it resolves `true`.

**`click`** `(id: string, data: Data, payload: Payload) => void | Promise<void>`

Called when the user clicks this specific row's value on the card.

## Callbacks

**`onChangeColumn`** `(id: string, column: ColumnType, data: Data, payload: IAnything) => void | Promise<void>`

Called after a card is successfully dragged to a new column. `id` is the card's identifier; `column` is the target column. This is where you persist the status change to your backend.

**`onCardLabelClick`** `(id: string, data: Data, payload: IAnything) => void`

Called when the user clicks the card's header label.

**`onDataRequest`** `(initial: boolean) => void`

Called on mount (`initial = true`) and whenever the page becomes visible again (`initial = false`). Use it to trigger a data refresh from your parent component.

**`onLoadStart`** `() => void`

Called when an async row value computation begins.

**`onLoadEnd`** `(isOk: boolean) => void`

Called when an async row value computation completes. `isOk` is `false` if it threw.

**`fallback`** `(e: Error) => void`

Error handler invoked when an async row callback rejects.

**`filterFn`** `(item: IBoardItem<Data, Payload, ColumnType>) => boolean`

Client-side predicate applied to `items` before partitioning into columns. Cards that return `false` are never rendered.

## Slot components

**`AfterCardContent`** `React.ComponentType<{ id: string; data: Data; payload: IAnything }>`

Custom content rendered below the declarative rows inside every card.

**`BeforeColumnTitle`** `React.ComponentType<{ column: ColumnType; payload: IAnything }>`

Custom content rendered to the left of the column header title.

**`AfterColumnTitle`** `React.ComponentType<{ column: ColumnType; payload: IAnything }>`

Custom content rendered to the right of the column header title (e.g. a card count badge).

## Layout and performance

**`sx`** `SxProps`

MUI `sx` prop applied to the root container.

**`style`** `React.CSSProperties`

Inline styles applied to the root container.

**`className`** `string`

CSS class applied to the root container.

**`bufferSize`** `number`

Number of extra cards rendered outside the visible viewport by the internal virtual list. Increase this value if you see blank cards during fast scrolling. Defaults to `15`.

**`minRowHeight`** `number`

Minimum card height in pixels used by the virtual list to calculate scroll positions. Defaults to `125`.

**`rowTtl`** `number`

Time-to-live in milliseconds for the async `value` and `label` result cache. After this duration, the next render triggers a fresh call. Defaults to `500`.

**`cardLabel`** `React.ReactNode | ((id: string, data: Data, payload: Payload) => React.ReactNode | Promise<React.ReactNode>)`

Default card header label used when an individual item does not specify its own `label`. Can be a static node or an async function receiving `(id, data, payload)`.

## Flags

**`withGoBack`** `boolean`

When `true`, cards can be dragged to a column that appears earlier in the `columns` array (i.e. "backwards" in the workflow). Defaults to `false`.

**`withUpdateOrder`** `boolean`

When `true`, cards within each column are sorted by `updatedAt` in descending order — most recently updated cards appear at the top.

**`withHeaderTooltip`** `boolean`

When `true`, a tooltip is shown on column headers, useful when the label is truncated.

**`disabled`** `boolean`

When `true`, all drag interactions are disabled and the board is read-only.

**`deps`** `any[]`

Dependency array. Changing any value clears the row cache and triggers a re-partition of `items`.

**`reloadSubject`** `TSubject<void>`

Emit on this subject to clear the row and label caches and force all cards to re-evaluate their `value` functions.

## Static helper

`KanbanView.enableScrollOnDrag(ref, options?)` is a static utility you can attach as a `useEffect` to enable auto-scroll while dragging near the left or right edges of the board.

```tsx
const boardRef = useRef<HTMLDivElement>();

useEffect(
  KanbanView.enableScrollOnDrag(boardRef, { threshold: 200, speed: 15 }),
  []
);

return <KanbanView ref={boardRef} ... />;
```

| Option | Type | Default | Description |
|---|---|---|---|
| `threshold` | `number` | `200` | Pixel distance from the edge that triggers auto-scroll. |
| `speed` | `number` | `15` | Pixels scrolled per interval tick. |
