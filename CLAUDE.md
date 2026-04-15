# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

The entire game is contained in a single `index.html` file. To play:
1. Open `/c/Users/janba/Dev/Asteroids/index.html` in any modern web browser
2. Press Spacebar to start
3. Use Arrow Keys or WASD to rotate/thrust, Spacebar/F to shoot

## Architecture Overview

The game is a **single-file HTML5/Canvas application** organized into logical sections (marked with `// ─── SectionName ` comments):

### Core Systems

- **Audio** (`initAudio`, `sndShoot`, `sndExplosion`, etc.)
  - Web Audio API-based procedural sound generation
  - Persistent thrust oscillator (started on first keypress to respect browser autoplay policy)
  - No audio files—all sounds generated in real-time

- **Game Loop** (`update`, `draw`, `loop`)
  - `update()`: handles input, physics, collisions, state transitions
  - `draw()`: renders everything with canvas
  - `requestAnimationFrame` for 60 FPS timing

- **State Management** (the `state` object)
  - `phase`: `'title'` | `'playing'` | `'levelup'` | `'gameover'` — `'levelup'` is a brief timed pause between levels driven by `phaseTimer`
  - `ship`: player ship object (position, velocity, angle, `invincible` countdown in frames)
  - `asteroids`, `bullets`, `particles`, `powerups`: pooled object arrays
  - `activeShield`, `activeRapidfire`, `activeTripleshot`: power-up timers (in frames, counted down each tick)

- **Controls** (`keydown`/`keyup` listeners)
  - `keys` object maps `e.code` → boolean for held keys
  - `initAudio()` is called on the first keydown to satisfy browser autoplay policy

- **HUD** (`updateHUD()`)
  - Updates DOM elements `#score`, `#lives`, `#hiscore` and toggles `.active` on `.pu-indicator` elements
  - Called after score/lives changes; power-up bar reflects active timers

- **Utils**
  - `wrap(v, max)`: toroidal screen wrapping for all moving objects
  - `dist(x1,y1,x2,y2)`: Euclidean distance used for collision and spawn-safe checks

### Game Features

- **Level Themes** (`THEMES` array, `getTheme()`)
  - 9 themed levels cycling through 3 aesthetics: Sector/Nebula/Inferno
  - Each theme defines colors, star density, and speed multiplier
  - Starfield regenerated per level based on theme

- **Power-Ups** (`POWERUP_DEFS`, `makePowerUp()`, `activatePowerUp()`)
  - Shield (cyan): temporary invincibility
  - Rapid Fire (orange): halved shot cooldown
  - Triple Shot (magenta): fire 3 bullets in fan pattern
  - 15% spawn chance when any asteroid is destroyed
  - Collected by ship proximity, shows active status in HUD bar

- **Particle System** (`spawnParticles()`, `spawnExhaust()`)
  - Explosions, exhaust, power-up collection, death burst
  - Each particle has position, velocity, life timer, optional color
  - Friction applied each frame for natural deceleration

- **Rendering with Neon Glow** (`glow()`, `noGlow()`)
  - Uses `ctx.shadowBlur` + `ctx.shadowColor` for all glowing elements
  - Ship, asteroids, bullets, power-ups have themed colors
  - Particles fade based on `life / maxLife` alpha

### Key Constants & Configs

| Config | Purpose |
|--------|---------|
| `SHIP_SIZE`, `SHIP_TURN`, `SHIP_THRUST` | Ship physics |
| `FRICTION` | Per-frame velocity decay (0.985) applied to ship and particles |
| `BULLET_SPEED`, `BULLET_LIFE`, `MAX_BULLETS` | Bullet behavior |
| `SHOT_COOLDOWN` | Fire rate in frames (halved by rapid-fire power-up) |
| `ASTEROID_SIZES` | Tiers: large/medium/small with radius, speed, score |
| `THEMES` | 9 level themes with colors, stars, speed multipliers |
| `POWERUP_DEFS` | 3 power-up types with color, symbol, duration |

Canvas is fixed at **900 × 650 px**. All timing is **frame-based** (targeting 60 fps via `requestAnimationFrame`); durations in comments or constants expressed as frames (e.g. `invincible: 180` = 3 s, power-up `life: 360` = 6 s).

## Workflow Notes

**Plan Mode**: Use `EnterPlanMode` before implementing non-trivial features or substantial changes. The user prefers reviewing approach before coding.

## Common Tasks

### Adding a New Feature
1. Use plan mode to design the approach
2. Identify which sections (Audio, Controls, State, Update, HUD, Draw, Utils) are affected
3. Update the relevant `// ─── SectionName ` block(s)
4. Test in browser by opening `index.html`

### Tweaking Audio
Look in the **Audio** section. Sound functions accept theme/config and use the Web Audio API to synthesize tones procedurally.

### Adjusting Difficulty
- Change `ASTEROID_SIZES[tier].speed` multipliers
- Modify `THEMES[i].speedMult` per level
- Adjust power-up `duration` values in `POWERUP_DEFS`
- Change power-up spawn chance in `explodeAsteroid()` (currently `Math.random() < 0.15`)

### Modifying Colors/Themes
- Edit `THEMES` array entries to change level aesthetics
- Update ship/bullet/particle colors in the **Draw** section
- `glow(color, blurRadius)` is used before drawing any glowing element
