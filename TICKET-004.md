# TICKET-004: Live Waypoint Visualization

**Milestone:** Phase 1 — Inference Demo  
**Priority:** P1 (user experience)  
**Estimate:** 2-3 hours  
**Status:** TODO  
**Depends on:** TICKET-003

## Goal

Real-time video feed met MolmoAct waypoints overlay op laptop.

## UI Design

```
┌────────────────────────────────────────────────┐
│  [MolmoAct Live Feed]              [●] 21 FPS │
├────────────────────────────────────────────────┤
│                                                │
│        ╔════════════════════════╗             │
│        ║   🟡 W1 ──→ 🟠 W2      ║  Camera    │
│        ║         ↓               ║  View      │
│        ║        🔴 W3            ║            │
│        ║                         ║            │
│        ║   🔴 🔵  🍎  🍌         ║            │
│        ╚════════════════════════╝             │
│                                                │
│  Instruction: "pick up red block"             │
│  Latency: 187 ms  |  GPU: 28/128 GB           │
│                                                │
│  [Capture]  [Clear]  [Save Frame]             │
└────────────────────────────────────────────────┘
```

## Implementation

### Tech stack
**Option A: OpenCV + Qt**
```python
import cv2
import requests
import base64
from PyQt5.QtWidgets import QMainWindow, QLabel

class MolmoActViewer(QMainWindow):
    def __init__(self):
        self.cap = cv2.VideoCapture('/dev/videostereo')
        self.timer = QTimer()
        self.timer.timeout.connect(self.update_frame)
        self.timer.start(50)  # 20 FPS
    
    def update_frame(self):
        ret, frame = self.cap.read()
        
        # Send to API every N frames (throttle)
        if self.frame_count % 5 == 0:
            waypoints = self.get_waypoints(frame)
            self.draw_waypoints(frame, waypoints)
        
        self.display(frame)
```

**Option B: Matplotlib Animation**
```python
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation

fig, ax = plt.subplots()
im = ax.imshow(np.zeros((720, 1280, 3)))

def update(frame_num):
    frame = capture_stereo()
    waypoints = get_waypoints(frame)
    overlay = draw_waypoints(frame, waypoints)
    im.set_data(overlay)
    return [im]

ani = FuncAnimation(fig, update, interval=50, blit=True)
plt.show()
```

**Recommendation:** Option A (Qt) for better performance + UI controls.

## Tasks

### 1. Video capture loop
- [ ] Read from `/dev/videostereo` @ 30 FPS
- [ ] Split to left view (1280×720)
- [ ] Display in Qt window
- [ ] Show FPS counter (top-right)

### 2. API client
- [ ] Throttle requests: 1 inference per 5 frames (6 FPS inference)
- [ ] Async HTTP client (don't block video loop)
- [ ] Cache last waypoints (display until new inference arrives)
- [ ] Handle API errors (show "Inference failed" overlay)

### 3. Waypoint overlay
- [ ] Draw circles at waypoint coords (radius=20px)
- [ ] Draw arrows between waypoints (thick, colored)
- [ ] Color scheme: W1=yellow (#FFD700), W2=orange (#FF8C00), W3=red (#FF0000)
- [ ] Add labels: "W1", "W2", "W3" above circles
- [ ] Fade-out animation (optional): old waypoints disappear gradually

### 4. Status HUD
- [ ] Bottom bar: instruction text
- [ ] Latency display: "187 ms" (color-coded: <200=green, 200-500=orange, >500=red)
- [ ] GPU usage: query via `/health` endpoint
- [ ] Connection status: green dot if API responsive

### 5. Controls
**Buttons:**
- **Capture:** Freeze frame + save to disk
- **Clear:** Remove waypoint overlays
- **Save Frame:** Export current view as PNG

**Keyboard shortcuts:**
- `Space`: Toggle inference (pause/resume)
- `C`: Capture frame
- `Q`: Quit

## Deliverables

1. **Viewer app:** `client/molmoact_viewer.py`
2. **Demo video:** `results/live_demo.mp4` (30 sec, waypoints on real objects)
3. **Screenshot:** `results/viewer_screenshot.png`

## Acceptance Criteria

- [ ] Video displays at ≥20 FPS (smooth, no lag)
- [ ] Waypoints update every ~150-200 ms (5 FPS inference)
- [ ] Overlay is accurate (waypoints align with objects)
- [ ] UI is responsive (buttons work, no freeze)
- [ ] Latency <300 ms end-to-end (capture → inference → display)

## Test Scenario

**Setup:**
1. Place rode blok, blauwe blok, appel, banaan op tafel
2. Run viewer: `python client/molmoact_viewer.py`
3. Instruction: "point to the red block"

**Expected behavior:**
- Camera shows live feed
- After ~200 ms, yellow circle appears (W1: approach point)
- Arrow draws to orange circle (W2: above object)
- Second arrow to red circle (W3: on red block)
- Waypoints stay visible for 2-3 sec, then refresh

**Demo script:**
```bash
# Terminal 1 (Spark):
cd ~/molmoact
./start_server.sh

# Terminal 2 (Laptop):
cd ~/molmoact-workspace/client
python molmoact_viewer.py --instruction "pick up red block"
```

## Performance Optimization

### Bottlenecks:
1. **API latency:** 180-300 ms (inference)
2. **Network:** 10-20 ms (laptop ↔ Spark)
3. **Video capture:** 33 ms (30 FPS)

### Optimizations:
- [ ] Use HTTP/1.1 keep-alive (avoid TCP handshake per request)
- [ ] Compress image before sending (JPEG quality=85)
- [ ] WebSocket upgrade (future): streaming video + real-time waypoints

### Target:
- **End-to-end latency:** <300 ms (capture → display)
- **UI FPS:** ≥20 (smooth video)
- **Inference rate:** 5-6 FPS (acceptable for manipulation tasks)

## Notes

- Stereo camera resolution: 2560×960 → downsample to 1280×720 for speed
- Qt5 preferred over Tkinter (better video performance)
- Consider adding audio feedback: beep when waypoints update

## Next Ticket

→ TICKET-005: Test Objects Setup + Scene Configuration
