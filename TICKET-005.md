# TICKET-005: Test Objects Setup + Scene Configuration

**Milestone:** Phase 1 — Inference Demo  
**Priority:** P2 (nice-to-have, maar makkelijk)  
**Estimate:** 1 hour  
**Status:** TODO  
**Depends on:** None (can start immediately)

## Goal

Standaard test scene voor reproduceerbare MolmoAct evaluatie.

## Test Objects

### Aanwezig (Koen brengt aan):
- ✅ 2× rode blokken (10×10×10 cm)
- ✅ 2× blauwe blokken (10×10×10 cm)
- ✅ 1× appel (~8 cm diameter, rood)
- ✅ 1× banaan (~15 cm length, geel)

### Optional additions (later):
- [ ] Groene blok (kleur variatie)
- [ ] Oranje (contrast test)
- [ ] Kleine objecten (pen, USB stick)
- [ ] Transparant glas (depth challenge)

## Scene Layout

### Standard arrangement:
```
          [Stereo Camera Overhead]
                    ↓
    ┌─────────────────────────────────┐
    │                                 │
    │   🔴₁  🔵₁        🍎            │  Y
    │                                 │  ↑
    │        🔴₂  🔵₂       🍌        │  │
    │                                 │  │
    └─────────────────────────────────┘  └──→ X
           Tafel (75 cm hoog)
```

**Coordinates (image space, left camera):**
- Red block 1: ~(400, 300)
- Blue block 1: ~(600, 300)
- Apple: ~(900, 280)
- Red block 2: ~(500, 450)
- Blue block 2: ~(700, 450)
- Banana: ~(950, 470)

**Physical distances (from camera):**
- Camera height: ~80 cm above table
- Object distances: 50-70 cm from camera
- Inter-object spacing: ≥15 cm (avoid occlusion)

## Camera Setup

### Position:
- **Height:** 80 cm above table surface
- **Angle:** 15° tilt (slight top-down view)
- **FOV coverage:** 60×40 cm workspace area

### Lighting:
- **Overhead LED:** 5000K daylight (avoid shadows)
- **Avoid:** Direct sunlight (washes out colors)
- **Fallback:** Ring light around camera (diffuse)

### Calibration markers:
- [ ] Place ArUco markers at table corners (optional)
- [ ] Measure exact camera→table distance (laser rangefinder)
- [ ] Record baseline (stereo camera separation): ~12 cm

## Scene Variations

### Test cases:
1. **Sparse:** 4 objects (1 red, 1 blue, 1 apple, 1 banana)
2. **Dense:** 6 objects (all items, close spacing)
3. **Occluded:** Stack blue block on red block
4. **Mixed heights:** Book + block tower
5. **Extreme distance:** Object at edge of FOV

### Per test case:
- [ ] Capture reference photo (no waypoints)
- [ ] Record ground truth coords (manual annotation)
- [ ] Save as `test_scenes/case_N_ref.jpg`

## Documentation

### Scene metadata: `test_scenes/metadata.json`
```json
{
  "camera": {
    "model": "OAK-D",
    "resolution": "2560x960",
    "baseline_cm": 12.0,
    "height_cm": 80.0,
    "tilt_deg": 15.0
  },
  "table": {
    "width_cm": 120,
    "depth_cm": 80,
    "height_cm": 75,
    "surface_color": "white"
  },
  "objects": [
    {
      "id": "red_block_1",
      "type": "block",
      "color": "red",
      "size_cm": [10, 10, 10],
      "position_image_px": [400, 300],
      "distance_cm": 55
    },
    ...
  ],
  "test_cases": [
    {
      "name": "sparse",
      "objects": ["red_block_1", "blue_block_1", "apple", "banana"],
      "reference_image": "case_1_ref.jpg"
    },
    ...
  ]
}
```

## Tasks

### 1. Physical setup
- [x] Koen legt objecten neer
- [ ] Position stereo camera (overhead mount)
- [ ] Adjust lighting (no harsh shadows)
- [ ] Verify FOV coverage (all objects visible)

### 2. Capture reference images
- [ ] Case 1: Sparse (4 objects)
- [ ] Case 2: Dense (6 objects)
- [ ] Case 3: Occluded (stacked blocks)
- [ ] Case 4: Mixed heights
- [ ] Save to `test_scenes/`

### 3. Document scene
- [ ] Measure camera height (laser rangefinder)
- [ ] Measure object positions (ruler from camera center)
- [ ] Record in `metadata.json`
- [ ] Annotate ground truth waypoints (manual)

### 4. Validation script
```python
# test_scenes/validate_scene.py
def validate_scene(image_path):
    """Check if all expected objects are visible."""
    img = cv2.imread(image_path)
    detections = detect_objects(img)  # Simple color thresholds
    
    assert len(detections) == expected_count
    for obj in expected_objects:
        assert obj in detections, f"Missing: {obj}"
    
    print("✅ Scene valid")
```

- [ ] Run on all test cases
- [ ] Verify all objects detected
- [ ] Check depth range (min 40 cm, max 80 cm)

## Deliverables

1. **Test images:** `test_scenes/case_1_ref.jpg` ... `case_5_ref.jpg`
2. **Metadata:** `test_scenes/metadata.json`
3. **Validation script:** `test_scenes/validate_scene.py`
4. **Photo documentation:** `test_scenes/setup_photo.jpg` (wide shot of full setup)

## Acceptance Criteria

- [ ] All 6 test objects visible in camera FOV
- [ ] No occlusion in "sparse" case
- [ ] Lighting uniform (no dark corners)
- [ ] Reference images captured @ 1280×720
- [ ] Ground truth distances measured (±1 cm accuracy)

## Instructions for Koen

**Quick setup (5 min):**
1. Leg rode en blauwe blokken in patroon zoals hierboven
2. Appel en banaan ertussen (niet te dicht)
3. Zorg dat alles binnen de ~60×40 cm tafeloppervlak blijft
4. Als je klaar bent: ping me, dan maak ik reference photos

**Klaar?** → Run viewer en kijk of waypoints kloppen!

## Notes

- Table surface color matters: wit/licht = beter contrast
- Avoid glossy objects (specularity confuses depth)
- Keep consistent spacing (±15 cm) for reproducibility

## Next Ticket

→ TICKET-006: Benchmark Suite (accuracy metrics)
