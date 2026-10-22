# TASK-M00 · Foundation Round

> This brief is self-contained; no other context is required. Deliver one complete single-file HTML when done.
> Later rounds will fill in the actual content (starfield / spacecraft / EDL / astronaut) on top of this foundation. This round uses **placeholders only to validate the pipeline — do NOT implement any final content ahead of schedule.**

---

## 0. Project Background (30-second version)

《登陆火星》"Landing on Mars": a 60-second cinematic 3D animation, pure viewing experience, played in the browser as a single file. Three acts (Approach 0–16s / EDL 16–46s / The New World 46–60s). This round builds the **technical foundation only**, so that later rounds do "fill in content" and never "re-architect".

## 1. Scope

**Do**: render pipeline, timeline engine, camera system, UI shell (intro overlay / control bar / letterbox), audio engine skeleton.
**Don't**: any final act content. Placeholder validation objects are defined in §5.

## 2. Technical Rules

1. Single-file HTML with an importmap:
   ```html
   <script type="importmap">
   { "imports": {
       "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
       "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
   } }
   </script>
   ```
2. Code MUST be organized with section banners, in this exact order:
   `/* ============== SECTION: CONFIG ============== */` style one-line banners, sections ordered:
   `CONFIG → Render Pipeline → Timeline Engine → Camera System → Audio Engine → UI Controls → Act Content (placeholder) → Main Loop`
3. Every tunable value lives in the top-level `CONFIG` object. Zero magic numbers in business code.
4. Code comments in English, concise.

## 3. Core Architecture: Pure-Function Timeline (the lifeline of this project — implement exactly as specified)

```js
// Single global state entry: any t fully rebuilds the visual state, zero accumulated side effects.
// FORBIDDEN in animation logic: += accumulation across frames, setTimeout/setInterval for choreography.
function applyState(t) {
  // 1. Camera: look up camera table (see §4)
  // 2. Actor nodes: visibility / transform / material params computed from t
  // 3. Post uniforms: grain time, fade/blackout alpha, HUD values, subtitle states
}
```

Main loop: while playing, `t = clock.getElapsedTime()` clamped to `[0, CONFIG.DURATION]`; paused → do NOT advance t but KEEP rendering (grain keeps animating); seeking → set t and call `applyState(t)` once. Playback must remain **frame-rate independent** (state derives from t, never from frame deltas).

## 4. Camera System: table-driven

```js
// One camera CUT per entry, hard cuts between entries (no cross-fade). First match wins.
const CAMERA_MOVES = [
  { start: 0, end: 8, from: { pos: [0,2,10], look: [0,0,0], fov: 50 },
                       to:   { pos: [0,1,6],  look: [0,0,0], fov: 50 }, ease: 'easeInOutCubic' },
  // ...
];
function cameraStateAt(t) { /* returns {pos, look, fov}; handles hold-before-first / hold-after-last */ }
```
Provide at least: `linear, easeInOutCubic, easeOutQuart` in an easing library (pure functions).

## 5. Placeholder Content (pipeline validation ONLY — not final content)

- A glowing test sphere (emissive, drives Bloom) at the origin + a small ring of orbiting particles around it (Points, ≤200).
- One slow camera move spanning 0–8s and a second cut 8–60s (different angle) — proves table-driven cuts.
- **Blackout rehearsal at t∈[21,22)**: screen goes fully black with centered white monospace text `SIGNAL LOST` (this rehearses the real Act-2 blackout mechanism; implement as a timeline-driven overlay alpha, not a DOM class toggle hack).
- A test HUD layer: bottom-left, two monospace readouts driven by functions of t (e.g. `VEL: 2451.3 M/S` / `ALT: 87.4 KM`), proving the HUD mechanism. Style: thin, technical, low-opacity — no decorative boxes.

## 6. Render Pipeline

- `WebGLRenderer` + `ACESFilmicToneMapping` + correct output color space (sRGB).
- `EffectComposer`: `RenderPass → UnrealBloomPass → single custom ShaderPass` combining **vignette + animated film grain** (grain uses a `uTime` uniform; vignette darkens corners; both strengths from CONFIG).
- Letterbox: outer black body, inner viewport locked to **2.39:1**, centered, recomputed on resize (canvas sized to the viewport, not fullscreen).

## 7. UI Shell

1. **Intro overlay**: title 「登陆火星」 + subtitle "A MARS LANDING FILM" + a thin "CLICK TO BEGIN" line. Click → init audio, fade overlay (CSS transition), start playback.
2. **Control bar** (bottom, appears on mouse move, fades out after 3s idle; stays visible while paused):
   - Play/pause button · seekable progress bar (click + drag) · timecode `mm:ss / mm:ss` · mute toggle · three act-boundary tick marks on the bar (at 16s and 46s, from CONFIG, not hardcoded).
   - Style: hairline borders, semi-transparent white, brightens on hover, monospace numerals. **NO gradients, NO big rounded cards, NO emoji.**
3. **End state**: at `t ≥ DURATION` freeze on final frame, show a centered "REPLAY" button; clicking resets to t=0 and plays.

## 8. Audio Engine Skeleton

```js
const AudioEngine = {
  ctx, master,        // init() on intro click: new AudioContext + master GainNode
  muted, setMuted(b), // controlled by the control bar; default from CONFIG.MUTED_BY_DEFAULT = true
  cues: [],           // SFX_CUES: {t, id, params}
  // tick(prevT, t) every frame: fire cues whose trigger point falls inside (prevT, t].
  // IRON RULE: fire ONLY on natural playback crossing; seeking/replay must NOT fire
  // (on seek, sync prevT = t so the window stays empty).
  // This round: register one test cue at t=2s — a short sine beep (validates the cue mechanism only).
};
```

## 9. Self-Check Checklist (verify manually before delivery; report results item by item)

1. Opens directly in browser, Console clean (zero errors, zero warnings)
2. Intro overlay → click → fades out and playback starts automatically
3. Viewport is strictly 2.39:1; still correct after resize / maximize
4. Dragging the seek bar to any position (e.g. 5s / 30s / 59s) yields a consistent, artifact-free state (pure-function timeline proof)
5. Blackout + SIGNAL LOST active exactly in t∈[21,22); correct when seeking in/out of the range
6. Bloom glow, vignette, animated grain all clearly visible
7. Control bar: appears on move / fades after 3s / stays while paused; play, pause, seek, mute, act ticks all work
8. Playback stops at the last frame and REPLAY appears; replay works
9. Setting `CONFIG.DURATION` to 10 rescales the whole film: act ticks, end logic (set it back to 60 before delivering)
10. Audio: beep fires when playback crosses t=2s (silent if muted); seeking past 2s does NOT fire it

## 10. Deliverable Format

1. One complete single-file HTML (full functionality; NO ellipses / pseudo-code / multi-part paste)
2. Attach: item-by-item self-check results, total line count, and a **section index** (start line number of every SECTION banner)
3. If worried about output length: protect runnability and completeness first — you may trim comments/blank lines, but **no architecture or feature may be dropped**

Expected size: 800–1200 lines. If you have architectural objections, raise them in the delivery notes — do NOT change the architecture unilaterally.
