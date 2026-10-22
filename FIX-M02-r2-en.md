# FIX-M02-r2-en.md — M02-v2 review · 3 items (1 P0 / 2 P1)

QA of M02-v2 (headless chromium, ?selftest=1, plus frame captures at multiple wall-times).
**Verdict: FIX-A/B/C/D verified working. 1 new P0 regression, 2 overexposure items.**
General note on timing: headless runs at ~2.5fps and the film clock applies a dt clamp, so picture time lags wall time by ~4x. Frame captures below are identified by the on-screen VEL/ALT HUD values, not wall time.

## PASS (recorded, no action)

- **FIX-A geometry**: plasma sheath now visible and dominant, white-hot core + orange falloff, ship silhouette reads against it. Frame `review/v2/p17_6.png`.
- **FIX-B gradient direction**: haze plane now dark-top → bright-bottom, top-quarter mean luma measured 8.8 (<40 required). Edge fade confirmed (no slab boundary).
- **FIX-C**: no sunGlow disc anywhere in act 2 frames. ✓
- **FIX-D**: starfield fades back in after blackout ✓
- **Blackout regression**: pure black + HUD-only at expected window ✓

## P0-1 · HUD shows NaN after blackout (NEW REGRESSION)

**Evidence**: frame at picture-time ≈20-21s shows `VEL: NaN M/S` / `ALT: NaN KM` (capture `review/v2/p25.png`). The velocity/altitude chain produces NaN from roughly the chute-deploy region onward.

**Suspect**: the FIX-F piecewise velocity curve — the segment junction at t=23.3 (VEL_END-90 → VEL_END-130) or a divide by a zero-width interval. NaN then propagates into the HUD and any downstream consumer (camera shake, chute math).

**Required**:
1. Fix the NaN source; audit the whole velocity/altitude chain for divide-by-zero and out-of-domain lerps (NaN can enter anywhere and spread).
2. Add a guard: if the computed VEL/ALT is not finite, fall back to the previous frame's value (defensive clamp), so a single bad sample can never reach the HUD.
3. **SELFTEST**: add `hudVel` and `hudAlt` (the actual displayed strings parsed to numbers) to the t=25 payload. Acceptance: both finite and VEL within [0, VEL_END+200].

## P1-1 · Plasma core overexposure (FIX-A follow-up)

At plasma peak the lower ~40% of frame is blown pure white (tens of thousands of r=255 pixels) and the ship's lower half is swallowed. FIX-A asked for white-core **≥200 px**, not a white flood.

**Required**:
1. Reduce core intensity/radius: white-hot region confined to a band hugging the heatshield (target: white pixels < 8% of frame at t=19.5 peak), keep the orange falloff.
2. Ship silhouette must stay visible end-to-end (dark body + rim light), not half-vanish into the core.
3. Keep all FIX-A acceptance numbers (warm ≥8000, arc ≥250px) — they still pass.

## P1-2 · t=25 haze white-out still over budget (FIX-B follow-up)

hazeTopLuma selftest = 0.498 → that is 127/255 luma, more than double the <40 (0.157) requirement. The gradient direction is right, the color is not.

**Required**:
1. HAZE.HIGH_COLOR (top of plane) must be a near-black warm tone, luma < 0.157 (=40). The bright #E8A97E band belongs only to the lower 30-35%.
2. **SELFTEST**: extend t=25 payload with `whitePct` (share of pure-white pixels via readPixels over a downsampled framebuffer). Acceptance: < 2%.
3. Chute canopy must read as a darker silhouette with warm rim against the glow (currently it pokes into the white as a bump — see FIX-M02-r1 §FIX-B item 3, still open).

## Delivery

- `M02-v3.html`, same single-file + `?selftest=1` protocol.
- SELFTEST payloads must now include: `hudVel`, `hudAlt`, `whitePct` (new), plus all previous fields.
- Delivery notes: state which lines were touched. Do not refactor anything outside the listed items.
