# NIDAR AirMouse — Team Task Breakdown & Learning Plan

**Team:** HANTRAMANAV
**Mission:** Track 1 (Drone Innovation) — GPS-denied indoor search, mapping & survivor localisation

---

## Workstreams & Ownership

### Hardware assembly & power — Salman, Rithik
- Frame build, motor/ESC mounting, prop balancing, wiring the 6S battery → 4-in-1 ESC → Pixhawk 6C/PM02
- Setting up the dedicated 5V/5A UBEC rail for the Pi 4 + USB hub, kept electrically separate from the ESC BEC
- Bench power-on, ESC calibration (DShot600), motor direction/rotation checks

### Sensor integration — Kishore, Yogeshwar
- Wiring and bringing up each sensor individually: RPLIDAR A2M8 (USB), Matek 3901-L0X (UART/I2C to Pixhawk), OAK-D (USB3), FLIR Lepton + PureThermal (USB UVC)
- Confirming each sensor works in isolation before integration (LiDAR scan visible, OAK-D stream live, thermal frames readable)

### IoT / comms — Rahul
- RFD900x telemetry link setup (868MHz India variant), MAVLink routing between Pixhawk, companion computer, and ground station
- ExpressLRS RC link as the manual safety-override channel
- USB hub power budgeting so RPLIDAR + OAK-D + thermal cam don't brown out the Pi

### Software & ML — Suyambu, Naveen B
- **Suyambu:** training/compiling the person-detection model to Myriad X `.blob` format for on-device OAK-D inference, thermal heat-blob detection logic, RGB+thermal fusion confidence scoring
- **Naveen B:** the ROS 2 mission nodes — detection aggregator, survivor log/reporter, mission state machine

### ROS 2 / SLAM / flight bridge — Rithik, GK (you)
- ROS 2 Humble setup on the Pi, `slam_toolbox` bring-up on the RPLIDAR feed, MAVROS/MAVSDK bridge to ArduPilot, EKF3 GPS-denied tuning (optical flow + external nav from SLAM pose)
- This is the riskiest technical thread — it's where the whole system either holds together or doesn't, so plan the most bench time here

### Design — Naveen
- Frame CAD (the v2 STL already referenced in the BOM), sensor mounts (vibration isolation for the optical flow sensor especially), OAK-D/thermal camera alignment fixture so their fields of view are known and fixed relative to each other

### Docs & presentation — Sweetha
- Consolidating the BOM, specs, and process doc into competition submission format
- Documenting test results and flight logs as they come in

---

## Suggested Build Order

So work doesn't block on other work:

1. **Hardware + sensor bring-up in parallel** — Salman/Rithik on frame+power, Kishore/Yogeshwar on sensors. These don't depend on each other.
2. **Rahul's comms link** can be tested standalone with just the Pixhawk on the bench.
3. **ROS 2/SLAM** (you + Rithik) starts once RPLIDAR is confirmed working.
4. **Suyambu's model training** can happen entirely offline/on a laptop — doesn't need the hardware at all, so start it now in parallel.
5. **Naveen B's mission nodes** need the detection pipeline (Suyambu) and SLAM pose (you) as inputs — this is the critical path. Start stubbing interfaces early so nobody's blocked waiting on the others.
6. **Naveen's mounts** need to be finalized before final sensor wiring, so lock OAK-D/thermal/LiDAR placement early even if the model/software work is still ongoing.

---

## What Each Person Should Learn

| Person | Core skill to build |
|---|---|
| Salman | ESC/motor wiring, DShot config, power budgeting, basic Pixhawk parameter setup |
| Rithik | ROS 2 fundamentals (nodes/topics), MAVROS bring-up, hardware-software integration debugging |
| Kishore, Yogeshwar | I2C/UART/USB sensor interfacing, RPLIDAR SDK basics, ROS 2 driver bring-up |
| Rahul | MAVLink protocol basics, telemetry radio configuration, serial comms troubleshooting |
| Suyambu | DepthAI SDK, OpenVINO model conversion to Myriad X blob, OpenCV for thermal processing |
| Naveen B | ROS 2 node development (Python), state machine design |
| Naveen | CAD (Fusion 360/Onshape), 3D printing tolerances for PETG, vibration isolation design |
| Sweetha | Technical documentation structure, reading flight logs well enough to summarize them accurately |
| Gopikrishna (GK) | Full-stack integration — ROS 2, EKF3 tuning, SLAM, and understanding how every subsystem's output feeds the next one |

---

## Biggest Risk

The ROS 2/SLAM/EKF3 integration is the single biggest risk to the timeline — it's the one piece nobody but Rithik and GK can really own, and the one most likely to eat unplanned time. Start bench-testing that thread as early as possible, even with sensors only partially mounted, rather than saving it for last.
