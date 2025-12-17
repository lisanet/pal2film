# ❓ FAQ – Pitch, PAL Speedup, DVDs and Blu-rays

This FAQ explains common questions and misconceptions about PAL speedup, pitch correction, and differences between PAL DVDs and Blu-ray releases.

## ❓ Why does PAL audio sound higher in pitch?

PAL speedup increases playback speed by ~4.3%.

If audio pitch is not corrected, this results in:
- higher voices
- faster music
- shorter runtime

This is extremely common on PAL DVDs.


## ❓ Was PAL audio usually pitch-corrected?

### Short answer:
**Often no.**

### Detailed answer:

| Medium      | Typical behavior |
|-------------|------------------|
| PAL DVD     | Usually **not pitch-corrected** |
| PAL Blu-ray | Mixed |
| NTSC BD     | No speedup |

Pitch correction costs time and money and was often skipped, especially for English audio tracks.


## ❓ Why do English tracks usually benefit from lowered pitch?

Most English PAL releases:
- were sped up
- kept higher pitch
- sound unnaturally sharp

Reversing PAL speedup restores:
- original voices
- correct musical pitch
- theatrical timing

This is why the `-np` option is often recommended for English audio.


## ❓ What about dubbed audio tracks?

Many localized dubs (German, French, etc.):
- *were* pitch-corrected during PAL mastering

In those cases:
- PAL audio already has correct pitch
- reversing speedup will lower pitch again

Whether this is acceptable depends on preference and reference material.


## ❓ How can I tell if pitch correction was applied?

There is no reliable metadata flag.

Possible indicators:
- Compare music pitch to NTSC / streaming version
- Check runtime differences
- Listen for unnaturally high voices
- Look for studio documentation (rare)


## ❓ What is the “correct” choice?

There is no universal answer.

| Goal | Recommendation |
|-----|----------------|
| Film accuracy | Use `-np` |
| Preserve PAL dub | Default mode |
| English audio | Usually `-np` |
| Dubbed audio | Depends |

pal2film gives you the choice.


## ❓ Do Blu-rays still have PAL speedup?

### Usually no — but exceptions exist.

- Blu-ray supports 23.976 fps natively
- Most modern releases use native film speed
- Some early or lazy PAL masters still use 25 fps

Always check the actual frame rate.


## ❓ Does pal2film re-encode video?

**No. Never.**

Video is:
- copied bit-for-bit
- only timestamps are changed
- no quality loss occurs


## ❓ Is this the same as frame interpolation?

No.

pal2film:
- does not create or remove frames
- only adjusts playback timing


## ❓ Is this archival-safe?

Yes.

pal2film is suitable for:
- archival restoration
- reference encodes
- film-accurate libraries

As long as the source is truly film-based PAL material.


## 📌 Final Advice

PAL speedup is a historical artifact.

pal2film exists to undo it — precisely, transparently, and without quality loss.
