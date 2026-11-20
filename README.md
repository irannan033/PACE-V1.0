# PACE: Precision AI Capacity Evaluation

**AI-powered cycle-time, motion and capacity analysis for garment sewing lines. It runs entirely on a local computer.**

![Status](https://img.shields.io/badge/status-Phase%201%20prototype-orange)
![Processing](https://img.shields.io/badge/processing-100%25%20local-green)
![Hardware](https://img.shields.io/badge/hardware-CPU%20compatible-blue)

PACE replaces stopwatch time studies with computer vision. It reads a video of a sewing operation, detects every operation cycle, tracks the operator's upper-body movement, and compares the operator against a benchmark (reference) video. Results are stored in a local database for process and line analysis.

> **Project status:** Phase 1 (offline, prerecorded video) is a working prototype. Phase 2 (live cameras) is planned. The methods have **not yet been validated on real garment-sewing footage**. See [Limitations](#limitations).

---

## Why PACE

Manual time studies are slow, cover only a few sampled minutes of a shift, and depend on the observer. PACE aims to:

- time **every** cycle in a recording instead of a sample,
- compare an operator with a benchmark method fairly, even when they work faster or slower,
- show *what the AI saw* (skeleton, wrist trails, cycle markers, confidence),
- keep all video on the user's own machine.

## Key features

| Feature | Description |
|---|---|
| Cycle detection | A cycle is a START-region event followed by an END-region event. Both regions are drawn by the user; a single line is not assumed to define a cycle. |
| Pose and skeleton | Shoulders, elbows, wrists and hands tracked with MediaPipe. The skeleton, wrist trails and a confidence value are drawn on the video. |
| Reference comparison | Dynamic Time Warping (DTW) compares movement shape independently of speed. |
| Scoring on a 1.0 baseline | Separate cycle-time, motion-similarity and overall scores; 1.0 equals the benchmark. |
| Production metrics | Cycle statistics, productive and idle time, cycles per hour, estimated SAM, theoretical capacity. |
| Side-by-side player | Synchronized reference and test videos with overlays. |
| Local history | SQLite storage, a trend chart and CSV export. |
| Privacy | No cloud service, no upload, no internet needed to analyse video. |

## How it works

<p align="center">
<img src="images/Motion Analysis Workflow Infographic.png" alt="PACE Project Workflow" width="400">
<img src="images/Workflow.jpg" alt="PACE Project Workflow" width="400">   
</p>

### Methodology in brief

1. **Motion signal.** Each frame is converted to grayscale and blurred. The difference from the previous frame, averaged inside the START region, inside the END region and over the whole frame, gives three signals.
2. **Event detection.** Signals are smoothed. An event is the moment a signal rises above an adaptive threshold, with a short refractory window to prevent double counting.
3. **Cycle pairing.** A cycle is a START event followed by an END event before the next START. Very short cycles are rejected.
4. **Pose estimation.** MediaPipe Pose provides 33 landmarks per frame, each with a confidence value. Low-confidence joints are not drawn.
5. **Movement comparison.** Per-cycle motion profiles and wrist-path shapes are resampled and compared with DTW. The distance is mapped to a 0 to 1 similarity.
6. **Scoring.**
   - Cycle-time ratio = reference average cycle time / test average cycle time
   - Overall score = 0.5 x min(ratio, 1.5) + 0.3 x similarity + 0.2 x completeness
   - The weights are design assumptions, not validated values.

Observed measurements (cycle count, cycle time) are kept separate from estimates (SAM) and model outputs (pose confidence, similarity).

## Architecture

<p align="center">
<img src="images/Desktop Analysis System Flowchart.png" alt="PACE Project Workflow" width="500">
</p>

The same analysis engine is designed to serve both phases: Phase 1 feeds it frames from video files, and Phase 2 will feed it frames from live cameras.

## Tech stack

Python, OpenCV, NumPy, MediaPipe (optional, for pose), PySide6 (desktop GUI), pyqtgraph, SQLite, pytest.

> MediaPipe's pose module requires an older Python version (3.9 to 3.12). Use a virtual environment with a supported version if pose is needed.

## Current status

| Capability | Status |
|---|---|
| Data entry, validation, video loading | Done |
| Side-by-side playback with overlays | Done |
| ROI-based cycle detection | Done (thresholds need tuning on real footage) |
| Skeleton, wrist trails, confidence | Done (requires MediaPipe) |
| DTW similarity and 1.0-baseline scoring | Done |
| Metrics, SAM estimate, SQLite history, CSV export | Done |
| Efficiency percentage | Partial (needs a reference SAM input) |
| Line bottleneck analysis | Partial (calculation only, no dashboard screen) |
| Excess or missing movement detection | Planned |
| Cycle-boundary correction and model training | Planned |
| Live camera monitoring | Planned (Phase 2) |

## Roadmap

| Phase | Focus | Planned work |
|---|---|---|
| **1A** | Prototype (current) | Local desktop app, ROI cycles, pose, DTW, scoring, SQLite |
| **1B** | Validation | Annotated videos, precision / recall / F1, boundary and cycle-time error, stopwatch comparison, threshold calibration, correction screen |
| **2A** | Live pilot | Camera capture thread, rolling-window detection, one station, live dashboard |
| **2B** | Line scale | Multiple stations and lines, work-in-progress-aware bottlenecks, models trained from corrected data |

## Limitations

- **Not validated on garment sewing.** The underlying methods are general-purpose, and no accuracy claim is made until evaluated on annotated sewing video.
- **Simple cycle detection.** It assumes a fixed camera and visible regions. Lighting changes, passers-by or vibration can cause false events.
- **Crude idle rule.** Productive time is based on overall motion and cannot tell value-adding work from other movement.
- **Assumed values.** SAM rating and allowance, score weights and DTW scales are assumptions awaiting calibration. SAM is always an estimate, not a standard time.
- **No quality or safety judgement.** A faster cycle is not automatically better. Human review is required.
- **No learning yet.** Corrections can be stored, but no model is trained or updated. PACE does not learn autonomously.
- **Pose needs a visible upper body**, and playback syncs by frame index, so videos with different frame rates can drift visually (measured times stay correct).

## Future scope

- Human-in-the-loop correction of cycle boundaries and threshold calibration
- Detection of excess, missing and repeated movements using sequence alignment and action segmentation
- Wrist trajectories and optical flow as additional comparison features
- Line dashboard with parallelism- and WIP-aware bottleneck analysis
- Reusable reference-video library per buyer, style and process
- Multi-camera, multi-station real-time monitoring
- Reinforcement learning only if a real reward function and decision problem can be defined

## Privacy

All video processing, inference and storage happen on the local computer. PACE does not send video or measurements to any cloud service. If you use PACE on real production video, handle recordings and the database according to your organisation's data and worker-privacy policies.

## References

These sources support the methods used. None of them tested garment sewing. Please verify details before citing.

- Sakoe, H. and Chiba, S. (1978). Dynamic programming algorithm optimization for spoken word recognition. *IEEE Transactions on Acoustics, Speech, and Signal Processing*. DOI: 10.1109/TASSP.1978.1163055
- Cao, Z., Hidalgo, G., Simon, T., Wei, S.-E. and Sheikh, Y. (2019). OpenPose: Realtime multi-person 2D pose estimation using Part Affinity Fields. *IEEE TPAMI*. DOI: 10.1109/TPAMI.2019.2929257
- Bazarevsky, V. et al. (2020). BlazePose: On-device real-time body pose tracking. arXiv:2006.10204
- Lugaresi, C. et al. (2019). MediaPipe: A framework for building perception pipelines. arXiv:1906.08172

## ⚠️ Disclaimer

This repository contains a sanitized representation of a professional industrial automation project.
Proprietary company information, production data, network configurations and confidential materials have been excluded.

---

<p align="center">
  © 2026 Itmam Rubayet Annan — All Rights Reserved  
</p>

