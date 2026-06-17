# Reconcile history log Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a persisted history log of completed Reconcile sessions (EXACT/SHORT verdict, timestamp, full box detail) viewable and reprintable from a new "History" overlay, and fix a related bug where `printReconcile()` could print a premature verdict before scanning finished.

**Architecture:** A new `packcalc_history` localStorage array (capped at 200, newest first) is written by a single `logRecon()` function, called from the two existing exit points of a reconciliation (`printReconcile()` and the "New" button), guarded by a `recon.logged` flag so the same finished pallet is never logged twice — but that flag resets on any further mutation so genuinely different states each get their own entry. A new bottom-sheet overlay (same pattern as the existing Settings sheet) lists entries and can reprint any of them.

**Tech Stack:** Vanilla HTML/CSS/JS, no build, no deps, no git repo.

## Global Constraints

- Everything lives in `index.html` (CLAUDE.md) — no new files.
- No build, no deps, no git (CLAUDE.md) — every "commit" step below is a grep/`node --check` verification instead; nothing is committed.
- Icons: inline SVG sprite only, never emoji (CLAUDE.md).
- UI stays in English (CLAUDE.md).
- Optimize module and the print stylesheet (`printCss()`) are out of scope — untouched.
- Reconcile has no rack/location concept (deliberate, earlier change) — history entries do not include a rack field.
- After all tasks, the inline `<script>` block must still pass `node --check`.

---

### Task 1: Storage layer — `LS.history`, `history` state, `recon.logged`, `logRecon()`

**Files:**
- Modify: `index.html` (`LS` object line 529; `cfg`/`recon` declarations lines 534–535; new function placed right before `printReconcile()`, currently line 928)

**Interfaces:**
- Produces: `LS.history` (string key), `history` (array, module-scoped, newest-first), `logRecon()` (no args, no return) — consumed by Task 2.
- Consumes: `store.get`/`store.set` (already defined), `recon`, `reconBoxes()`, `solveExact()` (all already defined).

- [ ] **Step 1: Add the `history` localStorage key**

Old code:
```js
const LS = { cfg:'packcalc_cfg', pallets:'packcalc_pallets', active:'packcalc_active', opt:'packcalc_opt', recon:'packcalc_reconcile' };
```

New code:
```js
const LS = { cfg:'packcalc_cfg', pallets:'packcalc_pallets', active:'packcalc_active', opt:'packcalc_opt', recon:'packcalc_reconcile', history:'packcalc_history' };
```

- [ ] **Step 2: Add the `history` array and `recon.logged` default**

Old code:
```js
const cfg     = Object.assign({ printer:'qln420', mode:'home', theme:'light' }, store.get(LS.cfg,{}));
let   recon   = Object.assign({ pick:null, boxes:[] }, store.get(LS.recon,{}));
let   pallets = store.get(LS.pallets, []);
```

New code:
```js
const cfg     = Object.assign({ printer:'qln420', mode:'home', theme:'light' }, store.get(LS.cfg,{}));
let   recon   = Object.assign({ pick:null, boxes:[], logged:false }, store.get(LS.recon,{}));
let   history = store.get(LS.history, []);
let   pallets = store.get(LS.pallets, []);
```

- [ ] **Step 3: Add `logRecon()` right before `printReconcile()`**

Old code:
```js
const stamp=()=> new Date().toLocaleString('en-GB');

function printReconcile(){
```

New code:
```js
const stamp=()=> new Date().toLocaleString('en-GB');

function logRecon(){
  if(recon.logged || !recon.pick || !recon.boxes.length) return;
  const pieces=recon.boxes.reduce((a,b)=>a+b.pieces,0);
  if(pieces<recon.pick) return;
  const res=solveExact(reconBoxes(),recon.pick);
  history.unshift({
    ts:Date.now(), pick:recon.pick,
    boxes:recon.boxes.map(b=>({seq:b.seq,pieces:b.pieces})),
    pieces, boxCount:recon.boxes.length,
    result: res.exact?'EXACT':'SHORT',
    shortfall: res.exact?0:(recon.pick-res.closest)
  });
  if(history.length>200) history.length=200;
  store.set(LS.history,history);
  recon.logged=true; saveRecon();
}

function printReconcile(){
```

- [ ] **Step 4: Verify**

Run:
```
grep -n -- "LS.history\|function logRecon\|logged:false" index.html
```
Expected: at least 4 matches (the `LS` entry, the `store.get(LS.history` line, the `recon` default, and the function definition).

---

### Task 2: Wire `logRecon()` into the two exit points + fix the premature-print bug

**Files:**
- Modify: `index.html` (`printReconcile()` lines 928–930; the `rc-new` click handler, currently lines 1015–1020)

**Interfaces:**
- Consumes: `logRecon()` from Task 1.

- [ ] **Step 1: Add the finished-scanning gate and the `logRecon()` call to `printReconcile()`**

Old code:
```js
function printReconcile(){
  if(!recon.pick){ flash('Enter a pick first'); return; }
  if(!recon.boxes.length){ flash('No boxes scanned'); return; }
  const boxes=reconBoxes(); const res=solveExact(boxes,recon.pick);
```

New code:
```js
function printReconcile(){
  if(!recon.pick){ flash('Enter a pick first'); return; }
  if(!recon.boxes.length){ flash('No boxes scanned'); return; }
  const scannedPieces=recon.boxes.reduce((a,b)=>a+b.pieces,0);
  if(scannedPieces<recon.pick){ flash('Finish scanning before printing'); return; }
  logRecon();
  const boxes=reconBoxes(); const res=solveExact(boxes,recon.pick);
```

- [ ] **Step 2: Log on "New" before resetting `recon`**

Old code:
```js
$('rc-new').onclick=()=>{
  if(recon.boxes.length && !confirm('Start a new reconciliation? This clears the current pick and scanned boxes.')) return;
  recon={ pick:null, boxes:[] }; saveRecon();
  $('in-pick').value='';
  render(); focusEl('in-pick');
};
```

New code:
```js
$('rc-new').onclick=()=>{
  if(recon.boxes.length && !confirm('Start a new reconciliation? This clears the current pick and scanned boxes.')) return;
  logRecon();
  recon={ pick:null, boxes:[], logged:false }; saveRecon();
  $('in-pick').value='';
  render(); focusEl('in-pick');
};
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "Finish scanning before printing\|logRecon();" index.html
```
Expected: 3 matches (the flash message, the call inside `printReconcile()`, the call inside the `rc-new` handler).

---

### Task 3: Invalidate `recon.logged` on every mutation after a log

**Files:**
- Modify: `index.html` (`handleReconScan` ~line 1006; the `#rc-chips` delete handler ~line 1057–1059; `saveEditBox()` recon branch ~lines 775–777; `commitPick()` ~line 1003)

**Interfaces:**
- Consumes: `recon.logged` field from Task 1.

- [ ] **Step 1: Reset on new scan**

Old code:
```js
function handleReconScan(raw){
  if(isDup(raw)) return;
  const pc=cleanToPieces(raw);
  if(pc==null){ flash('Invalid scan'); return; }
  recon.boxes.push({ seq:recon.boxes.length+1, raw:String(raw), pieces:pc, ts:Date.now() });
  saveRecon(); render();
}
```

New code:
```js
function handleReconScan(raw){
  if(isDup(raw)) return;
  const pc=cleanToPieces(raw);
  if(pc==null){ flash('Invalid scan'); return; }
  recon.boxes.push({ seq:recon.boxes.length+1, raw:String(raw), pieces:pc, ts:Date.now() });
  recon.logged=false;
  saveRecon(); render();
}
```

- [ ] **Step 2: Reset on chip delete (Reconcile only)**

Old code:
```js
$('rc-chips').addEventListener('click',e=>{
  const del=e.target.closest('.chip-del');
  if(del){ const c=del.closest('.chip'); recon.boxes=recon.boxes.filter(b=>b.seq!==+c.dataset.seq); saveRecon(); render(); return; }
```

New code:
```js
$('rc-chips').addEventListener('click',e=>{
  const del=e.target.closest('.chip-del');
  if(del){ const c=del.closest('.chip'); recon.boxes=recon.boxes.filter(b=>b.seq!==+c.dataset.seq); recon.logged=false; saveRecon(); render(); return; }
```

- [ ] **Step 3: Reset on box-quantity edit (Reconcile only)**

Old code:
```js
  if(mode==='recon'){
    const b=recon.boxes.find(x=>x.seq===seq); if(b) b.pieces=v;
    saveRecon();
  } else {
```

New code:
```js
  if(mode==='recon'){
    const b=recon.boxes.find(x=>x.seq===seq); if(b) b.pieces=v;
    recon.logged=false;
    saveRecon();
  } else {
```

- [ ] **Step 4: Reset on pick change**

Old code:
```js
function commitPick(){ const v=cleanToPieces($('in-pick').value); recon.pick=v; $('in-pick').value=v!=null?v:''; saveRecon(); render(); }
```

New code:
```js
function commitPick(){ const v=cleanToPieces($('in-pick').value); if(v!==recon.pick) recon.logged=false; recon.pick=v; $('in-pick').value=v!=null?v:''; saveRecon(); render(); }
```

- [ ] **Step 5: Verify**

Run:
```
grep -n -- "recon.logged=false" index.html
```
Expected: 4 matches.

---

### Task 4: New sprite icon, CSS, History overlay markup, header button

**Files:**
- Modify: `index.html` (sprite `<svg id="sprite">` ~line 382; CSS after `.chip-del:active` rules ~line 245; markup after the Settings overlay ~line 506; Reconcile header ~line 417)

**Interfaces:**
- Produces: `#i-history` sprite symbol, `.hist-row`/`.hr` CSS, `#history-overlay`/`#history-list`/`#history-close`/`#history-clear` DOM nodes, `#rc-history-btn` button — all consumed by Task 5's JS.

- [ ] **Step 1: Add the sprite symbol**

Old code:
```html
  <symbol id="i-moon" viewBox="0 0 24 24"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79Z"/></symbol>
```

New code:
```html
  <symbol id="i-history" viewBox="0 0 24 24"><circle cx="12" cy="12" r="9"/><path d="M12 7v5l3 3"/></symbol>
  <symbol id="i-moon" viewBox="0 0 24 24"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79Z"/></symbol>
```

- [ ] **Step 2: Add the history-row CSS**

Old code:
```css
  .chip-del:active{color:var(--danger);background:rgba(var(--danger-rgb),.14);}
```

New code:
```css
  .chip-del:active{color:var(--danger);background:rgba(var(--danger-rgb),.14);}
  .hist-row{background:var(--panel);border:1px solid var(--border);border-radius:11px;margin-bottom:8px;overflow:hidden;}
  .hist-row .hh{display:flex;align-items:center;gap:10px;padding:10px 11px;cursor:pointer;}
  .hist-row .hh .ht{flex:1;min-width:0;}
  .hist-row .hh .hd{font-size:12px;color:var(--muted);}
  .hist-row .hh .hp{font-family:var(--mono);font-weight:700;font-size:14px;margin-top:1px;}
  .hist-row .hh .hr{font-weight:800;font-size:12px;padding:3px 8px;border-radius:999px;flex:none;}
  .hist-row .hh .hr.exact{background:var(--okbg);color:#14532d;}
  .hist-row .hh .hr.short{background:var(--dangerbg);color:#7f1d1d;}
  [data-theme="dark"] .hist-row .hh .hr.exact{color:#86efc6;}
  [data-theme="dark"] .hist-row .hh .hr.short{color:#fca5a5;}
  .hist-row .hb{display:none;padding:0 11px 11px;border-top:1px solid var(--border);}
  .hist-row.open .hb{display:block;padding-top:9px;}
  #history-list{max-height:60vh;overflow-y:auto;}
```

- [ ] **Step 3: Add the History overlay markup, right after the Settings overlay's closing `</div>`**

Old code:
```html
    <div class="btns" style="margin-top:12px"><button class="btn sec" id="set-close">Close</button></div>
  </div>
</div>

<div class="flash" id="flash"></div>
```

New code:
```html
    <div class="btns" style="margin-top:12px"><button class="btn sec" id="set-close">Close</button></div>
  </div>
</div>

<!-- History sheet (Reconcile only) -->
<div class="overlay" id="history-overlay">
  <div class="sheet">
    <div class="grab"></div>
    <h3>Reconciliation history</h3>
    <div id="history-list"></div>
    <div class="btns" style="margin-top:12px">
      <button class="btn danger" id="history-clear"><svg class="ico"><use href="#i-trash"/></svg>Clear history</button>
      <button class="btn sec" id="history-close">Close</button>
    </div>
  </div>
</div>

<div class="flash" id="flash"></div>
```

- [ ] **Step 4: Add the header button to the Reconcile screen**

Old code:
```html
      <button class="iconbtn" data-theme-toggle aria-label="Toggle dark mode"><svg class="ico"><use href="#i-moon"/></svg></button>
      <button class="iconbtn" id="rc-guide-btn" aria-label="Quick guide" aria-expanded="false" aria-controls="rc-guide">?</button>
```

New code:
```html
      <button class="iconbtn" data-theme-toggle aria-label="Toggle dark mode"><svg class="ico"><use href="#i-moon"/></svg></button>
      <button class="iconbtn" id="rc-history-btn" aria-label="Reconciliation history"><svg class="ico"><use href="#i-history"/></svg></button>
      <button class="iconbtn" id="rc-guide-btn" aria-label="Quick guide" aria-expanded="false" aria-controls="rc-guide">?</button>
```

- [ ] **Step 5: Verify**

Run:
```
grep -n -- "i-history\|hist-row\|history-overlay\|rc-history-btn" index.html
```
Expected: 6 matches (sprite symbol, CSS selector mentions collapse to one grep line per distinct line — expect: 1 sprite line, 1 `#i-history` use in the header button, 1 `.hist-row` CSS line [the others like `.hist-row .hh` etc. also match `hist-row` — see note below], `#history-overlay` markup line, `#rc-history-btn` markup line). Because `hist-row` appears in many CSS lines, don't worry about an exact count here — just confirm the four distinct pieces (sprite symbol, header button, overlay markup, at least one `.hist-row` CSS rule) are all present in the output.

---

### Task 5: JS — render, open/close, reprint, clear, wiring

**Files:**
- Modify: `index.html` (new functions placed right after `printReconcile()`, currently ending around line 945; new event wiring placed in the "Reconcile inputs" init block, after the existing `$('rc-new').onclick=...` block)

**Interfaces:**
- Consumes: `history` (Task 1), `chip()`, `pchip()`, `fmt()`, `Bnum()`, `ic()`, `solveExact()`, `doPrint()` (all pre-existing), `$('history-list')`/`$('history-overlay')`/`$('history-clear')`/`$('history-close')`/`$('rc-history-btn')` (Task 4).
- Produces: `renderHistory()`, `openHistory()`, `printHistoryEntry(i)` — no other task depends on these internals.

- [ ] **Step 1: Add `renderHistory()` and `printHistoryEntry()` right after `printReconcile()`**

Find the end of `printReconcile()` (it currently ends with `doPrint(inner);` followed by a closing `}` and then `function printMix(sol){`). Insert the new functions between that closing `}` and `function printMix(sol){`:

Old code:
```js
  inner+='<div class="foot">'+stamp()+'</div>';
  doPrint(inner);
}

function printMix(sol){
```

New code:
```js
  inner+='<div class="foot">'+stamp()+'</div>';
  doPrint(inner);
}

function renderHistory(){
  const c=$('history-list');
  if(!history.length){ c.innerHTML='<div class="empty">'+ic('history')+'No reconciliations logged yet.</div>'; return; }
  c.innerHTML = history.map((h,i)=>{
    const cls = h.result==='EXACT' ? 'exact' : 'short';
    const label = h.result==='EXACT' ? 'EXACT' : ('SHORT −'+fmt(h.shortfall));
    const boxesHtml = h.boxes.slice().sort((a,b)=>a.seq-b.seq).map(b=>chip(b.seq,b.pieces,false,false)).join('');
    return '<div class="hist-row" data-idx="'+i+'">'
      + '<div class="hh"><div class="ht"><div class="hd">'+new Date(h.ts).toLocaleString('en-GB')+'</div>'
      +   '<div class="hp">Pick '+fmt(h.pick)+' · scanned '+fmt(h.pieces)+' ('+h.boxCount+' box'+(h.boxCount>1?'es':'')+')</div></div>'
      +   '<span class="hr '+cls+'">'+label+'</span></div>'
      + '<div class="hb"><div class="chips">'+boxesHtml+'</div>'
      +   '<button class="smallbtn" style="margin-top:9px" data-hist-print="'+i+'">'+ic('printer')+'Print</button></div>'
      + '</div>';
  }).join('');
}
function printHistoryEntry(i){
  const h=history[i]; if(!h) return;
  const boxes=h.boxes.map(b=>({_id:'r'+b.seq,pieces:b.pieces,seq:b.seq,locKey:'r',palletName:'Pallet'}));
  const res=solveExact(boxes,h.pick);
  const pull=new Set(res.picks.map(b=>b._id));
  let inner='<div class="h1">RECONCILE — PICK SLIP</div>'
    +'<div class="meta"><b>Pick:</b> '+fmt(h.pick)+'<br>'
    +'<b>Scanned:</b> '+fmt(h.pieces)+' ('+h.boxCount+' boxes)</div>';
  if(res.exact){
    inner+='<div class="res">PICK OK · EXACT 100% — pull highlighted</div>'
      +'<div class="chips">'+h.boxes.slice().sort((a,b)=>a.seq-b.seq).map(b=>pchip(b.seq,b.pieces,pull.has('r'+b.seq))).join('')+'</div>';
  } else {
    const pct=Math.round(res.closest/h.pick*1000)/10;
    inner+='<div class="res bad">NOT POSSIBLE IN WMS · closest '+fmt(res.closest)+' ('+pct+'%)</div>'
      +'<div class="chips">'+h.boxes.slice().sort((a,b)=>a.seq-b.seq).map(b=>pchip(b.seq,b.pieces,pull.has('r'+b.seq))).join('')+'</div>';
  }
  inner+='<div class="foot">'+new Date(h.ts).toLocaleString('en-GB')+' (reprint)</div>';
  doPrint(inner);
}

function printMix(sol){
```

- [ ] **Step 2: Wire the overlay open/close/clear/print and the header button, right after the existing `rc-new` block**

Old code:
```js
$('rc-print').onclick=printReconcile;
$('rc-new').onclick=()=>{
  if(recon.boxes.length && !confirm('Start a new reconciliation? This clears the current pick and scanned boxes.')) return;
  logRecon();
  recon={ pick:null, boxes:[], logged:false }; saveRecon();
  $('in-pick').value='';
  render(); focusEl('in-pick');
};
```

New code:
```js
$('rc-print').onclick=printReconcile;
$('rc-new').onclick=()=>{
  if(recon.boxes.length && !confirm('Start a new reconciliation? This clears the current pick and scanned boxes.')) return;
  logRecon();
  recon={ pick:null, boxes:[], logged:false }; saveRecon();
  $('in-pick').value='';
  render(); focusEl('in-pick');
};

/* History overlay */
function openHistory(){ renderHistory(); $('history-overlay').classList.add('open'); }
$('rc-history-btn').onclick=openHistory;
$('history-close').onclick=()=>$('history-overlay').classList.remove('open');
$('history-overlay').addEventListener('click',e=>{ if(e.target.id==='history-overlay') $('history-overlay').classList.remove('open'); });
$('history-clear').onclick=()=>{
  if(!history.length) return;
  if(!confirm('Clear all reconciliation history? This cannot be undone.')) return;
  history=[]; store.set(LS.history,history); renderHistory();
};
$('history-list').addEventListener('click',e=>{
  const printBtn=e.target.closest('[data-hist-print]');
  if(printBtn){ printHistoryEntry(+printBtn.dataset.histPrint); return; }
  const head=e.target.closest('.hh');
  if(head){ head.closest('.hist-row').classList.toggle('open'); }
});
```

- [ ] **Step 3: Verify**

Run:
```
grep -n -- "function renderHistory\|function printHistoryEntry\|function openHistory\|history-list" index.html
```
Expected: at least 5 matches.

---

### Task 6: Documentation + final whole-file verification

**Files:**
- Modify: `CLAUDE.md` (localStorage keys line)
- Verify only: `index.html`

- [ ] **Step 1: Update `CLAUDE.md`'s localStorage keys line**

Old code:
```
`packcalc_cfg` (printer, mode, theme) · `packcalc_pallets` · `packcalc_active` · `packcalc_opt` (target) ·
`packcalc_reconcile` (pick, boxes).
```

New code:
```
`packcalc_cfg` (printer, mode, theme) · `packcalc_pallets` · `packcalc_active` · `packcalc_opt` (target) ·
`packcalc_reconcile` (pick, boxes, logged) · `packcalc_history` (array of completed Reconcile results, capped
at 200 entries, newest first — see `docs/superpowers/specs/2026-06-18-reconcile-history-log-design.md`).
```

- [ ] **Step 2: Run the project's syntax check**

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

- [ ] **Step 3: Logic check — simulate the dedup/invalidation rules in a standalone Node script**

This project's established test pattern (CLAUDE.md) is to copy pure logic into a temp Node script and assert behavior, since there's no test framework. Write this script to verify `logRecon`'s gating without needing a DOM:

```bash
cat > /tmp/test_history.js << 'EOF'
function makeHistoryHarness(){
  let history=[];
  let recon={pick:null,boxes:[],logged:false};
  // Real bitmask brute-force (same technique CLAUDE.md describes for verifying the
  // actual engine) — correct for the small box counts these tests use, so shortfall
  // assertions below reflect a real achievable-subset answer, not an approximation.
  function solveExactStub(boxes,pick){
    const n=boxes.length; let best=0;
    for(let mask=0; mask<(1<<n); mask++){
      let sum=0;
      for(let i=0;i<n;i++) if(mask&(1<<i)) sum+=boxes[i].pieces;
      if(sum<=pick && sum>best) best=sum;
    }
    return { exact: best===pick, closest: best };
  }
  function logRecon(){
    if(recon.logged || !recon.pick || !recon.boxes.length) return;
    const pieces=recon.boxes.reduce((a,b)=>a+b.pieces,0);
    if(pieces<recon.pick) return;
    const res=solveExactStub(recon.boxes,recon.pick);
    history.unshift({ts:Date.now(),pick:recon.pick,boxes:recon.boxes.slice(),pieces,boxCount:recon.boxes.length,
      result:res.exact?'EXACT':'SHORT',shortfall:res.exact?0:(recon.pick-res.closest)});
    if(history.length>200) history.length=200;
    recon.logged=true;
  }
  return {history:()=>history, recon, logRecon};
}

let pass=0, fail=0;
function check(name, cond){ if(cond) pass++; else { fail++; console.log('FAIL', name); } }

// 1: mid-scan New must not log
{ const h=makeHistoryHarness(); h.recon.pick=500; h.recon.boxes=[{seq:1,pieces:200}];
  h.logRecon(); check('mid-scan does not log', h.history().length===0); }

// 2: print then New on same state logs exactly once
{ const h=makeHistoryHarness(); h.recon.pick=500; h.recon.boxes=[{seq:1,pieces:500}];
  h.logRecon(); h.logRecon();
  check('print+New same state logs once', h.history().length===1);
  check('result is EXACT', h.history()[0].result==='EXACT'); }

// 3: short pick logs with correct shortfall (boxes 300+250=550 total, but no subset
// hits 500 exactly; best achievable <=500 is the single 300 box, so shortfall=200)
{ const h=makeHistoryHarness(); h.recon.pick=500; h.recon.boxes=[{seq:1,pieces:300},{seq:2,pieces:250}];
  h.logRecon();
  check('SHORT result', h.history()[0].result==='SHORT');
  check('shortfall computed', h.history()[0].shortfall===(500-300)); }

// 4: mutation after log allows a second entry
{ const h=makeHistoryHarness(); h.recon.pick=500; h.recon.boxes=[{seq:1,pieces:500}];
  h.logRecon();
  h.recon.boxes.push({seq:2,pieces:10}); h.recon.logged=false; // simulates handleReconScan's reset
  h.logRecon();
  check('mutation + relog produces 2 entries', h.history().length===2); }

// 5: cap at 200
{ const h=makeHistoryHarness();
  for(let i=0;i<205;i++){ h.recon.pick=10; h.recon.boxes=[{seq:1,pieces:10}]; h.recon.logged=false; h.logRecon(); }
  check('capped at 200', h.history().length===200); }

console.log(pass+' passed, '+fail+' failed');
process.exit(fail?1:0);
EOF
node /tmp/test_history.js
rm -f /tmp/test_history.js
```
Expected: `5 passed, 0 failed` (the harness re-implements `logRecon`'s exact gating logic from Task 1 step 3 — if this fails, the real implementation likely has a typo in the same conditions).

- [ ] **Step 4: Whole-file grep sanity check**

Run:
```
grep -c -- "recon.logged" index.html
```
Expected: 6 matching lines (the 4 `recon.logged=false` reset call sites from Task 3, plus the
`if(recon.logged || ...)` check and the `recon.logged=true` set, both inside `logRecon()`). Note this
does NOT count the two `logged:false` object-literal fields — at startup and in the `rc-new` reset —
since those read `logged:false` inside a `{...}` literal, not as a `recon.logged` property access.

- [ ] **Step 5: Manual visual check**

Run:
```
Start-Process index.html
```
In the Reconcile screen: enter a pick, scan boxes until it reads EXACT, tap "Print slip" (confirm it prints), then tap the new clock-icon "History" button — confirm the entry appears, expand it to see the box list, tap "Print" on that entry and confirm it reprints with the original timestamp plus "(reprint)". Then start a new pick that falls short, scan boxes, try printing **before** reaching the target (confirm it's blocked with "Finish scanning before printing"), finish scanning, print, and confirm the SHORT entry with the correct shortfall appears in history. Tap "Clear history" and confirm the list empties after the confirmation prompt.
