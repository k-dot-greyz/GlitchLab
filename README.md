# GlitchLab 🎨

> Digital glitch art tool — pixel sorting, displacement shaders, real-time WebGL

Hosted at **glitchlab.live** (pending DNS). Part of the **GlitchWorks** constellation.

---

## What It Does

Real-time browser-based glitch art creation engine. Upload an image or connect a live camera feed, apply destructive/constructive glitch algorithms, export as image or looping GIF/video.

---

## Architecture

```
Input (image / camera / URL)
    → WebGL shader pipeline
    → Glitch effect stack (stackable, parameterised)
    → Real-time canvas preview
    → Export (PNG / WebP / GIF / MP4)
    → Optional: MIDI CC control (via midi-gem)
```

## Stack

| Layer | Tech |
|---|---|
| Runtime | TypeScript + Vite |
| Rendering | WebGL2 + GLSL shaders |
| UI | Astro (or vanilla) + CSS custom properties |
| Export | `gif.js` + `ffmpeg.wasm` |
| MIDI control | `WebMIDI API` → midi-gem |

## Glitch Effects (planned)

- [ ] **Pixel sorting** — horizontal/vertical, by luminance/hue/saturation
- [ ] **Displacement map** — custom noise, sine wave, image-based
- [ ] **Chromatic aberration** — RGB channel offset
- [ ] **Scan lines** — CRT simulation
- [ ] **Bit crush** — colour depth reduction
- [ ] **Databend** — raw byte manipulation simulation
- [ ] **Bloom / bleed** — light leak effects
- [ ] **Custom GLSL** — user-editable shader input

## MIDI Control

Each effect parameter mappable to a MIDI CC via `WebMIDI API`. Works with midi-gem for routing from any hardware controller.

## Related

- [`midi-gem`](https://github.com/k-dot-greyz/midi-gem) — MIDI I/O routing
- [`glitch-that-shit`](https://github.com/k-dot-greyz/glitch-that-shit) — browser extension sibling
- [`GlitchWorks`](https://github.com/k-dot-greyz/GlitchWorks) — parent project
- `dex_id`: TBD — tracked in [dev-master/dex](https://github.com/k-dot-greyz/dev-master/tree/main/dex)

## Status

`spec-complete` → `prototype-pending`

---

*Unacceptable conditions will be glitched into submission.*
