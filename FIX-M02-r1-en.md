# FIX-M02 · Rectification Round 1

> QA update on M02-v1. What worked: **blackout segment is perfect** (pure black + centered SIGNAL LOST, exact 21.0 hard cut — plasma warm px 133 → 0 across the boundary), shake is properly deterministic (phased sines), swing decay matches the prescribed formula exactly (0.11° at t=25), SELFTEST module is clean, Act I frames at t=6.5/12 are pixel-identical, paused double-frame IDENTICAL at 18s and 25s.
> But Shot 4's hero visual (plasma) and Shot 5's composition both failed. Root causes below are runtime-proven, not guesses.

| # | Severity | Issue |
|---|---|---|
| FIX-A | **BLOCKER** | Plasma invisible: sheath occluded by the heatshield itself; only two ~2×8px flecks escape at the rim (warm px: 153 @18.5s, 389 @20.5s peak vs required ≥8000) |
| FIX-B | **BLOCKER** | Haze backdrop is a frame-filling blinding slab: gradient inverted (brightest #E8A97E at TOP), plane edges visible as hard rectangle, bloom blows the top half pure white — the deployed chute (correctly positioned at (0,3.35,−8)!) drowns inside the white |
| FIX-C | **BLOCKER** | sunGlow (r=5 sphere at (50,15,−25)) visible at top-right corner in Act II — no visibility rule covers 16–28s |
| FIX-D | QUALITY | ACT2 starfield fade is a **no-op**: `starfield.material.opacity = 0.12` — the starfield is a custom ShaderMaterial, `.opacity` is ignored; stars render full brightness at 18.5/20.5s |
| FIX-E | QUALITY (QA-authorized) | Starfield positions/phases use unseeded `Math.random()` (lines ~1104-1114) → every page load has a different sky; replace with seeded PRNG |
| FIX-F | NIT | Shot 5 HUD deceleration too weak (VEL 1560→1520 across chute deploy); add a knee: −250±30 m/s concentrated around inflation end 23.3s, still monotonic |

## FIX-A · Plasma sheath (evidence-based)

Runtime probe: sheath is centered ON the spacecraft position; the heatshield mesh (in front of it, from the camera's angle) occludes everything except two rim slivers.

Prescribed:
1. Offset the sheath **prograde**: translate it along the heatshield's outward normal (toward the camera/velocity direction) by ~0.45 world units, and scale it to wrap the heatshield rim: radius ≈ heatshield world radius × 1.25 (heatshield ≈ 1.1 world → sheath ≈ 1.35).
2. AdditiveBlending, depthWrite false (keep), `side: DoubleSide` so it reads from outside.
3. Color ramp per spec: white-hot core region (center of the cap, #FFF4D6) → #FFB566 → #FF5A26 at the fringe. A radial-gradient ShaderMaterial on a cap/disc is enough — silhouette-first, no volumetrics.
4. Particles: 900 alive at peak; increase point size so each streak is 6–14px at 1600×900 (sizeAttenuation), color #FFB566→#FF5A26, positions streaming backward (−velocity) via `pos = f(t, seed)` — pure function.
5. **Acceptance (self-verify with SELFTEST numbers, QA re-measures pixels):** at t=19.5, within the ship bounding box: warm pixels (r>60, r>b+15) ≥ 8000; white-core pixels (r>240) ≥ 200; a continuous bright arc spanning ≥ 250px along the heatshield edge.

## FIX-B · Haze backdrop composition

1. **Flip the gradient**: top of the plane (upper sky, near-space) = near-black #0D0A08; middle = #C97B4E; the bright #E8A97E band confined to the **lower ~30–35%** of the plane (horizon glow). Bright-at-top was my r0 wording ambiguity — this sentence is the binding one.
2. **Fade the plane's left/right/top edges to alpha 0** (horizontal + top falloff in the shader) so no plane boundary is ever visible — it must read as sky, not a slab.
3. Bloom must not saturate: after the flip, no region of the frame at t=25 should be blown white. Acceptance at t=25: pixels with r>250 < 2% of picture area; mean luminance of the top quarter of the picture < 40; the chute canopy reads as a **darker silhouette with a warm rim** against the glow (it currently pokes into the white as a "bump").
4. Optional (better): shrink the plane and/or push it to z=−30 so it composes as a horizon band rather than a backdrop wall — your call, acceptance is the look, not the numbers above.

## FIX-C · sunGlow in Act II

Extend the visibility rule: `sunGlow.visible = false` for t ≥ ACT2.SHOT4_START (keep Act I behavior exactly as frozen — visible from Shot 2 through 16s).

## FIX-D · Starfield fade (make it real)

Add a `uFade` uniform to the star shader (multiply the fragment alpha), drive it per-frame: 1.0 for t < 16, 0.12 for 16 ≤ t < 28, 0 after. Remove the ineffective `.opacity` writes.

## FIX-E · Seeded starfield (authorized determinism hardening)

Replace `Math.random()` in starfield generation (positions, phases, sizes) with a seeded PRNG (e.g. mulberry32(1234)). This changes the star layout once — approved by QA, it is not an accepted-behavior change (layouts were never reproducible before). Afterwards, cross-build pixel regression for 0–16s becomes exact.

## Regression guard (must hold)

Act I 0–16s (after FIX-E, must be pixel-identical across builds) · blackout purity + SIGNAL LOST · 21.0 hard cut · paused double-frame IDENTICAL at 18/25 · 28–46s black · HUD flicker-back timing · all M0/M1 frozen interfaces.

## SELFTEST — extend once

Add one line at t=25: `SELFTEST {"t":25,"chuteScale":...,"swingDeg":...,"hazeTopLuma":<0-255 estimate from your own math>,"drawCalls":...}` — `hazeTopLuma` = your computed top-quarter mean of the haze plane color (should come out < 40). Keep all six existing targets.

## Re-delivery checklist (M02-v2.html)

1. SELFTEST lines (six targets + new t=25 field) + line count + section index
2. Notes: any deviation from the prescriptions above, with reasons
