## 🎬 pal2film – Revert PAL Speedup: 25 fps -> 23.976 fps

**pal2film** is a Z shell script that reverses the traditional **PAL speedup** applied to film-based video material.

It converts **25 fps PAL masters** back to their original **23.976 fps (24000/1001) film speed**, while:
- copying the video stream without re-encoding
- correcting timestamps and container timebase
- properly adjusting audio duration and pitch
- updating chapters and subtitles accordingly

## ✅ Features

* Lossless video stream passthrough, no re-encoding
* Exact 25 → 23.976 fps timing restoration
* Audio speed and pitch control
* Chapter timestamp correction
* Subtitle timestamp correction (separate srt only)
* Multi-audio stream support
* Language metadata preservation
* Dry-run mode for inspection

## 🔊 Audio Format Handling

- AC3, AAC, MP3 input supported
- Optional downmixing to Dolby Pro Logic II matrix encoding including LFE channel
- Dynamic Range Compression adjustable
- Manual volume adjustment
- Optional peak volume normalization 

## ⚙️ Usage

```
pal2film [options] filename
change from 25 fps to 23.976 fps without reencoding video
and adjusting audio so there's no change in audio pitch
options:
     -i:   input pal video
     -a:   file which contains audio streams to process
     -aac: force aac, downmix if necessary
     -drc: set dyamic range compression used for ac3 input (default 0.0)
     -b:   set audio bitrate (deafult ac3: 448k, aac: 256k)
     -np:  no pitch correction (audio speed change only)
     -peak: set peak volume normalization (in dB, e.g. -1.0dB)
     -vol: set volume adjustment (ffmpeg volume filter syntax, default 0.0dB)
     -o:   output filename
     -s:   process only subtitle (srt only)
     -n:   dry run (no conversion)
     -h:   this info
```

## 🛠 Requirements

pal2film uses the following tools, which need to be installed.

- ffmpeg
- ffprobe
- mediainfo

If ffmpeg has been compiled eith support for the libsoxr resampling engine, pal2film will automatically use this resampler, otherwise the standard ffmpeg swr engine is used. Although ffmpeg's swr is really good in resampling, libsoxr is known for its very high quality and should be preferred. You need to compile ffmpeg yourself, if you want support for libsoxr.

## 🚀 Installation 

To install pal2film, simply clone this repo

```
   git clone https://github.com/lisanet/pal2film.git
```
Then navigate to the directory and copy the script into a directory in your PATH (e.g. `/usr/local/bin`)

```
cd pal2film
sudo cp pal2film /usr/local/bin
```

## 🧪 Examples

### Basic Conversion

```
./pal2film -i input_pal.mp4 -o output_film.mp4
```

### Disable Pitch Correction (recommended for many English tracks)

```
./pal2film -i input_pal.mp4 -np -o output_film.mp4
```

### Downmix 5.1 AC3 to Dolby ProLogic II (including LFE channel) with audio bitrate 128kbps

```
./pal2film -i input_pal.mp4 -aac -b 128k -o output_film.mp4
````

### Process Subtitles Only

```
./pal2film -s subtitles.srt
```

## 📚 Technical Background

### Film vs. PAL Framerates

Almost all motion pictures are produced at 23.976 fps. The PAL television standard, however, uses 25 fps.

Instead of frame interpolation, PAL releases historically applied a speed-up factor of `25 / 23.976`,
or, more precisely, `25 / (24000 / 1001) = 25050 / 24000 = 1.04375`.


This speedup results in a ~4.3% faster playback, shorter runtime and increased audio pitch (unless corrected).

## 🔁 What pal2film does

pal2film now performs the **exact inverse operation** by using a

```
speed factor = (24000 / 1001) / 25000 = 24000 / 25050 = 0.9580838323
```

### Video Processing

For video processing thers is **no video re-encoding**. The video stream is copied bit-exact. Only frame timestamps are recalutated using the above mentioned factor. This ensures correct NTSC-film timing inside the container and avoids quality loss and preserves the original PAL master.


### Audio Processing

Audio must be slowed down by the same factor as the video to maintain A/V sync. This can only be down by resampling the audio stream an reencoding the audio. A one time reencoding will likeyl not introduce noticable artefacts. In order to get the best results, pal2film checks if ffmpeg has been compiled with libsoxr support. libsoxr is a resampling engine, known for its very high quality. Although the standard ffmpeg resampler ist quiet good, libsoxr shpuld be used where available. To get libsoxr support, you need to compile ffmpeg an libsoxr from source.

Since audio has always be resampled an reencoded, pal2film offers some usefull audio manipulating options, so that it's not necessary to re-encode audio in multiple steps.

These options are:
- pitch correction
- downmixing 5.1 ac3 to Dolby ProLogic II, including the LFE channel
- adjustable dynamic range compression
- volume manipulation
- peak normalization

### Pitch Correction

By default, pal2film preserves the original pitch. This is ideal for PAL releases where pitch correction was already applied

With the `-np` option, pal2film changes playback speed only, which lowers pitch accordingly and restores original theatrical pitch for many English audio tracks.

This is often **desired**, because many PAL releases did **not** pitch-correct English audio during the original speedup.

See the FAQ for more information.

### Downmixing 5.1 to Dolby ProLogic II

Dolby ProLogic II is a 2 channel matrix encoding, which, when played through a Dolby capable decoder, will restore the original multi-channel layout quiet well. By downmixing 5.1 AC3 audio to Dolby ProLogic II you get stero compatible audio while still preserving the surround information, when played on a capable setup.

pal2film enhances the standard downmix by including the low frequency effects (LFE channel) into the downmixed audio stream, so you don't loose those sound effects.

### Dynamic range compression

When reencoding 5.1 AC§, pal2film uses no dynamic range compression (drc). To apply range compression, use the `-drc`option. A value of `0`means no compression, a value of `1`means fill compression.

### Volume manipulation

Although the overall volume is preserved when re-encoding or downmixing, some audio streams might need some volume attenuation. This can be done with the `-vol`option. You can either specify a factor (e.g. `-vol 2.0` means doubling the volume) or a decibel value, eg. `-vol 4.5dB` will give an attenuation of 4.5dB.

### Peak normalisation

pal2film offers some basic peak normalisation with the `-peak` option. You must specify some headroom value, eg. `-peak -1.0dB` will give you a headroom of 1dB. Be very careful with this option and always use some headroom to avoid clipping, especially when downmixing.

## 🧾 Chapters and Subtitles

pal2film also adjusts time-based metadata:

- Chapters: all timestamps rescaled using the same speed factor
- SubRip Subtitles (`.srt`) Must be processed independently using `-s`


## ⚠️ Limitations

* Intended for **film-based PAL material only**
* Input video must be **25 fps**
* Not suitable for native 25 fps productions (TV studio content, live video)
* Pitch correctness depends on original PAL mastering


## 📬 License

Feel free to copy, modify, distribute this script under the terms of the BSD 2-Clause License.


## 🤝 Contributing

Bug reports, technical improvements, and pull requests are welcome.  
Please include detailed media info and ffmpeg logs when reporting issues.


## ⚠️ Disclaimer

pal2film is provided **"as is"**, without warranty of any kind.  
Always verify results and keep backups of your original media.





