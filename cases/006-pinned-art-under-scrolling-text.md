# 006 — Pinned art under scrolling text

- **Project / files:** frankie-tarot — `RitualCard.tsx` (phone detail view)
- **Stack:** React 19, motion 12, Tailwind 4
- **Date:** 2026-10-07

## Symptom
In the phone detail view, scrolling the text made the card rise away with it
(plus a −72 px scroll-driven drift). Wanted: the card stays in place and
fades and blurs under the text.

## Conditions
Phone detail layout: art and text in one vertical scroller.

## Measurement
After the change, scrolling the text through its full height kept the art's
block at top 0; the image itself moved ~13 px (from scaling to 0.94).

## Cause
The art lived in a normal full-height block at the top of the scroller, so it
scrolled out; the drift was a scroll-driven `y` on the art.

## Didn't work
—

## Fix
```tsx
<div className="absolute inset-0 overflow-y-auto" onScroll={trackProgress}>
  <div className="sticky top-0 h-[100dvh]">
    <motion.div style={{ scale /* 1→0.94 */, opacity /* 1→0.16 */, filter /* blur 0→18px */ }}>
      {art}
    </motion.div>
  </div>
  <div className="relative -mt-16 bg-linear-to-b from-transparent to-black">{text}</div>
</div>
```
The scroll-driven effects sit on a wrapper, not on the layout-animated art
(see 001).

## Rule
To keep an object present while content scrolls over it, pin its block with
`sticky` inside the scroller and drive fade/blur/recede on a wrapper; don't
translate it with the scroll.

## Responsive detail as foreground and background (2026-10-17)

The user explicitly endorsed this design direction: opening a card should adapt
naturally to phone and desktop, with smooth transitions and depth expressing
hierarchy rather than a stack of visibly nested panels. Source reference:
`frankie-tarot/src/features/tarot/components/RitualCard.tsx`, reviewed at
`9b2530b`. This addition records design intent and source mechanisms; the earlier
measurement above is historical, not a new device verification.

### One object, two reading arrangements

- **Desktop:** use the available width for an artwork region beside a readable
  text region. Keep the artwork visible while long text scrolls independently.
  The reference uses a 48%/52% split; choose proportions from the actual content,
  not a universal ratio. Reuse the same card renderer and content semantics.
- **Phone:** initially give the card the visual center. Use one vertical reading
  scroller with a pinned artwork block. As text moves into the foreground, let
  the artwork recede in place through a slight scale reduction, fade and optional
  blur. Text enters over a transparent-to-opaque reading gradient so it remains
  legible without a bordered modal containing another card-shaped text panel.
- **Both:** preserve the collection slot, selected identity and a clear return
  control. Fit the composition to usable width AND height, including host/device
  insets. Adapt arrangement without duplicating the object's state or controls.

### Depth is a visual hierarchy, not a ban on DOM wrappers

Use a small explicit layer order: retained collection/world → selected artwork
→ reading text → navigation and object controls. Use `z-index` to establish
occlusion, and opacity/scale/contrast to communicate attention. Do not attempt to
animate `z-index` as if it interpolated depth; establish stacking at the handoff
and animate the visible properties. Inspect stacking contexts: a child's large
z-index cannot escape an ancestor's context.

The phone implementation DOES have structural wrappers for scrolling, sticky
positioning and isolated motion. Those wrappers are visually transparent and
become `display: contents` when inactive. “No nested containers” here means no
repeated visible shells (panel background + border + padding inside another
panel), not removing wrappers needed to preserve identity or measure motion.
Use spacing, type and foreground/background contrast for content hierarchy.

Keep one deliberate reading scroll owner on phones. Decorative layers must not
intercept gestures; interactive artwork/comparison controls remain reachable.
If detail is modal, give it appropriate focus/return behavior and prevent the
covered collection from receiving interaction. Depth alone is not accessibility.

### Make the whole transition continuous

1. **Open:** move the retained artwork from its measured collection slot to the
   detail position. Recede the collection and introduce text in coordination
   with that travel; avoid simultaneously showing duplicate prominent cards.
2. **Read:** derive recession from a bounded scroll progress value. The reference
   maps progress to scale 1→0.94, opacity 1→0.16 and blur 0→18 px. These are local
   art-direction values, not required defaults. Keep scroll effects on a wrapper
   separate from the layout-moving artwork; prevent tilt from competing with
   precise comparison gestures.
3. **Close or reverse:** return from the current visible presentation to the
   retained slot. Unwind scroll recession together with travel; do not reset it
   before frame zero and produce a flash. Keep required layers alive until the
   return completes, and let interruptions retarget current motion rather than
   restart from a stale endpoint.
4. **Resize:** preserve selection and semantics while rearranging. Prefer a
   stable artwork tree path across the breakpoint. The reviewed implementation
   branches between phone and desktop wrappers, so cross-breakpoint DOM retention
   is NOT established by this example.

Use the product's shared easing with coordinated durations; the reference's
layout tween is 0.45 s. Smoothness depends on measured endpoints, frame pacing and
interruptibility, not a particular duration. Profile blur/compositing cost on the
actual target; reduce expensive effects while retaining spatial continuity.
Reduced motion keeps identity, reading order and destination while removing
unnecessary travel, tilt and depth effects.

### Acceptance for this pattern

Check open → scroll → close on both sizes, including a scrolled collection,
long text, a short viewport and covered screen edges. Inspect the first close
frame after deep scrolling, intermediate occlusion/contrast, rapid reversal,
breakpoint crossing, touch scrolling and focus return. Confirm there is no
visible panel-within-panel hierarchy on phone, no duplicate active artwork,
and no jump from resetting fade/blur before return. Source inspection alone
does not establish these runtime results; the current source's close reset is
specifically a point to verify before reusing it as a smooth-return reference.
