---
title: "Teleoperation of Industrial Robotic Arms Using Joysticks in a Digital Twin Environment"
date: 2026-02-18
draft: false
author: "Shivam Shivam"
tags:
  - Teleoperation
  - MoveIt Servo
  - Digital Twin
  - Isaac Sim
  - ROS 2
  - Robotics
image: /images/blogs/digital-twin/xbox-addedbuttons.png
description: "A detailed overview of joystick-based teleoperation for dual-arm industrial robots using a synchronized digital twin in NVIDIA Isaac Sim."
toc:
---

## Abstract

Teleoperation is an essential capability for industrial research laboratories, enabling safe remote manipulation, rapid prototyping of task strategies, and controlled evaluation of robot behaviors. This article presents a joystick‑based teleoperation framework implemented for a dual‑arm setup consisting of a UR10 and HC10DT industrial manipulator, integrated with a high‑fidelity digital twin in **NVIDIA Isaac Sim 5.0**. The system leverages **MoveIt Servo**, **ROS 2**, and synchronized hardware–simulation interfaces to achieve real‑time teleoperation and trajectory mirroring between physical and simulated robots.

---

## 1. Introduction

Teleoperation provides an intuitive mechanism for human operators to interact with robotic systems, especially in early‑stage prototyping, safety‑critical environments, or tasks requiring fine manipulation. Within this digital‑twin architecture, teleoperation serves two key research functions:

1. **Validation of robot kinematics and collision modeling**  
2. **Evaluation of perception and planning modules under operator-guided motion**

By connecting hardware controllers to both physical robots and their digital replicas, the system supports real-time cross‑environment comparisons of trajectories, sensor observations, and joint‑space behavior.

---

## 2. System Overview

### 2.1 Physical Hardware

The teleoperation setup uses:

- **UR10** and **HC10DT** industrial robots  
- **Robotiq 2f‑140 and 2f‑85** grippers  
- **Two joysticks** for teleoperation command input  

These components interface with an Ubuntu workstation running both ROS 2 and Isaac Sim.

### 2.2 Simulation Environment

The digital twin is constructed in **Isaac Sim 5.0**, where robot models and their kinematic structures are imported as **USD assets**. Camera extrinsics, robot frames, and collision geometries are aligned to the real environment via ROS‑based calibration.

---

## 3. Teleoperation Framework

### 3.1 MoveIt Servo for Real‑Time Control

Joystick inputs are processed using **MoveIt Servo** (version 2.5.7), which supports real‑time Cartesian and joint‑space velocity control. This allows smooth, responsive manipulation of the robot end‑effector.
MoveIt Servo ensures:

- Low‑latency control loop execution  
- Real‑time velocity scaling  
- Singularities and joint‑limit avoidance  
- Compatibility with ROS 2 hardware interfaces  

### 3.2 Bi‑Directional Synchronization via ROS 2

All teleoperation commands flow through a universal ROS 2 namespace that is shared between:

- **Physical robots**
- **Isaac Sim digital twin**

This ensures that joystick motions generate **identical trajectories** in both environments, enabling:

- Direct trajectory comparison  
- Simulation-based safety validation  
- Parallel multi-robot experiments  

### 3.3 Collision-Free Planning Assistance

Although MoveIt Servo allows direct operator control, the system integrates **cuRobo with Isaac ROS Nvblox** to ensure that even under joystick teleoperation, movements remain collision-free.  
The GPU‑accelerated planner provides:

- Real‑time signed‑distance field (SDF) updates  
- On‑the‑fly trajectory validation  
- Safe bimanual operation, even when both robots move in close proximity  

![Digital-twin](/images/blogs/digital-twin/ur_motoman_joints.png)
---

## 4. Teleoperation Workflow

The teleoperation workflow consists of three phases:

### Phase 1 — Initialization

- The operator initializes both robots and sensors via ROS 2.  
- Isaac Sim loads the USD models and synchronizes TF frames with the physical setup.  
- Calibration ensures identical camera and robot poses in both environments.  

### Phase 2 — Live Teleoperation

- Joystick commands are translated to Cartesian velocity commands by MoveIt Servo.  
- Commands propagate to both the physical robot controllers and the digital twin via ROS topics.  
- Force‑torque and perception data can be visualized in RViz for feedback.  

### Phase 3 — Monitoring and Validation

- The operator observes end‑effector motion in both real and virtual environments.  
- Any discrepancies in motion, latency, or collision detection can be diagnosed via logs.  
- The system provides a safe sandbox for testing teleoperated dual‑arm tasks.  

---

## 5. Performance Characteristics

The system achieves:

- **~100 Hz hardware communication** between robot controllers and ROS PC  
- **30 FPS simulation performance** in Isaac Sim  
- **Real-time, collision-free trajectories** with cuRobo + Nvblox  
- Responsive joystick control of both arms, even in tight manipulation spaces  

These results demonstrate that joystick teleoperation can reliably drive both real and simulated robots under synchronized conditions with minimal latency.

---

## 6. Example Teleoperation Scenario

A typical experiment might involve:

1. Using the joystick to position the UR10 near an object.  
2. Observing how the digital twin replicates the motion exactly.  
3. Monitoring the stereo camera feed and TF alignment to verify perception accuracy.  
4. Engaging the gripper while validating that both the simulated and real gripper produce consistent state feedback.  

Such scenarios are essential for debugging perception-driven grasping pipelines and testing operator‑assisted manipulation strategies.  

---

## 7. Conclusion

Joystick‑based teleoperation within this digital‑twin architecture provides a powerful and safe framework for controlling industrial manipulators. The combination of MoveIt Servo, ROS 2 synchronization, and GPU‑accelerated collision avoidance ensures high responsiveness, operational safety, and reproducibility across both simulated and real‑world executions. This teleoperation capability is a key enabler for validating robotic setups, training operators, and accelerating development of intelligent manipulation strategies.