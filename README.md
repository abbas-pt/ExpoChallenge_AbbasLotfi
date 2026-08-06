# ExpoChallenge_AbbasLotfi
# ♻️ ECO-SORT AI: Smart Waste Sorting Robot & Industrial Dashboard

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
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

```text
[ Camera / Video Feed ]
          │
          ▼
[ Frame Pre-processing & Letterboxing (640x640) ]
          │
          ▼
[ YOLO11n Detection + ByteTrack Object Tracking ]
          │
          ▼
[ 18 ──► 5 Class Mapping & Trigger-Line Filter ]
          │
          ▼
[ Priority Queue & Kinematic Math (X, Y, Z, θ, TTG) ]
          │
          ├─────────────────────────────────────────┐
          ▼                                         ▼
[ Serial JSON Link (Arduino / Robot Arm) ]   [ Gradio + PyWebview GUI (OEE, Bins, Logs) ]
```

---

## 🤖 AI Model & Training Configuration

The perception system uses **YOLO11n (nano)** trained on the public **TACO (Trash Annotations in Context)** dataset.

### Why YOLO11n?
1. **Low Latency:** Crucial for fast-moving conveyor belts where detection must run within milliseconds.
2. **CPU Deployment:** Allows the dashboard to run smoothly on standard laptops without requiring a dedicated CUDA GPU.
3. **Small Footprint:** Keeps desktop installer sizes minimal and enables fast app startup.

### Training Parameters (Google Colab Run)
* **Epochs:** 100 (Early stopping patience: 15)
* **Image Size:** 640x640 | **Batch Size:** 16
* **Augmentation Pipeline:** Mosaic (1.0), MixUp (0.15), Copy-Paste (0.10), Rotation (+/- 15 deg), Perspective distortion (0.0005), HSV Jitter, and Random Erasing (0.40).

---

## 🔌 Serial Communication Protocol (Arduino API)

Commands are transmitted over USB Serial at **9600 Baud** as newline-terminated JSON payloads.

### Sample `PICK` Payload
When an item crosses the virtual trigger line, the dashboard emits:

```json
{
  "cmd": "PICK",
  "cls": "Plastic",
  "x": -42.5,
  "y": 187.3,
  "z": -150.0,
  "force": 50,
  "theta": 12.4,
  "ttg_ms": 1248,
  "ts": 1730000000
}
```

### Protocol Fields

| Field | Type | Description |
| :--- | :--- | :--- |
| `cmd` | `String` | Command type (`PICK`, `STOP_CONVEYOR`, `START_CONVEYOR`, `EMERGENCY_STOP`) |
| `cls` | `String` | Operational category (*Glass, Metal, Paper, Plastic, Waste*) |
| `x`, `y`, `z` | `Float` | Target coordinates in millimeters relative to the arm's base |
| `force` | `Int` | Gripper force in Newtons (e.g., Glass: 20N, Paper: 85N) |
| `theta` | `Float` | Gripper orientation angle in degrees |
| `ttg_ms` | `Int` | Time-to-Grab: delay in ms before object reaches pick point |
| `ts` | `Int` | UNIX Timestamp of command generation |

---

## 📊 18-to-5 Class Mapping Reference

| Raw TACO Class (18) | Operational Category (5) | Priority | Grip Force |
| :--- | :--- | :--- | :--- |
| Aluminium foil, Can, Pop tab | **Metal** | 1 (Highest) | 70 N |
| Bottle cap, Bottle, Lid, Other plastic, Plastic bag, Plastic container, Straw | **Plastic** | 2 | 50 N |
| Broken glass | **Glass** | 3 | 20 N |
| Carton, Cup, Paper | **Paper** | 4 | 85 N |
| Cigarette, Other litter, Styrofoam piece, Unlabeled litter | **Waste** | 5 (Lowest) | 60 N |

---

## ⚙️ Configuration & Calibration Variables

Key physical constants live at the top of `app.py` for easy benchtop setup:

```python
TRIGGER_LINE_RATIO = 0.50        # Vertical trigger position (50% frame height)
TRIGGER_TOLERANCE  = 25          # Acceptance band around trigger line (pixels)
SCALE_FACTOR_MM    = 1.5         # Pixel-to-mm scaling factor
CONVEYOR_SPEED_MM_S= 150.0       # Belt velocity (mm/s)
GRASPING_ZONE_Y_MM = 600.0       # Distance from trigger line to physical gripper
BIN_CAPACITIES     = 10          # Units per category before conveyor auto-halt
```

---

## 🚀 Getting Started (Developer Setup)

### Prerequisites
* Python 3.10+
* OpenCV & PyTorch
* Arduino connected via USB (Optional: App automatically falls back to offline mode)

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/abbas-pt/ExpoChallenge_AbbasLotfi.git
   cd ExpoChallenge_AbbasLotfi
   ```

2. Create and activate a virtual environment:
   ```bash
   python -m venv venv
   # On Windows:
   venv\Scripts\activate
   # On macOS/Linux:
   source venv/bin/activate
   ```

3. Install dependencies:
   ```bash
   pip install gradio ultralytics torch opencv-python numpy pandas pyserial pywebview pyyaml
   ```

4. Place your model weights (`best_abbas.pt`) and dataset spec (`data.yaml`) in the project root.

5. Run the application:
   ```bash
   python app.py
   ```

---

## 🛠️ Project Scope & Boundaries

* **Implemented:** Full vision pipeline, YOLO11n model, ByteTrack tracking, 18-to-5 class mapping, kinematics engine, JSON serial protocol, Gradio/PyWebview dashboard, OEE metrics, E-Stop logic, and PyInstaller bundling.
* **Out of Scope (Integration Target):** The physical 6-DOF/SCARA robotic arm hardware and its internal Inverse Kinematics motor firmware. The dashboard delivers precise JSON targets designed to be parsed by any downstream controller.

---

## 👨‍💻 Team & Credits

Developed for the **Innoverse America** competition:
* **Sina Niknejadi**
* **Reza Esmaeili**
* **Ara Tajaddini**
* **Abbas Lotfi**
