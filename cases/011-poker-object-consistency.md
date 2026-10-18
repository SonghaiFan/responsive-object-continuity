# 011 — Poker object consistency: identity, journey and interaction

- **Project:** `frankie-hold-em`, revision `78b276e`.
- **Date:** 2026-10-17.
- **Evidence:** source review of a clean checkout. No runtime geometry, device QA,
  paid model calls or tournament runs. This is a positive design case, not a bug fix.
- **Scope:** Frank and Offsuit are distinct presentation branches. Findings below
  name their scope; switching styles is not demonstrated to preserve DOM identity.

## 1. Retain the workspace; change the arrangement

`components/frank/TableLayout.tsx` renders the table, roster and hand log once.
React breakpoint state selects compact (<1024), desktop (1024–1439) or wide
(>=1440). `index.css` arranges those same children as a compact information
sheet, a tabbed side panel, or two flanking panels. In wide mode the information
wrapper becomes `display: contents`. The active tab and open state live above
these arrangements. Hidden compact information is inert and aria-hidden;
position layout motion has zero duration under reduced motion.

**UX value:** the user keeps the same table and information objects while the
screen exposes more context. Components need not reconstruct their local state.
Retention is source-confirmed; scroll/focus preservation and movement through
`display: none` still need browser testing. Mount survival alone does not prove
that every hidden-to-visible transition animates.

The Offsuit branch of `components/PokerGame.tsx` offers a related spatial idea:
a central column (opponents → board → player) stays at most 480px wide while
large screens add roster/history alongside it. This is a stable task arrangement,
not evidence that Offsuit has Frank's compact information-sheet behavior.

## 2. Give a collection one reversible journey

`components/SeatSnake.tsx` retains avatar spans keyed by `row.id` for both the
collapsed stack and expanded list. A scalar `travel` places each avatar on the
same bent track: horizontally toward the corner, then vertically down its row.
The leading avatar travels furthest; followers stop at their own destinations.
There is no separate expanded avatar handed off by a cross-fade.

- `travelRef` stores current progress; a toggle stops the old animation and starts
  from that value. This supports position continuity during reversal; it does
  not establish velocity continuity, because easing restarts.
- Each avatar's normalized journey drives padding, background and border shadow.
  Shape changes reverse with movement rather than jumping with the boolean mode.
- Container height follows the deepest current avatar, keeping surrounding layout
  coupled to the collection instead of immediately reserving final height.
- Row text/actions become available only when `arrived(j)` is true. Before that,
  pointer events are disabled, tabIndex is -1 and aria-hidden is true.
- Removed seats leave a short noninteractive fade at their last sampled position;
  surviving IDs remain mounted and close the gap. This exit ghost is a deliberate
  decorative copy, not proof that the removed node itself survives.
- `isolate` keeps avatar stacking order inside the collection. Reduced motion
  goes directly to the destination and skips removal ghosts.

**Rule:** coordinate position, appearance, surrounding space and interaction
readiness. Sharing a duration alone does not give objects a shared journey.
Insertion/removal during motion requires separate validation: changing the number
of rows changes track endpoints and follows a different code path from toggling.

## 3. Reveal information inside persistent slots

Offsuit `components/TableStratum.tsx` always renders five community-card slots,
keyed by position. Game phase changes which cards show their front, rather than
inserting new layout boxes and pushing existing cards sideways. These keys name
board slots, not persistent card identities across hands. Hole cards in
`components/PlayerStratum.tsx` instead use `card.id`.

`components/PlayingCard.tsx` separates outer geometry, deal animation, face flip,
and hover/winning transforms into nested layers. Both faces remain in the card.
Its em-based dimensions, corner radius and typography scale together; slot
containers supply `10cqw` font sizing. Winning cards lift in their existing places.
Nonwinning fronts receive a dark overlay rather than whole-card transparency,
so overlapping cards retain opaque physical surfaces.

**Rule:** first decide whether the stable object is a slot or an entity. Reveal,
selection and emphasis should update that object's state without disturbing the
spatial arrangement. Separate transform ownership by layer.

## 4. Make direct manipulation explain the state change

Offsuit `components/PlayerStratum.tsx` lets the player drag their actual card group
upward to fold. Pointer capture maintains the drag beyond the card boundary;
a below-threshold release returns the group, and pointer cancellation clears the
drag. Crossing the threshold animates departure before invoking the fold action
(after 220ms). An outline/glyph trace remains in the reserved card region.

**UX value:** the manipulated object explains what the action did and where it
came from; removing it need not collapse its place in the layout.

**Boundary:** the trace is a separate decorative representation. Do not generalize
the magic timeout as an animation-completion protocol. This review did not verify
a keyboard-equivalent fold action, reduced-motion behavior for this gesture, or
state changes during its delayed commit; these are checks, not proven strengths.

## Evidence boundaries and reuse checks

`components/OpponentSheet.tsx` shares section definitions and seat data across
phone/desktop, and portals the overlay outside the transformed lobby. That is
useful semantic reuse and correct overlay placement, but its conditional trees
reparent sections; do not cite it as retained editor DOM or caret continuity.
Likewise, shared state across Frank/Offsuit does not prove retained view objects.

Before claiming visual continuity in a reuse:

1. Hold actual node references and compare identity across arrangements; inspect
   scroll position, selection and focus as well as React state.
2. Test 1023/1024 and 1439/1440 transitions both ways, with information open and
   closed and each tab selected. Check short screens and covered edges.
3. Reverse SeatSnake at intermediate progress; add/remove seats both settled and
   in motion. Inspect path, stacking, appearance, height and action availability.
4. Walk preflop → flop → turn → river → showdown → next hand; distinguish slot
   persistence from fresh card identity and check for stale visible faces.
5. Exercise fold cancellation, threshold release, rapid updates, keyboard access
   and reduced motion. Confirm traces cannot receive input or expose stale actions.

No numerical frame measurements or perceptual smoothness claims were established
in this review. Preserve these evidence boundaries when citing the case.
