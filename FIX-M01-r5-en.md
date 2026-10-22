# FIX-M01 · Rectification Round 5

> QA update on M01-v5. First, an honest QA correction: **my r4 prescription for FIX-D was based on a wrong assumption** — I assumed the top/bottom rim positions were derived per-frame from `frameHalfHeight`. They are not: `topBar.position.y = frameHeight / 2` is a creation-time constant (±2.5 → NDC ±1.40). Your diff implemented r4 faithfully; the prescription itself was broken. The depthTest:false fix (FIX-C) worked perfectly — the rim finally renders. Now let's finish it with correct targets.

## Verified in v5

- ✅ FIX-C closed: rims render (proven: hiding rims removes the band, hiding bars doesn't)
- ✅ Left/right rims visible at NDC ±0.93
- ✅ All regressions hold (t=2 / t=6.5 / blackout / paused-double-frame determinism IDENTICAL)
- ❌ Top/bottom rims still off-screen (projected y-range −182..−157 px, entirely above the viewport)
- ❌ Rim band still 28–33px wide (v4: 30px — the thickness change was irrelevant, see FIX-E2)

## FIX-D2 · BLOCKER · Bring top/bottom edges into view

The per-frame scale loops fight the fixed positions (horizontal bars scale to ±2.79 world but rims sit at ±4 — corners can never close). Replace the whole scaling system with fixed geometry that matches the fixed positions:

1. `frameHeight = 5` → `3.2` → top/bottom bars+rims land at ±1.6 world ≈ NDC ±0.90
2. `frameWidth = 8` stays (left/right at ±4 ≈ NDC ±0.93 — measured, correct)
3. **Delete both per-frame scale loops** (the `for i<4` scale loop and the `for i=4..7` copy loop). Keep only the opacity update (`rimMesh.material.opacity = rimIntensity`)
4. Bar/rim geometries then need no scaling: top/bottom = `BoxGeometry(frameWidth*1.02, t, d)`, left/right = `BoxGeometry(t, frameHeight*1.02, d)` — they span to the corners exactly (±4.08 × ±1.63)
5. Gate: at **t=9, t=12, t=15.5** all four edges must stay within NDC 0.80–0.95 (camera orbits during the shot — verify it doesn't drift out; if it does, add per-frame `position.y = ±frameHalfHeight` updates as fallback)

## FIX-E2 · QUALITY · Kill the 33px perspective smear

Root cause (measured, not guessed): the rim box's **depth** dimension (`frameThickness*0.8 = 0.12`) sits at x=±4, far off-axis → near/far faces project 33px apart. Your `*0.10` thickness change affected the wrong dimension.

Fix: rim depth `frameThickness * 0.8` → `frameThickness * 0.05` (0.0075 world → smear < 2px). Keep X/Y thickness `*0.10` as is. Expected result: ~4px warm core + bloom halo. Gate: core band ≤ 8px at 1600×900.

## Cosmetic (no action required, noted for the record)

- leftRim/rightRim render mirrored (X swap after `lookAt`) — symmetric, no visual impact

## Process amendment — self-proof without screenshots (harness-adjusted)

The harness cannot run a browser or capture screenshots — that is a platform limit, not negligence. The self-proof gate is redefined:

1. **Embed a self-test mode** in M01-v6.html, activated by `?selftest=1` in the URL:
   - Autoplay muted; pause at t = 9.0, 12.0, 15.5; at each pause compute:
     a. each rim's center-in-view projection: `rim.getWorldPosition(v).project(camera)` → NDC of the four edges;
     b. `renderer.info.render` draw calls.
   - Log one JSON line per sample, prefixed `SELFTEST`, e.g.:
     `SELFTEST {"t":12,"ndc":{"top":0.88,"bottom":-0.88,"left":-0.93,"right":0.93},"drawCalls":42}`
   - After the last sample log `SELFTEST-DONE`. ~25 lines of code, no GL readback needed.
2. **Delivery notes** = line count + section index + the SELFTEST lines. The human relay runs `M01-v6.html?selftest=1` in Firefox (F12 → console) and pastes the output back — no screenshots required from anyone.
3. QA still does independent pixel-level screenshot verification on its side; the SELFTEST numbers are your pre-flight check — if your own output shows any edge |NDC| > 0.95 or missing, fix before submitting.

## Regression guard (must hold)

Shot 1 (starfield + title + 2px Mars dot, no sunGlow) · Shot 2 spacecraft silhouette + growing Mars + sun glow · paused double-frame determinism · black 16–60s + HUD · titles · frozen interfaces.

## Re-delivery checklist (M01-v6.html)

1. SELFTEST output at t=9/12/15.5: all four edge |NDC| ≤ 0.95 (no off-screen edges)
2. SELFTEST + delivery notes: line count, section index, draw-call counts
3. t=2 / t=6.5 behavior unchanged (QA verifies independently; state any intentional change in notes)
