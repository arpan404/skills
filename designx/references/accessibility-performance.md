# Accessibility and Performance

## Accessibility

### Structure
- Use semantic HTML: `<button>`, `<a>`, `<nav>`, `<main>`, `<section>`, `<header>`, `<footer>`, `<article>`, `<aside>`, `<ul>`, `<ol>`, `<table>`.
- Never use click handlers on `<div>` or `<span>`. Use `<button>` for actions, `<a>` for navigation.
- Use ARIA roles and attributes only when native semantics are insufficient.
- Do not override native semantics with ARIA unless you are replacing native behavior entirely.

### Labels and Identity
- Every `<input>`, `<select>`, and `<textarea>` must have a `<label>`. `for` + `id` or wrapping label.
- Never use placeholder text as the sole label. Placeholder disappears on input.
- Every icon-only button needs `aria-label` and a `title` attribute (shows as tooltip).
- Every image needs `alt` text. Decorative images: `alt=""`.
- Every chart or data visualization needs a text summary or accessible table.

### Focus
- All interactive elements must be keyboard-reachable via `Tab` and `Shift+Tab`.
- Never use `outline: none` or `outline: 0` without providing a visible custom focus ring.
- Focus ring style: at minimum `2px solid` in a color that contrasts with both background and element.
- Logical focus order that matches visual reading order.
- Modals and drawers must trap focus inside while open.
- When a modal closes, focus must return to the element that triggered it.

### Keyboard Behavior
- `Tab` / `Shift+Tab`: move between interactive elements.
- `Enter` / `Space`: activate buttons, checkboxes, toggles.
- `Escape`: close dropdowns, tooltips, modals, drawers, command palettes.
- Arrow keys: navigate within menus, tab lists, listboxes, grids, sliders.
- `Home` / `End`: jump to first/last item in a list or grid.
- Keyboard shortcuts must not override browser or OS shortcuts.

### Color and Contrast
- Text on background: 4.5:1 minimum (WCAG AA). 7:1 for AAA on critical content.
- UI components (buttons, inputs, icons used as controls): 3:1 minimum.
- Color must never be the only signal for state (error, success, warning, disabled, selected).
- Pair color with: icon, text label, pattern, shape, or weight change.
- Test in forced colors mode (Windows High Contrast) for critical tools.

### Dynamic Content
- Use `aria-live="polite"` for async content updates (search results, notifications).
- Use `aria-live="assertive"` only for critical alerts.
- Toasts and snackbars must be announced to screen readers.
- Loading states: use `aria-busy="true"` on the region being loaded.
- Errors must be programmatically associated with their input via `aria-describedby`.

### Reduced Motion
- Always include `@media (prefers-reduced-motion: reduce)` that collapses animation durations.
- Check `window.matchMedia('(prefers-reduced-motion: reduce)')` in JS.
- In Framer Motion: use `useReducedMotion()` hook.
- In React Native: use `AccessibilityInfo.isReduceMotionEnabled()`.
- Reduced motion means no transforms, no fades — instant state change. Never disable the state change itself.

## Performance

### Rendering
- Avoid unnecessary re-renders. Memoize only when profiling confirms a problem.
- Virtualize all long lists and tables: `react-window`, `react-virtual`, `FlashList` (RN).
- Lazy-load heavy routes, images, and media. Use `React.lazy` + `Suspense`.
- Keep component trees shallow. Avoid deeply nested provider chains.

### Animation Performance
- **Only animate `transform` and `opacity`**. These are GPU-composited and do not cause layout.
- Never animate `width`, `height`, `top`, `left`, `margin`, `padding` — these trigger layout reflow.
- Use `will-change: transform` only on elements actively animating. Remove it after animation ends.
- Use `contain: layout paint` on animated container regions to isolate reflow.
- Prefer CSS animations and transitions over JS-driven animation for simple cases.
- For JS animation: use `requestAnimationFrame`. Never `setTimeout` / `setInterval` for animation loops.
- Target 60fps. Profile with Chrome DevTools Performance tab or React Native Flipper.
- Test on low-end device or CPU throttle (4x) in DevTools before shipping.

### Images and Media
- Use `srcset` and `sizes` for responsive images.
- Use `loading="lazy"` for below-fold images.
- Use `width` and `height` attributes on `<img>` to prevent layout shift (CLS).
- Use modern formats: WebP, AVIF with `<picture>` fallback.
- Never ship unoptimized images or uncompressed SVGs.

### Loading Strategy
- Skeletons must match the final layout shape exactly — same dimensions, same grid.
- Skeletons use shimmer animation (GPU-composited background-position animation).
- Handle: slow network (≥ 3s), failed fetch, partial data, stale data.
- Stale-while-revalidate: show cached content immediately, update in background.
- Never block the entire UI for a single failed request.
- Show partial data when it's useful — do not withhold loaded data while other parts load.

### Web Vitals
- Target: LCP < 2.5s, INP < 200ms, CLS < 0.1.
- Avoid layout shifts from: unset image dimensions, fonts loading, dynamic content injection.
- Use `font-display: swap` or `optional` for web fonts.
- Preload critical above-fold fonts and LCP image.
- No console errors in production.
