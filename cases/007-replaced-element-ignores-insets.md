# 007 — Replaced element ignores insets

- **Project / files:** frankie-tarot — `AuraHeroCard.tsx` (glow canvas)
- **Stack:** React 19, Tailwind 4
- **Date:** 2026-10-06

## Symptom
An orange haze spilled to the right of the hero card and into the water
below it.

## Conditions
Any size; the glow was a blurred `<canvas className="absolute inset-[10%]">`.

## Measurement
The canvas measured 300×150 CSS px regardless of the card's size.

## Cause
`canvas`, `img`, `video` and `iframe` are replaced elements: with `width`/
`height` auto they keep their intrinsic size (canvas: 300×150) instead of
stretching between `left`/`right`/`top`/`bottom` insets.

## Didn't work
—

## Fix
`className="absolute left-[10%] top-[10%] h-[80%] w-[80%]"` — explicit size.

## Rule
Give replaced elements an explicit width and height; insets alone don't size
them.
