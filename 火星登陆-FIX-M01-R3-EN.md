# FIX-M01 · Rectification Round 3

> QA update on M01-v3. Two QA-side corrections first, honestly:
> 1. **Determinism: CLEARED.** Frame-exact rebuild confirmed (paused double-frame test = identical). The earlier "DIFFERENT" was a QA test-method artifact (seek landing drift). No action needed.
> 2. **The "Mars regression" in r2 was a QA misjudgment.** The tan oval in Shot 1 is the **sun glow sphere** (`0xFFDD88`, r=5, at (50,15,-25), always visible, no visibility control) — not the Mars mesh. Your Mars scale 0.008 was never wrong. Apologies for the churn.
> Real blockers below — **this round every visual parameter is quantified; previous rounds' wording was contradictory and that's on QA.**

| # | Severity | Issue |
|---|---|---|
| FIX-A | **BLOCKER** | Cockpit rim glow invisible: opacity 0.015–0.025 ≈ transparent |
| FIX-B | **BLOCKER** | sunGlow visible in Shot 1 (should be hidden); Mars dot imperceptible |

---

## FIX-A · Cockpit rim — target values

Current: `rimIntensity = 0.015 + shotT * 0.01` → invisible.

**Required**: `rimIntensity = 0.30 + shotT * 0.08` (range 0.30 → 0.38 across 9–16s).
- Warm lines on black must be **clearly readable** at normal viewing distance, while staying thin and elegant
- Bar base `0x101014` stays as-is (correct value now)
- Your per-frame FOV-based sizing logic is good — keep it

**Self-proof gate (new, mandatory)**: attach a screenshot at t=12s in your delivery notes. All four warm inner edges must be visible in it. If you cannot see the frame in your own screenshot, it is not done — do not submit.

## FIX-B · sunGlow visibility + Mars dot

1. **sunGlow**: `visible = false` during Shot 1 (`t < ACT1.SHOT1_END`); `visible = true` from Shot 2 onward. Shot 1's frame must contain ONLY: starfield + faint Mars dot + title.
2. **Mars dot in Shot 1**: scale `0.008` → `0.035` (target: a dim rust-red disc ~0.15% of frame width ≈ 2–3px at 1080p — findable, never attention-grabbing). Keep its position.
3. No other Mars changes (v3 Mars shader/limb are approved quality).

## Regression guard (must hold)

- Shot 2 spacecraft silhouette (approved in v2)
- Paused double-frame determinism
- Black 16–60s, HUD in-picture countdown, titles, frozen interfaces

## Re-delivery checklist (M01-v4.html)

1. t=2s: starfield + title + **2–3px dim red dot**, no tan oval anywhere
2. t=12s screenshot **with visible warm window frame edges** (self-proof gate)
3. t=6.5s: unchanged from v2 (ship + growing Mars + rim flare)
4. Line count + section index + draw-call count at t=10s
