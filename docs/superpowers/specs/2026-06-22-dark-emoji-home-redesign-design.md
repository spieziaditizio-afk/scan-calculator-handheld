# Dark-mode Home redesign (emoji list rows) — design spec

## Context

The user shared a reference screenshot of a different app ("ARROW Warehouse
Tools") with a dark background, a large centered branding header (factory
emoji + app name + tagline + location line), and a vertical list of full-width
rows (colored emoji badge, title, description, trailing chevron) instead of a
card grid. They want this visual style — list rows, emoji icons, dark
background — applied to this app, while keeping the existing light bento-grid
theme (`2026-06-17-light-bento-restyle-design.md`) untouched.

Decisions confirmed during brainstorming (in priority order):

1. **Emoji icons are used**, matching the reference exactly. This is an
   explicit, deliberate override of the current `CLAUDE.md` rule ("Icons:
   inline SVG sprite ... never emoji"), scoped narrowly (see "CLAUDE.md
   update" below) — the same kind of override precedent as the light bento
   restyle overriding the old dark-OLED-default rule.
2. **Scope is the whole app, but the new look only replaces dark mode.** The
   existing light theme (bento grid, slim header, SVG icons) is not touched at
   all. Toggling to dark mode is what reveals the new list-row/emoji look on
   Home; Reconcile/Optimize screens in dark mode keep their current slim
   header structure, just recolored to the dark palette as today.
3. **Home gets a new large branding header in dark mode only** — emoji + app
   name + tagline + `cfg.site` location line — replacing the current slim
   bar. Reconcile/Optimize headers are unchanged structurally in both themes.
4. **Settings and Printer become list rows too** (not just Reconcile/
   Optimize), for a consistent 4-row list in dark mode.

## Header (Home, dark mode only)

Replaces the slim header bar (`<header><span class="brand">...</span>...`)
with a tall centered block, visible only when `data-theme="dark"`:

- 🏭 emoji, large, centered
- "Scan Calculator" — bold, large
- "WAREHOUSE TOOLS" — static tagline, small caps, accent color
- `cfg.site` location line, small, muted — rendered only when `cfg.site` is
  non-empty (uses the existing Settings → Site field, no new config)
- Theme toggle button (`[data-theme-toggle]`, existing moon/sun icon) anchored
  top-right of the block — same handler as today, just repositioned in markup

In light mode (or on Reconcile/Optimize in either theme), the header is
exactly what exists today — no markup or behavior change.

## List rows (Home, dark mode only)

Replaces `.bento` (2 spanning tiles + 2 minitiles) with 4 stacked full-width
rows, visible only under `[data-theme="dark"]`. Each row: colored square
emoji badge (left) · title + description (middle) · "›" chevron (right,
`var(--faint)`, plain character, not a new SVG symbol).

| Row | Emoji | Badge tint | Title | Description | Click handler |
|---|---|---|---|---|---|
| Reconcile | 📦 | amber (`--amber-rgb`) | "Count & verify" | same copy as current `.tile.recon .d` | `setMode('reconcile')` |
| Optimize | 🧮 | cyan (`--cyan`) | "Mix Optimizer" | same copy as current `.tile.optm .d` | `setMode('optimize')` |
| Settings | ⚙️ | neutral gray (`--panel3`-based) | "Settings" | "Printer & data" | `openSettings()` |
| Printer | 🖨️ | sky blue (`--sky`) | "Printer" | current printer label, dynamic (`QLn420`/`A4`) | `openSettings()` |

No numbered kickers ("① RECONCILE") on the dark rows — the reference doesn't
use them and they don't fit the flatter row layout. Kickers stay on the light
bento tiles, unchanged.

## Technical approach

Pure CSS/markup addition, no business-logic changes:

- New markup block added as a sibling to the existing `.bento` block inside
  `.home-body`, and a new tall-header block added as a sibling to the
  existing `<header>` on the Home screen. Both new blocks are `display:none`
  by default.
- `[data-theme="dark"] .bento, [data-theme="dark"] header.home-slim
  {display:none}` and the inverse (`[data-theme="dark"] .home-list,
  [data-theme="dark"] .home-header-dark {display:flex}`) toggle visibility —
  consistent with the existing `[data-theme="dark"] <selector>` override
  pattern already used throughout the stylesheet.
- New row elements get their own IDs (`#row-recon`, `#row-opt`,
  `#row-settings`, `#row-printer`) since IDs must be unique; each is wired to
  the **same** handler already bound to its bento counterpart (e.g.
  `$('tile-recon').onclick = $('row-recon').onclick = ()=>setMode('reconcile')`).
  No new functions, no duplicated logic.
- The existing dynamic printer-label update (`$('home-printer-lbl').textContent
  = ...`) is extended to also set the new row's label element
  (`home-printer-lbl2`), at the same call site.
- Emoji are inserted as plain text characters in markup — no new sprite
  symbols, no JS changes to icon rendering.

## CLAUDE.md update

The "Icons: ... never emoji" rule is narrowed, not removed: emoji are
permitted **only** in the Home screen's dark-mode list rows (the 4 rows
above). Everywhere else — Reconcile, Optimize, the light-mode bento tiles,
and the existing SVG sprite — stays SVG-only, unchanged. This mirrors how the
bento-restyle spec updated the stale "dark OLED default" line in `CLAUDE.md`
once implemented.

## Out of scope

- No changes to business logic, localStorage schema, or the Reconcile/Optimize
  screens beyond their existing dark-mode recoloring.
- No new config field for an "area" sub-line (like the reference's "MICROSOFT
  AREA") — only the existing `cfg.site` is used.
- Light theme (bento grid, slim header, SVG-only icons) is untouched in every
  respect.
- Print stylesheet (`printCss()`) is unaffected — already plain B/W, out of
  scope per existing project rules.

## Acceptance criteria

- In light mode, Home renders exactly as it does today (bento grid, slim
  header) — pixel-for-pixel unchanged markup path.
- In dark mode, Home renders the tall branding header (showing `cfg.site`
  only when set) and the 4-row emoji list instead of the bento grid.
- All 4 new rows route to the same destinations as their bento counterparts
  (Reconcile, Optimize, Settings sheet, Settings sheet) — verified by reading
  the click-handler wiring.
- Reconcile/Optimize screens are visually unaffected by this change in either
  theme.
- `node --check` passes on the inline script after the change.
- `CLAUDE.md`'s "never emoji" line is updated to scope the exception to the
  Home dark-mode rows.
