# 015 — Blog card-view carousel: one deck, one continuous position

- **Project:** `knowledge-vault`
- **Source:** `_system/site/vitepress/docs/.vitepress/theme/CardDeck.vue`, especially `place`, `paint`, `glide`, `move`, `startDrag`, `updateDrag`, `finishDrag`, and `onDeckWheel`.
- **Source revision:** `90c65c09dca366c6be57ddf9cc6487fefddf8114`
- **Date:** 2026-10-17
- **Evidence:** user explicitly endorsed the blog card-view carousel as a good UX case. Mechanisms reviewed in source; no new browser measurements or device acceptance performed.
- **Use for:** stacked-card readers, interruptible carousels, looping galleries, and several input methods controlling one spatial object.

## What feels consistent

The reader handles a stack of sheets. Dragging pulls the top sheet away and
reveals the next; reversing brings a sheet back onto the stack. Release continues
from the live position, and grabbing mid-flight takes control there. Neither a
new input method nor crossing the last page creates a new spatial story.

### One position for all inputs

`CardDeck.vue` has one continuous, unwrapped `pos`, a destination `target`, and
one `place(n - pos)` projection. Drag writes `pos`; release, buttons and keys
use `glide()`. A new glide captures the current `pos`, while `startDrag()` cancels
the glide and records that same live position. Repeated buttons advance from
`target`, preserving queued intent. `paint()` writes transforms directly and
updates reactive slot membership only when the integer base changes.

**Transfer:** separate current position from destination; every input controls
the same motion state. Interruption must start where the object is drawn.
This code preserves position continuity, but restarting a quart-out easing does
not prove velocity continuity; preserve velocity too if the interaction needs it.

### Loop content, not spatial coordinates

Four slots cover leaving/top/under/next-under positions, keyed by unwrapped
`slot.n`; only their page lookup uses modulo. Crossing the last page therefore
does not reset the travel coordinate or send the sheet around the long way.
Retained slots survive while relevant; this is a bounded window, not a promise
that every page's DOM stays mounted forever.

**Transfer:** distinguish content ID from occurrence/slot ID in cyclic views.
Wrap the data lookup, keep the travel coordinate continuous, and maintain enough
neighbour slots to cover intermediate frames in both directions.

### Occlusion makes paper legible

`place()` uses position-derived paint order, a small drop and scale for the
sheet underneath. The outgoing sheet remains opaque until the final 8% of
travel, avoiding an extended dissolve of two text pages. These constants are
project tuning, not universal clearance guarantees: verify actual overlap.

**Transfer:** use depth and occlusion to communicate stacks; fade only when the
surface has sufficiently cleared readable content underneath.

### Follow the finger, then settle with bounded intent

`updateDrag()` converts pointer distance into card steps using the same card
width and travel factor as `place()`. Within the dragged top-sheet segment this
makes its horizontal motion one-to-one with the pointer. Pointer capture keeps
the gesture attached when the pointer leaves the original hit area.

`finishDrag()` uses a smoothed velocity estimate and a 300 ms look-ahead, clamps
the carry, and settles toward a nearby integer position. Cancellation removes
the velocity carry. A drag suppresses the following click, preventing a gesture
from accidentally triggering a second action. The deck loops, so it does not
invent end-of-list rubber-band resistance.

### Different inputs express the same intent

Buttons and arrows call `move(delta)`, which advances from the existing target;
rapid clicks therefore accumulate rather than repeatedly choosing the same next
page from a partly completed position. A horizontal trackpad gesture is latched
to one step until a 180 ms idle gap resets it, preventing momentum events from
turning many pages. Editable controls are excluded from global key handling.
These input adapters share motion state, but need not share raw gesture rules.

## Minimal transferable model

```text
pos = current continuous position
target = intended destination
slot occurrence n -> content modulo page count
render slot n using place(n - pos)

pointer down: stop animation; remember live pos
pointer move: pos = startPos - dx / travelWidth; paint
pointer up: glide from live pos toward bounded velocity-projected integer
next/previous: glide from live pos toward target + delta
```

Keep `pos`, `target`, slot occurrence and content identity distinct. Do not run
one CSS transform for dragging and a competing transition for snapping. Retain
the slots that remain in the visible window and pre-render enough neighbours
for the departure and reveal. Do not key loop slots only by wrapped page index.

## Tuning and limits

The source uses a 560 ms quart-out settle, 1.04 card-width departure, 28 px lower
sheet offset and 0.95 lower-sheet scale. These are reference values to tune to
the product, not universal requirements. Restarting quart-out from the live
position preserves position, but does not guarantee velocity continuity.
The four-slot window does not retain every page DOM forever. Text-to-card mode
switching is separate from this carousel and is not a same-node morph.

Reduced motion jumps to the destination. Direct transform painting keeps motion
independent of full component rerenders; it is an implementation mechanism, not
a measured frame-rate claim.

## Acceptance when reusing

- Drag slowly, flick, reverse, cancel, and grab during settle: no position reset.
- Press next several times during settle: each press contributes to the target.
- Loop forward and backward across the seam: no long-way travel, visible remount
  flash, premature disappearance or z-order swap.
- Inspect intermediate overlap: readable sheets should not dissolve through one
  another. Test the actual card proportions rather than trusting fade constants.
- Test trackpad inertia, keyboard, reduced motion, resize and short viewports.
- Record visible / first-frame / settled positions and compare the reference
  interaction before claiming an equivalent implementation.

## Rule

A carousel is one continuously manipulated arrangement, not a succession of
slide-change animations. All inputs change one position; loop only the content
lookup; let retained neighbouring sheets and occlusion explain the transition.
