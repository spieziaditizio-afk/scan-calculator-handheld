# Light bento restyle — design spec

## Context

`index.html` (Scan Calculator 0.2) currently uses a dark-OLED theme with neon glow
accents, documented in `CLAUDE.md` as the project's design language. The user
provided a reference screenshot of a different app ("PackCalc — Pallet Scan
Counter") with a light theme, dark navy header, green/orange-red status colors,
monospace box codes (slashed zero), and a bento-card dashboard layout, and asked
for the colors/typography/bento style to be applied to this app.

Decisions confirmed during brainstorming (in priority order):

1. **Switch to a light theme**, overriding the current dark-OLED default. This is
   an explicit, deliberate override of the existing `CLAUDE.md` design rule and
   will be reflected back into `CLAUDE.md` once implemented.
2. **Keep the existing single-column, mobile-first screen structure** (header →
   counters → body → action bar / bottom nav). Do NOT port the reference's
   desktop sidebar+panel layout — it doesn't fit a 5" handheld.
3. **Typography**: use a system monospace font stack for codes/quantities only
   (scan input, pick input, chip "Bxx" labels, edit-overlay quantity field).
   Everything else keeps the current sans-serif stack (`Inter, system-ui, ...`).
   No embedded/downloaded fonts — the app must keep working fully offline.
4. **Scope is visual only** — recolor/retype existing components. Do not import
   reference features that don't exist in this app today (e.g. a "Worker" field,
   a literal "Segregation by box size %" table, history/session UI). The existing
   Optimize "Board" view (already groups boxes by size) gets restyled in place,
   not restructured into a new table.

## Color tokens

Replace the `:root` dark palette with a light palette. Per-screen accent
overrides (`#screen-recon`, `#screen-opt`) keep their *role* (amber = Reconcile,
cyan = Optimize) but use darker shades for AA contrast on white.

| Token | New value | Old value | Notes |
|---|---|---|---|
| `--bg` | `#f3f5f7` | `#07090d` | Page background |
| `--bg2` | `#eef1f4` | `#0b0e14` | |
| `--panel` | `#ffffff` | `#11151c` | Cards, inputs |
| `--panel2` | `#f7f8fa` | `#161c25` | Secondary panel fill |
| `--panel3` | `#eef0f3` | `#1c2330` | Tertiary fill (icon badges, etc.) |
| `--border` | `#e3e6ea` | `#232b36` | |
| `--border2` | `#d4d9df` | `#2e3845` | |
| `--text` | `#16202b` | `#f0f4f8` | |
| `--muted` | `#6b7686` | `#9aa6b2` | |
| `--faint` | `#9aa3af` | `#5b6675` | |
| `--accent` (default/Optimize) | `#0891b2` | `#22d3ee` | Darkened for contrast on white |
| `--cyan` | `#0891b2` | `#22d3ee` | |
| `--sky` | `#0ea5e9` | `#7dd3fc` | |
| `--amber` (Reconcile) | `#d97706` | `#fbbf24` | Darkened for contrast on white |
| `--ok` | `#16a34a` | `#34d399` | |
| `--okbg` | `#eaf7ee` | `#0c2a20` | Light tint, not dark |
| `--danger` | `#dc2626` | `#f87171` | |
| `--dangerbg` | `#fdecea` | `#2c1414` | Light tint, not dark |
| `--header-bg` (new) | `#1b2533` | n/a | Header stays dark, like the reference |
| `--header-text` (new) | `#f5f7fa` | n/a | |

Decorative neon glow effects (`text-shadow`/`box-shadow` with large blur tuned
for black backgrounds) are removed and replaced with flat elevation shadows,
e.g. `0 1px 2px rgba(0,0,0,.05), 0 6px 16px -8px rgba(0,0,0,.08)`.

Hardcoded dark colors embedded directly in component CSS (not via variables)
must also be converted to light equivalents — at minimum:
`.scanfield input` background, `#edit-input` background, `.chip.pull` gradient,
`.opt.sel` background, `.kv` dashed border color, body background radial-gradient
tints, `.flash` toast colors.

## Typography

Add a `--mono` custom property:
`ui-monospace, 'Cascadia Mono', 'Segoe UI Mono', 'Roboto Mono', 'SFMono-Regular', Consolas, monospace`

Apply `font-family:var(--mono)` to:
- `#in-pick`, `#in-rscan`, `#in-target`, `#in-oscan` (and any other scan/qty inputs)
- `.chip .bi` (the "Bxx" box-number label)
- `#edit-input`
- Counter/stat numeric values (`.counter .val`, `.stat .sv`, `.chip .pc`,
  `.banner .bb`) — these already use `font-variant-numeric:tabular-nums`; adding
  monospace reinforces the "scanner readout" feel from the reference.

Everything else (buttons, labels, hints, banner titles, nav, settings sheet)
keeps the existing sans-serif stack unchanged.

## Component mapping (no new components)

| Component | Treatment |
|---|---|
| `header` | Dark (`--header-bg`/`--header-text`), icon buttons adapted for dark-on-light-page contrast |
| `.counters` / `.counter` | White cards, flat shadow, value colors per role (amber/cyan/sky → darkened) |
| `.banner.ok` / `.banner.bad` / `.banner.idle` | Light green / light red-orange / light gray backgrounds, matching reference alert style |
| `.chip`, `.chip.pull`, `.chip.newest` | White cards, light green/accent highlight instead of dark glow |
| `.btn`, `.btn.sec`, `.btn.danger` | Flat fills, no glow; danger button uses light danger bg + dark danger text for contrast |
| `.panel`, `.kv`, `.pct-bar`/`.pct-fill` | White/light-gray panel, light progress track |
| `.band` (Optimize live result) | Light panel with colored left border (accent/ok) |
| `.statgrid`, `.stat`, `.item`, `.palcard` (Optimize Board) | White cards, same data/layout, light shadows |
| `.opt` (settings sheet options), `.sheet` | Light sheet background, selected state uses light accent tint instead of dark `#0c1f22` |
| `.flash` toast | Light background variants of ok/danger |
| Edit overlay (`#edit-overlay`, `#edit-card`, `#edit-input`) | White card over a dimmed backdrop (backdrop stays dark/semi-transparent, unaffected) |
| Print output (`printCss()`) | **Out of scope** — already plain B/W per `CLAUDE.md`, must stay untouched |

## Out of scope

- No new fields, screens, or data (no "Worker" field, no session history, no
  literal % segregation table).
- No change to underlying logic (subset-sum engine, scan routing, localStorage
  schema) — this is a pure CSS/markup-class restyle.
- Print stylesheet (`printCss()`/`doPrint()`) stays plain B/W text, unaffected.

## Acceptance criteria

- All three screens (Home, Reconcile, Optimize) and the Settings/Edit overlays
  render on a light background with the dark header, using the token table
  above — verified visually (`Start-Process index.html`).
- No remaining hardcoded dark hex values from the old palette in component CSS
  (spot-checked via grep for the old hex values list above).
- Inline script still passes `node --check` after the change (no JS was meant
  to change, but markup class changes must not break selectors used in JS).
- `CLAUDE.md` "design = dark OLED" line is updated to reflect the new light
  theme decision, since it's now stale documentation.

## Process note

This project explicitly has no git repository (`CLAUDE.md`: "no build, no deps,
no git"). This spec is saved to disk for review but will **not** be committed,
per the standard brainstorming-skill step — there is no repo to commit to.
