# 018 — Linked projections preserve object meaning

- **Project:** Token Flow, `token-flow`
- **Evidence snapshot:** `55f214a`, 2026-10-17; source review and existing model/render tests.
- **Scope:** A positive implementation case, not a newly fixed animation bug.
- **Use for:** Chart/list/detail/raw views of the same objects, responsive inspectors,
  grouped content and compact navigation. The lessons do not require a motion library.

## What works and why

### 1. One semantic object, several projections

`ui/lib/types.ts` defines `TurnModel.blocks`; categories retain `memberIds`.
`ui/lib/cross-link.ts::selectedBlocks` resolves selection against those blocks.
`ui/components/workspace/request-view.tsx` projects them as Dialog, Tokens and
Raw; `TimelineEntry` uses the shared `EntryRow` for secondary detail.

The user can change the question they ask of the content without changing what
content they are looking at. Parse/classify once. Let views choose presentation,
not independently reconstruct identity or reinterpret categories.

### 2. Selection belongs to the workspace

`ui/components/views/workspace-view.tsx::WorkspaceView` owns `selectedId`,
`tokenSelection` and `requestView`. Flow and inspector receive the same selection.
Selecting the active turn again preserves its block selection; selecting another
turn clears that scoped selection while retaining the chosen view mode.
`request-view.tsx::rowMarks` opens matches and dims unrelated rows using their
existing category colors. Multi-member selection exposes previous/next matches.

Store identity rather than a display index. Define when selection survives and
when its scope invalidates it. Do not give each projection its own selected item.
Color, label and evidence state should keep their meanings across views.

### 3. Grouping changes reading density without destroying members

`request-view.tsx::InstructionRows` groups adjacent instructions with the same
category and label. `InstructionGroupRow` keeps a `BlockAnchor` for each member
and passes all member IDs to the linked swatch. Selecting a member can reveal the
group without manufacturing a replacement identity.

`ui/components/workspace/request-search.ts::locate` maps matches using `rawPath`
and half-open UTF-16 `rawRange`, including split sections inside one source string.
Raw-only metadata stays searchable without claiming a structured block target.
Inherited content must not acquire an invented location in the current request.

Keep grouping reversible. Separate local selection IDs, source provenance and
cross-turn identity: similar text or matching local IDs are not proof of lineage.
Preserve unknown mappings instead of linking to a plausible but wrong object.

### 4. One interaction grammar, appropriate density

`ui/components/ui/disclosure.tsx` shares chevron, header, keyboard activation and
panel behavior across section, row, inline and tree tiers. `FoldLine` reserves the
same glyph slot even when there is nothing to expand. `RowTierContext` lets the
same content rows fit Dialog and Tokens while their default-open rules differ.

`ui/components/motion.tsx::useAccordion` lets users fold manually after automatic
selection focus; a changed reset key reapplies focus. Programmatic reveals open
immediately so navigation can target final geometry; manual collapse waits for
the motion duration before unmounting.

Reuse the actual primitive and renderer, not just a similar chevron or border.
Keep reading alignment stable. When navigation reveals a target, either settle
its layout first or explicitly coordinate scrolling with its animation.

### 5. Responsive layout keeps the same task and objects

`WorkspaceView` renders one `TurnFlow` and one selected inspector. CSS places the
inspector beside the flow on wide screens and over it on narrow screens. The
flow remains present; its top-to-bottom chronology does not change with width.
Selection lives outside the responsive presentation.

Change placement and density before introducing an alternative mobile workflow.
Preserve order, labels, selected identity and access to evidence. On returning
from an overlay, verify the user's prior place rather than merely reopening a list.

### 6. Compression preserves reachability

`ui/components/workspace/turn-flow.tsx` uses the same `FlowLane` renderer for full
and strip presentations. Folded turns retain full-width chronological button
regions, including the gaps around narrow marks. The category projection retains
small categories, with selectable legend entries for discoverability.

A compact representation should reduce visual space without reducing the semantic
object's usable target to a tiny mark. Keep hit regions aligned with the displayed
chronology; verify overlapping regions do not intercept other controls.

## Boundaries: do not overclaim continuity

- Shared component code is not proof of retained DOM identity. Lane transitions
  temporarily render full and strip layers, with the leaving full layer inert.
  Reuse the semantic/rendering lesson; do not cite it as a single-node morph.
- Overview and turn inspector use different keyed branches. This is not evidence
  that all scroll, focus or local disclosure state survives every navigation.
- `fullyCachedBlockIds` refuses to assign member cache state from a mixed aggregate.
  Consistent visuals must preserve evidence strength, not manufacture certainty.
- Tests below establish model and rendered-markup properties. No live browser,
  device, intermediate-frame or accessibility acceptance was performed in this review.

## Validation to reuse

Run the relevant existing tests, then test actual interaction when changing UI:

1. Select a chart group with multiple members; switch Dialog → Tokens → Raw.
   Confirm the same member set, all matches revealed and correct source ranges.
2. Select one member inside a reading group. Confirm the group opens while its
   individual anchor remains addressable. Test repeated text and raw-only metadata.
3. Manually fold a selected section, then select a different object. Check that
   manual control works and the new focus reapplies; reselecting the active turn
   must not clear its linked block.
4. Resize with a selection active; open/close the narrow inspector after scrolling.
   Verify order, selection, return position, visible focus and local overflow.
5. Activate a folded-row gap and a tiny category via the legend; verify the exact
   target and keyboard access. Check interrupted transitions and inert exits.

At the evidence snapshot, `cd ui && npm run test:model` passed: canonical identity,
category membership and conserved totals; cross-view match disclosure and cache
certainty; source spans, repeated text and raw-only search. See
`ui/lib/token-model.test.ts` for the executable fixtures. Browser steps remain a
verification recipe, not a claim that they were run.
