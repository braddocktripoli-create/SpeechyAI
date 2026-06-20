# SpeechyAI

SpeechyAI is a browser-based tool that analyzes the clarity of an /s/ sound from a short audio recording, with a focus on detecting lisp-like patterns. Everything runs locally in the browser — there's no server, no account, and no audio is ever uploaded anywhere.

## What it does

Lisps are typically diagnosed by ear, by a speech-language pathologist listening for how an /s/ sound is produced. SpeechyAI takes a different angle: it analyzes the actual frequency content of a recorded /s/ sound and produces a numeric "clarity score," along with a comparison against a growing set of labeled example recordings.

The core idea is simple. A clean /s/ sound concentrates its energy in a higher frequency band (roughly 4–8kHz, the "sibilant" range). A lisped or distorted /s/ tends to leak more energy into a lower band (roughly 2–4kHz). SpeechyAI measures the balance between these two bands and uses that ratio as its primary signal.

## How it works

**1. Record or upload.** You can record directly in the browser (saying a word with a clear /s/ sound, like "sun" or "sister") or upload an existing audio file.

**2. Trim to just the /s/.** A waveform editor lets you drag handles to isolate exactly the portion of the recording you want analyzed — similar to trimming a voice memo. This matters because the analysis is sensitive to what's actually in the clip; trimming out silence, breath noise, or surrounding words gives a cleaner result.

**3. DSP analysis.** The trimmed audio goes through a short-time Fourier transform (STFT) pipeline:
- Volume is normalized first, so how loud or close-to-the-mic the recording was doesn't affect the score.
- The audio is broken into overlapping frames, and energy in each frame is measured across the frequency spectrum.
- Rather than relying on a single "loudest" frame (which is often a noisy onset transient), the score is computed as the **mean ratio across the sustained portion** of the sound — the frames where energy is consistently above a threshold, which is more representative of the actual /s/ articulation.
- A few additional features are extracted alongside the main score: how the ratio shifts from onset to sustained portions, how stable the ratio is moment-to-moment, and the spectral centroid (a measure of overall brightness).

**4. Clarity score.** The result is displayed as a single number, "Your Clarity Score," along with the supporting frequency-band measurements that produced it.

**5. Adaptive comparison.** This is the more experimental part. Because the same raw dB measurement can mean very different things for different speakers (vocal tract size, mic setup, recording conditions all affect the absolute number), a fixed universal severity scale is unreliable. Instead, SpeechyAI uses a k-nearest-neighbors comparison: you can label any analyzed recording as Clear /s/, Mild, Moderate, or Severe, and future recordings get compared against that growing personal history rather than an arbitrary fixed cutoff. The more examples you label, the more the comparison reflects real, identified patterns instead of guesswork.

The app ships with 8 built-in reference examples (4 paired sets of clear vs. lisped /s/ recordings) to seed this comparison from the start, so it isn't starting from nothing.

## How to use it

1. Open `index.html` in a browser (Chrome recommended — microphone recording requires running the page as a real standalone page, not embedded in another tool).
2. Click record and say a word with a clear /s/ sound, or upload an existing audio file.
3. Drag the trim handles to isolate just the /s/ portion.
4. Click Analyze.
5. Review the clarity score and the supporting DSP measurements.
6. Optionally, label the result (Clear /s/, Mild, Moderate, Severe) to add it to your comparison history — this improves the accuracy of future estimates.

## What this is — and isn't

This is a signal-processing project built to explore whether lisp severity can be estimated from audio alone, not a diagnostic or medical device. It hasn't been validated against clinical ground truth, and the severity labels are based on self-reported, by-ear judgments rather than a speech-language pathologist's assessment. It's best understood as a measurement and comparison tool — useful for tracking change in a single person's /s/ sound over time, or as a starting point for a conversation with someone who actually does this clinically, rather than a replacement for one.

## Technical notes

- Single self-contained HTML file — no build step, no dependencies, no backend.
- Uses the Web Audio API for recording and analysis, with a custom JavaScript port of an STFT/FFT pipeline.
- Labeled comparison history is stored locally in the browser via `localStorage` — it stays on your device and isn't sent anywhere.
- No external network calls of any kind once the page is loaded.
