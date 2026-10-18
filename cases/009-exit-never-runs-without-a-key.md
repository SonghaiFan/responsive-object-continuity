# 009 — Exit never runs without a key

- **Project / files:** frankie-tarot — `src/app/App.tsx` (`renderPhase`), `src/features/tarot/components/IntroSection.tsx`
- **Stack:** React 19, motion 12 (`AnimatePresence mode="sync"`)
- **Date:** 2026-10-07

## Symptom
Pressing Start, the home page's card should fly off to the top right while the
next stage appears; instead it vanished at once. The page's own fade-out exit
had never played either.

## Conditions
Every stage change out of the home page. Exits on the stage components'
*root* elements (`<motion.div key="intro" exit={…}>`) were all ignored.

## Measurement
Sampling the card's rect every 200 ms after the click: present at 0 ms,
`null` at 200 ms. After the fix: centre (512,327) → (797,78), width 202 → 135,
opacity falling, gone at ~1.4 s.

## Cause
`AnimatePresence` tracks its **direct** children by `key`. `renderPhase()`
returned `<IntroSection/>`, `<InputSection/>`, … without keys, so every stage
had the same (absent) key: AnimatePresence saw one child that merely
re-rendered, and React swapped the component in place. The `key` inside each
component's root is invisible to AnimatePresence.

## Didn't work
—

## Fix
Key the element AnimatePresence sees:

```tsx
case GameState.INTRO:
  return <IntroSection key="intro" … />;
```
Only the home page was keyed, to switch on its exit without changing how the
other stages (still unkeyed, still swapping) behave; keying them too would
enable their exits and is a separate change to test.

## Rule
For an exit to play, the element passed to `AnimatePresence` must have a key
that differs between the things it switches between — on the element in the
switch, not on the component's own root.
