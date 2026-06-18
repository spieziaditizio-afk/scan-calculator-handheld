# Pallet Inventory Label (Reconcile) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a second, independent print action to Reconcile — a plain inventory label (box grid + total boxes/pieces + site/date/printer footer + an unconditional VERIFIED badge) that the operator can reprint any time a box is removed or a quantity is corrected, with no dependency on the pick target.

**Architecture:** Everything lives in the single `index.html` file (inline CSS + JS). One new global config field (`cfg.site`), one new Settings input, one new button next to Reconcile's "Scanned boxes" list, one new print-CSS block, and two new JS functions (`printerLabel()`, `printPalletLabel()`) that build a plain HTML string and hand it to the existing `doPrint()` pipeline. No changes to `recon` state, `logRecon()`, or `printReconcile()`.

**Tech Stack:** Vanilla JS, inline CSS, `localStorage`. No build step, no test framework — verification is `node --check` for syntax plus grep-counted structural checks (this codebase's established substitute for unit tests on markup/wiring changes) and a small Node script for the one piece of string-building logic.

## Global Constraints

- Everything is inline in `index.html` — no new files, no dependencies.
- No emoji anywhere; the checkmark on the label is the plain-text glyph `✓` (U+2713 / `&#10003;`), not an emoji.
- Print output stays plain B/W text — the print iframe is a separate document with no access to the main page's CSS vars or SVG sprite (reuse the existing `pchip()`/`printCss()` plain-text patterns).
- This feature must never call `logRecon()`, never mutate `recon`, and never touch `packcalc_history` — it is a read-only print of `recon.boxes`.
- Reconcile has no rack/per-box location concept (deliberately removed) — `cfg.site` is one global value for the whole app, not a per-box field. Do not add anything resembling a per-box location to Reconcile.
- Never commit without being explicitly asked — this plan's tasks end with verification steps, not git commits.

---

### Task 1: `cfg.site` field + Settings UI

**Files:**
- Modify: `index.html` (cfg default object, ~line 562; Settings sheet markup, ~lines 514-517; `openSettings()` + new input listener, ~lines 1094-1097)

**Interfaces:**
- Produces: `cfg.site` (string, default `''`), persisted via the existing `saveCfg()` → `store.set(LS.cfg,cfg)`. Read by Task 4's `printPalletLabel()`.

- [ ] **Step 1: Add `site:''` to the `cfg` default object**

Find (around line 562):
```js
const cfg     = Object.assign({ printer:'qln420', mode:'home', theme:'light' }, store.get(LS.cfg,{}));
```
Replace with:
```js
const cfg     = Object.assign({ printer:'qln420', mode:'home', theme:'light', site:'' }, store.get(LS.cfg,{}));
```

- [ ] **Step 2: Add a "Site" section to the Settings sheet**

Find (the printer options block, ~lines 514-517):
```html
    <div class="sec-title" style="margin-top:0"><svg class="ico"><use href="#i-printer"/></svg>Printer</div>
    <div class="opt" data-printer="qln420"><span class="oi"><svg class="ico"><use href="#i-printer"/></svg></span><div><div class="ot">Zebra QLn420</div><div class="od">Mobile thermal label · 113 × 63 mm</div></div><span class="dot"></span></div>
    <div class="opt" data-printer="a4"><span class="oi"><svg class="ico"><use href="#i-printer"/></svg></span><div><div class="ot">Regular printer</div><div class="od">Standard A4 page</div></div><span class="dot"></span></div>
    <div class="sec-title"><svg class="ico"><use href="#i-database"/></svg>Data</div>
```
Replace with:
```html
    <div class="sec-title" style="margin-top:0"><svg class="ico"><use href="#i-printer"/></svg>Printer</div>
    <div class="opt" data-printer="qln420"><span class="oi"><svg class="ico"><use href="#i-printer"/></svg></span><div><div class="ot">Zebra QLn420</div><div class="od">Mobile thermal label · 113 × 63 mm</div></div><span class="dot"></span></div>
    <div class="opt" data-printer="a4"><span class="oi"><svg class="ico"><use href="#i-printer"/></svg></span><div><div class="ot">Regular printer</div><div class="od">Standard A4 page</div></div><span class="dot"></span></div>
    <div class="sec-title"><svg class="ico"><use href="#i-pin"/></svg>Site</div>
    <div class="field"><label>Site / warehouse name</label><div class="inwrap"><svg class="ico"><use href="#i-pin"/></svg><input id="in-site" placeholder="e.g. SEVENUM" autocomplete="off"></div></div>
    <div class="sec-title"><svg class="ico"><use href="#i-database"/></svg>Data</div>
```

- [ ] **Step 3: Populate `#in-site` on open, save it on input**

Find (~lines 1094-1097):
```js
function openSettings(){ $('overlay').classList.add('open'); }
$('set-close').onclick=()=>$('overlay').classList.remove('open');
$('overlay').addEventListener('click',e=>{ if(e.target.id==='overlay') $('overlay').classList.remove('open'); });
document.querySelectorAll('.opt[data-printer]').forEach(el=>el.onclick=()=>{ cfg.printer=el.dataset.printer; saveCfg(); render(); });
```
Replace with:
```js
function openSettings(){ $('overlay').classList.add('open'); $('in-site').value = cfg.site; }
$('set-close').onclick=()=>$('overlay').classList.remove('open');
$('overlay').addEventListener('click',e=>{ if(e.target.id==='overlay') $('overlay').classList.remove('open'); });
document.querySelectorAll('.opt[data-printer]').forEach(el=>el.onclick=()=>{ cfg.printer=el.dataset.printer; saveCfg(); render(); });
$('in-site').addEventListener('input',()=>{ cfg.site = $('in-site').value; saveCfg(); });
```

- [ ] **Step 4: Verify structure (PowerShell)**

Run:
```powershell
(Get-Content "index.html" | Select-String 'cfg\.site').Count
(Get-Content "index.html" | Select-String 'in-site').Count
```
Expected: `2` for `cfg.site` (the `openSettings` populate line + the new input listener's assignment — the `site:''` default doesn't match this pattern), `4` for `in-site` (the HTML `id="in-site"`, the populate line, and two occurrences on the listener line — `$('in-site').addEventListener` and `$('in-site').value` inside its own callback).

- [ ] **Step 5: Syntax check**

Run (PowerShell, from the project root):
```powershell
$l=Get-Content index.html; $s=($l|Select-String '^<script>'|Select-Object -First 1).LineNumber; $e=($l|Select-String '^</script>'|Select-Object -First 1).LineNumber; $l[$s..($e-2)]|Set-Content tmp.js -Encoding utf8; node --check tmp.js; Remove-Item tmp.js
```
Expected: no output (clean exit) — `node --check` only prints on error.

---

### Task 2: "Print pallet label" button next to "Scanned boxes"

**Files:**
- Modify: `index.html` (`.sec-title` CSS rule, ~line 274; Reconcile body markup, ~line 451)

**Interfaces:**
- Produces: `#rc-printlabel` button element. Task 4 wires its `onclick`.

- [ ] **Step 1: Add the `.sec-title-row` CSS wrapper**

Find (~line 274):
```css
  .sec-title{display:flex;align-items:center;gap:6px;font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:1.5px;margin:16px 0 8px;}
  .sec-title .ico{width:14px;height:14px;color:var(--faint);}
```
Replace with:
```css
  .sec-title{display:flex;align-items:center;gap:6px;font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:1.5px;margin:16px 0 8px;}
  .sec-title .ico{width:14px;height:14px;color:var(--faint);}
  .sec-title-row{display:flex;align-items:center;justify-content:space-between;gap:8px;margin:16px 0 8px;}
  .sec-title-row .sec-title{margin:0;}
```

- [ ] **Step 2: Wrap the "Scanned boxes" title with the new button**

Find (~line 451):
```html
      <div class="sec-title"><svg class="ico"><use href="#i-box"/></svg>Scanned boxes</div>
      <div class="chips" id="rc-chips"></div>
```
Replace with:
```html
      <div class="sec-title-row">
        <div class="sec-title"><svg class="ico"><use href="#i-box"/></svg>Scanned boxes</div>
        <button class="smallbtn" id="rc-printlabel"><svg class="ico"><use href="#i-printer"/></svg>Print pallet label</button>
      </div>
      <div class="chips" id="rc-chips"></div>
```

- [ ] **Step 3: Verify structure (PowerShell)**

Run:
```powershell
(Get-Content "index.html" | Select-String 'sec-title-row').Count
(Get-Content "index.html" | Select-String 'rc-printlabel').Count
```
Expected: `3` for `sec-title-row` (the two CSS selectors + the one HTML wrapper div), `1` for `rc-printlabel` (Task 4 will add a second match when it wires the click handler).

- [ ] **Step 4: Syntax check**

Run the same `node --check` one-liner as Task 1, Step 5. Expected: clean exit.

---

### Task 3: Print-CSS for the inventory label

**Files:**
- Modify: `index.html` (`printCss()`, ~lines 926-946)

**Interfaces:**
- Produces: CSS classes `.sec-title-print`, `.totals`, `.tbox` (`.tlabel`/`.tval`), `.foot-row`, `.verified`, consumed by Task 4's `printPalletLabel()`.

- [ ] **Step 1: Add the new rules after `.foot`**

Find (~line 944, inside the `printCss()` template literal):
```css
   .foot{margin-top:${small?'1mm':'2mm'};font-size:${small?'6pt':'9pt'};color:#000;}
  `;
```
Replace with:
```css
   .foot{margin-top:${small?'1mm':'2mm'};font-size:${small?'6pt':'9pt'};color:#000;}
   .sec-title-print{font-weight:800;font-size:${small?'7pt':'10pt'};letter-spacing:.04em;margin:0 0 ${small?'.8mm':'1.5mm'};text-transform:uppercase;}
   .totals{display:flex;gap:${small?'2mm':'4mm'};margin:${small?'1.5mm 0':'3mm 0'};}
   .tbox{flex:1;border:1pt solid #000;border-radius:1.5mm;padding:${small?'1mm 1.5mm':'2mm 3mm'};text-align:center;}
   .tbox .tlabel{font-size:${small?'6pt':'8pt'};letter-spacing:.04em;}
   .tbox .tval{font-weight:800;font-size:${small?'13pt':'20pt'};}
   .foot-row{display:flex;align-items:center;justify-content:space-between;gap:2mm;}
   .verified{font-weight:800;border:1pt solid #000;border-radius:1.5mm;padding:.5mm 1.5mm;background:#000;color:#fff;font-size:${small?'6pt':'9pt'};white-space:nowrap;}
  `;
```

- [ ] **Step 2: Verify structure (PowerShell)**

Run:
```powershell
(Get-Content "index.html" | Select-String 'sec-title-print|\.totals\{|\.tbox\{|\.tlabel\{|\.tval\{|foot-row|\.verified\{').Count
```
Expected: `7` (one match per new selector line added in Step 1).

- [ ] **Step 3: Syntax check**

Run the same `node --check` one-liner. Expected: clean exit.

---

### Task 4: `printerLabel()` / `printPalletLabel()` + wiring

**Files:**
- Modify: `index.html` (insert after `printReconcile()`, ~line 1009; wire click handler next to `$('rc-print').onclick`, ~line 1113)
- Test: temp Node script (created and deleted in Step 2, not committed to the repo)

**Interfaces:**
- Consumes: `recon.boxes` (`{seq:number, pieces:number}[]`), `cfg.site` (string), `cfg.printer` (`'qln420'|'a4'`) from Task 1; `pchip(seq,pc,pull)` (existing, line 947); `fmt(n)`, `esc(s)`, `flash(msg)`, `stamp()`, `doPrint(inner)` (all existing).
- Produces: `printerLabel()` → string (`'Zebra QLn420'` or `'Regular printer'`); `printPalletLabel()` → void (calls `doPrint`).

- [ ] **Step 1: Write a Node script asserting the expected HTML output**

This project has no DOM test runner, so verify the string-building logic in isolation the same way `solveExact` was verified — copy the function and its small dependencies into a throwaway script and assert against expected substrings.

Create `tmp-label-test.js`:
```js
const fmt = n => Number(n||0).toLocaleString('en-US');
const esc = s => String(s==null?'':s).replace(/[&<>"]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
const Bnum = seq => 'B'+String(seq).padStart(2,'0');
const stamp = () => 'STAMP';
function pchip(seq,pc,pull){ return '<div class="chip'+(pull?' pull':'')+'"><div class="bi">'+Bnum(seq)+'</div><div class="pc">'+fmt(pc)+'</div></div>'; }

let cfg = { printer:'qln420', site:'SEVENUM' };
function printerLabel(){ return cfg.printer==='qln420' ? 'Zebra QLn420' : 'Regular printer'; }
function buildLabel(boxesIn){
  const boxes = boxesIn.slice().sort((a,b)=>a.seq-b.seq);
  const pieces = boxes.reduce((a,b)=>a+b.pieces,0);
  return '<div class="sec-title-print">PIECES PER BOX</div>'
    + '<div class="chips">'+boxes.map(b=>pchip(b.seq,b.pieces,false)).join('')+'</div>'
    + '<div class="totals">'
    +   '<div class="tbox"><div class="tlabel">TOTAL BOXES</div><div class="tval">'+boxes.length+'</div></div>'
    +   '<div class="tbox"><div class="tlabel">TOTAL PIECES</div><div class="tval">'+fmt(pieces)+'</div></div>'
    + '</div>'
    + '<div class="foot foot-row">'
    +   '<span>'+(cfg.site? esc(cfg.site)+' · ' : '')+stamp()+' · '+printerLabel()+'</span>'
    +   '<span class="verified">&#10003; VERIFIED</span>'
    + '</div>';
}

function assert(cond,msg){ if(!cond){ console.error('FAIL: '+msg); process.exitCode=1; } else console.log('PASS: '+msg); }

// Case 1: normal pallet, site set, qln420 printer
let out = buildLabel([{seq:2,pieces:20},{seq:1,pieces:20},{seq:3,pieces:10}]);
assert(out.includes('<div class="tval">3</div>'), 'TOTAL BOXES = 3');
assert(out.includes('<div class="tval">50</div>'), 'TOTAL PIECES = 50');
assert(out.indexOf('B01')<out.indexOf('B02') && out.indexOf('B02')<out.indexOf('B03'), 'boxes sorted by seq ascending');
assert(!out.includes('chip pull'), 'no pull-highlighted boxes (this label never highlights a pick subset)');
assert(out.includes('SEVENUM · STAMP · Zebra QLn420'), 'footer shows site, date stamp, and printer name');
assert(out.includes('<span class="verified">&#10003; VERIFIED</span>'), 'VERIFIED badge always present');
assert(!/pick|target/i.test(out), 'label never references pick/target');
assert(!out.includes('<div class="h1">') && !/PLT|RECONCILE/.test(out), 'no title text on the label');

// Case 2: empty site falls back to just the date/printer
cfg = { printer:'a4', site:'' };
out = buildLabel([{seq:1,pieces:5}]);
assert(out.includes('">STAMP · Regular printer</span>'), 'empty site omits the leading "· " segment, a4 printer name used');

process.exit(process.exitCode||0);
```

- [ ] **Step 2: Run it, confirm it fails for the right reason (functions don't exist yet in `index.html`)**

This script is self-contained (it redefines the functions locally), so it will actually PASS on its own — that's expected, since Step 1 is verifying the *design* of the string-building logic before it's pasted into `index.html`. Run:
```powershell
node tmp-label-test.js
```
Expected: 9 `PASS` lines, exit code 0. If anything fails, fix the `buildLabel`/`printerLabel` logic above before proceeding — do not paste broken logic into `index.html`.

- [ ] **Step 3: Delete the temp script**

```powershell
Remove-Item tmp-label-test.js
```

- [ ] **Step 4: Paste the verified functions into `index.html`**

Find (~lines 1006-1011):
```js
  inner+='<div class="foot">'+stamp()+'</div>';
  doPrint(inner);
}

function renderHistory(){
```
Replace with:
```js
  inner+='<div class="foot">'+stamp()+'</div>';
  doPrint(inner);
}

function printerLabel(){ return cfg.printer==='qln420' ? 'Zebra QLn420' : 'Regular printer'; }

function printPalletLabel(){
  if(!recon.boxes.length){ flash('No boxes scanned'); return; }
  const boxes = recon.boxes.slice().sort((a,b)=>a.seq-b.seq);
  const pieces = boxes.reduce((a,b)=>a+b.pieces,0);
  let inner = '<div class="sec-title-print">PIECES PER BOX</div>'
    + '<div class="chips">'+boxes.map(b=>pchip(b.seq,b.pieces,false)).join('')+'</div>'
    + '<div class="totals">'
    +   '<div class="tbox"><div class="tlabel">TOTAL BOXES</div><div class="tval">'+boxes.length+'</div></div>'
    +   '<div class="tbox"><div class="tlabel">TOTAL PIECES</div><div class="tval">'+fmt(pieces)+'</div></div>'
    + '</div>'
    + '<div class="foot foot-row">'
    +   '<span>'+(cfg.site? esc(cfg.site)+' · ' : '')+stamp()+' · '+printerLabel()+'</span>'
    +   '<span class="verified">&#10003; VERIFIED</span>'
    + '</div>';
  doPrint(inner);
}

function renderHistory(){
```

- [ ] **Step 5: Wire the button click**

Find (~line 1113):
```js
$('rc-print').onclick=printReconcile;
```
Replace with:
```js
$('rc-print').onclick=printReconcile;
$('rc-printlabel').onclick=printPalletLabel;
```

- [ ] **Step 6: Verify structure (PowerShell)**

Run:
```powershell
(Get-Content "index.html" | Select-String 'printPalletLabel').Count
(Get-Content "index.html" | Select-String 'printerLabel').Count
(Get-Content "index.html" | Select-String 'rc-printlabel').Count
```
Expected: `2` for `printPalletLabel` (function definition + click wiring), `2` for `printerLabel` (function definition + the one call site inside `printPalletLabel`), `2` for `rc-printlabel` (the button's `id` from Task 2 + this task's click wiring).

- [ ] **Step 7: Syntax check**

Run the same `node --check` one-liner. Expected: clean exit.

---

### Task 5: Doc sync — CLAUDE.md

**Files:**
- Modify: `CLAUDE.md` (localStorage keys line)

- [ ] **Step 1: Add `site` to the documented `packcalc_cfg` fields**

Find:
```
`packcalc_cfg` (printer, mode, theme) · `packcalc_pallets` · `packcalc_active` · `packcalc_opt` (target) ·
```
Replace with:
```
`packcalc_cfg` (printer, mode, theme, site) · `packcalc_pallets` · `packcalc_active` · `packcalc_opt` (target) ·
```

- [ ] **Step 2: Verify**

Run:
```powershell
(Get-Content "CLAUDE.md" | Select-String 'packcalc_cfg').Count
```
Expected: `1`, and the matched line should read `...(printer, mode, theme, site)...`.

---

### Task 6: Final whole-file check + manual visual confirmation

**Files:** none (verification only)

- [ ] **Step 1: Full syntax check**

Run the Task 1 Step 5 `node --check` one-liner one more time against the final state of `index.html`. Expected: clean exit.

- [ ] **Step 2: Open the app**

```powershell
Start-Process index.html
```

- [ ] **Step 3: Manual walkthrough (ask the user to confirm visually — this environment cannot render the page or print output)**

Ask the user to:
1. Open Settings → enter a site name (e.g. "SEVENUM") → close → reopen Settings → confirm the value persisted.
2. Go to Reconcile, enter a pick target, scan 2-3 boxes.
3. Confirm a "Print pallet label" button appears next to "Scanned boxes", above the chip list.
4. Tap it before reaching the pick target — confirm it still prints (no gate on the pick).
5. Confirm the printed label shows: no title, "PIECES PER BOX", every scanned box as a chip (no black/pull-highlighted chips), TOTAL BOXES, TOTAL PIECES, and a footer with the site name, date/time, printer name, and a `✓ VERIFIED` badge — and that it does **not** mention the pick/target anywhere.
6. Delete a box, tap "Print pallet label" again — confirm the totals update.
7. Tap "Print slip" (the existing button) and confirm it behaves exactly as before (unaffected by this feature).

## Self-Review Notes

- **Spec coverage:** all 7 numbered decisions and both acceptance-criteria groups in the design spec map to Tasks 1-4 (data model → Task 1, button placement → Task 2, label content/CSS → Tasks 3-4, independence from `logRecon`/`printReconcile` → enforced by Task 4 only reading `recon.boxes`, never writing `recon` or calling `logRecon`). Doc sync is Task 5. No spec requirement is missing a task.
- **Placeholder scan:** none found — every step has literal code, not a description of code.
- **Type/name consistency:** `printerLabel()`, `printPalletLabel()`, `cfg.site`, `#rc-printlabel`, and all new CSS class names are spelled identically across Tasks 1-4 and the Task 4 Node test script (checked against each other directly, not just visually).
