# The Card Box

A single-file HTML app implementing the "index card home routine" method
(a la Sidetracked Home Executives / Natasha's Healthy Living) — chores are
written on cards, sorted into Daily / Weekly / Monthly / Seasonal / Personal,
and today's due cards surface automatically in a recipe-box style view.

It's built as a digital card-filing system that mirrors its physical
counterpart in both look and usability: an index box of routine chore
cards you pull for the day, and a corkboard of one-off tasks you physically
"spike" (skewer) once they're done.

- **`index.html`** — the app: the recurring-card "Today" box, plus a
  "This Week" corkboard/spike view for ad hoc tasks, and a persistent
  mini calendar with week-based color coding.
- **`legacy-spike/index.html`** — the original standalone Spike app
  (corkboard + recurring-task tool), kept as reference. Its due-date
  engine, corkboard/spike interaction, and mini calendar have all been
  ported into `index.html`; this file is not otherwise maintained.

## Status

Working app, single file: `index.html`. No build step — open it directly
in a browser, or serve it statically.

## Views

- **Today** — the index box: cards due today or overdue, pulled from the
  whole recurring-card deck below. Tap a card to mark it done (and again
  to undo same-day). A "Whole box" list lets you add, edit, and delete
  cards.
- **This Week** — a corkboard for one-off tasks that don't belong on a
  recurring card. Tap a note to edit it, double-tap to spike it (with a
  thunk sound, haptic buzz, and a paper-fleck burst) onto the pile on the
  right.

## Data model

Recurring cards: `{ id, name, zone, category, points, weekday?, dayOfMonth?, month?, nextDue, lastDone }`

- `category` is one of `daily | weekly | monthly | seasonal | personal` and
  determines both the color coding and the recurrence engine.
- `weekly` cards pin to a `weekday` (mon..sun); `monthly` cards pin to a
  `dayOfMonth`; `seasonal` cards pin to a `month` + `dayOfMonth` anchor and
  repeat every `intervalMonths` (1, 2, 3/quarterly, 4, 6, or 12/yearly —
  defaults to 12 for cards saved before this field existed); `daily`/
  `personal` reset every day.
- Due-date math advances from the card's *own* `nextDue`, not from "today",
  so a late completion doesn't compress the next cycle. Verified against
  month-end clamping and leap years (e.g. a day-31 monthly card clamps to
  Feb 28/29 and un-clamps back to 31 the following month; a Feb-29
  seasonal card clamps to Feb 28 in non-leap years and recovers Feb 29 on
  the next one).
- `nextDue < today` → overdue (amber ring); `nextDue === today` → due today
  (terracotta ring); otherwise upcoming (not shown in the daily box).

One-off notes (This Week): `{ id, text, category, weekKey, weekNum, colorHex, colorName, spiked, createdAt, spikedAt }`

- `category` here is free text, used only for filter chips — unlike
  recurring cards, notes are colored by the ISO week they were created in
  (shared `colorForWeek()` mapping with the mini calendar), not by
  category, so stale notes visually stand out from this week's.

## Persistence

Uses a small storage adapter that prefers the Claude Artifact
`window.storage` API when present (get/set) and falls back to
`localStorage` otherwise, so the same file runs unmodified as a Claude.ai
artifact or as a plain static page.

## Visual language

Fraunces (headings), Work Sans (body), IBM Plex Mono (labels/meta).
Category colors (recurring cards): daily = ochre, weekly = dusty blue,
monthly = clay, seasonal = sage, personal = dusty rose. Week colors (This
Week notes + mini calendar): a 6-color palette cycling by ISO week number.

## Known rough edges / open questions

- Editing a card's category now shows an inline warning that its due
  date will be recalculated from scratch, but there's no equivalent
  guard on the This Week side (e.g. editing a note's category doesn't
  warn about anything, since notes don't have a due-date to reset).
- Editing a seasonal card's repeat interval (without changing category)
  doesn't recompute `nextDue` — it only affects how the card advances
  *after* its currently scheduled occurrence. This mirrors the existing
  category-change behavior but hasn't been surfaced as its own inline
  note; worth a small warning if it proves confusing in practice.
