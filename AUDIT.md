# AUDIT.md — Acumatica Balance Calculator (ACal)

Audited cold, from a fresh clone of `phantomdenied/ACal` @ `master` (commit `2627454`).
All line numbers below refer to `balance_calculator.html` as it exists in that clone.

## 1. Repo structure

The repo contains exactly **one file**: `balance_calculator.html` (686 lines). No `index.html`,
no `package.json`, no build step, no framework — plain HTML/CSS/vanilla JS in a single document.
No `README`, `LICENSE`, `.github/` workflows, `docs/` folder, or `CNAME`.

**Deployment status: unconfirmed, and likely "not deployed."** There is no `docs/` folder, no
`gh-pages` branch, no GitHub Actions workflow, and no `CNAME` file — none of the usual signals
that GitHub Pages is turned on for this repo. I don't have a tool that can read or change a
repo's Pages settings (only file/branch/PR-level tools), and this sandbox's network egress to
`*.github.io` is blocked, so I couldn't directly confirm by hitting the URL either. Best guess:
you currently open this file locally / via a raw GitHub link, not a live Pages URL. **This
matters for Step 6 of your plan** — see the Judgment Calls in PLAN.md.

## 2. What the app currently does

Three (functionally four) sections, all client-side, using SheetJS (`xlsx.full.min.js` from a
CDN, line 7) to read/write `.xlsx`:

1. **Upload** (lines 96–107, handler at 211–244): multiple Acumatica account-history `.xlsx`
   files. Each file = one account. Parses filename as `INITIALS AMOUNT.xlsx` (e.g.
   `SW 237.76.xlsx` → initials `SW`, amount `237.76`) via `parseFilename()` (lines 678–682).
2. **Single-account balance estimator** (lines 109–140, `calculate()` at 427–483): pick one
   uploaded account, current balance auto-fills from the filename (`autoFillCurrentBalance()`,
   365–383), pick a target date, and it walks backward from "now" by reversing every tracked
   transaction *after* that date.
3. **Batch mode** (lines 142–156, `calculateBatch()` at 485–541) — the feature added this
   session: one target date, every uploaded account listed with its estimated balance, using
   each file's filename-parsed amount as the starting balance. No manual override.
4. **Multi-account audit export** (lines 158–185, `exportReport()` at 543–653): date range +
   a known ending balance per account → downloads one `.xlsx` workbook with one tab per account,
   each tab showing a reconstructed running balance and reconciliation summary.

Tracked transaction codes are hardcoded: credits `{690, 730, 750, 760}`, debits `{670, 710}`
(lines 171–181... actually 188–198 in current file). Everything else is ignored.

## 3. Versioning

**None exists.** No version string, no `<!-- v... -->` comment, nothing in the UI. There is no
convention to follow — one needs to be established (flagged as a judgment call in PLAN.md).

## 4. Existing help text vs. actual functionality — DRIFTED

The only in-app help is the "Tracked Transaction Codes" legend (lines 77–94) and the subtitle
(line 75: *"Upload one or more account history files · Estimate past balances · Export audit
report"*). Neither mentions:
- the `INITIALS AMOUNT.xlsx` filename convention or auto-fill behavior (added over two commits,
  `d5a15ad` and today's session, but never surfaced in the legend/subtitle),
- Batch Mode (section 2B) at all — it's a whole feature with zero explanation of *why* it only
  works for filename-matched files, or what "current balance" is assumed to mean (see below).

This is a direct case of help text not matching functionality, per your Step 1 ask.

## 5. Bugs (logic that doesn't match its own labels, or breaks)

**B1 — Step 3's "Ending Balance" field is pre-filled with the wrong number, mislabeled.**
Line 175 labels the column "Ending balance for each account (**as of the report end date**)".
But line 407's prefill (`f.parsedAmount != null ? f.parsedAmount : ''`) uses the balance parsed
from the filename — which represents the balance *right now* (per your own description: "SW
237.76.xlsx... there is 237.76 current cash balance"). Those are only the same number when the
report's end date happens to be today. Pick any past end date and the prefilled "ending balance"
is silently wrong unless the user notices and overwrites it. This is the same bug pattern the
audit was told to look for explicitly: "logic that doesn't match its own labels."

**B2 — `exportReport()` has no error handling; `parseFile()` does.** `parseFile()` (251–336)
wraps everything in try/catch and reports failures per-file. `exportReport()` (543–653) has no
try/catch at all — if `XLSX` failed to load (CDN down/blocked) or anything else throws mid-export,
the user gets a silent JS console error and a dead "Download Excel Report" button with no
on-page message, unlike every other action in the app which reports status via `.status` divs.

**B3 — Race condition on rapid re-upload.** `fileInput`'s change handler (211–244) resets
`loadedFiles = []` and `pendingCount` synchronously, then kicks off async `FileReader`s per file,
each closing over its own `idx`. If a user selects a new file set before the previous set's
readers finish, in-flight callbacks from the *old* batch will still fire and write into the
*new* (shorter or differently-ordered) `loadedFiles` array — either corrupting a slot or writing
past the end silently. Low-probability in practice (files usually parse in well under a second)
but real.

**B4 — Column detection silently falls back to hardcoded letters beyond a fixed window.**
`colLetters = 'abcdefghijklm'.split('')` (line 275, i.e. columns A–M only) is scanned for header
text. If a real Acumatica export's "Amount" or "Date" header lives past column M, or is worded
differently than substring-matched (`'amount'`, `'transaction'`, exact `'description'`), detection
fails and the code falls back to hardcoded columns A/B/D/E (lines 282–289) *without telling the
user* — it'll either silently misread data or silently pass the "no valid transactions" check
with wrong data. I can't verify against a real export file since none is in the repo; flagged
as a judgment call too.

**B5 — Amount sign is assumed, not verified.** The app never inspects the sign of the `Amount`
column — it treats every tracked-code row as an unsigned magnitude and infers direction purely
from the code (credit set vs. debit set, lines 188–189, applied at 449–451 / 521–524 / 601–603).
If Acumatica actually exports signed amounts (negative for debits), the math would double-count
direction. No sample file in the repo to verify against — flagged as judgment call.

## 6. Unfinished features

**U1 — Batch Mode has no manual balance entry.** Section 2B (142–156) only works for accounts
whose filename matches the `INITIALS AMOUNT.xlsx` pattern; anything else is listed as "skipped"
with no way to fill in a number, unlike Step 3's table which gives every account an editable
input (line 418). Feels like the half of the feature that didn't get built.

**U2 — No way to remove/clear an uploaded file or reset the tool** without re-triggering the
native file picker (and in some browsers, re-selecting the exact same files doesn't even
re-fire the `change` event). No "×" per row, no "clear all."

## 7. Dead code / leftovers

None found — no `console.log`, `TODO`, `FIXME`, commented-out blocks, or unused functions/CSS
classes. This part of the app is clean.

## 8. Polish / UX gaps

- **No horizontal scroll wrapping** on the transaction table or file tables for narrow
  viewports — `.tx-table-wrap` (line 48) only scrolls vertically (`overflow-y`), and the
  5–7 column tables will overflow on a phone screen.
- **Labels aren't associated with inputs** (no `for="..."` / matching `id`) anywhere in the
  form — works visually but is inaccessible to screen readers and doesn't let a tap on the
  label focus the field.
- No favicon, no `<meta name="description">`.
- No validation/warning for a negative current balance, a target date in the future, or a
  target/report date range with zero tracked transactions in it (the last one is handled
  gracefully in the export, less so as a proactive warning).
- CDN script (line 7) has no Subresource Integrity hash and no offline fallback — if the CDN is
  unreachable the whole app silently breaks (this is exactly what happened when I tested
  auto-fill/batch mode this session inside a network-restricted sandbox — every file errored
  with "XLSX is not defined" until I substituted a local copy of the library for the test).

## Summary

The two features added this session (auto-fill balance from filename, batch target-date mode)
work correctly and are already live on `master`. The rest of the app is small, clean of cruft,
but has one real mislabeled-logic bug (B1), one missing-error-handling gap (B2), a low-probability
race condition (B3), two data-format assumptions I can't verify without a real Acumatica sample
file (B4/B5), an intentionally-incomplete-looking batch mode (U1), and no version/help-text
discipline yet. Nothing here looks catastrophic — this reads like a fast, competent build that
was never given a finishing pass.
