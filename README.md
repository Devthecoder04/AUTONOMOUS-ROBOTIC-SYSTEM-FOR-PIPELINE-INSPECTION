# 🤖 Autonomous Robotic System for Pipeline Inspection

An AI-based, low-cost autonomous robot that crawls through pipelines, detects internal **cracks** in real time with a CNN running on a Raspberry Pi (TensorFlow Lite), and reports status and alerts wirelessly to an operator dashboard.


## 📌 Overview

Manual pipeline inspection is unsafe, slow and inconsistent, as it requires people to enter confined, hazardous spaces. This project combines **computer vision, deep learning and embedded robotics** to automate it:

- A camera-equipped wheeled robot captures the pipe's inner surface.
- A custom **CNN** classifies each frame as `crack` / `no crack`.
- The model is converted to **TensorFlow Lite** for real-time inference on a Raspberry Pi 4.
- **Ultrasonic sensors** provide obstacle detection and safe navigation.
- An **ESP32** provides wireless communication to a web dashboard with auto/manual modes and live sensor data.
- Detected crack images are saved with timestamps for maintenance planning.

## ✨ Features

- Real-time edge inference (~65 ms/frame on Raspberry Pi 4, about 15 FPS)
- Lightweight custom CNN (1,568,321 parameters, ~5–6 MB in TFLite)
- Auto and manual driving modes, with an emergency stop
- Web dashboard with live front/top distance readings, pipe-diameter setting and drive controls
- Timestamped crack image logging

## 🏗️ System Architecture

| Block | Component | Role |
|---|---|---|
| Vision / AI | Raspberry Pi 4 + USB camera | Frame capture, TFLite crack classification, crack display |
| Control / Comms | ESP32 | Sensor reading, motor control, Wi-Fi link to the dashboard |
| Sensing | 2× ultrasonic sensors (front, top) | Obstacle detection and distance measurement |
| Actuation | L298N motor driver + 2 DC geared motors + wheels | Locomotion, PWM speed control, differential steering |
| Power | Li-ion pack, DC-DC buck converter, power bank | 12 V for motors, 5 V for logic, separate supply for the Pi |

See [`hardware/`](hardware/) for wiring and pin details.

## 🧠 Model

| Item | Value |
|---|---|
| Input | 128 × 128 × 3 RGB, rescaled to [0, 1] |
| Backbone | 4 × (Conv2D 3×3 + ReLU + MaxPool 2×2) with 32 → 64 → 128 → 256 filters |
| Head | Flatten (9,216) → Dropout 0.5 → Dense 128 (ReLU) → Dropout 0.3 → Dense 1 (sigmoid) |
| Loss / Optimizer | Binary cross-entropy / Adam (lr = 5e-4) |
| Callbacks | EarlyStopping (patience 5), ModelCheckpoint, ReduceLROnPlateau (×0.3, patience 3) |
| Augmentation | Rotation ±20°, zoom 0.2, horizontal flip, shear 0.1 (training set only) |
| Batch size / Epochs | 32 / max 15 |
| Parameters | 1,568,321 (all trainable) |

**Dataset:** public sewer/pipeline crack images from Kaggle, in `train/` and `valid/` folders with `crack` and `no_crack` classes. See [`dataset/README.md`](dataset/README.md).

## 📊 Results

| Metric | Value |
|---|---|
| Training accuracy | ≈ 99.06 % |
| Validation accuracy | ≈ 99.04 % |
| Inference time (Raspberry Pi 4, TFLite) | ≈ 65 ms / frame |

> **Note:** the comparison chart in the report (Raptor CNN, YOLO-DeepSort, Faster R-CNN, SSD, Mask R-CNN, VGG19) uses figures reported in other papers on different datasets and tasks, so it is indicative rather than a like-for-like benchmark.

Training curves and screenshots go in [`results/`](results/).

## 📁 Repository Structure

```
pipeline-inspection-robot/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── docs/               # Diagrams and images
├── dataset/            # Dataset instructions (data itself is not committed)
├── ml/                 # Model definition, training, TFLite conversion, Pi inference
├── models/             # Trained .h5 / .tflite files
├── hardware/           # Wiring, BOM, motor-control example
├── firmware/esp32/     # ESP32 firmware (sensors, motors, web server)
├── dashboard/          # Operator dashboard (web UI)
└── results/            # Training curves, detection screenshots
```

## 🚀 Getting Started

### 1. Clone and install
```bash
git clone https://github.com/<your-username>/pipeline-inspection-robot.git
cd pipeline-inspection-robot
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Prepare the dataset
Download the Kaggle dataset and arrange it as described in [`dataset/README.md`](dataset/README.md).

### 3. Train
```bash
python ml/train.py --data dataset --epochs 15 --batch-size 32
```

### 4. Convert to TensorFlow Lite
```bash
python ml/convert_tflite.py --model models/crack_cnn.h5 --out models/crack_cnn.tflite
```

### 5. Run on the Raspberry Pi
```bash
pip install tflite-runtime opencv-python numpy
python ml/infer_pi.py --model models/crack_cnn.tflite --camera 0
```
Crack frames are saved to `detections/` with timestamps.

### 6. Flash the ESP32 and open the dashboard
See [`firmware/esp32/README.md`](firmware/esp32/README.md) and [`dashboard/README.md`](dashboard/README.md).

## 🔮 Future Scope

- NVIDIA Jetson Nano with YOLOv8 for bounding-box localisation
- GPS module (NEO-6M) to geotag cracks
- Wide-angle Raspberry Pi Camera v3 (120° FOV)
- IMU (MPU-6050) for tilt and orientation
- Li-ion pack with USB-C charging and BMS
- Cloud-based predictive maintenance

## 👤 Author

**Devaprakash**: [GitHub](https://github.com/<your-username>)

Developed as a B.Tech final-year project at Jyothi Engineering College (Autonomous), Cheruthuruthy.

## 📚 Citation

```bibtex
@misc{pipeline_inspection_2026,
  title  = {Autonomous Robotic System for Pipeline Inspection},
  author = {Devaprakash},
  year   = {2026},
  note   = {B.Tech project, Jyothi Engineering College, APJ Abdul Kalam Technological University}
}
```

## 📄 License

Released under the [MIT License](LICENSE).# AUTONOMOUS-ROBOTIC-SYSTEM-FOR-PIPELINE-INSPECTION
