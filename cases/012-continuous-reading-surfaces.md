# 012 — Continuous reading surfaces and controlled page handoff

- **Project:** `knowledge-vault`
- **Files:** `_system/site/vitepress/docs/.vitepress/theme/` — `CardDeck.vue`, `ArticlePull.vue`, `SidebarMarker.vue`, `PullHint.vue`, `PullMark.vue`, `Layout.vue`, `custom.css`, `UI_COMPONENTS.md`.
- **Stack:** Vue / VitePress; custom requestAnimationFrame motion and CSS.
- **Date:** 2026-10-17
- **Evidence:** source review of existing good patterns, not a newly reproduced/fixed bug. No runtime geometry, phone test, or perceptual equivalence was measured in this review. Documentation describes intent; the functions below establish implementation.

## UX problem

A reader should be able to follow the same sheet through a drag, release,
interruption and reversal; recognise the incoming article when navigation
commits; and change selection without surrounding text moving. Smooth easing
alone does not establish any of those properties.

## What the project does well

### Card-view carousel

The user singled out this carousel as a positive UX reference. Its motion,
slot identity and occlusion patterns now have a [dedicated case](015-blog-card-view-carousel.md).

### Navigation is a deliberate visual handoff

`ArticlePull.vue` renders the neighbour's actual page module with shared
`PostHeader` and article styling. It measures the reading column and title's
resting position rather than inventing another preview layout. Only the article
column moves; surrounding navigation provides stable context. The next-preview
surface covers its previous-link zone, and `nextExitExtra` independently clears
the outgoing article tail. One distance need not satisfy both entry and exit.

`handOff()` awaits routing, a Vue DOM update and scroll-to-top; the next direction
also waits for an animation-frame callback before clearing the preview. This is
an explicit readiness boundary, **not proof that pixels have painted**. The
previous direction follows a different path and needs its own verification.
Preview descendant IDs are removed to prevent duplicate anchor/outline targets.

**Transfer:** for genuinely different routes, shared content/renderers plus a
controlled handoff can preserve visual identity without retaining one DOM node.
Keep the cover until destination geometry and scroll are ready, compare final
preview against first destination, and keep preview copies non-interactive and
out of semantic navigation. ID stripping alone is not a full accessibility audit.

### Feedback changes without moving its anchor

`SidebarMarker.vue` retains one gutter bar, moving and resizing it against the
active item's measured box. Selection adds no border/padding to the title, so
its wrapping stays stable. Initial placement does not animate from an arbitrary
origin; `ResizeObserver` recalculates geometry.

`PullHint.vue` overlays both verbs in one grid cell whose width accommodates the
longer label, so switching to “let go” does not shift the centred mark.
`PullMark.vue` derives stroke progress from the pull threshold; spring-back
reverses it. Feedback expresses the same action state rather than a decorative
independent timer.

**Transfer:** reserve space for alternate labels, keep selection decoration out
of content layout, and derive progress feedback from the action's real progress.

### Preview and export share composition

`capture()` clones the displayed `.social-card`, removes presentation transforms,
and exports at its authored 1080 × 1350 size. It waits for fonts and source image
decoding, embeds image data, then waits for the clone's image decoding too.
Failures abort instead of silently producing a card with missing imagery.

**Transfer:** use the same composition for preview and export; check readiness
again after cloning or replacing image sources. A clone does not inherit decoded
image readiness. This preserves composition, not literal DOM identity.

## Boundaries — do not overgeneralise

- Text ↔ cards in `Layout.vue` is a staged exit/entry between different renderings;
  `CardDeck` is conditionally mounted. It is not a retained-document morph and
  does not establish reading-position restoration.
- Desktop article navigation uses explicit clicks; phone touch pulls use edge
  thresholds. Wheel scrolling does not navigate articles. Preserve action meaning
  across input modes; identical gesture bindings everywhere are not required.
- A looping deck has no terminal boundary, so its drag has no rubber band. Article
  edges do have a boundary and use damping plus a commit threshold. Resistance
  should express real structure, not be applied as a universal animation effect.
- Existing reduced-motion paths skip travel but keep the destination. Host safe
  areas, focus transfer and velocity continuity require separate verification.

## Verification to run when applying this case

1. Grab mid-glide, reverse, cancel, and press next repeatedly; compare positions
   immediately before/after input handoff, and velocity if required.
2. Cross both loop seams; inspect slot identity, paint order and text overlap at
   intermediate frames, including resize and a short viewport.
3. Navigate next and previous with long titles and long outgoing tails; compare
   preview/destination geometry, test delayed loading, scroll and browser history.
4. Change selection with wrapped labels; assert unchanged content boxes and no
   marker travel on initial placement. Test both languages' threshold labels.
5. Compare preview/export line breaks and image composition with uncached assets;
   test failed image decode and reduced motion. Record observed results separately
   from implementation intent.

## Rule

Continuity requires one owner for motion, stable layout anchors, believable
occlusion and an explicit readiness boundary when surfaces must be replaced.
State which guarantee is present: retained node, shared composition, or visual
handoff. Do not describe all three as the same element surviving.
