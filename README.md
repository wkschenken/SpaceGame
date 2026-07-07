# Space Race — Relativistic Journey

> **Note:** This project was generated with AI assistance (via the Cascade / Windsurf agent). Every line of code, every design decision, and this README itself was produced through iterative prompting rather than written from scratch by a human. Treat it as a demo of AI-driven game development, not a hand-crafted production codebase.

## What it is

A single-file 2D space game where the player (or an autopilot) flies a ship from a random starting point to Earth across a large 2D world, optionally using a massive star's gravity for a slingshot assist. Time dilation is simulated: the ship's onboard clock runs slower than Earth's when moving fast or deep in the star's gravity well.

## How to run

1. Open `space-game.html` in any modern browser (double-click works).
2. That's it. No build, no server, no `npm install`.

The file loads Phaser 3.60 from the jsDelivr CDN, so an internet connection is required the first time you open it (browser caching handles subsequent loads).

## Modes

- **MANUAL** — fly the ship yourself with the keyboard.
- **AUTOPILOT** — the autopilot plans a slingshot trajectory (numerical sweep of departure angles under simulated gravity) and flies the ship to Earth automatically.
- **TEST** — batch experiment that runs many autopilot trials at a sweep of approach angles and reports success rates. Used for tuning the autopilot.

## Controls (MANUAL mode)

- **Arrow keys** — rotate / thrust
- **Space** — fire (if the mode / build includes weapons)
- **R** — restart current run after landing or crashing

## HUD & overlays

- Top-left: fuel, ammo, distance, radial/tangential velocities, attitude error, phase, etc.
- Bottom-left: **Trajectory Map** — a minimap showing the start point, Earth, the star, the planned red trajectory, and the ship's live position along it.
- Top-right (after a run finishes): **Retry Same Seed** / **New Seed** buttons.

## Notable mechanics

- **Numerical slingshot planner.** For AUTOPILOT / TEST runs, the game sweeps candidate departure angles in two cones on either side of the straight start→Earth line, integrates each trajectory under gravity, and picks the fastest one whose intercept lands within the Earth-radius circle. A finer refinement sweep tightens the winning candidate.
- **Retry same seed.** After a run, the exact start position, Earth position, and star position are snapshotted. "Retry Same Seed" replays that identical world; "New Seed" re-randomizes.
- **Relativistic clocks.** Ship time advances more slowly than Earth time as a function of velocity and gravitational potential. Both are shown on the HUD, and the "time lost" delta is reported at the end of successful runs.

## Files

The entire game lives in a single file:

- `space-game.html` — HTML shell, inline CSS, and inline Phaser scene (~4000 lines of JS).

That's the whole project. No other files are required.

## Caveats

- Because it was AI-generated iteratively, the codebase has stylistic inconsistencies, comments that reference now-deleted features, and defensive checks that may be dead code. It works, but it is not architected for long-term maintenance.
- The Phaser CDN link pins version 3.60.0. If that URL ever goes away, update the `<script src="...">` in `space-game.html`.
