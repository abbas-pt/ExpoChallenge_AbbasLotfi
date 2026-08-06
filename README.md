# ExpoChallenge_AbbasLotfi
# ♻️ ECO-SORT AI: Smart Waste Sorting Robot & Industrial Dashboard

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![YOLO11n](https://img.shields.io/badge/AI_Model-YOLO11n-green.svg)](https://github.com/ultralytics/ultralytics)
[![GUI](https://img.shields.io/badge/UI-Gradio_%2B_PyWebview-orange.svg)](https://gradio.app/)
[![Hardware](https://img.shields.io/badge/Hardware-Arduino_Serial_Bridge-red.svg)](https://www.arduino.cc/)
[![Platform](https://img.shields.io/badge/Platform-Windows_%7C_macOS-lightgrey.svg)]()

**ECO-SORT AI** is an industrial-grade vision, control, and monitoring dashboard developed for the **Smart Waste Sorting Robot** project (submitted to *Innoverse America*). 

The system acts as the "brain and cockpit" of an automated waste sorting cell. It captures a camera feed, detects and tracks 18 distinct litter classes frame-by-frame, maps them into 5 operational categories, calculates kinematic parameters (X, Y, Z, Angle, Time-to-Grab), and transmits JSON commands over USB serial to an Arduino-based robot arm controller—all while calculating real-time industrial KPIs (**OEE**).

---

## 📸 Core Capabilities

* **Real-time AI Vision & Tracking:** Custom-trained **YOLO11n** model integrated with **ByteTrack** for persistent multi-object tracking.
* **18-to-5 Class Mapping:** Collapses 18 fine-grained TACO dataset classes into 5 action-oriented categories (*Metal, Plastic, Glass, Paper, Waste*) to drive distinct gripper forces and bin routing.
* **Environmental Priority Queue:** Resolves conflicts when multiple objects cross the trigger line simultaneously by ranking recoverable economic/environmental value first (*Metal > Plastic > Glass > Paper > Waste*).
* **Kinematics & Orientation Engine:** Computes real-world X/Y/Z offsets, object orientation angle ($\theta$) via Otsu-contour analysis, and remaining travel time (`ttg_ms`).
* **Hardware Interlocks & Safety:** Dedicated E-Stop workflow, automatic conveyor halt on full bins, and graceful offline fallback if hardware is disconnected.
* **OEE Dashboard & Benchmarking:** Real-time Overall Equipment Effectiveness calculation ($Availability \times Performance \times Quality$) and an offline mAP@50 validator.

---

## 🏗️ System Architecture
