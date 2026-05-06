---
title: docs/components/scaffold
group: docs
---
# Scaffold2: config-driven Material Design app shell

`<Scaffold2 />` implements the Material Design app shell pattern — the top app bar, collapsible side navigation drawer, and content area — entirely from a configuration array. You declare groups and menu items as plain objects, attach async visibility and disabled guards, and let the component wire up the chrome. Your page content goes in as `children`. `Scaffold3` is a newer variant with an updated visual style; the configuration API is identical.

## Installation

```bash
npm install --save react-declarative tss-react @mui/material @emotion/react @emotion/styled
```

## Basic setup

### Step 1: Define your navigation groups

Create an `IScaffold2Group[]` array. Each group has an `id`, an optional `label`, and a `children` array of `IScaffold2Option` items.

```tsx navigation.ts
import {
  IScaffold2Group,
  IScaffold2Option,
} from 'react-declarative';
import PeopleIcon from '@mui/icons-material/People';
import DnsIcon from '@mui/icons-material/Dns';
import PublicIcon from '@mui/icons-material/Public';

export const options: IScaffold2Group[] = [
  {
    id: 'build',
    label: 'Build',
    children: [
      {
        id: 'authentication',
        label: 'Authentication',
        icon: PeopleIcon,
        isVisible: async () => await authService.hasRole('admin'),
      },
      {
        id: 'database',
        label: 'Database',
        icon: DnsIcon,
      },
      {
        id: 'hosting',
        label: 'Hosting',
        icon: PublicIcon,
        isDisabled: async () => await maintenanceGuard(),
      },
    ],
  },
  {
    id: 'release',
    label: 'Release',
    children: [
      {
        id: 'analytics',
        label: 'Analytics',
      },
    ],
  },
];
```

### Step 2: Render Scaffold2 with your options

Pass `options`, `activeOptionPath` (the currently selected item id chain), and your page content as `children`.

```tsx App.tsx
import { Scaffold2 } from 'react-declarative';
import { options } from './navigation';

export const App = () => {
  const [activePath, setActivePath] = React.useState('build.authentication');

  return (
    <Scaffold2
      appName="My App"
      options={options}
      activeOptionPath={activePath}
      onOptionClick={(path, id) => {
        setActivePath(path);
        router.navigate(`/${id}`);
      }}
    >
      {/* your routed page content */}
      <Outlet />
    </Scaffold2>
  );
};
```

## Key props

**`options`** `IScaffold2Group[]` (required)

The full navigation tree. Groups contain options; options can nest further options and declare tabs.

---

**`activeOptionPath`** `string` (required)

A dot-separated path that identifies the currently highlighted menu item, e.g. `"build.authentication"`. The Scaffold highlights the matching item in the side drawer.

---

**`activeTabPath`** `string`

Identifies the active tab within the current option, e.g. `"tab1"`. Tabs are rendered in the top app bar.

---

**`appName`** `string`

The application name shown in the top app bar. Defaults to `"Scaffold2"`.

---

**`noSearch`** `boolean`

Hides the search box in the side drawer when `true`.

---

**`noAppName`** `boolean`

Hides the app name from the top bar.

---

**`payload`** `T`

An arbitrary object forwarded to every `isVisible` and `isDisabled` callback on options and tabs. Use it to pass the current user session.

---

**`onOptionClick`** `(path: string, id: string) => void | boolean`

Called when the user clicks a menu item. `path` is the dot-joined ancestor chain; `id` is the clicked option's own `id`. Return `false` to prevent the default highlight update.

---

**`onTabChange`** `(path: string, tab: string, id: string) => void`

Called when the user switches between tabs in the top bar.

---

**`onInit`** `() => void | Promise<void>`

Called once after mount. Use it to trigger async operations such as loading the user profile before computing visibility.

## Async visibility and disabled guards

Both `IScaffold2Group` and `IScaffold2Option` accept async functions for `isVisible` and `isDisabled`. The Scaffold evaluates all guards on mount (and whenever `deps` changes) and hides or disables items accordingly.

```tsx async-guards.tsx
import { IScaffold2Group } from 'react-declarative';

const options: IScaffold2Group[] = [
  {
    id: 'admin',
    label: 'Admin',
    isVisible: async () => {
      // hide this entire group for non-admins
      const role = await authService.getCurrentRole();
      return role === 'admin';
    },
    children: [
      {
        id: 'users',
        label: 'Users',
        isDisabled: async (payload) => {
          return !payload.hasUsersPermission;
        },
      },
    ],
  },
];
```

> **Warning:** Guards run asynchronously. Items are visible and enabled by default until the guard resolves. If you need to block access immediately, set `loading` on the `Scaffold2` while guards are pending.

## Tabs and nested options

An `IScaffold2Option` can have both `tabs` (rendered in the top app bar) and `options` (nested sub-items in the side menu):

```tsx tabs-and-options.tsx
import { IScaffold2Group } from 'react-declarative';

const options: IScaffold2Group[] = [
  {
    id: 'build',
    label: 'Build',
    children: [
      {
        id: 'authentication',
        label: 'Authentication',
        // tabs appear in the top bar when this option is active
        tabs: [
          { id: 'users',    label: 'Users' },
          { id: 'sessions', label: 'Sessions' },
        ],
        // options appear as sub-items in the side menu
        options: [
          { id: 'users',    label: 'Users list' },
          { id: 'sessions', label: 'Active sessions' },
        ],
      },
    ],
  },
];
```

## Scaffold2 vs Scaffold3 vs Scaffold (v1)

**Scaffold (v1)**

The original app shell. Minimal API but limited customisation. Consider migrating to Scaffold2.

**Scaffold2**

Recommended for most apps. Full async guard support, tabs, nested options, payload forwarding, and slot components.

**Scaffold3**

Updated visual style with the same configuration API as Scaffold2. Drop-in replacement when you want a refreshed look.

> **Tip:** All three variants accept the same `IScaffold2Group[]` array for `options`. You can switch between Scaffold2 and Scaffold3 by changing the import without touching your navigation config.

## Slot components

`Scaffold2` exposes named slot props for inserting custom React components at specific positions in the layout:

| Prop | Position |
|---|---|
| `AfterAppName` | After the app name in the top bar |
| `BeforeActionMenu` | Before the action icon row |
| `BeforeSearch` / `AfterSearch` | Around the search box in the drawer |
| `BeforeMenuContent` / `AfterMenuContent` | Around the menu item list |
| `BeforeContent` / `AfterContent` | Around the main content area |
| `Copyright` | Footer of the side drawer |
