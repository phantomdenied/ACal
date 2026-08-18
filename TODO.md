# TODO.md — Acumatica Balance Calculator buildout

Ordered checklist for Step 4, grouped so related edits happen together in one pass (and one
commit) instead of touching the same functions repeatedly. **Nothing here starts until PLAN.md
is reviewed and the judgment calls (J1–J5) are resolved or explicitly deferred.**

## Group 0 — Setup
- [ ] Copy `balance_calculator.html` → `index2.html` (or the renamed target from J5), unmodified,
      as the starting point for all further work.
- [ ] Add a visible "TEST / BETA — not the production page" badge to `index2.html`.
- [ ] Establish the version string per J3 and display it in the footer of `index2.html`.

## Group 1 — Core bug fixes (parsing & math)
- [ ] B4: widen column detection beyond A–M; add a visible "using fallback column mapping"
      warning when header text isn't found.
- [ ] B5: verify (or document as an open assumption) signed-vs-unsigned Amount handling per J2.
- [ ] B3: fix the re-upload race condition with a generation token on `loadedFiles`/`parseFile`.

## Group 2 — Step 3 export fixes
- [ ] B1: change the "Ending Balance" prefill behavior per the resolution of J1.
- [ ] B2: wrap `exportReport()` in try/catch, report failures via `#exportStatus`.

## Group 3 — Batch Mode completion
- [ ] U1: add editable manual-balance inputs to Batch Mode for filename-unmatched accounts,
      mirroring Step 3's table pattern.
- [ ] Re-verify batch math against the same test cases used to validate it this session
      (known transactions, known target date, hand-computed expected balance).

## Group 4 — File management UX
- [ ] U2: add per-row remove ("×") and a "Clear All" control for uploaded files.

## Group 5 — Polish pass
- [ ] Add `overflow-x: auto` wrappers to all data tables.
- [ ] Associate all `<label>`s with their inputs via `for`/`id`.
- [ ] Add favicon + meta description.
- [ ] Add validation/warnings: negative current balance, future target date, zero-transaction
      ranges.
- [ ] Rewrite the top-of-page legend/subtitle to document the filename convention, auto-fill,
      and Batch Mode — close the help-text drift from AUDIT.md §4.

## Group 6 — Verification
- [ ] Manually re-derive expected output for at least 2 accounts × 2 target dates using
      synthetic test `.xlsx` files (same approach used to verify this session's changes),
      confirm `index2.html` matches by hand-calculation.
- [ ] Load-test with a file that deliberately violates the filename convention, an empty file,
      and a file with headers in an unexpected order, to confirm the new warnings/fallbacks
      fire correctly instead of silently misbehaving.

## Group 7 — Ship
- [ ] Add the small "Test version →" link to the bottom of `balance_calculator.html`
      (only change allowed to that file).
- [ ] Commit in logical chunks (roughly one commit per group above, not one giant commit).
- [ ] Push to `master` (confirmed as this repo's default branch).
- [ ] Resolve J4 (Pages) with you, or confirm current serving method, before promising live URLs.
- [ ] Final delivery summary + both URLs (or clear next step if J4 is still open).
