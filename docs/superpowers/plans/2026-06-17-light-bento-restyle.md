# Light bento restyle Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restyle `index.html` from its current dark-OLED palette to the light, bento-card theme approved in `docs/superpowers/specs/2026-06-17-light-bento-restyle-design.md`, with a dark header retained and a monospace font applied to codes/quantities — without changing any JS logic, markup structure, or the print stylesheet.

**Architecture:** This is a single-file app — the entire change lives inside the `<style>` block of `index.html` (currently lines 13–296), region by region, plus one token-table correction in `CLAUDE.md`. No HTML markup or `<script>` content changes; class names and IDs are untouched so existing JS selectors keep working.

**Tech Stack:** Vanilla HTML/CSS/JS, no build step, no dependencies, no git repo.

## Global Constraints

- Everything lives in `index.html` (CLAUDE.md) — do not extract CSS to a separate file.
- No build, no deps, no git (CLAUDE.md) — every task's "commit" step is replaced by a grep-based verification; nothing is committed anywhere.
- UI stays in English (CLAUDE.md).
- Print output (`printCss()` / `doPrint()`, lines ~821–839) is plain B/W `#000`/`#fff` already and is explicitly **out of scope** — do not touch it.
- No new features/components/markup — only CSS property values and a handful of new CSS custom properties change.
- After all tasks, the inline `<script>` block must still pass `node --check` (sanity check that no JS was accidentally altered).

---

## Token reference (used throughout all tasks)

| Token | New value |
|---|---|
| `--bg` | `#f3f5f7` |
| `--bg2` | `#eef1f4` |
| `--panel` | `#ffffff` |
| `--panel2` | `#f7f8fa` |
| `--panel3` | `#eef0f3` |
| `--border` | `#e3e6ea` |
| `--border2` | `#d4d9df` |
| `--text` | `#16202b` |
| `--muted` | `#6b7686` |
| `--faint` | `#9aa3af` |
| `--accent` / `--cyan` | `#0891b2` (rgb `8,145,178`) |
| `--amber` | `#d97706` (rgb `217,119,6`) |
| `--sky` | `#0ea5e9` |
| `--ok` | `#16a34a` (rgb `22,163,74`) |
| `--okbg` | `#eaf7ee` |
| `--danger` | `#dc2626` (rgb `220,38,38`) |
| `--dangerbg` | `#fdecea` |
| `--header-bg` (new) | `#1b2533` |
| `--header-text` (new) | `#f5f7fa` |
| `--header-muted` (new) | `rgba(245,247,250,.62)` |
| `--header-border` (new) | `rgba(245,247,250,.16)` |
| `--on-accent` (new) | `#ffffff` |
| `--mono` (new) | `ui-monospace,'Cascadia Mono','Segoe UI Mono','Roboto Mono','SFMono-Regular',Consolas,monospace` |

---

### Task 1: Root tokens + per-screen/tile accent overrides

**Files:**
- Modify: `index.html` (the `:root{...}` block and the two screen-accent override rules, currently lines 14–26 and 45–46)

**Interfaces:**
- Produces: every CSS variable listed in the token reference table above, available to all later tasks.

- [ ] **Step 1: Replace the `:root` block**

Old code (lines 14–26):
```css
  :root{
    --bg:#07090d; --bg2:#0b0e14;
    --panel:#11151c; --panel2:#161c25; --panel3:#1c2330;
    --border:#232b36; --border2:#2e3845;
    --text:#f0f4f8; --muted:#9aa6b2; --faint:#5b6675;
    --accent:#22d3ee; --accent-rgb:34,211,238;
    --amber:#fbbf24; --amber-rgb:251,191,36;
    --cyan:#22d3ee; --sky:#7dd3fc;
    --ok:#34d399; --ok-rgb:52,211,153; --okbg:#0c2a20;
    --danger:#f87171; --danger-rgb:248,113,113; --dangerbg:#2c1414;
    --radius:14px;
    --t:.16s cubic-bezier(.4,0,.2,1);
  }
```

New code:
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

- [ ] **Step 2: Replace the per-screen accent overrides**

Old code (lines 45–46):
```css
  #screen-recon{--accent:#fbbf24;--accent-rgb:251,191,36;}
  #screen-opt{--accent:#22d3ee;--accent-rgb:34,211,238;}
```

New code:
```css
  #screen-recon{--accent:#d97706;--accent-rgb:217,119,6;}
  #screen-opt{--accent:#0891b2;--accent-rgb:8,145,178;}
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "--bg:#07090d\|--accent:#22d3ee;--accent-rgb:34,211,238\|#fbbf24" index.html
```
Expected: no output (old root values gone). Then run:
```
grep -n -- "--header-bg:#1b2533" index.html
```
Expected: one match inside `:root`.

---

### Task 2: Body background, Header, icon buttons, guide panel

**Files:**
- Modify: `index.html` (body rule ~line 29–39; header block ~line 56–72)

**Interfaces:**
- Consumes: `--header-bg`, `--header-text`, `--header-muted`, `--header-border`, `--accent`, `--accent-rgb` from Task 1.
- Produces: a dark header (matches the reference screenshot) sitting on top of the now-light body; `.iconbtn` restyled for that dark header (it is only ever used inside `<header>` — confirmed by grep, no other usage in the file).

- [ ] **Step 1: Replace the body background gradient**

Old code:
```css
  body{
    font-family:'Inter',system-ui,-apple-system,'Segoe UI',Roboto,sans-serif;
    background:
      radial-gradient(120% 60% at 50% -10%, rgba(34,211,238,.06), transparent 60%),
      radial-gradient(120% 60% at 50% 110%, rgba(251,191,36,.05), transparent 60%),
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
```

- [ ] **Step 2: Replace the header block (header, brand, mode pill, iconbtn, guide panel)**

Old code:
```css
  /* Header */
  header{display:flex;align-items:center;gap:10px;padding:10px 12px;border-bottom:1px solid var(--border);
    background:linear-gradient(180deg,var(--panel),rgba(17,21,28,.6));backdrop-filter:blur(6px);}
  header .brand{display:flex;align-items:center;gap:8px;font-weight:700;font-size:15px;letter-spacing:.3px;}
  header .brand .ico{color:var(--accent);filter:drop-shadow(0 0 8px rgba(var(--accent-rgb),.5));}
  header .mode{font-size:11px;color:var(--muted);margin-left:auto;text-transform:uppercase;letter-spacing:1px;
    border:1px solid var(--border);border-radius:999px;padding:3px 9px;}
  .iconbtn{background:var(--panel2);border:1px solid var(--border);color:var(--text);border-radius:11px;
    min-width:44px;height:44px;display:flex;align-items:center;justify-content:center;cursor:pointer;transition:var(--t);}
  .iconbtn:active{background:var(--panel3);transform:scale(.94);}
  .iconbtn.active{color:var(--accent);border-color:var(--accent);}
  .guide-panel{overflow:hidden;max-height:0;transition:max-height .22s cubic-bezier(.4,0,.2,1);background:var(--panel2);border-bottom:1px solid var(--border);}
  .guide-panel.open{max-height:280px;}
  .guide-steps{margin:0;padding:10px 16px 12px 34px;display:flex;flex-direction:column;gap:5px;list-style:decimal;}
  .guide-steps li{font-size:12px;color:var(--muted);line-height:1.4;}
  .guide-steps li::marker{color:var(--accent);font-weight:700;}
  .guide-steps b{color:var(--text);}
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
  .guide-panel{overflow:hidden;max-height:0;transition:max-height .22s cubic-bezier(.4,0,.2,1);background:var(--panel2);border-bottom:1px solid var(--border);}
  .guide-panel.open{max-height:280px;}
  .guide-steps{margin:0;padding:10px 16px 12px 34px;display:flex;flex-direction:column;gap:5px;list-style:decimal;}
  .guide-steps li{font-size:12px;color:var(--muted);line-height:1.4;}
  .guide-steps li::marker{color:var(--accent);font-weight:700;}
  .guide-steps b{color:var(--text);}
```

Note: `backdrop-filter:blur(6px)` is dropped — it was only useful for the old translucent gradient header; the new header is a solid color.

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "rgba(17,21,28" index.html
```
Expected: no output in the header rule (it may still appear elsewhere until later tasks — that's fine, re-check globally in Task 8).
Run:
```
grep -n -- "background:var(--header-bg)" index.html
```
Expected: one match.

---

### Task 3: Counters (bento stats) + Home screen cards/hero/tiles

**Files:**
- Modify: `index.html` (counters block ~line 74–90; home/bento block ~line 92–123)

**Interfaces:**
- Consumes: `--panel`, `--border`, `--radius`, `--mono`, `--amber`, `--cyan`, `--sky`, `--accent`, `--accent-rgb` from Task 1.

- [ ] **Step 1: Replace the counter bar block**

Old code:
```css
  /* Counter bar (bento stats) */
  .counters{display:grid;grid-template-columns:repeat(3,1fr);gap:10px;padding:12px;
    background:linear-gradient(180deg,rgba(17,21,28,.7),transparent);}
  .counter{position:relative;background:
      linear-gradient(160deg,var(--panel2),var(--panel));border:1px solid var(--border);
    border-radius:var(--radius);padding:9px 10px 10px;min-width:0;overflow:hidden;}
  .counter .ch{display:flex;align-items:center;gap:5px;color:var(--muted);}
  .counter .ch .ico{width:14px;height:14px;}
  .counter .lbl{font-size:10px;text-transform:uppercase;letter-spacing:1px;}
  .counter .val{font-size:clamp(22px,7.5vw,32px);font-weight:800;line-height:1.05;margin-top:2px;
    font-variant-numeric:tabular-nums;overflow:hidden;text-overflow:ellipsis;}
  .counter.target .val{color:var(--amber);text-shadow:0 0 16px rgba(var(--amber-rgb),.35);}
  .counter.target .ch{color:var(--amber);}
  .counter.boxes .val{color:var(--cyan);text-shadow:0 0 16px rgba(34,211,238,.3);}
  .counter.boxes .ch{color:var(--cyan);}
  .counter.pieces .val{color:var(--sky);}
  .counter.pieces .ch{color:var(--sky);}
```

New code:
```css
  /* Counter bar (bento stats) */
  .counters{display:grid;grid-template-columns:repeat(3,1fr);gap:10px;padding:12px;}
  .counter{position:relative;background:var(--panel);border:1px solid var(--border);
    border-radius:var(--radius);padding:9px 10px 10px;min-width:0;overflow:hidden;
    box-shadow:0 1px 2px rgba(16,24,32,.04);}
  .counter .ch{display:flex;align-items:center;gap:5px;color:var(--muted);}
  .counter .ch .ico{width:14px;height:14px;}
  .counter .lbl{font-size:10px;text-transform:uppercase;letter-spacing:1px;}
  .counter .val{font-family:var(--mono);font-size:clamp(22px,7.5vw,32px);font-weight:800;line-height:1.05;margin-top:2px;
    font-variant-numeric:tabular-nums;overflow:hidden;text-overflow:ellipsis;}
  .counter.target .val{color:var(--amber);}
  .counter.target .ch{color:var(--amber);}
  .counter.boxes .val{color:var(--cyan);}
  .counter.boxes .ch{color:var(--cyan);}
  .counter.pieces .val{color:var(--sky);}
  .counter.pieces .ch{color:var(--sky);}
```

- [ ] **Step 2: Replace the Home bento block**

Old code:
```css
  /* Home — bento */
  .home-body{flex:1;display:flex;flex-direction:column;justify-content:center;padding:14px;}
  .bento{display:grid;grid-template-columns:repeat(2,1fr);gap:12px;}
  .span2{grid-column:1 / -1;}
  .card{position:relative;border:1px solid var(--border);border-radius:18px;overflow:hidden;
    background:linear-gradient(160deg,var(--panel2),var(--panel));
    box-shadow:inset 0 1px 0 rgba(255,255,255,.04), 0 12px 30px -18px rgba(0,0,0,.8);}
  .deco{position:absolute;right:-14px;bottom:-16px;width:104px;height:104px;color:var(--accent);opacity:.10;pointer-events:none;}
  .hero{padding:18px 18px;display:flex;align-items:center;gap:14px;
    background:radial-gradient(120% 140% at 100% 0%, rgba(34,211,238,.12), transparent 55%), linear-gradient(160deg,var(--panel2),var(--panel));}
  .hero .badge{width:54px;height:54px;border-radius:15px;display:flex;align-items:center;justify-content:center;flex:none;
    background:linear-gradient(160deg,rgba(34,211,238,.2),rgba(34,211,238,.06));border:1px solid var(--border2);color:var(--accent);
    box-shadow:0 0 24px -4px rgba(34,211,238,.4);}
  .hero .badge .ico{width:30px;height:30px;}
  .hero h1{margin:0;font-size:20px;font-weight:800;letter-spacing:.2px;}
  .hero p{margin:3px 0 0;font-size:12.5px;color:var(--muted);line-height:1.35;}
  .home-h{font-size:11px;color:var(--faint);text-transform:uppercase;letter-spacing:2px;text-align:center;margin:0 0 12px;}

  .tile{cursor:pointer;padding:18px 16px;display:flex;flex-direction:column;gap:7px;min-height:128px;transition:var(--t);}
  .tile:active{transform:scale(.985);}
  .tile .tbadge{width:46px;height:46px;border-radius:13px;display:flex;align-items:center;justify-content:center;
    border:1px solid var(--border2);margin-bottom:2px;}
  .tile .kick{font-size:11px;font-weight:800;letter-spacing:1.5px;}
  .tile .t{font-size:18px;font-weight:800;line-height:1.1;}
  .tile .d{font-size:12.5px;color:var(--muted);line-height:1.4;}
  .tile.recon{--accent:#fbbf24;--accent-rgb:251,191,36;}
  .tile.recon .deco,.tile.recon .kick{color:var(--amber);}
  .tile.recon .tbadge{background:linear-gradient(160deg,rgba(251,191,36,.18),rgba(251,191,36,.05));color:var(--amber);box-shadow:0 0 22px -6px rgba(251,191,36,.5);}
  .tile.recon{box-shadow:inset 0 1px 0 rgba(255,255,255,.04), 0 12px 30px -18px rgba(0,0,0,.8), inset 0 0 0 1px rgba(251,191,36,.06);}
  .tile.optm{--accent:#22d3ee;--accent-rgb:34,211,238;}
  .tile.optm .deco,.tile.optm .kick{color:var(--cyan);}
  .tile.optm .tbadge{background:linear-gradient(160deg,rgba(34,211,238,.18),rgba(34,211,238,.05));color:var(--cyan);box-shadow:0 0 22px -6px rgba(34,211,238,.5);}
```

New code:
```css
  /* Home — bento */
  .home-body{flex:1;display:flex;flex-direction:column;justify-content:center;padding:14px;}
  .bento{display:grid;grid-template-columns:repeat(2,1fr);gap:12px;}
  .span2{grid-column:1 / -1;}
  .card{position:relative;border:1px solid var(--border);border-radius:18px;overflow:hidden;
    background:var(--panel);
    box-shadow:0 1px 2px rgba(16,24,32,.04), 0 6px 16px -8px rgba(16,24,32,.08);}
  .deco{position:absolute;right:-14px;bottom:-16px;width:104px;height:104px;color:var(--accent);opacity:.07;pointer-events:none;}
  .hero{padding:18px 18px;display:flex;align-items:center;gap:14px;background:var(--panel);}
  .hero .badge{width:54px;height:54px;border-radius:15px;display:flex;align-items:center;justify-content:center;flex:none;
    background:rgba(8,145,178,.1);border:1px solid var(--border2);color:var(--accent);}
  .hero .badge .ico{width:30px;height:30px;}
  .hero h1{margin:0;font-size:20px;font-weight:800;letter-spacing:.2px;}
  .hero p{margin:3px 0 0;font-size:12.5px;color:var(--muted);line-height:1.35;}
  .home-h{font-size:11px;color:var(--faint);text-transform:uppercase;letter-spacing:2px;text-align:center;margin:0 0 12px;}

  .tile{cursor:pointer;padding:18px 16px;display:flex;flex-direction:column;gap:7px;min-height:128px;transition:var(--t);}
  .tile:active{transform:scale(.985);}
  .tile .tbadge{width:46px;height:46px;border-radius:13px;display:flex;align-items:center;justify-content:center;
    border:1px solid var(--border2);margin-bottom:2px;}
  .tile .kick{font-size:11px;font-weight:800;letter-spacing:1.5px;}
  .tile .t{font-size:18px;font-weight:800;line-height:1.1;}
  .tile .d{font-size:12.5px;color:var(--muted);line-height:1.4;}
  .tile.recon{--accent:#d97706;--accent-rgb:217,119,6;}
  .tile.recon .deco,.tile.recon .kick{color:var(--amber);}
  .tile.recon .tbadge{background:rgba(217,119,6,.12);color:var(--amber);}
  .tile.optm{--accent:#0891b2;--accent-rgb:8,145,178;}
  .tile.optm .deco,.tile.optm .kick{color:var(--cyan);}
  .tile.optm .tbadge{background:rgba(8,145,178,.12);color:var(--cyan);}
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "text-shadow\|0 0 2[24]px\|0 0 16px\|rgba(0,0,0,.8)" index.html
```
Expected: no matches remaining in the counters/home block (some may still exist further down the file — that's fine, those are cleaned up in later tasks).

---

### Task 4: Fields/inputs (incl. scan field) + Buttons

**Files:**
- Modify: `index.html` (fields block ~line 132–147; buttons block ~line 149–159)

**Interfaces:**
- Consumes: `--panel`, `--panel2`, `--border`, `--accent-rgb`, `--mono`, `--on-accent`, `--dangerbg`, `--danger`, `--danger-rgb` from Task 1.
- Produces: every `.field input` (covers `#in-pick`, `#in-rscan`, `#in-target`, `#in-oscan` — all share the `.field` markup, confirmed by grep) renders in `var(--mono)`.

- [ ] **Step 1: Replace the Fields block**

Old code:
```css
  /* Fields */
  .field{margin-bottom:11px;}
  .field label{display:flex;align-items:center;gap:5px;font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:1px;margin-bottom:5px;}
  .field label .ico{width:13px;height:13px;}
  .inwrap{position:relative;}
  .inwrap>.ico{position:absolute;left:12px;top:50%;transform:translateY(-50%);color:var(--muted);width:18px;height:18px;}
  .field input{width:100%;background:var(--panel);border:1px solid var(--border);color:var(--text);
    border-radius:12px;padding:13px 12px 13px 38px;font-size:18px;outline:none;transition:var(--t);}
  .field input:focus{border-color:var(--accent);box-shadow:0 0 0 3px rgba(var(--accent-rgb),.22);}
  .field.noicon input{padding-left:12px;}
  .scanfield input{font-size:20px;font-weight:600;border-color:rgba(var(--accent-rgb),.55);background:#0c1219;
    box-shadow:inset 0 0 22px -10px rgba(var(--accent-rgb),.5);}
  .scanfield input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.3),inset 0 0 22px -10px rgba(var(--accent-rgb),.5);}
  .scanfield .ico{color:var(--accent);}
  .row2{display:flex;gap:10px;}
  .row2 .field{flex:1;}
```

New code:
```css
  /* Fields */
  .field{margin-bottom:11px;}
  .field label{display:flex;align-items:center;gap:5px;font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:1px;margin-bottom:5px;}
  .field label .ico{width:13px;height:13px;}
  .inwrap{position:relative;}
  .inwrap>.ico{position:absolute;left:12px;top:50%;transform:translateY(-50%);color:var(--muted);width:18px;height:18px;}
  .field input{width:100%;background:var(--panel);border:1px solid var(--border);color:var(--text);
    border-radius:12px;padding:13px 12px 13px 38px;font-size:18px;outline:none;transition:var(--t);font-family:var(--mono);}
  .field input:focus{border-color:var(--accent);box-shadow:0 0 0 3px rgba(var(--accent-rgb),.18);}
  .field.noicon input{padding-left:12px;}
  .scanfield input{font-size:20px;font-weight:600;border-color:rgba(var(--accent-rgb),.55);background:var(--panel2);}
  .scanfield input:focus{box-shadow:0 0 0 3px rgba(var(--accent-rgb),.22);}
  .scanfield .ico{color:var(--accent);}
  .row2{display:flex;gap:10px;}
  .row2 .field{flex:1;}
```

- [ ] **Step 2: Replace the Buttons block**

Old code:
```css
  /* Buttons */
  .btn{display:inline-flex;align-items:center;justify-content:center;gap:7px;width:100%;
    background:var(--accent);color:#04181c;border:none;border-radius:12px;padding:13px;font-size:15px;font-weight:800;cursor:pointer;
    transition:var(--t);}
  .btn .ico{width:19px;height:19px;stroke-width:2.4;}
  .btn:active{transform:scale(.98);filter:brightness(.96);}
  .btn.sec{background:var(--panel2);color:var(--text);border:1px solid var(--border2);}
  .btn.danger{background:var(--dangerbg);color:#fca5a5;border:1px solid #5c2626;}
  .btn.blue{background:var(--accent);color:#04181c;}
  .btn:disabled{opacity:.45;cursor:not-allowed;}
  .btns{display:flex;gap:10px;margin-top:10px;}
```

New code:
```css
  /* Buttons */
  .btn{display:inline-flex;align-items:center;justify-content:center;gap:7px;width:100%;
    background:var(--accent);color:var(--on-accent);border:none;border-radius:12px;padding:13px;font-size:15px;font-weight:800;cursor:pointer;
    transition:var(--t);}
  .btn .ico{width:19px;height:19px;stroke-width:2.4;}
  .btn:active{transform:scale(.98);filter:brightness(.96);}
  .btn.sec{background:var(--panel2);color:var(--text);border:1px solid var(--border2);}
  .btn.danger{background:var(--dangerbg);color:var(--danger);border:1px solid rgba(var(--danger-rgb),.35);}
  .btn.blue{background:var(--accent);color:var(--on-accent);}
  .btn:disabled{opacity:.45;cursor:not-allowed;}
  .btns{display:flex;gap:10px;margin-top:10px;}
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "#0c1219\|#04181c\|#fca5a5\|#5c2626" index.html
```
Expected: no matches in the fields/buttons block (some may still appear further down — cleaned up in Task 6/8).

---

### Task 5: Panels / banners / kv / progress bar / live band

**Files:**
- Modify: `index.html` (panels/banners block ~line 161–179; live band block ~line 181–189)

**Interfaces:**
- Consumes: `--panel2`, `--border`, `--border2`, `--okbg`, `--ok-rgb`, `--ok`, `--dangerbg`, `--danger-rgb`, `--danger`, `--mono` from Task 1.

- [ ] **Step 1: Replace the Panels/banners block**

Old code:
```css
  /* Panels / banners */
  .panel{background:var(--panel2);border:1px solid var(--border);border-radius:var(--radius);padding:12px;margin-bottom:10px;}
  .banner{display:flex;align-items:center;gap:12px;border-radius:var(--radius);padding:14px;margin-bottom:10px;border:1px solid;}
  .banner .ico{width:30px;height:30px;flex:none;}
  .banner .bt{font-weight:800;font-size:15px;line-height:1.15;}
  .banner .bb{font-size:23px;font-weight:800;margin-top:2px;font-variant-numeric:tabular-nums;}
  .banner.ok{background:var(--okbg);border-color:#1f6f54;color:#86efc6;box-shadow:0 0 26px -10px rgba(var(--ok-rgb),.5);}
  .banner.ok .ico{color:var(--ok);}
  .banner.bad{background:var(--dangerbg);border-color:#7a2b2b;color:#fca5a5;box-shadow:0 0 26px -10px rgba(var(--danger-rgb),.5);}
  .banner.bad .ico{color:var(--danger);}
  .banner.idle{background:var(--panel2);border-color:var(--border);color:var(--muted);}
  .banner.idle .ico{color:var(--faint);}
  .kv{display:flex;justify-content:space-between;font-size:14px;padding:4px 0;border-bottom:1px dashed #283040;}
  .kv:last-child{border-bottom:none;}
  .kv .k{color:var(--muted);} .kv .v{font-weight:700;font-variant-numeric:tabular-nums;}
  .pct-wrap{margin-top:9px;}
  .pct-bar{height:10px;background:#0a0f16;border:1px solid var(--border);border-radius:6px;overflow:hidden;}
  .pct-fill{height:100%;background:linear-gradient(90deg,var(--accent),var(--sky));transition:width var(--t);}
  .hint{font-size:12px;color:var(--muted);margin-top:7px;line-height:1.45;}
```

New code:
```css
  /* Panels / banners */
  .panel{background:var(--panel2);border:1px solid var(--border);border-radius:var(--radius);padding:12px;margin-bottom:10px;}
  .banner{display:flex;align-items:center;gap:12px;border-radius:var(--radius);padding:14px;margin-bottom:10px;border:1px solid;}
  .banner .ico{width:30px;height:30px;flex:none;}
  .banner .bt{font-weight:800;font-size:15px;line-height:1.15;}
  .banner .bb{font-family:var(--mono);font-size:23px;font-weight:800;margin-top:2px;font-variant-numeric:tabular-nums;}
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
  .pct-fill{height:100%;background:linear-gradient(90deg,var(--accent),var(--sky));transition:width var(--t);}
  .hint{font-size:12px;color:var(--muted);margin-top:7px;line-height:1.45;}
```

- [ ] **Step 2: Replace the Live band block**

Old code:
```css
  /* Live band */
  .band{display:flex;align-items:center;gap:11px;background:var(--panel2);border:1px solid var(--border);
    border-left:4px solid var(--accent);border-radius:12px;padding:10px 12px;margin-bottom:11px;}
  .band .ico{color:var(--accent);width:24px;height:24px;flex:none;}
  .band.exact{border-left-color:var(--ok);} .band.exact .ico{color:var(--ok);}
  .band.none{border-left-color:var(--border);color:var(--muted);} .band.none .ico{color:var(--faint);}
  .band .b1{font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:.5px;}
  .band .b2{font-size:18px;font-weight:800;font-variant-numeric:tabular-nums;line-height:1.1;}
  .band .b2 .sm{font-size:12px;color:var(--muted);font-weight:600;}
```

New code:
```css
  /* Live band */
  .band{display:flex;align-items:center;gap:11px;background:var(--panel2);border:1px solid var(--border);
    border-left:4px solid var(--accent);border-radius:12px;padding:10px 12px;margin-bottom:11px;}
  .band .ico{color:var(--accent);width:24px;height:24px;flex:none;}
  .band.exact{border-left-color:var(--ok);} .band.exact .ico{color:var(--ok);}
  .band.none{border-left-color:var(--border);color:var(--muted);} .band.none .ico{color:var(--faint);}
  .band .b1{font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:.5px;}
  .band .b2{font-family:var(--mono);font-size:18px;font-weight:800;font-variant-numeric:tabular-nums;line-height:1.1;}
  .band .b2 .sm{font-size:12px;color:var(--muted);font-weight:600;}
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "#1f6f54\|#86efc6\|#7a2b2b\|#283040\|#0a0f16" index.html
```
Expected: no matches.

---

### Task 6: Chips + Edit overlay

**Files:**
- Modify: `index.html` (chips block ~line 191–204; edit overlay block ~line 205–216)

**Interfaces:**
- Consumes: `--panel`, `--border`, `--mono`, `--okbg`, `--ok`, `--accent-rgb`, `--danger`, `--danger-rgb` from Task 1.

- [ ] **Step 1: Replace the Chips block**

Old code:
```css
  /* Chips */
  .chips{display:flex;flex-wrap:wrap;gap:8px;}
  #rc-chips,#op-chips{max-height:30vh;overflow-y:auto;overflow-x:hidden;align-content:flex-start;scrollbar-width:thin;scroll-behavior:smooth;}
  .chip{background:var(--panel);border:1px solid var(--border);border-radius:11px;padding:7px 9px;min-width:62px;text-align:center;transition:var(--t);position:relative;}
  .chip .bi{font-size:10px;color:var(--muted);letter-spacing:.5px;font-weight:600;}
  .chip .pc{font-size:18px;font-weight:800;font-variant-numeric:tabular-nums;line-height:1.1;}
  .chip.pull{background:linear-gradient(160deg,#0f2a20,#0c2018);border-color:var(--ok);box-shadow:0 0 16px -6px rgba(var(--ok-rgb),.6);}
  .chip.pull .pc{color:#86efc6;}
  .chip.newest{border-color:var(--accent);box-shadow:0 0 0 1px rgba(var(--accent-rgb),.4),0 0 16px -6px rgba(var(--accent-rgb),.7);}
  .chip.editable{padding-right:22px;}
  .chip-pc{font-size:18px;font-weight:800;font-variant-numeric:tabular-nums;line-height:1.1;cursor:pointer;border-radius:5px;padding:1px 3px;margin:-1px -3px;transition:background var(--t);}
  .chip-pc:active{background:rgba(var(--accent-rgb),.18);}
  .chip-del{position:absolute;top:4px;right:4px;width:20px;height:20px;border:none;background:none;color:var(--muted);cursor:pointer;border-radius:5px;font-size:13px;font-weight:700;line-height:1;display:flex;align-items:center;justify-content:center;padding:0;touch-action:manipulation;transition:var(--t);}
  .chip-del:active{color:var(--danger);background:rgba(var(--danger-rgb),.18);}
```

New code:
```css
  /* Chips */
  .chips{display:flex;flex-wrap:wrap;gap:8px;}
  #rc-chips,#op-chips{max-height:30vh;overflow-y:auto;overflow-x:hidden;align-content:flex-start;scrollbar-width:thin;scroll-behavior:smooth;}
  .chip{background:var(--panel);border:1px solid var(--border);border-radius:11px;padding:7px 9px;min-width:62px;text-align:center;transition:var(--t);position:relative;}
  .chip .bi{font-family:var(--mono);font-size:10px;color:var(--muted);letter-spacing:.5px;font-weight:600;}
  .chip .pc{font-family:var(--mono);font-size:18px;font-weight:800;font-variant-numeric:tabular-nums;line-height:1.1;}
  .chip.pull{background:var(--okbg);border-color:var(--ok);}
  .chip.pull .pc{color:#14532d;}
  .chip.newest{border-color:var(--accent);box-shadow:0 0 0 1px rgba(var(--accent-rgb),.3);}
  .chip.editable{padding-right:22px;}
  .chip-pc{font-family:var(--mono);font-size:18px;font-weight:800;font-variant-numeric:tabular-nums;line-height:1.1;cursor:pointer;border-radius:5px;padding:1px 3px;margin:-1px -3px;transition:background var(--t);}
  .chip-pc:active{background:rgba(var(--accent-rgb),.14);}
  .chip-del{position:absolute;top:4px;right:4px;width:20px;height:20px;border:none;background:none;color:var(--muted);cursor:pointer;border-radius:5px;font-size:13px;font-weight:700;line-height:1;display:flex;align-items:center;justify-content:center;padding:0;touch-action:manipulation;transition:var(--t);}
  .chip-del:active{color:var(--danger);background:rgba(var(--danger-rgb),.14);}
```

- [ ] **Step 2: Replace the Edit overlay block**

Old code:
```css
  #edit-overlay{display:none;position:fixed;inset:0;background:rgba(0,0,0,.72);z-index:200;align-items:center;justify-content:center;}
  #edit-overlay.open{display:flex;}
  #edit-card{background:var(--panel);border:1px solid var(--border2);border-radius:18px;padding:20px;width:min(320px,90vw);display:flex;flex-direction:column;gap:14px;}
  #edit-title{font-weight:700;font-size:15px;color:var(--text);}
  #edit-input{font-family:inherit;font-size:28px;font-weight:800;background:#0c1219;border:1px solid var(--border2);border-radius:11px;color:var(--text);padding:10px 14px;width:100%;text-align:center;font-variant-numeric:tabular-nums;}
  #edit-input:focus{border-color:var(--accent);outline:none;box-shadow:0 0 0 3px rgba(var(--accent-rgb),.22);}
  #edit-btns{display:flex;gap:10px;}
  #edit-btns button{flex:1;height:44px;border-radius:11px;border:1px solid var(--border);font-family:inherit;font-size:14px;font-weight:600;cursor:pointer;transition:var(--t);}
  #edit-cancel{background:var(--panel2);color:var(--muted);}
  #edit-save{background:var(--accent);color:var(--bg);border-color:var(--accent);}
  .sec-title{display:flex;align-items:center;gap:6px;font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:1.5px;margin:16px 0 8px;}
  .sec-title .ico{width:14px;height:14px;color:var(--faint);}
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
  #edit-btns{display:flex;gap:10px;}
  #edit-btns button{flex:1;height:44px;border-radius:11px;border:1px solid var(--border);font-family:inherit;font-size:14px;font-weight:600;cursor:pointer;transition:var(--t);}
  #edit-cancel{background:var(--panel2);color:var(--muted);}
  #edit-save{background:var(--accent);color:var(--on-accent);border-color:var(--accent);}
  .sec-title{display:flex;align-items:center;gap:6px;font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:1.5px;margin:16px 0 8px;}
  .sec-title .ico{width:14px;height:14px;color:var(--faint);}
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "#0f2a20\|#0c2018\|#86efc6\|#0c1219\|rgba(0,0,0,.72)\|var(--bg);border-color:var(--accent)" index.html
```
Expected: no matches.

---

### Task 7: Optimize "Board" dashboard (statgrid / stat / item / palcard / smallbtn)

**Files:**
- Modify: `index.html` (dashboard block ~line 227–254)

**Interfaces:**
- Consumes: `--panel`, `--panel2`, `--border`, `--border2`, `--panel3`, `--mono`, `--cyan`, `--sky`, `--amber`, `--ok` from Task 1.
- No JS changes — `renderBoard()` (unchanged) emits the exact same class names (`statgrid`, `stat a/b/c/d`, `item`, `palcard`, `smallbtn`) this task restyles.

- [ ] **Step 1: Replace the Dashboard block**

Old code:
```css
  /* Dashboard */
  .statgrid{display:grid;grid-template-columns:repeat(2,1fr);gap:10px;margin-bottom:4px;}
  .stat{background:linear-gradient(160deg,var(--panel2),var(--panel));border:1px solid var(--border);border-radius:13px;padding:11px 12px;}
  .stat .sh{display:flex;align-items:center;gap:6px;color:var(--muted);font-size:11px;text-transform:uppercase;letter-spacing:1px;}
  .stat .sh .ico{width:15px;height:15px;}
  .stat .sv{font-size:25px;font-weight:800;font-variant-numeric:tabular-nums;margin-top:3px;}
  .stat.a .sv{color:var(--cyan);} .stat.a .sh{color:var(--cyan);}
  .stat.b .sv{color:var(--sky);} .stat.b .sh{color:var(--sky);}
  .stat.c .sv{color:var(--amber);} .stat.c .sh{color:var(--amber);}
  .stat.d .sv{color:var(--ok);} .stat.d .sh{color:var(--ok);}
  .item{display:flex;justify-content:space-between;align-items:center;gap:10px;padding:9px 11px;background:var(--panel);
    border:1px solid var(--border);border-radius:11px;margin-bottom:6px;}
  .item .szwrap{display:flex;align-items:center;gap:9px;min-width:0;}
  .item .szwrap .ico{width:18px;height:18px;color:var(--accent);flex:none;}
  .item .sz{font-weight:800;font-size:16px;}
  .item .mid{color:var(--muted);font-size:13px;}
  .item .tot{font-weight:800;color:var(--cyan);font-variant-numeric:tabular-nums;}
  .palcard{background:var(--panel2);border:1px solid var(--border);border-radius:13px;padding:11px;margin-bottom:8px;}
  .palcard .ph{display:flex;justify-content:space-between;align-items:center;gap:8px;margin-bottom:7px;}
  .palcard .pl{display:flex;align-items:center;gap:8px;min-width:0;}
  .palcard .pl .ico{width:19px;height:19px;color:var(--cyan);flex:none;}
  .palcard .pn{font-weight:800;}
  .palcard .pr{font-size:12px;color:var(--sky);}
  .palcard .pstat{font-size:13px;color:var(--muted);padding-left:27px;}
  .smallbtn{display:inline-flex;align-items:center;gap:6px;background:var(--panel3);border:1px solid var(--border2);color:var(--text);
    border-radius:9px;padding:8px 11px;font-size:12px;font-weight:700;cursor:pointer;transition:var(--t);min-height:38px;}
  .smallbtn .ico{width:15px;height:15px;}
  .smallbtn:active{transform:scale(.96);}
```

New code:
```css
  /* Dashboard */
  .statgrid{display:grid;grid-template-columns:repeat(2,1fr);gap:10px;margin-bottom:4px;}
  .stat{background:var(--panel);border:1px solid var(--border);border-radius:13px;padding:11px 12px;
    box-shadow:0 1px 2px rgba(16,24,32,.04);}
  .stat .sh{display:flex;align-items:center;gap:6px;color:var(--muted);font-size:11px;text-transform:uppercase;letter-spacing:1px;}
  .stat .sh .ico{width:15px;height:15px;}
  .stat .sv{font-family:var(--mono);font-size:25px;font-weight:800;font-variant-numeric:tabular-nums;margin-top:3px;}
  .stat.a .sv{color:var(--cyan);} .stat.a .sh{color:var(--cyan);}
  .stat.b .sv{color:var(--sky);} .stat.b .sh{color:var(--sky);}
  .stat.c .sv{color:var(--amber);} .stat.c .sh{color:var(--amber);}
  .stat.d .sv{color:var(--ok);} .stat.d .sh{color:var(--ok);}
  .item{display:flex;justify-content:space-between;align-items:center;gap:10px;padding:9px 11px;background:var(--panel);
    border:1px solid var(--border);border-radius:11px;margin-bottom:6px;}
  .item .szwrap{display:flex;align-items:center;gap:9px;min-width:0;}
  .item .szwrap .ico{width:18px;height:18px;color:var(--accent);flex:none;}
  .item .sz{font-family:var(--mono);font-weight:800;font-size:16px;}
  .item .mid{color:var(--muted);font-size:13px;}
  .item .tot{font-family:var(--mono);font-weight:800;color:var(--cyan);font-variant-numeric:tabular-nums;}
  .palcard{background:var(--panel2);border:1px solid var(--border);border-radius:13px;padding:11px;margin-bottom:8px;}
  .palcard .ph{display:flex;justify-content:space-between;align-items:center;gap:8px;margin-bottom:7px;}
  .palcard .pl{display:flex;align-items:center;gap:8px;min-width:0;}
  .palcard .pl .ico{width:19px;height:19px;color:var(--cyan);flex:none;}
  .palcard .pn{font-weight:800;}
  .palcard .pr{font-size:12px;color:var(--sky);}
  .palcard .pstat{font-size:13px;color:var(--muted);padding-left:27px;}
  .smallbtn{display:inline-flex;align-items:center;gap:6px;background:var(--panel3);border:1px solid var(--border2);color:var(--text);
    border-radius:9px;padding:8px 11px;font-size:12px;font-weight:700;cursor:pointer;transition:var(--t);min-height:38px;}
  .smallbtn .ico{width:15px;height:15px;}
  .smallbtn:active{transform:scale(.96);}
```

- [ ] **Step 2: Verify**

Run:
```
grep -n -- "linear-gradient(160deg,var(--panel2),var(--panel))" index.html
```
Expected: no matches (all gradient-card backgrounds have been flattened to solid `var(--panel)`/`var(--panel2)` across Tasks 3 and 7).

---

### Task 8: Bottom nav / action bar / settings sheet / flash toast

**Files:**
- Modify: `index.html` (nav/actbar block ~line 256–267; settings sheet block ~line 269–283; flash block ~line 285–290)

**Interfaces:**
- Consumes: `--panel`, `--border`, `--accent`, `--accent-rgb`, `--dangerbg`, `--danger`, `--danger-rgb`, `--okbg`, `--ok`, `--ok-rgb` from Task 1.

- [ ] **Step 1: Replace the Bottom nav / action bar block**

Old code:
```css
  /* Bottom nav */
  .nav{display:flex;border-top:1px solid var(--border);background:linear-gradient(0deg,var(--panel),rgba(17,21,28,.6));
    padding:5px 6px calc(env(safe-area-inset-bottom,0px) + 5px);}
  .nav button{flex:1;background:none;border:none;color:var(--muted);padding:8px 2px 7px;font-size:11px;font-weight:600;
    display:flex;flex-direction:column;align-items:center;gap:4px;cursor:pointer;border-radius:11px;transition:var(--t);position:relative;}
  .nav button .ico{width:23px;height:23px;}
  .nav button.on{color:var(--accent);}
  .nav button.on::before{content:"";position:absolute;top:-5px;left:50%;transform:translateX(-50%);width:26px;height:3px;
    border-radius:3px;background:var(--accent);box-shadow:0 0 12px rgba(var(--accent-rgb),.8);}
  .actbar{padding:9px 13px calc(env(safe-area-inset-bottom,0px) + 9px);border-top:1px solid var(--border);
    background:var(--panel);display:flex;gap:10px;}
  .actbar .btn{flex:1;}
```

New code:
```css
  /* Bottom nav */
  .nav{display:flex;border-top:1px solid var(--border);background:var(--panel);
    padding:5px 6px calc(env(safe-area-inset-bottom,0px) + 5px);}
  .nav button{flex:1;background:none;border:none;color:var(--muted);padding:8px 2px 7px;font-size:11px;font-weight:600;
    display:flex;flex-direction:column;align-items:center;gap:4px;cursor:pointer;border-radius:11px;transition:var(--t);position:relative;}
  .nav button .ico{width:23px;height:23px;}
  .nav button.on{color:var(--accent);}
  .nav button.on::before{content:"";position:absolute;top:-5px;left:50%;transform:translateX(-50%);width:26px;height:3px;
    border-radius:3px;background:var(--accent);}
  .actbar{padding:9px 13px calc(env(safe-area-inset-bottom,0px) + 9px);border-top:1px solid var(--border);
    background:var(--panel);display:flex;gap:10px;}
  .actbar .btn{flex:1;}
```

- [ ] **Step 2: Replace the Settings sheet block**

Old code:
```css
  /* Settings sheet */
  .overlay{position:fixed;inset:0;background:rgba(0,0,0,.55);display:none;align-items:flex-end;z-index:50;backdrop-filter:blur(3px);}
  .overlay.open{display:flex;}
  .sheet{background:var(--panel);border-top-left-radius:22px;border-top-right-radius:22px;width:100%;max-width:560px;margin:0 auto;
    padding:8px 16px calc(env(safe-area-inset-bottom,0px) + 16px);border:1px solid var(--border);animation:slideup var(--t);}
  @keyframes slideup{from{transform:translateY(20px);opacity:.6}to{transform:translateY(0);opacity:1}}
  .grab{width:38px;height:4px;border-radius:3px;background:var(--border2);margin:6px auto 12px;}
  .sheet h3{margin:0 0 12px;font-size:17px;}
  .opt{display:flex;align-items:center;gap:11px;padding:13px;border:1px solid var(--border);border-radius:12px;margin-bottom:9px;cursor:pointer;background:var(--panel2);transition:var(--t);}
  .opt .oi{width:38px;height:38px;border-radius:10px;display:flex;align-items:center;justify-content:center;flex:none;background:var(--panel3);border:1px solid var(--border2);color:var(--muted);}
  .opt.sel{border-color:var(--accent);background:#0c1f22;}
  .opt.sel .oi{color:var(--accent);border-color:var(--accent);}
  .opt .dot{margin-left:auto;width:20px;height:20px;border-radius:50%;border:2px solid var(--border2);flex:none;}
  .opt.sel .dot{border-color:var(--accent);background:var(--accent);box-shadow:inset 0 0 0 3px var(--panel);}
  .opt .ot{font-weight:700;font-size:14px;} .opt .od{font-size:12px;color:var(--muted);}
```

New code:
```css
  /* Settings sheet */
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
  .opt.sel .oi{color:var(--accent);border-color:var(--accent);}
  .opt .dot{margin-left:auto;width:20px;height:20px;border-radius:50%;border:2px solid var(--border2);flex:none;}
  .opt.sel .dot{border-color:var(--accent);background:var(--accent);box-shadow:inset 0 0 0 3px var(--panel);}
  .opt .ot{font-weight:700;font-size:14px;} .opt .od{font-size:12px;color:var(--muted);}
```

- [ ] **Step 3: Replace the Flash toast block**

Old code:
```css
  .flash{position:fixed;left:50%;top:16px;transform:translateX(-50%) translateY(-8px);display:flex;align-items:center;gap:8px;
    background:var(--dangerbg);color:#fca5a5;border:1px solid #5c2626;padding:9px 15px;border-radius:12px;font-size:13px;font-weight:700;
    z-index:80;opacity:0;transition:var(--t);pointer-events:none;max-width:90%;text-align:center;box-shadow:0 10px 30px -10px rgba(0,0,0,.7);}
  .flash .ico{width:18px;height:18px;}
  .flash.show{opacity:1;transform:translateX(-50%) translateY(0);}
  .flash.good{background:var(--okbg);color:#86efc6;border-color:#1f6f54;}
```

New code:
```css
  .flash{position:fixed;left:50%;top:16px;transform:translateX(-50%) translateY(-8px);display:flex;align-items:center;gap:8px;
    background:var(--dangerbg);color:var(--danger);border:1px solid rgba(var(--danger-rgb),.35);padding:9px 15px;border-radius:12px;font-size:13px;font-weight:700;
    z-index:80;opacity:0;transition:var(--t);pointer-events:none;max-width:90%;text-align:center;box-shadow:0 10px 30px -10px rgba(16,24,32,.25);}
  .flash .ico{width:18px;height:18px;}
  .flash.show{opacity:1;transform:translateX(-50%) translateY(0);}
  .flash.good{background:var(--okbg);color:var(--ok);border-color:rgba(var(--ok-rgb),.35);}
```

- [ ] **Step 4: Verify**

Run:
```
grep -n -- "#0c1f22\|#fca5a5\|#5c2626\|#86efc6\|#1f6f54\|rgba(0,0,0,.5\|rgba(0,0,0,.7\|rgba(17,21,28" index.html
```
Expected: no matches anywhere in the file.

---

### Task 9: Documentation + final whole-file verification

**Files:**
- Modify: `CLAUDE.md` (design language line)
- Verify only: `index.html`

**Interfaces:** none (final step).

- [ ] **Step 1: Update `CLAUDE.md`'s design-language line**

Old code (in `CLAUDE.md`):
```
Single-file warehouse pick tool: **everything lives in `index.html`** (inline CSS + JS, vanilla, no
framework, no build, no deps, no git). Runs offline from `localStorage` on a Zebra TC520L 5" handheld
(Chromium WebView). UI in English; design = dark OLED + bento grid.
```

New code:
```
Single-file warehouse pick tool: **everything lives in `index.html`** (inline CSS + JS, vanilla, no
framework, no build, no deps, no git). Runs offline from `localStorage` on a Zebra TC520L 5" handheld
(Chromium WebView). UI in English; design = light bento grid with a dark header bar (see
`docs/superpowers/specs/2026-06-17-light-bento-restyle-design.md`); codes/quantities use a monospace
system font via `--mono`.
```

- [ ] **Step 2: Run the project's syntax check**

Run (from CLAUDE.md's documented PowerShell one-liner, or the Bash equivalent used throughout this session):
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
Expected: `SYNTAX OK` (confirms the `<script>` block was never touched by this restyle).

- [ ] **Step 3: Whole-file grep for every old hardcoded dark-theme value**

Run:
```
grep -n -- "#07090d\|#0b0e14\|#11151c\|#161c25\|#1c2330\|#232b36\|#2e3845\|#f0f4f8\|#9aa6b2\|#5b6675\|#22d3ee\|#fbbf24\|#7dd3fc\|#34d399\|#0c2a20\|#f87171\|#2c1414\|#0c1219\|#04181c\|#fca5a5\|#5c2626\|#86efc6\|#1f6f54\|#7a2b2b\|#283040\|#0a0f16\|#0f2a20\|#0c2018\|#0c1f22\|rgba(0,0,0,\|rgba(17,21,28" index.html
```
Expected: no matches.

- [ ] **Step 4: Open the app for a manual visual check**

Run:
```
Start-Process index.html
```
Manually confirm: light background, dark header bar with visible icon buttons, amber Reconcile / teal Optimize accents readable on white, codes/quantities rendering in a monospace font, no leftover neon glow effects, print preview (Print slip / Print summary) still plain black-and-white.
