# 003 — Host chrome covers the app

- **Project / files:** frankie-tarot — `src/app/index.css` (`--safe-*`), `mcp/ui/index.tsx` (host insets)
- **Stack:** MCP Apps (`@modelcontextprotocol/ext-apps`) in ChatGPT, React 19, Tailwind 4
- **Date:** 2026-10-07

## Symptom
In ChatGPT's mobile app, fullscreen, the app's title sat under ChatGPT's top
bar ("Open F.Tarot", menu, ···) and the footer under its message box.

## Conditions
Fullscreen on a phone inside the host. Not in a normal mobile browser, where
only the notch matters.

## Measurement
`env(safe-area-inset-*)` is 0 inside the host's iframe. From the user's
screenshots (393 pt wide phone): host top bar + status bar ≈ 112 px, message
box + home indicator ≈ 120 px.

## Cause
The app only knew the device's safe areas. The host draws its own chrome over
the iframe and reports it separately: MCP Apps host context
`safeAreaInsets {top,right,bottom,left}`, and ChatGPT's `window.openai.safeArea.insets`
(changes announced by the `openai:set_globals` window event).

## Didn't work
—

## Fix
One set of variables, the larger of device and host:

```css
:root {
  --safe-top: max(env(safe-area-inset-top, 0px), var(--host-inset-top, 0px));
  /* …right, bottom, left alike */
}
```
```ts
function applyHostInsets() {
  const ctx = bridge.getHostContext();
  const reported = ctx?.safeAreaInsets ?? window.openai?.safeArea?.insets;
  const touch = ctx?.deviceCapabilities?.touch ?? matchMedia("(pointer: coarse)").matches;
  // If a touch host in fullscreen reports nothing, keep clear of what it visibly draws.
  const insets = reported ?? (ctx?.displayMode === "fullscreen" && touch ? ASSUMED : undefined);
  for (const side of ["top", "right", "bottom", "left"])
    insets ? root.style.setProperty(`--host-inset-${side}`, `${insets[side]}px`)
           : root.style.removeProperty(`--host-inset-${side}`);
}
bridge.onhostcontextchanged = applyHostInsets;
addEventListener("openai:set_globals", applyHostInsets);
```
Verified by simulating: set `--host-inset-top: 112px`, `--host-inset-bottom: 120px`
and draw translucent red strips over those areas, then walk every screen.

## Rule
The visible area is the viewport minus device *and* host chrome. Read the
host's reported insets, fold them into the same safe-area variables as the
notch, and test with the covered strips simulated.
