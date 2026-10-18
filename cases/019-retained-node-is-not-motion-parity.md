# 019 — A retained node is not motion parity

- **Project:** ROC UI, `registry/roc/expandable-card.tsx`, `source-stack.tsx`, `card-preview.tsx`.
- **Stack:** React 19, native dialog, sampled presentation springs.
- **Date:** 2026-10-18.
- **Reference:** Syuze `eac9e39c47c913ac389fc84b5717b678bc38f4e5`, `src/spatial/document-view.js`, `asset-preview.js`, `presentation-motion.js`, `spring.js`.

## Symptom and cause

The user reported that ROC did not reproduce the feel of the real Syuze project,
then identified card preview and return as the clearest mismatch. The earlier ROC
kept its React subtree but expanded layout width and height, reflowed its text,
and applied a generic FLIP tween. Source inspection similarly enlarged an inline
card. Tests of node identity and final bounds passed; they did not establish
reference motion fidelity.

## Correction

Treat the card as a first-page crop of one fixed reading composition. Retain the
paper, content and editor in a native dialog that is embedded in its source slot
while closed and promoted to the top layer while reading. A separate, persistent
slot supplies the return anchor. Do not replace its content or add a second editor.

Drive position, scale, clipping height, radius, reveal and closing scroll from one
presentation state. Use the reference viewer's action-specific profiles: stiffness
100 and critical damping, mass .48 for opening and .32 for return. These values
are provenance for this port, not universal constants. That Syuze viewer restarts
velocity on retargeting; do not claim the velocity-preserving behavior of its
separate canvas motion engine.

An initial implementation sampled a later time immediately when closing and
painted that new position before returning, which failed a same-task first-frame
comparison. ROC now starts from the last painted pose. Remeasure the actual slot
while closing and keep the paper in the top layer until landing. Unwind scroll
with the return rather than resetting it before the first return frame.

## Evidence

- 42 analytic samples compared to the actual Syuze spring function; maximum
  position/velocity difference below 5e-13.
- 15 focused Chromium checks: same paper/editor, unchanged typesetting width,
  preserved text and selection, interrupted close below one pixel of first-frame
  displacement, landing within one pixel, focus return, phone bounds, scroll
  unwind, source input identity and reduced motion.
- 23 gallery regression checks and registry consumer TypeScript compilation passed.
- The real Syuze document open/close code ran in a visible synthetic canvas fixture;
  missing fixture bridge capabilities were supplied in a temporary copy. The
  source application was not edited.

The two displays have different content, visual styling and reading destinations.
This is a source-backed correction, not pixel equality, physical-device acceptance,
or a port of Syuze's whole canvas. Folder layout/camera, filing, drag, source spread
and gallery inspector transitions remain separate work.

## Rule

Evaluate reference fidelity separately from identity correctness. Compare the
composition, path, profile, interruption policy, clipping, scroll and actual
return anchor. A passed retained-node test cannot certify any of those by itself.
