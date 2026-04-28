# Quality and Tokens

## Self-Audit

Score every dimension 1–10 before delivery. Do not skip any dimension. Report the score.

| Dimension              | Score | Fail threshold |
|------------------------|-------|----------------|
| Product fit            | /10   | < 8 → revise   |
| Visual hierarchy       | /10   | < 8 → revise   |
| Layout originality     | /10   | < 7 → revise   |
| Information architecture | /10 | < 7 → revise   |
| Interaction depth      | /10   | < 8 → revise   |
| State completeness     | /10   | < 8 → revise   |
| Animation & motion     | /10   | < 7 → revise   |
| Accessibility          | /10   | < 8 → revise   |
| Responsiveness         | /10   | < 8 → revise   |
| Implementation quality | /10   | < 8 → revise   |
| Aesthetic consistency  | /10   | < 8 → revise   |

**Revise immediately if any dimension is below its threshold.** Do not deliver a failing score.

## QA Checklist

Run through every item before delivery:

### Tokens
- [ ] Motion tokens defined (`--motion-fast`, `--motion-normal`, `--motion-enter`, etc.)
- [ ] Color tokens defined with semantic naming
- [ ] Type scale tokens defined
- [ ] Spacing tokens defined
- [ ] Radius tokens defined
- [ ] No raw hardcoded values where tokens should be used

### Visual
- [ ] Custom visual direction applied — not default library appearance
- [ ] Coherent type scale and spacing rhythm
- [ ] Semantic colors with clear intent
- [ ] No card spam — layout archetype fits the product
- [ ] Clear primary action hierarchy per screen

### States
- [ ] Loading state implemented (skeleton matches content layout)
- [ ] Empty state implemented (headline + reason + action)
- [ ] Error state implemented (message + recovery action)
- [ ] Populated state with realistic sample data (no lorem ipsum)
- [ ] Partial/offline/forbidden states where relevant
- [ ] Pending/async state for every mutating action

### Animation and Motion
- [ ] Motion tokens used (no raw `ms` values inline)
- [ ] Every button has hover + active CSS transition
- [ ] Every input has focus transition (border + shadow)
- [ ] All dropdowns, tooltips, popovers have enter/exit animation
- [ ] All drawers, sheets, modals have slide/fade enter/exit animation
- [ ] Toasts have enter/exit animation
- [ ] Skeleton uses shimmer animation
- [ ] Page or route transitions implemented
- [ ] Tab switches animated
- [ ] Accordion/collapsible animated (grid-template-rows or max-height)
- [ ] `prefers-reduced-motion` handled — all durations collapse to 0ms
- [ ] No `transition: all` — properties named explicitly
- [ ] No `width/height/top/left` animation — `transform` used instead
- [ ] React Native: Reanimated 2 used for gesture-driven animations

### Accessibility
- [ ] All inputs have visible labels
- [ ] All icon-only buttons have `aria-label`
- [ ] Focus rings visible and styled
- [ ] No `outline: none` without custom focus replacement
- [ ] Tab order is logical
- [ ] Modals trap focus and restore focus on close
- [ ] Keyboard: Tab/Enter/Space/Escape/arrows all work correctly
- [ ] Color is not the only signal for state
- [ ] Contrast passes WCAG AA (4.5:1 text, 3:1 UI components)
- [ ] Errors are tied to their input or action, not just a global message

### Responsiveness
- [ ] Tested at 375px — no overflow, no broken layout
- [ ] Tested at 430px
- [ ] Tested at 768px — layout adapts (nav, panels, density)
- [ ] Tested at 1024px
- [ ] Tested at 1280px
- [ ] Tested at 1440px+
- [ ] Mobile: no hover-only controls
- [ ] Mobile: touch targets ≥ 44pt / 48dp
- [ ] Mobile: body text ≥ 14px
- [ ] Mobile: safe areas respected
- [ ] No horizontal overflow at any width

### Code Quality
- [ ] Semantic HTML (no divs for interactive elements)
- [ ] No console errors
- [ ] No `alert()` / `confirm()` for dialogs
- [ ] Realistic sample data used throughout
- [ ] Components organized and reusable
- [ ] State managed correctly (local / URL / server cache / global)

## Token Template

Define these in every project. Extend as needed for the specific aesthetic and domain.

```css
:root {
  /* Typography */
  --font-display: system-ui, sans-serif;
  --font-body:    system-ui, sans-serif;
  --font-mono:    ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;

  --text-xs:   11px;
  --text-sm:   13px;
  --text-base: 15px;
  --text-md:   17px;
  --text-lg:   20px;
  --text-xl:   24px;
  --text-2xl:  30px;
  --text-3xl:  38px;

  /* Spacing */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-7: 32px;
  --space-8: 40px;
  --space-9: 56px;

  /* Radius */
  --radius-sm:   3px;
  --radius-md:   6px;
  --radius-lg:   10px;
  --radius-xl:   16px;
  --radius-full: 9999px;

  /* Motion */
  --motion-instant: 50ms;
  --motion-fast:    100ms;
  --motion-normal:  200ms;
  --motion-enter:   250ms;
  --motion-exit:    180ms;
  --motion-page:    320ms;
  --motion-slow:    450ms;

  --ease-standard:   cubic-bezier(0.2, 0, 0, 1);
  --ease-decelerate: cubic-bezier(0, 0, 0.2, 1);
  --ease-accelerate: cubic-bezier(0.4, 0, 1, 1);
  --ease-spring:     cubic-bezier(0.34, 1.56, 0.64, 1);
  --ease-out:        cubic-bezier(0.16, 1, 0.3, 1);
  --ease-in-out:     cubic-bezier(0.4, 0, 0.2, 1);

  /* Semantic Colors */
  --color-bg:          #ffffff;
  --color-surface:     #f8fafc;
  --color-surface-2:   #f1f5f9;
  --color-text:        #0f172a;
  --color-text-muted:  #475569;
  --color-text-subtle: #94a3b8;
  --color-border:      #cbd5e1;
  --color-border-subtle: #e2e8f0;
  --color-accent:      #f97316;
  --color-accent-subtle: color-mix(in srgb, var(--color-accent) 12%, transparent);
  --color-success:     #16a34a;
  --color-warning:     #d97706;
  --color-error:       #dc2626;
  --color-info:        #2563eb;
}

/* Reduced motion override — always include */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration:       0.01ms !important;
    animation-iteration-count: 1     !important;
    transition-duration:      0.01ms !important;
    scroll-behavior:          auto   !important;
  }
}
```
