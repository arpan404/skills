# Aesthetic Directions

Choose a direction before any visual work. State it explicitly in the output brief. Never default to the generic Inter+purple SaaS aesthetic.

## How to Choose

1. Identify product class, audience, density, and platform.
2. Select the closest catalog direction (or define a hybrid).
3. State the direction, its protected rules, and what it borrows.

Use a hybrid only when the product genuinely needs mixed qualities. Declare it:

```txt
DIRECTION:       [primary direction name] + [secondary trait]
BASE:            [primary direction]
BORROWED TRAITS: [what is taken from secondary]
DO NOT BORROW:   [what must stay from primary]
PROTECTED RULES: [the non-negotiable visual constraints]
WHY:             [one sentence product rationale]
```

Vary composition across tasks. Change at least three of: layout archetype, nav model, density, type choice, palette, component geometry, data presentation, interaction model, media treatment, content rhythm. Never produce the same visual twice.

## Direction Catalog

### Brutal Tool
Stark contrast, sharp geometry, monospace or grotesque type, exposed structure, raw surfaces.
Use for: hacker tools, terminal-inspired UIs, devtools, experimental interfaces, security tools.
Palette: black/white/red, or dark gray + neon accent.
Type: monospace display, grotesque body. Never serif.
Layout: asymmetric, exposed grid, no decorative borders — borders ARE the structure.
Motion: instant or very fast. No decorative animations. State changes are snappy.

### Editorial Serif
Strong serif display, generous vertical rhythm, image-led hierarchy, structured whitespace.
Use for: publications, portfolios, luxury narratives, research pages, journalism.
Palette: off-white/ivory/cream background, ink text, one accent color.
Type: display serif (Georgia, Playfair, Cormorant, Fraunces, etc.) with humanist body sans.
Layout: editorial columns, pull quotes, large imagery, asymmetric feature placement.
Motion: slow, graceful. Long fades (300–500ms). Parallax only when subtle.

### Soft Consumer
Warm palette, rounded controls, friendly spacing, approachable scale.
Use for: wellness, family apps, lifestyle, personal finance, social consumer apps.
Palette: warm pastels or muted earth tones. Single accent.
Type: rounded or humanist sans. No harsh geometric type.
Layout: generous spacing, soft radius, card-light (cards only for objects, not every section).
Motion: spring-based, playful but not excessive. Toggle switches and checkboxes must feel satisfying.

### Data Terminal
Dark or neutral background, monospace accents, dense grids, status color semantics.
Use for: monitoring, infrastructure, logs, trading, SRE, observability.
Palette: dark gray/charcoal base, mono-spaced green/amber/red status colors, minimal accent.
Type: monospace for data and code; compact sans for labels. Small type sizes (11–13px).
Layout: maximum information density. Tables, timelines, status grids, live feeds.
Motion: minimal. Status changes flash briefly. No decorative transitions. Skeleton shimmer optional.

### Luxury Refined
Restrained palette, premium type, large imagery, precise micro-spacing.
Use for: luxury commerce, architecture, premium finance, high-end services.
Palette: near-black/cream/gold or silver accent. One or two colors maximum.
Type: light-weight refined serif or elegant sans. Generous letter-spacing. No bold display.
Layout: structured whitespace, large photography, extreme typographic precision.
Motion: slow and deliberate. Nothing bounces. Fades are long (400–600ms). Micro-transitions are precise.

### Retro Tech
Pixel/CRT/synth references, chunky controls, textured surfaces, playful motion.
Use for: audio tools, retro games, creative instruments, music players, nostalgia products.
Palette: dark with saturated retro accents (orange, green, magenta), or desaturated vintage.
Type: pixel fonts for display, or chunky condensed type. Monospace for data readouts.
Layout: skeuomorphic references, knob-and-slider metaphors, LED display panels.
Motion: springy, physical-feeling. Sound on/off toggle for audio products. Retro easing curves.

### Glass Depth
Translucent layers, blur, luminous accents, depth through frosted surfaces.
Use for: media apps, consumer dashboards, fintech/crypto apps where the brand supports it.
**Do not use** for dense enterprise, operations, or any product where readability is critical.
Palette: dark or rich color base with translucent surface layers. Single bright accent.
Type: medium-weight sans. Sufficient contrast with all translucent backgrounds.
Layout: layered panels, depth hierarchy through blur levels (8px, 16px, 32px).
Motion: smooth, fluid. Surfaces reveal with fade + subtle scale. Spring-based panel entry.

### Swiss Grid
Strict grid discipline, strong alignment, restrained palette, confident typography.
Use for: design-led SaaS, serious product tools, agency sites, design portfolios, systems work.
Palette: black/white with one strong accent. No gradients. No decoration.
Type: geometric or grotesque sans (Neue Haas Grotesk, Aktiv, Inter at correct weight). Grid-locked sizing.
Layout: strict column/row grids. Whitespace is structural. No random margin values.
Motion: precise. Elements move on grid. No playful curves. Easing is standard, not spring.

### Organic Natural
Earth tones, soft geometry, tactile surfaces, handmade quality.
Use for: sustainability, agriculture, wellness, food/beverage, eco products.
Palette: earth tones (ochre, clay, moss, sand), natural whites, no pure black.
Type: humanist sans or mild serif. Imperfect, warm letterforms.
Layout: soft asymmetry, textured backgrounds, natural imagery bleed.
Motion: slow, organic. Elements ease in naturally. No hard snapping or mechanical precision.

### Neon Cyberpunk
Dark base, electric saturated accents, high contrast, glitch aesthetics.
Use for: gaming, esports, security, nightlife, speculative work, crypto, hackathon tools.
Palette: near-black base, electric cyan/magenta/green accents. Neon glow effects.
Type: condensed or techno-styled sans for display. Clean mono for data.
Layout: angular, asymmetric, energetic. Diagonal elements used intentionally.
Motion: fast and electric. Flicker effects, glitch transitions (sparingly). Neon pulsing loops.

### Minimal Functional
Quiet UI, high utility, low ornament, content-first.
Use for: productivity tools, writing apps, note-taking, internal systems, devtools, focus apps.
Palette: white or light gray base, single low-contrast accent, near-black text. No gradients.
Type: clean geometric or transitional sans. No decorative type. System fonts acceptable.
Layout: single-column focus, generous whitespace only where it aids focus, no unnecessary borders.
Motion: instant or near-instant for interactive elements. No entrance animations in content flow.

### Industrial Operations
Compact, rugged, clarity-first, map/table/schedule emphasis.
Use for: construction, logistics, manufacturing, field management, maintenance, facilities.
Palette: gray/slate base, high-saturation status colors (red/amber/green), minimal accent.
Type: compact sans with strong legibility at small sizes. Tabular figures everywhere.
Layout: schedule grids, map + list splits, crew assignment boards. Dense tables.
Motion: minimal. Status color changes flash. Critical alerts pulse (reduced motion must stop pulse).

### Calm Enterprise
Restrained color, predictable layout, high legibility, trust-signaling structure.
Use for: B2B SaaS, HR systems, procurement, finance ops, insurance, legal.
Palette: white/light gray base, blue or neutral accent, semantic status colors.
Type: humanist or transitional sans. Generous line height. No display type in app UI.
Layout: standard sidebar + content. Predictable, no surprises. Tables and forms dominate.
Motion: subtle. Hover states, drawer/modal transitions only. No playful spring animations.

### AI Workspace
Split-pane conversation/artifact/tool trace, trust indicators, transparency panels.
Use for: AI chat apps, coding assistants, agent IDEs, research tools, automation platforms.
Palette: dark or dark-neutral base. Cool accent (blue, violet, teal). Status colors for tool states.
Type: code-friendly mono for artifacts; clean sans for conversation and metadata.
Layout: three-panel: conversation | artifact/editor | tool trace + context. Resizable panes.
Motion: streaming text appears with fade-in. Tool calls animate as they trigger. Progress indicators for long runs. Regenerate/stop button accessible immediately.
