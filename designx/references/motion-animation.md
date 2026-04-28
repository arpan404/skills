# Motion and Animation

Motion is a design material. Every transition, entrance, exit, and micro-interaction must serve a purpose: orient the user, confirm an action, signal state, or reduce perceived latency. Never add motion for decoration alone.

## Principles

- **Purposeful**: motion must communicate something — position, hierarchy, cause, result.
- **Responsive**: motion must react immediately to input. Do not delay the start of a transition.
- **Natural**: prefer spring/easing curves over linear. Objects accelerate and decelerate.
- **Consistent**: same action → same motion everywhere. Build a motion token set.
- **Respectful**: honor `prefers-reduced-motion`. Provide instant or minimal alternatives.

## Motion Token Set

Define these tokens and use them everywhere. Never use raw `ms` or `cubic-bezier` values inline.

```css
:root {
  /* Durations */
  --motion-instant:    50ms;
  --motion-fast:      100ms;
  --motion-normal:    200ms;
  --motion-enter:     250ms;
  --motion-exit:      180ms;
  --motion-page:      320ms;
  --motion-slow:      450ms;

  /* Easing curves */
  --ease-standard:    cubic-bezier(0.2, 0, 0, 1);       /* Material standard */
  --ease-decelerate:  cubic-bezier(0, 0, 0.2, 1);       /* elements entering */
  --ease-accelerate:  cubic-bezier(0.4, 0, 1, 1);       /* elements leaving */
  --ease-spring:      cubic-bezier(0.34, 1.56, 0.64, 1); /* bouncy/elastic */
  --ease-out:         cubic-bezier(0.16, 1, 0.3, 1);    /* quick settle */
  --ease-in-out:      cubic-bezier(0.4, 0, 0.2, 1);     /* panels, drawers */

  /* Reduced motion overrides */
  @media (prefers-reduced-motion: reduce) {
    --motion-fast:   0ms;
    --motion-normal: 0ms;
    --motion-enter:  0ms;
    --motion-exit:   0ms;
    --motion-page:   0ms;
    --motion-slow:   0ms;
  }
}
```

## Duration Guidelines

| Interaction type              | Duration           | Easing               |
|-------------------------------|--------------------|----------------------|
| Hover / focus ring            | 80–120ms           | ease-out             |
| Button press / ripple         | 100–150ms          | ease-out             |
| Tooltip appear                | 100ms              | ease-decelerate      |
| Tooltip disappear             | 80ms               | ease-accelerate      |
| Checkbox / toggle / switch    | 150–180ms          | ease-spring          |
| Dropdown open                 | 150–200ms          | ease-decelerate      |
| Dropdown close                | 120–150ms          | ease-accelerate      |
| Drawer / sheet open           | 220–280ms          | ease-decelerate      |
| Drawer / sheet close          | 180–220ms          | ease-accelerate      |
| Modal appear                  | 200–250ms          | ease-decelerate      |
| Modal dismiss                 | 160–200ms          | ease-accelerate      |
| Toast / snackbar enter        | 250ms              | ease-decelerate      |
| Toast / snackbar exit         | 200ms              | ease-accelerate      |
| Page / route transition       | 280–350ms          | ease-in-out          |
| Tab switch                    | 180–220ms          | ease-standard        |
| Accordion open/close          | 200–240ms          | ease-standard        |
| Skeleton pulse                | 1200ms (loop)      | ease-in-out          |
| List item stagger (per item)  | 30–50ms delay      | ease-decelerate      |
| Shared element transition     | 300–400ms          | ease-spring          |
| Scroll-driven entrance        | 300–500ms          | ease-out             |
| Number/counter animation      | 600–800ms          | ease-out             |
| Chart draw-in                 | 400–700ms          | ease-out             |

Never exceed 500ms for interactive responses. Never exceed 700ms for page transitions.
Decorative animations may be slower but must be opt-in and reduced-motion safe.

## Entrance and Exit Patterns

### Fade + Translate (most common)
```css
/* Enter */
@keyframes fadeSlideIn {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Exit */
@keyframes fadeSlideOut {
  from { opacity: 1; transform: translateY(0); }
  to   { opacity: 0; transform: translateY(4px); }
}
```
Use for: dropdowns, tooltips, popovers, toasts, modals (vertical slide), inline messages.

### Scale + Fade (contextual menus, popovers anchored to a trigger)
```css
@keyframes scaleIn {
  from { opacity: 0; transform: scale(0.94); transform-origin: top left; }
  to   { opacity: 1; transform: scale(1); }
}
```
`transform-origin` must match the anchor point of the popover relative to its trigger.

### Slide (drawers, sheets, sidebars)
```css
/* Right drawer */
@keyframes slideInRight {
  from { transform: translateX(100%); }
  to   { transform: translateX(0); }
}

/* Bottom sheet */
@keyframes slideInUp {
  from { transform: translateY(100%); }
  to   { transform: translateY(0); }
}
```

### Page / Route Transitions
- Web: fade + subtle Y translate (6–12px). Never full-page slide unless it's a native app shell.
- Mobile web / React Native: horizontal slide for stack push/pop; fade for tab switches.
- Shared element: scale and position the shared element while content fades.

```css
/* Web route enter */
@keyframes routeEnter {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

## Micro-Interaction Patterns

### Button

```css
button {
  transition: background-color var(--motion-fast) var(--ease-out),
              box-shadow var(--motion-fast) var(--ease-out),
              transform var(--motion-fast) var(--ease-out);
}
button:hover  { transform: translateY(-1px); }
button:active { transform: translateY(0) scale(0.98); }
```

### Checkbox / Radio
```css
/* Check mark draw animation */
@keyframes checkDraw {
  from { stroke-dashoffset: 24; }
  to   { stroke-dashoffset: 0; }
}
```

### Toggle / Switch
- Track color transitions in `--motion-normal`.
- Thumb slides with `transform: translateX()` using `--motion-normal` + `--ease-spring`.

### Input Focus
```css
input {
  transition: border-color var(--motion-fast) var(--ease-out),
              box-shadow var(--motion-fast) var(--ease-out);
}
input:focus {
  border-color: var(--color-accent);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--color-accent) 20%, transparent);
}
```

### Skeleton Loading
```css
@keyframes shimmer {
  from { background-position: -400px 0; }
  to   { background-position: 400px 0; }
}
.skeleton {
  background: linear-gradient(
    90deg,
    var(--color-surface) 25%,
    color-mix(in srgb, var(--color-border) 60%, transparent) 50%,
    var(--color-surface) 75%
  );
  background-size: 800px 100%;
  animation: shimmer 1.2s var(--ease-in-out) infinite;
}
```
Skeletons must match the exact shape and layout of loaded content.

### Progress / Loading Spinner
- Spinner rotation: 700–900ms linear infinite.
- Indeterminate progress bar: use `scaleX` + translate on a pseudo-element.
- Determinate progress: transition `width` or `stroke-dashoffset`.

### Notification / Toast
- Enter from bottom-right (desktop) or top-center (mobile) with slide + fade.
- Auto-dismiss after 4–6 seconds.
- Pause dismiss on hover.
- Stack multiple toasts with stagger.

### Drag and Drop
- Pickup: scale to 1.04, slight shadow increase, 150ms ease-out.
- Over valid target: target gets highlight with fade-in, 100ms.
- Drop: spring settle to 1.0, 200ms ease-spring.
- Invalid drop: shake (horizontal translate oscillation, 300ms).
- Reorder: smooth position transition using `transform`, 200ms.

### Number Counter / Value Change
- When a metric updates, animate from old to new value.
- Use a counting animation (requestAnimationFrame loop) for large jumps.
- Flash background briefly (success/warning color, 400ms fade) for significant changes.

### Row Selection / Highlight
```css
tr {
  transition: background-color var(--motion-fast) var(--ease-out);
}
tr:hover     { background-color: var(--color-surface-hover); }
tr.selected  { background-color: var(--color-accent-subtle); }
```

### Accordion / Collapsible
- Animate `max-height` from 0 to content height OR use CSS Grid row trick.
- Grid approach (preferred, no JS height calculation):
```css
.accordion-body {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows var(--motion-normal) var(--ease-standard);
}
.accordion-body.open {
  grid-template-rows: 1fr;
}
.accordion-body > div { overflow: hidden; }
```

## Scroll-Driven and Entrance Animations

Use sparingly. Never animate content that blocks reading.

```css
/* Intersection observer entrance */
.animate-on-scroll {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity var(--motion-slow) var(--ease-out),
              transform var(--motion-slow) var(--ease-out);
}
.animate-on-scroll.visible {
  opacity: 1;
  transform: translateY(0);
}
```

CSS scroll-driven (modern browsers):
```css
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(16px); }
  to   { opacity: 1; transform: translateY(0); }
}
.card {
  animation: fadeUp linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 40%;
}
```

Rules:
- Stagger sibling items: 30–60ms delay between each.
- Only animate items in the viewport, not off-screen.
- Never block content behind animation — use `opacity` only for entrance, not `display: none` during transition.

## Chart and Data Animations

- Bar charts: `scaleY` from 0 to 1 on bar elements, 400–600ms, ease-out, stagger bars by 30ms.
- Line charts: `stroke-dashoffset` draw-in, 500–700ms, ease-out.
- Pie/donut: `stroke-dashoffset` draw-in per segment, staggered.
- Number counters: count up from 0 or previous value on mount/update.
- Chart update (new data): transition bar heights and line paths, not a full redraw.

## Animation Libraries (reference)

Use when native CSS is insufficient or cross-platform is needed.

| Library           | Best for                                                  |
|-------------------|-----------------------------------------------------------|
| CSS Animations    | Simple transitions, micro-interactions, skeleton loading  |
| Framer Motion     | React: layout animations, shared elements, gestures       |
| React Spring      | React: physics-based spring animations                    |
| GSAP              | Complex timelines, SVG, canvas, scroll-driven sequences   |
| Lottie            | Designer-created JSON animations (icons, loaders, etc.)   |
| Motion (Svelte)   | Svelte tweened/spring stores                              |
| Reanimated 2      | React Native: gesture + animation (mandatory for native)  |
| Moti              | React Native + Expo: declarative Reanimated wrapper        |

**Rules when using libraries:**
- Wrap library usage behind a single abstraction so it can be swapped.
- Never import an animation library for a single transition you can do in CSS.
- Always test library animations with `prefers-reduced-motion`.
- Reanimated 2 is mandatory for any gesture-driven or 60fps native mobile animation.

## Mobile-Specific Animation

### React Native
- Use `Animated` API only for simple cases. Use **Reanimated 2** for any gesture-driven or 60fps animation.
- Use `useSharedValue`, `useAnimatedStyle`, `withSpring`, `withTiming`.
- Platform-matched transitions:
  - iOS: stack push = horizontal slide right; modal = slide up from bottom.
  - Android: stack push = fade + slight Y translate (Material); modal = slide up.
- Gesture-driven animations (`react-native-gesture-handler`):
  - Swipe to delete: translate X, reveal action behind, spring snap.
  - Pull to refresh: translate Y, spinner rotation.
  - Swipe between tabs: horizontal translate, follow finger exactly.
  - Bottom sheet drag: translate Y, follow finger, spring to snap points.

### Expo
- Use `expo-reanimated` (Reanimated 2) for all animations.
- Use `expo-haptics` to pair animations with haptic feedback on significant events.
- Use `react-native-screens` native stack for real platform navigation animations.

### Gesture Animation Rules
- Animation must follow the gesture 1:1 (no lag on finger position).
- Spring values: `mass: 1`, `damping: 18–22`, `stiffness: 200–300` as starting point.
- Always define snap points and handle gesture interruption gracefully.
- Provide fallback tap interaction for any swipe gesture.

## Reduced Motion

**This is a hard requirement, not optional.**

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

For React / JS animations, check the media query:
```js
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
```

In Framer Motion:
```jsx
import { useReducedMotion } from 'framer-motion';
const reduced = useReducedMotion();
```

Reduced motion alternative: instant state change, no opacity fade, no transform. Preserve functionality — never disable a state transition entirely, only remove the motion.

## Animation Anti-Patterns

Never do these:

- Animate `width`, `height`, `top`, `left` — always use `transform` instead.
- Animate `margin` or `padding` for layout effects.
- `transition: all` — always specify properties explicitly.
- Run animations on every scroll event without throttling or Intersection Observer.
- Use `setTimeout` to fake animation completion — use `animationend` / `transitionend`.
- Bounce or shake for every interaction — reserve for errors or emphasis.
- Animate elements the user didn't interact with.
- Looping decorative background animations on operational tools.
- Apply animation only via JavaScript class toggling without a CSS fallback.
- Forget to clean up animation event listeners or `requestAnimationFrame` loops.
- Use animation to hide slow renders — fix the render, not the symptom.
