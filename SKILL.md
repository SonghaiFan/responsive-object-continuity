---
name: responsive-object-continuity
description: Build responsive UIs where objects keep their identity and place through every layout change — breakpoints (phone ↔ desktop), modes (grid ↔ detail, spread ↔ zoom, list ↔ modal), state changes (reveal, filter, reorder) and embedding hosts (notches, in-app browsers, chat hosts that overlay chrome) — using retained elements and coherent layout or spring animations instead of letting things jump or reappear. Use when designing a layout that differs between screen sizes, adding a transition between two views of the same content, fitting UI between covered screen edges, or fixing a layout animation or responsive layout that jumps, starts from the wrong place, lands under device or host chrome, or only breaks on mobile, after scrolling, or inside an overlay. Also covers native navigation, drag reordering and live data identity. Includes a case library of solved bugs and reviewed patterns — read the relevant case before fixing a similar one, and add a case after solving a new one.
---

# Responsive object continuity

Responsive design changes the layout; it must not change **what the user is
looking at**, and it must not put it **where they can't see it**. An object
the user has noticed — a card, a photo, a tile — has a place. When the layout
changes, it should be seen to travel to its new place; and wherever it lands
must be inside the part of the screen that is actually visible.

This file holds the principles, the method and the checklist. Concrete bugs
live in [`cases/`](cases/) — see the index at the end. Tooling pitfalls that
make these bugs hard to see are in
[`references/tooling.md`](references/tooling.md).

## Principles

**1. Continuity of position.** The first frame of any layout change draws the
object exactly where it was; the last frame is exactly where it rests.

**2. Continuity of identity.** The same element — ideally the same DOM node —
travels. A decorative drag proxy is a narrow exception: retain the source slot,
share state, preserve the grab offset and hand back at the measured destination.
It is not native instance continuity; avoid it for stateful editors or media.
→ [014](cases/014-native-object-continuity.md)
Two copies handing over to each other (`layoutId`) are for joining
genuinely different components or screens, not a cover for a remount.
→ [001](cases/001-remount-breaks-shared-layout.md)

**3. Continuity of state.** Effects that belong to one layout (scroll fade,
blur, tilt, rotation) unwind as part of the motion, not before or after it,
and never live on the element that is being layout-animated.
→ [001](cases/001-remount-breaks-shared-layout.md),
[006](cases/006-pinned-art-under-scrolling-text.md)

**4. The visible area is the viewport minus everything covering it.** Notches
and home bars, and also an embedding host's own chrome (a chat app's bar and
message box over an in-app view). Every layout measures against that area.
→ [003](cases/003-host-chrome-covers-the-app.md),
[004](cases/004-viewport-height-ignores-covered-edges.md)

**5. Every arrangement is one tree.** Breakpoints and modes rearrange one
component tree; they don't swap in different markup. Wrappers that only
matter in one arrangement become `display: contents` in the others.
→ [001](cases/001-remount-breaks-shared-layout.md)

**6. The background is a world, not a wallpaper.** When stages of a flow have
different backgrounds, draw one world and move a camera through it (zoom and
centre per stage, eased on change), rather than swapping or cross-fading
images. Elements that belong to one stage leave by moving through that world.
→ [008](cases/008-canvas-clip-survives-the-frame.md) (worked example: frankie-tarot `scene/`)

**7. Continuity of composition.** When a preview represents the first page or
an image with annotations, preserve its internal layout through the transition.
Keeping the outer node alone is insufficient if text reflows or captions drift.
→ [010](cases/010-retained-document-and-image-presentation.md)

**8. Continuity of domain identity.** A saved object outlives its current
connection, port or process. Key user order and saved state by durable domain ID;
resolve changing runtime evidence separately and reject ambiguous matches.
→ [014](cases/014-native-object-continuity.md)

**9. Continuity of meaning and attention.** Different views of an object share
its canonical identity, membership, classification and selection. A group retains
its members; a change of representation does not create a new selected object.
Source locations stay attached so returning to evidence lands on the same content.
Logical identity across projections does not imply retaining one DOM node across
fundamentally different renderers.
→ [018](cases/018-linked-projections-preserve-object-meaning.md)

## Building it

### One tree, many arrangements

```tsx
// Rendered once. On phones the open view puts it in a scroller with a
// pinned block; on desktop and when closed those wrappers vanish.
<div className={openOnPhone ? "absolute inset-0 overflow-y-auto" : "contents"}>
  <div className={openOnPhone ? "sticky top-0 h-[100dvh]" : "contents"}>
    <motion.div className={openOnPhone ? "flex h-full items-center justify-center" : "contents"}>
      <motion.div layout layoutId={`card-${id}`} className={slotOrDetailClasses} />
    </motion.div>
  </div>
  {open && detailText}
</div>
```

### Responsive card detail through depth

For card-to-detail views, use [006](cases/006-pinned-art-under-scrolling-text.md):
desktop places retained artwork beside reading text; phone pins the artwork
while text scrolls into the foreground and the art gently recedes. Express
hierarchy through occlusion, spacing and contrast rather than nested visible
panel shells. Transparent scroll/sticky/motion wrappers are still appropriate.
Coordinate open, scroll and return; unwind depth effects from the current
presentation without a reset flash, and verify both sizes and interruption.

### Document and image presentations

Read [010](cases/010-retained-document-and-image-presentation.md) when a card is
the document's actual first page, or an image expands with caption, notes and
metadata. Establish whether the content should reflow or scale as a composition
before implementing it. For the latter, typeset once and project the retained
content; keep annotation widths, offsets and typography in one local coordinate
system. Preserve the editor root, selection and state as well as the card shell.
Use the existing motion engine when it already owns the presentation; these
principles do not require replacing it with Framer Motion.

### Collection → workspace → detail lifecycles

Read [013](cases/013-tarot-object-lifecycle.md) when objects travel between a
collection, selected workspace and detail view, or when lazy loading/GPU
painting could disrupt that flow. Preserve both the object and its return slot;
distinguish visual instance identity from content identity. Optimize painting
without discarding active interaction anchors, and sequence dependent stage
changes so the user can follow the chosen object to its destination.

### Continuous controls and cross-route handoffs

Read [012](cases/012-continuous-reading-surfaces.md) for a reading deck driven
by one position, a route-preview handoff, and selection feedback that does not
reflow content. Keep current position separate from target; drag, release,
buttons and interruption share one projection. In loops, wrap content lookup,
not travel coordinates or occurrence keys. Use occlusion rather than prolonged
text-on-text cross-fades. Reserve label space and move selection markers outside
content layout. When routes really require different nodes, reuse renderers and
hold the preview until destination geometry and scroll are ready; verify both
directions. Distinguish retained DOM, shared composition and visual handoff.

### Native navigation, drag reordering and live lists

Read [014](cases/014-native-object-continuity.md) for retained return surfaces,
direct pointer tracking with animated neighbor reflow, durable saved identity,
interruptible in-place expansion and object-owned action layers. Keep covered
pages alive only where preserving context matters, and disable their input and
accessibility traversal. For dragging, use one coordinate space and animate
reordering/settling separately from direct pointer tracking. The case distinguishes
source evidence from native runtime acceptance and documents proxy limitations.

### Linked views of the same content

Read [018](cases/018-linked-projections-preserve-object-meaning.md) for
chart/list/detail/raw workspaces. Parse once and project canonical objects into
views. Own selection above the views, scope it to its parent object, and retain
member IDs when grouping. Reuse content renderers and disclosure primitives;
views may change density and initial disclosure without changing meaning.
Keep small or folded objects reachable through larger semantic hit targets.

### Focused lanes with context strips

Read [016](cases/016-focused-lanes-context-strips.md) for wide, sparse trees,
Gantt charts and timelines: expand the focused category and retain the others
as strips. Preserve the shared time/order/depth axis and lane order; compress
perpendicular detail, keep compact events reachable, and transfer space between
lanes coherently. Distinguish lane focus from event selection and document what
the compact encoding preserves or normalizes. This is a user-endorsed Token Flow
pattern with source-backed mechanisms and explicit runtime verification limits.

### Material behavior and coordinated presentation

Read [017](cases/017-syuze-material-object-continuity.md) for Syuze's source-backed
folder, source-card and book patterns, with Franklin Board refactor comparisons.
Make hinge, opening, travel and occlusion agree. Derive shading and dependent
motion from one physical state; retain native controls and return anchors.
Coordinate background sync with presentation ownership and reconcile the latest
state afterward. Distinguish renderer reuse, retained instances and illustrative
handoffs; the case records specific limitations and verification scenarios.

### Choosing the Framer Motion tool

- **`layout`** — same element, new size or position. The default.
- **`layout="position"`** — position only; for text or anything that would
  visibly stretch while its size tweens.
- **`layoutId`** — the same logical object in two different components or
  screens.
- **`LayoutGroup`** — siblings that reflow together; scopes `layoutId`s.
- **`AnimatePresence`** — objects that truly enter or leave; give them exits,
  and key its *direct* children, or nothing exits. → [009](cases/009-exit-never-runs-without-a-key.md)
- **`layoutDependency`** — layout changes Framer can't see from a re-render.
- **`layoutScroll`** on a scrolling ancestor whose scroll moves the object;
  **`layoutRoot`** on a `position: fixed` ancestor. These fix measurement of a
  surviving element; they do nothing for a remount.

### Rules

- Keep the `layout` element's transform its own: put scroll- and
  gesture-driven `scale`, `y`, `rotate`, `filter` on a wrapper.
- Drive breakpoint changes through React state (a resize hook) so `layout` can
  animate them, rather than CSS that snaps.
- A `position: fixed` element inside an ancestor with `transform`, `filter`,
  `backdrop-filter`, `perspective`, `will-change: transform` or
  `contain: paint` is positioned against that ancestor, not the viewport.
  Make such wrappers fill the viewport, or keep fixed elements out of them.
  → [002](cases/002-fixed-inside-filtered-ancestor.md)
- Use one set of safe-area variables everywhere:
  `--safe-* = max(env(safe-area-inset-*), host-reported inset)`.
  → [003](cases/003-host-chrome-covers-the-app.md)
- Every `100dvh`/`100vh` calculation subtracts `--safe-top` and
  `--safe-bottom`; anything sized by width is also capped by the height
  left. → [004](cases/004-viewport-height-ignores-covered-edges.md)
- Replaced elements (`canvas`, `img`, `video`, `iframe`) don't stretch between
  `inset`s; give them explicit width and height.
  → [007](cases/007-replaced-element-ignores-insets.md)
- A camera over a world draws in world units through one transform; keep
  strokes and blurs constant on screen (divide by the zoom), and keep detail
  constant too (a starfield reveals fainter stars as it zooms, so density holds).
- On a reused canvas, wrap every `clip()` in `save()`/`restore()`.
  → [008](cases/008-canvas-clip-survives-the-frame.md)
- Use a coherent motion family; vary profiles deliberately by material and action
  (for example paper versus cover), while coupled effects share one state. → [017](cases/017-syuze-material-object-continuity.md)
- Reduced motion keeps identity and destination; shorten or cross-fade the
  travel, never drop the change.

## Debugging: measure, don't guess

Record `getBoundingClientRect()` of the moving element:

- `visible` — where it is drawn just before the change;
- `firstFrame` — right after triggering it, before any animation frame
  (`await new Promise(r => setTimeout(r, 0))`); Framer has already applied its
  "start from the old place" transform;
- `settled` — after it finishes, against the slot it should fill.

```js
const R = e => { const r = e.getBoundingClientRect();
  return [r.left, r.top, r.width, r.height].map(Math.round); };
const visible = R(art());
toggle();
await new Promise(r => setTimeout(r, 0));
console.log({ visible, firstFrame: R(art()), slot: R(slot()) });
```

Re-query after the toggle (`art()` as a function): if the node you held is
gone, it was remounted.

1. **Reproduce in every arrangement:** each breakpoint, the page scrolled a
   known amount, inner scrollers scrolled, and — for embedded apps — the
   host's covered edges simulated (see `references/tooling.md`).
2. **Make sure you're editing the code that runs** at that breakpoint: read
   the live element's classes and inline styles first.
   → [005](cases/005-edited-the-branch-that-doesnt-run.md)
3. **Read how the error moves.** `firstFrame ≠ visible` is a wrong start;
   `settled ≠ slot` a wrong end. Vary one condition at a time; an error that
   scales with scroll or appears on one breakpoint points at the path that
   differs.
4. **Diff the trees, not just the styles,** of the working and broken cases.
5. **Test one cause at a time, live;** keep a change only if the measurement
   moves; revert the rest.
6. **Compare the real reference UI.** Use identical content, assets, fonts and
   viewport dimensions. Open and close both implementations in a visible browser;
   inspect intermediate frames and line wrapping, not only settled screenshots
   or passing tests. Separate measured geometry from perceptual conclusions.
7. **Verify every direction:** open, close, close after inner scroll, settle,
   breakpoint change — at each size, page scrolled, edges covered.

## Checklist

- [ ] Each tracked object retains one state owner and interactive representation across arrangements; any decorative drag proxy follows the bounded exception in case 014.
- [ ] Arrangement-only wrappers are `display: contents` where they don't apply.
- [ ] Layout changes, including breakpoint switches, go through React state.
- [ ] `layoutId` only joins different components or screens.
- [ ] The `layout` element carries no scroll- or gesture-driven transforms.
- [ ] Scrolling ancestors are `layoutScroll`; fixed overlays considered for `layoutRoot`.
- [ ] No `position: fixed` element sits inside a transformed or filtered ancestor that doesn't fill the viewport.
- [ ] Safe areas = max(device, host); every viewport-height calc subtracts them.
- [ ] Content sized by width is capped by the height left on short or covered screens.
- [ ] Frame 0 = visible and settled = slot: open, close, inner scroll, breakpoint change; every size; page scrolled; edges covered.
- [ ] Stage backgrounds are one world under a moving camera, not swapped images.
- [ ] Canvas clips are scoped with save/restore.
- [ ] Every direct child of `AnimatePresence` has a distinct key.
- [ ] Reduced motion keeps identity and destination.
- [ ] Expanded objects retain a source slot; visual instance IDs survive cross-component handoffs.
- [ ] Rendering optimizations retain selected/traveling anchors and preserve hit-test geometry.
- [ ] One gesture produces one transition; deferred stage changes cannot affect a newer session.

- [ ] First-page previews retain the same typesetting and line breaks when expanded.
- [ ] Composition-bound annotations preserve normalized offsets, scale and wrapping during motion.
- [ ] Scroll, corners and reveal unwind together; nothing unmounts before the return finishes.
- [ ] Editing preserves caret/selection; floating tools do not reflow or scale the document.
- [ ] A visible reference comparison complements focused core tests; debug controls stay out of product previews.

- [ ] Drag, release, repeated commands and interruption share one live motion state; loop seams do not reset travel coordinates.
- [ ] Selection and status changes preserve content geometry; overlapping text surfaces remain legible.
- [ ] Cross-route previews match destination geometry and survive until its DOM/scroll are ready; both handoff directions are verified.
- [ ] Export reuses the preview composition and waits for cloned assets to decode.

- [ ] Saved identity and user order survive runtime discovery changes; ambiguous matches do not silently rebind objects.
- [ ] Returning preserves relevant context; covered retained surfaces cannot receive input or accessibility focus.
- [ ] Dragging preserves grab offset; direct tracking and layout springs are separate; proxy handoff is verified at the latest slot.
- [ ] Reversal starts from current presentation state; asynchronous results reconcile with the same object.

For linked-view workspaces:

- [ ] Changing view or width preserves the selected object, its members and semantic labels.
- [ ] Groups retain individual anchors and source locations; unknown mappings do not invent destinations.
- [ ] Selection reveals all matches; manual folding remains usable until the next selection.
- [ ] Compact marks retain usable hit targets and tiny categories remain discoverable.
- [ ] Distinguish model/render tests from browser checks of focus, scroll and motion.

## Case index

| # | Case | Rule it taught |
|---|------|----------------|
| 017 | [Syuze material and object continuity](cases/017-syuze-material-object-continuity.md) | Material-led paths, shared shading state, native source controls and sync ownership |
| 001 | [Remount breaks shared layout](cases/001-remount-breaks-shared-layout.md) | One element at one tree position; `display: contents` wrappers |
| 002 | [Fixed inside a filtered ancestor](cases/002-fixed-inside-filtered-ancestor.md) | Transformed/filtered wrappers capture `position: fixed` |
| 003 | [Host chrome covers the app](cases/003-host-chrome-covers-the-app.md) | Safe area = max(device notch, host-reported insets) |
| 004 | [Viewport height ignores covered edges](cases/004-viewport-height-ignores-covered-edges.md) | Subtract safe areas in `dvh` calcs; cap width by height |
| 005 | [Edited the branch that doesn't run](cases/005-edited-the-branch-that-doesnt-run.md) | Read the live element before editing responsive branches |
| 006 | [Pinned art under scrolling text](cases/006-pinned-art-under-scrolling-text.md) | `sticky` block + effects on a wrapper, not the layout element |
| 007 | [Replaced element ignores insets](cases/007-replaced-element-ignores-insets.md) | Size `canvas`/`img`/`video` explicitly |
| 008 | [Canvas clip survives the frame](cases/008-canvas-clip-survives-the-frame.md) | Scope canvas clips; background as one world + camera |
| 009 | [Exit never runs without a key](cases/009-exit-never-runs-without-a-key.md) | Key the direct children of `AnimatePresence` |
| 010 | [Retained document and image presentation](cases/010-retained-document-and-image-presentation.md) | Typeset once; project the full composition; verify intermediate frames and editing |
| 016 | [Focused lanes with context strips](cases/016-focused-lanes-context-strips.md) | Expand focus; preserve shared coordinates, context strips and event reachability |
| 011 | [Poker object consistency](cases/011-poker-object-consistency.md) | Source-backed poker object consistency patterns |
| 014 | [Native object continuity](cases/014-native-object-continuity.md) | Retained navigation, bounded drag proxies, durable identity and reversible local interactions |
| 012 | [Continuous reading surfaces](cases/012-continuous-reading-surfaces.md) | One motion owner; unwrapped slots; stable feedback; explicit route handoff |
| 015 | [Blog card-view carousel](cases/015-blog-card-view-carousel.md) | User-endorsed reference: one position, interruptible input and continuous loop slots |
| 013 | [Tarot object lifecycle](cases/013-tarot-object-lifecycle.md) | Preserve return slots, visual identity and input semantics while optimizing paint; source review with explicit boundaries |

| 018 | [Linked projections preserve object meaning](cases/018-linked-projections-preserve-object-meaning.md) | Shared selection, reversible grouping, source anchors and one interaction grammar |

### Positive reference: blog card-view carousel

Use [015](cases/015-blog-card-view-carousel.md) for the user-endorsed stacked-card carousel: one
continuous position across drag, settle and repeated commands; unwrapped slot
identity; legible occlusion; and one-step trackpad intent.

## Maintaining this skill

This skill grows from real bugs and source-backed implementation reviews. For a
reviewed pattern, record source revision, mechanisms, limits and outstanding
runtime acceptance; do not invent a bug, failed attempt or measurement.

This skill grows from real bugs. After solving a bug in this area — a layout
animation that jumps, an object that lands under chrome, a breakpoint that
breaks continuity:

1. **Add a case.** Copy [`cases/_TEMPLATE.md`](cases/_TEMPLATE.md) to
   `cases/NNN-short-slug.md` (next number). Record the symptom, the conditions
   that trigger it, the measurement that proved the cause, what did *not*
   work, the fix, and the general rule. Name the project and files in the
   case; keep the rule itself project-neutral.
2. **Add it to the case index** above.
3. **Promote the rule** if it's new: add it to Principles, Rules or the
   Checklist, linking the case. If it sharpens an existing rule, edit that
   rule instead of adding a near-duplicate.
4. **Prune.** Keep this file short: principles, method, checklist, index.
   Detail belongs in cases. Merge cases that turn out to be the same bug.
5. **Commit** in this skill's own git repository, one case per commit:
   `git add SKILL.md cases/NNN-short-slug.md` followed by
   `git commit -m "case NNN: …"` from your skill checkout.
