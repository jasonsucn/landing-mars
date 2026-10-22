# M02-v1 Delivery Notes

## Artifact

- File: `M02-v1.html` (1802 lines)
- Scope: Act II-A, 16–28s
- Reserved black interval: 28–46s
- Runtime self-test URL: `M02-v1.html?selftest=1`

## Sections

- CONFIG: line 347
- Timeline Engine: line 622
- Camera System: line 639
- Audio Engine: line 722
- UI Controls: line 826
- Act Content: line 1065
- Main Loop: line 1721

## Static implementation checks

- Act II-A config contains Shot 4 (16–21s), blackout (21–22s), and Shot 5 (22–28s).
- Plasma particle count is 900, below the 1200 budget.
- Plasma positions use fixed deterministic seeds; no per-frame random calls were added.
- Camera shake uses named fixed frequencies and phases.
- Chute swing uses the specified exponential decay form.
- Blackout restores the parked UI mechanism through the configured 21–22s interval.
- Audio cue definitions are seek-safe and crossed through the existing AudioEngine tick contract.
- SelfTest targets are `[9, 12, 15.5, 18, 21.5, 25]`.
- SelfTest emits M01 rim NDC fields for 9/12/15.5 and M02 plasma/blackout/chute fields for 18/21.5/25.
- Spacecraft retains named `hull`, `serviceModule`, `antenna`, `heatshield`, and hidden `legs` extension nodes for M3.

## Runtime evidence required from browser console

The implementation emits six lines with the `SELFTEST` prefix followed by `SELFTEST-DONE`. Runtime NDC and draw-call values are intentionally not fabricated here; they must be copied from a browser run of the self-test URL.

Expected field shapes:

```text
SELFTEST {"t":9,"ndc":{"top":...,"bottom":...,"left":...,"right":...},"drawCalls":...}
SELFTEST {"t":12,"ndc":{"top":...,"bottom":...,"left":...,"right":...},"drawCalls":...}
SELFTEST {"t":15.5,"ndc":{"top":...,"bottom":...,"left":...,"right":...},"drawCalls":...}
SELFTEST {"t":18,"plasmaCount":...,"shakeAmp":...,"drawCalls":...}
SELFTEST {"t":21.5,"blackout":true,"drawCalls":...}
SELFTEST {"t":25,"chuteScale":...,"swingDeg":...,"drawCalls":...}
SELFTEST-DONE
```

## Verification boundary

Static source checks are complete. Browser/WebGL pixel verification, console output, and actual draw-call counts require running the file in a browser with WebGL enabled.
