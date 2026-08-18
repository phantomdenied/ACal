# TODO.md — Acumatica Balance Calculator buildout

Ordered checklist for Step 4, grouped so related edits happen together in one pass (and one
commit) instead of touching the same functions repeatedly. **Nothing here starts until PLAN.md
is reviewed and the judgment calls (J1–J5) are resolved or explicitly deferred.**

## Group 0 — Setup
- [x] Copy `balance_calculator.html` → `index2.html` (or the renamed target from J5), unmodified,
      as the starting point for all further work.
- [x] Add a visible "TEST / BETA — not the production page" badge to `index2.html`.
- [x] Establish the version string per J3 and display it in the footer of `index2.html`.

## Group 1 — Core bug fixes (parsing & math)
- [x] B4: widen column detection beyond A–M; add a visible "using fallback column mapping"
      warning when header text isn't found.
- [x] B5: verify (or document as an open assumption) signed-vs-unsigned Amount handling per J2.
- [x] B3: fix the re-upload race condition with a generation token on `loadedFiles`/`parseFile`.

## Group 2 — Step 3 export fixes
- [x] B1: change the "Ending Balance" prefill behavior per the resolution of J1.
- [x] B2: wrap `exportReport()` in try/catch, report failures via `#exportStatus`.

## Group 3 — Batch Mode completion
- [x] U1: add editable manual-balance inputs to Batch Mode for filename-unmatched accounts,
      mirroring Step 3's table pattern.
- [x] Re-verify batch math against the same test cases used to validate it this session
      (known transactions, known target date, hand-computed expected balance).

## Group 4 — File management UX
- [x] U2: add per-row remove ("×") and a "Clear All" control for uploaded files.

## Group 5 — Polish pass
- [x] Add `overflow-x: auto` wrappers to all data tables.
- [x] Associate all `<label>`s with their inputs via `for`/`id`.
- [x] Add favicon + meta description.
- [x] Add validation/warnings: negative current balance, future target date, zero-transaction
      ranges.
- [x] Rewrite the top-of-page legend/subtitle to document the filename convention, auto-fill,
      and Batch Mode — close the help-text drift from AUDIT.md §4.

## Group 6 — Verification
- [x] Manually re-derive expected output for at least 2 accounts × 2 target dates using
      synthetic test `.xlsx` files (same approach used to verify this session's changes),
      confirm `index2.html` matches by hand-calculation.
- [x] Load-test with a file that deliberately violates the filename convention, an empty file,
      and a file with headers in an unexpected order, to confirm the new warnings/fallbacks
      fire correctly instead of silently misbehaving.

## Group 7 — Ship
- [x] Add the small "Test version →" link to the bottom of `balance_calculator.html`
      (only change allowed to that file).
- [x] Commit in logical chunks (roughly one commit per group above, not one giant commit).
- [x] Push to `master` (confirmed as this repo's default branch).
- [x] Resolve J4 (Pages) with you, or confirm current serving method, before promising live URLs.
- [x] Final delivery summary + both URLs (or clear next step if J4 is still open).
