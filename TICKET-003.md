# TICKET-003: MolmoAct Inference API

**Milestone:** Phase 1 — Inference Demo  
**Priority:** P0 (critical path)  
**Estimate:** 3-4 hours  
**Status:** TODO  
**Depends on:** TICKET-001, TICKET-002

## Goal

Wrap MolmoAct 2 inference in Flask REST API op Spark (port 8095).

## API Spec

### Endpoint: POST /predict

**Request:**
```json
{
  "image": "base64_encoded_jpg",
  "depth": "base64_encoded_depth_map",  // optional, computed if missing
  "instruction": "pick up the red block",
  "depth_mode": "stereo",  // or "depth_anything" or "precomputed"
  "return_visualization": true
}
```

**Response (200 OK):**
```json
{
  "waypoints": [
    {"x": 640, "y": 200, "label": "approach"},
    {"x": 580, "y": 350, "label": "pre_grasp"},
    {"x": 580, "y": 420, "label": "grasp"}
  ],
  "perception_tokens": [[3, 7, 11, ...], ...],  // 15×20 grid
  "actions": [
    {"joints": [0.1, 0.2, 0.3, 0.4, 0.5], "gripper": "open"},
    {"joints": [0.15, 0.25, 0.35, 0.45, 0.55], "gripper": "close"}
  ],
  "visualization": "base64_encoded_overlay_image",  // RGB + waypoints drawn
  "latency_ms": 187,
  "model": "MolmoAct-7B-D-0812"
}
```

**Error (500):**
```json
{
  "error": "CUDA out of memory",
  "traceback": "...",
  "request_id": "abc123"
}
```

## Implementation

### File structure
```
molmoact-workspace/
├── server/
│   ├── app.py              # Flask app
│   ├── inference.py        # MolmoAct wrapper
│   ├── depth_processor.py  # Depth-Anything / SGBM
│   ├── visualizer.py       # Draw waypoints on image
│   └── config.py           # Model paths, port
├── requirements.txt
└── start_server.sh
```

### app.py skeleton
```python
from flask import Flask, request, jsonify
import base64
import time
from inference import MolmoActInference

app = Flask(__name__)
model = MolmoActInference(
    checkpoint_path="~/models/MolmoAct2/MolmoAct-7B-D-0812",
    device="cuda:0"
)

@app.route('/predict', methods=['POST'])
def predict():
    t0 = time.perf_counter()
    
    data = request.json
    image_b64 = data['image']
    instruction = data['instruction']
    
    # Decode image
    image = decode_base64_image(image_b64)
    
    # Get depth (stereo or Depth-Anything)
    if data.get('depth_mode') == 'stereo':
        depth = stereo_depth_from_image(image)
    else:
        depth = depth_anything_v2(image)
    
    # Run MolmoAct
    result = model.predict(image, depth, instruction)
    
    # Optionally draw waypoints
    if data.get('return_visualization'):
        vis_image = draw_waypoints(image, result['waypoints'])
        result['visualization'] = encode_base64_image(vis_image)
    
    latency = (time.perf_counter() - t0) * 1000
    result['latency_ms'] = latency
    
    return jsonify(result)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8095)
```

## Tasks

### 1. Core inference wrapper
- [ ] `inference.py`: Load MolmoAct checkpoint
- [ ] Implement `predict(image, depth, instruction)` method
- [ ] Handle CUDA memory (allocate/free per request)
- [ ] Return waypoints + perception tokens + actions
- [ ] Add error handling (OOM, invalid input)

### 2. Depth processing
- [ ] `depth_processor.py`: SGBM stereo pipeline
- [ ] Integrate Depth-Anything-V2 (fallback)
- [ ] Auto-detect mode (stereo if 2560×960, mono otherwise)
- [ ] Normalize depth to [0, 1] range

### 3. Visualization
- [ ] `visualizer.py`: Draw waypoints as circles + arrows
- [ ] Color code by sequence (W1=yellow, W2=orange, W3=red)
- [ ] Add instruction text overlay
- [ ] Return as PNG (base64 encoded)

### 4. Flask app
- [ ] Basic auth (optional, Bearer token)
- [ ] CORS headers (allow laptop client)
- [ ] Request logging (timestamp, latency, instruction)
- [ ] Health check endpoint: `GET /health`

### 5. Deployment
- [ ] `start_server.sh`: Activate venv, export CUDA vars, run Flask
- [ ] Systemd service (optional): `molmoact-api.service`
- [ ] Test from laptop: `curl -X POST http://192.168.86.29:8095/predict`

## Acceptance Criteria

- [ ] Server starts without errors
- [ ] `/health` returns 200 OK
- [ ] POST /predict with test image → valid waypoints in <3 sec
- [ ] Visualization shows waypoints correctly overlaid
- [ ] CUDA memory freed after each request (no leak)
- [ ] Latency logged to console: "Inference: 187ms"

## Testing

**Test script (laptop):**
```python
import requests
import base64

# Capture stereo frame
img = open('stereo_test.jpg', 'rb').read()
img_b64 = base64.b64encode(img).decode()

# Send request
response = requests.post('http://192.168.86.29:8095/predict', json={
    'image': img_b64,
    'instruction': 'pick up the red block',
    'depth_mode': 'stereo',
    'return_visualization': True
})

result = response.json()
print(f"Waypoints: {result['waypoints']}")
print(f"Latency: {result['latency_ms']} ms")

# Save visualization
vis_img = base64.b64decode(result['visualization'])
open('waypoints_overlay.png', 'wb').write(vis_img)
```

## Performance Target

- **Cold start:** <5 sec (first request, model load)
- **Warm inference:** <300 ms (subsequent requests)
- **Memory:** <30 GB VRAM per request
- **Throughput:** 3-5 requests/sec (single GPU)

## Notes

- Flask dev server OK for testing, use gunicorn for production
- GPU lock: only 1 request at a time (no batching yet)
- Error handling: catch OOM, return 503 Service Unavailable

## Next Ticket

→ TICKET-004: Waypoint Visualization (Live Feed)
