# FIX-M01 · Rectification Round 4

> QA update on M01-v4. First: the diff discipline was excellent — exactly the three prescribed changes, nothing else. Keep that.
> FIX-B is fully verified and closed (sunGlow hidden ✓, Mars dot 2px dim red at screen (802,450) ✓).
> But FIX-A **did not actually work**: the rim has never been rendered in ANY version so far — not in v2 (opacity), not in v3/v4 (occlusion). Your delivery notes with the t=12 self-proof screenshot were also missing, which is exactly how this slipped through. The gate exists so that "not visible" can never ship. Root cause analysis below is fully evidenced — read it before coding.

| # | Severity | Issue |
|---|---|---|
| FIX-C | **BLOCKER** | Rim meshes are depth-tested away: embedded inside the opaque bars |
| FIX-D | **BLOCKER** | Top/bottom rim lines are off-screen at 16:9 (NDC y = ±1.38) |
| FIX-E | QUALITY | Rim reads as a ~30px soft band; spec is a thin elegant line |

---

## FIX-C · Rim occlusion (root cause, evidenced)

Runtime facts measured at t=12 on your v4 build:

- Rim opacity = 0.334 (formula applied correctly), `visible = true`, in-frustum (left/right rim NDC x = ±0.92)
- Yet **zero warm pixels** anywhere in the frame (full-image pixel scan)
- Experiment: `children[0..3].visible = false` (hide bars) → rims render instantly (warm pixels at left edge: 0 → 10,606)

**Cause:** rim boxes (depth `frameThickness * 0.8`, z-offset −0.01) sit entirely inside the bar boxes (depth `frameThickness`). The bars are opaque, their front faces are closer to the camera, so every rim fragment fails the depth test.

**Warning — do not "just move the rim forward":** `cockpitFrame.lookAt()` orients the group so its local **+z points AWAY from the camera** (also evidenced by left/right rims swapping world X after lookAt). A tested `rim.position.z = +0.12` moved rims *deeper* and changed nothing. Any sign-based offset must be verified by screenshot.

**Prescribed fix (pick one, a is recommended — one line, sign-proof):**

- **(a)** Rim material: `depthTest: false` — the rim is an additive-blend glow line that is always the nearest geometry; disabling depth test has zero visual side effects here
- (b) Bars: `depthWrite: false`
- (c) Rim z-offset toward camera with **empirically verified sign** (test both, keep the one that renders)

## FIX-D · All four edges on-screen

Measured at t=12 (1600×900, fov 55): top/bottom rim NDC y = **±1.38** → outside the viewport. The self-proof gate requires all four warm inner edges visible.

- Current: `frameHalfHeight = tan(fov/2) * frameDistance * 1.15`
- **Required: vertical margin factor 1.15 → 0.75** (lands top/bottom at NDC ≈ ±0.90)
- Keep the horizontal chain unchanged (left/right at ±0.92 ✓)
- Left/right bar+rim heights use the same half-height — they shorten accordingly, which is correct: the whole frame (dark bars + warm inner edges) must be fully in view

## FIX-E · Thin elegant line

Measured: rim band ≈ 30px wide at 1600×900 (left band x≈46–75). Target: **core line ≤ 8px at 1600×900** (bloom halo excluded). Minimal change: rim thickness `frameThickness * 0.3` → `* 0.10–0.12`; keep color 0xE9A568 and opacity 0.30–0.38 unchanged.

## Regression guard (must hold)

- Shot 1: starfield + title + 2px dim Mars dot, no sunGlow (v4 state — do not touch)
- Shot 2 spacecraft silhouette, Mars growth, sun glow visible from Shot 2
- Paused double-frame determinism, black 16–60s + HUD, titles, frozen interfaces

## Re-delivery checklist (M01-v5.html) — notes are MANDATORY this time

1. **t=2s screenshot** (starfield + title + dot) — plus a zoomed crop around the dot
2. **t=12s screenshot at 1600×900 with all four warm inner edges visible** — this is the gate. If you cannot see the frame in your own screenshot, it is not done — do not submit
3. t=6.5s screenshot (unchanged from v2)
4. Line count, section index, and `renderer.info.render` draw-call count at t=10s (missing from v4 notes)
