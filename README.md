# JANATICS-Automated-Sorting-Challenge-PLC-Pneumatic-Control-System

## 📌 Project Abstract
This repository details the design, hardware architecture, and programmable logic control strategy for an autonomous Conveyor-Based Material Sorting System. Built as a laboratory-scale industrial model during my B.Tech in Robotics and Automation Engineering at SIT Pune, this system detects, classifies, and mechanically diverts metallic and non-metallic objects into designated bins. 

The core intelligence is powered by a Siemens SIMATIC S7-1200 PLC, which processes sequential data from photoelectric, inductive, and capacitive sensors to trigger precise pneumatic actuation using JANATICS components.

---

## 📑 Table of Contents
1. [System Architecture](#-system-architecture)
2. [Pneumatic Circuit Design](#-pneumatic-circuit-design)
3. [Control Strategy & Ladder Logic](#-control-strategy--ladder-logic)
4. [Operation Flow](#-operation-flow)
5. [Future Enhancements (AI & SCADA)](#-future-enhancements)

---

## 🏗️ System Architecture

### 1. Programmable Logic Controller (PLC)
* **Model:** Siemens SIMATIC S7-1200 CPU 1214C DC/DC/DC
* **Why this model?** The transistor outputs provide sub-millisecond switching response times, which is critical for the highly precise, timer-triggered solenoid valve actuation required in high-speed sorting. 
* **Environment:** Programmed using Ladder Diagram (LAD) within the **TIA Portal V17** framework (IEC 61131-3 compliant).

### 2. Sensor Modalities
The system utilizes a three-stage, false-positive-resistant detection chain:
* **Stage 1 (Presence): Omron E3JK-TP12 2M (Through-Beam)** * High immunity to ambient light and surface reflectivity. Triggers the initial inspection halt.
* **Stage 2 (Metal Detection): Omron E2B-M18KN10-WP-C1 2M (Inductive)**
  * Detects eddy currents. Strictly specific to electrically conductive materials, acting as a binary classifier.
* **Stage 3 (Non-Metal Detection): Pepperl+Fuchs CBB4-12GH70-E2 (Capacitive)**
  * Detects changes in dielectric capacitance. Identifies plastics, woods, etc., that bypass the inductive zone.

### 3. Motor Drive & Fail-Safe Relay
* **Motor:** 12V DC Geared Motor for high-torque, low-speed belt driving.
* **Relay:** A 24V DC electromagnetic relay isolates the PLC from the motor's higher current. 
* **Safety:** Programmed with Normally-Closed (NC) logic. If a PLC fault occurs, the relay de-energizes, halting the conveyor safely.

---

## 💨 Pneumatic Circuit Design
The mechanical diversion is handled by a custom pneumatic circuit utilizing **JANATICS** hardware operating at a regulated 3-5 bar pressure.

1. **Air Preparation:** A 3-stage FRL (Filter-Regulator-Lubricator) unit removes particulates/moisture and introduces pneumatic oil to reduce spool friction.
2. **Distribution:** A manifold block splits the regulated air to two separate actuation channels.
3. **Directional Control Valves (DCVs):** 5/2 spring-return, solenoid-actuated DCVs control the airflow. The 24V solenoids are driven directly by the PLC's transistor outputs (with surge-suppression diodes to absorb inductive kickback).
4. **Actuators:** Double-acting cylinders provide positive force on both the extend and retract strokes, ensuring the diverter paddles reset reliably even if obstructed.
5. **Flow Control:** Meter-out flow control valves throttle the exhaust air, preventing mechanical shock and ensuring smooth cylinder extension.
<img width="1600" height="764" alt="image" src="https://github.com/user-attachments/assets/c7141b85-82a8-4145-a751-15106fb68381" />

---

## 🧠 Control Strategy & Ladder Logic
The control program consists of 15 networks programmed in TIA Portal, designed with robust industrial logic patterns:

* **Edge Detection (R_TRIG):** Prevents the system from continuously re-triggering while the through-beam sensor is blocked, ensuring only one inspection cycle initiates per object.
* **Retentive State Management (SET/RESET):** Latches the conveyor stop command and sensor detections so the system remembers the object's classification as it travels down the belt.
* **Staggered Timing (TON Pairs):** * **Metallic Station:** 3,700 ms transport delay to actuation → 3,800 ms auto-reset (100 ms differential ensures full cylinder extension).
  * **Non-Metallic Station:** 2,300 ms transport delay to actuation → 2,400 ms auto-reset.

---

## ⚙️ Operation Flow

1. **Detection & Inspection:** An object interrupts the through-beam sensor. The conveyor halts for exactly 2 seconds to allow the proximity sensors to stabilize and read the material composition.
2. **Classification:** The PLC registers the material type based on the inductive/capacitive sensor input and sets the corresponding memory latch.
3. **Transport:** The conveyor resumes. The PLC begins the specific countdown timer for the identified material.
4. **Ejection & Auto-Reset:** Once the object reaches the correct station, the pneumatic cylinder extends, sweeping the object into the bin. The system automatically clears its latches and re-arms for the next object.

<img width="2485" height="1398" alt="image" src="https://github.com/user-attachments/assets/f2d0cb2a-bca0-4454-908a-3f690d7f47b0" />


https://github.com/user-attachments/assets/d033a459-f77f-4d12-b25b-1ee7e7d8a68f




---

## 🚀 Future Enhancements

Building upon this electromechanical foundation, the next phase of this project will integrate advanced software and telemetry layers:

* **Computer Vision Semantic Sorting:** Upgrading from binary composition sorting to semantic object classification. This will utilize **YOLOv8** and OpenCV, optimized to run on a **Blackwell architecture RTX 5060 GPU utilizing CUDA 12.8** for ultra-low latency, real-time inference.
* **SCADA Integration:** Leveraging the S7-1200’s PROFINET RJ45 interface to transmit telemetry data (sort counts, cycle times, pneumatic pressure faults) to a centralized SCADA dashboard.
* **HMI Deployment:** Integrating a Siemens KTP700 Basic Panel to allow operators to adjust timer setpoints and monitor system diagnostics without needing to reprogram the PLC.
