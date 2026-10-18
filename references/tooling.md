# Tooling for seeing these bugs

Layout and transition bugs are easy to misread when testing with automated or
embedded browsers. These are the traps met so far and how to work around them.

## Measuring a transition

- **Frame 0 is reliable; later frames may not be.** Framer Motion applies the
  "start from the old place" transform synchronously when the layout changes,
  so a measurement after `await new Promise(r => setTimeout(r, 0))` shows
  where the transition starts. Anything later depends on
  `requestAnimationFrame`.
- **Hidden preview panes throttle `requestAnimationFrame`.** In an automated
  browser whose pane is hidden or in the background, animations barely
  advance: mid-animation samples look frozen, and an element can look stuck
  with a leftover transform. Take a screenshot (or otherwise bring the pane to
  the front) between steps, and treat only frame 0 and fully settled
  positions as evidence.
- **Hold elements by query, not by reference.** Re-find the moving element
  after a toggle; if the old reference is detached, the element was remounted.
- **Synthetic clicks may not reach the handler.** Some components only react
  to real pointer input. If `element.click()` does nothing, click with the
  browser automation at the element's on-screen coordinates (convert CSS px to
  screenshot px by the viewport/screenshot ratio).

## Simulating covered edges (notches, host chrome)

Set the host inset variables and draw the covered strips so they're visible
in screenshots:

```js
const r = document.documentElement.style;
r.setProperty('--host-inset-top', '112px');
r.setProperty('--host-inset-bottom', '120px');
const sim = document.createElement('div');
sim.innerHTML =
  '<div style="position:fixed;top:0;left:0;right:0;height:112px;background:rgba(255,0,0,.18);z-index:99999;pointer-events:none"></div>' +
  '<div style="position:fixed;bottom:0;left:0;right:0;height:120px;background:rgba(255,0,0,.18);z-index:99999;pointer-events:none"></div>';
document.body.append(sim);
```

Use a phone-sized viewport (e.g. 375×812), and re-apply after reloads. Also
check with nothing covered, so the fix doesn't change the normal layout.

## Tailwind (v4) in development

- **New arbitrary classes can lag in the dev server.** A freshly added class
  such as `h-[min(20dvh,180px)]` may not be in the dev CSS yet, so the element
  renders with the wrong size (e.g. 0 px tall) and looks like a layout bug.
  Check `document.styleSheets` for the rule, run a production build and grep
  its CSS, and touch the main CSS file or reload to regenerate.
- **Classes must appear literally in source.** Classes assembled from runtime
  values (`` `w-[${n}px]` ``) are never generated; use inline `style` for
  computed sizes.
