# 017 — Syuze: material behavior and object continuity

## Evidence and scope

Source review on 2026-10-17, requested by the user as a positive UX reference.
Syuze (叙仄): `syuze`, clean revision
`eac9e39c47c913ac389fc84b5717b678bc38f4e5`. Paths below are relative to it.
Franklin Board legacy checkout: `franklin-board`,
`d10e4908e72ad2ecfe4539f56c5c492e0193ba6f`, with existing uncommitted filing changes.
React refactor: `franklin-board-react-library`, same base revision,
with uncommitted library implementation. Read-only review of all three.

This is a collection of source-backed patterns, not a newly reproduced bug or
fresh browser acceptance. Seven existing tests passed in this review:
`side-folder.test.mjs`, `folder-fan.test.mjs`, `native-motion.test.mjs` under
`scripts/spatial-tests/`. They verify geometry and spring properties, not perceived
smoothness. `docs/spatial-motion-parity.md` records an earlier 53-assertion browser
comparison; it is historical evidence and does not certify this newer revision.

## 1. Keep the object recognizable, including its native controls

`src/spatial/document-card.js` mounts the title, content and editor together;
`document-view.js` presents the retained content instead of drawing a summary
and replacing it with an unrelated reader. A card is a view onto the document.

`document-source-fan.js` calls the shared `createSource` factory for references.
Images remain images, voices retain their playback controls, and source documents
retain their paper presentation. `asset-preview.js` routes the selected reference
to the native viewer/sheet. Its fallback physically moves the chosen node into a
dialog, leaves a placeholder, measures the real return anchor and keeps the card
in the top layer until arrival. Focus returns to that card afterward.

**Reuse:** distinguish three promises: canonical content identity, shared renderer,
and retained instance identity. Canvas and source references reuse a renderer;
they are not globally one DOM node. A chosen reference retains its own instance
through its preview. The fan retains nodes while toggling, but `update(refs)`
rebuilds them when its serialized input changes. Do not cite that as incremental
reconciliation or uninterrupted playback across every data update.

For Franklin's document geometry, editor selection and proportionally attached
image captions, use [010](010-retained-document-and-image-presentation.md).

## 2. The container's construction determines its motion

`folder-appearance.js` gives Assets, Development and Editing different silhouettes
without forking folder identity. Editing sets a side hinge; `sideFolderLayout`
turns the fan's coordinate system while preserving each card's dimensions.
`sideFolderPocket` places resting content at the corresponding side opening.
`folder-container.js` keeps a stack as one leading tile with its members beneath
it; presentation does not dissolve the collection's membership.

In `mount.js`, filing into Editing gathers the actual cards to the right of the
folder before sinking them into its pocket. Other folders gather above their
opening. The shell, fan, pocket and intake direction therefore tell one story.

**Reuse:** define hinge, opening, resting pocket, travel path and occlusion together.
Changing a folder icon while leaving its intake traveling through the closed side
breaks the physical explanation. Preserve content sizes and group identity while
changing layout. These folder metaphors are appropriate here, not mandatory for
every product.

## 3. Occlusion makes membership and depth legible

`mount.js` uses a gather phase followed by a sink phase. The group waits for the
maximum first-stage duration; every member then enters together. Filing tracks
ownership and checks that the document and folder still exist before continuing.
`syncFolderGroups` composites folder back, pocket cards and shell together before
muting the group. Content disappears *behind its container*, rather than fading
out as if deleted.

Focus uses a shared scrim and nearby objects move aside according to distance.
Closing releases focus chrome immediately while refold motion continues; the
workspace responds to the user's decision without waiting for all springs to rest.

**Reuse:** coordinate membership, layer order, visual completion and interaction
availability as distinct events. Keep returning content alive until it is occluded
or reaches its anchor. Use one focus hierarchy for a composite object; independently
fading every child can make overlapping layers look translucent. Do not copy the
reference's pixel distances, timing estimates or native-coordinate adapters as
universal constants. Franklin's `research/native-folder-archive.md` explicitly
separates recovered evidence from browser adaptations.

## 4. Material qualities come from linked state, not extra animation

`src/components/memoir/MemoirBook.tsx` uses one sampled spring for leaf angle,
shading and cover-related book translation. Shading follows
`sin(abs(angle)) * SHADE`: it grows as a page turns upright and disappears as it
lies flat. Layer order identifies the active leaf. Settling writes the resting
pose before canceling animation fill, avoiding a handoff flash.

Paper and cover boards have different critically damped profiles (`PAPER` and
`BOARD`); direct dragging and settling share angle-based painting, with release
velocity passed into settling. A heavier cover and lighter leaf convey structure
without bouncing every surface. The gather entrance uses representative sheets;
it is visual storytelling, not retained chapter-card DOM from the canvas.

**Reuse:** derive shadows, occlusion, rotation and dependent translations from a
shared physical state. Use a small family of profiles by material/action instead
of one duration for all objects or unrelated timers for every visual effect.
Preserve the existing engine if it already provides this coordination.
Do not generalize this book into a proven fully interruptible control: busy leaves
reject another turn, and responsive repagination does not establish a stable
paragraph anchor across widths. Short viewports and host-covered edges also need
separate acceptance; `measureLayout` alone does not establish safe-area support.

## 5. Synchronization must not seize presentation ownership

`mount.js` gates board adoption on gestures, local edits/saves, filing, previews,
open dialogs, camera frames, lids and finite animations. It coalesces pending
notifications, then reads current data when idle. Identical echoes refresh bridge
bookkeeping without repainting; relevant viewport state survives adoption.
Infinite decorative animations are excluded so they cannot block it forever.

**Reuse:** protect active presentation, coalesce replaceable refresh notifications,
and reconcile the newest model afterward. Do not discard durable operations or
block all data handling indefinitely. In this implementation story refresh is
called separately before board adoption, so the gate is not proof that every
subview update is deferred. Test eventual delivery as well as lack of interruption.

`presentation-motion.js` samples current position before a new target and uses a
generation token against stale completion. It restarts with zero velocity: this
supports positional reversal, not a claim of velocity continuity.

## 6. Quiet chrome lets the object explain the interaction

`AGENTS.md`, `src/app/globals.css` and `src/spatial/fabula.css` establish warm
paper/ink tokens, low-chroma controls, shared panels and consistent focus cues.
The content carries differentiation; reading tools, sources and revision history
belong to the document rather than competing permanent panel shells. Density can
change between capture and board without inventing another visual language.

**Reuse:** spend contrast and depth on the active object and usable controls.
Keep tools outside the transformed typesetting when they should stay screen-sized.
Minimal chrome must not remove focus visibility, dismissal or reachable actions.

## Applying this to the Franklin Board refactor

The current React `packages/board/src/react/presentation.ts` projects one typeset
page, retains presentation through return, and scales image caption/note/info in
local coordinates. `core/presentation.ts` carries radius, reveal and scroll with
geometry. These sharpen Syuze's composition continuity: Syuze's older
`document-view.js` still uses fixed caption gaps/sidebar placement, so it is not
the reference for proportional annotation shrink. Preserve this distinction.

When extracting the reusable library, keep object identity, editor state and
presentation ownership in the shared layer; keep stage membership and story
semantics in the product adapter. Avoid recreating the object in React merely to
match its appearance. Reuse case 010's checks instead of duplicating them here.

## Acceptance scenarios for future changes

- Open a source fan, play a voice, preview and return the chosen card; assert
  exact node/control identity, focus, and state appropriate to the asset.
- File a whole stack into both top-opening and side-opening folders; inspect
  intermediate gather/sink frames, membership, occlusion, cancellation and undo.
- Close during opening and after reader scroll; compare the first return frame
  and current landing geometry, not only final bounds.
- Inject sync during drag, filing and camera travel, then let the interaction end;
  check no premature reset and eventual latest-state adoption. Echoes do not repaint.
- Drag and release book pages in both directions; inspect shade/angle alignment,
  cover centering, rapid input, reduced motion and resize while reading.
- Compare the real reference and refactor at the same content, fonts and viewport;
  separately record browser/device observations and numerical assertions.
