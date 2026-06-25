# lufs-vectorscope-fog-demo

An audio-reactive **vectorscope Lissajous** behind a **brand-cycling aurora fog** — the intended
landing-page background for [lufs.audio](https://lufs.audio). Self-contained static HTML, no build
step, no dependencies beyond a pinned CDN. Built by **Amacher** (LUFS design agent) with Daniel
Ramirez; the full design exploration lives in the `agent-knowledge` repo
(`docs/product/reactive-backgrounds/`).

> **TL;DR:** open `index.html`, click **Enable audio**, then hit **⚙ Dials** to play with it.

## What it is

Two layers, one canvas:

1. **Vectorscope (foreground).** A true audio vectorscope — a Lissajous figure of the Left × Right
   channels, rotated 45° via the **Mid/Side** transform (`X = (R−L)/√2`, `Y = (L+R)/√2`) so a mono
   signal reads as a vertical line. It reacts to the *actual* audio in real time, with a CRT-style
   **phosphor trail** (fading after-images).
2. **Brand-cycling aurora fog (background).** A fragment-shader fluid mesh gradient rendered as a
   translucent fog *on top* of the scope, so the busy trace reads as veiled / receded. Its accent
   colour slowly rotates across the LUFS palette (teal → blue → gold → rust), slew-limited so it
   stays calm.

Before you press **Enable audio**, the visuals idly wisp (calm drift, faint vertical breath). Press
play and it locks to the live stream.

## Run it

It's a single static file — no install:

```bash
# any static server works, e.g.
python3 -m http.server 8080
# then open http://localhost:8080
```

Or just open `index.html` in a browser. (A server is recommended so the Web Audio fetch/CORS path
behaves exactly like production.)

## Audio

The track is streamed from the LUFS catalog signing worker:

```
https://stream.lufsaud.io/api/stream/lufs-59b422dd   →  302  →  presigned R2 URL  ("elegant-picking")
```

The cross-origin **AnalyserNode** needs CORS on the **R2 bytes**, not just the worker — the bucket
CORS policy (`AllowedOrigins: ["*"]`) must be applied (`worker/scripts/apply-r2-cors.mjs` in
`lufs-catalog-website`). The page fetch-probes CORS and falls back to a calm idle wisp if it's off.

## The dial panel (dev tool)

**⚙ Dials** (top-right) opens a live tuning panel — every parameter, default = the shipped value,
ranges pushed to extremes. **Copy settings** dumps the current values as JSON. It's here on purpose:
it's a fun dev surface and the place to re-tune before baking new defaults.

### Shipped defaults (v5)

```json
{
  "scopeGain": 1.7, "scopeBright": 0.9, "scopeTrail": 16, "scopeDecay": 0.52,
  "scopeEnergy": 0.7, "scopePoints": 2048, "scopeIdle": 0.61,
  "fogOpacity": 0.9, "fogBright": 0.65, "fogDrift": 0.06, "fogWarp": 1.2,
  "fogBloom": 0.32, "fogDensity": 1,
  "colorCycle": 0.01, "colorAudio": 3, "colorSlew": 0.2,
  "reactSmooth": 0.3, "idleDrift": 0.3, "engageTime": 2.2,
  "cursorCoreSize": 6, "cursorOrbitSize": 4, "cursorOrbitSpeed": 0.018,
  "cursorOrbitAudio": 0.05, "cursorOrbitAudioSize": 3, "cursorIdleScale": 0.5, "cursorRadius": 9
}
```

On **Enable audio**, the scene doesn't snap to full reactivity — it **ramps from the idle wisp into
the live signal** over `engageTime` seconds (an exponential engage envelope on both the bands and
the waveform), then falls back faster on pause. Fully responsive (desktop → mobile), honours
`prefers-reduced-motion`, and the custom cursor hides on touch.

| Group | Dial | Meaning |
|---|---|---|
| Vectorscope | Gain | trace amplitude |
| | Trace brightness | additive line brightness |
| | Phosphor trail (frames) | how many fading after-images |
| | Phosphor decay | tail falloff (higher = longer) |
| | Energy → brightness | how loudness scales brightness |
| | Trace resolution | points along the trace |
| | Idle vertical wisp | rest-state vertical line amount |
| Fog | Fog opacity | how much the fog veils the scope |
| | Fog brightness / Drift / Warp / Bloom / Density | aurora look + motion |
| Colour | Auto cycle speed | autonomous hue rotation |
| | Bass → colour | how much bass pushes the hue |
| | Slew rate | how fast colour reacts (lower = calmer) |
| Reactivity | Level smoothing | analyser easing |
| | Pre-audio wisp | idle drift intensity |
| | Audio engage ramp (s) | idle → reactive transition time on enable |
| Cursor | Center dot size | the classic dot at the pointer |
| | Orbit dot size / speed | the dot orbiting the ring (base, when engaged) |
| | Bass → orbit speed / size | how much the music drives the orbit |
| | Idle scale | how much slower/smaller the orbit is when idle vs engaged |
| | Orbit radius padding | gap between ring edge and orbit path |

## Cursor

Custom cursor: the classic **center dot** at the pointer, a lerp-following **ring**, and a dot that
**orbits** the ring clockwise (adapted from the lufs.audio site). The orbit is calm by default and
scales with the **engage** envelope + live **bass** — slowest/smallest when idle, fuller once the
music is engaged. All of it is dial-controlled (Cursor group).

## Stack

- Self-contained `index.html` — inline CSS/JS, no build step.
- `three@0.128.0` global build via jsDelivr (the only external dependency).
- Web Audio API: `ChannelSplitter` → per-channel time-domain analysers (the vectorscope), plus a
  full analyser for bands + spectral centroid.
- Honors `prefers-reduced-motion`; hides the custom cursor on touch.

## `archive/`

The exploration that led here, oldest → newest:

- `01-fluid-mesh-v1.html` — fluid mesh gradient × point-cloud (3 looks)
- `02-audio-reactive-v2.html` — first wire to the real stream (4 looks, incl. the first Lissajous)
- `03-lissajous-v3.html` — Lissajous focus (real vectorscope + 3 abstract variants)
- `04-vectorscope-fog-v4.html` — the combined view + the dial panel (pre-tuning)

## Credit

Design & build: **Amacher** for **LUFS Audio** / Daniel Ramirez. Vectorscope DSP per standard audio
metering (RTW / iZotope / audiomasterclass). Not affiliated with the reference sites studied during
the exploration.
