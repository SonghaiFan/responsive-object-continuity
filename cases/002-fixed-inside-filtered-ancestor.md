# 002 — Fixed inside a filtered ancestor

- **Project / files:** frankie-tarot — `src/app/App.tsx` (header wrapper), `src/app/components/HeaderBar.tsx`
- **Stack:** React 19, motion 12, Tailwind 4
- **Date:** 2026-10-07

## Symptom
Moving the header to a bottom dock on phones (`fixed bottom-0`), it would have
landed at the top of the screen.

## Conditions
Any `position: fixed` child anchored to the bottom (or sized to the viewport)
inside the header's wrapper.

## Measurement
The wrapper was a `motion.div` animating `filter: blur(...)` (to blur the
header when a card is open), `position: relative`, zero height at the top of
the page.

## Cause
An element with `transform`, `filter`, `backdrop-filter`, `perspective`,
`will-change: transform` or `contain: paint` becomes the containing block for
`position: fixed` descendants. The header's "viewport" was the zero-height
wrapper, so `top-0` happened to work and `bottom-0` would not.

## Didn't work
Nothing tried; caught by reading the wrapper before the change shipped.

## Fix
Make the wrapper fill the viewport so the containing block is the viewport
again, and keep it from catching clicks:

```tsx
<motion.div animate={{ filter }} className="fixed inset-0 z-[60] pointer-events-none">
  <HeaderBar /> {/* fixed top-0 / bottom-0 now measure against the screen */}
</motion.div>
```
(Its children opt back in with `pointer-events-auto`.)

## Rule
Before positioning something `fixed`, check its ancestors for transforms and
filters (animation libraries add them). Such a wrapper must fill the viewport,
or the fixed element must live outside it.
