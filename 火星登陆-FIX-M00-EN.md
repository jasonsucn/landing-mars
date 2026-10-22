# FIX-M00 · Rectification Round 1

> QA review of M00-v1 complete. Architecture is **approved** (pure-function timeline, blackout mechanism, letterbox, post pipeline all verified working). 3 blocking fixes + 3 minor items below.
> Deliver a **complete single file** as `M00-v2.html` with the self-check list re-run. Do not refactor beyond what is listed.

## Verdict Summary

| # | Severity | Issue |
|---|---|---|
| FIX-1 | **BLOCKER** | Init order crash: whole app never starts |
| FIX-2 | **BLOCKER** | Emoji icons in control bar (explicitly forbidden) |
| FIX-3 | **BLOCKER** | Control bar fades out while cursor hovers it (required behavior missing) |
| FIX-4 | minor | HUD overlaps the letterbox black bars |
| FIX-5 | minor | Magic index `composer.passes[2]` |
| FIX-6 | minor | Click-to-seek conversion offset (QA observation, ≤1s) |

---

## FIX-1 · BLOCKER — Init order crash

**Symptom (reproduced in headless Chrome, file://):**
```
[PAGEERROR] Cannot read properties of undefined (reading 'setSize')
```
`updateViewportSize()` is called inside `initRenderPipeline()` at a point where `composer` is still `undefined` (it is created ~50 lines later). The exception aborts module top-level execution, so `initPlaceholderContent / PlaybackEngine.init / UI.init / animate` never run — the intro button does nothing, the canvas stays empty. This is why the page "errors out" on open.

**Fix (one-line move):** delete the early call and place it right after the composer is fully assembled:

```diff
   renderer.toneMapping = THREE.ACESFilmicToneMapping;
   renderer.toneMappingExposure = 1.0;
   renderer.outputColorSpace = THREE.SRGBColorSpace;
-  
-  updateViewportSize();
-  
   const renderPass = new RenderPass(scene, camera);
   ...
   composer = new EffectComposer(renderer);
   composer.addPass(renderPass);
   composer.addPass(bloomPass);
   composer.addPass(postPass);
+  
+  updateViewportSize();
   window.addEventListener('resize', updateViewportSize);
```

**QA-verified:** with exactly this patch applied, the full flow runs clean (no pageerror, intro → playback → seek → blackout → end state all functional). Do not change anything else in the init path.

## FIX-2 · BLOCKER — No emoji, ever

Reproduced: `#mute-btn` renders 🔇/🔊 (and the muted state glyph 🚫). Task brief §9 explicitly forbids emoji **anywhere, buttons included**.

**Fix:** replace all button glyphs with plain monospace text labels (NASA-instrument style, fits the film's look):
- play: `▶` and pause: `❚❚` are geometric glyphs — allowed, keep them
- mute/unmute: use text `SND ON` / `SND OFF` (or two tiny inline SVG paths if you prefer, but text is the reference solution)
- replay button: keep text `REPLAY`

## FIX-3 · BLOCKER — Hover must keep the control bar visible

Reproduced: while playing, holding the cursor still over the control bar for 3s fades it out (and the next click lands on nothing). Task brief §7: *"Stays visible while paused AND while cursor hovers the bar."*

**Fix:** track whether the cursor is over the bar and suppress the fade timer accordingly:

```js
// in UI init:
this.mouseOverBar = false;
this.controlBar.addEventListener('mouseenter', () => { this.mouseOverBar = true; this.showControls(); });
this.controlBar.addEventListener('mouseleave', () => { this.mouseOverBar = false; });

// showControls():
showControls() {
  this.controlBar.classList.add('visible');
  clearTimeout(this.controlFadeTimer);
  if (this.isPlaying && !this.mouseOverBar) {          // ← add the guard
    this.controlFadeTimer = setTimeout(() => this.controlBar.classList.remove('visible'), CONFIG.UI.CONTROL_FADE_DELAY);
  }
}
```

## FIX-4 · minor — HUD belongs inside the picture, not on the black bar

`#hud-overlay` is pinned to the viewport corner, so it sits on the letterbox black bar. HUD readouts are diegetic instruments — anchor them to the **rendered picture area** (the 2.39:1 box), e.g. position them relative to `#viewport-container`'s canvas rect with a small inner margin (≥24px inside the picture edge).

## FIX-5 · minor — No magic indices

`getPostUniforms()` returns `composer.passes[2].uniforms`. You already hold the reference — use it: `return postPass.uniforms;` (keep the helper or inline it at the single call site).

## FIX-6 · minor — Seek click offset

QA measured up to ~1s discrepancy between the clicked progress position and the resulting timecode (e.g. clicking 30/60 landed at 00:29; 21.5/60 was accurate). The math in `onProgressClick` is correct, so suspect the **hit area**: if `#progress-bar` (the fill) or any sibling overlaps/shadows `#progress-container`, the visual track and the clickable track diverge. Make the container the single full-width click target (fill layer `pointer-events: none`), and re-verify clicking at 30/60 lands at `00:30`. Low priority — just make the clickable region and the visible track coincide.

---

## Re-delivery checklist (attach with M00-v2.html)

1. Open via double-click, Console clean, intro → click → playback starts
2. Letterbox 2.39:1 correct after resize
3. Pause → click 30/60 on the track → timecode reads `00:30`, picture stable; two seeks to the same spot give identical frames
4. t=21.5 shows full blackout + SIGNAL LOST
5. Playing + cursor parked on the control bar for 4s → bar stays visible; cursor leaves → bar fades after 3s
6. No emoji anywhere (check muted/unmuted states both)
7. HUD sits inside the picture area
8. End of film → REPLAY works
9. Section index + total line count again

QA will re-run the full assertion suite (including the hover test that caught FIX-3) on v2.
