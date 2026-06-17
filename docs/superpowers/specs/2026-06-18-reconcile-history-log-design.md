# Reconcile history log — design spec

## Context

The Reconcile module is a team leader's physical stock check before approving/
adjusting a pick in the WMS (see `project_wms_pick_approval_workflow.md`).
Today, pressing "New" discards all evidence of a completed reconciliation —
there's no record of when it happened, whether it was an exact match, or by
how much it fell short. This spec adds a history log scoped **only** to
Reconcile (Optimize is untouched).

Decisions confirmed during brainstorming:

1. **Log trigger**: on "Print slip" OR on "New" — whichever happens first for
   a given reconciliation. Never both, for the same boxes/pick combination.
2. **Fields stored**: timestamp, pick, full box list (seq + pieces), total
   pieces, box count, result (`EXACT`/`SHORT`), shortfall.
3. **View**: a new "History" button in the Reconcile header opens an overlay
   (same bottom-sheet pattern as Settings) listing entries newest-first, each
   expandable to show the box-by-box detail, each with a "Print" button that
   reprints that entry's slip using its original timestamp.
4. **Retention**: keep the most recent 200 entries automatically (oldest
   dropped), plus a manual "Clear history" button.

## A related bug this spec also fixes

`printReconcile()` currently has **no gate** on whether scanning is actually
finished — it calls `solveExact` and prints "EXACT" or "NOT POSSIBLE" even if
total scanned pieces are still below the pick target (mid-scan). `renderRecon()`
already has this gate (added in an earlier change) so the on-screen banner
never shows a premature verdict — print just never inherited it. Since the
history log's correctness depends on only recording *final* results, this spec
adds the same `pieces>=pick` gate to `printReconcile()`: if scanning isn't
finished, printing is blocked with a flash message instead of producing (and
now also logging) a misleading "SHORT" result.

## Data model

New localStorage key `packcalc_history` (`LS.history`), a plain array, newest
entry first, capped at 200:

```js
{
  ts: 1750000000000,        // Date.now() when logged
  pick: 500,
  boxes: [{seq:1,pieces:120}, {seq:2,pieces:380}, ...],
  pieces: 500,               // sum of boxes[].pieces
  boxCount: 2,
  result: 'EXACT',           // or 'SHORT'
  shortfall: 0                // pick - closest achievable, 0 when EXACT
}
```

`recon` (the existing reconcile state object, `packcalc_reconcile`) gains one
new field: `logged: false`. This is the in-session dedup flag — it's persisted
so a page reload between "print" and "New" doesn't cause a double-log.

**`recon.logged` resets to `false`** (so the *next* print/New produces a fresh
entry instead of being silently skipped) whenever the underlying data changes
after a log was already recorded:
- a box is scanned (`handleReconScan`)
- a box is deleted (chip delete handler)
- a box's quantity is edited (`saveEditBox`)
- the pick value is changed (`commitPick`)

This means: print → keep scanning more boxes → print again (or "New") produces
**two** distinct entries (correct — the pallet state genuinely changed), while
print → immediately press "New" with nothing changed produces only **one**.

## Logging function

```js
function logRecon(){
  if(recon.logged || !recon.pick || !recon.boxes.length) return;
  const pieces=recon.boxes.reduce((a,b)=>a+b.pieces,0);
  if(pieces<recon.pick) return; // not a final result yet — same gate as renderRecon()
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
```

Called from `printReconcile()` (right after its existing pick/boxes guards and
the new finished-scanning gate) and from the `rc-new` click handler (right
before `recon` is reset), so it's a no-op the second time either trigger fires
for an already-logged, unchanged reconciliation.

## UI

**Header button** (Reconcile only): a new `.iconbtn` with the clock-style
`#i-history` sprite icon (new symbol, stroke-based like the rest:
`<circle cx="12" cy="12" r="9"/><path d="M12 7v5l3 3"/>`), placed after the
theme-toggle button.

**History overlay** (`#history-overlay` / `.sheet`, same visual pattern as the
existing Settings overlay): title "Reconciliation history", then a scrollable
list (`#history-list`, same `max-height:30vh;overflow-y:auto` treatment as the
chip lists) of rows. Each row shows: relative date/time, pick, scanned pieces,
and a result pill (EXACT in green / SHORT −N in red, reusing the existing
`.banner.ok`/`.banner.bad` color tokens for consistency). Tapping a row toggles
an expanded box-chip list beneath it (reusing the existing `chip()`/`.chips`
markup, read-only — no delete/edit affordances). Each row also has a "Print"
button.

Empty state: "No reconciliations logged yet." (reusing the existing `.empty`
pattern).

Footer buttons: "Clear history" (`btn sec`, with a `confirm()` guard mirroring
the existing "New" confirmation) and "Close".

**Reprint function** `printHistoryEntry(i)` rebuilds the exact same slip
markup `printReconcile()` produces (recomputing `solveExact` fresh from the
stored `pick`/`boxes` — deterministic, so it reproduces the original result
exactly), but stamps the footer with the entry's original `ts` plus `(reprint)`
instead of the current time.

## Out of scope

- Optimize module is untouched — no history logging there.
- No operator/worker ID field (a separate, not-yet-approved idea).
- No shortage reason codes (a separate, not-yet-approved idea).
- No export (CSV/file) of history — view + reprint only.
- No rack/location field in history entries — Reconcile has no rack concept
  (deliberately removed in an earlier change); this spec does not revisit
  that decision.

## Acceptance criteria

- Printing a finished reconciliation (pieces ≥ pick) adds exactly one history
  entry; pressing "New" right after does not add a second one.
- Pressing "New" on a finished-but-never-printed reconciliation logs exactly
  one entry.
- Printing while `pieces<pick` is blocked with a flash message and does not
  log anything (verifies the bug-fix gate).
- Scanning/editing/deleting a box, or changing the pick, after a log was
  recorded allows exactly one more entry on the next print/New.
- The History overlay lists entries newest-first, expands box detail per row,
  and "Print" on a past entry reproduces the same EXACT/SHORT verdict with the
  original timestamp.
- "Clear history" empties `packcalc_history` after confirmation.
- History never exceeds 200 entries.
- `node --check` still passes on the inline script.
