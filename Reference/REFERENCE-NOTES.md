# Reference notes — Nike Training Club / Fitbod / Hevy patterns

Source: screenshots in this folder. Priority order per brief: NTC > Fitbod > Hevy.
Goal: pull concrete UI rules for PiperPump's exercise card, not a mood board.

## The one rule that shows up everywhere

**During an active set, exactly one exercise is on screen, and within that
exercise exactly one number pairing dominates: current weight × current reps.**
Everything else on the card is either gone, shrunk to a label, or pushed below
the fold. None of these apps show a full history chart, a muscle diagram, a
plan overview, AND a timer AND a complete-set button all at once mid-set. That
stacking is exactly what PiperPump's card currently does — it's the thing to cut.

## What's visible during an active set

- **Exercise name** — one line, medium weight, not competing for attention.
- **Weight × reps** — this is the hero. Fitbod renders the working number
  ("110 lbs", "125 lb") at roughly the same scale PiperPump already reserves
  for its Anton hero stat (~40-60px, tabular numerals). Reps sit next to or
  under it, same treatment but visually secondary (smaller weight/size, same
  family) — never a caption-sized number.
  - Concrete for PiperPump: promote the working weight to hero-number size
    (the size currently used for NOW/TARGET on the home screen), reps next to
    it at ~60% that size, everything else on the card drops to label size.
- **Set indicator** — a row of small pips/dots or a compact "Set 2 of 4" line,
  not a full table. Fitbod uses a single collapsed line per prior set
  ("10 x 70 lbs") only in the *list* view, not the active-set view.
- **A single primary action** — log/complete this set. One button, thumb reach,
  bottom of card. Nothing else competes with it for tap priority.

## What disappears or gets tucked away

- **Exercise illustration/photo** — shrinks to a small thumbnail (Fitbod: ~48px
  square) or disappears entirely once a set is in progress. It's identification,
  not the focus — PiperPump's animated figure should shrink/dock rather than
  stay full-width once a set starts.
- **Muscle diagrams, recovery %, "target muscles" grids** — these live on a
  separate pre-workout/overview screen (Fitbod's heatmap, NTC's "Target
  muscles" tile row). They never appear on the same screen as an active set.
- **Program/plan chrome** — day name, week focus, session note, gear list: all
  overview-screen content. Not present during the set-by-set flow.
- **Top nav / tab bar** — NTC and Fitbod both suppress or minimize the primary
  navigation once you're inside a workout; only a back/close affordance and
  the persistent HUD remain.
- **Cue/coaching text** — when present, it's one short line, collapsed by
  default, expandable — not a paragraph sitting permanently on the card.

## Rest timer behavior — small and persistent, never a takeover

This is the clearest, most consistent pattern across all three apps:

- **NTC**: the timer/heart-rate/calorie readout is a small fixed HUD in one
  corner of the screen (top-left, ~120px wide), overlaid on whatever's behind
  it. It never becomes a full-screen countdown.
- **Fitbod/Hevy-style flow** (the "Jesse's Plan" reference): rest/transport
  controls (prev/pause/next) sit in a slim bar pinned to the bottom of the
  screen, well below the exercise content, with a thin linear progress
  indicator rather than a giant numeral.
- **Generic template reference (images.jpg)**: the clearest 1:1 match for
  PiperPump — a compact bottom-docked card showing the countdown ("00:35"),
  a thin radial/linear progress ring, Prev/Pause/Next, and total-session time,
  all inside a fixed-height bar that never exceeds roughly the bottom 15% of
  the screen. The rest of the screen keeps showing the *next* exercise info.

**Concrete rule for PiperPump:** the rest timer should be a persistent bottom
dock (this already exists structurally as `.dock`/timer controls — reduce its
visual weight, not its position). It should never occupy the full card or
replace the exercise content; it should look like the small bottom-docked
countdown in images.jpg, with the countdown numeral roughly IBM Plex Mono at
current size or slightly smaller, not enlarged into a hero number.

## Size hierarchy, concretely (largest to smallest)

1. Weight number (hero, Anton-scale)
2. Reps number (secondary hero, ~55-65% of weight size)
3. Exercise name (Barlow Condensed, header scale)
4. Set indicator / pips
5. Rest-timer countdown (small, persistent, bottom-docked)
6. Cue text, gear tags, plan chrome (label scale, collapsed/hidden by default)

## What to cut from PiperPump's current exercise card

Based on the brief's own description of the current card (hero stat + chips +
cue text + side plate + sometimes front plate + timer buttons + complete bar,
all stacked at once):

- **Don't show side + front plate simultaneously.** None of the reference apps
  show two illustration angles at once during a set. Pick one (side, since
  that's PiperPump's primary rig) and only surface the front view as a
  tap-to-expand, not default-visible.
- **Shrink the illustration once a set is active** — full-width only on the
  overview/rest state, docks small once counting reps.
- **Cue text collapses by default** — one line, tap to expand, matching the
  "short line, expandable" pattern above.
- **Chips (gear, tags) belong on the overview card, not the active-set card.**
- **Complete-bar and timer controls shouldn't coexist as two separate button
  rows** — consolidate into the single bottom dock described above, matching
  the images.jpg reference (progress + transport controls in one compact bar).

## Imagery note

If the redesign needs new in-app iconography (e.g. small pictogram icons for
gear/equipment tags, replacing text-only chips), the webp reference in this
folder shows the pictogram style worth matching: flat, single-color (volt or
white), simple silhouette icons — consistent with PiperPump's existing volt/
black/white palette. Source free stock icons in that style rather than
photographic imagery; photographic/video content is NTC's language, not ours
(we're a tracker, not a workout-video app).
