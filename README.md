# Rotating Lidar Proximity Sensor — Interceptor Simulation

Interactive 3D simulation of a 1D lidar proximity sensor mounted on an interceptor tube, rotating around the flight axis to create a cone-shaped detection zone.

## Features

- **Accurate sensor model**: 4×2×6 cm sensor cube with laser diode and receiver lenses
- **Two simulation stages**:
  - Stage 1: Fixed sensor at mount angle X from flight axis
  - Stage 2: Sensor rotating at R RPM for full 360° coverage
- **Structural rod occlusion**: 3 or 4 rods with geometric ray-cylinder intersection check
- **FOV cone**: Configurable full cone angle D, proper cone-box detection against target
- **Swept volume visualization**: Annular cone showing full detection zone during rotation
- **6 engagement scenarios**: Head-on, tail chase, crossing (H/V/45°), oblique
- **Live performance metrics**: Detection count, rate, duty cycle, time-to-intercept, closing speed
- **Real-time plots**: Detection timeline and range-to-target

## Usage

Open `index.html` in a browser, or serve locally:

```bash
# Python
python -m http.server 3001

# Perl (if Python unavailable)
perl -MIO::Socket::INET -e '...' 
```

Then open [http://localhost:3001](http://localhost:3001).

## Controls

- **Drag** to orbit the 3D view, **scroll** to zoom
- Adjust parameters in the right sidebar (resizable by dragging the divider)
- Click **Start Simulation** to run, **Pause** to freeze, **Reset** to restart
