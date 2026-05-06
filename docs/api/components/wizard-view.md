---
title: "WizardView props: steps, routes, and navigation"
description: "Props reference for WizardView in react-declarative: IWizardStep, IWizardOutlet, WizardNavigation, history integration, and conditional step visibility."
---

`WizardView` renders a multi-step wizard backed by an internal router. The stepper header at the top highlights the active step; the content area renders whichever route outlet is active for the current `pathname`. Steps and routes are defined separately: `steps` controls what appears in the stepper header and `routes` maps URL paths to outlet components. The component is generic over `Data` (the form data flowing through the wizard), `Payload` (context forwarded to outlet props), and `Params` (optional route parameters).

`WizardView` also ships a companion `WizardNavigation` component — a pre-built previous/next button bar you place inside your outlet components.

```tsx
import { WizardView, WizardNavigation } from "react-declarative";
import { createBrowserHistory } from "history";

const history = createBrowserHistory();

const steps = [
  { id: "contact", label: "Contact info" },
  { id: "review",  label: "Review" },
  { id: "confirm", label: "Confirm" },
];

const routes = [
  {
    id: "contact",
    isActive: (path) => path === "/contact",
    element: ({ history, onSubmit }) => (
      <div>
        <p>Step 1 content</p>
        <WizardNavigation
          hasNext
          onNext={() => history.replace("/review")}
        />
      </div>
    ),
  },
  {
    id: "review",
    isActive: (path) => path === "/review",
    element: ({ history }) => (
      <div>
        <p>Step 2 content</p>
        <WizardNavigation
          hasPrev
          hasNext
          onPrev={() => history.replace("/contact")}
          onNext={() => history.replace("/confirm")}
        />
      </div>
    ),
  },
  {
    id: "confirm",
    isActive: (path) => path === "/confirm",
    element: ({ onSubmit, payload }) => (
      <div>
        <p>Step 3 — confirm and submit</p>
        <WizardNavigation
          hasPrev
          hasNext={false}
          onPrev={() => history.replace("/review")}
        />
      </div>
    ),
  },
];

export default function SignupWizard() {
  return (
    <WizardView
      steps={steps}
      routes={routes}
      history={history}
      pathname="/contact"
      payload={{ userId: 42 }}
    />
  );
}
```

## Core props

**`steps`** — `IWizardStep<Payload>[]` (required)

Array of step descriptors that populate the stepper header. Each step maps to one or more routes via matching `id` values or the `isMatch` predicate. Steps with `passthrough: true` are excluded from the visual stepper but still participate in routing.

**`routes`** — `IWizardOutlet<Data, Payload>[]` (required)

Array of outlet definitions. Each outlet specifies an `isActive` predicate that determines which outlet renders for a given pathname, plus an `element` factory function that renders the outlet's content.

**`pathname`** — `string`

The initial path used to determine which route and step are active. Defaults to `"/"`.

**`history`** — `History`

A `history` object (from the `history` package) used for internal navigation. When omitted, `WizardView` creates its own local history instance.

**`payload`** — `Payload`

Context object forwarded to every outlet's `element` props and to `IWizardStep.isVisible` predicates.

## `IWizardStep` shape

**`id`** — `string`

Identifier used to match this step against the active route's `id`. A step is highlighted when `route.id === step.id` or when `step.isMatch(route.id)` returns `true`.

**`label`** — `string`

Text label rendered inside the `StepLabel` in the stepper header.

**`icon`** — `React.ComponentType<any>`

Custom step icon component passed to `StepLabel`'s `StepIconComponent` prop.

**`isMatch`** — `(id: string) => boolean`

Custom predicate for matching this step to a route ID. Use this when multiple routes should highlight the same step.

**`isVisible`** — `(payload: Payload) => boolean`

When provided, the step is filtered out of the stepper if this returns `false`. Called once on mount with the current `payload`.

**`passthrough`** — `boolean`

When `true`, the step is hidden from the stepper header (`display: none`) and the entire `WizardView` chrome (stepper + `PaperView` wrapper) is bypassed — only the active outlet renders. Use this for intermediate steps such as loading screens.

## `IWizardOutlet` shape

**`id`** — `string` (required)

Identifier matched against step `id` values to determine which stepper item to highlight.

**`isActive`** — `(path: string) => boolean` (required)

Predicate evaluated on every navigation event. The outlet whose `isActive` returns `true` for the current path is rendered in the content area.

**`element`** — `(props: IWizardOutletProps<Data, Payload>) => React.ReactElement` (required)

Factory function that receives outlet props and returns the JSX to render. The props include `history`, `payload`, `onSubmit`, `data`, and the `OtherProps` context (`size`, `loading`, `setLoading`, `progress`, `setProgress`).

## Outlet context (`OtherProps`)

Every `element` function receives these additional props alongside the standard outlet props.

**`size`** — `ISize`

Current width and height of the `WizardView` content area in pixels. Useful for responsive layouts inside outlet components.

**`loading`** — `boolean`

Whether a load operation is in progress. `WizardView` shows a linear progress bar at the top of the stepper when this is `true`.

**`setLoading`** — `(loading: boolean) => void`

Call this from inside your outlet component to show or hide the loading progress indicator.

**`progress`** — `number`

Determinate progress value (0–100). When non-zero, `WizardView` switches the progress bar to determinate mode.

**`setProgress`** — `(progress: number) => void`

Set a specific progress percentage. Setting this also resets the loading counter to 0.

## Callbacks

**`onNavigate`** — `(update: Update) => void`

Called on every history navigation event (`REPLACE` action) with the `history` `Update` object. Use this to synchronise the wizard's internal path with your application's global router.

**`onLoadStart`** — `() => void`

Called when an async operation within an outlet begins.

**`onLoadEnd`** — `(isOk: boolean) => void`

Called when an async operation completes. `isOk` is `false` on error.

## Layout flags

**`outlinePaper`** — `boolean`

Renders the outer `PaperView` wrapper with a visible border outline instead of an elevation shadow.

**`transparentPaper`** — `boolean`

Renders the outer `PaperView` wrapper with a transparent background, including the stepper header.

**`withScroll`** — `boolean`

Enables vertical scrolling inside the content area. Useful when outlet content can exceed the wizard's height.

**`fullScreen`** — `boolean`

Passes the `fullScreen` flag through to the internal `OutletView`, enabling full-viewport rendering for modal-style wizards.

**`sx`** — `SxProps`

MUI `sx` prop applied to the root `PaperView` element.

**`style`** — `React.CSSProperties`

Inline styles applied to the root element.

**`className`** — `string`

CSS class applied to the root element.

---

## WizardNavigation

`WizardNavigation` is a companion toolbar component you render inside outlet elements to give users previous and next buttons. It manages its own loading state to prevent double-clicks on async navigation handlers.

```tsx
import { WizardNavigation } from "react-declarative";

function ReviewStep({ history }) {
  return (
    <div>
      <p>Review your details</p>
      <WizardNavigation
        hasPrev
        hasNext
        labelPrev="Back"
        labelNext="Continue"
        onPrev={() => history.replace("/contact")}
        onNext={async () => {
          await saveToServer();
          history.replace("/confirm");
        }}
      />
    </div>
  );
}
```

**`hasPrev`** — `boolean`

When `true`, the Previous button is enabled. Defaults to `false`.

**`hasNext`** — `boolean`

When `true`, the Next button is enabled. Defaults to `false`.

**`onPrev`** — `() => void | Promise<void>`

Called when the user clicks the Previous button. Can be async — the button disables automatically while the promise is pending.

**`onNext`** — `() => void | Promise<void>`

Called when the user clicks the Next button. Can be async — both buttons disable while the promise is pending.

**`labelPrev`** — `string`

Label for the Previous button. Defaults to `"Prev"`.

**`labelNext`** — `string`

Label for the Next button. Defaults to `"Next"`.

**`disabled`** — `boolean`

When `true`, both buttons are disabled regardless of `hasPrev` and `hasNext`.

**`fallback`** — `(e: Error) => void`

Error handler invoked if `onPrev` or `onNext` rejects.

**`AfterPrev`** — `React.ComponentType<any>`

Custom content rendered immediately after the Previous button.

**`BeforeNext`** — `React.ComponentType<any>`

Custom content rendered immediately before the Next button.
