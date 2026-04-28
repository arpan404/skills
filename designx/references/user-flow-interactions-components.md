# Flows, Interactions, Components

## Flow

Map every screen:

```txt
Entry → primary intent → main action → async feedback → result → next action → recovery path
```

Define for each screen:

- user goal
- primary action (one per screen scope)
- secondary actions
- destructive actions (require confirmation or undo)
- async actions (must show pending state)
- error conditions and recovery
- empty / loading / partial / forbidden states
- next best action after completion

Navigation:

- Few destinations (≤ 5): top nav bar or bottom tab bar.
- Many destinations: sidebar or rail with collapse.
- Hierarchical content: tree + sidebar.
- Dense complex app: command palette + persistent nav.
- Mobile: bottom tabs for top-level, stack navigation for depth.
- Canvas/tool: floating toolbar + fixed panels + inspector drawer.

Multi-screen patterns:

- Onboarding/setup: focused stepper or wizard flow. One decision per step.
- Record browsing: master-detail split. No full-page navigation for detail.
- Editing: editor shell or inspector panel. Autosave or explicit save/cancel.
- Approval/triage: inbox queue with priority, assignment, and bulk actions.
- Monitoring: command center with live data, alerts first, drill-down available.
- Creation (simple): modal or drawer with form.
- Creation (complex): full-page wizard or multi-step editor.

## Interaction

### Timing (use motion tokens, not raw values)

| Action                     | Duration token        |
|----------------------------|-----------------------|
| Hover / focus ring         | `--motion-fast`       |
| Button press               | `--motion-fast`       |
| Dropdown open              | `--motion-normal`     |
| Drawer / panel open        | `--motion-enter`      |
| Drawer / panel close       | `--motion-exit`       |
| Modal appear               | `--motion-enter`      |
| Toast enter                | `--motion-enter`      |
| Page / route transition    | `--motion-page`       |
| Tab switch                 | `--motion-normal`     |

Never use slow decorative motion for interactive transitions.
Never use `transition: all`.
Always use `transform` and `opacity` — never animate `width`, `height`, `top`, `left`.

### Feedback

- Every click or tap must produce a visible response within one frame.
- Async actions: show a pending/spinner state immediately on trigger.
- Destructive actions: require confirmation dialog or offer immediate undo.
- Errors: explain the cause and provide a clear recovery action.
- Optimistic updates: show result immediately, roll back on failure with error explanation.
- Form save: show saving → saved → (error if failed) state clearly.

### Required Micro-Interactions

Every one of these must be animated:

| Element                  | Animation requirement                                          |
|--------------------------|----------------------------------------------------------------|
| Button hover             | subtle lift (`translateY(-1px)`) or color transition          |
| Button active/press      | scale down (`scale(0.97–0.98)`)                               |
| Button loading           | keep label + show spinner; disable pointer events             |
| Input focus              | border color + focus ring shadow transition                   |
| Input validation         | error border + message fade-in                                |
| Checkbox check           | SVG stroke-dashoffset draw animation                          |
| Toggle / switch          | thumb `translateX` + track color, spring easing               |
| Row hover                | background color transition                                   |
| Row select               | accent background transition                                  |
| Drag pickup              | `scale(1.04)` + shadow elevation                              |
| Drag over target         | highlight target zone with animated border/background          |
| Drop                     | spring settle to `scale(1.0)`                                 |
| Drawer open              | `translateX` or `translateY` from off-screen                  |
| Drawer close             | reverse translate, faster duration                            |
| Modal appear             | fade + `translateY(8px→0)` + backdrop fade                    |
| Modal dismiss            | reverse, faster                                               |
| Dropdown open            | fade + `translateY(4px→0)` + `scale(0.96→1.0)`                |
| Toast enter              | slide from edge + fade                                        |
| Toast exit               | slide back + fade                                             |
| Tab switch               | indicator slides to active tab                                |
| Accordion open/close     | CSS grid-template-rows or max-height transition               |
| Skeleton loading         | shimmer animation matching content layout                     |
| Upload progress          | smooth progress bar width transition                          |
| Save/pending/saved       | icon/text morphs through three states                         |
| Filter applied           | chip appears with fade-in; active count badge updates          |

### Forms

- Always use visible labels above inputs. Never placeholder-only.
- Inline validation on blur (not on every keystroke).
- Clear error messages tied to the specific field.
- Required/optional distinction visible without asterisk spam.
- Input masks for phone, credit card, date, currency when helpful.
- Long forms: error summary at top on failed submit.
- Save/cancel state for non-modal editing forms.
- Unsaved-change warning before navigation.
- Keyboard submit via Enter on single-line inputs.
- Disabled state must include tooltip or helper text explaining why.

## Components

Every reusable component must define:

- **Purpose**: what problem does it solve
- **Props/API**: what it accepts, defaults, required vs optional
- **Variants**: size, visual style, intent (primary/secondary/destructive)
- **States**: default, hover, focus, active, loading, disabled, error, selected, empty
- **Responsive behavior**: what changes at mobile breakpoints
- **Accessibility**: role, aria attributes, keyboard behavior, focus management
- **Animation**: enter, exit, and state-change transitions

### Typography Scale

- App body: 14–16px (standard), 13–15px (dense)
- Mobile body: 15–17px
- Marketing body: 16–19px
- Use tabular figures (`font-variant-numeric: tabular-nums`) for all numeric data.
- Avoid ALL-CAPS for more than 2–3 words. Never for body text.
- Use semantic text color tokens: text, muted, subtle, on-accent, error.

### Buttons

```txt
Primary      — one per scope. Clear verb label. Loading state keeps label.
Secondary    — alternative non-destructive action.
Tertiary/ghost — low-emphasis. Often for cancel.
Destructive  — red/danger. Confirm before executing.
Icon-only    — must have aria-label. Show tooltip on hover/focus.
```

All buttons: hover (lift or color), active (scale-down), focus ring, loading spinner, disabled (muted + no pointer events + tooltip why).

### Inputs

- Visible label above the field (not inside).
- Animated focus state (border + glow ring).
- Animated validation state (border + icon + helper text fade-in).
- Disabled and read-only must look visually distinct.
- Prefix/suffix for units, currency, or icons.
- Search inputs: clear (×) button fades in when value is present.

### Navigation

- Always show current location (active state must not rely on color alone).
- Animated active indicator: slide, underline, or highlight transition.
- Keyboard: arrows navigate items in sidebar/tabs/menus.
- Breadcrumbs for hierarchy deeper than 2 levels.
- Preserve nav selection across page refreshes (URL state or localStorage).

### Feedback Components

- **Toast**: non-critical confirmation. Auto-dismiss 4–6s. Pause on hover. Stack with stagger. Enter/exit animated.
- **Inline error**: tied to specific field or action. Fade-in on error. Disappear on fix.
- **Banner**: persistent page/system issue. Dismissible. High-contrast for critical.
- **Modal**: blocking decision. Overlay backdrop animated. Focus trapped. ESC closes. Animated enter/exit.
- **Empty state**: centered, plain headline, one useful sentence, one primary action. No clip art unless product uses illustrations.
- **Skeleton**: shimmer animation. Must exactly match the shape and dimensions of the loaded content.
- **Progress**: bar transitions width smoothly. Spinner rotates continuously. Completion animates to 100% then fades.

### Data Display

- **Table**: for comparison, operations, bulk actions, sorting. Use for most data-dense scenarios.
- **Cards**: only for browsable object grids where visual identity of the object matters.
- **List**: for feeds, queues, search results.
- **Timeline**: for chronological history or activity log.
- **Map**: only when location is central to the task.
- **Chart**: only to show patterns, trends, or composition — not as decoration. Animate draw-in on mount.
