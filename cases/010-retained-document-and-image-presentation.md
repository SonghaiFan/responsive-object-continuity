# 010 — Retained document and image presentation

- **Project / files:** Franklin Board React library, worktree `franklin-board-react-library`; `packages/board/src/core/presentation.ts`, `react/presentation.ts`, `react/Cards.tsx`, `react/EditableText.tsx`, `rich-text/RichText.tsx`, `motion/presentation.ts`, `styles.css`; core browser scenario `library-tests/browser-smoke.scenario.js`.
- **Reference:** syuze `syuze/src/spatial/{document-view.js,document-card.js,text-editor.js,notes-toolbar.js,image-viewer.js}`; retained Franklin legacy UI at `/legacy/`. syuze was read-only.
- **Stack:** TypeScript, React, Lexical, analytic spring/RAF, native manual popovers.
- **Date:** 2026-10-17

## Symptom

The user repeatedly corrected an implementation whose tests passed:

- “原版的 preview card 就是 document 首页的真实样子”: the document card
  must be the actual first page, not a separately styled summary.
- Image caption and note drifted during return. Making their centres follow the
  image was still insufficient: “应该保持相对位置不变，当缩小的时候”.
- An input-like focus rectangle surrounded the otherwise paper-like editor.
- Text keeps / Image keeps debug buttons polluted the preview.

## Conditions

Canvas cards expand to a full-window reader, then return after editing, image
zoom/pan, or vertical scrolling. The board can be embedded in a scrolled page,
zoomed, or contain rotated/stacked objects. Images have independent caption,
note and metadata regions. Rapid reversal and viewport resize expose errors
that a settled screenshot misses.

## Measurement

Visible legacy/React comparisons used the same document and image at 1400 × 1100.
Legacy document content remained 760 layout pixels wide: its projection changed
from .225 on the card to .833333 expanded. Matching typography preserved the
same title wrapping and paragraph positions. Font differences had to be removed
before judging layout; the playground supplied legacy Unica through a host token.

The shared browser scenario retained references to the card, editor, image and
image parent; compared content width/height and title height across opening;
sampled opening, reversal and return under non-unit camera zoom; checked
annotation opacity and scale against the image; and closed after reader scroll.
Case-specific geometry tolerances were .08px for sampled motion and .1px for the
first closing frame. They are evidence thresholds, not universal skill defaults.
An explicit scroll-extent condition was needed before testing long-document
scroll: settled animation RAFs do not imply ResizeObserver/layout completion.

## Cause

Identity had been preserved at the outer React node, but not at the composition
level. Separate card/full-view font sizes caused reflow. Text below the image
was initially anchored to the viewport; later it followed the image centre but
kept fixed pixel gaps, widths and font sizes. That still changed the composition
while shrinking. A sticky formatting toolbar also lived inside the transformed
text layout, affecting geometry and shrinking with the page.

## Didn't work

- Interpreting a sketch without checking the actual reference code and UI.
- Replacing the source presentation with a new rounded image-plus-text paper,
  arbitrary padding/minimum note height, or a different document aspect ratio.
- Matching only outer bounds or spring constants while text reflowed inside.
- Following the moving image with fixed-size annotations: movement alone is not
  proportional scaling, and changing caption width each frame changes wrapping.
- Hiding/unmounting extras as soon as preview state becomes false: the return
  animation still owns the visible presentation at that point.
- Calling a motion identical based solely on core tests or hidden-tab captures.

## Fix

### One typeset document, projected into two presentations

Use one retained editor/content subtree. Keep layout width, fonts, paragraph
spacing and line breaks consistent at a given viewport. Scale and translate that
content into the card, clipping everything below the first page. Expansion
changes the projection; overflowing content extends the reading surface downward.
Use one reading scroller rather than nested editor/paper scrollbars.

In this project the canonical document is 210 × 297; layout width is
`min(760, viewportWidth - 44)`, padding follows `26/280` of paper width and top
offset follows `29/280` of paper height. Those values reproduce this reference,
not a universal document standard. Keep the layout stable during a flight; a
viewport change may legitimately establish a new layout.

### Image and annotations share local coordinates

Typeset caption, note and metadata at their expanded layout widths. Let `W` be
the expanded image width and `r = currentImageWidth / W`. Project every attached
region from that layout, rather than recomputing its wrapping at the current
image width:

```text
annotation width for layout = expandedAnnotationWidth
annotation transform       = scale(r), origin top-left
annotation left            = localX * r
annotation top below image = currentImageHeight + localGap * r
annotation opacity         = sharedPresentationReveal
```

For an aspect-preserving image this keeps normalized offsets, gaps and text
size constant. Include metadata in the same rule; choose desktop/sidebar or
narrow/below-body layout before projecting it. Apply the composition's rotation
consistently. Keep the image's own clipping radius separate from any reader
background; avoid adding an enclosing paper unless the design calls for one.

Here the user requested image-centred vertical placement, metadata on the left
on desktop and below on narrow screens, and proportional shrink of all extras.
These deliberately refine the original syuze behaviour; do not silently treat
them as proof of exact legacy parity.

### Return owns the presentation until its last frame

Animate position, size, rotation, radius, reveal and reader scroll together.
Sample the last painted frame for reversal. Measure the current canvas landing
location, including embedded-page offset and camera transform. Preserve the
scroll extent during return to prevent browser clamping from causing a jump.

Keep extras mounted and visible through their fade. They may become inert when
closing, but restore canvas clipping and release top-layer presentation only
after the zero-reveal return frame. Keep original nodes, editor instances and
component state; never substitute a clone just to simplify animation.

### Editing and tools are separate from page geometry

In an opened reader, a single click can establish a caret; double-click should
retain native word selection. Beginning a history session must not refocus the
editor in a way that resets that selection. Preserve IME composition, formatting
selection, undo grouping and saved content through transitions.

The formatting toolbar can use the same retained DOM node promoted to a manual
popover: it escapes ancestor transforms/clipping without reparenting the editor
or creating a second React root. Anchor it to the viewport and use pointer-down
prevention to retain selection. Clean up popovers and animation callbacks.

The user requested no input-box outline on these full-view text surfaces. Scope
that choice to the reading editor; keep accessible labels, caret/selection and
visible focus indicators on toolbar/close controls. Empty caption hints are
presentation only, never saved text. Rich editors can represent emptiness as
`<p><br></p>`, so `:empty` alone is insufficient.

## Verification and delivery

- Open the real reference and candidate visibly with identical content; inspect
  card, open view, mid-return, scrolled close, reversal and narrow layout.
- Check line breaks and normalized annotation offsets/scales as well as outer
  bounds, clipping radii, opacity and exact node identity. Capture the first
  closing frame synchronously before animation advances.
- Keep document/image gesture transforms separate from presentation transforms.
- Wait for observable layout readiness, not only a fixed timeout or empty RAF queue.
- Avoid concurrent foreground-dependent browser flows competing for focus.
- Retain focused core regression checks instead of accumulating one script per
  iteration. Keep state probes in test fixtures/assertions, not visible previews.
- Report actual visual and measured evidence separately; local Chromium success
  does not establish every browser/device or perfect perceptual parity.

## Rule

A retained shell is not enough: preserve the internal typesetting and local
geometry of the composition the user expects to travel. Verify the real reference,
intermediate frames and editing state before declaring continuity correct.
