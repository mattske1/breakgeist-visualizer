# Breakgeist Visualizer

Audio-reactive "liquid metal" visuals — developed by Matthew Berman at
**Koryuai**, the tech division of Liberty Rose Studios, and built for
**Breakgeist** of Edged Out Records.

**Breakgeist** — *breakthrough* meets the *zeitgeist*, built on *break beats*
as the foundation of its musical identity — was the first AI-powered artist
launched with the label. This is the engine behind its videos.

Feed it any song. It analyzes the track's energy, brightness, and pulse, then
renders a flowing field of turbulent color that breathes with the music. Two
modes:

- **Loop** — a short, seamlessly-tiling loop (10s default). The song's *entire
  energy arc* is folded into the loop phase, so it feels like the track, not a
  random animation. Made for Shorts / Reels / TikTok promos.
- **Full-length** — the whole song rendered straight through as one continuous
  flow that never repeats, with the song's audio muxed in. Upload-ready.

## Quick start

```bash
pip install -r requirements.txt
# needs ffmpeg on PATH:  apt install ffmpeg  /  brew install ffmpeg

# 10-second seamless promo loop (vertical 1080x1920)
python3 visualizer.py song.wav --out loop.mp4 --seconds 10 --palette breakgeist

# full-length video with audio (cinema 1920x1080)
python3 visualizer.py song.wav --out full.mp4 --full-length --palette breakgeist
```

Or with Docker (ffmpeg included):

```bash
docker build -t breakgeist-visualizer .
docker run --rm -v "$PWD:/work" -w /work breakgeist-visualizer \
  song.wav --out loop.mp4 --seconds 10 --palette breakgeist
```

## Options

| Flag | Default | What it does |
|---|---|---|
| `--seconds` | 10 | loop length (loop mode) |
| `--full-length` | off | render the whole song with audio muxed |
| `--palette` | breakgeist | color palette (`breakgeist`, `frost`, `ember` — add your own in `PALETTES`) |
| `--fps` | 30 | frame rate |
| `--width/--height` | 540x960 | render resolution (upscaled after) |
| `--upscale-factor` | 2 | 4 = 1080x1920 from a 270x480 render (fast path) |
| `--no-upscale` | off | skip upscaling |
| `--scaler` | lanczos | ffmpeg upscale filter (`bilinear` is faster) |
| `--x264-preset` | medium | `veryfast` trades a little quality for speed |
| `--crf` | 20 | quality (lower = better/larger) |
| `--fast` | off | numba fast path — use it, it's much quicker |
| `--workers` | all | process pool size (ignored with `--fast`, which uses all cores itself) |
| `--seed` | 7 | visual variation seed |

Aspect trick: default 540x960 renders vertical. For square, pass
`--width 540 --height 540`; for cinema 16:9, `--width 960 --height 540`.
`--upscale-factor 4` then lands exactly on 1080x1920 / 1080x1080 / 1920x1080.

## Palettes

Palettes are just lists of color stops in `PALETTES` at the top of
`visualizer.py` — add one per band, per era, per mood. That's the whole
"theming" system, and it's enough.

## Performance (honest version)

This is CPU rendering. A 10-second promo loop takes a couple of minutes on a
modest box. A full-length song takes a while — tens of minutes — because every
frame is simulated, not played back. `--fast --fps 24 --x264-preset veryfast
--scaler bilinear` is the speed preset; a GPU box is the real 10x and is on
the roadmap.

## How it works

1. **Analyze** — the song is decoded to mono and three envelopes are extracted
   at 10 Hz: *intensity* (RMS energy), *brightness* (spectral centroid), and
   *pulse* (onset strength).
2. **Simulate** — each frame is a domain-warped fractal noise field (numba
   fused kernel), driven by the envelopes. Intensity warps the flow, brightness
   shifts the palette, pulse punches the contrast.
3. **Encode** — frames pipe straight into ffmpeg (libx264, faststart) with an
   upscale pass, so there's never a giant pile of PNGs on disk.

## License

MIT — use it for your own band, your own visuals, your own thing.
If it blows up, Breakgeist did it first.
