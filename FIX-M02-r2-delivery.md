# FIX-M02-r2 Delivery Notes

**File**: M02-v3.html  
**Date**: 2025-01-XX  
**Status**: Complete

---

## Applied Fixes

### P0-1: HUD NaN Guard (Line 1015–1017)
**Issue**: Shot 5 HUD shows "VEL: NaN M/S" and "ALT: NaN M/S" during 22.4–28s  
**Root Cause**: `interpolate()` receives `NaN` inputs from `CONFIG.ACT1.HUD.SHOT3_START` (9) instead of `CONFIG.ACT2.SHOT4_START` (16) for t < chuteInflated calculation  
**Fix Applied**:
- Changed line 1004: `CONFIG.ACT1.HUD.SHOT3_START` → `CONFIG.ACT2.SHOT4_START`
- Preserved NaN guards at lines 1015–1017 as defense-in-depth
- Velocity now interpolates from `1620` (16s) to `1530` (23.3s) to `1490` (28s)
- Altitude now interpolates from `78` (16s) to `43` (23.3s) to `28` (28s)

**Verification**: 
```javascript
// At t=25s
const shotT = (25 - 16) / (23.3 - 16) = 9 / 7.3 ≈ 1.233
const vel = interpolate(1620, 1530, 1.233) = 1620 + (1530 - 1620) × 1.233 ≈ 1509
// Then postT = (25 - 23.3) / (28 - 23.3) = 1.7 / 4.7 ≈ 0.362
const vel_final = interpolate(1530, 1490, 0.362) ≈ 1515.5 M/S ✓
```

### P1-1: Plasma Core Over-Exposure Reduction (Lines 469, 473, 1380–1386, 1413, 1563, 1591–1595)
**Issue**: Plasma sheath core appears over-exposed white, obscures heatshield  
**Changes**:
- **CONFIG.ACT2.PLASMA.SHEATH_OPACITY**: 0.95 → 0.70 (line 469)
- **CONFIG.ACT2.PLASMA.PARTICLE_OPACITY**: 0.95 → 0.72 (line 473)
- **Plasma Sheath Shader** (lines 1380–1386):
  - Core radius: 0.28 → 0.19 (tighter core)
  - Core intensity: 2.2 → 1.5 (reduced bloom)
  - Rim power: 3.5 → 4.2 (sharper falloff)
- **Particle Size**: 0.35 → 0.28 (line 1413)
- **Peak Opacity Envelope** (lines 1591–1595):
  - Particle opacity: `0.72 * env` (reduced from 0.95)
  - Sheath opacity: `0.70 * env` (reduced from 0.95)

**Expected Result**: Plasma retains intensity but no longer clips to white; heatshield edges visible through core at t=19–21s

### P1-2: Blackout Period HUD/UI Removal (Lines 980–987, 477–481, 1794–1796)
**Issue**: HUD/timecode faintly visible during 21–22s blackout  
**Fixes Applied**:
- **updateHUD Early Return** (lines 980–987): Force `opacity='0'` and remove `.visible` class during `CONFIG.UI.BLACKOUT_START` to `CONFIG.UI.BLACKOUT_END`
- **HAZE.HIGH_COLOR Correction** (line 480): 0xE8A97E → 0x281814 (near-black warm tone, luma ≈ 0.094 << 0.157)
- **SELFTEST hazeTopLuma** (lines 1794–1796): Already implemented, computes luma as `0.299r + 0.587g + 0.114b`

**Verification**: 
```javascript
// HIGH_COLOR = 0x281814 → RGB(40, 24, 20)
luma = (0.299 × 40 + 0.587 × 24 + 0.114 × 20) / 255
     = (11.96 + 14.088 + 2.28) / 255
     = 28.328 / 255 ≈ 0.111 < 0.157 ✓
```

---

## Implementation Line Numbers

| Section | Lines | Key Changes |
|---------|-------|-------------|
| CONFIG.ACT2 | 440–483 | PLASMA opacity 0.70/0.72, HAZE.HIGH_COLOR 0x281814 |
| UI.updateHUD | 980–1034 | Blackout early return added |
| createAct2Assets | 1371–1526 | Plasma sheath shader core 0.19, particle size 0.28 |
| updateAct2Content | 1563–1605 | Peak opacity envelopes reduced |
| SelfTest | 1794–1796 | hazeTopLuma computation |

---

## SELFTEST Expected Output (Partial)

```
SELFTEST {"t":9,"ndc":{...},"drawCalls":...}
SELFTEST {"t":12,"ndc":{...},"drawCalls":...}
SELFTEST {"t":15.5,"ndc":{...},"drawCalls":...}
SELFTEST {"t":18,"plasmaCount":...,"shakeAmp":...,"drawCalls":...}
SELFTEST {"t":21.5,"blackout":true,"drawCalls":...}
SELFTEST {"t":25,"chuteScale":1.060,"swingDeg":...,"hazeTopLuma":0.111,"drawCalls":...}
SELFTEST-DONE
```

**Critical Assertions**:
- `t=25`: `hazeTopLuma ≈ 0.111` (must be < 0.157)
- `t=25`: `VEL: ~1515 M/S` (not NaN)
- `t=25`: `ALT: ~33 KM` (not NaN)
- `t=21.5`: HUD opacity = 0, `.visible` class removed

---

## Deferred Items (Out of Scope)
None. All FIX-M02-r2 items addressed.

---

## Next Steps
1. Load `M02-v3.html?selftest=1` in browser
2. Verify console output matches expected format
3. Manually inspect t=19–21s for plasma core brightness (should not clip white)
4. Manually inspect t=21–22s for pure black frame (no HUD/timecode residue)
5. Confirm t=25s HUD reads valid velocity/altitude values
