# Space Race — Relativistic Journey

> **Note:** This project (including this README file) was generated with AI assistance, obviously, via the Cascade / Windsurf agent through iterative prompting.

## What it is

A single-file 2D space game where the player (or an autopilot) flies a ship from a random starting point to Earth across a large 2D world, threading through a small cluster of massive stars that pull on the ship and on each other. Time dilation is simulated: the ship's onboard clock runs slower than Earth's when moving fast or deep in a star's gravity well.

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
- Bottom-left: **Trajectory Map** — a large minimap showing the start point, Earth, every star (with live positions), the planned red trajectory, a dashed cyan free-fall ghost, and the ship's live position along the plan.
- Top-right (after a run finishes): **Retry Same Seed** / **New Seed** buttons.

## Star system

Each run generates eight identical massive stars, all sharing the same radius, mass, and gravitational parameter. Their t=0 positions and velocities are sampled with essentially no structure — the only constraints are physical safety:

- **Position** — uniform random over the whole world, rejection-sampled subject to (a) at least 10 star radii from Earth, (b) at least 5 star radii from the ship's spawn, (c) at least 4 star radii from any other star (numerical-stability floor for the N-body integrator).
- **Velocity** — isotropic direction over `[0, 2π)`, Gaussian speed (mean 200, sd 50, clamped positive), further rejection-sampled so the star's straight-line trajectory stays at least 10 star radii from Earth over the full gameplay horizon.

That Earth-clearance filter (on both position and velocity) is what lets us safely neglect Earth-star gravitational coupling. The stars themselves then interact under full pairwise Newtonian gravity (symplectic Euler, both in the pre-flight trajectory sweep and in live gameplay).

## Notable mechanics

- **Numerical slingshot planner (N-body aware).** For AUTOPILOT / TEST runs, the game sweeps candidate departure angles in two cones on either side of the straight start→Earth line. Each candidate integrates the ship's trajectory alongside the full moving, gravitationally-interacting star cluster. It picks the fastest candidate whose intercept lands within the Earth-radius circle, and a finer refinement sweep tightens the winning angle.
- **Aligned launch + minimal ORIENT.** The ship spawns pre-aligned along the S-E baseline. The autopilot's ORIENT phase runs a single bang-bang attitude schedule (usually trivially short) and immediately hands off to BOOST; any residual attitude error is trimmed by the CRUISE PID controller in flight.
- **Retry same seed.** After a run, the exact start position, Earth position, and every star's `(x0, y0, vx, vy)` are snapshotted. "Retry Same Seed" reproduces the identical world; "New Seed" re-randomizes.
- **Relativistic clocks.** Ship time advances more slowly than Earth time as a function of velocity and gravitational potential. Both are shown on the HUD, and the "time lost" delta is reported at the end of successful runs.

## Files

The entire game lives in a single file:

- `space-game.html` — HTML shell, inline CSS, and inline Phaser scene (~4500 lines of JS).

That's the whole project. No other files are required.

## Physics approximations

Explicit simplifications made in the current build:

- **Earth-star gravitational coupling is neglected.** Enforced by the Earth-clearance placement filter (see Star system).
- **Symplectic-Euler integration** for both the ship and the N-body star cluster, with no softening term. This is stable at typical star separations but can produce numerical artefacts on very close star encounters (rare given the placement filters).
- **Trajectory sweep** uses a fixed 40 ms integrator step, while live gameplay uses the browser's variable frame `delta`. Small tracking mismatch is absorbed by the cross-track PID.
- **No collision resolution between stars.** Stars pass through each other geometrically; only their gravity interacts.

## Caveats

- Because it was AI-generated iteratively, the codebase has stylistic inconsistencies, comments that reference now-deleted features, and defensive checks that may be dead code. It works, but it is not architected for long-term maintenance.
- The Phaser CDN link pins version 3.60.0. If that URL ever goes away, update the `<script src="...">` in `space-game.html`.
