# MolmoAct Workspace

Real-world deployment van Ai2's MolmoAct 2 voor spatial reasoning + robot manipulation.

## 🎯 Project Goal

Live visual feedback robot control via MolmoAct 2:
- Stereo depth input → Waypoint planning → SO-100 execution
- Draw-to-steer interface (tablet sketching)
- Zero-shot pick-and-place met household objects

## 📦 Hardware Stack

- **Vision:** Stereo camera (2560×960, `/dev/videostereo`)
- **Compute:** Spark GB10 (192.168.86.29, 128GB VRAM)
- **Robot:** SO-100 arm (5 DOF) + Reachy Mini (dual-arm optional)
- **Interface:** Laptop + iPad/tablet (future)

## 🚀 Milestones

### Phase 1: Inference Demo (Week 1)
✅ Visual waypoint overlay on live stereo feed  
⏸️ No robot execution yet

### Phase 2: SO-100 Integration (Week 2)
🎯 End-to-end pick-and-place  
⏸️ Basic safety checks

### Phase 3: Tablet Interface (Week 3)
🎨 User-friendly control UI  
⏸️ Draw-to-steer capability

### Phase 4: Bimanual (Future)
🤝 Dual-arm coordination  
⏸️ Advanced tasks (folding, handover)

## 📋 Tickets

See [GitHub Issues](https://github.com/koenvanwijk/molmoact-workspace/issues) (pending repo creation).

Quick links:
- [TICKET-001: Spark Setup + Model Download](#)
- [TICKET-002: Stereo Depth Pipeline](#)
- [TICKET-003: MolmoAct Inference API](#)
- [TICKET-004: Waypoint Visualization](#)
- [TICKET-005: Test Objects Setup](#)

## 🛠️ Tech Stack

**Backend (Spark):**
- MolmoAct 2-7B (13GB checkpoint)
- Depth-Anything-V2 (380MB)
- PyTorch 2.11 + CUDA 13
- Flask REST API (port 8095)

**Frontend (Laptop):**
- Python 3.12 + OpenCV
- Stereo SGBM matching
- Live video overlay (Matplotlib/Qt)

**Future:**
- React + Three.js (tablet UI)
- WebSocket streaming (low-latency video)

## 📊 Status

**Current:** Planning phase  
**Next:** TICKET-001 (Spark model download)  
**Blocked:** None  

## 🔗 References

- [MolmoAct 2 Blog](https://allenai.org/blog/molmoact2)
- [GitHub Repo](https://github.com/allenai/molmoact)
- [HuggingFace Models](https://huggingface.co/collections/allenai/molmoact2-models)
- [Paper (arXiv)](https://arxiv.org/abs/2508.07917)

## 📝 Notes

- Test objects: rode/blauwe blokken, appel, banaan
- Scene setup: tafel + overhead stereo camera
- First task: "pick up red block"
