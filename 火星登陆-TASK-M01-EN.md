# TASK-M01 · Act I "THE APPROACH" (0–16s)

> Foundation (M0) is **FROZEN**: timeline engine, camera system, UI shell, audio skeleton, render pipeline are accepted interfaces. Do NOT modify their behavior — only ADD content through the designed extension points (see §3). If you believe something must change, raise it in delivery notes instead of changing it.
> Deliver a complete single file as `M01-v1.html` with the self-check list.

---

## 0. What this round adds

Act I (0–16s) real content, replacing the placeholder. The placeholder test sphere/ring is **removed**. Act II/III time ranges stay black with their existing blackout demo REMOVED as well (the 21–22s SIGNAL LOST demo was a pipeline test; keep the blackout MECHANISM code intact, but it must not trigger in M01 — its cues come back in M2).

Between acts / where no content exists yet (16–60s), screen stays black (letterbox + subtle grain only, timecode running).

## 1. Shot script (director's intent — treat compositions as acceptance criteria)

### SHOT 1 · 0.0–4.5s — "A red dot in the void"
- Deep space: starfield (see §2-A), near-black background
- Mars: tiny dim red-brown disc, roughly 2–3% of frame height, slightly off-center (rule of thirds, upper-right quadrant)
- Camera: very slow push-in toward Mars (linear or gentle ease; move must be perceptible across the 4.5s)
- Titles: fade in at t=0.8s, fade out at t=4.2s — main line 「火星 · 距离 225 公里」 (white, light weight), sub-line `MARS — 225 KM` (monospace, letterspaced, dimmer, smaller). Position: lower-left INSIDE the picture area (see §2-D)
- Audio: low ambient drone fades in from silence (see §2-E)

### SHOT 2 · 4.5–9.0s — "Silhouette against the sun" (hard cut)
- The spacecraft (see §2-B) enters frame as a near-black silhouette; the sun sits just off-frame edge or behind the ship's rim, producing a hard rim-light edge on the hull
- Mars now visibly larger (≈15–20% frame height), ship small in frame traveling toward it
- Camera: slow push, slight lateral drift for parallax
- Intent: Interstellar-like restraint. NO lens flares stacked, no glowing engines — a faint engine glow is allowed

### SHOT 3 · 9.0–16.0s — "Through the window" (hard cut)
- Interior cockpit POV: dark window-frame silhouette framing the view (simple geometric frame geometry, near-black)
- Mars fills the lower-front view (large sphere, surface detail visible, see §2-C); thin blue atmospheric limb glowing along the planet's edge (this is THE hero visual of the act)
- A faint thruster glow at frame bottom edge (subtle, warm)
- HUD (see §2-D): velocity and altitude numbers ticking DOWN continuously (monotonic decreasing with t — pure functions of t)
- Camera: very slow descent/push toward Mars, slight drift
- Intent: quiet technical tension. The numbers doing the storytelling

## 2. Asset & system specs

**A. Starfield** — THREE.Points, ≤3000 vertices, size attenuation on, per-star twinkle (subtle opacity/size oscillation via shader or attribute phase; deterministic function of t, NOT random per frame). Static positions. Budget: 1 draw call.

**B. Spacecraft** — procedural low-poly, silhouette-first: capsule/box hull + service module + dish antenna + lander stage (this SAME model gets reused landing in Act II — build it as a group with named children: `hull`, `serviceModule`, `antenna`, `heatshield`, `legs` — legs/heatshield may be simple boxes for now, detailed in M2/M3). Materials: dark gray, high roughness; rim light does the separation. Sun = strong DirectionalLight from behind/side + very low ambient.

**C. Mars sphere** — shader-based procedural surface (fragment shader fbm noise): rust-orange base + darker basaltic regions + subtle polar cap whitening. Plus atmosphere limb: thin blue-white fresnel rim (backside sphere or shader rim term). Texture resolution n/a (procedural), no large textures. The limb glow is the money shot — get the falloff tight and physical.

**D. HUD + Titles (DOM overlay, inside picture area)**
- Both must be positioned **inside the letterbox picture area** (compute picture-area rect in JS on resize; never overlap black bars — this was a v1 finding)
- HUD: monospace, small, low-opacity white, bottom-left of picture area; VEL in m/s and ALT in km; values = deterministic functions of t (e.g. start 1680 m/s → 1620 m/s across the shot; make the deceleration profile smooth)
- Titles: DOM elements, CSS opacity transitions driven from applyState (opacity as pure function of t — no CSS transition classes for timeline-driven fades; compute opacity in JS each frame)

**E. Audio (skeleton only, real score is M5)**
- Ambient drone: 1–2 detuned low oscillators + lowpass filter, gain envelope: fade in 0→3s, constant to 16s, fade out by 17.5s — all as deterministic gain scheduling on seek-correct paths (envelope = function of t applied per tick, or scheduled cues — must survive seek/scrub consistently per the frozen AudioEngine contract: only naturally-crossed cues fire)

## 3. Extension points (how to add without touching frozen code)

- Shots/props: extend `applyState(t)`'s content section and the `CAMERA_MOVES` table (hard cuts between shots; one move per shot)
- Add new CONFIG blocks for Act I (colors, sizes, timings) — no magic numbers
- Titles/HUD: add a `TITLES` table `{start, end, main, sub}` + render loop reading it
- Remove placeholder content; keep `updatePlaceholderContent`'s slot structure (rename to `updateActContent` if cleaner)

## 4. Performance budget (same as SPEC)

draw calls ≤ 40 total for this act (starfield 1 + mars 1–2 + atmosphere 1 + ship ~1 + cockpit 1 + glow sprites ≤2) · stars ≤ 3000 · no render-target-size textures beyond 1024 · keep 60fps headroom on a real machine

## 5. Self-check list (verify manually, attach results)

1. Console clean (file:// double-click open)
2. Shot 1: starfield twinkles subtly; Mars disc small upper-right; title fades in/out at stated times; slow push perceptible
3. Shot 2 hard cut: ship silhouette + rim light read clearly against black; Mars ~1/5 frame height
4. Shot 3 hard cut: window frame silhouette; Mars surface detail visible; blue limb glow tight along the edge; HUD numbers tick down monotonically; thruster glow subtle
5. All text/HUD inside picture area at any window size (resize test)
6. Seek determinism: pause at 12s, screenshot; seek away and back; identical frame
7. 16–60s: black picture, timecode runs, no blackout demo
8. Audio: drone fades in after clicking begin; seek into 5s → drone audible (envelope consistent); seek past 17.5s → silent
9. Frozen interfaces untouched (control bar behavior, letterbox, audio contract)
10. Draw call count reported (renderer.info in console log once at t=10s)

## 6. Deliverable

`M01-v1.html` complete single file + self-check results + line count + section index + one screenshot per shot (t=2s, 6.5s, 12s) if you can capture them. QA will run full assertions + frame-grading review on all three shots.
