# Scan Calculator 0.2

Single-file warehouse pick tool: **everything lives in `index.html`** (inline CSS + JS, vanilla, no
framework, no build, no deps, no git). Runs offline from `localStorage` on a Zebra TC520L 5" handheld
(Chromium WebView). UI in English; design = light bento grid with a dark header bar by default (see
`docs/superpowers/specs/2026-06-17-light-bento-restyle-design.md`), with a dark-OLED theme toggle
(`cfg.theme`, `data-theme` attribute on `<html>`, see
`docs/superpowers/specs/2026-06-17-dark-light-theme-toggle-design.md`); codes/quantities use a
monospace system font via `--mono`.

## Run & test (Windows / PowerShell)
- Open the app: `Start-Process index.html` (launches default browser).
- Syntax-check the inline script (PowerShell):
  ```
  $l=Get-Content index.html; $s=($l|Select-String '^<script>'|Select-Object -First 1).LineNumber; $e=($l|Select-String '^</script>'|Select-Object -First 1).LineNumber; $l[$s..($e-2)]|Set-Content tmp.js -Encoding utf8; node --check tmp.js; Remove-Item tmp.js
  ```
- Test pure logic: copy the engine functions into a temp Node script and assert vs a brute-force
  oracle (this is how the subset-sum engine was verified — 400 random cases vs bitmask brute force).
- Bash tool: cwd resets between calls — always use absolute paths. Prefer Glob over `ls`/`find`.
- Syntax-check via Bash/Git Bash:
  ```bash
  cd "<project dir>" && node -e "
  const fs=require('fs');const l=fs.readFileSync('index.html','utf8').split(/\r?\n/);
  let s=-1,e=-1;for(let i=0;i<l.length;i++){if(s===-1&&/^<script>/.test(l[i]))s=i;if(s!==-1&&/^<\/script>/.test(l[i])){e=i;break;}}
  fs.writeFileSync('tmp.js',l.slice(s+1,e).join('\n'),'utf8');"
  node --check tmp.js && rm -f tmp.js
  ```
- Claude cannot view the rendered app or print output in this environment — after `Start-Process
  index.html`, ask the user to confirm visually rather than assuming.

## Critical rules — getting these wrong corrupts counts
- Box barcode → pieces: strip ALL non-digits + leading zeros → `parseInt` (e.g. `NL-00250-A` → 250).
- Rack location: matches `^[A-Z]{2}[0-9]{8}$` (e.g. `CG35071601`). Stored VERBATIM, never cleaned to a
  number. Auto-routed in the **Optimize** scan field only: rack regex → set location; else → count as
  box. A rack miscounted as a box = catastrophic there. Box codes must never match the rack regex.
- **Reconcile has no location/rack concept** — every scan in that module is cleaned to pieces and
  counted as a box, including rack-shaped codes (deliberate; Reconcile only tracks pick vs scanned
  boxes, no per-box location).
- Subset-sum reconstruction must never reuse the same physical box; verify with edge cases.
- **Never pick over target**: `solveExact`/`solveUnder` must only match/approach the pick or target from
  ≤ — deliberate WMS rule (team leader adjusts the order down to available stock), not a limitation to relax.
- Any code path calling `solveExact(boxes, pick)` must first check `pieces>=pick` (reachedPick), or it
  reports a false "NOT POSSIBLE" mid-scan — already bit `renderRecon()` and `printReconcile()` separately.
- Hands-free scan (`wireScan()`): Enter/Tab fires instantly; the idle-burst auto-fire (~100ms) only arms
  once two keystrokes land ≤`SCAN_BURST_GAP_MS` (60ms) apart (scanner speed) — slower manual typing falls
  back to requiring Enter/Tab, so it never auto-submits a half-typed number. Keep scan field auto-focused.
- Print uses a hidden iframe with dynamic `@page` — it's a SEPARATE document, so it CANNOT use the inline
  SVG sprite. Keep print output plain B/W text only.
- Thermal-label printing auto-shrinks to fit one physical label (`fitToLabel()` in `doPrint()`, via CSS
  `zoom`); `.chip`/`.grp` use `break-inside:avoid` so a box can never be split across two labels.
- Icons: inline SVG sprite (`<use href="#i-…">`), never emoji — **except** the 4 Home-screen list rows
  shown in dark mode (Reconcile/Optimize/Settings/Printer: 📦🧮⚙️🖨️), which use real emoji per
  `docs/superpowers/specs/2026-06-22-dark-emoji-home-redesign-design.md`. The sprite has NO help/question
  icon — use a plain `?` text character inside `.iconbtn` when a help button is needed.

## localStorage keys
`packcalc_cfg` (printer, mode, theme, site) · `packcalc_pallets` · `packcalc_active` · `packcalc_opt` (target) ·
`packcalc_reconcile` (pick, boxes, logged) · `packcalc_history` (array of completed Reconcile results, capped
at 200 entries, newest first — see `docs/superpowers/specs/2026-06-18-reconcile-history-log-design.md`).

## Architecture notes
- Design rationale for past features lives in `docs/superpowers/specs/` (design specs) and
  `docs/superpowers/plans/` (implementation plans) — check there before re-deriving why something works.
- `setMode(m)` (~line 900) switches Reconcile ↔ Optimize. Any feature that holds cross-screen state
  must reset it at the top of `setMode` — otherwise state bleeds when the user navigates between modes.
- Web app manifest is embedded in `<head>` as a `data:application/manifest+json;base64,...` link (display:
  standalone), kept single-file on purpose. To change name/colors/icons, decode the base64, edit the JSON,
  re-encode, and replace the `href`. Paired with `apple-mobile-web-app-capable` / `mobile-web-app-capable`
  meta tags so Chrome/Android "Add to Home screen" and iOS "Add to Home Screen" open without browser chrome.
