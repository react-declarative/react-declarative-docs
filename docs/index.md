---
title: "react-declarative docs: schema-driven React UI library"
description: "react-declarative builds forms, data grids, and app shells from TypedField JSON schemas. Zero manual state wiring. TypeScript IntelliSense on every field."
---

react-declarative is a TypeScript React library that turns JSON field schemas into fully functional MUI-based forms, data grids, and application shells — with zero manual state wiring. Define your UI declaratively, and the library handles rendering, validation, and state management automatically.

- [Installation](/installation) — Install react-declarative and its MUI peer dependencies in minutes.
- [Quick Start](/quickstart) — Build your first declarative form in under five minutes.
- [Field Types](/concepts/field-types) — Explore all 40+ field types: text, combo, date, rating, file, and more.
- [Component Reference](/api/components/one) — Full prop reference for One, List, Scaffold2, KanbanView, and WizardView.

## What you can build

react-declarative is not just a form library. It provides a complete toolkit for building data-driven React applications:

- [Forms & Validation](/components/one) — Nested grid forms with inline validation, conditional fields, and file uploads.
- [Data Grids](/components/list) — Filterable, sortable, paginated data grids with mobile-first adaptive layout.
- [App Shells](/components/scaffold) — Material Design app shells with navigation, tabs, and action menus from config.
- [Kanban Boards](/components/kanban) — Drag-and-drop kanban boards with real-time column updates.

## How it works

1. **Install the library** — Add react-declarative and its MUI peer dependencies to your project.
2. **Define a field schema** — Write a `TypedField[]` array describing your form fields, layouts, and validation rules.
3. **Render with One or List** — Pass your schema to `<One />` for forms or `<List />` for data grids. State is managed automatically.
4. **Connect your data** — Provide a `handler` function (async or sync) to load initial data, and an `onChange` callback to receive updates.

> **Tip:** Try react-declarative in your browser without installing anything at the [interactive playground](https://react-declarative-playground.github.io/).
