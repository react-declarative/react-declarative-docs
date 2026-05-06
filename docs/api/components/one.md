# One component: full props and API reference

`One` is the core form-rendering component in react-declarative. You describe your form as a flat array of field descriptors (the `fields` prop), provide a data source via `handler` or `data`, and `One` takes care of rendering, validation, and change propagation. The component is generic over three type parameters: `Data` (the shape of your form object), `Payload` (extra context passed to field callbacks), and `Field` (defaults to `TypedField<Data, Payload>`).

```tsx
import { One, FieldType } from "react-declarative";

interface IUserForm {
  name: string;
  email: string;
  age: number;
}

const fields = [
  {
    type: FieldType.Text,
    name: "name",
    title: "Full name",
  },
  {
    type: FieldType.Text,
    name: "email",
    title: "Email address",
  },
  {
    type: FieldType.Text,
    name: "age",
    title: "Age",
    inputType: "number",
  },
];

export default function UserForm() {
  return (
    <One<IUserForm>
      fields={fields}
      handler={async () => ({ name: "Alice", email: "alice@example.com", age: 30 })}
      onChange={(data, initial) => {
        if (!initial) console.log("changed:", data);
      }}
    />
  );
}
```

## Data

### `fields` — `Field[]` (required)

Array of field descriptors that define the form layout and inputs. Each descriptor is a `TypedField<Data, Payload>` object with at minimum a `type` (e.g. `FieldType.Text`) and `name` matching a key in `Data`.

---

### `handler` — `OneHandler<Data, Payload>`

Data source for the form. Accepts a plain `Data` object, a synchronous function `(payload: Payload) => Data | null`, or an async function returning `Promise<Data | null>`. Called once on mount (and on `reloadSubject` emission) to populate the form. Use `handler` when data must be fetched; use `data` for controlled, React-state-owned values.

---

### `data` — `Data | null`

Alternative to `handler` for React developers who prefer a controlled pattern. Pass your data object directly; `One` re-renders whenever this prop changes. Cannot be used simultaneously with `handler`.

---

### `payload` — `Payload | (() => Payload)`

Arbitrary context object passed through to every field callback (`isVisible`, `isDisabled`, `validate`, etc.) and to `handler`. Use it to thread user identity, permissions, or other cross-cutting state into your field definitions without adding it to `Data`.

---

### `context` — `Record<string, any>`

Similar to `payload` but participates in change detection. Updates to `context` trigger field re-renders, making it suitable for values that your field visibility or validation logic depends on dynamically.

---

## Callbacks

### `onChange` — `(data: Data, initial: boolean) => void`

Called every time any field value changes. `data` is the full, up-to-date form object. `initial` is `true` only for the first emission that fires immediately after `handler` resolves — use this flag to distinguish the initial hydration from user edits.

---

### `onFocus` — `(name: string, data: Data, payload: Payload, onValueChange: (value: Value) => void, onChange: (data: Data) => void) => void`

Called when a field receives focus. Receives the field name, current form data, payload, and two imperative helpers to push a new field value or a full data object from inside the callback.

---

### `onBlur` — `(name: string, data: Data, payload: Payload, onValueChange: (value: Value) => void, onChange: (data: Data) => void) => void`

Called when a field loses focus. Same signature as `onFocus`.

---

### `onClick` — `(name: string, data: Data, payload: Payload, onValueChange: (value: Value) => void, onChange: (data: Data) => void, e: React.MouseEvent) => void | Promise<void>`

Called when the user clicks a field. The native `MouseEvent` is provided as the last argument. Useful for custom action fields or triggering navigation on click.

---

### `onMenu` — `(name: string, action: string, data: Data, payload: Payload, onValueChange: (value: Value) => void, onChange: (data: Data) => void) => void`

Called when the user picks an item from a field's context menu. `action` is the string identifier of the chosen menu item.

---

### `onReady` — `() => void`

Fired once after all fields have completed their first render. Use this as a signal that the form is fully interactive, for example to focus a field programmatically.

---

### `onInvalid` — `(name: string, msg: string, payload: Payload) => void`

Called whenever a field fails validation. `name` is the field's data key and `msg` is the validation error string returned by the field's `validate` function.

---

### `onLoadStart` — `(source: string) => void`

Called when an async operation (such as the `handler` fetch) begins. `source` identifies which part of the component triggered the load.

---

### `onLoadEnd` — `(isOk: boolean, source: string) => void`

Called when an async operation completes. `isOk` is `false` if the operation threw an error.

---

### `fallback` — `(e: Error) => void`

Error handler invoked if `handler` rejects. Use this to show a toast or navigate away rather than letting the error propagate silently.

---

## Layout

### `className` — `string`

CSS class applied to the root group element.

---

### `style` — `React.CSSProperties`

Inline styles applied to the root group element.

---

### `sx` — `SxProps`

MUI `sx` prop applied to the root group element, giving you access to the theme and responsive breakpoints.

---

### `baseline` — `boolean`

When `true`, all fields are anchored to the bottom edge of their row, aligning multi-height fields along a common baseline.

---

### `noBaseline` — `boolean`

When `true`, fields and layouts are anchored to the top edge instead. Overrides `baseline`.

---

### `outlinePaper` — `boolean`

Converts any `FieldType.Paper` layout in the schema into a `FieldType.Outline` style, which renders with a visible border instead of an elevated card background.

---

### `transparentPaper` — `boolean`

Renders `FieldType.Paper` layouts with a transparent background.

---

## Flags

### `readonly` — `boolean`

When `true`, all input fields are rendered in a read-only state. Values are visible but not editable.

---

### `disabled` — `boolean`

When `true`, all input fields are disabled. Unlike `readonly`, disabled fields typically render with reduced opacity and do not receive focus.

---

### `dirty` — `boolean`

When `true`, validation errors are shown immediately on all fields without waiting for the user to focus or blur them.

---

### `features` — `Record<string, Value> | string[] | (() => string[] | Record<string, Value>)`

Business feature flags that control field visibility. Fields in your schema can declare a `features` list; only fields whose features intersect this set are rendered. Pass an array of active feature names or a record mapping feature names to values.

---

## Advanced

### `apiRef` — `React.Ref<IOneApi>`

Attach a ref to receive the `IOneApi` imperative handle, which lets you trigger reloads, read current values, or force re-validation from outside the component.

---

### `changeDelay` — `number`

Debounce delay in milliseconds applied to `onChange` emissions. Useful when you want to reduce the frequency of server round-trips triggered by rapid typing.

---

### `fieldDebounce` — `number`

Debounce in milliseconds applied specifically to `FieldType.Text` inputs before their value is committed to the form state.

---

### `reloadSubject` — `TSubject<void>`

RxJS-like subject. Emit on it to trigger a fresh call to `handler`, re-hydrating the form without unmounting.

---

### `changeSubject` — `TSubject<Data>`

Subject that, when emitted, calls `change(data, true)` — treating the emission as an initial data replacement rather than a user edit.

---

### `updateSubject` — `TSubject<Data>`

Subject that, when emitted, calls `change(data, false)` — treating the emission as an incremental update.

---

### `readTransform` — `(value: string | string[], name: string, data: Data, payload: Payload) => Value`

Transform applied when reading a field value out of `Data` before displaying it. Use this to deserialize stored values (e.g. convert a comma-separated string to an array).

---

### `writeTransform` — `(value: string | string[], name: string, data: Data, payload: Payload) => Value`

Transform applied when writing a field value back into `Data`. The inverse of `readTransform`.

---

### `slots` — `Partial<ISlotFactoryContext>`

Override the default slot components used to render individual field types. Each key in `ISlotFactoryContext` corresponds to a `FieldType` and accepts a custom React component.
