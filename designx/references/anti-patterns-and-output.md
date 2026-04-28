# Anti-Patterns and Output

## Forbidden — Visual

- Inter + purple gradient as a default aesthetic. Choose a direction with intent.
- Every section wrapped in a `Card` component.
- Generic centered hero with two buttons and a gradient blob.
- Decorative icons with no semantic meaning.
- Random grays not from a defined token set.
- Drop shadows on everything. Elevation must be intentional.
- Oversized headings in operational or data-dense tools.
- Filler subtitles, slogans, or marketing copy inside application UI.
- Fake SaaS jargon (e.g., "Orchestration Nexus", "Command Intelligence Suite").
- Text over images without a contrast overlay or treatment.
- Inconsistent corner radii not from a token set.
- More than one primary color accent competing for attention per screen.

## Forbidden — Layout

- Fixed pixel widths that break at unexpected breakpoints.
- Any horizontal overflow at 375px, 768px, or 1280px.
- No max-width constraint on reading content.
- Dashboard composed entirely of metric cards with no table, chart, or drill-down.
- Hamburger menu on desktop.
- Using margins where CSS gap should be used in grid/flex layouts.
- Fixed sidebars on mobile.
- One layout that doesn't adapt behavior (not just width) across breakpoints.

## Forbidden — UX

- No loading state. No empty state. No error state.
- Inputs labeled only with placeholder text.
- Disabled buttons with no explanation of why they are disabled.
- Multiple competing primary CTAs on the same screen.
- Async action (form submit, mutation) with no pending feedback.
- No permission/forbidden state when role-based access applies.
- Optimistic update with no rollback path.

## Forbidden — Interaction and Animation

- Interactive elements with no `hover`, `focus`, or `active` state.
- `outline: none` without a custom visible focus replacement.
- Click handlers on `<div>` or `<span>` instead of `<button>` or `<a>`.
- Missing `aria-label` on any icon-only button.
- `alert()` or `confirm()` used for in-app dialogs.
- Toast used for a critical persistent error (use a banner or modal).
- Drawers, modals, sheets, dropdowns, or toasts that open/close without animation.
- Skeleton loaders that don't match the final content layout.
- No `prefers-reduced-motion` support.
- `transition: all` — always name specific properties.
- Animating `width`, `height`, `top`, `left` instead of `transform`.
- Raw hardcoded millisecond values for animation instead of motion tokens.
- Looping decorative background animations in operational or data tools.
- Using `Animated` API in React Native for gesture-driven animation (use Reanimated 2).
- Animations that run every scroll event without Intersection Observer or throttle.

## Forbidden — Mobile

- Touch targets under 44pt (iOS) / 48dp (Android).
- Hover-only controls with no touch equivalent.
- Body text under 14px on mobile.
- Desktop sidebar nav on phone.
- Layouts that do not account for safe areas (notch, home indicator, status bar).
- Gesture with no fallback tap interaction.
- Missing keyboard-aware scroll behavior on forms.

## Forbidden — Library Usage

- Unmodified shadcn card stack as a design system.
- Default MUI, Bootstrap, or Ant Design visual appearance without customization.
- Assuming an imported component library equals a design. It does not.
- Using library defaults for spacing, radius, and color without defining custom tokens.

## Output Contracts

Full UI must include in order:

1. Brief (PRODUCT / PLATFORM / CLASS / SYSTEM / AESTHETIC / LAYOUT / DENSITY / MOTION / KEY STATES / KEY DECISION)
2. Token definitions (color, spacing, type, radius, motion)
3. Layout and navigation structure
4. Primary flow map
5. All relevant states per screen (loading, empty, error, populated, partial, pending, forbidden)
6. Animation and motion spec per component
7. Component specs with props, variants, states, accessibility
8. Responsive behavior across breakpoints
9. Full implementation code (not pseudocode, not described)
10. QA audit with per-dimension scores

Redesign must include:

1. Audit of current state (hierarchy, layout, nav, density, type, color, components, states, interaction, a11y, responsive, copy, product fit)
2. Problems ranked by severity
3. Proposed direction (evolutionary / pivotal / transformative)
4. Layout and structure changes
5. Component and system changes
6. Interaction and animation fixes
7. Full implementation
8. QA audit

Code-only output:

1. Summary of what is being implemented
2. Files / component structure
3. Complete implementation code
4. State, responsive, accessibility, and animation notes

Shorten output for small, scoped tasks but never skip implementation.

## Every Screen Must Answer

- Where am I?
- What matters most here?
- What can I do?
- What is happening right now (loading / processing / error)?
- What just happened (confirmation / result)?
- What should I do next?
- What if something goes wrong?
