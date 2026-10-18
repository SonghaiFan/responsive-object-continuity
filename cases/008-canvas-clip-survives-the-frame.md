# 008 — Canvas clip survives the frame

- **Project / files:** frankie-tarot — `src/features/tarot/scene/painters.ts` (`paintPlanet`), `scene/SkyScene.tsx`
- **Stack:** Canvas 2D, motion 12 (camera values), React 19
- **Date:** 2026-10-07

## Symptom
After the camera moved from the home shot to the next stage's closer shot,
the planet's glow was cut off by a vertical edge and a horizontal edge, as if
painted into a rectangle.

## Conditions
Only after the camera had moved; the first (home) frame was fine. A full
reload into the new stage would also have been fine.

## Measurement
The horizontal edge sat at screen y 348 = 0.58 × 600 — exactly where the
horizon was on screen in the *first* shot, not the current one.

## Cause
The planet is drawn in a reused scratch canvas, clipped to "above the
horizon" in world coordinates through the camera transform. `clip()` was
called every frame without `save()`/`restore()`. Clip regions intersect and
persist on a canvas context (`clearRect` and `setTransform` don't reset them),
so every later frame was also clipped to the first frame's region.

## Didn't work
— (a stale painter from hot reload made the first fix look ineffective;
reload before judging a canvas fix, see references/tooling.md).

## Fix
```ts
layer.save();
applyView(layer, width, height, view, dpr);
layer.beginPath();
layer.rect(/* above the horizon, in world px */);
layer.clip();
// …draw…
layer.restore();
```

## Rule
On a reused canvas, scope every `clip()` (and any state you don't reset
explicitly) inside `save()`/`restore()`; a camera that moves exposes clips
left over from earlier frames.
