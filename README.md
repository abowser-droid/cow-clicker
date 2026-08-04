# Cow Clicker v1

Ground-up rebuild of the original cow-clicker (see `../cow-clicker-original/`).
One self-contained file, no dependencies: open `index.html` in a browser, or serve
the folder with any static server.

Play it at [abowser-droid.github.io/cow-clicker](https://abowser-droid.github.io/cow-clicker/).

## Gameplay

A level-based arcade clicker. Each level sets a quota of animals to pop; clear it
before the field fills to capacity, or the herd wins. Spawns and speeds ramp within
each level and from level to level.

The first ten levels are scripted: ground Holsteins, then flying cows, then faster
brown Jerseys, then chickens (small and erratic), pigs (speed bursts), and sheep
(arrive in flocks) — some levels single-species, some mixed. Surprises along the
way: a rare golden cow worth +10 that leaves on its own, stampede waves (announced
by a rumble and a dust cloud on the edge they'll charge in from), a starlit
night level, and a four-voice chiptune of Chopin's Marche funèbre (Piano Sonata
No. 2, Op. 35, third movement) that plays before Total Farmageddon begins. Past
level 10 the game generates ever-harder rounds forever. Points vary by species;
each cleared level adds a bonus, and the clear screen auto-advances after a
4-second countdown. Game over offers Continue (replay the level you lost on,
score reset — arcade rules) or Start Over from level 1. Hi-score is kept in
localStorage.

## Settings

The Settings button opens four sliders (persisted in localStorage):

- **Herd Speed** (50–200%) — how fast animals move
- **Spawn Rate** (50–200%) — how quickly they arrive
- **Field Capacity** (15–50) — how many fit before game over; lower is harder
- **Level Quota** (50–200%) — scales how many pops each level takes to clear

## What changed from the original

- True low-res pixel buffer scaled to the window (instead of drawing scaled
  rects on a full-res canvas), so everything shares one chunky pixel grid.
- Redrawn cows with more realistic proportions and a 4-leg walk cycle, plus
  chicken, pig, sheep, and golden-cow sprites in the same ASCII-pixel-map style.
- Level system with quotas, interstitials, and procedural post-10 difficulty
  (the original was a single endless round).
- Species-specific pop sounds (cluck, oink, bleat, chime) on top of the chiptune,
  which gains tempo with level and escalates with herd pressure: the tune turns
  minor when the herd meter goes yellow, and in the red zone it becomes a
  pulsating two-tone alarm while the whole scene pulses red.
- Touch hardened for phones: pinch-zoom, double-tap zoom, scroll-bounce, and
  long-press callouts are all blocked.
- Richer scenery: barn, treeline, fence, drifting clouds — with a night variant.

## Debug

- `index.html?gallery` — renders all sprite variants and animation frames large
  (`&scale=N` to change zoom) for art iteration.
- `window.__cowTick(ms)` advances the simulation manually; `window.__cowState()`
  returns game state (level, quota, kills, animals); `window.__cowSetLevel(n)`
  jumps straight to a level. Used for headless testing.
