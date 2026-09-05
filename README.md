# The Card Box (future home of a Card Box + Spike merge)

This repo is meant to become the merged home of two sibling projects:

- **`index.html`** (this app, "Card Box") — the index-card home-routine
  method, considered the better/more current version going forward.
- **`legacy-spike/index.html`** — the original Spike app (corkboard +
  recurring-task productivity tool) kept as-is for reference until the
  merge happens. Do not treat this as dead code to delete casually —
  it's the source of the due-date engine and visual language Card Box
  already borrows from, and may still have board/spike features worth
  carrying over.

## Merge intent

The eventual goal is one app, not two. Card Box already reuses Spike's
palette, fonts, and due-date engine conventions. What Spike has that
Card Box doesn't yet: the "This Week" corkboard/post-it board view, a
mini calendar, and week-based color coding — evaluate whether any of
that is worth pulling into Card Box, or whether Card Box's index-box
metaphor fully replaces the old recurring-tasks view.

A single-file HTML app implementing the "index card home routine" method
(a la Sidetracked Home Executives / Natasha's Healthy Living) — chores are
written on cards, sorted into Daily / Weekly / Monthly / Seasonal / Personal,
and today's due cards surface automatically in a recipe-box style view.

## Status

Working prototype, single file: `index.html`. No build step — open it
directly in a browser, or serve it statically.

## Data model

Each card: `{ id, name, zone, category, points, weekday?, dayOfMonth?, month?, nextDue, lastDone }`

- `category` is one of `daily | weekly | monthly | seasonal | personal` and
  determines both the color coding and the recurrence engine.
- `weekly` cards pin to a `weekday` (mon..sun); `monthly` cards pin to a
  `dayOfMonth`; `seasonal` cards pin to a `month` + `dayOfMonth` and recur
  yearly; `daily`/`personal` reset every day.
- Due-date math advances from the card's *own* `nextDue`, not from "today",
  so a late completion doesn't compress the next cycle.
- `nextDue < today` → overdue (amber ring); `nextDue === today` → due today
  (terracotta ring); otherwise upcoming (not shown in the daily box).

## Persistence

Uses the Claude Artifact `window.storage` API (get/set/delete/list), not
localStorage — this file is built to run as a Claude.ai artifact as well as
a plain static page. If serving outside Claude.ai, `window.storage` won't
exist and card data won't persist — that's the main thing to fix if this
becomes a standalone deployed app (swap in localStorage or a real backend).

## Visual language

Shares a palette and paper/index-card aesthetic with a sibling project,
"The Spike" (a separate corkboard + recurring-task productivity app):
Fraunces (headings), Work Sans (body), IBM Plex Mono (labels/meta).
Category colors: daily = ochre, weekly = dusty blue, monthly = clay,
seasonal = sage, personal = dusty rose.

## Known rough edges / open questions

- Seasonal cards fire once a year on a fixed date. Anything meant to
  happen quarterly currently needs 3-4 separate seasonal cards rather
  than one recurring quarterly card — worth deciding if that's fine or
  if seasonal should support a repeat-interval instead.
- The due-date engine was hand-derived to match Spike's approach rather
  than sharing literal code with it — worth spot-checking against real
  dates (especially month-end clamping for day-of-month cards).
- No edit-in-place validation yet (e.g. changing category doesn't warn
  you that the due date will be recalculated from scratch).
