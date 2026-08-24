# PiperPump

A personal home-workout web app — single HTML file, no build step, no
framework, no dependencies beyond Google Fonts. Built over an extended session in
Claude chat; this file exists so a fresh Claude Code session doesn't have to
re-derive any of it from scratch.

## What it is

A 5-day-a-week (Mon–Fri) kettlebell-primary training program, plus two optional
weekend days (Sat = full-body conditioning circuit, Sun = light recovery/mobility),
with exercise animations, a timer, completion tracking, weekly/monthly adherence
charts, and a 3-week rotating program (Hypertrophy → Strength → Metabolic focus,
undulating periodization). Designed around one specific home-gym equipment set and
a stated knee sensitivity.

## User context (so you don't have to ask again)

- Goals: lose weight, build muscle. Wants effective, evidence-based programming —
  this has been checked against ACSM/NSCA guidance and adjusted accordingly
  (see "Programming notes" below).
- Equipment: adjustable kettlebell (10–40 lb), adjustable dumbbells (5–50 lb/side),
  adjustable bench, Jayflex barbell (holds the dumbbells as plates — no confirmed
  squat rack, so heavy barbell squatting isn't programmed), treadmill, weighted
  vest, resistance bands, ab roller, stability ball, 20 lb and 30 lb medicine
  balls with handles (hard rubber, **not** slam/wall balls — no slamming, no
  throwing at walls, home gym is in a basement).
- Knee-conscious: prefers depth capped at parallel, reverse (not forward) lunges,
  knee-tracking cues on squat-pattern moves.

## Architecture

**Single file, vanilla JS.** No React, no bundler. Everything — CSS, pose data,
program data, rendering logic — lives in `index.html`.

**Persistence:** `localStorage`, key `mb:v4`. (Started as Claude-artifact-only
`window.storage`, migrated to plain `localStorage` so it works on any host —
see git history / chat log if you need the old migration logic.)

**Figure illustrations — two separate rigs, both forward-kinematics, both
verified by rendering to PNG and visually inspecting before porting to JS:**
- Sagittal (side-view) rig: `figureGroup()` / `paint()`. Near/far limb pairs for
  depth, a torso silhouette (not a stick figure), tapered capsule limbs.
- Frontal (front-view) rig: `figureGroupF()` / `paintF()`. Used only where a
  front view actually adds information the side view can't — lateral raise,
  suitcase carry (anti-lateral-flexion), windmill, and knee-tracking on the
  goblet squat. Everything else is side-view only; don't add front views
  reflexively, only where the view genuinely clarifies the movement.
- A "bench" element (`figureGroup`'s `.bench`) renders for exercises that need
  one (flat bench press, hip thrust, rear-foot-elevated split squat). Bench
  press has its own dedicated pose — don't let it drift back to reusing floor
  press's pose, they're biomechanically different (bench height + leg position).
- **If you add new poses:** don't hand-tune joint angles blind. Build the
  candidate in a throwaway Python/matplotlib script (pattern exists in earlier
  chat history — near-black background, white figure, volt load marker), render
  it, actually look at the image, iterate, *then* port the exact numbers to JS.
  Every pose in this app was built that way and it's caught real bugs (limbs
  crossing, floating off the ground, mismatched anatomy) that pure code review
  missed every time.

**Program data — `WEEK_A` / `WEEK_B` / `WEEK_C`, selected by ISO week number:**
```js
const WEEKS=[WEEK_A,WEEK_B,WEEK_C];
const PROGRAM=WEEKS[(isoWeekNumber(new Date())-1)%WEEKS.length];
```
- **Hard constraint, do not break it:** every week variant must have the exact
  same number of exercises for a given day-of-week slot (currently
  Mon=6, Tue=7, Wed=7, Thu=6, Fri=7, Sat=5, Sun=4 — consistent across all three
  weeks). `sessionDoneOnDate()` checks completion by comparing checked count
  against `PROGRAM[dayIdx].ex.length` — if that length can differ depending on
  which week is live, historical completion records silently become wrong for
  dates logged under a different week variant. If you need to add/remove an
  exercise, change the count in **all three weeks** for that day, or accept
  (and tell the user) that past records for that day will be affected.
- Wednesday's kettlebell complex deliberately avoids swings — it sits ~24h
  before Thursday's heavy deadlift/snatch day, and swings would overlap the
  same posterior-chain recovery window (this was a real fix, not a style
  choice — flagged by checking the program against ACSM/exercise-science
  sources).
- **Day indexing is 0=Mon..6=Sun everywhere** (`S.day=(new Date().getDay()+6)%7`).
  Sat/Sun were added later as optional bonus days — Sat is a full-body metabolic
  circuit (kettlebell + medicine ball + vest), Sun is light recovery/mobility with
  identical content across all three week variants (same reasoning as Wednesday's
  consistent "Low impact" tag). The weekday-only adherence stats (`sDone` on the
  home screen, the finish-card "this week" stat) deliberately stay `/5` — Sat/Sun
  are bonus, not required, so they're excluded from the core adherence target.
  The weekly-adherence pip chart (`buildWeekTrackerPanel`) does show all 7 days.
- Every `PROGRAM[i]` day object also carries a `watchMode` field — a suggested
  Apple Watch workout-mode string, rendered on the workout intro screen
  (`buildOverviewCard`'s `.ovwatch`). It's keyed by movement pattern, not by
  week/phase, so it doesn't need to vary across `WEEK_A/B/C`.
- The workout-intro screen's "Muscles worked" diagram (`muscleDiagram()`,
  `MUSCLE_MAP`) is also keyed by day index 0–6, separate from `watchMode` — if
  you add an 8th day type or change a day's movement pattern, update both
  `MUSCLE_MAP` and `watchMode` for that index, they won't stay in sync
  automatically.

**Completion tracking:** `S.done[dateKey:dayIdx:exerciseIndex] = true`, where
`dateKey` is local (not UTC) `YYYY-MM-DD`. Weekly/monthly stats are derived from
this, not stored separately — `weeklyStats()`, `monthlyStats()`,
`dayDoneThisWeek()`, `sessionDoneOnDate()`. Tapping a weekly-adherence pip
bulk-toggles every exercise for that date+day (`toggleDaySession()`).

**Carousel:** cards (one per exercise, plus an overview card and a finish card)
swipe via a real damped spring (`springStep`/`springFrame`), not CSS
transitions — deliberately, so a touch mid-animation interrupts cleanly from
the live position instead of fighting a fixed-duration transition. Buttons and
inputs opt out of the swipe gesture entirely on `touchstart` (`onControl` flag)
— a real bug existed early on where jittery taps on buttons got silently
swallowed by the swipe handler; don't remove that guard.

**Animation timing:** `MOTION` table (`explosive` / `flow` / `controlled`) sets
period + hold-fraction per exercise via a `motion` field — poses hold briefly at
each end rather than moving continuously, which is what makes them read as real
reps rather than a metronome.

## Design language

True black (`--ink:#0A0A0A`) / white / one accent, volt (`--signal:#D7FF3D`).
Anton for the wordmark and hero numbers only (no true italic exists for it —
the "bold italic" wordmark is a CSS `skewX()` transform, not a font feature).
Barlow Condensed for headers/buttons/exercise names. Inter for body and small
labels. IBM Plex Mono is reserved for the timer countdown only — don't let it
creep back into general UI labels, that was a deliberate move away from a
"technical dashboard" feel toward something closer to Nike/Apple.

## Known limitations (stated honestly, not hidden)

- No rack confirmed → no true heavy barbell squat in the Strength week; the
  Jayflex bar is used for the deadlift specifically, where it's safe and
  genuinely heavy.
- The 8-bit flexed-bicep icon and favicon are hand-authored pixel art (PIL,
  nearest-neighbor scaling) embedded as base64 data URIs directly in
  `index.html` — regenerate via the same approach if the palette changes.
- Rear-foot-elevated split squat's bench position jumps slightly mid-loop
  (it's a single animated figure interpolating between two poses that each
  carry a different bench rect — cosmetic, not a functional bug).
- No test suite ships with the file. Testing so far has been ad hoc jsdom
  scripts written per-session (spring physics, touch gestures, pose rendering,
  rotation-week logic, localStorage round-trips). Worth turning into a real
  persisted test suite now that this has a proper home — that's the main thing
  chat couldn't give this project.

## Suggested first task in Code

Set up git, commit `index.html` as-is, then port the ad hoc test patterns above
into a real test file so they persist across sessions instead of being
rebuilt from memory every time.
