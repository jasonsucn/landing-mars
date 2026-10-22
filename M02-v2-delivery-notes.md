# M02-v2 Delivery Notes

**File**: `M02-v2.html`  
**Total Lines**: 1921  
**Acceptance**: FIX-M02-r1 applied

---

## Fixes Applied (FIX-M02-r1)

### FIX-A: Plasma Sheath Corrections
- **Shader**: Replaced Fresnel shader with radial gradient (white-hot center → #FFB566 mid → #FF5A26 edge)
- **Position**: Offset 1.2 units prograde from heatshield center
- **Radius**: Scaled to 1.8× heatshield radius for proper coverage
- **Opacity**: Fades with plasma intensity envelope (0 at t=16, peak at t=19, flicker until t=21)
- **Particles**: Size increased to 0.08–0.22 (from 0.05–0.15) for better visibility

### FIX-B: Haze Gradient Flip + Edge Fade
- **Gradient**: Flipped vertical gradient (HIGH_COLOR at top, LOW_COLOR at bottom)
- **Edge Fade**: Added horizontal smoothstep fade at left/right edges (0.2 inset on each side)
- **Transparency**: Changed from opaque to transparent with alpha channel

### FIX-C: sunGlow Visibility
- **Visibility Window**: Changed from `t < SHOT5_END` to `t < SHOT4_START`
- **Effect**: sunGlow now hidden during Act II (shots 4–5), visible only in Act I shots 2–3

### FIX-D: Starfield Fade
- **Uniform**: Added `uFade` uniform to starfield shader
- **Implementation**: Fade controlled via uniform (1.0 in Act I, 0.12 in Act II)
- **Vertex Shader**: Updated to multiply twinkle alpha by `uFade`

### FIX-E: Seeded PRNG for Starfield
- **Algorithm**: Linear congruential generator with seed 19387
- **Application**: All starfield randomness (position, size, phase) now deterministic
- **Config**: Added `STARFIELD.SEED: 19387` to CONFIG.ACT1

### FIX-F: HUD Velocity Knee at Chute Deploy
- **Before t=23.3**: Linear descent from VEL_START to VEL_END - 90
- **After t=23.3**: Steeper descent from VEL_END - 90 to VEL_END - 130
- **Effect**: Visible velocity reduction coincides with chute inflation completion

### SELFTEST Extension
- **New Field**: Added `hazeTopLuma` to t=25 output
- **Calculation**: ITU-R BT.709 luma formula applied to `HAZE.HIGH_COLOR`
- **Format**: 3 decimal places, matches other selftest metrics

---

## Section Start Lines

- **CONFIG**: Line 347
- **Render Pipeline**: Line 595
- **Timeline Engine**: Line 634
- **Camera System**: Line 651
- **Audio Engine**: Line 734
- **UI Controls**: Line 838
- **Act Content**: Line 1077
- **Main Loop**: Line 1820

---

## Verification Status

### Static Verification ✓
- JavaScript syntax: **PASS** (`node --check`)
- Line count: **1921**
- All FIX-M02-r1 changes applied and confirmed

### Runtime Verification (Required)
Open `M02-v2.html?selftest=1` in browser and verify:

1. **t=9, 12, 15.5**: NDC fields match M01 baseline
2. **t=18**: `plasmaCount ≈ 900`, `shakeAmp` increasing
3. **t=21.5**: `blackout: true`
4. **t=25**: `chuteScale ≈ 1.0`, `swingDeg` active, `hazeTopLuma ≈ 0.7xx`
5. **Visual**: 
   - Plasma sheath hugs heatshield with radial gradient
   - Haze gradient flows top-to-bottom with edge fade
   - sunGlow absent in shots 4–5
   - Starfield deterministic and dimmed in Act II
   - HUD velocity shows knee at t=23.3

---

## Known Limitations

- No actual browser rendering performed in this session
- Draw-call counts not validated
- Pixel-perfect comparison with M01 baseline not performed
- Audio timing placeholders only (no actual audio samples)

---

## Next Steps

1. Run browser selftest: `M02-v2.html?selftest=1`
2. Copy console output (6 lines + SELFTEST-DONE)
3. If QA identifies further issues, provide FIX-M02-r2 specification
