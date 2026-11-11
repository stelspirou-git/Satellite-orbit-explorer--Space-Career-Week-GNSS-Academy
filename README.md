# Satellite Orbit Explorer

A two-body (Keplerian) orbit propagator in Python. It reads satellite mean elements from a General Perturbations (GP/OMM) JSON file, propagates four satellites over 24 hours on a 5-second grid, and produces ground-track and speed plots, Google Earth KML files and a text report.

Built during **Space Career Week 2026 (GNSS Academy)** by following the workshop's eight-stage structure.

![Ground tracks over 24 hours](docs/ground_tracks.png)

## What it does

1. Sets up constants (μ = 398600.4418 km³/s², R⊕ = 6378.137 km) and UTC-aware time handling.
2. Loads GP/OMM JSON element sets (optional live download from CelesTrak, with a 2-hour cache).
3. Converts mean motion, eccentricity and the four angles into physical Keplerian elements; the semi-major axis comes from Kepler's third law.
4. Builds one common time grid starting at UTC midnight of the newest epoch, and flags element sets more than 7 days from the grid start as stale.
5. Propagates each orbit: mean anomaly, Newton-Raphson solution of Kepler's equation, true anomaly, perifocal to inertial (ECI) frame, then to Earth-fixed (ECEF) using GMST.
6. Summarises each orbit (period, perigee, apogee, speed range).
7. Plots ground tracks over a coastline map and speed against time.
8. Exports Google Earth KML files (3D orbit rings and a time-animated version) and a text report.

## Results

The four satellites cover four different orbit types:

| Satellite | NORAD | Period (h) | Perigee / apogee (km) | Inclination (°) | Orbit type |
|---|---|---|---|---|---|
| PULSAR-0 | 64541 | 1.58 | 500 / 506 | 97.48 | Sun-synchronous LEO |
| GSAT0225 (Galileo 29) | 59598 | 14.08 | 23,217 / 23,227 | 55.22 | MEO, navigation |
| SES-5 | 38652 | 23.93 | 35,778 / 35,794 | 0.05 | Geostationary |
| QZS-2 (Michibiki-2) | 42738 | 23.94 | 32,650 / 38,935 | 39.25 | Inclined geosynchronous (figure-eight track) |

The speed plot shows the vis-viva behaviour: flat for the near-circular orbits, and varying between 2.85 and 3.31 km/s for QZS-2 (e ≈ 0.075).

![Speed against time](docs/velocity_timeseries.png)

## Checks built into the notebook

- GMST at J2000 is asserted against the reference value of 280.460618°.
- The Kepler solver is tested on three (M, e) pairs and must reach a residual below 1e-10.
- The Galileo 29 period (14.08 h) is checked against its expected value.

## Limitations

- **Two-body only.** There is no J2, drag, solar radiation pressure or third-body effect, and the Earth is treated as a sphere. GP elements are SGP4 *mean* elements, so using them in a pure Keplerian model is an approximation, and errors grow with time from the epoch.
- **Epoch age matters.** The QZS-2 element set is dated 17 Oct 2026, about 24 days before the propagation window, and the notebook flags it as stale.
- **No independent reference yet.** The positions have not been compared against an SGP4 implementation.
- Positions are geometric ground tracks, with no ground-station visibility, passes or pointing.

## Running it

The notebook was written for Google Colab and uses `/content/...` paths.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/stelspirou-git/satellite-orbit-explorer/blob/main/satellite_orbit_explorer.ipynb)

It needs three input files, which came with the course and are not included in this repository:

- `orbital_elements.json` (GP/OMM element sets)
- `data_manifest.json` (retrieval timestamp)
- `world_map.txt` (coastline segments)

Zip them as `orbit_workshop_inputs.zip` and drop the zip into Colab's file panel when Stage 2 asks for it. Alternatively, set `LIVE_GROUP` in Stage 1 to a CelesTrak group name to pull live elements. Python dependencies are in `requirements.txt` (`matplotlib`, `requests`).

## Possible next steps

- Compare the propagated positions with an SGP4 implementation and report the error over time.
- Add a ground station and compute visibility passes (elevation, azimuth, range).
- Add the J2 secular terms for RAAN and argument of perigee.

## Credits

Workshop structure and input data: GNSS Academy, Space Career Week 2026. Satellite elements are from CelesTrak (GP data).
