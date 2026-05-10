# TICKET-002: Stereo Depth Pipeline

**Milestone:** Phase 1 — Inference Demo  
**Priority:** P0 (critical path)  
**Estimate:** 2-3 hours  
**Status:** TODO  
**Depends on:** TICKET-001

## Goal

Build production-grade stereo depth estimation pipeline voor MolmoAct input.

## Background

MolmoAct gebruikt depth-aware perception tokens. Twee opties:
1. **Depth-Anything-V2** (monocular, ~200ms)
2. **SGBM stereo** (binocular, ~50ms, more accurate)

Voor real-time control (180ms target) → **SGBM stereo preferred**.

## Tasks

### 1. Stereo calibration
**Current status:** `/dev/videostereo` produces 2560×960 side-by-side  
**Need:** Camera intrinsics + extrinsics for metric depth

```python
# Checklist:
- [ ] Capture checkerboard pattern (10×7, 25mm squares)
- [ ] Run stereo calibration (OpenCV)
- [ ] Save camera matrices: intrinsics_left.npz, extrinsics.npz
- [ ] Verify reprojection error <0.5 px
```

**Output:** `~/molmoact-workspace/calibration/stereo_params.npz`

### 2. SGBM parameter tuning
**Current params** (from demo):
```python
minDisparity=0
numDisparities=128
blockSize=5
P1=8*3*5**2
P2=32*3*5**2
```

**Optimize for:**
- [ ] Test scene: tafel met blokken/appel/banaan
- [ ] Minimize noise in flat surfaces (tafel)
- [ ] Maximize object boundary sharpness
- [ ] Target: <50ms latency @ 1280×720

**Tuning script:** `scripts/tune_sgbm.py` (grid search P1/P2/blockSize)

### 3. Depth normalization
**Challenge:** SGBM disparity → metric depth requires calibration

```python
# Formula: depth_m = (baseline * focal_length) / disparity_px
baseline = 0.12  # meters (measure between cameras)
focal_length = 640  # pixels (from intrinsics)
```

- [ ] Measure baseline physically (ruler)
- [ ] Extract focal length from calibration
- [ ] Implement disparity_to_depth() function
- [ ] Validate: place object at known distance (0.5m), measure error

**Target accuracy:** ±2 cm @ 0.5m distance

### 4. Integration with MolmoAct
**MolmoAct expects:** Depth map (H×W, float32, meters)  
**SGBM produces:** Disparity map (H×W, int16, pixels)

```python
# Pipeline:
stereo_frame = capture_stereo()
left, right = split_lr(stereo_frame)
disparity = sgbm.compute(left, right)
depth_m = disparity_to_depth(disparity, baseline, focal_length)
depth_normalized = (depth_m - min_depth) / (max_depth - min_depth)  # [0, 1]

# Feed to MolmoAct:
perception_tokens = molmoact.encode_depth(depth_normalized)
```

- [ ] Write `stereo_depth.py` module
- [ ] Test end-to-end: capture → depth → tokens
- [ ] Benchmark latency (target <50ms)

## Deliverables

1. **Calibration data:** `calibration/stereo_params.npz`
2. **Depth module:** `src/stereo_depth.py`
3. **Tuning results:** `results/sgbm_params_optimized.json`
4. **Test video:** `results/depth_pipeline_demo.mp4` (side-by-side RGB + depth)

## Acceptance Criteria

- [ ] Stereo calibration reprojection error <0.5 px
- [ ] Depth accuracy ±2 cm @ 0.5m (validated with ruler)
- [ ] Pipeline latency <50ms (measured with `time.perf_counter()`)
- [ ] Depth map feeds cleanly into MolmoAct (no shape mismatches)

## Test Scene Setup

**Objects to place:**
- 2× rode blokken (10×10×10 cm)
- 2× blauwe blokken (10×10×10 cm)
- 1× appel (~8 cm diameter)
- 1× banaan (~15 cm length)

**Arrangement:**
```
   [Camera overhead]
   ──────────────────────
   │  🔴  🔵     🍎    │ Table
   │     🔴  🔵  🍌    │
   ──────────────────────
```

**Ground truth measurements:**
- Tafel hoogte: 75 cm
- Object distances: measure met laser rangefinder

## Notes

- Stereo camera: `/dev/videostereo` (OAK-D or similar)
- Resolution: 2560×960 → 1280×720 per eye (downscale for speed)
- Frame rate: 30 fps (capture), 20 fps (after SGBM processing)

## Next Ticket

→ TICKET-003: MolmoAct Inference API
