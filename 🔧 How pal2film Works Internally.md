# 🔧 How pal2film Works Internally

This document describes the internal processing steps of **pal2film** and explains how PAL speedup is reversed without re-encoding the video stream.

## 🧮 Core Timing Constants

pal2film uses rational timebase math instead of rounded frame rates:

| Meaning            | Value |
|--------------------|-------|
| PAL timebase       | 25000 |
| Film timebase      | 24000 |
| Film frame rate    | 24000 / 1001 |
| Factor             | 25000 / (24000 / 1001) = 25025 / 24000 |

These values ensure exact timestamp conversion without cumulative drift.


## 🎞 Video Processing

### 1. Video Stream Analysis

The script uses `ffprobe` to detect:

- codec (`h264`, `hevc`, etc.)
- frame rate (must be `25`)
- container time base (`tbn`)

If the video is not 25 fps, processing is aborted.


### 2. Container Timebase Correction

If the container timebase is not already `24000`, pal2film performs a **lossless remux**:

ffmpeg -i input.mp4 -c copy -video_track_timescale 24000 remux_video.mp4

This ensures correct NTSC-film timing inside the container without touching the encoded video frames.


### 3. Timestamp Rescaling

During final muxing, timestamps are rescaled using:

-itsscale (25025 / 24000)

This stretches the timeline back to film speed while copying the video bit-for-bit.

No frames are dropped, duplicated, or re-encoded.


## 🎧 Audio Processing

Each audio stream is processed independently.

### 1. Codec Detection

For each audio stream, pal2film detects:

- codec (AC3, AAC, MP3)
- sample rate
- channel layout
- language metadata
- start offset

MP3 is automatically converted to AAC.


### 2. Speed Adjustment Modes

#### Default: Pitch-Corrected Mode

atempo = 24000 / 25025

This:
- slows audio playback
- preserves pitch
- maintains sync with video

Used when the original PAL audio was pitch-corrected.


#### No Pitch Correction (`-np`)

asetrate = sample_rate × (24000 / 25025)
aresample = sample_rate

This:
- slows audio
- lowers pitch naturally
- restores theatrical pitch

Recommended for most English PAL DVD and Blu-ray releases.


### 3. Resampling Engine

pal2film prefers **libsoxr** if available: aresample=resampler=soxr

Falls back to ffmpeg builtin software resampler (SWR) if soxr not present.


### 4. Optional Processing

- Dynamic Range Compression for AC3 (`-drc`)
- Dolby Pro Logic II downmix
- Peak normalization (`-peak`)
- Manual gain adjustment (`-vol`)
- Forced AAC output (`-aac`)


## 🧾 Chapters

Chapters are extracted as FFmetadata:

ffmpeg -i input.mp4 -f ffmetadata chaps.txt

Each timestamp is rescaled using the same PAL→film factor.


## 💬 Subtitles (SRT)

SRT subtitles are processed line-by-line:

- timestamps parsed to milliseconds
- converted using rational timebase math
- rewritten with preserved formatting

Subtitle processing must be run independently.


## 🧩 Final Muxing

All elements are muxed together using stream copy:

- video (unchanged)
- processed audio streams
- updated chapters
- preserved language metadata

This ensures:
- zero video quality loss
- exact sync
- correct film timing


## 🧪 Dry Run Mode

With `-n`, pal2film prints all ffmpeg commands without executing them.

Useful for:
- debugging
- learning
- verifying filters and parameters


## 📌 Summary

pal2film reverses PAL speedup by:

- exact rational time math
- container-level timestamp correction
- lossless video handling
- configurable audio pitch behavior

No heuristics. No guessing. No quality loss.