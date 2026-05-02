---
title: Sound, From Air to Bytes
description: A foundational mental model for sound — what it is physically, why sine waves are "pure," and how a computer turns continuous waves into the integers stored in a WAV file.
pubDatetime: 2026-05-02T08:27:43Z
tags: [audio, dsp, sound, fundamentals, concepts, wav]
---

You can copy a numpy script and produce a 440 Hz tone in five minutes. But the code only really *clicks* once you have a mental model of what sound actually is and how a computer represents it. This post builds that model from the physics outward, so the next time you read `np.sin(2 * np.pi * frequency * t)` it reads like a sentence rather than a magic spell.

## 1. Sound is a pressure wave

Sound is just **air pressure changing over time**. A speaker cone pushes forward and the air in front of it compresses; it pulls back and the air rarefies. Those alternating pressure changes propagate outward, hit your eardrum, and your brain interprets them as sound.

If you sit at one point in space and graph the air pressure there against time, you get a wiggly line — a **waveform**.

```mermaid
flowchart LR
    A[Speaker cone moves] --> B[Air compresses & rarefies]
    B --> C[Pressure wave travels]
    C --> D[Eardrum vibrates]
    D --> E[Brain interprets pitch & loudness]
```

Two properties of that wiggle do most of the perceptual work:

| Property      | What it is                       | What you perceive |
| ------------- | -------------------------------- | ----------------- |
| **Frequency** | Cycles per second (Hertz)        | Pitch             |
| **Amplitude** | How tall the pressure swings are | Loudness          |

440 Hz means the wave completes 440 full cycles every second. That specific frequency is the musical note A above middle C — the standard tuning reference for orchestras. Doubling the frequency (880 Hz) raises it by exactly one octave; halving it (220 Hz) drops it by one octave.

## 2. Why sine waves are special

A **sine wave** is the simplest possible periodic wiggle. Think of a frictionless mass on an ideal spring: its position over time traces out a sine curve. There's no other "stuff" mixed in — just one frequency, perfectly clean.

Acoustically, a sine wave sounds like a slightly hollow whistle or a tuning fork: pure, somewhat sterile. Real instruments produce a fundamental sine *plus* extra frequencies (harmonics) at integer multiples of the fundamental. Those harmonics are what make a violin and a flute playing the same note sound completely different — same pitch, different **timbre**.

The math behind one sine cycle:

$$
y(t) = A \cdot \sin(2\pi f \cdot t)
$$

- `A` is the amplitude (peak height).
- `f` is the frequency in Hz.
- `t` is time in seconds.
- `2π` is there because one full cycle of sine is `2π` radians; multiplying by `f · t` says "complete `f` cycles per second."

In code:

```python
waveform = amplitude * np.sin(2 * np.pi * frequency * t)
```

That single line *is* the wave. Everything else in an audio script is plumbing around it.

## 3. The continuous-to-digital problem

A real sound wave is continuous — at any instant in time, the pressure has some exact value. A computer can't store infinity points. So we make two compromises:

### Sampling: pick moments in time

We measure the wave's value at evenly spaced moments. **Sample rate** is how many measurements per second.

| Sample rate | Where you see it           |
| ----------- | -------------------------- |
| 8 kHz       | Old telephone audio        |
| 22.05 kHz   | Low-quality voice          |
| 44.1 kHz    | CD audio, most WAV files   |
| 48 kHz      | Video / film audio         |
| 96 kHz      | High-resolution / studio   |

Why 44100? That's the **Nyquist–Shannon theorem** in action: to faithfully represent a frequency `f`, you need to sample at *at least* `2f`. Human hearing tops out around 20 kHz, so 44.1 kHz captures everything we can hear with margin to spare.

Sample too slowly and you get **aliasing** — high frequencies fold down and masquerade as lower ones, the audio equivalent of wagon wheels appearing to spin backward in old films.

### Quantization: round each sample to a fixed precision

Each sampled value also has to be stored as a number with a finite number of bits. **Bit depth** controls how many distinct levels are possible:

| Bit depth | Range                  | Levels        | Typical use   |
| --------- | ---------------------- | ------------- | ------------- |
| 8-bit     | `[-128, 127]`          | 256           | Retro / chip  |
| **16-bit**| `[-32768, 32767]`      | 65,536        | **CD / WAV**  |
| 24-bit    | ~16.7 million          | 16,777,216    | Studio        |
| 32-bit float | ±1.0 with float precision | "huge" | DAW internals |

16-bit is the sweet spot for delivery: roughly 96 dB of dynamic range, which matches what most listening environments can actually distinguish.

So every digital audio file is, at its core, a long array of integers: `[s₀, s₁, s₂, …]` taken at regular time intervals.

## 4. From wave to integer array

Here's the full pipeline, written as one chunk so you can see how each parameter slots in:

```python
import numpy as np

frequency = 440.0      # Hz — the pitch
duration = 10.0        # seconds
sample_rate = 44100    # samples per second
amplitude = 0.5        # peak, on a -1.0..1.0 scale

n_samples = int(sample_rate * duration)         # 441,000 samples total
t = np.linspace(0, duration, n_samples, endpoint=False)
waveform = amplitude * np.sin(2 * np.pi * frequency * t)
samples = (waveform * 32767).astype(np.int16)
```

What each line does:

1. **`n_samples`** — how many measurements: `44100 × 10 = 441,000`.
2. **`t`** — an array of timestamps from `0.0` up to (but not including) `10.0` seconds. `endpoint=False` matters: including `t = 10.0` would start the next cycle.
3. **`waveform`** — for every timestamp, evaluate the sine. Result: 441,000 floats in `[-0.5, 0.5]`. Half-amplitude on purpose, to leave headroom and avoid clipping at the digital ceiling.
4. **`samples`** — multiply by 32767 (the maximum signed 16-bit integer, `2¹⁵ − 1`) and cast to `int16`. Floats become the integers a WAV file expects.

That's the whole chain: `time → continuous wave → samples → integers`.

## 5. What's actually in a `.wav` file

A WAV file is one of the simplest audio formats. It's just:

1. A small **header** (≈ 44 bytes) describing the format: number of channels, sample rate, bit depth, total length.
2. The **raw sample bytes**, packed back-to-back.

No compression, no surprises. Python's standard library writes one in four lines:

```python
import wave

with wave.open("tone.wav", "wb") as w:
    w.setnchannels(1)            # 1 = mono, 2 = stereo
    w.setsampwidth(2)            # 2 bytes per sample = 16 bits
    w.setframerate(sample_rate)  # 44100
    w.writeframes(samples.tobytes())
```

### Sanity-check the file size

For a 10-second, mono, 16-bit, 44.1 kHz WAV:

- 441,000 samples × 2 bytes/sample = **882,000 bytes** of audio data
- + ~44-byte header
- ≈ **882,044 bytes** total

That number isn't magic — it falls directly out of the parameters. If your output file is wildly different, something is wrong with your duration, sample rate, or bit depth.

## 6. Things to try once you have a tone

Once `tone.wav` plays, the interesting work begins:

- 🎵 Change `frequency` to `220` (one octave down) or `880` (one octave up) and listen — pitch perception is logarithmic, not linear.
- 📈 Plot the first ~0.01 seconds of `t` vs `waveform` and you'll see the sine shape directly. (At 440 Hz, one cycle is `1/440 ≈ 2.27 ms`, so 10 ms shows ~4.4 cycles.)
- 🎼 Add two sines: `sin(2π·440·t) + sin(2π·554·t)` — you'll hear an interval (a major third, roughly).
- 🌅 Multiply `waveform` by an envelope (e.g. a slow fade-in/fade-out) to remove the click at start and end. Abrupt amplitude changes contain *all* frequencies, which is what your ear hears as a click.
- 🎚 Sweep frequency over time — `frequency = 220 + 660 * (t / duration)` — and you'll synthesize a glide.

## 7. Glossary

A handful of terms worth knowing as you go deeper:

- **Hertz (Hz)** — cycles per second.
- **Sample** — one measurement of the waveform.
- **Frame** — one sample per channel; a stereo frame contains two samples.
- **Sample rate / framerate** — frames per second.
- **Bit depth / sample width** — bits per sample.
- **Mono / stereo** — 1 channel / 2 channels.
- **Clipping** — values that exceed the representable range, which sounds like crunchy distortion.
- **Headroom** — deliberately leaving a margin below the maximum (e.g. amplitude 0.5 instead of 1.0) to avoid clipping.
- **Nyquist frequency** — half the sample rate; the highest frequency you can faithfully represent.
- **Aliasing** — what happens to frequencies above Nyquist: they fold down and impersonate lower frequencies.
- **Timbre** — the "color" of a sound, determined by its harmonic content.

## Summary

A digital audio file is a deeply unglamorous thing: a long list of integers with a tiny header on top. The interesting physics — pressure waves, frequency, amplitude, harmonics — happens before the integers and after them, in air and in your ear. Everything in the middle is just bookkeeping: pick a sample rate, pick a bit depth, evaluate `A·sin(2π·f·t)` at each timestamp, scale, cast, and write.

Once you've internalized that pipeline, every other audio task — chords, envelopes, filters, FFTs, synthesis — is a variation on the same theme.
