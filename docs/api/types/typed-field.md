---
title: docs/api/types/typed-field
group: docs
---
# TypedField\<Data, Payload\>: typed schema definition

`TypedField<Data, Payload>` is a TypeScript discriminated union that gives you full IntelliSense when building `fields` arrays for the `<One />` component. As soon as you set the `type` property on a field object to a specific `FieldType` value, TypeScript narrows the available props to exactly those that are valid for that field type — no more guessing which properties apply and which don't.

```ts
import { TypedField, FieldType } from 'react-declarative';
```

---

## Generic parameters

```ts
type TypedField<
  Data = IAnything,
  Payload = IAnything
>
```

**`Data`** _(generic)_

The shape of the data object your form operates on. When you provide this type, all data-aware callbacks — `isInvalid`, `isDisabled`, `isVisible`, `isReadonly`, `compute`, `onChange` — receive `Data` as their argument type, giving you typed access to your form values. Defaults to `IAnything` (an open `Record<string, any>`).

**`Payload`** _(generic)_

A secondary context object passed down through the form without being part of the data. Use `Payload` for things like user permissions, feature flags, or API handles that your field callbacks need but that should not be saved. Defaults to `IAnything`.

---

## Usage

Annotate your fields array with `TypedField<Data, Payload>[]` to get narrowed props per field type:

```ts
import { TypedField, FieldType, One } from 'react-declarative';

interface UserData {
  firstName: string;
  lastName: string;
  role: string;
  isAdmin: boolean;
  bio: string;
}

interface AppPayload {
  canEditRole: boolean;
}

const fields: TypedField<UserData, AppPayload>[] = [
  {
    type: FieldType.Group,
    fields: [
      {
        type: FieldType.Text,
        name: 'firstName',
        title: 'First name',
        columns: '6',
      },
      {
        type: FieldType.Text,
        name: 'lastName',
        title: 'Last name',
        columns: '6',
      },
    ],
  },
  {
    type: FieldType.Combo,
    name: 'role',
    title: 'Role',
    itemList: ['admin', 'editor', 'viewer'],
    // payload is typed to AppPayload here
    isDisabled: (data, payload) => !payload.canEditRole,
  },
  {
    type: FieldType.Switch,
    name: 'isAdmin',
    title: 'Admin',
    // data is typed to UserData — no casting needed
    isVisible: (data) => data.role === 'admin',
  },
];
```

---

## Common IField properties

All field types share the properties defined on `IField<Data, Payload>`. The table below covers the properties you will use most often.

### Identity and layout

**`type`** _(FieldType, required)_

The field type. Determines which component is rendered and which additional props are available. See the [FieldType reference](/api/types/field-type).

**`name`** _(string)_

The key in your `Data` object that this field reads from and writes to. Omit for layout containers (`Group`, `Paper`, etc.) and display-only fields like `Typography` or `Line`.

**`title`** _(string)_

The field label shown to the user. Maps to the `label` prop of Material UI form controls.

**`description`** _(string)_

Secondary helper text rendered below the field. Maps to the `helperText` prop.

**`placeholder`** _(string)_

Placeholder text for `Text`, `Combo`, `Items`, and `Complete` fields.

### Responsive columns

The `<One />` component uses a 12-column CSS grid. All column values are CSS fractions expressed as strings.

**`columns`** _(string)_

Column span applied at all breakpoints (e.g. `'6'` for half-width). When set, it overrides `phoneColumns`, `tabletColumns`, and `desktopColumns`.

**`phoneColumns`** _(string)_

Column span on phone-sized viewports.

**`tabletColumns`** _(string)_

Column span on tablet-sized viewports.

**`desktopColumns`** _(string)_

Column span on desktop-sized viewports.

### Default value

**`defaultValue`** _(Value | ((payload: Payload) => Value))_

The initial value to use when the field's `name` key is absent or `undefined` in the data object. Can be a static value or a function that receives `Payload` and returns a value.

### Visibility and interactivity callbacks

These callbacks receive the full `Data` object and `Payload` as arguments and are called on every render cycle. Keep them pure and fast.

**`isVisible`** _((data: Data, payload: Payload) => boolean)_

Return `false` to hide the field from the DOM. The field's value is still present in the data object.

**`isDisabled`** _((data: Data, payload: Payload) => boolean)_

Return `true` to disable the field control, preventing user interaction.

**`isReadonly`** _((data: Data, payload: Payload) => boolean)_

Return `true` to make the field read-only (visible but not editable).

**`isInvalid`** _((data: Data, payload: Payload) => null | string)_

Return `null` when the value is valid, or a non-null error string to show a validation error. Blocks form submission when non-null.

**`hidden`** _(boolean | ((payload: Payload) => boolean))_

Statically removes the field from the render tree. Unlike `isVisible`, this does not receive `data` — it is intended for permission-level or payload-driven visibility that does not depend on form values.

### Computed value

**`compute`** _((data: Data, payload: Payload) => Value | Promise\<Value\>)_

Makes the field read-only and derives its value dynamically from the data object. The computed value is not written back to the data — it is display-only. Supports async computation.

### Icons

**`leadingIcon`** _(React.ComponentType\<any\>)_

An icon component rendered on the left side of a `Text` field input.

**`trailingIcon`** _(React.ComponentType\<any\>)_

An icon component rendered on the right side of a `Text` field input.

**`leadingIconClick`** _((value, data, payload, onValueChange, onChange) => void)_

Click handler for the leading icon. Use this to open a modal or trigger a lookup that sets the field value.

**`trailingIconClick`** _((value, data, payload, onValueChange, onChange) => void)_

Click handler for the trailing icon.

### Focus / blur callbacks

**`focus`** _((name, data, payload, onValueChange, onChange) => void)_

Called when the field receives focus. You can call `onChange(newData)` to update other fields in response — for example, to populate dependent dropdowns.

**`blur`** _((name, data, payload, onValueChange, onChange) => void)_

Called when the field loses focus. Useful for async validation or data enrichment on leaving the field.

---

## Nested fields

Layout containers accept child field definitions via `fields` (for multi-child layouts) or `child` (for single-child layouts):

```ts
{
  type: FieldType.Paper,
  fields: [               // <-- TypedField<Data, Payload>[]
    {
      type: FieldType.Group,
      fields: [
        { type: FieldType.Text, name: 'email', title: 'Email' },
      ],
    },
  ],
}
```

Both `fields` and `child` are typed as `TypedField<Data, Payload>[]` and `TypedField<Data, Payload>` respectively, so the same generic parameters flow through the entire nested tree.

---

## Data generic in practice

Without the `Data` generic, every callback argument is `IAnything` (essentially `any`). Providing your data type gives you autocompletion on field names and type-safe callbacks:

```ts
// Without Data generic — no type safety
const fields: TypedField[] = [
  {
    type: FieldType.Text,
    name: 'email',
    isInvalid: (data) => data.emial ? null : 'Required', // typo: no error
  },
];

// With Data generic — TypeScript catches the typo
const fields: TypedField<UserData>[] = [
  {
    type: FieldType.Text,
    name: 'email',
    isInvalid: (data) => data.emial ? null : 'Required',
    //                         ^^^^^ Property 'emial' does not exist on type 'UserData'
  },
];
```

## Payload generic in practice

Use `Payload` to pass read-only context that your fields need but that should not be stored in form data:

```ts
interface FormPayload {
  userPermissions: string[];
}

const fields: TypedField<OrderData, FormPayload>[] = [
  {
    type: FieldType.Text,
    name: 'discount',
    title: 'Discount %',
    isDisabled: (data, payload) =>
      !payload.userPermissions.includes('apply_discount'),
  },
];

// Pass payload to One:
<One
  fields={fields}
  data={orderData}
  payload={{ userPermissions: currentUser.permissions }}
  onChange={handleChange}
/>
```
