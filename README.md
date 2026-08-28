# Driver Drowsiness Detection System

Real-time webcam-based drowsiness detection using facial landmark tracking
(MediaPipe Face Mesh) and geometric eye/mouth ratios — no model training or
dataset required.

## How it works

1. **Face Mesh** (MediaPipe) detects 468 facial landmarks per frame.
2. **Eye Aspect Ratio (EAR)** is computed from 6 landmarks around each eye.
   EAR stays roughly constant while eyes are open and drops sharply when
   eyes close.
3. **Mouth Aspect Ratio (MAR)** is computed similarly, and rises when the
   mouth opens wide (yawning).
4. If EAR stays below a threshold for `EAR_CONSEC_FRAMES` in a row, a
   **drowsiness alert** triggers (red border + on-screen warning).
5. If MAR stays above a threshold for `MAR_CONSEC_FRAMES` in a row, a
   **yawn alert** triggers.
6. Both ratios are smoothed over a short rolling window to reduce false
   triggers from single-frame jitter or blinking.

## Setup

```bash
pip install -r requirements.txt
```

## Run

```bash
python3 drowsiness_detector.py
```

- A webcam window opens showing your face with eye/mouth landmarks marked,
  live EAR/MAR values, FPS, and status.
- Press `q` to quit.

## Tuning

If it's too sensitive or not sensitive enough for your webcam/lighting,
adjust these constants at the top of `drowsiness_detector.py`:

| Constant | Effect |
|---|---|
| `EAR_THRESHOLD` | Lower = eyes must close more to count as "closed" |
| `EAR_CONSEC_FRAMES` | Higher = eyes must stay closed longer before alert |
| `MAR_THRESHOLD` | Higher = mouth must open wider to count as a yawn |
| `MAR_CONSEC_FRAMES` | Higher = mouth must stay open longer before alert |

Good starting point: run it once, watch the live EAR value in the corner
while blinking normally vs. closing your eyes for a couple seconds, and set
`EAR_THRESHOLD` roughly halfway between those two readings.

## Extending it further (optional, if you have more time)

- **Audio alarm**: play a `.wav` file (e.g. with `playsound` or `simpleaudio`)
  when `drowsy_alert_active` becomes `True`.
- **Head pose / nodding detection**: use landmark 1 (nose tip) and solvePnP
  to estimate head pitch — a drooping head is another drowsiness cue.
- **Logging**: write timestamped alert events to a CSV for a "session report".
- **CNN alternative**: if you want a trained-model component for the
  report, you could train a small CNN (e.g. on the MRL Eye Dataset) to
  classify cropped eye patches as open/clo sed, and use it instead of (or
  alongside) the EAR calculation.