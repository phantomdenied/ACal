# PLAN.md — Acumatica Balance Calculator

Derived from AUDIT.md. Nothing here is implemented yet — this is the proposal for review.

## Bugs to fix

1. **B1 — Step 3 "Ending Balance" mis-prefill.** Stop prefilling that field from the filename
   amount unless the report end date is actually today. See Judgment Call J1 for the exact
   behavior to implement.
2. **B2 — Wrap `exportReport()` in try/catch**, report failures through `#exportStatus` the same
   way every other action in the app reports status, instead of failing silently to the console.
3. **B3 — Fix the re-upload race condition.** Give each upload batch a generation/token number;
   have in-flight `FileReader` callbacks check they still belong to the current generation before
   writing into `loadedFiles`, and no-op (not throw) if a newer batch has started.
4. **B4 — Widen/harden column detection** past the 13-letter (A–M) window, and **surface a
   visible warning** when header-based detection fails and the app falls back to hardcoded
   column letters, instead of silently trusting a guess.
5. **B5 — Verify amount-sign handling against a real sample file** (see Judgment Call J2)
   and adjust the math if signed amounts turn out to be in play.

## Unfinished features to complete

1. **U1 — Add manual balance entry to Batch Mode** for any account whose filename didn't parse,
   mirroring the editable-input pattern already used in Step 3's table, so "skipped" accounts
   become fillable instead of dead ends.
2. **U2 — Add a way to clear/remove uploaded files** (a "×" per row and/or a "Clear All" button)
   so the tool doesn't require juggling the native file picker to start over.

## Polish items

1. Add `overflow-x: auto` wrappers around all data tables so they don't break layout on phone
   screens.
2. Associate every `<label>` with its input via matching `for`/`id`.
3. Add a favicon and a `<meta name="description">`.
4. Add light input validation/warnings: negative current balance, target date in the future,
   date ranges with zero tracked transactions (as a visible heads-up, not just a quiet result).
5. Update the in-app help/legend (top of page) to document the filename convention, the
   auto-fill behavior, and Batch Mode's dependency on that convention — closing the help-text
   drift noted in AUDIT.md §4.
6. Establish and display a version string (see Judgment Call J3).

## Judgment calls — RESOLVED

- **J1** → Never prefill Step 3's "Ending Balance" field. Always start blank.
- **J2** → Amounts are unsigned; keep current math (direction inferred from code only). Document
  the assumption in help text.
- **J3** → `vX.Y.Z` semantic versioning, bumped every meaningful change, applied to `index2.html`
  going forward. (Not added to `balance_calculator.html` — Step 5 restricts that file to a single
  added link, so no version footer goes there.)
- **J4** → User will enable GitHub Pages (Settings → Pages → Deploy from `master`/root) themselves.
- **J5** → New file is `index2.html`; the "Test version" link goes at the bottom of
  `balance_calculator.html`, which is the only change made to that file.

## Judgment calls — need your input before I touch code (archived, all resolved above)

**J1 — What should Step 3's "Ending Balance" field actually do?**
Options: (a) only auto-prefill when the report end date equals today's date, leaving it blank
otherwise; (b) never auto-prefill it at all — remove that behavior entirely since "ending
balance" and "current balance from filename" are different concepts; (c) keep prefilling as-is,
label it more honestly (e.g. rename to "starting point — verify against your end date"), and
let the user self-correct. I'd lean toward (a) as the least surprising fix, but this is your
data-entry workflow, not mine to guess at.

**J2 — Are the Amount values in a real Acumatica export signed or unsigned?**
The whole credit/debit math hinges on this (AUDIT.md B5), and there's no sample file in the repo
to check against. If you can attach or describe one real export's Amount column for a known
debit and a known credit row, I can verify (or fix) this in five minutes. Otherwise I'll leave
the current behavior (treat as unsigned, direction from code only) untouched and just document
the assumption in the help text, since I have no evidence it's wrong.

**J3 — Versioning convention.**
Nothing currently exists. Proposal: a small `v0.1.0`-style string in the footer for the current
production page (retroactively, since this is genuinely a first real feature set), bumped on
every meaningful change from here on, plus a visible "TEST / BETA" badge on `index2.html` so
it's never confused with the live page. Tell me if you'd rather use dates, build numbers, or
skip versioning on the original page entirely.

**J4 — GitHub Pages / live URL status is unconfirmed (blocks Step 6 as written).**
I found no `docs/` folder, no `gh-pages` branch, no Pages workflow, and no `CNAME` in this repo,
and I can't query or change repo Settings → Pages from here (no tool exposes that, and this
sandbox can't reach `*.github.io` to check by hand). My best guess is this repo isn't currently
served by GitHub Pages at all — you likely open the file locally or via a raw/blob GitHub link.
If you want real `https://phantomdenied.github.io/ACal/...` URLs at the end of this, **you'll
need to flip Settings → Pages → "Deploy from a branch: master / (root)" on once** (a 10-second
change only you or a repo admin can make) — after that, every push I make will go live
automatically with no further action from you. If Pages is already on and I'm just missing a
signal, let me know and I'll skip this. Either way, pushing the code itself doesn't depend on
this — only the "visit a live URL" part of Step 6 does.

**J5 — `index.html` doesn't exist, so "link from the original page" needs a target.**
The live/original entry point is `balance_calculator.html`, not `index.html`. Two sane options:
(a) add the "Test version" link to the bottom of `balance_calculator.html` itself, and name the
new file `index2.html` as you specified; or (b) rename nothing but note that if Pages ever does
get enabled with "root" serving, `index2.html` would auto-serve at the repo's Pages root while
`balance_calculator.html` needs its full filename in the URL either way — GitHub Pages has no
automatic "default page" behavior for a file that isn't literally named `index.html`. I'd
recommend (a): leave `balance_calculator.html` as the one entry point, add the link at its
bottom, and create `index2.html` alongside it. Let me know if you want it named something else
instead (e.g. `balance_calculator_v2.html`) so both files are equally discoverable if someone's
browsing the repo instead of a Pages URL.
