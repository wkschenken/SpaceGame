# Space Race — Relativistic Journey

> **Note:** This project (including this README file) was generated with AI assistance, obviously, via the Cascade / Windsurf agent through iterative prompting.
> A small error in the simulator causes some trajectories to lag behind the expected trajectory, sometimes causing the ship to crash. The cause for this has not been found. 

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
- **AUTOPILOT** — the autopilot first tests whether the direct start→Earth chord clears every star; if it does, it flies straight to Earth with only cross-track PID corrections for gravitational tugs from off-path stars. If the chord clips a star, it falls back to a numerical slingshot sweep and flies the resulting curved trajectory. A short-lived top-center banner announces which of the two paths was chosen (`Straight line path has been cleared` / `Straight line trajectory not safe. Calculating slingshot...`).
- **Ensemble test...** — pops up a prompt for the number of runs (1–500, default 20), then runs that many fully-random autopilot trials back-to-back (each equivalent to clicking Autopilot with a new seed) and reports success rate, plus mean±sd of Earth-frame flight time, ship-frame flight time, time lost to relativistic dilation, and fuel used. A live top-right HUD tracks the running totals as trials complete. At the end, an **Adopt / Keep** dialog offers to persist the currently-active tuning parameters as the new defaults (or discard them).
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

- **Position** — uniform random over a rectangle whose area is ~23% of the full world (linear factor ~0.48 per axis), centered on the S-E midpoint and then **pre-offset opposite to the bulk-drift direction** by `perpDriftMag · |SE| / (4·vCruise)`. This "leads" the drift: the star ensemble sweeps across the play corridor and its mean coincides with the S-E midpoint at roughly the moment the ship (flying at ~vCruise) reaches the ~1/4 point of the chord, so the star density peaks right where and when the ship is most exposed. The final center is clamped to keep the box fully inside the world bounds. Rejection-sampled subject to (a) at least 10 star radii from Earth, (b) at least 5 star radii from the ship's spawn, (c) at least 4 star radii from any other star (numerical-stability floor for the N-body integrator).
- **Velocity** — a common bulk drift of 200 world-units/s perpendicular to the S→E baseline (90° CCW), plus a per-star isotropic random kick with Gaussian(mean=100, sd=50) speed. The result is further rejection-sampled so the star's straight-line trajectory stays at least 10 star radii from Earth over the full gameplay horizon. The bulk drift sweeps the whole ensemble across the trajectory corridor while the random component preserves internal diffusive motion.

That Earth-clearance filter (on both position and velocity) is what lets us safely neglect Earth-star gravitational coupling. The stars themselves then interact under full pairwise Newtonian gravity (symplectic Euler, both in the pre-flight trajectory sweep and in live gameplay).

## Notable mechanics

- **Two-stage path planner.** For AUTOPILOT / TEST runs, the planner runs in two stages:
  1. **Full-N-body chord-clearance check (fast path).** Run the exact same trajectory integrator the slingshot sweep uses on every candidate, but with a single candidate: heading angle 0 (ship departs straight along S→E). The sim starts the ship at rest, applies BOOST-phase thrust with relativistic acceleration and fuel-mass drain until it hits `vCruise`, then coasts; simultaneously advances all N massive objects under full pairwise Newtonian gravity via symplectic Euler; and tracks minimum center-to-center approach distance to any object over the full trajectory. If the ship reaches Earth (impact-radius test) and its minimum approach to any object stays above `objRadius + shipRadius + 20`, install a synthetic straight-line plan (a 100-point S→E polyline) and skip the sweep. Because both the ship and the stars are simulated with the same physics as the slingshot sweep, the fast-path verdict is bit-for-bit consistent with what the sweep would return for `angle = 0` — there is no linear-motion approximation anywhere in the check. The 20-unit margin still buffers live-flight PID drift and star-motion divergence from a subtly different `dt`.
  2. **Numerical slingshot sweep (fallback).** If the chord clips a star, the game sweeps candidate departure angles in two cones on either side of the S→E line. Each candidate integrates the ship's trajectory alongside the full moving, gravitationally-interacting star cluster (symplectic Euler, pairwise Newtonian gravity for the star cluster, ship as a test particle under the stars' combined gravity). The ship model has two phases: **BOOST** (constant main-engine thrust `F = 2 P_jet / v_e` along the candidate departure direction, ship mass shrinking as fuel burns at the ideal rocket-equation rate `ṁ = F / v_e`, acceleration computed relativistically — see **Relativistic thrust** below) until it reaches cruise speed or exhausts its fuel, then **ballistic coast** under gravity only. The planner picks the fastest candidate whose intercept lands within the Earth-radius circle, and a finer refinement sweep tightens the winning angle. CRUISE-phase PID corrections and the terminal BRAKE burn are not modeled in the sweep — those are handled by the live-flight controller.
  
  This ordering matters: straight-line flight is faster (shorter arc), cheaper in fuel (only PID corrections), and produces less time dilation (no deep gravity-well transit), so it's the preferred plan whenever it's safe. The slingshot planner is only invoked when the direct chord is geometrically blocked.
- **Planning-latency compensation (station-keeping + pre-roll).** Planning is not free: the numerical sweep takes real wall-clock time during which the star cluster is actively evolving under N-body gravity. To make the plan match the world it will actually fly through, the ship engages a **station-keeping controller** the moment the sweep starts: it computes the local gravitational acceleration on the ship each frame and thrusts opposite it, cancelling gravity so the ship coasts inertially (from rest, for the initial plan) in a force-free frame. Fuel is drawn at the ideal rocket-equation rate `ṁ = m·|g_net| / v_e`, so hovering next to a star during a long planning window has a real cost. Meanwhile the sweep itself **pre-rolls the star ensemble** forward by an estimated `T_plan_est` seconds using the same pairwise-gravity + symplectic-Euler integrator; the sweep's `t = 0` state is that post-roll snapshot, so every candidate is scored against the world state that will exist at hand-off. When the sweep finishes, the actual planning time `T_plan_actual` is measured and the winning heading is **refit against the true live star state** with one extra ~0.5 ms simulation call, so the polyline the autopilot actually flies is anchored to reality (not the pre-roll estimate). At that point station-keeping releases, autopilot enables, and BOOST engages from the ship's actual current state.
  - **Estimate source.** `T_plan_est` is the running mean of the last 5 measured `T_plan_actual` values, defaulting to 500 ms until at least one measurement exists. Because the numerical sweep is dominated by animation-frame count (2 candidates/frame at 60 fps, so a 600-candidate sweep takes ~5 s of wall clock, not CPU), that measurement is much larger than the CPU-only planning time — and the rolling average tracks it correctly.
  - **Persistence across scene restarts.** The rolling-average buffer lives at *module scope*, not on the scene instance. Since `refineSlingshotPlanNumerically()` runs exactly once per scene lifetime and every retry / new-seed rebuilds the scene, a per-scene buffer would reset to empty on every plan and the estimate would be stuck at the 500 ms default forever. The module-scope buffer means: first plan of a page load uses the 500 ms default (expect a large est-vs-actual mismatch), the second plan uses the first plan's actual (~4-5 s), and steady-state estimates converge within ~2 retries. A hard page reload clears the history.
  - **HUD.** After each hand-off, a top-screen banner reports `Plan ready in T.TT s (est E.EE s) -- station-keeping fuel: F.FF kg` so the estimate/actual convergence is directly visible across retries. The fuel figure is cumulative across any fail-retune-replan cycles within the run.
  - **Live retune on plan failure.** When the sweep fails to find a viable trajectory (or the accepted plan misses Earth at hand-off reconciliation), station-keeping is *not* released — the ship keeps hovering on thrusters while a "Retune & Replan (live)" button is added to the retry panel. Clicking it opens an in-scene slider overlay for `vCruise`, PID gains, brake distance, etc.; sliders live-write `activeTuningParams`. Meanwhile stars keep evolving under N-body gravity and fuel keeps burning against the local gravity gradient (real time — the world does not pause). Apply tears down the failure UI, resets the sweep state, and re-invokes the planner against a fresh live snapshot of the world. The player can iterate until a plan converges, or fall back to "Retry Same Seed" / "New Seed" (both restart the scene). Fuel runout during station-keeping releases the controller and the ship falls under normal gravity — a natural pressure to commit.
- **Aligned launch + minimal ORIENT.** The ship spawns pre-aligned along the S-E baseline. The autopilot's ORIENT phase runs a single bang-bang attitude schedule (usually trivially short) and immediately hands off to BOOST; any residual attitude error is trimmed by the CRUISE PID controller in flight.
- **Retry same seed.** After a run, the exact start position, Earth position, and every star's `(x0, y0, vx, vy)` are snapshotted. "Retry Same Seed" reproduces the identical world; "New Seed" re-randomizes.
- **Relativistic clocks.** Ship time advances more slowly than Earth time as a function of velocity and gravitational potential. Both are shown on the HUD, and the "time lost" delta is reported at the end of successful runs. Game-unit speed of light is `c = 400`, so cruise speeds put you well into relativistic territory (β = 0.5 at cruise 200, β ≈ 0.975 at the vCruise maximum).
- **Relativistic thrust.** Both live-flight thrust (main engine + lateral side-thrusters, MANUAL and AUTOPILOT alike) and the planner's BOOST-phase simulation apply Newton's second law in its relativistic form: `F = dp/dt` with `p = γmv`. Decomposing the lab-frame thrust force into components along and perpendicular to the ship's current velocity, the resulting acceleration is `a_∥ = F_∥ / (mγ³)` and `a_⊥ = F_⊥ / (mγ)`. At rest this reduces to `a = F/m`; as `β → 1` longitudinal acceleration vanishes so the ship asymptotes to `c` instead of blowing past it. All four thrust sites (autopilot main, autopilot lateral, manual main, planner BOOST) share a single `relativisticThrustAccel(Fx, Fy, m, vx, vy)` helper. Fuel burn rate is unaffected — that's a rocket-equation quantity, not a kinematics one.
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
