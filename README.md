# Rotating Lidar Proximity Sensor — Interceptor Simulation

Interactive 3D simulation of a 1D lidar proximity sensor mounted on a rotating interceptor body, used to evaluate detection performance against an incoming target in various engagement geometries.

Built with Three.js r128 (3D rendering) and Chart.js 4.4.4 (detection timeline). Single self-contained HTML file — no build step required.

---

## What This Simulates

A tube-launched interceptor flies toward a target. A 1D lidar sensor is mounted on the interceptor body at a fixed angle from the flight axis. The sensor fires pulses at a configurable rate and detects if the pulse ray intersects the target box surface (ray–AABB intersection — physically correct, no sampling gaps).

In **Stage 2**, the sensor rotates around the flight axis at a set RPM, sweeping a cone-shaped detection zone (annular torus). This increases coverage at the cost of mechanical complexity.

Structural rods run along the interceptor body and can occlude the sensor's line of sight, creating blind sectors. The simulation checks rod occlusion geometrically (ray–cylinder intersection) for every pulse.

---

## Layout

```
┌─────────────────────────────────┬──────────────────────┐
│                                 │                      │
│         3D Viewport             │      Sidebar         │
│      (Three.js canvas)          │   (parameters +      │
│                                 │    metrics)          │
│                                 │                      │
├─────────────────────────────────┴──────────────────────┤
│              Detection Chart (bottom panel)            │
└────────────────────────────────────────────────────────┘
```

- Sidebar is resizable by dragging the vertical divider
- Bottom panel: detection-over-time chart (always visible)
- 3D viewport fills remaining space; Three.js renderer tracks its size correctly

---

## Sensor Model

- Physical dimensions: 4 × 2 × 6 cm box, rendered as a grey cube with a blue lens face
- Mounted on the interceptor at **Mount angle X** degrees from the flight axis
- Detection is a **single ray cast** along the sensor boresight — the boresight intersects the target box surface using the slab method (ray vs AABB). Any surface hit within range counts.
- The **FOV cone** (visual only) represents beam divergence; it is displayed as a transparent cone around the boresight. Rod holes in the cone are computed geometrically per frame.
- Pulses fire at the configured **Sample rate** (Hz), accumulated via a clock so the rate is exact regardless of frame rate

---

## Simulation Stages

### Stage 1 — Fixed sensor
Sensor is fixed at mount angle X from the flight axis and azimuth φ₀. The FOV cone illuminates a fixed sector. Use this to evaluate single-position coverage and sensitivity to engagement geometry.

### Stage 2 — Rotating sensor
Sensor rotates at **Spin rate** RPM around the flight axis. The swept detection zone is a torus (annular cone) from `X − FOV/2` to `X + FOV/2` degrees from the axis. A translucent swept volume mesh is shown. Use this to evaluate full 360° coverage performance.

---

## Parameters

### Sensor
| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| Mount angle X | 10–80° | 13° | Angle of sensor axis from flight axis |
| Sensor FOV (D) | 0.3–30° | 1.8° | Full cone angle (half-angle = D/2) |
| Max range | 5–200 m | 35 m | Maximum detection range |

### Rotation (Stage 2)
| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| Spin rate | 100–6000 RPM | 1500 RPM | Sensor rotation speed |
| Sample rate | 10–10000 Hz | 300 Hz | Pulse fire rate |
| Mount offset φ | 0–360° | 0° | Azimuth angle at t=0 |

### Structural Rods
| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| Rod count | 3 or 4 | 4 | Number of longitudinal structural rods |
| Rod diameter | 5–40 mm | 12 mm | Rod cross-section diameter |
| Rod mount radius | 5–20 cm | 10 cm | Radial distance of rods from flight axis (= body radius by default) |

### Interceptor
| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| Speed | 50–2000 m/s | 300 m/s | Interceptor closing speed |
| Body radius | 5–25 cm | 10 cm | Outer radius of interceptor tube |
| Pitch rate | ±30 °/s | 0 | Body pitch rotation rate |
| Yaw rate | ±30 °/s | 0 | Body yaw rotation rate |
| Roll rate | ±180 °/s | 0 | Body roll rotation rate |

### Target
| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| Width (Y) | 0.2–20 m | 5 m | Target cross-section width |
| Height (Z) | 0.2–20 m | 3 m | Target cross-section height |
| Depth (X) | 0.2–30 m | 8 m | Target length along flight axis |
| Speed | 0–1500 m/s | 200 m/s | Target speed |
| Initial range | 10–150 m | 60 m | Starting range at t=0 |

### Engagement Scenario
6 preset geometries:
- **Head-On** — target closing directly toward interceptor
- **Tail Chase** — interceptor overtaking target from behind
- **Crossing — Horizontal** — target crosses flight path laterally
- **Crossing — Vertical** — target crosses flight path vertically
- **Crossing — 45° Diagonal** — diagonal crossing
- **Oblique — 30° off-axis** — angled approach

Lateral offsets Y and Z shift the target's pass point relative to the interceptor.

### Control
| Parameter | Range | Default | Description |
|-----------|-------|---------|-------------|
| Chart window | 0.1–10 s | 0.4 s | Time span shown in detection chart |
| Time scale | 0.05–5× | 0.1× | Simulation speed multiplier |
| Auto-repeat | on/off + interval | off / 10 s | Restart sim automatically at interval end |
| Max loops | 0–999 | 0 (∞) | Stop after N auto-repeat loops |

---

## Detection Logic

Each pulse:
1. Compute boresight ray from sensor origin in world space
2. **Ray–AABB intersection** (slab method) against the target box — detects any face, no sampling gaps
3. If ray hits box within max range: check **rod occlusion** (ray–cylinder intersection for each rod)
4. If not occluded: **detection registered**

This is physically equivalent to a real 1D lidar: one pulse, one ray, surface contact = detect.

---

## Performance Metrics (sidebar)

| Metric | Description |
|--------|-------------|
| Detections | Total pulses that detected the target |
| Det/s | Rolling detection rate (detections per second) |
| Duty | Detected pulses / total pulses fired (%) |
| TTI | Estimated time to intercept at current closing speed |
| 1st Det | Simulation time of first detection |
| Close m/s | Current closing speed (range rate) |

Status indicators (top bar): **SIM** (running), **DETECT** (last pulse detected), **ROD BLOCK** (last pulse occluded by rod), **STAGE 1/2**.

---

## Display Options

- Show/hide sensor FOV cone (with rod occlusion holes computed per frame)
- Show/hide swept detection volume (Stage 2 torus)
- Show/hide structural rods
- Show/hide target track (trajectory trail)
- Show/hide detection rays (red flash lines from sensor to hit point)

---

## Detection Chart (bottom panel)

Stepped 0/1 timeline: **green fill = detected**, dark = missed. One point per pulse. Rolls left as time advances; window width controlled by the Chart window slider. Simulation time (t) is visible in the 3D overlay (top-left of viewport).

---

## 3D View Controls

| Action | Control |
|--------|---------|
| Orbit | Left-click drag |
| Zoom | Scroll wheel |
| Resize sidebar | Drag the vertical divider |

---

## Running Locally

Open `index.html` directly in a browser (Chrome/Edge/Firefox). No server needed for basic use.

To serve over a local network (useful for testing on another device):

```bash
# Python 3
python -m http.server 3001
```

Then open `http://localhost:3001`.

A PowerShell server (`server.ps1`) is also included for Windows environments without Python.

---

## File Structure

```
index.html    — entire simulation (Three.js scene, physics, UI, Chart.js chart)
server.ps1    — optional local HTTP server for Windows PowerShell
README.md     — this file
```

---

## Version History

| Tag | Description |
|-----|-------------|
| v1.1 | Body dynamics: pitch/yaw/roll sliders |
| v1.2 | Roll marker, FOV cone with rod occlusion holes, pulse beam flash |
| v1.3 | Pulse-based detection (exact Hz rate), loop control, Chart.js chart |
| v1.4 | Bottom panel detection chart, ray–AABB detection, wider sidebar |
