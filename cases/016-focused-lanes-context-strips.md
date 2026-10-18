# 016 — Focused lanes with context strips

- **Reference:** Token Flow, user-endorsed UX pattern, 2026-10-17.
- **Source:** `token-flow`, revision `55f214a`.
- **Files:** `ui/components/workspace/turn-flow.tsx` (`TurnFlow`, `FlowLane`),
  `ui/styles/motion.css` (`.t-lane`, `.t-lane-layer`).
- **Evidence:** Source inspection plus the user's explicit endorsement. No new
  browser motion, touch or interruption acceptance is claimed.

## Use when

A multi-turn tree, timeline, Gantt chart or other sparse diagram becomes too wide
when every category is expanded, but hiding categories would destroy context.
Partition the structure into lanes, expand the focused lane and retain every
other lane as a narrow strip in its original position.

The user's intended pattern is category-based focus plus context. Token Flow's
actual lane membership uses captured thread/scope IDs; its token categories are
inside each lane. Generalize the interaction without confusing these two levels.

## The continuity contract

1. **Keep the shared axis.** Compress the axis perpendicular to time/order. Keep
   events at their original time or sequence coordinates and preserve gaps. In
   Token Flow, all lanes use `top = globalTurnIndex * ROW` and the same total
   height. Independently packing each strip would make unrelated turns look aligned.
   For a horizontal-time Gantt chart, preserve x/time and compress lane height.
2. **Keep the context visible.** A strip still shows where its objects occur,
   their semantic colors and enough continuity to recognize the group. It is a
   reduced representation of the same lane, not merely a tab or collapsed title.
   Lane order stays stable when focus changes; do not move the focused lane first.
3. **Transfer space with focus.** The incoming lane grows as the outgoing one
   contracts on a shared motion curve, pushing neighbors coherently. Users can
   follow the same lane between detailed and compact states. One expanded lane
   is the baseline; use explicit pinning only if simultaneous comparison warrants it.
4. **Separate focus from selection.** Expanding a lane can inspect that category
   without inventing a selected event. Selecting an event in a strip both reveals
   its owning lane and selects that exact object. Keep these state transitions
   explicit so a previous lane override cannot hide a newly selected object.
5. **Preserve access while reducing detail.** Give compact events usable targets
   and accessible labels. Token Flow places full-width chronological row buttons
   over folded turns, so the gap around a tiny square is also actionable. For a
   chart with simultaneous events, scope targets to their lanes or otherwise
   disambiguate overlaps; full-width rows are not a universal recipe.
6. **State what the strip encodes.** Token Flow's expanded lane uses token widths;
   the strip uses fixed-size squares with composition slices and cache hatching.
   Equal square sizes do not mean equal token totals. Its quiet connecting band
   conveys thread continuity, not a quantitative flow or an inferred dependency.
   Minimum slice widths aid visibility, so tiny slices are not exact proportions.

## Implementation lessons from Token Flow

- `TurnFlow` groups by stable scope/thread identity and keys lane shells by that
  identity. A local explicit lane choice takes precedence until event selection
  changes; then the selected event's owning lane is revealed.
- `FlowLane` renders both full and strip representations from the same data and
  preserves global row positions. The compact view keeps color, hatching, corner
  treatment and selection fading, while simplifying links and removing labels.
- The expanded width is computed from available width minus the other strips.
  The current values (28px strip, 280px minimum expanded lane) are product choices,
  not reusable constants. Bounded horizontal overflow handles insufficient width.
- `.t-lane` animates allocated width. Full content is laid out at its destination
  width and clipped/revealed by the moving shell, avoiding per-frame text and
  graph reflow. Full and strip layers briefly coexist; outgoing layers are inert
  and removed after the transition. This is stable lane identity plus a visual
  handoff, not proof that every mark retains the same DOM instance.
- The strip's SVG is decorative; the row and lane buttons provide interaction.
  Keep one interactive representation during handoff and preserve keyboard access.

## Transfer to other diagrams

| Diagram | Lane grouping | Axis that must stay stable | Strip can retain |
|---|---|---|---|
| Multi-turn trace | Thread, actor or task scope | Global turn order | Event positions, category composition, state |
| Gantt chart | Team, resource or project | Shared time scale and task intervals | Duration marks, milestones, selected task |
| Timeline | Category or source | Timestamp positions | Event marks, density and selected event |
| Sparse tree | Stable branch or subtree | Shared depth and meaningful alignment | Branch landmarks and explicit topology |

For trees, retain parent/child structure and clearly defined correspondence; do
not imply that unrelated nodes share a time coordinate. For Gantt charts, keep
actual intervals instead of copying Token Flow's fixed squares. When precise
cross-lane magnitude comparison is the primary task, shared-scale overview or
multiple pinned lanes may serve better than a single detailed lane.

## Verify when implementing

- Switch focus A → B → A. Membership, lane order, time/depth anchors and event
  selection remain stable; only the detail allocation changes.
- Select an event in a strip, including a gap-sized hit area. The correct lane
  opens and the exact event remains selected in the linked inspector.
- Compare full and compact encodings: colors/states agree, while totals and
  dependencies are not inferred from normalized sizes or decorative connectors.
- Resize, scroll and reverse focus mid-transition; measure anchors on the shared
  axis and verify no stale outgoing layer receives pointer or keyboard input.
- Check narrow screens, many categories and simultaneous events. Preserve local
  overflow and distinguish overlapping targets; do not silently remove context.

## Rule

Compress detail perpendicular to the shared structural axis. Keep unfocused
objects recognizable, positioned and reachable, then transfer space to the
focused group without changing its identity or the surrounding context.
