---
title: "Scaffold2 component props and navigation config"
description: "Props reference for Scaffold2 in react-declarative: options tree, activeOptionPath, async guards, tab configuration, payload, and slot components."
---

`Scaffold2` is an application shell component. It renders a sidebar with grouped navigation items, an optional search field, a top toolbar with action buttons, and a content area where you render your page content as `children`. Navigation items are defined as a tree of groups, options, and nested sub-options — all with async visibility and disabled predicates so you can gate items behind permission checks. The component is generic over a `Payload` type that is threaded through all visibility and action callbacks.

```tsx
import { Scaffold2 } from "react-declarative";
import HomeIcon from "@mui/icons-material/Home";
import PeopleIcon from "@mui/icons-material/People";

export default function AppShell() {
  return (
    <Scaffold2
      appName="My App"
      activeOptionPath="home"
      options={[
        {
          id: "main",
          label: "Main",
          children: [
            {
              id: "home",
              label: "Home",
              icon: HomeIcon,
            },
            {
              id: "users",
              label: "Users",
              icon: PeopleIcon,
              isVisible: async (payload) => payload.isAdmin,
            },
          ],
        },
      ]}
      onOptionClick={(path, id) => {
        console.log("navigated to", id);
      }}
    >
      <main>Page content goes here</main>
    </Scaffold2>
  );
}
```

## Core props

**`options`** `IScaffold2Group<T>[]` _(required)_

The navigation tree. Each element is a top-level group that contains one or more option items. Groups can be labelled sections in the sidebar; options are the clickable navigation entries.

---

**`activeOptionPath`** `string` _(required)_

The dot-separated path string of the currently active option (e.g. `"main.users"`). `Scaffold2` uses this to highlight the selected item in the sidebar.

---

**`children`** `React.ReactNode` _(required)_

The page content rendered in the main content area to the right of the sidebar.

---

**`appName`** `string`

Application name displayed at the top of the sidebar. Defaults to `"Scaffold2"`.

---

**`payload`** `T`

Context object passed to every `isVisible` and `isDisabled` callback in the options tree and actions array. Use it to carry user roles, feature flags, or tenant context without prop-drilling.

---

**`actions`** `IScaffold2Action<T>[]`

Toolbar action buttons rendered at the top right. Each action extends `IOption` and adds `isVisible(payload)` and `isDisabled(payload)` predicates. The `action` string is forwarded to `onAction` when clicked.

---

**`activeTabPath`** `string`

The dot-separated path of the currently active tab within the active option. Use this when an option has `tabs` defined.

---

**`loading`** `boolean | number`

When truthy, renders a loading indicator in the sidebar. Pass a number to show determinate progress.

---

## `IScaffold2Group` shape

Each entry in `options` must conform to this structure.

**`id`** `string` _(required)_

Unique identifier for the group, used when constructing path strings.

---

**`label`** `string`

Human-readable group heading shown in the sidebar. Omit to render the group without a visible heading.

---

**`icon`** `React.ComponentType`

Icon component rendered next to the group label.

---

**`noHeader`** `boolean`

When `true`, the group heading row is suppressed and only the child option items are rendered.

---

**`isVisible`** `() => boolean | Promise<boolean>`

Async predicate that controls whether the entire group (and all its children) is shown. Evaluated once on mount and after each `deps` change.

---

**`isDisabled`** `() => boolean | Promise<boolean>`

Async predicate that disables all items in the group when it resolves to `true`.

---

**`children`** `IScaffold2Option<T>[]` _(required)_

Navigation options belonging to this group.

---

## `IScaffold2Option` shape

**`id`** `string` _(required)_

Unique identifier for the option. Contributes to the path string (e.g. group `"main"` + option `"users"` → path `"main.users"`).

---

**`label`** `React.ReactNode`

Display label for the navigation item.

---

**`icon`** `React.ComponentType<any>`

Icon rendered to the left of the label in the sidebar.

---

**`isVisible`** `(payload: T) => boolean | Promise<boolean>`

Controls visibility of this specific option. Receives the current `payload`.

---

**`isDisabled`** `(payload: T) => boolean | Promise<boolean>`

Controls whether this option is interactive. A disabled option is rendered but cannot be clicked.

---

**`tabs`** `IScaffold2Tab<T>[]`

Sub-tabs displayed below the option label when the option is active. Each tab has `id`, optional `label`, optional `icon`, and `isVisible`/`isDisabled`/`isActive` predicates that all receive `payload`.

---

**`options`** `IScaffold2Option<T>[]`

Nested child options. Use to build multi-level sidebar trees.

---

## Callbacks

**`onOptionClick`** `(path: string, id: string) => void | boolean | undefined`

Called when a navigation option is clicked. `path` is the full dot-separated path; `id` is the option's own `id`. Return `false` to prevent the default active-path update.

---

**`onOptionGroupClick`** `(path: string, id: string) => void | boolean | undefined`

Called when a group header is clicked.

---

**`onTabChange`** `(path: string, tab: string, id: string) => void`

Called when the user switches tabs within an active option. `tab` is the selected tab's `id`.

---

**`onAction`** `(name: string) => void`

Called when a toolbar action button is activated. `name` is the `action` string from the matching `IScaffold2Action`.

---

**`onInit`** `() => void | Promise<void>`

Called once on component mount. Use it to trigger initial data loads.

---

**`onLoadStart`** `() => void`

Called before an async `isVisible`/`isDisabled` evaluation begins.

---

**`onLoadEnd`** `(isOk: boolean) => void`

Called after an async evaluation completes. `isOk` is `false` if it threw.

---

**`fallback`** `(e: Error) => void`

Error handler invoked if an async predicate or `onInit` rejects.

---

## Slot components

**`AfterAppName`** `React.ComponentType<any>`

Custom content rendered immediately after the app name in the sidebar header.

---

**`BeforeSearch`** `React.ComponentType<any>`

Custom content rendered above the sidebar search field.

---

**`AfterSearch`** `React.ComponentType<any>`

Custom content rendered below the sidebar search field.

---

**`BeforeMenuContent`** `React.ComponentType<any>`

Custom content rendered before the navigation option list.

---

**`AfterMenuContent`** `React.ComponentType<any>`

Custom content rendered after the navigation option list.

---

**`BeforeContent`** `React.ComponentType<any>`

Custom content rendered before the main `children` content area.

---

**`AfterContent`** `React.ComponentType<any>`

Custom content rendered after the main `children` content area.

---

**`Copyright`** `React.ComponentType<any>`

Copyright notice component rendered at the bottom of the sidebar.

---

## Flags

**`noSearch`** `boolean`

When `true`, the search field in the sidebar is hidden. Defaults to `false`.

---

**`noAppName`** `boolean`

When `true`, the app name display is hidden. Defaults to `false`.

---

**`fixedHeader`** `boolean`

When `true`, the top bar remains fixed as the content area scrolls.

---

**`dense`** `boolean`

Reduces vertical padding on navigation items for a more compact sidebar.

---

**`deps`** `any[]`

Dependency array. Changing any value in this array re-evaluates all async `isVisible` / `isDisabled` predicates in the options tree.
