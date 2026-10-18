# 005 — Edited the branch that doesn't run

- **Project / files:** frankie-tarot — `RitualCardStage.tsx`
- **Stack:** React 19, Tailwind 4
- **Date:** 2026-10-07

## Symptom
Two edits meant to shrink the card on phones (a shorter absolute-layout stage,
and a `maxWidth` on the flex-layout card) changed nothing on the phone.

## Conditions
The component chooses between three layouts in JS: absolute (scaled),
flex-wrap, and a compact grid — and uses the compact grid whenever the stage
is narrower than 960 px, i.e. on every phone.

## Measurement
Reading the live element first would have shown it: the wrapper's classes had
no width class and an inline `width: 208px` — the grid branch's
`gridCardWidth`, computed in JS.

## Cause
Both edits targeted branches that never render at that breakpoint.

## Didn't work
Exactly the two edits above.

## Fix
Edit the branch that runs (cap `gridCardWidth` by the available height) and
revert the dead edits.

## Rule
Before changing responsive code, inspect the live element at that breakpoint
(classes, inline styles, which branch produced them). After changing it,
measure; revert edits to branches that don't run.
