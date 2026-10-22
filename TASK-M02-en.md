# TASK-M02 · Act II-A "ENTRY" (16–28s)

> Frozen interfaces: everything delivered and accepted in M0 + M01 (timeline engine, camera system, UI shell, audio skeleton, render pipeline, Act I shots 0–16s, Mars shader, spacecraft group with named children `hull`/`serviceModule`/`antenna`/`heatshield`/`legs`, cockpit frame, SELFTEST protocol). Do NOT modify accepted behavior — ADD through the extension points only. 0–16s must remain **pixel-identical**; if you believe a frozen interface must change, state it in delivery notes instead of changing it.
> Deliver a complete single file as `M02-v1.html` with the self-check list.

---

## 0. Scope

Fill t ∈ [16, 28] with real content (shots 4–5 + blackout). t ∈ [28, 46] stays black (M3 content later). Everything is a **pure function of t** — zero per-frame randomness (`Math.random()` per frame is banned; deterministic hash of t is fine).

## 1. Shot script (compositions are acceptance criteria)

### SHOT 4 · 16.0–21.0s — "Slamming into the sky" (hard cut from cockpit)
- **Exterior** view: the lander descends **heatshield-first** (orient the ship group so the `heatshield` child faces the velocity vector — down/toward Mars). Antenna may remain attached (jettison is M3).
- Background: black sky, Mars haze starting to glow at frame bottom. Stars fade to faint (upper third only) as plasma brightens.
- **Plasma sheath (the money shot of this shot):** a glowing additive layer hugging the heatshield's leading edge — white-hot core → #FFB566 mid → #FF5A26 outer, fading to nothing. Implementation suggestion: a shader sheath (disc/dome conforming to the heatshield silhouette, BackSide fresnel falloff) + streaming plasma streak particles (≤1200, positions = f(t, fixed per-particle seed), streaming backward past the hull, stretching with speed).
- **Intensity envelope (quantified):** 0.0 at t=16.0 → ramps to 1.0 by t=19.0 → flicker ±15% between 19.0–20.8 (flicker = deterministic hash noise of t, 12–18 Hz character) → **hard cut at t=21.0**.
- **Camera shake (quantified, deterministic — NO per-frame random):** amplitude 0 at 16.0 → max at t≈20.6 (position ±0.06 units, rotation ±0.8°) → cut at 21.0. Waveform = fixed-phase sum of sines, e.g. 1.7 / 3.1 / 5.3 Hz mixed; phases are named CONFIG constants.
- **HUD (inherits M1 styling):** velocity/altitude still ticking down but **jittering**: add ±3% deterministic jitter (hash of floor(t×8)) and occasional flicker of the whole HUD block (opacity dips, ≤0.3 dips, 2–3 times across the shot). Values continue the M1 deceleration profile (no discontinuity at 16s).
- Camera: tracking the ship, slow drift, plus the shake above.

### BLACKOUT · 21.0–22.0s
- **Pure black** (letterbox only, grain off or near-off), no scene, no HUD.
- Centered monospace text `SIGNAL LOST` — letterspacing 0.35em, color #D8CFC0 at 65% opacity. Fade in 21.2→21.6, hold, fade out 21.8→22.0.
- This is the M0 blackout mechanism returning (its trigger was parked in M01 — wire it back; do not rewrite the mechanism).

### SHOT 5 · 22.0–28.0s — "The canopy blooms" (signal restores)
- **Signal restore:** exposure ramps 0 → 1 over 22.0–22.45 (ease-out); grain strength ×3 ONLY for t ∈ [22.0, 22.15] (two-frame static spike), then back to normal.
- **Backdrop (new):** high-altitude Mars haze — vertical gradient dome or large backdrop plane: #2A1B14 low → #C97B4E mid → #E8A97E high, plus 2–3 faint horizontal haze bands (alpha ≤ 0.15). Stars very faint up top (reuse the starfield, low opacity). **No ground mesh** — bottom of frame dissolves into haze.
- **Chute deploy (quantified):**
  - t=22.0–22.2: mortar ejection — a small packed-chute streak pops upward from the hull top (fast translate + scale, 0.2s)
  - t=22.2–23.3: canopy inflation — scale from streamer to full, curve with slight overshoot (×1.06 near 90% then settle); final canopy radius ≈ 2.2 × hull width
  - Canopy: low-poly dome-ish silhouette, off-white/tan #D9C6A8 with darker shading, backlit (silhouette-first); 4 shroud lines from canopy edge to hull top (single LineSegments or thin cylinders)
- **Swing (quantified):** assembly pendulum about the canopy point: θ(t) = 6° · e^(−0.55·(t−22.8)) · sin(1.4·(t−22.8)) for t > 22.8, 0 before. Visually settled by ~27s.
- Whole assembly drifts slowly downward in world space across the shot.
- Camera: pull back from close (ship fills ~1/3 frame height at 22.5s) to wide (full canopy + lines + ship in frame by 26s, ship ≈ 1/8 frame height), gentle ease.
- HUD: absent 21.0–22.4; flickers back 22.4–23.0 (opacity 0→1 with 2 dips), then steady monotonic tick-down, M1 styling.
- Intent: relief after the blackout. The canopy blooming is the emotional beat — give the inflation the limelight, keep everything else quiet.

## 2. System specs

**A. Determinism** — plasma flicker, camera shake, HUD jitter, particle positions: all pure functions of t with fixed seeds/phases (CONFIG constants). Seek to any t → identical frame.

**B. Audio (skeleton cues only, real score is M5)** — wire via the frozen AudioEngine contract (only naturally-crossed cues fire; seek-safe):
- 21.0: entry roar hard-cuts to static noise bed (21.0–22.0 only)
- 22.0: chute mortar thump (short, punchy)
- 22.4: wind noise bed fades in, stays through 28s

**C. SelfTest extension** — extend the existing `?selftest=1` targets to `[9, 12, 15.5, 18, 21.5, 25]`:
- t=9/12/15.5: keep the M1 fields (rim NDC + drawCalls)
- t=18: `{"t":18,"plasmaCount":<alive>,"shakeAmp":<units>,"drawCalls":N}`
- t=21.5: `{"t":21.5,"blackout":true,"drawCalls":N}`
- t=25: `{"t":25,"chuteScale":<0–1+>,"swingDeg":<deg>,"drawCalls":N}`

## 3. Extension points (same as M1)

- New CONFIG blocks for Act II-A (colors, envelopes, timings, seeds) — no magic numbers inline
- `CAMERA_MOVES` gets shots 4–5 entries; blackout handled through the parked blackout mechanism
- New assets (plasma sheath, plasma particles, chute group, backdrop) attach via `initActContent` / `updateActContent` — do not restructure frozen sections
- Ship orientation per shot = pure function of t set in `applyState`

## 4. Performance budget

- plasma particles ≤ 1200 (1 draw call) + sheath shader ≤ 2 draw calls
- chute (canopy + lines) ≤ 2 draw calls; backdrop ≤ 2
- Total scene draw calls ≤ 40 (unchanged); no textures beyond 1024; keep 60fps headroom

## 5. Anti-patterns (repeated, binding)

Purple-blue gradient AI look / emoji icons / uniform card layouts / full-page single background color / per-frame random camera shake / setTimeout-scheduled animation / scattered magic numbers — all banned.

## 6. Self-check list (verify manually, attach results)

1. Console clean on file:// open
2. t=16 hard cut: cockpit → exterior, ship heatshield-first, no discontinuity in HUD values
3. Plasma: ramps per envelope, flickers 19–20.8, hard cut at 21.0; particles stream backward
4. Camera shake perceptible but never nauseating, zero outside 16–21
5. Blackout exact: 21.0 cut, SIGNAL LOST per spec, nothing else visible
6. Restore: exposure ramp, grain spike only 22.0–22.15
7. Chute: ejection → inflation with overshoot → settled by 27s; shroud lines taut; swing decays per formula
8. HUD: jitter in shot 4, absent in blackout, flicker-back 22.4–23.0, monotonic after
9. 28–46s stays black
10. Seek determinism: pause at 18s and 25s, seek away and back, identical frames
11. 0–16s pixel-identical to M01 accepted state
12. SELFTEST output lines for all six targets
13. Line count + section index + draw-call counts reported

## 7. Deliverable

`M02-v1.html` complete single file + self-check results + SELFTEST console output. QA runs independent pixel verification on all segments (plasma envelope sampling, blackout purity, inflation curve, swing decay, determinism, Act I regression).
