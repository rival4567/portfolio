---
title: "Technical Description of an Industrial Digital Twin Using NVIDIA Isaac Sim and ROS 2"
date: 2026-02-18
draft: false
author: "Shivam Shivam"
tags:
  - Digital Twin
  - Robotics
  - Isaac Sim
  - ROS 2
  - Motion Planning
  - Perception
image: /images/blogs/digital-twin/dt-photo.png
description: "A detailed technical overview of a hybrid physical–simulation robotics platform integrating industrial manipulators, multi-modal sensing, and GPU-accelerated planning."
---

> :warning: **Note:** This digital twin is complete and the platform is being used to develop and test new ideas.
## Abstract

This article presents the technical description of a laboratory‑validated **industrial digital twin** that integrates physical robotic hardware with a physics‑accurate simulation built in **NVIDIA Isaac Sim 5.0**. The system synchronizes two industrial manipulators, grippers, multi-modal visual sensing, and force-torque data through **ROS 2**, enabling virtual commissioning, trajectory validation, and perception testing within a unified software and hardware pipeline.

---

## 1. Introduction

Digital twins provide a high-fidelity intermediary between simulation and physical robotic systems. The setup described here couples two industrial robots with an Isaac‑Sim‑based virtual environment using ROS 2 middleware, providing a reproducible platform for manipulator control, perception experiments, and GPU‑accelerated path planning.

---

## 2. Hardware Configuration

The physical system includes industrial manipulators, adaptive grippers, stereo and wrist‑mounted cameras, teleoperation controllers, and a host workstation.

### Table 1 — Hardware Components
| Component            | Description                                                                                                           | Qty |
|---------------------|-----------------------------------------------------------------------------------------------------------------------|----:|
| Industrial robots   | UR10 (Universal Robots), HC10DT (Yaskawa Motoman), 6‑axis manipulators                                                | 2   |
| Grippers            | Robotiq 2f‑140 and 2f‑85                                                                                               | 2   |
| RGB cameras         | Robotiq wrist cameras                                                                                                 | 2   |
| Force‑torque sensor | Robotiq FT300‑S                                                                                                       | 1   |
| Stereo cameras      | Stereolabs ZED 2i                                                                                                     | 2   |
| Joysticks           | Teleoperation controllers                                                                                             | 2   |
| Compute workstation | Ubuntu PC running Isaac Sim and ROS 2                                                                                 | 1   |
---

## 3. Software Stack

The system uses:

- **Isaac Sim 5.0** for digital-twin rendering and physics.
- **ROS 2 Humble** for hardware abstraction and message passing.
- **MoveIt Servo** for teleoperation.
- **cuRobo + Isaac ROS Nvblox** for GPU‑accelerated collision-free trajectory generation.
- **YOLO, RF‑DETR, DOPE, FoundationPose** for perception and pose estimation.  
---

## 4. Digital Twin Model

URDF models of robots and grippers are converted into **USD** assets inside Isaac Sim to achieve high spatial fidelity. Camera intrinsics and extrinsics are aligned using **hand‑eye calibration**, allowing the digital twin to produce synthetic sensor outputs that match real camera views.

![Physical-setup](/images/blogs/digital-twin/physical-setup.png)  
---

## 5. Methodology

### Stage 1 — Data Acquisition  
Hardware is connected to ROS 2, with synchronized acquisition of RGB, depth, force‑torque data, and robot joint states. Hand‑eye calibration yields transformations between cameras and the TCP.

### Stage 2 — Twin Construction  
URDFs are converted to USD; an **XRDF** description enables cuRobo‑based GPU trajectory generation inside Isaac Sim. Identical ROS namespaces are used for both environments.

### Stage 3 — Teleoperation & Validation  
MoveIt Servo supports real‑time joystick control. Robot motion and camera images are compared across physical and simulated environments to validate digital twin fidelity.

---

## 6. System Performance

Key real‑world performance metrics include:

- Robot controllers communicate with ROS at **~100 Hz**.  
- ZED 2i stereo cameras operate at **1080p @ 30 FPS** for RGB–depth generation.  
- Isaac Sim runs at **~30 FPS** on workstation hardware.  
- cuRobo + Nvblox enables **real‑time collision‑free trajectories** even with close‑proximity manipulators.  
---

## 7. Conclusion

The digital twin infrastructure provides a robust foundation for industrial robotics research, enabling unified experimentation across perception, planning, and teleoperation. The architecture ensures reproducibility, safety, and reduced development time for advanced manipulation tasks.