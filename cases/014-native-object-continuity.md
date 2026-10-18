# 014 — Native objects through navigation, dragging and live updates

- **Project:** LeftOpen, `leftopen`, commit `a91dccb`.
- **Stack / date:** SwiftUI + AppKit; 2026-10-17.
- **Evidence:** Source review of an existing implementation, not a newly fixed bug. No native interaction recording or frame measurements were performed. The mechanisms below are verified in code; perceptual smoothness, focus restoration and interruption behavior still require runtime acceptance.

## Retain the place the user will return to

`Sources/LeftOpenApp/MenuPanel.swift`, `path`, `body`, `goBack`:
the keyed page stack keeps `.list` beneath `.port` and `.review`. Covered pages
are transparent, disabled and accessibility-hidden rather than removed. Header
and footer sit outside the stack. Returning removes the upper page instead of
rebuilding the list. Search and expansion state live on the panel; retaining the
list supports scroll continuity. Depth changes explicitly update search focus.

General rule: keep the return surface alive when its local context matters;
remove covered surfaces from interaction and accessibility navigation. Retention
alone does not prove keyboard focus restoration. Distinct pages need not pretend
to be a shared-element morph.

## Separate pointer motion from layout motion

`MenuPanel.swift`, `savedProjects`, `ShortcutDragState`, `ShortcutDragGesture`:

- Shortcuts are keyed by binding ID. The source stays in layout at opacity zero
  while a non-hit-testing overlay reuses `shortcutLabel(binding)`.
- One named coordinate space supplies source frames, hit targets and landing
  frames. The initial grab offset is preserved, avoiding a jump to the center.
- Pointer updates explicitly disable animation. Neighbor reordering and release
  use a spring, so the object follows the hand while surrounding slots yield.
- On release, the code yields for layout, looks up the current landing frame,
  moves the overlay there, then restores the source. Opening is guarded while a
  drag is active; a six-point movement threshold distinguishes drag from click.

This is a deliberate visual proxy, **not the same native view instance**. It is
appropriate for a stateless shortcut label; do not transfer this pattern blindly
to editors, videos or controls with selection/playback state. Keep one state owner,
one interactive source and one visible representation; hide decorative proxies
from accessibility as well as hit testing in implementations that need it.

Limit: LeftOpen clears the proxy after a fixed 240 ms. That is not proof a spring
has settled. Prefer animation completion or verified geometry for robust handoff;
test interruption, layout changes and reduced motion before copying the timing.

## Keep durable identity separate from runtime identity

`Sources/LeftOpenCore/FixedAddress.swift`, `FixedAddressBinding`: a saved project
has a persistent ID and service matching evidence independent of its current
port. Resolution refuses ambiguous candidates. `ProjectShortcutOrder.swift`
normalizes preferred IDs against available IDs, retains user order, removes
stale/duplicate entries and appends new ones. `FixedAddressManager.swift`,
`orderedBindings` / `moveShortcut`, persists that order. `savedProjects` renders
bindings and varies running status without requiring a running listener.

General rule: anchor saved objects and user order to durable domain IDs; treat
connection, process, port and availability as changing state. Do not reuse a
runtime listener key as a durable project key. Here `Activity.id` is port + PID;
it is not a globally durable process identity.

## Expand relationships in place and make reversal explicit

`MenuPanel.swift`, `groupRows` / `toggle`: the same process-header role anchors
its child port rows. A count summarizes the collapsed children; expanding exposes
sorted ports beneath their parent. The chevron consistently means folding, while
leaf selection opens details. Per-group tasks are cancelled on reversal and
continue from the current revealed count; collapse removes children in reverse
order. Reduced motion skips the stagger.

General rule: preserve the parent as a spatial anchor and give controls stable
meanings. Reverse from current presentation state, not a replay from an endpoint.
Limit: conditional dividers and the single-listener/multi-listener branch do not
prove native header-instance retention across all scan updates; verify those
transitions before making that stronger claim.

## Make actions belong to the object

`MenuPanel.swift`, `PortRow`: one row component owns the card, gesture state and
action layer. Horizontal movement offsets the card to uncover actions underneath;
the opaque card hides the layer until exposed. Mouse and trackpad use the same
tracker; accessibility actions expose the corresponding operations. Unavailable
actions resist movement. The review page enters from the leading edge, consistent
with the close-action side; details enter from the trailing edge.

`Sources/LeftOpenApp/AppMain.swift`, `closeNow` / `runScans`: optimistic hiding is
a temporary set of activity IDs, not deletion of the scan data. Cleanup removes
the mask, and refresh requests made during a scan queue another scan. This helps
reconcile immediate feedback with actual service state. It does not guarantee
that a close succeeds; failures must remain visible and associated with the object.

## Acceptance before claiming the experience works

- Scroll, search and expand; enter details/review and return. Check scroll,
  query, expansion, focus and accessibility traversal of covered pages.
- Grab a shortcut off-center, cross rows, release, interrupt the landing and
  reopen. Check initial position, latest slot, single visible label, no accidental
  open, and persisted order. Repeat with reduced motion.
- Reverse folding mid-stagger; change listener count while expanded. Check parent
  identity and remaining child state, including the one-to-many boundary.
- Stop/restart a saved project on a different port. Check its saved position and
  name persist and ambiguous discovery does not attach another service.
- Exercise failed close and an overlapping refresh using controlled fixtures;
  check restoration and actionable error context without touching unrelated services.

`Sources/LeftOpenApp/PanelSnapshot.swift`, `ShortcutGesturePreview`, already
reuses the production drag state/modifier with neutral fixtures and an open
counter. Use such a harness for gesture evidence; screenshots alone cannot prove
tracking or handoff. No runtime acceptance is claimed by this case.
