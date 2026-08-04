# Cow Clicker v1

Ground-up rebuild of the original cow-clicker (see `../cow-clicker-original/`).
One self-contained file, no dependencies: open `index.html` in a browser, or serve
the folder with any static server.

## Gameplay

Cows wander the field (some fly). Click one to pop it and score a point. Cows
spawn faster the longer you last; if 30 are on screen at once, the herd wins and
it's game over. The chiptune speeds up when the herd meter goes red. Hi-score is
kept in localStorage.

## What changed from the original

- True low-res pixel buffer scaled to the window (instead of drawing scaled
  rects on a full-res canvas), so everything shares one chunky pixel grid.
- Redrawn cows with more realistic proportions: tapered head with eye, nostril,
  horn and ear, hip bone, deeper chest, udder with teats between the rear legs,
  swaying tail, 4-leg walk cycle. Three variants: Holstein, Jersey (brown), and
  the winged sky cow with a 3-frame flap.
- Richer scenery: barn, treeline, fence, drifting clouds, flowers.
- Added: hi-score, "+1" score floaters, moo sound on spawns, noise hats in the
  fast-tempo music, taller farmer in the title-screen chase.

## Debug

- `index.html?gallery` — renders all sprite variants and animation frames large
  (`&scale=N` to change zoom) for art iteration.
- `window.__cowTick(ms)` advances the simulation manually; `window.__cowState()`
  returns game state. Used for headless testing.
