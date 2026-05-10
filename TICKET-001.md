# TICKET-001: Spark Setup + MolmoAct 2 Model Download

**Milestone:** Phase 1 — Inference Demo  
**Priority:** P0 (blocker)  
**Estimate:** 4-6 hours  
**Status:** TODO

## Goal

Installeer MolmoAct 2-7B op Spark en test basic inference.

## Tasks

### 1. Complete molmoact dependencies
- [x] PyTorch 2.11 installed (`venv_molmo`)
- [ ] Build decord from source (ARM64 compatibility)
- [ ] Install remaining deps: `pip install -e .[all]`
- [ ] Verify install: `python -c "import olmo; print('OK')"`

**Blocker:** decord build in progress (~/decord/, needs cmake + ffmpeg-dev)

### 2. Download checkpoints
- [ ] Depth-Anything-V2: `depth_anything_v2_vitl.pth` (380 MB)
- [ ] MolmoAct VQVAE: `vae-final.pt` (size TBD)
- [ ] MolmoAct 2-7B: `allenai/MolmoAct-7B-D-0812` (~13 GB)

**Paths:**
- Models: `~/models/MolmoAct2/`
- Checkpoints: `~/molmoact/checkpoints/`

### 3. Test inference (single frame)
```bash
cd ~/molmoact
source venv_molmo/bin/activate
python scripts/generate.py \
  --checkpoint ~/models/MolmoAct2/MolmoAct-7B-D-0812 \
  --image /tmp/test_frame.jpg \
  --instruction "point to the red block" \
  --output /tmp/molmoact_output.json
```

**Expected output:**
```json
{
  "waypoints": [[640, 200], [580, 350]],
  "perception_tokens": [...],
  "actions": [{"joints": [...], "gripper": "open"}]
}
```

### 4. GPU memory check
- MolmoAct 2-7B: ~28 GB VRAM (Q4 quant, FP16)
- Current Spark usage: 56 GB (2× llama-servers)
- Available: 72 GB
- **Decision:** Temporarily stop 1 llama-server during inference tests

## Acceptance Criteria

- [ ] `import olmo` works without errors
- [ ] All 3 checkpoints downloaded and verified (checksums)
- [ ] Single-frame inference completes in <5 sec
- [ ] Output JSON contains valid waypoints (2D pixel coords)

## Blockers

- **decord build:** Requires cmake + ffmpeg-dev headers (apt install)
- **GPU memory:** May need to stop llama-server temporarily

## Notes

- MolmoAct 2 repo cloned: `~/molmoact/` (1233 files)
- venv ready: `~/molmoact/venv_molmo/`
- Current Python: 3.12.3 (externally-managed, venv required)

## Next Ticket

→ TICKET-002: Stereo Depth Pipeline

## Progress Update (21:48 UTC+2)

### ✅ Completed
- [x] PyTorch 2.11 installed
- [x] Skip decord (ARM64 build issue, use imageio/opencv instead)
- [x] MolmoAct package installed (`pip install -e .`)
- [x] Depth-Anything-V2 downloaded (1.3 GB, ~/models/MolmoAct2/checkpoints/)
- [x] Test import: `import olmo` works

### 🔄 In Progress
- [ ] MolmoAct-7B-D-0812 download (6% done, ETA ~10 min)
  - 7 safetensors shards (~1.8 GB each)
  - PID: 1765586
  - Log: ~/models/MolmoAct2/download.log

### 📸 Test Scene
- [x] Initial stereo capture: test_scene_initial.jpg (225 KB)
- Objects present: Colored blocks (Koen placed)
- Link: http://192.168.86.25:7788/test_scene_initial.jpg

### Next
- Wait for checkpoint download
- Test single-frame inference
- Move to TICKET-002 (stereo depth pipeline)
