# Baby Sleep · Audio and scheduling logic

## 1. Preparing colored noise

The offline generator creates white noise, transforms it with a real FFT, scales frequency amplitudes by `1 / f^(exponent/2)`, then transforms it back. Exponents of 0, 1 and 2 give the intended white, pink and brown power-spectrum shapes. The zero-frequency bin is handled explicitly to avoid division by zero.

Low-pass and high-pass response curves shape those signals further. Rhythmic envelopes, low-frequency modulation and synthesized note partials build the other sound textures. This uses established DSP operations assembled into the project’s audio catalog.

## 2. Preparing loop boundaries

The generator blends the beginning with an extra tail section using square-root crossfade weights. Low-frequency oscillators are snapped to an integer number of cycles over the loop length. Together these choices reduce boundary discontinuities in the generated files.

Peak normalization happens before writing mono PCM WAV data. These are asset-generation steps, separate from AVAudioPlayer’s runtime looping.

## 3. Independent layers with shared master volume

Each active layer has its own looping player. Effective volume is `layerVolume × masterVolume`. During a timed fade it becomes `layerVolume × startingMasterVolume × fadeFactor`, where the fade factor moves from 1 toward 0.

The manager prevents ordinary volume reapplication from overwriting the active timer fade. Live mix editing changes the relevant layer rather than rebuilding every sound from the UI.

## 4. Sleep timer based on a deadline

The sleep timer stores an end date. Displayed remaining time is recomputed as `max(0, endDate - now)` rather than decremented by one on every callback. Fade-out begins when remaining time reaches the selected fade interval.

This accounts for delayed timer callbacks. The routine-step timer is a separate implementation that decrements its remaining value on ticks; the deadline behavior should not be attributed to every timer in the app. Pausing playback stops the sleep-timer callback, but the stored deadline continues to advance with wall-clock time.

## 5. Preventing a routine from restarting after the sleep timer

Routine steps can start sounds/mixes, wait, show reminders or drive a fade. A global sleep-timer expiry has a dedicated cancellation path that clears routine scheduling without advancing to another sound-producing step.

The distinction matters when a routine and a global stop time overlap: stopping audio is not enough if a scheduled next step could start it again.

## 6. Session-based sleep summaries

The log manager computes duration from start/end dates and averages completed sessions whose start is within the last seven days. The result is average session duration, not total sleep divided by seven days. An unfinished session is excluded from that completed-session average.

[Architecture](ENGINEERING.md) · [Local data layout](DATA_MODEL.md)
