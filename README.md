# N-Defender Drone Detection System

**7" Touchscreen, 1 MHz–6 GHz RF Coverage, Analog FPV Receive, DF & Triangulation**

> Status: **Draft v0.1**
> Owner: AnuShakti Infotech Pvt. Ltd. (FlySpark)
> Target use: FPV/RC spectrum monitoring, drone presence detection, Remote ID display, direction finding (DF), coarse-to-precise localization, and optional analog 5.8 GHz video reception.

---

## 1) Executive Summary

Build a rugged, field-deployable handheld/vehicle-mounted system with a 7″ capacitive touchscreen that:

* Scans and monitors **≈1 MHz–6 GHz** for drone-related activity.
* Detects **analog FPV (5.8 GHz)** and displays live video.
* Provides **direction finding (DF)** bearings and **location estimates** via single-station drive-around or multi-station triangulation.
* Decodes and displays **Remote ID** when present (BLE/Wi‑Fi).
* Presents a **beautiful, operator-friendly UI** with map, spectrum, targets list, and statistics.

---

## 2) Objectives & Success Criteria

**Objectives**

* Real-time awareness of drone/control/video links across key bands: 433/868/915 MHz, 1.2–1.3 GHz, **2.4 GHz**, **5.8 GHz**.
* **20 km** (LoS) **detection** potential using appropriate high‑gain antennas, LNAs, and clean RF front-ends.
* **Sub‑GHz true DF** (phase-coherent) and **2.4/5.8 GHz presence+bearing** via RSSI/directivity; optional high‑band DF via down‑conversion.

**Success Criteria**

* Detect & list active RF emitters with **<2 s** discovery latency (configurable).
* Show **bearing** with **≤±10°** RMS error (sub‑GHz DF, good geometry).
* Show **coarse location** (single station, drive-around) and **pinpoint** (two‑station triangulation) with **<150 m CEP** under LoS conditions.
* Display analog FPV with **≤200 ms** end‑to‑end latency.
* Log all events and export CSV/KML/GeoJSON.

---

## 3) Scope

**In scope**: Passive RF detection, analog FPV reception, Remote ID decoding, DF & triangulation, UI, logging, export, multi-node fusion.

**Out of scope**: Any active transmission/jamming. Decoding proprietary encrypted digital video/control links (e.g., DJI OcuSync) beyond legal metadata.

---

## 4) Definitions

* **DF (Direction Finding):** Determining angle-of-arrival (AoA) of a transmitter.
* **RID (Remote ID):** Standards-compliant broadcast (BLE/Wi‑Fi) with drone ID/telemetry.
* **RSSI:** Received Signal Strength Indicator (log detector output or SDR-derived).

---

## 5) System Architecture (High-Level)

**Blocks**

1. **Compute/UI**: Raspberry Pi 4/5 running kiosk UI (React/Browser) + control daemons.
2. **Wideband SDR**: 1 MHz–6 GHz scanning (HackRF or USRP tier).
3. **Coherent DF array**: 5‑channel coherent SDR for sub‑GHz AoA (Kraken‑class).
4. **Analog FPV Receiver**: 5.8 GHz (RX5808/diversity) → USB capture → UI window.
5. **RSSI Heads**: AD8318 log detectors with directional antennas for fast 2.4/5.8 bearing.
6. **GNSS & IMU**: GPS for geotagging; IMU/compass for heading stabilization.
7. **Networking**: Wi‑Fi/LTE for multi‑node triangulation & remote viewing.

**Data Flow**

* SDR/DF pipelines → Detection service → Event bus (WebSocket) → UI.
* RID receiver → RID service → Map overlays & database.
* Analog RX → UVC capture → UI video panel.
* Logs → SQLite/CSV/KML exporters.

---

## 6) RF Coverage & Performance Targets

* **Frequency coverage**: Target **≈1 MHz–6 GHz** (SDR + log detectors + analog RX).
* **Key bands**: 433/868/915 MHz (control/telemetry), 1.2–1.3 GHz (video), **2.4 GHz** (control/digital video), **5.8 GHz** (analog video).
* **Sensitivity goal**: Achieve front-end system **NF < 2 dB** (per band with LNA near feed) and minimize cable loss.
* **Dynamic range**: Front-end filtering to prevent overload (SAW/BPF per band).
* **DF accuracy**: ≤±10° RMS (sub‑GHz) with calibrated 5‑antenna array; high‑band optional via down‑conversion.
* **Range (LoS)**: Presence detection to ≈20 km using 18–27 dBi directional antennas + low‑loss coax + LNAs.

---

## 7) Hardware Requirements (Named Blocks)

### 7.1 Compute & Display

* **C1 – UI Brain**: Raspberry Pi 4/5 (4–8 GB), microSD/SSD, USB 3.0 ports.
* **C2 – 7″ Touch**: 7″ capacitive HDMI/DSI. Brightness ≥ 400 nits, 800×480 or higher.
* **C3 – Field Compute (opt.)**: x86 NUC/Jetson for heavy DSP/GNU Radio flows.

### 7.2 SDR & DF

* **R1 – Wideband SDR (value)**: HackRF One (1 MHz–6 GHz) for scan/waterfall/capture.
* **R2 – Wideband SDR (pro)**: USRP B200mini/B205mini‑i (70 MHz–6 GHz, up to 56 MHz BW).
* **D1 – Coherent DF Engine**: 5‑channel coherent SDR (Kraken‑class) + server.
* **D2 – DF Antenna Set**: 5× matched whips (100 MHz–1 GHz), equal‑length coax, roof/mast mount.
* **R3 – High‑Band Down‑Converters (opt.)**: 2.4 GHz & 5.8 GHz → sub‑GHz IF for coherent DF.

### 7.3 Analog Video Chain (5.8 GHz)

* **V1 – 5.8 GHz Analog Receiver**: RX5808/diversity or ready base; A/V → USB UVC capture.
* **V2 – Directional Antenna**: 24–27 dBi grid/parabolic for long‑range reception.

### 7.4 RSSI/Bearing Modules

* **M1 – Log Detector**: AD8318 module (1 MHz–6 GHz) + MCU (UART/USB) for RSSI stream.
* **M2 – Pan/Tilt (opt.)**: Servos + driver to auto‑sweep 2.4/5.8 GHz antennas.

### 7.5 Antennas (by band)

* **A1 – Sub‑GHz**: Omni (discovery) + 12–15 dBi Yagi (bearing/range).
* **A2 – 2.4 GHz**: 18–25 dBi Yagi/panel; optional circular polarization variants.
* **A3 – 5.8 GHz**: 24–27 dBi grid/parabola; diversity patch for nearby/medium range.
* **A0 – Coax & Accessories**: Low‑loss LMR‑400/240, SMA/N, arrestors, weatherproofing.

### 7.6 Front‑End Conditioning

* **F1 – LNAs**: Low‑NF LNAs per band (433/868/915, 1.3, 2.4, 5.8 GHz) at antenna.
* **F2 – Filters**: SAW/BPF per band; optional notch filters for cellular/Wi‑Fi where needed.
* **F3 – RF Splitters/Combiners**: Share antennas sensibly; protect inputs.

### 7.7 Geo/Timing/Controls

* **G1 – GNSS**: USB GPS (u‑blox class) for position/time.
* **G2 – IMU/Compass**: 9‑axis for heading overlay & tilt compensation.
* **G3 – Timebase (opt.)**: GPSDO/10 MHz ref if distributing clocks.

### 7.8 Power & Enclosure

* **P1 – Power**: 12 V LiFePO₄ or vehicle DC; DC‑DC rails: 5 V/3–6 A (Pi/USB), clean 5 V for DF engine.
* **P2 – Enclosure**: Rugged case, airflow, EMI shielding, dust/water resistance goals (IP‑rating target per environment).
* **P3 – I/O**: Powered USB 3 hubs, shielded cables, cable glands, strain reliefs.

---

## 8) Software Requirements

### 8.1 Platform & Services

* **OS**: Raspberry Pi OS (64‑bit) + systemd services; optional Ubuntu for NUC.
* **SDR Stack**: GNU Radio, SoapySDR, RTL‑SDR libs, device drivers.
* **DF Stack**: Coherent DF server (Kraken‑compatible), bearing output via REST/WebSocket.
* **RID Stack**: OpenDroneID (BLE/Wi‑Fi) receiver; local service to publish RID targets.
* **Analog Video**: GStreamer/FFmpeg pipeline (UVC capture → H.264/RAW → UI canvas).
* **Fusion/Logic**: Python/Node services for detection, classification, track management, triangulation (least‑squares intersection, confidence ellipses).
* **Storage**: SQLite + CSV/KML/GeoJSON exporters; rolling logs with retention policy.

### 8.2 UI/UX

* **Framework**: React (SPA) served locally; Tailwind UI style guide.
* **Views**: Map (Leaflet), Spectrum/Waterfall, Targets List, Signal Detail, Analog Video panel, Settings, Logs.
* **Real‑time**: WebSocket bus; update rate ≥ 10 Hz for meters/graphs; spectrum frame rate ≥ 15 fps.
* **Accessibility**: Big touch targets (≥44 px), high contrast, day/night themes.

### 8.3 APIs (internal)

* `/api/targets` (GET/WS): list of detections {id, freq, band, rssi, bearing, pos_est, type, age}.
* `/api/bearings` (POST): submit bearing from remote node {node_id, lat, lon, heading, bearing, sigma, t}.
* `/api/video` (GET): MJPEG/H.264 stream endpoint when analog present.
* `/api/export` (POST): CSV/KML/GeoJSON export with filters.
* `/api/config` (GET/POST): thresholds, scansets, LNA states, recorder.

### 8.4 Non‑Functional Requirements

* Boot‑to‑UI ≤ 30 s; watchdog for service restarts.
* Data integrity: graceful power‑loss handling; journaled FS.
* Security: local‑only by default; optional VPN for remote nodes; API token for writes.

---

## 9) Detection Modes (Operational)

1. **Survey Mode**: wideband scan → show hotspots; tap to focus.
2. **Monitor Mode**: lock to band/set of channels; alarms on thresholds.
3. **DF Mode (Sub‑GHz)**: live AoA needle; record bearings while moving; auto‑solve.
4. **High‑Band Bearing**: RSSI + directional sweep on 2.4/5.8 (manual/auto pan‑tilt).
5. **RID Mode**: show RID drones and pilot location when available.
6. **Analog FPV Mode**: show live video; record snapshot/clip.

---

## 10) Triangulation & Localization

* **Single‑node**: accumulate bearings over path; compute LSQ intersection against map grid.
* **Multi‑node**: at least two nodes submit bearings → intersect vectors; output position + confidence ellipse; compensate for compass offset.
* **Map Layers**: target pins (status colors), bearing cones, heatmaps, history trails.

---

## 11) UI Design Notes (7″)

* **Main**: Map center; right rail = targets; bottom strip = video + quick stats.
* **Detail Card**: freq, band, type, RSSI dial, bearing ring, RID fields (if any), last‑seen, buttons: Track, Mark Safe, Export.
* **Spectrum**: pinch/zoom, peak markers, channel presets.
* **Themes**: Day/Night; Alert sounds & haptics.

---

## 12) Test & Validation Plan

**Bench**

* Calibrate RSSI chain with signal generator; verify NF and gain per band.
* Validate DF array with fixed beacon at known azimuths (±5° bins).

**Field**

* LOS range tests at 2.4/5.8 with 18–27 dBi antennas; record detection vs distance.
* Drive‑around DF: bearing residuals, CEP metrics for solved positions.
* RID test with compliant drone; verify map telemetry & metadata.
* Analog video: latency & SNR at 1/5/10/20 km (where lawful & practical).

**Acceptance**

* Meets success criteria (§2) across 3 representative environments (rural, suburban, light urban).

---

## 13) Compliance, Legal & Safety

* Passive monitoring only; no transmission/jamming.
* Respect local telecom/privacy laws; handle RID/pilot info per policy.
* Electrical: proper grounding, surge protection, fusing; thermal tests in hot climate.

---

## 14) Risk Register & Mitigations

* **R1 Overload/Intermod**: Use SAW/BPF, step attenuators, staged LNA placement.
* **R2 DF Accuracy Drift**: Routine array calibration; compass hard/soft‑iron calibration.
* **R3 High‑Band DF**: Use RSSI bearing or add down‑converters/alt hardware.
* **R4 Power/Heat**: Active cooling, thermal throttling, fan curves.
* **R5 UI Complexity**: Progressive disclosure; presets; operator training.

---

## 15) Bill‑of‑Materials (Selection Matrix – to be finalized)

| Subsystem          | Budget (Good)          | Pro (Best)                         | Notes                                    |
| ------------------ | ---------------------- | ---------------------------------- | ---------------------------------------- |
| Compute/UI         | Pi 4/5 + 7″ capacitive | Pi 5 + M.2 SSD + brighter 7″       | Pi handles UI; heavy DSP can move to NUC |
| Wideband SDR       | HackRF One             | USRP B200mini/B205mini‑i           | 1–6 GHz coverage; better SNR on USRP     |
| Coherent DF        | Kraken‑class 5‑ch      | Same + GPSDO                       | Sub‑GHz AoA                              |
| Down‑conversion    | None                   | 2.4/5.8 → IF modules               | For high‑band coherent DF                |
| Analog FPV         | RX5808/diversity + UVC | Higher‑grade diversity RX          | 5.8 GHz video                            |
| RSSI Heads         | AD8318 + MCU           | Dual heads (2.4 & 5.8) + pan/tilt  | Fast bearing                             |
| Antennas (Sub‑GHz) | Omni + 12 dBi Yagi     | Higher‑gain Yagi + mast            | Range vs beam                            |
| Antennas (2.4)     | 18 dBi panel           | 23–25 dBi Yagi                     | Narrow beam for 20 km                    |
| Antennas (5.8)     | 24 dBi grid            | 27 dBi grid/parabola               | Careful alignment                        |
| LNAs/Filters       | Per‑band LNAs + SAW    | Low‑NF LNAs + switchable BPFs      | Put LNAs at feed                         |
| GNSS/IMU           | USB GPS + 9‑axis       | GPSDO + calibrated IMU             | Geo & heading                            |
| Power              | 12 V LiFePO₄ + DC‑DC   | Vehicle power + clean rails        | EMI‑quiet supplies                       |
| Enclosure          | Rugged case            | IP‑rated, fan‑cooled, EMI shielded | Fieldable                                |

---

## 16) Project Plan (Milestones)

1. **M0 – Spec Freeze** (this doc) → sign‑off.
2. **M1 – PoC**: UI skeleton, SDR scan, analog video panel.
3. **M2 – DF Online**: Sub‑GHz AoA bearings; drive‑around solve.
4. **M3 – High‑Band Bearing**: 2.4/5.8 RSSI sweep + pan/tilt.
5. **M4 – RID Integration**: BLE/Wi‑Fi decoding and map overlay.
6. **M5 – Multi‑Node**: Bearing fusion server; CSV/KML/GeoJSON exports.
7. **M6 – Field Validation**: Range/accuracy tests; thermal & power burn‑in.
8. **M7 – Release v1.0**: Packaging, ops guide, spares list.

---

## 17) Deliverables

* Hardware pick list (with supplier links & INR estimates).
* Wiring & mounting drawings (roof array, pan/tilt, enclosure layout).
* Pi/NUC images (pre‑installed services, configs, presets).
* Operator handbook (UI guide; testing & troubleshooting).

---

## 18) Open Items (to decide)

* Final SDR choice (HackRF vs USRP tier).
* Whether to include high‑band down‑converters for coherent DF.
* Antenna SKUs per band (gain vs portability trade‑off).
* Power strategy (stand‑alone pack vs vehicle primary).

---

## 19) Change Log

* v0.1: Initial draft with architecture, requirements, selection matrix.
