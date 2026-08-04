# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A retro pixel-art, level-based arcade clicker. The entire game — HTML, CSS, canvas
rendering, game loop, sprite generation, level system, and a WebAudio chiptune
engine — lives in one self-contained file, `index.html`. No build step, no
dependencies, no package.json.

Ground-up rebuild of an earlier vibe-coded prototype (`../cow-clicker-original/` in
the parent directory, not part of this repo).

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server,
e.g. `python3 -m http.server` from this directory. There is no build/lint/test
command — it's plain HTML/CSS/JS, edited and reloaded directly.

### Debug modes

- `index.html?gallery` — renders every sprite variant and animation frame at a large
  fixed scale (add `&scale=N` to change zoom), for iterating on pixel art without
  playing through the game to see a given cow/frame.
- `window.__cowTick(ms)` — advances the simulation by `ms` and redraws, without
  waiting on `requestAnimationFrame`. Pair with `window.__cowState()` (mode, level,
  quota, kills, capacity, animal list) and `window.__cowSetLevel(n)` (jump straight
  to any level). Used to drive the game headlessly (e.g. from a browser automation
  tool) for testing spawn/click/level-clear/game-over flows without a real clock.
  `__cowTick` resyncs the frame clock to real time afterwards so live
  `requestAnimationFrame` frames never see a timestamp from the simulated future
  (dt is also clamped to ≥ 0 in the main loop for the same reason).

## Architecture

Everything is one IIFE in the `<script>` tag at the bottom of `index.html`, organized
top-to-bottom as: canvas/buffer setup, sprite generation, scenery, audio, game state,
input, game flow, main loop, boot. Reading it linearly in that order tracks the
runtime's own dependency order.

**Two-canvas pixel-perfect rendering.** There's a visible `#game-canvas` sized to the
real window, and an offscreen low-res buffer canvas (`buf`/`ctx`, default ~320x180,
resized in `resize()` based on window height). All game logic and drawing targets the
low-res buffer; each frame, `ctx`'s contents are blitted onto the visible canvas with
`drawImage` and nearest-neighbor scaling (`imageSmoothingEnabled = false`), which is
what gives every element — sprites, UI, background — a shared chunky pixel grid
regardless of actual window size. `pixelScale` is the integer scale factor between
the two; input coordinates must be divided by it (see `canvasToBuffer`) to land back
in buffer space.

**Sprites are generated at load, not loaded as images.** Each animal body is an ASCII
pixel-map (one string per row, one character per pixel) drawn facing left —
`COW_BODY_ROWS`, `CHICKEN_BODY_ROWS`, `PIG_BODY_ROWS`, `SHEEP_BODY_ROWS`;
`buildSprite()` rasterizes a map onto an offscreen canvas using a palette object that
maps characters to hex colors. Palette swaps (`HOLSTEIN_PALETTE`, `JERSEY_PALETTE`,
`SKY_PALETTE`, `GOLDEN_PALETTE`) reuse the same cow rows to produce the cow variants.
`flipSprite()` mirrors a left-facing sprite to get the right-facing version. Wings
(`WING_ROWS`, 3 flap frames) and the farmer (`FARMER_FRAMES`, 2 run frames) follow the
same pattern. `buildAllSprites()` runs once at boot and populates `sprites`.

**Legs, tails, and wings are drawn procedurally around the static body sprites**, not
baked in — `drawCow`/`drawChicken`/`drawPig`/`drawSheep` draw far legs, then the body
image, then near legs (cows also get a swaying tail; sky cows a wing), layering by
draw order to fake depth. Leg positions swing via `Math.sin(walkPhase + phase)`. Each
draw function hardcodes its own leg anchors and leg-top row to match its body map —
reshaping a body sprite means updating those anchors in its draw function (for cows:
`COW_LEG_ANCHORS` and `COW_LEG_TOP`). `drawAnimal()` dispatches on the species'
`family` and also handles the golden cow's despawn blink.

**Species behavior lives in one registry.** `SPECIES` maps each species key to its
family (which draw function), zone (`ground`/`sky`), hitbox, base speed, points,
walk-cycle rate, and behavior flags: `erratic` (chickens change direction on a
wander timer), `burst` (pigs multiply their speed briefly on a timer), `flock`
(sheep spawn 2–3 at once), `bonus`+`ttl` (golden cow doesn't count toward field
capacity and despawns after 7s, blinking near the end). `updateAnimals()` reads
these flags; there are no per-species update functions.

**The level system drives difficulty.** `LEVELS` is an array of ten scripted specs —
name, quota (pops to clear), base spawn interval, speed multiplier, weighted species
`mix`, and flags (`night` rebuilds the scenery in a dark palette, `stampede` runs a
periodic burst-spawn timer, `golden` allows the 5%-per-spawn bonus cow) — and
`levelSpec(n)` extends past them procedurally forever (rotating late-game mixes,
shrinking interval, growing speed and quota). Within a level, `levelTime` adds a
mild ramp on top. `levelState` (`running`/`clear`) is a sub-state of
`gameMode === "playing"`: clearing quota poofs the field, shows the interstitial,
and the Next Level button calls `startLevel(n+1)`; the overflow lose condition
(`fieldCount() >= capacity()`) checks non-bonus animals only.

**User difficulty settings multiply on top of level specs.** The `settings` object
(herdSpeed %, spawnRate %, capacity) is edited by three sliders in the settings
overlay (which pauses the game via the `paused` flag), persisted to localStorage
(`cowClickerSettings`), and applied inside `speedFor()`, `currentSpawnInterval()`,
and `capacity()` — level definitions never see them.

**Scenery is prerendered once, not redrawn per frame.** `buildScenery()` paints sky
bands, hills, treeline, barn, field, fence, and scattered grass/flowers onto a single
offscreen canvas (`scenery`) using a deterministic pseudo-scatter (index-based, not
`Math.random()`) so it's stable across resizes. It's rebuilt only in `resize()`. Only
clouds animate independently every frame (`drawClouds`), since they drift.

**Game state is a small set of module-level variables** (`gameMode`, `cows`,
`particles`, `floaters`, `score`, `hiScore`, `playTime`, `spawnTimer`) closed over by
every function in the IIFE — there's no state container or class. `gameMode` is one
of `"splash" | "playing" | "gameover"` and gates most per-frame behavior (spawning,
input handling, which scene draws). The splash screen runs its own lightweight
farmer-chases-cow loop (`updateIntro`/`drawIntro`) reusing the same cow-drawing code
with a single hardcoded `introCow`.

**Audio is a hand-rolled WebAudio step sequencer**, not audio files. `startMusic()`
schedules a `setInterval` that fires once per 16th-note step, walking fixed melody/
bass frequency arrays (`MELODY`, `BASS`) and synthesizing oscillators per note;
`updateMeter()` switches between `"normal"` and `"fast"` tempo based on herd density
each frame. One-shot effects (`playPop`, `playMoo`, `playSadTrombone`) build and fire
their own oscillator graphs on demand. `audioCtx` is created lazily on first user
gesture (`wakeAudio`) per browser autoplay policy.

**`hiScore` persists via `localStorage`** (`cowClickerHiScore`); everything else
(score, cow positions, difficulty ramp) resets each `startGame()`/restart and is
in-memory only.
