# Space Race — Relativistic Journey

> **Note:** This project (including this README file) was generated with AI assistance, obviously, via the Cascade / Windsurf agent through iterative prompting.

## What it is

A single-file 2D space game where the player (or an autopilot) flies a ship from a random starting point to Earth across a large 2D world. Depending on which world-options you enable, the trip may thread through a small cluster of gravitationally-interacting massive stars, dodge drifting asteroids, and negotiate a black hole. Special and general relativity are simulated end-to-end: the ship's onboard clock runs slower than Earth's when moving fast or deep in a gravity well, and the visible starfield is Doppler-shifted, relativistically aberrated, and intensity-beamed by the ship's velocity.

## How to run

1. Open `space-game.html` in any modern browser (double-click works).
2. That's it. No build, no server, no `npm install`.

The file loads Phaser 3.60 from the jsDelivr CDN, so an internet connection is required the first time you open it (browser caching handles subsequent loads).

## Menu

The mode-select screen has two sections:

**World options** (four checkboxes, off by default):
- **Massive objects (suns, black holes)** — spawn the star cluster (and, if enabled, one black hole). Without this the world is empty and every trip is a straight line.
- **Asteroids** — spawn drifting asteroids that can collide with the ship.
- **Tokens (fuel & ammo pickups)** — scatter collectible pickups.
- **Earth-time deadline (hard loss if exceeded)** — enforce a time budget on the trip (see Notable mechanics below).

**Modes**:
- **MANUAL** — fly the ship yourself with the keyboard.
- **AUTOPILOT** — the autopilot plans a slingshot trajectory (numerical sweep of departure angles under simulated gravity) and flies the ship to Earth automatically.
- **Ensemble test (20 autopilot runs)** — runs 20 fully-random autopilot trials back-to-back (each equivalent to clicking Autopilot with a new seed) and reports success rate, plus mean±sd of Earth-frame flight time, ship-frame flight time, time lost to relativistic dilation, and fuel used. A live top-right HUD tracks the running totals as trials complete. At the end, an **Adopt / Keep** dialog offers to persist the currently-active tuning parameters as the new defaults (or discard them).
- **Tuning (adjust parameters)** — opens a sliders panel for the autopilot's PID gains, cruise speed, Isp, deadline multiplier, etc. Values persist across page loads via `localStorage`.

## Controls (MANUAL mode)

- **Left / Right arrows** — rotate the ship
- **Up arrow** — main thruster
- **Space** — fire (if ammo > 0 and asteroids are enabled)
- **R** — restart current run after landing or crashing

## HUD & overlays

- Top-left: fuel, ammo, distance, radial/tangential velocities, attitude error, phase, etc.
- Below that: **Time-dilation panel** showing your clock, Earth's clock, current γ (Lorentz factor), and — when the Earth-time deadline is enabled — a color-coded Earth-time budget bar.
- Bottom-left: **Trajectory Map** — a large minimap showing the start point, Earth, every star (with live positions), the planned red trajectory, a dashed cyan free-fall ghost, and the ship's live position along the plan.
- Top-right (after a run finishes): **Retry Same Seed** / **New Seed** buttons.
- **Background starfield**: not static. At non-trivial velocity, stars are pulled toward the direction of motion (relativistic aberration), color-shifted (blue ahead / red behind), and re-brightened (relativistic beaming). At β > 0.9 the effect is dramatic.

## Star system

Each run generates eight identical massive stars, all sharing the same radius, mass, and gravitational parameter. Their t=0 positions and velocities are sampled with essentially no structure — the only constraints are physical safety:

- **Position** — uniform random over the whole world, rejection-sampled subject to (a) at least 10 star radii from Earth, (b) at least 5 star radii from the ship's spawn, (c) at least 4 star radii from any other star (numerical-stability floor for the N-body integrator).
- **Velocity** — a common bulk drift of 200 world-units/s perpendicular to the S→E baseline (90° CCW), plus a per-star isotropic random kick with Gaussian(mean=100, sd=50) speed. The result is further rejection-sampled so the star's straight-line trajectory stays at least 10 star radii from Earth over the full gameplay horizon. The bulk drift sweeps the whole ensemble across the trajectory corridor while the random component preserves internal diffusive motion.

That Earth-clearance filter (on both position and velocity) is what lets us safely neglect Earth-star gravitational coupling. The stars themselves then interact under full pairwise Newtonian gravity (symplectic Euler, both in the pre-flight trajectory sweep and in live gameplay).

## Notable mechanics

- **Numerical slingshot planner (N-body aware).** For AUTOPILOT / TEST runs, the game sweeps candidate departure angles in two cones on either side of the straight start→Earth line. Each candidate integrates the ship's trajectory alongside the full moving, gravitationally-interacting star cluster. It picks the fastest candidate whose intercept lands within the Earth-radius circle, and a finer refinement sweep tightens the winning angle.
- **Aligned launch + minimal ORIENT.** The ship spawns pre-aligned along the S-E baseline. The autopilot's ORIENT phase runs a single bang-bang attitude schedule (usually trivially short) and immediately hands off to BOOST; any residual attitude error is trimmed by the CRUISE PID controller in flight.
- **Retry same seed.** After a run, the exact start position, Earth position, and every star's `(x0, y0, vx, vy)` are snapshotted. "Retry Same Seed" reproduces the identical world; "New Seed" re-randomizes.
- **Relativistic clocks.** Ship time advances more slowly than Earth time as a function of velocity and gravitational potential. Both are shown on the HUD, and the "time lost" delta is reported at the end of successful runs. Game-unit speed of light is `c = 400`, so cruise speeds put you well into relativistic territory (β = 0.5 at cruise 200, β ≈ 0.975 at the vCruise maximum).
- **Relativistic visuals.** The rendered starfield each frame applies 2D relativistic aberration (`u′_∥ = (u_∥ + β)/(1 + β u_∥)`), Doppler color shift, and intensity beaming (`I ∝ D³`) based on the ship's current velocity.
- **Earth-time deadline (opt-in).** Toggle in the mode-select menu. When on, the ship must reach Earth within an Earth-frame time budget of `mult × (dist_S_to_E / vCruise)` (default multiplier 1.5). The budget bar is shown on the HUD; missing it is a hard loss. Gives real gameplay weight to the time-dilation mechanic: gravity dilation and slow trajectories both eat into your deadline, so the two clocks become an actual tradeoff instead of ambient decoration.
- **Optional black holes.** Toggleable in the menu. Contribute strongly to gravitational time dilation near the event horizon; grazing passes routinely produce γ > 5.

## Files

The entire game lives in a single file:

- `space-game.html` — HTML shell, inline CSS, and inline Phaser scene (~5000 lines of JS).

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
