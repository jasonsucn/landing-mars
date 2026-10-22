# FIX-M01 · Rectification Round 2

> QA review of M01-v2. **FIX-1 (Shot 2) PASSES** — spacecraft silhouette reads, Mars grows on schedule, rim flare works. **MINOR-1 fixed.** Two items remain; both are small and localized.

| # | Severity | Issue |
|---|---|---|
| FIX-A | **BLOCKER** | Shot 3: cockpit frame still invisible — material drowns in film grain |
| FIX-B | **BLOCKER** | Shot 1 REGRESSION: Mars now a large tan oval — was correct in v1 |
| MINOR-A | minor | Ship silhouette reads as an abstract slab — needs 2–3 signature details |

---

## FIX-A · BLOCKER — Frame invisible because material ≈ background

**Evidence** (pixel scan of t=9.5s frame): the strip where the top bar renders (mean luminance 5.2) is indistinguishable from reference background (6.2). Your frame material is `0x050505` — the animated film grain (per-pixel noise ±15 around mean ~5) completely swallows it. The geometry work is fine; the **value** is wrong.

**Required fix**:
1. Bar base color → `0x101014` (constant, reads as structure through grain without shouting)
2. Inner edge of each bar gets a thin warm rim from the Mars side: emissive strip or a 1–2% opacity `#E9A568` edge glow, brighter as Mars grows (tie intensity to t). This is the classic "cabin lit by planet light" cue — the viewer should feel the cockpit, not measure it.
3. Self-verify BEFORE delivering: at t=12s the bars must be identifiable in a screenshot without annotations. If you still can't see them, do not ship.

## FIX-B · BLOCKER — Shot 1 Mars regression

**Evidence**: at t≈4s Mars is a tan-yellow oval ~8% of frame width, brighter than any star. In v1 this shot was correct and passed review.

**Required** (restore v1 intent, now with explicit numbers):
- Apparent size at 4.5s: **≤1.5% of frame width**; grows only after Shot 2 begins
- Color: dark rust red (≈ `0x8B3A1E`, dim), never tan/bright
- Shape: perfectly circular disc (a sprite/oval with non-uniform scale is not acceptable — use the actual sphere mesh or a uniform-scale sprite)
- Read: "a red dot you can find if you look", per the title's 225 KM promise

## MINOR-A — Ship silhouette legibility

Currently reads as an abstract slab (5-slab composite). Add 2–3 signature elements so the silhouette says "spacecraft": a thin antenna mast, a radiator fin array (offset from centerline — asymmetry reads more mechanical), and a slight taper to the hull. Keep total triangles trivial. This ship is FROZEN after M1 — M2 jettisons the heatshield and M3 deploys the legs from it, so its silhouette gets finalized now.

---

## Re-delivery checklist (attach with M01-v3.html)

1. Console clean; all v2 passes hold (Shot 2, determinism, black 16–60s, HUD, titles)
2. t=2s AND t=4s: Mars = small dim red dot (≤1.5% frame width), circular
3. t=9.5/12/14: window bars clearly identifiable (base + warm inner edge), cockpit feel present
4. Ship silhouette has antenna/fin signature details
5. Attach screenshots t=2, 6.5, 12 and verify each frame's timecode in the shot before capturing
6. Line count + section index + draw-call count at t=10s
