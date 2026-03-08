---
title: "Robotic Workcell for Automated Bending: A Vision‑Enabled 7‑Axis Cobot, PLC, and Digital‑Twin UI"
date: 2026-02-18
draft: false
author:
  - "Shivam Shivam"
tags:
  - Industrial Automation
  - Metal Sheet Bending
  - Collaborative Robots
  - Computer Vision
  - PLC
  - ROS & Gazebo
  - Digital Twin
  - Web UI
image: /figures/robotic-workcell1.png
description: "A full-stack master’s thesis implementation of a robotic workcell that automates sheet-metal bending using a 7‑axis cobot, dual vision sensors, PLC orchestration, and a web-based digital twin."
toc: true
---

## Abstract

This work demonstrates a **robotic workcell** that automates a **sheet‑metal bending machine** with a **7‑axis collaborative robot**, **two vision sensors**, and a **web-based UI**—the latter serving as a **digital twin** for visualization and monitoring. The robot performs **automatic loading/unloading** of sheets; a wrist‑mounted camera enables **sheet detection and localization**, while a second camera conducts **post-bend angle measurement** for **closed‑loop quality control**. A **PLC** coordinates the bending machine, grippers, and inspection tasks. The proposed system reduces manual labor, increases throughput, and improves consistency of bend accuracy, offering a pragmatic blueprint for **rapid development and deployment** of robotic bending cells in industrial contexts.

**Keywords:** Robotic Workcell, Automation, Metal Sheet Bending, 7‑axis Robotic Arm, Computer Vision, Web UI, Digital Twin

---

## 1. Introduction

### 1.1 Background & Motivation
Industry 4.0 connects cyber‑physical systems through data, enabling **flexible production**, **predictive insight**, and **closed‑loop optimization**. Within this paradigm, manual sheet‑metal bending remains **labor-intensive** and **error‑prone**. This project addresses the gap with a **collaborative robot** capable of safe, adaptable bending operations, guided by **on‑arm perception**, **PLC‑driven orchestration**, and a **digital twin** that shortens commissioning, supports diagnostics, and improves operator insight.

### 1.2 Problem Statement
Manual bending leads to variability, quality drift, and ergonomic risks. Automating the process with a cobot requires: (i) **reliable sheet detection** and grasp planning; (ii) **precise bend execution** synchronized with press‑brake kinematics; (iii) **coordinated multi‑device control** (robot, grippers, sensors, PLC); and (iv) **operator‑friendly monitoring**.

### 1.3 Objectives
1. Engineer a **cobot‑based workcell** that loads/unloads sheets and executes bends autonomously.  
2. Integrate **vision** for **sheet detection** and **post‑bend angle measurement** with automated data exchange (robot↔camera, PLC↔camera).  
3. Build a **web UI** that acts as a **digital twin** for real‑time workcell monitoring.  
4. Evaluate **efficiency** and **accuracy** versus incumbent practices.

---

## 2. System Overview

The workcell is arranged around a press brake and includes a **storage station (10‑drawer shelf)**, an **unloading station** with a **gantry + swivel gripper**, the **7‑axis cobot** with a **pneumatic parallel gripper** and **robotic camera**, and the **inspection camera**. A **PLC** governs interlocks and sequences; the **web UI** visualizes the robot and task state as a digital twin.

<figure>
  <img src="/figures/robotic-workcell1.png" alt="Robotic workcell layout in simulation" width="100%">
  <figcaption><strong>Figure 1.</strong> Robotic workcell layout in simulation: 1) Bending machine, 2) Storage station, 3) Unloading station, 4) Handling robot, 5) Terminal‑operating robot area, 6) Safety fence.</figcaption>
</figure>

---

## 3. Hardware

### 3.1 Bending Machine (AMADA HFP 50-20)
A legacy **hydraulic press brake** (AMNC controller) is adapted for automation via **PLC‑driven foot‑pedal signals**. A **laser safety/monitoring** device is retained; a **laser distance sensor** is added to measure **open height** for synchronized robot release and collision avoidance.

**Table 1 — Bending Machine Technical Specifications**

| Description | Value |
|---|---|
| Model | HFP 50‑20 |
| Drive | Hydraulic press brake |
| Controller | AMADA AMNC graphic (color) |
| Controlled Axes | Y1/Y2, X1/X2, R1/R2, Z1/Z2 |
| Open Height | 470 mm |
| Stroke | 200 mm |
| Bending / Approach / Return Speeds | 10 / 100 / 100 mm/s |
| Working Length | 2090 mm |
| Distance Between Frames | 1665 mm |
| Laser Monitoring | FIESSLER |
| Dimensions (L×W×H) | 3458 × 2450 × 2450 mm |
| Weight | 4850 kg |

<figure>
  <img src="/figures/bending-machine.png" alt="Bending machine components and instrumentation" width="100%">
  <figcaption><strong>Figure 2.</strong> Instrumented bending machine: (1) Marker; (2) Open‑height laser sensor; (3) Bending stations; (4) Terminal fixture; (5) Laser monitoring.</figcaption>
</figure>

**Bending stations.** Three tool sets support 90°, 135°, and flattening operations:

<figure>
  <img src="/figures/bending-station.png" alt="Three bending stations" width="70%">
  <figcaption><strong>Figure 3.</strong> Three bending stations for sequential bends.</figcaption>
</figure>

### 3.2 Collaborative Robot (Kassow Robots KR1410)
A 7‑axis cobot is selected for **redundancy** in confined spaces, improved **reach**, and **singularity avoidance**.

**Table 2 — KR1410 Technical Specifications**

| Description | Value |
|---|---|
| Type | Collaborative, 7‑axis |
| Reach | 1400 mm |
| Repeatability | 0.1 mm |
| Max Payload | 10 kg |
| Typical Power | 400–1200 W |
| Joint Ranges | J1,3,5,6,7 ±360°; J2,4 −70°/+180° |
| Max Joint Speed | 163/225 °/s |
| Footprint | 160 × 160 mm |
| Ingress Protection | IP54 |
| Interfaces | Profinet; ROS support available |

<figure>
  <img src="/figures/kassow-robot-parts.png" alt="KR1410 functional parts" width="50%">
  <figcaption><strong>Figure 4.</strong> KR1410: controller, teach pendant, tool‑IO.</figcaption>
</figure>

### 3.3 End‑Effectors & Grippers
Primary end‑effector is a **Schunk PGN-plus-P 80‑1** **pneumatic parallel gripper** (≈550–610 N forces, 8 mm stroke/jaw). A **manual quick‑change** plate allows finger/tip swaps tailored to part variants.

<figure>
  <img src="/figures/gripper.jpeg" alt="Schunk gripper" width="50%">
  <figcaption><strong>Figure 5.</strong> Schunk PGN-plus-P 80‑1 pneumatic gripper.</figcaption>
</figure>

### 3.4 Vision Sensors (SensoPart VISOR V20)
Two **VISOR** vision sensors are used:

- **Robotic Camera (on-arm)**: sheet detection, markers, and auto **hand‑eye calibration**.  
- **Inspection Camera (fixed)**: **angle measurement** post‑bend using internal red LED illumination.

**Table 3 — VISOR V20 (Robotic/Object) Key Technical Data**

| Description | Value |
|---|---|
| Supply | 24 VDC (18–30 V); ≤300 mA |
| Interfaces | 100 M LAN, PROFINET, EtherNet/IP, SensoWeb |
| Protection | IP65/IP67; − |
| Resolution | 1440 × 1080 (CMOS, mono/color) |
| Lens | Integrated 6.5 mm (wide), motorized focus |
| Light | Red LED; class‑1 target laser |
| Housing | Die‑cast aluminum |
| Weight | ~200 g |
| Operating Temp | 0–50 °C (80% RH, non‑condensing) |

<figure>
  <img src="/figures/vision-sensor.png" alt="SensoPart VISOR" width="50%">
  <figcaption><strong>Figure 6.</strong> VISOR vision sensor.</figcaption>
</figure>

### 3.5 Storage & Unloading Stations
- **Storage:** 10‑drawer **shelf** with robot‑grippable locking handles and **marker plates** per drawer for localization.  
- **Unloading:** Gantry + **swivel gripper (180°)** separates a single sheet from a stack and hands it over to the cobot.

<figure>
  <img src="/figures/unloading-station-front-blender.png" alt="Unloading station front" width="100%">
  <figcaption><strong>Figure 7.</strong> Unloading station: front view (PLC, gripper, swivel unit in cabinet below).</figcaption>
</figure>

---

## 4. Software & Communications

### 4.1 Simulation & Motion
- **ROS (Noetic) + Gazebo**: full workcell simulation (URDF/SDF), kinematics, trajectories.  
- **MoveIt**: planning scene with **collision meshes**, interactive planning, and tool control.  
- **RViz**: visualization and TF introspection.

<figure>
  <img src="/figures/gazebo-rviz.png" alt="Gazebo and RViz views" width="100%">
  <figcaption><strong>Figure 8.</strong> Gazebo (left) and RViz (right) during simulation and planning.</figcaption>
</figure>

### 4.2 Web‑Based Digital Twin (UI)
A **Node.js** server hosts a **React** front end. **rosbridge** provides a WebSocket to ROS; **roslibjs** and **ros3djs** render the robot URDF and interactive markers for jogging and path preview. The UI shows the **live robot state**, **modes**, and **task panels**, enabling monitoring and (optionally) supervisory control.

<figure>
  <img src="/figures/webui/webui0.png" alt="Web UI overview" width="100%">
  <figcaption><strong>Figure 9.</strong> Web UI acting as a digital twin for the workcell.</figcaption>
</figure>

### 4.3 PLC Integration
**Profinet** connects PLC↔robot and PLC↔inspection camera. We map user I/O for handshakes, interlocks, and task states.

**Table 4 — KR1410 → PLC (selected)**

| Register | Purpose |
|---|---|
| Bit\[0] | Unloading gripper open/close |
| Bit\[1] | Robotic gripper open/close |
| Bit\[2] | Bending start request/release |
| Bit\[3] | Inspection trigger request/release |
| Bit\[4] | Robot active |
| Bit\[5] | Sheet request |
| Int\[0] | Robot program state (0–10: ready, bending steps, placing, calibration) |
| Int\[1] | Robotic camera trigger (0 none, 1 ok, 2 fail) |
| Int\[2] | Robotic camera health (0 ok, else restart) |

**Table 5 — PLC → KR1410 (selected)**

| Register | Purpose |
|---|---|
| Bit\[0–7] | Bending stage flags + finished/open indicators |
| Bit\[8] | Start bending sequence |
| Bit\[9] | Sheet ready in unloading station gripper |
| Bit\[10] | Terminal correction sequence in progress |
| Bit\[11] | Inspect request |
| Bit\[12] | Pause program |
| Bit\[13] | Calibration request |
| Bit\[14] | Terminate program |
| Bit\[15] | Storage station secured |
| Int\[0] | Inspection result (0 wait, 1 ok, 2 fail) |

### 4.4 Vision Communications
- **KR1410 ↔ VISOR (Robotic Camera):** custom **CBun device** exchanges data via **Telegram** protocol over **TCP** (ports 2005 implicit, 2006 explicit). Functions: switch jobs, trigger capture, return **pose** and **detection state**, run **auto‑calibration**.  
- **PLC ↔ VISOR (Inspection Camera):** **PROFINET**, and **UDP** ports (161, 34962–34964) for inspection triggers and results.

<figure>
  <img src="/figures/visor-cbun-connection.png" alt="KR CBun device dialog" width="55%">
  <figcaption><strong>Figure 10.</strong> KR CBun device dialog for VISOR communications.</figcaption>
</figure>

---

## 5. Safety Engineering

### 5.1 Payload & Workspace Limits
The mounted tool set (gripper + quick‑change + camera) ≈ **2.0 kg**; the sheet payload ≈ **0.1 kg** (negligible for worst‑case). Only ~1.1 m of the 1.4 m reach is exploited, remaining within the **static torque** envelope.

### 5.2 Stopping Time/Distance
We conservatively estimate braking time and distance for joint‑space and Cartesian moves. Let the maximum linear speed be \(v_{\max}\) and distance from joint axis to payload be \(r\). Converting to joint speed:
\[
\omega = \frac{180\,v_{\max}}{\pi r}\quad [\deg/s]
\]
With braking acceleration \(a_{\text{brake}}\),
\[
t_{\text{brake}} = \frac{\omega}{a_{\text{brake}}} + 0.020,\quad
s_{\text{brake}} = \left(\frac{t_{\text{brake}} + 0.02}{360}\right)\pi r\,\omega
\]
Speeds are reduced for challenging configurations (e.g., low joint‑torque margins) to keep \(s_{\text{brake}}\) within bounds.

### 5.3 Virtual Safety Zones & Functions
**Safety zones** cordon off the unloading, storage, press brake, terminal area, and floor plane. The **Protective Stop** pauses and **Emergency Stop** cuts power to actuators. Additional E‑stops on the press brake and station cabinet are wired via PLC for **global** halt.

<figure>
  <img src="/figures/safety-zones.png" alt="Safety zones in workcell" width="70%">
  <figcaption><strong>Figure 11.</strong> Virtual safety zones configured for the workcell.</figcaption>
</figure>

---

## 6. Calibration

### 6.1 Kinematic Model
The KR1410’s factory calibration is used on hardware; simulation URDF is tuned to match tool transforms (TCP located **216 mm** from the tool flange center).

### 6.2 Hand–Eye Calibration (On‑Arm Camera)
**Goal:** estimate the rigid transform between the camera and the robot TCP so that detected **poses** are emitted **in robot coordinates**.  
**Procedure:** capture ~20 images of a **calibration marker** at varied TCP poses (tilts + translations) ensuring full **field‑of‑view coverage**; compute camera intrinsics and the hand‑eye extrinsics; write updated parameters to the KR CBun device.

<figure>
  <img src="/figures/001calibration/calibration.png" alt="Hand-eye calibration samples" width="100%">
  <figcaption><strong>Figure 12.</strong> Sample captures during automated hand–eye calibration.</figcaption>
</figure>

**Runtime:** The automated sequence completes in **≈98 s**.

### 6.3 Workspace Calibration
Markers on **fixed** subsystems (press brake, unloading, storage drawers) establish **repeatable frames** for task geometry and drawer localization after maintenance or layout changes.

---

## 7. Operational Workflow

We map the manual bending into five automated stages, with robot/PLC/vision coordination:

1. **Unloading:** Gantry delivers a single sheet to the **unloading gripper** (swivel 180°).  
2. **Alignment:** Robotic camera detects **sheet features**; cobot grasps the sheet, then aligns it with the press‑brake **5‑axis backgauges**.  
3. **Bending:** Depending on part size, the robot either **releases** during bend (small parts) or **tracks** the bend (large parts) to avoid torsion and tool collisions.  
4. **Checking:** Robot presents the part to the **inspection camera** for **angle measurement**; if out‑of‑tolerance, a **reject** path and **terminal correction** are triggered.  
5. **Loading:** Finished parts are placed in **storage drawers**—with optional **regrasp** at unloading station to enable a collision‑free final placement.

<figure>
  <img src="/figures/sheet-pickup/sheet-placement05.png" alt="Sheet pickup sequence" width="100%">
  <figcaption><strong>Figure 13.</strong> Sheet pickup and handover sequence.</figcaption>
</figure>

<figure>
  <img src="/figures/bending/bending1-003.png" alt="Bending operation" width="100%">
  <figcaption><strong>Figure 14.</strong> Bending station operation under PLC‑synchronized control.</figcaption>
</figure>

<figure>
  <img src="/figures/inspection-setup.png" alt="Inspection setup" width="60%">
  <figcaption><strong>Figure 15.</strong> Post‑bend angle inspection with VISOR (internal red LED illumination).</figcaption>
</figure>

---

## 8. Vision Methods

### 8.1 Robotic Camera — Sheet & Marker Detection
- **Detectors:** *Contour* (object edges/shape) and *Target Mark 3D* (fiducials).  
- **Acquisition:** WD ≈ **300 mm**; multiple shutter presets (15–90 ms) to combat reflectivity variance; external illumination recommended at this distance.  
- **Output:** **Pose** in robot frame + **detection flag** via CBun/Telegram.

<figure>
  <img src="/figures/robotic-detection.png" alt="Robotic camera detections" width="100%">
  <figcaption><strong>Figure 16.</strong> Robotic camera: sheet feature detection (left) and marker pose (right).</figcaption>
</figure>

### 8.2 Inspection Camera — Angle Measurement
- **Detectors:** Alignment + **Calipers** (dark‑to‑light transitions) feeding **Result‑Processing/Math** to compute the **bend angle**.  
- **Acquisition:** WD ~**90–130 mm**, **1.0 ms** shutter; internal red LED only (ambient light effectively excluded).

<figure>
  <img src="/figures/measurement-detector.png" alt="Caliper/Math detectors for angle" width="60%">
  <figcaption><strong>Figure 17.</strong> Multi‑caliper and math pipeline for angle computation.</figcaption>
</figure>

---

## 9. Integration & Tests

### 9.1 Sheet Pickup
From **contour detection** to pose transfer and pick execution; recovery logic retries with alternative shutter settings if needed.

<figure>
  <img src="/figures/sheet-pickup/sensoconfig.PNG" alt="VISOR to robot pose transfer" width="100%">
  <figcaption><strong>Figure 18.</strong> Pose transfer from VISOR (SensoConfig) to KR via CBun/Telegram.</figcaption>
</figure>

### 9.2 Bending Sequences
Multi‑station bends (90°, 135°, flatten) with PLC **open‑height triggers** to command **gripper release** and **safe re‑grasp**.

### 9.3 Storage Drawer Handling
Handle approach, **100° turn** to unlock, slide, place part, close/lock. Drawer markers restore exact pose for reliable placement.

<figure>
  <img src="/figures/shelf-control/open-drawer.jpeg" alt="Drawer operation" width="100%">
  <figcaption><strong>Figure 19.</strong> Automated drawer opening and placement sequence.</figcaption>
</figure>

---

## 10. Results

### 10.1 Calibration
- **Automated hand–eye calibration** completes in **≈98 s** and yields stable intrinsics/extrinsics (example outputs recorded for traceability).  
- The digital twin and real system frames align sufficiently for **first‑try grasp success** and **repeatable drawer placements**.

<figure>
  <img src="/figures/001calibration/hand-eye_parameters.PNG" alt="Hand-eye outputs" width="100%">
  <figcaption><strong>Figure 20.</strong> Hand–eye solution parameters (excerpt).</figcaption>
</figure>

### 10.2 Inspection Assessment
Representative overlays (bending operations 1, 2, 5, 6) show **robust edge localization** and **stable angle estimates** under reflective surfaces, enabling **closed‑loop correction** and **reject handling**.

<figure>
  <img src="/figures/008_inspection/inspection_6_overlay_cleanup.png" alt="Inspection overlay" width="100%">
  <figcaption><strong>Figure 21.</strong> Inspection overlays confirming angle measurement robustness.</figcaption>
</figure>

---

## 11. Discussion

- **Flexibility:** The 7‑axis cobot with a quick‑change gripper and marker‑based localization adapts to **multiple part variants** without re‑fixturing.  
- **Reliability:** Marker frames and on‑demand **auto‑calibration** mitigate drift from temperature/vibration.  
- **Safety:** Virtual zones and conservative braking profiles reduce risk around the press brake while maintaining usable throughput.  
- **Digital Twin Value:** The web UI accelerates commissioning, shortens troubleshooting, and provides a foundation for **remote diagnostics** and **what‑if** simulation.

---

## 12. Conclusion

This master’s thesis delivers a **production‑oriented robotic workcell** for **automated sheet‑metal bending**. The integration of a **7‑axis cobot**, **dual vision sensors**, and **PLC‑driven orchestration**, coupled with a **web‑based digital twin**, demonstrates **repeatable handling**, **accurate bending**, and **closed‑loop quality control**. The approach is **replicable** and **scalable**, offering a practical path to Industry 4.0 adoption on legacy press brakes.

### Future Work
- **Controller Manager / Task Generator:** generate new robot programs from simulation and auto‑splice into teach‑pendant sequences.  
- **Richer Web UI:** add camera streams, authentication, mobile app, persistent storage.  
- **Adaptive Strategies:** use inspection outcomes for **automatic bend parameter tuning** and **learning‑based grasp planning**.  
- **Full Virtual Commissioning Loop:** add signal‑timing simulation for PLC/actuators to complement the digital twin.

---

## Appendix — Software & Environment

- **ROS:** Noetic Ninjemys  
- **Python:** 3.8.10; **C++:** C++11  
- **Gazebo (classic)**, **RViz**, **MoveIt**  
- **Node.js:** v18.14.0; **npm** 9.x; **roslibjs** 1.3.0; **ros3djs** 0.17.0; **visualization‑rwt** 18.2.0  
- **Dev OS:** Ubuntu 20.04.6 LTS (simulations) + Windows 10 (software)  
- **IDE:** VS Code (1.91); CBun dev in VS Code (1.85) within Ubuntu 18.04 Dev Containers  
``