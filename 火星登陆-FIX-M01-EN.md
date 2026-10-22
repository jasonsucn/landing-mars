# FIX-M01 · Rectification Round 1

> QA review of M01-v1 complete. **Foundations praised**: Mars fbm surface + blue limb glow + backlit bloom flare + HUD countdown + title system + frame-exact determinism (identical screenshots) + clean black 16–60s + frozen interfaces untouched. The Act I "look" of Shot 1 and Shot 3 is close to final quality.
> **Two blocking composition failures** — objects exist in code but never appear in frame. Deliver `M01-v2.html` with the self-check list and screenshots.

| # | Severity | Issue |
|---|---|---|
| FIX-1 | **BLOCKER** | Shot 2 (4.5–9s): spacecraft never visible, Mars far too small |
| FIX-2 | **BLOCKER** | Shot 3 (9–16s): cockpit frame never visible |
| MINOR-1 | minor | `t === 10` renderer.info log can never fire |

---

## FIX-1 · BLOCKER — Shot 2 composition failed

**Evidence** (headless Chrome, file://): at t=6.5s and t=7.5s the frame contains ONLY a tiny Mars disc (~5% of frame height). No spacecraft silhouette, no sun rim light, Mars not "occupying ~1/3 frame growing".

**Code evidence**: `spacecraft.visible = true` + `position.set(-3 + shotT*0.5, 1 - shotT*0.3, -10 - shotT*2)` + `scale 0.8` (≈line 1315-1321) — the ship IS created (hull/serviceModule/antenna/heatshield ≈line 1186+) and the visibility window is correct, but at camera distance ≥10 units with scale 0.8 it renders a few pixels or off-frame. The camera table for Shot 2 also keeps Mars far too small.

**Required composition (from TASK-M01 §1 Shot 2)**:
- Mars ~1/3 frame height at mid-shot, growing through the shot
- Spacecraft: dark silhouette **clearly readable** crossing the lower third or offset from center, scale it up (or move camera closer) so its outline is unambiguous — target ≥15% of frame width
- Sun: strong backlight from near the frame edge → warm rim light separating the ship silhouette from black space. If your sun is a distant glow sprite, place it just off-frame edge so only the rim + slight lens spill show. Bloom already active — let the rim catch it.

**How to iterate fast**: screenshot t=5.5 / 6.5 / 7.5 / 8.5 and check the ship is readable in all four before delivering. This is staging work — camera table + transforms only, do not touch frozen engine code.

## FIX-2 · BLOCKER — Shot 3 cockpit frame never visible

**Evidence**: at t=9.5 / 12 / 15.5 the frame is a clean exterior view — no window frame anywhere, so the "inside the cockpit" story beat is lost.

**Code evidence**: `cockpitFrame.visible = true; cockpitFrame.position.set(0,0,3)` (≈line 1332) within the correct time window. Visibility logic is fine — the placement is wrong. A frame bar of fixed width at z=3 will only cover the viewport if its width matches the camera frustum at that depth.

**Required composition (TASK-M01 §1 Shot 3)**:
- Frame bars: top/bottom (and optionally left) dark silhouette bars **overlapping the picture edges**, so Mars + limb glow are seen THROUGH the window — target the bars occluding 5–15% of picture edges
- Compute placement from the camera: if the camera sits at distance `d` from the frame plane with vertical FOV `θ`, the frame's half-width must be ≥ `d·tan(θ/2)·aspect`. Add margin so resize keeps bars at the edges (or reposition cockpitFrame relative to the camera each frame from cameraState — recommended, it guarantees coverage)
- Bars nearly black (barely-lit interior side), subtle — the viewer should feel the cabin without noticing the geometry

## MINOR-1 — dead debug line

`if (t === 10 && renderer)` never fires (float t). Use `if (renderer && Math.abs(t - 10) < 0.004)` or log once from the main loop. Harmless but it silently disabled the draw-call report TASK-M01 §5.10 asked for — include the draw-call number in your delivery notes instead.

---

## Re-delivery checklist (attach with M01-v2.html)

1. Console clean; intro → playback
2. t=2s: starfield + title, Mars dot faintly present
3. t=5.5/6.5/7.5/8.5: spacecraft silhouette readable, Mars growing to ~1/3 frame
4. t=9.5/12/15.5: window bars occlude picture edges, Mars fills lower frame, HUD counts down, thruster glow subtle
5. 21.5s and 30s: pure black (no SIGNAL LOST)
6. Seek determinism at 12s still identical
7. Draw-call count at t=10s reported in notes
8. Frozen interfaces untouched
