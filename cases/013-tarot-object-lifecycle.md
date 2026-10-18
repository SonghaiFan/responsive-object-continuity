# 013 — Tarot objects across a complete interaction lifecycle

- **Project:** frankie-tarot, source review at `9b2530b`, 2026-10-17.
- **Evidence:** implementation inspection, not a new browser/device verification or bug fix. Cases 001–008 contain earlier investigations; their measurements are not fresh acceptance evidence for this revision.
- **Read when:** an object moves from a collection into a workspace and detail view, or rendering optimizations threaten its identity, return position or input behavior.

## What works well, and why

### Keep the place as well as the object

`DeckLibrary.tsx` keeps a keyed `LibraryCardSlot` with the card aspect ratio while its `RitualCard` becomes fixed for detail. The original grid cell remains occupied. Other cards dim and blur rather than disappearing, so the user retains the surrounding collection and a meaningful return address.

Within the phone arrangement, `RitualCard.tsx` retains the artwork's tree position through open/close; scroll/sticky wrappers become `display: contents` when closed. The desktop arrangement also reuses its artwork position through open/close. Library previews and spread cards use this same component, including its comparison controls, rather than a separately maintained detail imitation.

**Transfer:** preserve the source slot and native object behavior when expanding. A matching visual alone does not preserve the return context. See case 001 for the remount investigation.

### Separate visual instance identity from content identity

`localDraw.ts` assigns a `visualId` to a dealt tile. `PickingSection.tsx` uses `card-${tile.id}` for its shared-layout source; `RitualCardStage.tsx` uses `card-${card.visualId ?? card.id}` for the destination. Library objects use the separate `library-card-${card.id}` namespace.

This connects the exact card back the user chose to its destination, even though tile identity and revealed content identity are different. The cloud and selected card are genuinely different components, so this is a deliberate shared-layout handoff, not evidence of a single DOM node across the whole flow.

**Transfer:** distinguish domain ID, visual instance ID and collection namespace. Do not use a new result ID or list index to identify the source of a flight. Scope identities to their active surface/session where collisions are possible.

### Assign different motion responsibilities to different layers

`RitualCard.tsx` assigns layout movement to the artwork shell, mouse tilt to an inner surface, face reveal to a separate rotating layer, and phone scroll recession to an outer wrapper. Its comparison slider disables detail tilt and captures the pointer during dragging. The object remains spatially stable while the user compares its content.

**Transfer:** layout, gesture, reveal and scroll transforms need clear ownership. Suspend competing decorative gestures during precise manipulation. Preserve content semantics independently of presentation: an upright detail view must not change the stored reversed-card meaning.

### Make the action traceable through its consequences

`App.tsx:handleCardSelect` guards repeated selection by tile identity and card count, preloads the chosen image, and waits 600 ms after the final pick before entering reading. The stated intent is to let the last card land before the next arrangement begins. Its delayed callback checks the reading ID to avoid changing a newer session.

`PickingSection.tsx` handles touch selection on release, rejects a drag, and suppresses compatibility clicks that might target a neighboring card after the chosen node disappears. Keyboard activation remains available.

**Transfer:** one gesture should cause one object transition. Sequence dependent changes after arrival; prefer completion signals when available. The 600 ms delay is a local implementation detail, not a reusable timing requirement. Guard deferred work against stale sessions.

### Optimize paint without discarding interaction identity

`DeckLibrary.tsx:LibraryCardSlot` defers first mount until near the viewport, then retains mounted cards while their slots remain in the collection. Selected cards render regardless of proximity. This is reveal-once loading, not continuous unmounting virtualization.

`PickingCloudCanvas.tsx` batches deck painting while DOM elements retain hit testing and shared-layout anchors. GPU readiness is announced after a successful render and health check; errors/context loss restore the DOM card-back path. `PickingSection.tsx` retains those anchors even when GPU painting hides their fallback surfaces.

**Transfer:** separate the cost of drawing from the lifetime of the interactive object. Keep selected or traveling objects alive through return. For very large collections, use bounded retention/pinning rather than copying this small-deck retention strategy without a memory budget. Verify visual and hit-test geometry stay aligned through renderer handoff.

### Preserve spatial relationships within the available space

`RitualCardStage.tsx` measures the stage, centers the spatial spread from its bounds, and scales it uniformly against both available width and height. Width determines whether the spatial arrangement fits; a short desktop does not automatically become a phone layout. Compact rows retain the original card indexes for labels and positions, and balance row counts.

`index.css` combines device and host insets; single-row phone cards also cap their width by remaining height. `scene/camera.ts` keeps picking and reading at the same world framing, adjusting the veil instead of replacing the sky.

**Transfer:** preserve relationships and semantic order when possible; choose a simpler arrangement when geometry requires it. Measure the actual container and usable height. See cases 003, 004 and 008 for safe-area and world/camera details already covered by this skill.

## Boundaries: do not turn source patterns into blanket guarantees

- `RitualCard` branches on `isDesktopDetail`; crossing that breakpoint changes the artwork's tree path. Open/close retention within one arrangement does not prove cross-breakpoint retention.
- `RitualCardStage` switches between direct children and row wrappers. Crossing that boundary, or moving cards between rows, can remount cards despite stable keys.
- The detail comparison branch replaces image descendants even while retaining the artwork shell. Shell identity and pixel/editor identity are separate claims.
- Phone scroll effects reset on close; reversed artwork orientation intentionally snaps. Whether those transitions look continuous needs intermediate-frame inspection, not inference from `layout` props.
- A CSS grid breakpoint is not automatically animated merely because a descendant uses layout animation.
- Reduced-motion handling exists for tilt, scroll effects and camera, but this review does not establish that every flight/reveal honors it.

## Acceptance checks for reuse

1. Track the source slot, visual instance, artwork node and domain state separately.
2. Open/close after page and detail scrolling; compare first-frame and settled geometry, not just screenshots at rest.
3. Cross phone/desktop and row-count boundaries while selected; explicitly check node identity.
4. Select the last item, double-tap, drag then release, and reset during a pending transition; confirm no duplicate action or stale stage change.
5. Scroll away/back, exercise renderer fallback, and confirm the same target, content, hit area and return slot.
6. Check short/covered viewports and reduced motion. Report source inspection, browser emulation and real-device evidence separately.

These are validation tasks for an implementation adopting the patterns, not tests performed by this source review.
