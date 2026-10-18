# 001 — Remount breaks shared layout

- **Project / files:** frankie-tarot — `src/features/tarot/components/RitualCard.tsx` (used by `RitualCardStage.tsx`, `DeckLibrary.tsx`)
- **Stack:** React 19, motion (Framer Motion) 12, Tailwind 4
- **Date:** 2026-10-07

## Symptom
Opening a card into its detail view and closing it, the card didn't fly back
from where it was shown: on phones it first appeared far below, then flew to
its slot. "More obvious on mobile, when we need scroll."

## Conditions
- Phone layout only; desktop was correct.
- Grew with the page's scroll; absent with the page at the top.
- Scrolling inside the detail view did **not** matter.
- Library grid and spread both affected (same component).

## Measurement
`visible` vs `firstFrame` on close (phone, 375×812):
- page scrolled 530 px → visible y 175, firstFrame y 635 (+460);
- page scrolled 300 px → visible y 175, firstFrame y 435 (+260);
  i.e. ≈ 0.87 × page scroll.
- Desktop, page scrolled 450 px → firstFrame = visible (82,103 vs 84,108).
After closing, the node held before closing was gone from the DOM → remount.

## Cause
The art element (`layout` + `layoutId`) was rendered at one place in the tree
when closed and at another when open on phones (inside the detail scroller):
`{!isDetailed && art}` vs `<Scroller>{!isDesktopDetail && art}</Scroller>`.
React remounted it on every toggle, so Framer had to hand the animation over
between two elements via `layoutId`, and that handover measured the old copy
offset by the page scroll (the page scroller is a `layoutScroll` container; the
detail is a `position: fixed` overlay inside it). On desktop the art was at
one tree position in both modes, so the same element animated by plain
`layout` and was correct.

## Didn't work
- `layoutRoot` on the fixed overlay — firstFrame unchanged.
- The detail scroller as a `motion.div` with `layoutScroll` + `layoutRoot` —
  unchanged.
- Removing `position: sticky` from the art's block — unchanged.
- Moving the scroll-driven `scale`/`opacity`/`filter` off the art onto a
  wrapper — unchanged (kept anyway: principle 3).
- `overflow: visible` on the detail scroller, live — unchanged.
All of these assume the element survives; it didn't.

## Fix
Render the art once, at a fixed tree position, for phones too; the phone-only
wrappers are `display: contents` when closed:

```tsx
{!isDesktopDetail ? (
  <div className={isDetailed ? "absolute inset-0 overflow-y-auto" : "contents"}>
    <div className={isDetailed ? "sticky top-0 h-[100dvh]" : "contents"}>
      <motion.div className={isDetailed ? "flex h-full items-center justify-center" : "contents"}
        style={isDetailed ? { scale, opacity, filter } : undefined}>
        {artwork}
      </motion.div>
    </div>
    {isDetailed && detailText}
  </div>
) : (<>{isDetailed && <aside>{detailText}</aside>}{artwork}</>)}
```
After: firstFrame = visible on open and close, after inner scroll, page
scrolled, both layouts; settled = slot.

## Rule
An object that moves between layouts must be rendered once at one tree
position in every arrangement; change the wrappers around it (with
`display: contents`), not its place. If `layoutScroll`/`layoutRoot` change
nothing, look for a remount.
