# Scroll

A TouchDesigner project that takes a scanned occult scroll print and layers real-time
VFX over it, for projection mapping onto a large physical print of the same artwork.

The guiding principle: **the print already carries the artwork — the projector adds
light, not image.** Output should be mostly black, with illumination only where an
effect is wanted.

## Files

| Path | Purpose |
| --- | --- |
| `scroll.toe` | Main TouchDesigner project |
| `scroll.4.toe` | Latest incremental save (byte-identical to `scroll.toe`) |
| `Backup/scroll.[1-3].toe` | TouchDesigner auto-backups |
| `scroll.png` | Source artwork, 1066 x 1475 (portrait, ≈0.72:1), 8-bit RGB |

## Source artwork

Woodcut/linocut-style print on aged parchment: black ink figures with red accent
glyphs. Three figures (crowned warrior with sword, central skeletal form, robed
figure), inverted-triangle motifs, and constructed-script text blocks top and bottom.

The two-tone ink (black + red) is the useful structure here — it gives two independent
mattes to drive effects from.

## Network

Everything lives in `/project1` (Container COMP).

```
moviefilein1 ──> scroll.png
      │
      └────────────────────> sub1 (in 1)
feedbackEdge ──────────────> sub1 (in 0)
                              │
                            lookup1 ──> null1
                              ▲
                            ramp1  <── ramp1_keys (DAT)

bloom     [tuned, currently unwired]
noise1    [CHOP, hermite — exports to bloom's Threshold]
```

### Components

- **`feedbackEdge`** — palette component (v8.0.4). Blur/feedback/edge chain with hue
  shift and dry/wet mix. Only `Drywet` (0.946) is off default.
- **`bloom`** — palette component (v1.1.1). Tuned: Blursize 13, Blackman blur,
  Preshrink 3, Iterations 7, Intensity 1.79, Glow 0.346, rgb32float. Its `Threshold`
  parameter is driven by `op('noise1')['chan1']` for animated flicker.
- **`noise1`** — Hermite noise CHOP (seed 1.33, period 1.13, amp 0.38, offset 0.62)
  running at timeline rate. Modulation source for the bloom threshold.
- **`sub1` / `lookup1` / `ramp1`** — subtract the feedback pass from the source, then
  colour-grade through a 2-key ramp LUT.

## Current state

The look-dev chain is roughed in. It is **not yet a working projection setup** — the
following need addressing:

1. **`feedbackEdge` has no input wired.** The COMP-level node has no input connection,
   so its internal `in1` falls through to the palette's placeholder `moviefilein1`,
   still pointing at `app.samplesFolder + '/Map/Nature/Movie.2.mp4'`. `sub1` is
   currently subtracting a stock nature clip from the scroll.
2. **`bloom` is orphaned.** Same placeholder movie inside, and its `out1` connects to
   nothing. The tuning above was done against the demo clip, so the values will need
   revisiting once it sees real input.
3. **Nothing renders in Perform mode.** `/project1`'s Background TOP is set to
   `./out1`, but no `out1` exists inside `project1`. The `display` flag on `null1`
   only affects the network editor. Add an Out TOP or repoint the parameter.
4. **Resolution and aspect mismatch.** The container is 1280x720 landscape; the print
   is portrait. `sub1` inherits resolution from input 0 — currently the 1080p demo
   movie. The whole chain should be locked to one portrait resolution matched to the
   projector's native output.
5. **No mapping stage exists.** No kantanMapper, no corner-pin, no Window COMP
   targeting the projector.

## Roadmap

**Matte extraction.** Derive two mattes from `scroll.png`: an ink matte (luminance
key / threshold) and a red-glyph matte (HSV key on hue). Feed the *mattes* into
`feedbackEdge` and `bloom` rather than the full image — this fixes issues 1 and 2 at
once and keeps the projector from washing the paper ground.

**Effect passes.** Glow crawling along the red sigils; ember flicker in the central
skeletal figure; a slow breathing bloom on the triangle motifs.

**Alignment layer.** A toggle that projects the unmodified `scroll.png` at full
brightness, for physically lining the projector up with the print. Switch to
VFX-only once aligned.

**Output.** Lock the comp to the projector's native resolution, add a Window COMP
bound to the projector display (borderless, correct monitor), and add a mapping stage
(kantanMapper is the quickest route).

## Notes for projecting

- Projected black is grey haze, not black — every lit pixel is a deliberate choice.
  Keep the frame dark and let the print's own ink do the drawing.
- Match the comp aspect to the print, not to a standard video format, or the mapping
  stage inherits a distortion it has to undo.

## Requirements

TouchDesigner (built against 2023.11225 / 2022.21870 palette components).
Open `scroll.toe`.
