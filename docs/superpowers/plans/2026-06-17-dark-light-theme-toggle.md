# Dark/light theme toggle Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a header toggle button that switches `index.html` between the current light bento theme and the original dark-OLED theme (restored exactly, including its glow effects), persisted across sessions.

**Architecture:** A `data-theme="dark"` attribute on `<html>` drives a `:root[data-theme="dark"]` token-override block plus ~20 selector-specific overrides for the few rules that have a literal color baked in instead of `var(--token)`. A new `cfg.theme` field (default `"light"`) persists the choice in the existing `packcalc_cfg` localStorage key. No business logic, markup structure, or print stylesheet changes beyond one new button per header and two new sprite icons.

**Tech Stack:** Vanilla HTML/CSS/JS, no build, no deps, no git repo.

## Global Constraints

- Everything lives in `index.html` (CLAUDE.md) — no new files.
- No build, no deps, no git (CLAUDE.md) — every "commit" step below is a grep/`node --check` verification instead; nothing is committed.
- Icons: inline SVG sprite only, never emoji (CLAUDE.md).
- UI stays in English (CLAUDE.md).
- Print stylesheet (`printCss()`/`doPrint()`) is explicitly out of scope — do not touch it.
- Dark mode must reproduce the **exact** original OLED values/glows documented in `docs/superpowers/plans/2026-06-17-light-bento-restyle.md`'s "Old value"/"Old code" columns — this is a restoration, not a new design.
- After all tasks, the inline `<script>` block must still pass `node --check`.

---

### Task 1: Root token overrides + per-screen/tile accent + body background tint

**Files:**
- Modify: `index.html` (`:root{...}` block, lines 14–30; `#screen-recon`/`#screen-opt`, line 49–50; body background, lines 33–43)

**Interfaces:**
- Produces: `:root[data-theme="dark"]{...}` block and the two dark accent overrides, consumed visually by every later task (they all rely on these tokens flipping).

- [ ] **Step 1: Add the dark token-override block right after `:root{...}`**

Old code:
```css
  :root{
    --bg:#f3f5f7; --bg2:#eef1f4;
    --panel:#ffffff; --panel2:#f7f8fa; --panel3:#eef0f3;
    --border:#e3e6ea; --border2:#d4d9df;
    --text:#16202b; --muted:#6b7686; --faint:#9aa3af;
    --accent:#0891b2; --accent-rgb:8,145,178;
    --amber:#d97706; --amber-rgb:217,119,6;
    --cyan:#0891b2; --sky:#0ea5e9;
    --ok:#16a34a; --ok-rgb:22,163,74; --okbg:#eaf7ee;
    --danger:#dc2626; --danger-rgb:220,38,38; --dangerbg:#fdecea;
    --header-bg:#1b2533; --header-text:#f5f7fa;
    --header-muted:rgba(245,247,250,.62); --header-border:rgba(245,247,250,.16);
    --on-accent:#ffffff;
    --mono:ui-monospace,'Cascadia Mono','Segoe UI Mono','Roboto Mono','SFMono-Regular',Consolas,monospace;
    --radius:14px;
    --t:.16s cubic-bezier(.4,0,.2,1);
  }
```

New code (unchanged block, plus the new dark override block immediately after it):
```css
  :root{
    --bg:#f3f5f7; --bg2:#eef1f4;
    --panel:#ffffff; --panel2:#f7f8fa; --panel3:#eef0f3;
    --border:#e3e6ea; --border2:#d4d9df;
    --text:#16202b; --muted:#6b7686; --faint:#9aa3af;
    --accent:#0891b2; --accent-rgb:8,145,178;
    --amber:#d97706; --amber-rgb:217,119,6;
    --cyan:#0891b2; --sky:#0ea5e9;
    --ok:#16a34a; --ok-rgb:22,163,74; --okbg:#eaf7ee;
    --danger:#dc2626; --danger-rgb:220,38,38; --dangerbg:#fdecea;
    --header-bg:#1b2533; --header-text:#f5f7fa;
    --header-muted:rgba(245,247,250,.62); --header-border:rgba(245,247,250,.16);
    --on-accent:#ffffff;
    --mono:ui-monospace,'Cascadia Mono','Segoe UI Mono','Roboto Mono','SFMono-Regular',Consolas,monospace;
    --radius:14px;
    --t:.16s cubic-bezier(.4,0,.2,1);
  }
  :root[data-theme="dark"]{
    --bg:#07090d; --bg2:#0b0e14;
    --panel:#11151c; --panel2:#161c25; --panel3:#1c2330;
    --border:#232b36; --border2:#2e3845;
    --text:#f0f4f8; --muted:#9aa6b2; --faint:#5b6675;
    --accent:#22d3ee; --accent-rgb:34,211,238;
    --amber:#fbbf24; --amber-rgb:251,191,36;
    --cyan:#22d3ee; --sky:#7dd3fc;
    --ok:#34d399; --ok-rgb:52,211,153; --okbg:#0c2a20;
    --danger:#f87171; --danger-rgb:248,113,113; --dangerbg:#2c1414;
    --on-accent:#04181c;
  }
```

- [ ] **Step 2: Add dark overrides for the per-screen and per-tile accent**

Old code:
```css
  #screen-recon{--accent:#d97706;--accent-rgb:217,119,6;}
  #screen-opt{--accent:#0891b2;--accent-rgb:8,145,178;}
```

New code:
```css
  #screen-recon{--accent:#d97706;--accent-rgb:217,119,6;}
  #screen-opt{--accent:#0891b2;--accent-rgb:8,145,178;}
  [data-theme="dark"] #screen-recon{--accent:#fbbf24;--accent-rgb:251,191,36;}
  [data-theme="dark"] #screen-opt{--accent:#22d3ee;--accent-rgb:34,211,238;}
```

(`.tile.recon`/`.tile.optm` accent overrides are handled in Task 3, alongside the rest of the Home bento block.)

- [ ] **Step 3: Override the body background tint**

Old code:
```css
  body{
    font-family:'Inter',system-ui,-apple-system,'Segoe UI',Roboto,sans-serif;
    background:
      radial-gradient(120% 60% at 50% -10%, rgba(8,145,178,.05), transparent 60%),
      radial-gradient(120% 60% at 50% 110%, rgba(217,119,6,.04), transparent 60%),
      var(--bg);
    color:var(--text);
    overflow-x:hidden;
    -webkit-user-select:none; user-select:none;
    letter-spacing:.1px;
  }
```

New code:
```css
  body{
    font-family:'Inter',system-ui,-apple-system,'Segoe UI',Roboto,sans-serif;
    background:
      radial-gradient(120% 60% at 50% -10%, rgba(8,145,178,.05), transparent 60%),
      radial-gradient(120% 60% at 50% 110%, rgba(217,119,6,.04), transparent 60%),
      var(--bg);
    color:var(--text);
    overflow-x:hidden;
    -webkit-user-select:none; user-select:none;
    letter-spacing:.1px;
  }
  [data-theme="dark"] body{background:
    radial-gradient(120% 60% at 50% -10%, rgba(34,211,238,.06), transparent 60%),
    radial-gradient(120% 60% at 50% 110%, rgba(251,191,36,.05), transparent 60%),
    var(--bg);}
```

- [ ] **Step 4: Verify**

Run:
```
grep -n -- "data-theme=\"dark\"" index.html
```
Expected: at least 4 matches (the `:root` block, the two screen-accent overrides, the body override).

---

### Task 2: Header / icon buttons / nav indicator dark overrides

**Files:**
- Modify: `index.html` (header block, lines 60–70; nav, lines 257–265)

**Interfaces:**
- Consumes: `--panel`, `--panel2`, `--panel3`, `--text`, `--muted`, `--border`, `--accent-rgb` (dark values from Task 1).

- [ ] **Step 1: Add dark overrides after the header/iconbtn block**

Old code:
```css
  /* Header */
  header{display:flex;align-items:center;gap:10px;padding:10px 12px;border-bottom:1px solid rgba(0,0,0,.18);
    background:var(--header-bg);color:var(--header-text);}
  header .brand{display:flex;align-items:center;gap:8px;font-weight:700;font-size:15px;letter-spacing:.3px;}
  header .brand .ico{color:var(--accent);}
  header .mode{font-size:11px;color:var(--header-muted);margin-left:auto;text-transform:uppercase;letter-spacing:1px;
    border:1px solid var(--header-border);border-radius:999px;padding:3px 9px;}
  .iconbtn{background:rgba(245,247,250,.08);border:1px solid var(--header-border);color:var(--header-text);border-radius:11px;
    min-width:44px;height:44px;display:flex;align-items:center;justify-content:center;cursor:pointer;transition:var(--t);}
  .iconbtn:active{background:rgba(245,247,250,.16);transform:scale(.94);}
  .iconbtn.active{color:var(--accent);border-color:var(--accent);}
```

New code:
```css
  /* Header */
  header{display:flex;align-items:center;gap:10px;padding:10px 12px;border-bottom:1px solid rgba(0,0,0,.18);
    background:var(--header-bg);color:var(--header-text);}
  header .brand{display:flex;align-items:center;gap:8px;font-weight:700;font-size:15px;letter-spacing:.3px;}
  header .brand .ico{color:var(--accent);}
  header .mode{font-size:11px;color:var(--header-muted);margin-left:auto;text-transform:uppercase;letter-spacing:1px;
    border:1px solid var(--header-border);border-radius:999px;padding:3px 9px;}
  .iconbtn{background:rgba(245,247,250,.08);border:1px solid var(--header-border);color:var(--header-text);border-radius:11px;
    min-width:44px;height:44px;display:flex;align-items:center;justify-content:center;cursor:pointer;transition:var(--t);}
  .iconbtn:active{background:rgba(245,247,250,.16);transform:scale(.94);}
  .iconbtn.active{color:var(--accent);border-color:var(--accent);}
  [data-theme="dark"] header{background:linear-gradient(180deg,var(--panel),rgba(17,21,28,.6));
    color:var(--text);border-bottom:1px solid var(--border);backdrop-filter:blur(6px);}
  [data-theme="dark"] header .brand .ico{filter:drop-shadow(0 0 8px rgba(var(--accent-rgb),.5));}
  [data-theme="dark"] header .mode{color:var(--muted);border-color:var(--border);}
  [data-theme="dark"] .iconbtn{background:var(--panel2);border-color:var(--border);color:var(--text);}
  [data-theme="dark"] .iconbtn:active{background:var(--panel3);}
```

- [ ] **Step 2: Add the dark nav-indicator glow**

Old code:
```css
  .nav button.on::before{content:"";position:absolute;top:-5px;left:50%;transform:translateX(-50%);width:26px;height:3px;
    border-radius:3px;background:var(--accent);}
```

New code:
```css
  .nav button.on::before{content:"";position:absolute;top:-5px;left:50%;transform:translateX(-50%);width:26px;height:3px;
    border-radius:3px;background:var(--accent);}
  [data-theme="dark"] .nav button.on::before{box-shadow:0 0 12px rgba(var(--accent-rgb),.8);}
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "data-theme=\"dark\"\] header\|data-theme=\"dark\"\] .iconbtn\|data-theme=\"dark\"\] .nav" index.html
```
Expected: 5 matches (header, header .brand .ico, header .mode, .iconbtn, .iconbtn:active) plus 1 for `.nav button.on::before` — 6 total.

---

### Task 3: Home bento cards + counters/dashboard stat glow restoration

**Files:**
- Modify: `index.html` (Home bento block, lines ~95–123; counters block, lines ~78–93; dashboard `.stat`, line ~229)

**Interfaces:**
- Consumes: `--panel2`, `--panel`, `--ok-rgb`, `--amber-rgb` and the dark accent values from Task 1.

- [ ] **Step 1: Add dark overrides after the Home bento block**

Old code:
```css
  .tile.recon{--accent:#d97706;--accent-rgb:217,119,6;}
  .tile.recon .deco,.tile.recon .kick{color:var(--amber);}
  .tile.recon .tbadge{background:rgba(217,119,6,.12);color:var(--amber);}
  .tile.optm{--accent:#0891b2;--accent-rgb:8,145,178;}
  .tile.optm .deco,.tile.optm .kick{color:var(--cyan);}
  .tile.optm .tbadge{background:rgba(8,145,178,.12);color:var(--cyan);}
```

New code:
```css
  .tile.recon{--accent:#d97706;--accent-rgb:217,119,6;}
  .tile.recon .deco,.tile.recon .kick{color:var(--amber);}
  .tile.recon .tbadge{background:rgba(217,119,6,.12);color:var(--amber);}
  .tile.optm{--accent:#0891b2;--accent-rgb:8,145,178;}
  .tile.optm .deco,.tile.optm .kick{color:var(--cyan);}
  .tile.optm .tbadge{background:rgba(8,145,178,.12);color:var(--cyan);}
  [data-theme="dark"] .tile.recon{--accent:#fbbf24;--accent-rgb:251,191,36;
    box-shadow:inset 0 1px 0 rgba(255,255,255,.04), 0 12px 30px -18px rgba(0,0,0,.8), inset 0 0 0 1px rgba(251,191,36,.06);}
  [data-theme="dark"] .tile.recon .tbadge{background:linear-gradient(160deg,rgba(251,191,36,.18),rgba(251,191,36,.05));box-shadow:0 0 22px -6px rgba(251,191,36,.5);}
  [data-theme="dark"] .tile.optm{--accent:#22d3ee;--accent-rgb:34,211,238;}
  [data-theme="dark"] .tile.optm .tbadge{background:linear-gradient(160deg,rgba(34,211,238,.18),rgba(34,211,238,.05));box-shadow:0 0 22px -6px rgba(34,211,238,.5);}
  [data-theme="dark"] .card{box-shadow:inset 0 1px 0 rgba(255,255,255,.04), 0 12px 30px -18px rgba(0,0,0,.8);}
  [data-theme="dark"] .deco{opacity:.10;}
  [data-theme="dark"] .hero{background:radial-gradient(120% 140% at 100% 0%, rgba(34,211,238,.12), transparent 55%), linear-gradient(160deg,var(--panel2),var(--panel));}
  [data-theme="dark"] .hero .badge{background:linear-gradient(160deg,rgba(34,211,238,.2),rgba(34,211,238,.06));box-shadow:0 0 24px -4px rgba(34,211,238,.4);}
```

- [ ] **Step 2: Add dark overrides for the counter bar**

Old code:
```css
  .counter.target .val{color:var(--amber);}
  .counter.target .ch{color:var(--amber);}
  .counter.boxes .val{color:var(--cyan);}
  .counter.boxes .ch{color:var(--cyan);}
  .counter.pieces .val{color:var(--sky);}
  .counter.pieces .ch{color:var(--sky);}
```

New code:
```css
  .counter.target .val{color:var(--amber);}
  .counter.target .ch{color:var(--amber);}
  .counter.boxes .val{color:var(--cyan);}
  .counter.boxes .ch{color:var(--cyan);}
  .counter.pieces .val{color:var(--sky);}
  .counter.pieces .ch{color:var(--sky);}
  [data-theme="dark"] .counter{box-shadow:none;}
  [data-theme="dark"] .counter.target .val{text-shadow:0 0 16px rgba(var(--amber-rgb),.35);}
  [data-theme="dark"] .counter.boxes .val{text-shadow:0 0 16px rgba(34,211,238,.3);}
```

- [ ] **Step 3: Zero out the dashboard `.stat` card shadow in dark mode**

Old code:
```css
  .stat{background:var(--panel);border:1px solid var(--border);border-radius:13px;padding:11px 12px;
    box-shadow:0 1px 2px rgba(16,24,32,.04);}
```

New code:
```css
  .stat{background:var(--panel);border:1px solid var(--border);border-radius:13px;padding:11px 12px;
    box-shadow:0 1px 2px rgba(16,24,32,.04);}
  [data-theme="dark"] .stat{box-shadow:none;}
```

- [ ] **Step 4: Verify**

Run:
```
grep -n -- "data-theme=\"dark\"\] .tile\|data-theme=\"dark\"\] .card\|data-theme=\"dark\"\] .counter\|data-theme=\"dark\"\] .stat\|data-theme=\"dark\"\] .hero\|data-theme=\"dark\"\] .deco" index.html
```
Expected: 9 matches.

---

### Task 4: Field focus / scan field dark overrides

**Files:**
- Modify: `index.html` (Fields block, lines ~132–146)

**Interfaces:**
- Consumes: `--accent-rgb` (dark value from Task 1).

- [ ] **Step 1: Add dark overrides after the Fields block**

Old code:
```css
  .scanfield input{font-size:20px;font-weight:600;border-color:rgba(var(--accent-rgb),.55);background:var(--panel2);}
  .scanfield input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.22);}
  .scanfield .ico{color:var(--accent);}
  .row2{display:flex;gap:10px;}
  .row2 .field{flex:1;}
```

New code:
```css
  .scanfield input{font-size:20px;font-weight:600;border-color:rgba(var(--accent-rgb),.55);background:var(--panel2);}
  .scanfield input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.22);}
  .scanfield .ico{color:var(--accent);}
  .row2{display:flex;gap:10px;}
  .row2 .field{flex:1;}
  [data-theme="dark"] .field input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.22);}
  [data-theme="dark"] .scanfield input{background:#0c1219;box-shadow:inset 0 0 22px -10px rgba(var(--accent-rgb),.5);}
  [data-theme="dark"] .scanfield input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.3),inset 0 0 22px -10px rgba(var(--accent-rgb),.5);}
```

- [ ] **Step 2: Verify**

Run:
```
grep -n -- "data-theme=\"dark\"\] .scanfield\|data-theme=\"dark\"\] .field" index.html
```
Expected: 3 matches.

---

### Task 5: Button / banner / progress-bar dark overrides

**Files:**
- Modify: `index.html` (Buttons block ~148–158; Panels/banners block ~160–178)

**Interfaces:**
- Consumes: `--ok-rgb`, `--danger-rgb` (dark values from Task 1).

- [ ] **Step 1: Add the dark danger-button override**

Old code:
```css
  .btn.danger{background:var(--dangerbg);color:var(--danger);border:1px solid rgba(var(--danger-rgb),.35);}
  .btn.blue{background:var(--accent);color:var(--on-accent);}
  .btn:disabled{opacity:.45;cursor:not-allowed;}
  .btns{display:flex;gap:10px;margin-top:10px;}
```

New code:
```css
  .btn.danger{background:var(--dangerbg);color:var(--danger);border:1px solid rgba(var(--danger-rgb),.35);}
  .btn.blue{background:var(--accent);color:var(--on-accent);}
  .btn:disabled{opacity:.45;cursor:not-allowed;}
  .btns{display:flex;gap:10px;margin-top:10px;}
  [data-theme="dark"] .btn.danger{color:#fca5a5;border-color:#5c2626;}
```

- [ ] **Step 2: Add the dark banner/progress-bar overrides**

Old code:
```css
  .banner.ok{background:var(--okbg);border-color:rgba(var(--ok-rgb),.35);color:#14532d;}
  .banner.ok .ico{color:var(--ok);}
  .banner.bad{background:var(--dangerbg);border-color:rgba(var(--danger-rgb),.35);color:#7f1d1d;}
  .banner.bad .ico{color:var(--danger);}
  .banner.idle{background:var(--panel2);border-color:var(--border);color:var(--muted);}
  .banner.idle .ico{color:var(--faint);}
  .kv{display:flex;justify-content:space-between;font-size:14px;padding:4px 0;border-bottom:1px dashed var(--border2);}
  .kv:last-child{border-bottom:none;}
  .kv .k{color:var(--muted);} .kv .v{font-weight:700;font-variant-numeric:tabular-nums;font-family:var(--mono);}
  .pct-wrap{margin-top:9px;}
  .pct-bar{height:10px;background:var(--panel3);border:1px solid var(--border);border-radius:6px;overflow:hidden;}
```

New code:
```css
  .banner.ok{background:var(--okbg);border-color:rgba(var(--ok-rgb),.35);color:#14532d;}
  .banner.ok .ico{color:var(--ok);}
  .banner.bad{background:var(--dangerbg);border-color:rgba(var(--danger-rgb),.35);color:#7f1d1d;}
  .banner.bad .ico{color:var(--danger);}
  .banner.idle{background:var(--panel2);border-color:var(--border);color:var(--muted);}
  .banner.idle .ico{color:var(--faint);}
  .kv{display:flex;justify-content:space-between;font-size:14px;padding:4px 0;border-bottom:1px dashed var(--border2);}
  .kv:last-child{border-bottom:none;}
  .kv .k{color:var(--muted);} .kv .v{font-weight:700;font-variant-numeric:tabular-nums;font-family:var(--mono);}
  .pct-wrap{margin-top:9px;}
  .pct-bar{height:10px;background:var(--panel3);border:1px solid var(--border);border-radius:6px;overflow:hidden;}
  [data-theme="dark"] .banner.ok{border-color:#1f6f54;color:#86efc6;box-shadow:0 0 26px -10px rgba(var(--ok-rgb),.5);}
  [data-theme="dark"] .banner.bad{border-color:#7a2b2b;color:#fca5a5;box-shadow:0 0 26px -10px rgba(var(--danger-rgb),.5);}
  [data-theme="dark"] .pct-bar{background:#0a0f16;}
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "data-theme=\"dark\"\] .btn\|data-theme=\"dark\"\] .banner\|data-theme=\"dark\"\] .pct-bar" index.html
```
Expected: 4 matches.

---

### Task 6: Chip / edit-overlay dark overrides

**Files:**
- Modify: `index.html` (Chips block ~190–203; Edit overlay block ~204–214)

**Interfaces:**
- Consumes: `--ok-rgb`, `--accent-rgb`, `--danger-rgb` (dark values from Task 1).

- [ ] **Step 1: Add the dark chip overrides**

Old code:
```css
  .chip.pull{background:var(--okbg);border-color:var(--ok);}
  .chip.pull .pc{color:#14532d;}
  .chip.newest{border-color:var(--accent);box-shadow:0 0 0 1px rgba(var(--accent-rgb),.3);}
  .chip.editable{padding-right:22px;}
  .chip-pc{font-family:var(--mono);font-size:18px;font-weight:800;font-variant-numeric:tabular-nums;line-height:1.1;cursor:pointer;border-radius:5px;padding:1px 3px;margin:-1px -3px;transition:background var(--t);}
  .chip-pc:active{background:rgba(var(--accent-rgb),.14);}
  .chip-del{position:absolute;top:4px;right:4px;width:20px;height:20px;border:none;background:none;color:var(--muted);cursor:pointer;border-radius:5px;font-size:13px;font-weight:700;line-height:1;display:flex;align-items:center;justify-content:center;padding:0;touch-action:manipulation;transition:var(--t);}
  .chip-del:active{color:var(--danger);background:rgba(var(--danger-rgb),.14);}
```

New code:
```css
  .chip.pull{background:var(--okbg);border-color:var(--ok);}
  .chip.pull .pc{color:#14532d;}
  .chip.newest{border-color:var(--accent);box-shadow:0 0 0 1px rgba(var(--accent-rgb),.3);}
  .chip.editable{padding-right:22px;}
  .chip-pc{font-family:var(--mono);font-size:18px;font-weight:800;font-variant-numeric:tabular-nums;line-height:1.1;cursor:pointer;border-radius:5px;padding:1px 3px;margin:-1px -3px;transition:background var(--t);}
  .chip-pc:active{background:rgba(var(--accent-rgb),.14);}
  .chip-del{position:absolute;top:4px;right:4px;width:20px;height:20px;border:none;background:none;color:var(--muted);cursor:pointer;border-radius:5px;font-size:13px;font-weight:700;line-height:1;display:flex;align-items:center;justify-content:center;padding:0;touch-action:manipulation;transition:var(--t);}
  .chip-del:active{color:var(--danger);background:rgba(var(--danger-rgb),.14);}
  [data-theme="dark"] .chip.pull{background:linear-gradient(160deg,#0f2a20,#0c2018);box-shadow:0 0 16px -6px rgba(var(--ok-rgb),.6);}
  [data-theme="dark"] .chip.pull .pc{color:#86efc6;}
  [data-theme="dark"] .chip.newest{box-shadow:0 0 0 1px rgba(var(--accent-rgb),.4),0 0 16px -6px rgba(var(--accent-rgb),.7);}
  [data-theme="dark"] .chip-pc:active{background:rgba(var(--accent-rgb),.18);}
  [data-theme="dark"] .chip-del:active{background:rgba(var(--danger-rgb),.18);}
```

- [ ] **Step 2: Add the dark edit-overlay overrides**

Old code:
```css
  #edit-overlay{display:none;position:fixed;inset:0;background:rgba(15,20,28,.55);z-index:200;align-items:center;justify-content:center;}
  #edit-overlay.open{display:flex;}
  #edit-card{background:var(--panel);border:1px solid var(--border2);border-radius:18px;padding:20px;width:min(320px,90vw);display:flex;flex-direction:column;gap:14px;
    box-shadow:0 20px 50px -20px rgba(16,24,32,.35);}
  #edit-title{font-weight:700;font-size:15px;color:var(--text);}
  #edit-input{font-family:var(--mono);font-size:28px;font-weight:800;background:var(--panel2);border:1px solid var(--border2);border-radius:11px;color:var(--text);padding:10px 14px;width:100%;text-align:center;font-variant-numeric:tabular-nums;}
  #edit-input:focus{border-color:var(--accent);outline:none;box-shadow:0 0 0 3px rgba(var(--accent-rgb),.18);}
```

New code:
```css
  #edit-overlay{display:none;position:fixed;inset:0;background:rgba(15,20,28,.55);z-index:200;align-items:center;justify-content:center;}
  #edit-overlay.open{display:flex;}
  #edit-card{background:var(--panel);border:1px solid var(--border2);border-radius:18px;padding:20px;width:min(320px,90vw);display:flex;flex-direction:column;gap:14px;
    box-shadow:0 20px 50px -20px rgba(16,24,32,.35);}
  #edit-title{font-weight:700;font-size:15px;color:var(--text);}
  #edit-input{font-family:var(--mono);font-size:28px;font-weight:800;background:var(--panel2);border:1px solid var(--border2);border-radius:11px;color:var(--text);padding:10px 14px;width:100%;text-align:center;font-variant-numeric:tabular-nums;}
  #edit-input:focus{border-color:var(--accent);outline:none;box-shadow:0 0 0 3px rgba(var(--accent-rgb),.18);}
  [data-theme="dark"] #edit-overlay{background:rgba(0,0,0,.72);}
  [data-theme="dark"] #edit-card{box-shadow:none;}
  [data-theme="dark"] #edit-input{background:#0c1219;}
  [data-theme="dark"] #edit-input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.22);}
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "data-theme=\"dark\"\] .chip\|data-theme=\"dark\"\] #edit" index.html
```
Expected: 9 matches.

---

### Task 7: Settings sheet / flash toast dark overrides

**Files:**
- Modify: `index.html` (Settings sheet block ~270–285; Flash block ~287–292)

**Interfaces:**
- Consumes: no new tokens — all literal restorations.

- [ ] **Step 1: Add the dark settings-sheet overrides**

Old code:
```css
  .overlay{position:fixed;inset:0;background:rgba(15,20,28,.45);display:none;align-items:flex-end;z-index:50;backdrop-filter:blur(3px);}
  .overlay.open{display:flex;}
  .sheet{background:var(--panel);border-top-left-radius:22px;border-top-right-radius:22px;width:100%;max-width:560px;margin:0 auto;
    padding:8px 16px calc(env(safe-area-inset-bottom,0px) + 16px);border:1px solid var(--border);animation:slideup var(--t);
    box-shadow:0 -10px 30px -10px rgba(16,24,32,.18);}
  @keyframes slideup{from{transform:translateY(20px);opacity:.6}to{transform:translateY(0);opacity:1}}
  .grab{width:38px;height:4px;border-radius:3px;background:var(--border2);margin:6px auto 12px;}
  .sheet h3{margin:0 0 12px;font-size:17px;}
  .opt{display:flex;align-items:center;gap:11px;padding:13px;border:1px solid var(--border);border-radius:12px;margin-bottom:9px;cursor:pointer;background:var(--panel2);transition:var(--t);}
  .opt .oi{width:38px;height:38px;border-radius:10px;display:flex;align-items:center;justify-content:center;flex:none;background:var(--panel3);border:1px solid var(--border2);color:var(--muted);}
  .opt.sel{border-color:var(--accent);background:rgba(var(--accent-rgb),.08);}
```

New code:
```css
  .overlay{position:fixed;inset:0;background:rgba(15,20,28,.45);display:none;align-items:flex-end;z-index:50;backdrop-filter:blur(3px);}
  .overlay.open{display:flex;}
  .sheet{background:var(--panel);border-top-left-radius:22px;border-top-right-radius:22px;width:100%;max-width:560px;margin:0 auto;
    padding:8px 16px calc(env(safe-area-inset-bottom,0px) + 16px);border:1px solid var(--border);animation:slideup var(--t);
    box-shadow:0 -10px 30px -10px rgba(16,24,32,.18);}
  @keyframes slideup{from{transform:translateY(20px);opacity:.6}to{transform:translateY(0);opacity:1}}
  .grab{width:38px;height:4px;border-radius:3px;background:var(--border2);margin:6px auto 12px;}
  .sheet h3{margin:0 0 12px;font-size:17px;}
  .opt{display:flex;align-items:center;gap:11px;padding:13px;border:1px solid var(--border);border-radius:12px;margin-bottom:9px;cursor:pointer;background:var(--panel2);transition:var(--t);}
  .opt .oi{width:38px;height:38px;border-radius:10px;display:flex;align-items:center;justify-content:center;flex:none;background:var(--panel3);border:1px solid var(--border2);color:var(--muted);}
  .opt.sel{border-color:var(--accent);background:rgba(var(--accent-rgb),.08);}
  [data-theme="dark"] .overlay{background:rgba(0,0,0,.55);}
  [data-theme="dark"] .sheet{box-shadow:none;}
  [data-theme="dark"] .opt.sel{background:#0c1f22;}
```

- [ ] **Step 2: Add the dark flash-toast overrides**

Old code:
```css
  .flash{position:fixed;left:50%;top:16px;transform:translateX(-50%) translateY(-8px);display:flex;align-items:center;gap:8px;
    background:var(--dangerbg);color:var(--danger);border:1px solid rgba(var(--danger-rgb),.35);padding:9px 15px;border-radius:12px;font-size:13px;font-weight:700;
    z-index:80;opacity:0;transition:var(--t);pointer-events:none;max-width:90%;text-align:center;box-shadow:0 10px 30px -10px rgba(16,24,32,.25);}
  .flash .ico{width:18px;height:18px;}
  .flash.show{opacity:1;transform:translateX(-50%) translateY(0);}
  .flash.good{background:var(--okbg);color:var(--ok);border-color:rgba(var(--ok-rgb),.35);}
```

New code:
```css
  .flash{position:fixed;left:50%;top:16px;transform:translateX(-50%) translateY(-8px);display:flex;align-items:center;gap:8px;
    background:var(--dangerbg);color:var(--danger);border:1px solid rgba(var(--danger-rgb),.35);padding:9px 15px;border-radius:12px;font-size:13px;font-weight:700;
    z-index:80;opacity:0;transition:var(--t);pointer-events:none;max-width:90%;text-align:center;box-shadow:0 10px 30px -10px rgba(16,24,32,.25);}
  .flash .ico{width:18px;height:18px;}
  .flash.show{opacity:1;transform:translateX(-50%) translateY(0);}
  .flash.good{background:var(--okbg);color:var(--ok);border-color:rgba(var(--ok-rgb),.35);}
  [data-theme="dark"] .flash{color:#fca5a5;border-color:#5c2626;box-shadow:0 10px 30px -10px rgba(0,0,0,.7);}
  [data-theme="dark"] .flash.good{color:#86efc6;border-color:#1f6f54;}
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "data-theme=\"dark\"\] .overlay\|data-theme=\"dark\"\] .sheet\|data-theme=\"dark\"\] .opt\|data-theme=\"dark\"\] .flash" index.html
```
Expected: 5 matches.

---

### Task 8: New sprite icons + new header toggle button (markup, all 3 screens)

**Files:**
- Modify: `index.html` (sprite `<svg id="sprite">`, ~line 322; Home header line 328; Reconcile header lines 358–364; Optimize header lines 392–398)

**Interfaces:**
- Produces: `#i-moon`/`#i-sun` sprite symbols and three `[data-theme-toggle]` buttons, consumed by Task 9's JS.

- [ ] **Step 1: Add the two new sprite symbols before `</svg>`**

Old code:
```html
  <symbol id="i-database" viewBox="0 0 24 24"><ellipse cx="12" cy="5.5" rx="8" ry="3"/><path d="M4 5.5v6c0 1.66 3.58 3 8 3s8-1.34 8-3v-6"/><path d="M4 11.5v6c0 1.66 3.58 3 8 3s8-1.34 8-3v-6"/></symbol>
</svg>
```

New code:
```html
  <symbol id="i-database" viewBox="0 0 24 24"><ellipse cx="12" cy="5.5" rx="8" ry="3"/><path d="M4 5.5v6c0 1.66 3.58 3 8 3s8-1.34 8-3v-6"/><path d="M4 11.5v6c0 1.66 3.58 3 8 3s8-1.34 8-3v-6"/></symbol>
  <symbol id="i-moon" viewBox="0 0 24 24"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79Z"/></symbol>
  <symbol id="i-sun" viewBox="0 0 24 24"><circle cx="12" cy="12" r="4.2"/><path d="M12 3v2.4M12 18.6V21M4.2 12H1.8M22.2 12h-2.4M5.8 5.8l1.7 1.7M16.5 16.5l1.7 1.7M18.2 5.8l-1.7 1.7M7.5 16.5l-1.7 1.7"/></symbol>
</svg>
```

- [ ] **Step 2: Add the toggle button to the Home header**

Old code:
```html
    <header><span class="brand"><svg class="ico"><use href="#i-warehouse"/></svg>Scan Calculator</span><span class="mode">v0.2</span></header>
```

New code:
```html
    <header><span class="brand"><svg class="ico"><use href="#i-warehouse"/></svg>Scan Calculator</span><span class="mode">v0.2</span>
      <button class="iconbtn" data-theme-toggle aria-label="Toggle dark mode"><svg class="ico"><use href="#i-moon"/></svg></button>
    </header>
```

- [ ] **Step 3: Add the toggle button to the Reconcile header**

Old code:
```html
      <button class="iconbtn" data-settings aria-label="Settings"><svg class="ico"><use href="#i-settings"/></svg></button>
      <button class="iconbtn" id="rc-guide-btn" aria-label="Quick guide" aria-expanded="false" aria-controls="rc-guide">?</button>
```

New code:
```html
      <button class="iconbtn" data-settings aria-label="Settings"><svg class="ico"><use href="#i-settings"/></svg></button>
      <button class="iconbtn" data-theme-toggle aria-label="Toggle dark mode"><svg class="ico"><use href="#i-moon"/></svg></button>
      <button class="iconbtn" id="rc-guide-btn" aria-label="Quick guide" aria-expanded="false" aria-controls="rc-guide">?</button>
```

- [ ] **Step 4: Add the toggle button to the Optimize header**

Old code:
```html
      <button class="iconbtn" data-settings aria-label="Settings"><svg class="ico"><use href="#i-settings"/></svg></button>
      <button class="iconbtn" id="op-guide-btn" aria-label="Quick guide" aria-expanded="false" aria-controls="op-guide">?</button>
```

New code:
```html
      <button class="iconbtn" data-settings aria-label="Settings"><svg class="ico"><use href="#i-settings"/></svg></button>
      <button class="iconbtn" data-theme-toggle aria-label="Toggle dark mode"><svg class="ico"><use href="#i-moon"/></svg></button>
      <button class="iconbtn" id="op-guide-btn" aria-label="Quick guide" aria-expanded="false" aria-controls="op-guide">?</button>
```

- [ ] **Step 5: Verify**

Run:
```
grep -n -- "i-moon\|i-sun\|data-theme-toggle" index.html
```
Expected: 2 matches for the sprite symbols (`i-moon`, `i-sun` definitions) + 3 matches for `data-theme-toggle` buttons + 3 more for the `#i-moon` used inside those buttons = 8 total.

---

### Task 9: JS wiring (`cfg.theme`, `applyTheme`/`toggleTheme`) + CLAUDE.md note + final verification

**Files:**
- Modify: `index.html` (`cfg` declaration, line 478; init block, lines 1061–1066)
- Modify: `CLAUDE.md` (architecture notes)

**Interfaces:**
- Consumes: `[data-theme-toggle]` buttons and `#i-moon`/`#i-sun` symbols from Task 8; `data-theme` attribute consumed by every CSS task (1–7).
- Produces: `applyTheme()` and `toggleTheme()`, global functions, no other code depends on their internals beyond calling `toggleTheme()` on click.

- [ ] **Step 1: Add `theme:'light'` to the `cfg` default**

Old code:
```js
const cfg     = Object.assign({ printer:'qln420', mode:'home' }, store.get(LS.cfg,{}));
```

New code:
```js
const cfg     = Object.assign({ printer:'qln420', mode:'home', theme:'light' }, store.get(LS.cfg,{}));
```

- [ ] **Step 2: Add `applyTheme`/`toggleTheme` and wire the buttons at the end of the script**

Old code:
```js
/* init */
$('rc-guide-btn').onclick=()=>toggleGuide('recon');
$('op-guide-btn').onclick=()=>toggleGuide('opt');
render();
if(cfg.mode==='reconcile') focusEl('in-rscan');
if(cfg.mode==='optimize') focusEl('in-oscan');
```

New code:
```js
/* Theme toggle */
function applyTheme(){
  document.documentElement.dataset.theme = cfg.theme;
  const href = cfg.theme==='dark' ? '#i-sun' : '#i-moon';
  document.querySelectorAll('[data-theme-toggle] use').forEach(u=>u.setAttribute('href',href));
}
function toggleTheme(){ cfg.theme = cfg.theme==='dark' ? 'light' : 'dark'; saveCfg(); applyTheme(); }
document.querySelectorAll('[data-theme-toggle]').forEach(b=>b.onclick=toggleTheme);

/* init */
$('rc-guide-btn').onclick=()=>toggleGuide('recon');
$('op-guide-btn').onclick=()=>toggleGuide('opt');
applyTheme();
render();
if(cfg.mode==='reconcile') focusEl('in-rscan');
if(cfg.mode==='optimize') focusEl('in-oscan');
```

- [ ] **Step 3: Update `CLAUDE.md`'s design-language line**

Old code:
```
Single-file warehouse pick tool: **everything lives in `index.html`** (inline CSS + JS, vanilla, no
framework, no build, no deps, no git). Runs offline from `localStorage` on a Zebra TC520L 5" handheld
(Chromium WebView). UI in English; design = light bento grid with a dark header bar (see
`docs/superpowers/specs/2026-06-17-light-bento-restyle-design.md`); codes/quantities use a monospace
system font via `--mono`.
```

New code:
```
Single-file warehouse pick tool: **everything lives in `index.html`** (inline CSS + JS, vanilla, no
framework, no build, no deps, no git). Runs offline from `localStorage` on a Zebra TC520L 5" handheld
(Chromium WebView). UI in English; design = light bento grid with a dark header bar by default (see
`docs/superpowers/specs/2026-06-17-light-bento-restyle-design.md`), with a dark-OLED theme toggle
(`cfg.theme`, `data-theme` attribute on `<html>`, see
`docs/superpowers/specs/2026-06-17-dark-light-theme-toggle-design.md`); codes/quantities use a
monospace system font via `--mono`.
```

- [ ] **Step 4: Run the project's syntax check**

Run:
```bash
cd "/c/Users/aspiezia/Downloads/scan calculator handheld" && node -e "
const fs = require('fs');
const lines = fs.readFileSync('index.html', 'utf8').split(/\r?\n/);
let s=-1,e=-1;
for(let i=0;i<lines.length;i++){
  if(s===-1 && /^<script>/.test(lines[i])) s=i;
  if(s!==-1 && /^<\/script>/.test(lines[i])){ e=i; break; }
}
fs.writeFileSync('tmp.js', lines.slice(s+1,e).join('\n'), 'utf8');
"
node --check tmp.js && echo "SYNTAX OK"
rm -f tmp.js
```
Expected: `SYNTAX OK`.

- [ ] **Step 5: Whole-file count of `data-theme="dark"` overrides**

Run:
```
grep -c -- "data-theme=\"dark\"" index.html
```
Expected: 33 (4 from Task 1 + 6 from Task 2 + 9 from Task 3 + 3 from Task 4 + 4 from Task 5 + 9 from Task 6 + 5 from Task 7 — every CSS override landed; if the count is lower, diff against this plan to find the missing selector).

- [ ] **Step 6: Manual visual check**

Run:
```
Start-Process index.html
```
Manually confirm: app opens in light mode by default; tapping the new moon-icon header button switches the whole app to the dark-OLED look (near-black background, neon cyan/amber glows, header blending into the background) and the icon becomes a sun; reloading the page keeps dark mode; tapping the sun returns to light mode; the toggle button is present and works from all three screens (Home, Reconcile, Optimize).
