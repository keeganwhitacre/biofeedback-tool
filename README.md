# KW Biofeedback

A mobile-first PWA for HRV biofeedback training using camera-based PPG (photoplethysmography). No external hardware required — the phone's rear camera and torch do the sensing.

## How it works

The red channel of camera frames is extracted and fed through an IIR bandpass filter (0.67–3.33 Hz, ~40–200 BPM) before beat detection. Beat onsets are found via the **WABP algorithm** (Zong et al. 2003), originally validated on invasive arterial waveforms and adapted here for camera PPG. Dicrotic notch artifacts are rejected with a median-anchored 60% IBI gate. All timestamps use `performance.now()` (monotonic) rather than wall-clock time.

## Signal pipeline

```
Camera red channel → IIR bandpass → WABP onset → dicrotic gate → IBI stream
```

HRV metrics computed in real-time:
- **RMSSD** — with 20% artifact rejection on successive differences
- **SDNN**
- **Poincaré SD1/SD2** — SD1 ≈ RMSSD/√2; SD2 captures longer-term variance
- **Coherence** — beat-phase alignment with the breathing pacer (RSA proxy)

## Features

- **Breathing pacer** — Resonant frequency, Box (4-4-4-4), 4-7-8, and Coherence (5.5 BPM) protocols
- **RF Finder** — Lehrer et al. protocol: steps 6.5→4.5 BPM in 0.5 increments, 2 min each, 30 s washout; winner = largest 90th–10th percentile IBI amplitude
- **2-min baseline** — resting HRV snapshot without the pacer
- **Live charts** — scrolling IBI tachogram + Poincaré plot
- **Session history** with RMSSD trend sparkline, CSV export

## Files

| File | Purpose |
|---|---|
| `epat-core.js` | Validated PPG + audio + motion signal stack (reusable, no UI coupling) |
| `index.html` | Application shell + all UI logic |
| `sw.js` | Service worker for offline/PWA support |
| `manifest.json` | PWA metadata |

## Notes

- Requires a rear camera with torch (flash LED). Front cameras are explicitly excluded.
- iOS may renegotiate frame rate after torch is enabled; `epat-core` re-reads the track settings post-torch to keep WABP's sample-count windows accurate.
- Signal quality is estimated via perfusion index (AC/DC ratio of the raw red channel), updated at 1 Hz.
- Hosted at `biofeedback.keeganwhitacre.com`
