# Pallet inventory label (Reconcile) — design spec

## Context

The team showed a label from a separate "receiving" app: a per-pallet manifest
listing every box's piece count in a grid (`B01`..`B60`), total boxes/pieces,
a site/date/printer footer, and a `VERIFIED` stamp. They want an equivalent
label printable from **Reconcile**, reprintable on demand whenever a box is
physically removed from the pallet (already-existing chip-delete action) or a
cycle-count quantity correction is made (already-existing chip-edit action) —
so the operator always has an up-to-date physical record of the pallet's
current box contents.

This is a pure **inventory snapshot**, independent of the pick/target concept
that the rest of Reconcile is built around. It is additive: the existing
"Print slip" (`printReconcile()`, gated on reaching the pick target, drives
the history log) is untouched.

Decisions confirmed during brainstorming:

1. **Separate from Print slip** — a second, independent print action. Doesn't
   touch `printReconcile()`, `logRecon()`, or the history log.
2. **Site name is global, not per-box** — `cfg.site` (new field in
   `packcalc_cfg`), set once in Settings. This does **not** reintroduce
   rack/location to Reconcile (that removal stands); it's the facility name,
   same value for every print everywhere.
3. **Printer name is reused, not new** — footer shows whichever printer is
   already selected in Settings (`Zebra QLn420` / `Regular printer`). No new
   printer option added.
4. **No pallet name/number field** — Reconcile has one flat box list per
   session, not multiple named pallets. No title at all on this label (no
   "PLT 1", no "RECONCILE — ..." header).
5. **No pick/target reference anywhere on this label** — purely
   `TOTAL BOXES` / `TOTAL PIECES`, matching the reference screenshot.
6. **VERIFIED is unconditional** — it certifies the count was produced by
   scanning in the app, not a judgement on whether it matches anything.
7. **Button**: a small secondary button next to the "Scanned boxes" section
   title (same pattern as the existing per-pallet `data-printpallet` button
   in Optimize's board view), not a third button in the bottom `actbar`.
   Enabled once `recon.boxes.length>0` — independent of the pick target.

## Data model

`cfg` (existing `packcalc_cfg` key) gains one field:

```js
const cfg = Object.assign({ printer:'qln420', mode:'home', theme:'light', site:'' }, store.get(LS.cfg,{}));
```

No changes to `recon`, `packcalc_reconcile`, or `packcalc_history`. This
feature reads `recon.boxes` but never writes to `recon` and never calls
`logRecon()`.

## Settings UI

New row in the Settings sheet (`#overlay .sheet`), its own section below the
existing Printer section:

```html
<div class="sec-title"><svg class="ico"><use href="#i-pin"/></svg>Site</div>
<div class="field"><label>Site / warehouse name</label>
  <div class="inwrap"><input id="in-site" placeholder="e.g. SEVENUM" autocomplete="off"></div>
</div>
```

Wired the same way other plain text settings would be: `input` event saves
`cfg.site = $('in-site').value; saveCfg();`. Populated from `cfg.site` when
the sheet opens (`openSettings()`).

## Reconcile UI changes

`#screen-recon`'s `.body`, the "Scanned boxes" section title gets a button
next to it:

```html
<div class="sec-title-row">
  <div class="sec-title"><svg class="ico"><use href="#i-box"/></svg>Scanned boxes</div>
  <button class="smallbtn" id="rc-printlabel"><svg class="ico"><use href="#i-printer"/></svg>Print pallet label</button>
</div>
```

`.sec-title-row` is a new minimal flex wrapper (`display:flex;align-items:center;justify-content:space-between`)
so the title and button sit on one line. No other Reconcile markup changes.

## Print function

```js
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
```

`&#10003;` is the plain-text check mark glyph (✓, U+2713) — not an emoji,
consistent with the rest of the print stylesheet being plain B/W text.

Wired: `$('rc-printlabel').onclick = printPalletLabel;`

## Print CSS additions (`printCss()`)

```css
.sec-title-print{font-weight:800;font-size:${small?'7pt':'10pt'};letter-spacing:.04em;
  margin:0 0 ${small?'.8mm':'1.5mm'};text-transform:uppercase;}
.totals{display:flex;gap:${small?'2mm':'4mm'};margin:${small?'1.5mm 0':'3mm 0'};}
.tbox{flex:1;border:1pt solid #000;border-radius:1.5mm;padding:${small?'1mm 1.5mm':'2mm 3mm'};text-align:center;}
.tbox .tlabel{font-size:${small?'6pt':'8pt'};letter-spacing:.04em;}
.tbox .tval{font-weight:800;font-size:${small?'13pt':'20pt'};}
.foot-row{display:flex;align-items:center;justify-content:space-between;gap:2mm;}
.verified{font-weight:800;border:1pt solid #000;border-radius:1.5mm;padding:.5mm 1.5mm;background:#000;color:#fff;
  font-size:${small?'6pt':'9pt'};white-space:nowrap;}
```

Reuses the existing `.chips`/`.chip` (via `pchip()`) and `.foot` rules
unchanged — `.foot-row` only adds the flex layout for the site/date/printer
text sitting next to the `VERIFIED` badge on one line.

## Out of scope

- No change to `printReconcile()`, `logRecon()`, or the history log — this is
  a fully independent print action with no `recon` state mutation.
- No automatic printing on box delete/edit — always a manual button press.
- No new printer option (`Zebra ZT` etc.) — reuses the existing two.
- No pallet name/number field, no title text on the label.
- No pick/target value anywhere on this label.
- No reintroduction of per-box rack/location — `cfg.site` is one global value
  for the whole app, set once in Settings.

## Acceptance criteria

- "Print pallet label" button appears next to "Scanned boxes", enabled once
  at least one box is scanned (works before, at, and after reaching the pick
  target — no gate on `recon.pick`).
- Tapping it with zero boxes scanned flashes "No boxes scanned" and prints
  nothing.
- The printed label shows: no title, "PIECES PER BOX" caption, a chip grid
  of every scanned box (`Bnn` + piece count, no pull highlighting), a
  TOTAL BOXES / TOTAL PIECES pair of boxes, and a footer with
  `site · date,time · printer name` plus an unconditional `VERIFIED` badge.
- Settings sheet has a new "Site / warehouse name" text field that persists
  to `cfg.site` (`packcalc_cfg`) and is reflected on the next print.
- Printing this label never touches `recon.logged`, never appends to
  `packcalc_history`, and never affects `printReconcile()`'s behavior.
- `node --check` still passes on the inline script.
- No emoji anywhere; the check mark on the label is the plain-text glyph `✓`.
