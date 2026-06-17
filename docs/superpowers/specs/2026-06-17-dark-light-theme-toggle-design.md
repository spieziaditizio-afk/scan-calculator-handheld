# Dark/light theme toggle — design spec

## Context

`index.html` was just fully restyled from its original dark-OLED palette to a light
bento-grid theme (see `2026-06-17-light-bento-restyle-design.md` and its plan,
already implemented). The user now wants **both** themes available with a toggle,
defaulting to the new light theme. The original dark-OLED CSS is fully known —
it's documented verbatim in the restyle plan's "Old code" blocks — so dark mode is
a restoration, not a new design.

Decisions confirmed during brainstorming:

1. **Toggle control**: an `.iconbtn` in the header of all three screens (Home,
   Reconcile, Optimize), alongside Settings/Guide.
2. **Dark palette**: reuse the **exact** original OLED values, including the glow
   effects (text-shadow/box-shadow) that were stripped out during the light
   restyle — not a new dark variant tuned to the new light theme's flatter look.
3. **Header in dark mode**: stops being the fixed dark-navy hybrid bar and merges
   back into the OLED background (`var(--panel)`-based), exactly like the original
   pre-restyle header. The fixed navy header is light-mode-only.
4. **Default & persistence**: light on first run; the user's choice is saved to
   `cfg.theme` in the existing `packcalc_cfg` localStorage key (alongside
   `printer`/`mode`) and restored on next load. No `prefers-color-scheme` lookup.

## Technical approach

Add `data-theme="dark"` to `<html>` (absent/`"light"` = light, the default). Add
one new override block `:root[data-theme="dark"]{...}` for token values, plus a
small set of selector-specific overrides under `[data-theme="dark"] <selector>`
for the ~20 places that have a literal color baked in (not a `var(--token)`),
because those literals were written assuming a light background and would be
wrong (e.g. invisible or low-contrast) on a dark one.

Rejected alternatives:
- **Duplicate the whole stylesheet per theme** — doubles ~280 lines for no
  benefit; almost every rule already reads from `var(--token)` and needs zero
  duplication.
- **Move all tokens into JS and set them via `style.setProperty`** — moves styling
  out of CSS for no gain; the existing per-screen accent overrides
  (`#screen-recon{--accent:...}`) already establish the "CSS variable override
  block" pattern this reuses.

## Token overrides (`:root[data-theme="dark"]`)

| Token | Dark value (= original OLED value) |
|---|---|
| `--bg` / `--bg2` | `#07090d` / `#0b0e14` |
| `--panel` / `--panel2` / `--panel3` | `#11151c` / `#161c25` / `#1c2330` |
| `--border` / `--border2` | `#232b36` / `#2e3845` |
| `--text` / `--muted` / `--faint` | `#f0f4f8` / `#9aa6b2` / `#5b6675` |
| `--accent` / `--accent-rgb` | `#22d3ee` / `34,211,238` |
| `--amber` / `--amber-rgb` | `#fbbf24` / `251,191,36` |
| `--cyan` / `--sky` | `#22d3ee` / `#7dd3fc` |
| `--ok` / `--ok-rgb` / `--okbg` | `#34d399` / `52,211,153` / `#0c2a20` |
| `--danger` / `--danger-rgb` / `--dangerbg` | `#f87171` / `248,113,113` / `#2c1414` |
| `--on-accent` | `#04181c` (near-black text on bright neon accent backgrounds) |

`--header-bg`/`--header-text`/`--header-muted`/`--header-border`/`--mono`/`--radius`/`--t`
are **not** overridden — the header stops using them in dark mode (see below), and
the rest are theme-agnostic.

`#screen-recon`/`#screen-opt`/`.tile.recon`/`.tile.optm` accent overrides also flip
back to the neon values (`#fbbf24`/`251,191,36` and `#22d3ee`/`34,211,238`) under
`[data-theme="dark"]`.

## Selector-specific overrides (literals that don't track tokens)

All restore the exact original pre-restyle declaration. Grouped by area:

**Header / nav** — header stops being a fixed navy bar and blends with the page:
```css
[data-theme="dark"] header{background:linear-gradient(180deg,var(--panel),rgba(17,21,28,.6));
  color:var(--text);border-bottom:1px solid var(--border);backdrop-filter:blur(6px);}
[data-theme="dark"] header .brand .ico{filter:drop-shadow(0 0 8px rgba(var(--accent-rgb),.5));}
[data-theme="dark"] header .mode{color:var(--muted);border-color:var(--border);}
[data-theme="dark"] .iconbtn{background:var(--panel2);border-color:var(--border);color:var(--text);}
[data-theme="dark"] .iconbtn:active{background:var(--panel3);}
[data-theme="dark"] .nav button.on::before{box-shadow:0 0 12px rgba(var(--accent-rgb),.8);}
```

**Body background tint** (hardcoded rgb triples, not tokens):
```css
[data-theme="dark"] body{background:
  radial-gradient(120% 60% at 50% -10%, rgba(34,211,238,.06), transparent 60%),
  radial-gradient(120% 60% at 50% 110%, rgba(251,191,36,.05), transparent 60%),
  var(--bg);}
```

**Home bento cards**:
```css
[data-theme="dark"] .card{box-shadow:inset 0 1px 0 rgba(255,255,255,.04), 0 12px 30px -18px rgba(0,0,0,.8);}
[data-theme="dark"] .deco{opacity:.10;}
[data-theme="dark"] .hero{background:radial-gradient(120% 140% at 100% 0%, rgba(34,211,238,.12), transparent 55%), linear-gradient(160deg,var(--panel2),var(--panel));}
[data-theme="dark"] .hero .badge{background:linear-gradient(160deg,rgba(34,211,238,.2),rgba(34,211,238,.06));box-shadow:0 0 24px -4px rgba(34,211,238,.4);}
[data-theme="dark"] .tile.recon .tbadge{background:linear-gradient(160deg,rgba(251,191,36,.18),rgba(251,191,36,.05));box-shadow:0 0 22px -6px rgba(251,191,36,.5);}
[data-theme="dark"] .tile.recon{box-shadow:inset 0 1px 0 rgba(255,255,255,.04), 0 12px 30px -18px rgba(0,0,0,.8), inset 0 0 0 1px rgba(251,191,36,.06);}
[data-theme="dark"] .tile.optm .tbadge{background:linear-gradient(160deg,rgba(34,211,238,.18),rgba(34,211,238,.05));box-shadow:0 0 22px -6px rgba(34,211,238,.5);}
```

**Counters / dashboard stat cards** (originally had no shadow at all — zero out
the light theme's flat elevation shadow, and add the glow back onto the values):
```css
[data-theme="dark"] .counter,[data-theme="dark"] .stat{box-shadow:none;}
[data-theme="dark"] .counter.target .val{text-shadow:0 0 16px rgba(var(--amber-rgb),.35);}
[data-theme="dark"] .counter.boxes .val{text-shadow:0 0 16px rgba(34,211,238,.3);}
```

**Fields**:
```css
[data-theme="dark"] .field input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.22);}
[data-theme="dark"] .scanfield input{background:#0c1219;box-shadow:inset 0 0 22px -10px rgba(var(--accent-rgb),.5);}
[data-theme="dark"] .scanfield input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.3),inset 0 0 22px -10px rgba(var(--accent-rgb),.5);}
```

**Buttons / banners / progress**:
```css
[data-theme="dark"] .btn.danger{color:#fca5a5;border-color:#5c2626;}
[data-theme="dark"] .banner.ok{border-color:#1f6f54;color:#86efc6;box-shadow:0 0 26px -10px rgba(var(--ok-rgb),.5);}
[data-theme="dark"] .banner.bad{border-color:#7a2b2b;color:#fca5a5;box-shadow:0 0 26px -10px rgba(var(--danger-rgb),.5);}
[data-theme="dark"] .pct-bar{background:#0a0f16;}
```

**Chips / edit overlay**:
```css
[data-theme="dark"] .chip.pull{background:linear-gradient(160deg,#0f2a20,#0c2018);box-shadow:0 0 16px -6px rgba(var(--ok-rgb),.6);}
[data-theme="dark"] .chip.pull .pc{color:#86efc6;}
[data-theme="dark"] .chip.newest{box-shadow:0 0 0 1px rgba(var(--accent-rgb),.4),0 0 16px -6px rgba(var(--accent-rgb),.7);}
[data-theme="dark"] .chip-pc:active{background:rgba(var(--accent-rgb),.18);}
[data-theme="dark"] .chip-del:active{background:rgba(var(--danger-rgb),.18);}
[data-theme="dark"] #edit-overlay{background:rgba(0,0,0,.72);}
[data-theme="dark"] #edit-card{box-shadow:none;}
[data-theme="dark"] #edit-input{background:#0c1219;}
[data-theme="dark"] #edit-input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.22);}
```

**Settings sheet / flash toast**:
```css
[data-theme="dark"] .overlay{background:rgba(0,0,0,.55);}
[data-theme="dark"] .sheet{box-shadow:none;}
[data-theme="dark"] .opt.sel{background:#0c1f22;}
[data-theme="dark"] .flash{color:#fca5a5;border-color:#5c2626;box-shadow:0 10px 30px -10px rgba(0,0,0,.7);}
[data-theme="dark"] .flash.good{color:#86efc6;border-color:#1f6f54;}
```

Everything not listed above (fields, panels, chips base, board/pallet cards,
bottom nav base, action bar, guide panel, etc.) already reads purely from
`var(--token)` and needs no override — it re-themes automatically.

## New sprite icons

Two new symbols added to the existing inline SVG sprite (`#sprite`), same
stroke-based Lucide style as the rest (`stroke:currentColor;fill:none`):

```html
<symbol id="i-moon" viewBox="0 0 24 24"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79Z"/></symbol>
<symbol id="i-sun" viewBox="0 0 24 24"><circle cx="12" cy="12" r="4.2"/><path d="M12 3v2.4M12 18.6V21M4.2 12H1.8M22.2 12h-2.4M5.8 5.8l1.7 1.7M16.5 16.5l1.7 1.7M18.2 5.8l-1.7 1.7M7.5 16.5l-1.7 1.7"/></symbol>
```

Moon = currently light, tap to go dark. Sun = currently dark, tap to go light
(icon shows the mode you're switching **to**, matching how the existing Optimize
guide-icon affordances read).

## Markup changes

One new `.iconbtn` per screen header:

- **Home** (`<header><span class="brand">...</span><span class="mode">v0.2</span></header>`):
  add the button right after `.mode`.
- **Reconcile** / **Optimize** headers: add the button between the existing
  `[data-settings]` button and the `#rc-guide-btn`/`#op-guide-btn` button.

Each button: `<button class="iconbtn" data-theme-toggle aria-label="Toggle dark mode"><svg class="ico"><use href="#i-moon"/></svg></button>`
(all three start with `#i-moon` in markup since light is the default; JS corrects
the icon on load if `cfg.theme` was already `"dark"` from a previous session).

## JS changes

```js
const cfg = Object.assign({ printer:'qln420', mode:'home', theme:'light' }, store.get(LS.cfg,{}));
```

```js
function applyTheme(){
  document.documentElement.dataset.theme = cfg.theme;
  const href = cfg.theme==='dark' ? '#i-sun' : '#i-moon';
  document.querySelectorAll('[data-theme-toggle] use').forEach(u=>u.setAttribute('href',href));
}
function toggleTheme(){ cfg.theme = cfg.theme==='dark' ? 'light' : 'dark'; saveCfg(); applyTheme(); }
document.querySelectorAll('[data-theme-toggle]').forEach(b=>b.onclick=toggleTheme);
applyTheme();
```

`applyTheme()` is called once at startup (alongside the existing initial `render()`
call) and again inside `toggleTheme()`. No other function changes — `render()`,
`setMode()`, and all business logic are untouched; this is purely a CSS+attribute
toggle layered on top.

## Out of scope

- No `prefers-color-scheme` detection.
- No per-screen independent theme (it's a single global `cfg.theme`).
- No change to the print stylesheet (`printCss()`) — print is already plain B/W
  and explicitly theme-independent.
- No change to business logic, localStorage schema beyond the new `theme` field,
  or existing markup beyond the one new button per header and two new sprite
  symbols.

## Acceptance criteria

- Toggling the new header button switches the whole app between the light theme
  (current) and the exact original dark-OLED look (verified by spot-checking a
  few of the listed overrides render the right colors/glows).
- Reloading the page after toggling to dark keeps it dark (persisted via
  `cfg.theme` in `packcalc_cfg`); a fresh `localStorage` (or a value with no
  `theme` field) defaults to light.
- The header is the fixed navy bar in light mode and blends into the OLED
  background in dark mode.
- `node --check` still passes on the inline script after the JS additions.
- No emoji anywhere; the two new icons follow the existing sprite's stroke style.
