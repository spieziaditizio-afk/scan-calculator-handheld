# Dark-mode Home redesign (emoji list rows) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** When dark mode is active, replace the Home screen's slim header + bento grid with a tall branding header (emoji + app name + tagline + site) and a 4-row emoji list (Reconcile, Optimize, Settings, Printer). Light mode and the Reconcile/Optimize screens are unaffected in both themes.

**Architecture:** New markup (a `.home-header-dark` header block and a `.home-list` block of `.row` elements) is added as siblings to the existing slim header and `.bento` grid inside `#screen-home`. Both new blocks default to `display:none` and are revealed only under `[data-theme="dark"]`, which simultaneously hides the existing slim header/bento — the same CSS-attribute-toggle pattern the codebase already uses for the light/dark theme switch (no JS theming logic changes). The 4 new rows reuse the exact same click handlers as their bento counterparts; no new functions are introduced.

**Tech Stack:** Vanilla HTML/CSS/JS, no build, no deps. (Despite the stale "no git" line in `CLAUDE.md`, this repo is actively using git — see `git log` — so each task below ends with a real commit in the existing `feat:`/`docs:` style.)

## Global Constraints

- Everything lives in `index.html` (CLAUDE.md) — no new files except the `CLAUDE.md` edit in Task 4.
- Emoji are permitted **only** in the 4 Home dark-mode rows added here — this is a deliberate, narrow override of CLAUDE.md's "never emoji" rule, formalized in Task 4. Every other icon in the app (sprite SVGs, light-mode bento tiles, Reconcile/Optimize) stays SVG-only.
- The light theme (bento grid, slim header) must render pixel-for-pixel identical to today — verified by the fact that no existing markup, class, or CSS rule is removed, only new sibling markup gated behind `[data-theme="dark"]`.
- Reconcile and Optimize screens are not touched in either theme.
- `node --check` must pass on the inline script after Task 3 (no JS syntax errors introduced).
- `index.html` currently has a pre-existing **uncommitted** change from an earlier, unrelated edit (Home's Optimize card renamed to "Mix Optimizer" with a new icon/description). Task 1's first step commits that separately so it doesn't get bundled into this feature's commits.

---

### Task 1: Home header — dark-mode branding block

**Files:**
- Modify: `index.html` (Home header markup, lines 400–402; Header CSS block, lines 78–94)

**Interfaces:**
- Produces: `.home-header-dark` block with child elements `.hhd-emoji`, `.hhd-name`, `.hhd-tag`, `#hhd-site`, and a 4th `[data-theme-toggle]` button (reuses the existing global `applyTheme()`/`toggleTheme()` wiring via `querySelectorAll`, already generalized — no JS changes needed for the toggle itself).
- Consumes: `--accent`, `--muted`, `--panel`, `--border`, `--text` tokens (already defined); the existing `.iconbtn` dark-mode styling (`[data-theme="dark"] .iconbtn{...}`, untouched).

- [ ] **Step 1: Commit the pre-existing pending change first, separately**

Run:
```bash
cd "/c/Users/aspiezia/Downloads/scan calculator handheld" && git add index.html && git commit -m "$(cat <<'EOF'
feat: rename Optimize home card to Mix Optimizer with new icon

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```
Expected: commit succeeds; `git status --short` afterwards shows `index.html` no longer listed as modified.

- [ ] **Step 2: Add `home-slim` class to the existing header, and the new dark header block right after it**

Old code:
```html
    <header><span class="brand"><svg class="ico"><use href="#i-warehouse"/></svg>Scan Calculator</span><span class="mode">v0.2</span>
      <button class="iconbtn" data-theme-toggle aria-label="Toggle dark mode"><svg class="ico"><use href="#i-moon"/></svg></button>
    </header>
```

New code:
```html
    <header class="home-slim"><span class="brand"><svg class="ico"><use href="#i-warehouse"/></svg>Scan Calculator</span><span class="mode">v0.2</span>
      <button class="iconbtn" data-theme-toggle aria-label="Toggle dark mode"><svg class="ico"><use href="#i-moon"/></svg></button>
    </header>
    <header class="home-header-dark">
      <button class="iconbtn" data-theme-toggle aria-label="Toggle dark mode"><svg class="ico"><use href="#i-moon"/></svg></button>
      <div class="hhd-emoji">🏭</div>
      <div class="hhd-name">Scan Calculator</div>
      <div class="hhd-tag">Warehouse Tools</div>
      <div class="hhd-site" id="hhd-site"></div>
    </header>
```

- [ ] **Step 3: Add the dark-header CSS after the existing Header block**

Old code:
```css
  [data-theme="dark"] .iconbtn{background:var(--panel2);border-color:var(--border);color:var(--text);}
  [data-theme="dark"] .iconbtn:active{background:var(--panel3);}
```

New code:
```css
  [data-theme="dark"] .iconbtn{background:var(--panel2);border-color:var(--border);color:var(--text);}
  [data-theme="dark"] .iconbtn:active{background:var(--panel3);}
  .home-header-dark{display:none;position:relative;flex-direction:column;align-items:center;text-align:center;
    padding:22px 16px 18px;background:var(--panel);border-bottom:1px solid var(--border);}
  .home-header-dark .iconbtn{position:absolute;top:10px;right:10px;}
  .hhd-emoji{font-size:40px;line-height:1;}
  .hhd-name{font-size:20px;font-weight:800;margin-top:6px;color:var(--text);}
  .hhd-tag{font-size:12px;font-weight:800;letter-spacing:2px;text-transform:uppercase;color:var(--accent);margin-top:2px;}
  .hhd-site{font-size:11px;color:var(--muted);letter-spacing:.5px;margin-top:6px;}
  .hhd-site:empty{display:none;}
  [data-theme="dark"] header.home-slim{display:none;}
  [data-theme="dark"] .home-header-dark{display:flex;}
```

- [ ] **Step 4: Verify**

Run:
```bash
grep -n -- "home-header-dark\|home-slim\|hhd-" index.html
```
Expected: 15 matches (2 header-tag lines + 4 `hhd-*` markup lines + 2 `.home-header-dark` CSS lines + 5 `.hhd-*` CSS lines + 2 `[data-theme="dark"]` toggle lines).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "$(cat <<'EOF'
feat: add dark-mode branding header to Home screen

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

---

### Task 2: Home rows — dark-mode emoji list

**Files:**
- Modify: `index.html` (Home body markup, lines 403–423; Home/bento CSS block, lines 122–158)

**Interfaces:**
- Consumes: existing IDs `tile-recon`, `tile-opt`, `home-settings`, `home-printer`, `home-printer-lbl` (unchanged, still present for light mode).
- Produces: `.home-list` block with rows `#row-recon`, `#row-opt`, `#row-settings`, `#row-printer`, and label element `#home-printer-lbl2` — all consumed by Task 3's JS wiring.

- [ ] **Step 1: Add the `.home-list` block as a sibling of `.bento`, inside `.home-body`**

Old code:
```html
        <div class="card minitile" id="home-settings"><span class="mi"><svg class="ico"><use href="#i-settings"/></svg></span><div><div class="mt">Settings</div><div class="ms">Printer &amp; data</div></div></div>
        <div class="card minitile" id="home-printer"><span class="mi"><svg class="ico"><use href="#i-printer"/></svg></span><div><div class="mt">Printer</div><div class="ms" id="home-printer-lbl">QLn420</div></div></div>
      </div>
    </div>
```

New code:
```html
        <div class="card minitile" id="home-settings"><span class="mi"><svg class="ico"><use href="#i-settings"/></svg></span><div><div class="mt">Settings</div><div class="ms">Printer &amp; data</div></div></div>
        <div class="card minitile" id="home-printer"><span class="mi"><svg class="ico"><use href="#i-printer"/></svg></span><div><div class="mt">Printer</div><div class="ms" id="home-printer-lbl">QLn420</div></div></div>
      </div>
      <div class="home-list">
        <div class="row" id="row-recon">
          <span class="remoji recon">📦</span>
          <div class="rt"><div class="rtt">Count &amp; verify</div><div class="rtd">Count and verify box quantities to ensure they match pick requirements before WMS processing.</div></div>
          <span class="rarrow">›</span>
        </div>
        <div class="row" id="row-opt">
          <span class="remoji opt">🧮</span>
          <div class="rt"><div class="rtt">Mix Optimizer</div><div class="rtd">Calculate the best combination of SPQ and partial boxes across multiple locations to validate stock availability before releasing the WMS pick.</div></div>
          <span class="rarrow">›</span>
        </div>
        <div class="row" id="row-settings">
          <span class="remoji settings">⚙️</span>
          <div class="rt"><div class="rtt">Settings</div><div class="rtd">Printer &amp; data</div></div>
          <span class="rarrow">›</span>
        </div>
        <div class="row" id="row-printer">
          <span class="remoji printer">🖨️</span>
          <div class="rt"><div class="rtt">Printer</div><div class="rtd" id="home-printer-lbl2">QLn420</div></div>
          <span class="rarrow">›</span>
        </div>
      </div>
    </div>
```

- [ ] **Step 2: Add the row/badge CSS after the existing `.minitile` rules, before `/* Fields */`**

Old code:
```css
  .minitile .mt{font-size:13px;font-weight:700;}
  .minitile .ms{font-size:11px;color:var(--muted);}

  /* Fields */
```

New code:
```css
  .minitile .mt{font-size:13px;font-weight:700;}
  .minitile .ms{font-size:11px;color:var(--muted);}

  .home-list{display:none;flex-direction:column;gap:10px;}
  [data-theme="dark"] .bento{display:none;}
  [data-theme="dark"] .home-list{display:flex;}
  .row{display:flex;align-items:center;gap:12px;background:var(--panel);border:1px solid var(--border);
    border-radius:16px;padding:13px 14px;cursor:pointer;transition:var(--t);}
  .row:active{transform:scale(.985);}
  .remoji{width:46px;height:46px;border-radius:13px;display:flex;align-items:center;justify-content:center;
    flex:none;font-size:22px;line-height:1;}
  .remoji.recon{background:rgba(251,191,36,.18);}
  .remoji.opt{background:rgba(34,211,238,.18);}
  .remoji.settings{background:var(--panel3);}
  .remoji.printer{background:rgba(125,211,252,.18);}
  .row .rt{flex:1;min-width:0;}
  .row .rtt{font-size:15px;font-weight:800;line-height:1.2;color:var(--text);}
  .row .rtd{font-size:12px;color:var(--muted);line-height:1.35;margin-top:2px;}
  .row .rarrow{font-size:20px;color:var(--faint);flex:none;}

  /* Fields */
```

- [ ] **Step 3: Verify**

Run:
```bash
grep -n -- "home-list\|remoji\|row-recon\|row-opt\|row-settings\|row-printer" index.html
```
Expected: 16 matches (9 markup lines: `.home-list` div, 4 `.row` divs, 4 `.remoji` spans — plus 7 CSS lines: `.home-list`, the dark-mode `.home-list` override, and the 5 `.remoji*` rules).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "$(cat <<'EOF'
feat: add dark-mode emoji list rows to Home screen

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

---

### Task 3: JS wiring — row click handlers + dynamic label/site sync

**Files:**
- Modify: `index.html` (click bindings, lines 1120–1123; `render()`, line 738)

**Interfaces:**
- Consumes: `#row-recon`, `#row-opt`, `#row-settings`, `#row-printer`, `#home-printer-lbl2`, `#hhd-site` (from Task 1/2); existing `setMode(m)` and `openSettings()` functions (unchanged signatures).

- [ ] **Step 1: Bind the new rows to the same handlers as their bento counterparts**

Old code:
```js
$('tile-recon').onclick=()=>setMode('reconcile');
$('tile-opt').onclick  =()=>setMode('optimize');
$('home-settings').onclick=()=>openSettings();
$('home-printer').onclick=()=>openSettings();
```

New code:
```js
$('tile-recon').onclick=$('row-recon').onclick=()=>setMode('reconcile');
$('tile-opt').onclick  =$('row-opt').onclick  =()=>setMode('optimize');
$('home-settings').onclick=$('row-settings').onclick=()=>openSettings();
$('home-printer').onclick=$('row-printer').onclick=()=>openSettings();
```

- [ ] **Step 2: Sync the new dark-mode label/site elements inside `render()`**

Old code:
```js
  if(cfg.mode==='home') $('home-printer-lbl').textContent = cfg.printer==='qln420'?'QLn420':'A4';
```

New code:
```js
  if(cfg.mode==='home'){
    const printerLbl = cfg.printer==='qln420'?'QLn420':'A4';
    $('home-printer-lbl').textContent = printerLbl;
    $('home-printer-lbl2').textContent = printerLbl;
    $('hhd-site').textContent = cfg.site || '';
  }
```

- [ ] **Step 3: Run the project's syntax check**

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

- [ ] **Step 4: Verify**

Run:
```bash
grep -n -- "row-recon').onclick\|row-opt').onclick\|row-settings').onclick\|row-printer').onclick" index.html
```
Expected: 4 matches.

Run:
```bash
grep -n -- "home-printer-lbl2\|hhd-site" index.html
```
Expected: 6 matches (2 markup/CSS-adjacent occurrences of `home-printer-lbl2` from Task 2's markup + this task's JS; 4 occurrences of `hhd-site` from Task 1's markup + CSS + this task's JS).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "$(cat <<'EOF'
feat: wire dark-mode Home rows to existing click handlers

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

---

### Task 4: CLAUDE.md update + final manual verification

**Files:**
- Modify: `CLAUDE.md` (icon rule, lines 51–52)

**Interfaces:**
- Consumes: nothing from earlier tasks (documentation only).

- [ ] **Step 1: Narrow the "never emoji" rule**

Old code:
```
- Icons: inline SVG sprite (`<use href="#i-…">`), never emoji. The sprite has NO help/question icon —
  use a plain `?` text character inside `.iconbtn` when a help button is needed.
```

New code:
```
- Icons: inline SVG sprite (`<use href="#i-…">`), never emoji — **except** the 4 Home-screen list rows
  shown in dark mode (Reconcile/Optimize/Settings/Printer: 📦🧮⚙️🖨️), which use real emoji per
  `docs/superpowers/specs/2026-06-22-dark-emoji-home-redesign-design.md`. The sprite has NO help/question
  icon — use a plain `?` text character inside `.iconbtn` when a help button is needed.
```

- [ ] **Step 2: Verify**

Run:
```bash
grep -n -- "dark-emoji-home-redesign" CLAUDE.md
```
Expected: 1 match.

- [ ] **Step 3: Manual visual check**

Run:
```bash
Start-Process index.html
```
Manually confirm:
1. App opens in **light mode** by default — Home still shows the 2-card bento grid + 2 mini-tiles + slim header, unchanged from before this plan.
2. Tap the moon-icon button to switch to **dark mode** — Home now shows the tall branding header (🏭 emoji, "Scan Calculator", "WAREHOUSE TOOLS" tagline, and — if a Site name is set in Settings — the site line beneath it) and the 4-row emoji list (📦 Count & verify, 🧮 Mix Optimizer, ⚙️ Settings, 🖨️ Printer showing the current printer name).
3. Tap each of the 4 rows and confirm it opens the same destination as its bento equivalent (Reconcile screen, Optimize screen, Settings sheet, Settings sheet).
4. Switch to the Reconcile and Optimize screens while in dark mode and confirm they are visually unchanged (slim header, no emoji, no list rows) — only the existing dark-OLED recoloring applies.
5. Toggle back to light mode and confirm Home returns to the bento grid exactly as it looked before this plan.

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "$(cat <<'EOF'
docs: scope CLAUDE.md emoji rule to Home dark-mode rows

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```
