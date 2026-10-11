---
name: motion-design-recipe
contains: premium-motion-design-system-for-code-generated-video
description: Use when a code-generated video must look "detail-rich / premium / techy" rather than merely functional. Provides the layering, easing, color, grain/glow, continuity, and pacing recipes that lift a passable motion-graphics render to a polished one. Triggers on "make it more detailed / more fun / more techy", "cinematic motion graphics", "premium explainer look", "add glow/grain/scanlines", "richer animation".
---

# Motion Design Recipe — the difference between "passable" and "premium"

A functional code-generated video has one clean element per shot and one easing curve.
A premium one has **depth (layers), rhythm (staggered easing), texture (grain/glow),
continuity (one idea that threads every shot), and pacing (fast/slow contrast)**.

Use this with `video-codegen` (engine selection / shot list) and `font-strategy`
(legible CJK text). All recipes here are deterministic: driven by the frame index only.

## 1. Depth — always three layers

Never render a flat shot. Every scene composes:

- **Back**: background texture (grid, gradient drift, subtle noise). Slowest motion.
- **Mid**: the subject (the element the shot is about). Normal motion.
- **Front**: HUD / labels / particles / captions. Fastest motion, highest contrast.

Rule: the further back, the slower and lower-contrast. Depth reads as production value.

```tsx
<AbsoluteFill>           {/* BACK: texture */}
  <Grid drift={0.15} />
  <div>{/* MID: subject */}</div>
  <HUD frame={frame} />  {/* FRONT */}
</AbsoluteFill>
```

## 2. Rhythm — staggered, layered easing

One spring on the whole scene looks cheap. Use **hierarchies**:

- **Primary** element: spring with overshoot (`damping ~12–18`).
- **Secondary** elements: same spring, offset by 30–60ms each (stagger).
- **Ambient** layer: never springs — it *drifts*. `sin(frame * k)` at very low amplitude.

```ts
// frames, not ms. 45ms @30fps ~= 1.35 -> use round(ms/1000*fps)
export const stagger = (i: number, gapFrames = 2, delay = 0) => delay + i * gapFrames;
export const easeOut = (t: number) => 1 - Math.pow(1 - t, 3);
export const easeInOut = (t: number) => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2,3)/2;
export const overshoot = (t: number, s=1.70158) => 1 + (s+1)*Math.pow(t-1,3) + s*Math.pow(t-1,2);
```

Never `linear`. If a value moves, ease it.

## 3. Texture — the four overlays that scream "expensive"

Apply globally in a wrapper, on top of everything, in this order:

1. **Grid**: 1px lines, ~64px pitch, opacity 0.04–0.08, slowly scrolling.
2. **Scanlines**: horizontal 2–4px bands, opacity 0.03, static (cheap, reads as CRT).
3. **Grain**: deterministic noise. Do NOT use `Math.random()` per frame (flicker + non-determinism).
   Use an SVG `feTurbulence` fixed `seed`, or precomputed dots tinted per frame index.
4. **Vignette**: radial gradient darkening the edges ~35%.

Optional 5th: **chromatic aberration** — red/cyan offset only on the HUD text, 1–2px.

```tsx
<svg style={{position:'absolute',inset:0,opacity:0.05,mixBlendMode:'overlay'}}>
  <filter id="g"><feTurbulence type="fractalNoise" baseFrequency="0.8" numOctaves="2" seed="7"/></filter>
  <rect width="100%" height="100%" filter="url(#g)"/>
</svg>
```

## 4. Light — glow, trails, additive energy

- Key elements get `boxShadow: 0 0 24px <accent>` or an SVG blur filter.
- **Trails**: draw the same point at `frame`, `frame-2`, `frame-4` with decreasing opacity — motion blur for free.
- **Energy**: a moving dot/line uses `mixBlendMode: 'screen'` / SVG `feGaussianBlur` to bloom.

## 5. Continuity — thread one element through every shot

Pick a motif (a current pulse, a cursor, a traveling particle block) and keep it visible
across all scenes, changing role. It turns a shot list into a story. Implement it as a
persistent `<AbsoluteFill>` layer rendered in `Main` outside the `<Sequence>`s, positioned
from the global `frame`.

## 6. Pacing — fast/slow contrast + beat grid

- Alternate dense ~2s shots with breathing 5–6s shots. Uniform 5s bores.
- Snap key animations to a **beat grid** (e.g. every 15 frames) so future music lands.
- Reserve a hard hold (no motion) for the single most important idea.

## 7. Color discipline

- 1 primary + 1 accent + a neutral ramp. Dark base (`#0a0e1a`).
- Let one accent own each scene; don't rainbow.
- Define ALL hex in `config.palette`. Fills via tokens, never inline one-offs.

## 8. Micro-detail checklist ("detail-rich")

- Live counters (HUD): voltage / clock / tokens-per-second as functions of `frame`.
- 1px hairlines, corner ticks, and a thin progress rail.
- Units and labels (`50 Hz`, `GB/s`, `0.3 kW.h`) — numbers make it feel real.
- A subtle "whoosh" placeholder: a thin arc that sweeps on scene entry.
- Consistent corner radius + margin across ALL scenes (from `config`).

## Anti-patterns
- Uniform easing everywhere -> cheap.
- Per-frame `Math.random()` grain -> flicker + non-deterministic.
- 6+ saturated colors -> rainbow, not premium.
- Static background on a "hero" shot -> dead frame.
- Every shot the same length -> no rhythm.
