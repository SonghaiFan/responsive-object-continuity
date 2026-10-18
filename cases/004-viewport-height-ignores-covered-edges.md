# 004 — Viewport height ignores covered edges

- **Project / files:** frankie-tarot — `RitualCardStage.tsx` (stage min-height, card width), `IntroSection.tsx` (home layout)
- **Stack:** React 19, Tailwind 4
- **Date:** 2026-10-07

## Symptom
With the host's bar and message box covering the phone screen, the
"reveal every card" prompt sat under the message box; on the home page the
tagline ran into the card and the signature into the start buttons.

## Conditions
Phone layout with covered edges (host chrome or large notches). Fine with
nothing covered.

## Measurement
375×812, top 112 px and bottom 120 px covered: card stage
`min-height: calc(100dvh - 14rem)` pushed the prompt to y 750 (covered from
692). After the fix: header ends 148, card 160–516, prompt 582. With nothing
covered, the layout was identical to before (prompt 660).

## Cause
Heights and positions were computed from the full viewport (`100dvh`, `%` of
the screen), not from the part that's visible. Separately, a single card's
width came from data (208 px → 357 px tall at 7:12), which can't fit a short,
covered screen.

## Didn't work
Shortening a container that only applied on desktop (see 005).

## Fix
```ts
minHeight: "calc(100dvh - var(--safe-top) - var(--safe-bottom) - 14rem)"
// width capped by the height left, via the real aspect ratio:
width: `min(${widthFromData}px, calc((100dvh - var(--safe-top) - var(--safe-bottom) - 14rem) / ${ASPECT_H_OVER_W}))`
```
For a full-screen hero, place blocks relative to the visible band, e.g. centre
the card between the title's bottom and the horizon, and the action row
between the horizon and the footer above the bottom inset.

## Rule
Every viewport-height calculation subtracts the safe areas; anything sized by
width is also capped by the height actually available. Check with nothing
covered too, so the fix is a no-op there.
