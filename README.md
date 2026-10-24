# 🚗 Road Sign Recognition Car Using YOLO and ESP32

A real-time road-sign recognition and autonomous movement project that combines computer vision, embedded systems, and sensor-based control. The system uses an AI-Thinker ESP32-CAM to capture road-sign images, a laptop running a trained YOLO classification model to recognize signs, and an ESP32 DevKit V1 with an L298N motor driver to control the car.

## ✨ Features

- 📷 Image capture using AI-Thinker ESP32-CAM
- 🧠 Road-sign classification using a trained YOLO model
- 📡 HTTP communication between the laptop and motor-control ESP32
- 🚗 Differential motor control using an L298N driver
- 🛑 IR sensor detection of black tape for stopping at designated points
- ↪️ Directional movement based on recognized road signs

## 🛠️ Hardware Requirements

- AI-Thinker ESP32-CAM with OV2640 camera
- ESP32 DevKit V1
- L298N dual H-bridge motor driver
- DC geared motors and robot chassis
- IR sensor module
- Suitable power supply and connecting wires
- Laptop or computer for YOLO inference

## 💻 Software Requirements

- Ubuntu Linux
- Python 3.12
- Ultralytics YOLO
- PyTorch
- OpenCV
- Arduino IDE
- ESP32 Arduino board support

## 🧠 Road-Sign Classes

The trained classification model recognizes five classes:

| Class | Purpose |
|---|---|
| `GO` | Move forward |
| `LEFT` | Turn left |
| `RIGHT` | Turn right |
| `STOP` | Stop the car |
| `WAIT` | Waiting state, as handled by the control logic |

## 📁 Project Structure

```text
road_sign_car/
├── dataset/                         # Original road-sign images
├── dataset_split/
│   ├── train/                        # Training images
│   └── val/                          # Validation images
├── runs/
│   └── classify/
│       └── train-2/
│           └── weights/
│               ├── best.pt          # Trained classification model
│               └── last.pt          # Last training checkpoint
├── fnlcam/
│   └── fnlcam.ino                   # ESP32-CAM firmware
├── fnlesp32/
│   └── fnlesp32.ino                 # ESP32 motor-controller firmware
├── yolo_computer_run.py             # Main recognition and control script
├── yolo.py                          # Optional prediction/display test
├── test.py                          # Optional image-collection script
├── yolo26n-cls.pt                   # Pretrained classification weights
└── README.md
```

*The structure above describes the main files; retain any additional files present in your actual project folder.*

## ⚙️ Working Principle

1. The ESP32-CAM captures an image of the road sign through its HTTP capture endpoint.
2. The laptop retrieves the image and processes it using the trained YOLO classification model.
3. The model predicts the road-sign class.
4. The Python controller sends the corresponding command to the motor-control ESP32 over HTTP.
5. The ESP32 controls the motors through the L298N driver.
6. The IR sensor detects black tape placed at designated points and provides the stop signal according to the motor-control logic.

### System Architecture

```text
       Road Sign
           │
           ▼
     ESP32-CAM
     (Image Capture)
           │
           ▼
        Laptop
   YOLO Classification
           │
           ▼
    HTTP Motor Command
           │
           ▼
      ESP32 DevKit
           │
           ▼
       L298N Driver
           │
           ▼
       DC Motors

 IR Sensor ──► ESP32 DevKit
 (Black Tape)   (Stop Control)
```

## 📊 Dataset

The project uses a custom road-sign image dataset organized by class.

- `dataset/` stores the original collected images.
- `dataset_split/train/` contains images used for training.
- `dataset_split/val/` contains images used for validation.
- Reported dataset size: **1,816 training images and 681 validation images** across five classes.

The dataset can be expanded to improve recognition performance under different lighting conditions, viewing angles, and distances.

## 🚀 Setup and Execution

### 1. Clone or Copy the Project

Place the project folder on your computer and open a terminal inside it.

### 2. Create a Python Environment

```bash
python3 -m venv venv
source venv/bin/activate
```

Install the Python dependencies:

```bash
pip install ultralytics opencv-python
```

Install a compatible PyTorch build for your system if it is not already installed. GPU acceleration requires a compatible NVIDIA driver and CUDA-enabled PyTorch installation.

### 3. Configure the ESP32-CAM

1. Open `fnlcam/fnlcam.ino` in Arduino IDE.
2. Configure the Wi-Fi credentials.
3. Select the appropriate ESP32-CAM board and upload the firmware.
4. Note the camera's IP address.

### 4. Configure the Motor Controller

1. Open `fnlesp32/fnlesp32.ino`.
2. Configure the Wi-Fi credentials and motor-control settings.
3. Check the L298N wiring and IR sensor connection.
4. Upload the firmware to the ESP32 DevKit V1.
5. Note the motor controller's IP address.

### 5. Configure the Python Script

Open `yolo_computer_run.py` and verify:

- The ESP32-CAM IP address
- The motor-controller IP address
- The trained model path
- The camera capture endpoint
- The confidence and prediction settings

The trained model path is:

```text
runs/classify/train-2/weights/best.pt
```

### 6. Run the Recognition System

Activate the virtual environment and execute:

```bash
python yolo_computer_run.py
```

Ensure that both ESP32 devices and the laptop are connected to the appropriate network.

## 🔌 Motor Control and IR Detection

The motor controller uses an L298N driver with the following GPIO assignments:

| Function | ESP32 GPIO |
|---|---:|
| Left motor enable (ENA) | 25 |
| Left motor input 1 (IN1) | 26 |
| Left motor input 2 (IN2) | 27 |
| Right motor enable (ENB) | 13 |
| Right motor input 3 (IN3) | 14 |
| Right motor input 4 (IN4) | 12 |
| IR sensor output | 34 |

Verify these assignments against the firmware before wiring.

The intended movement logic is:

- **GO:** Move forward until the designated black tape is detected.
- **LEFT / RIGHT:** Perform the configured turn, then continue forward until the next black tape is detected.
- **STOP:** Stop the motors.
- **WAIT:** Follow the behavior implemented in the current control script.

The exact command-handling and interruption behavior depends on the uploaded firmware.

## 📦 Model Files

| File | Purpose |
|---|---|
| `runs/classify/train-2/weights/best.pt` | Main trained classification model |
| `runs/classify/train-2/weights/last.pt` | Last training checkpoint |
| `runs/classify/precision/weights/best.pt` | Alternative trained model |
| `yolo26n-cls.pt` | Pretrained classification weights |
| `yolo26s-cls.pt` | Pretrained classification weights |
| `yolo26n.pt` | Base object-detection weights; not the main classification model |

Use the trained `best.pt` model for inference rather than replacing it with a pretrained model.

## 🔧 Troubleshooting

- **Camera image unavailable:** Check the ESP32-CAM IP, Wi-Fi connection, and `/capture` endpoint.
- **Motor commands not working:** Verify the motor ESP32 IP, HTTP endpoint, Wi-Fi connectivity, and motor power supply.
- **Incorrect road-sign predictions:** Check the model path, class-folder names, lighting conditions, and image quality.
- **Car does not stop at black tape:** Verify the IR sensor output, GPIO connection, sensor polarity, and firmware logic.
- **Slow inference:** Check whether PyTorch is using the NVIDIA GPU and whether the model fits within available GPU memory.

## 🔮 Future Improvements

- Improve recognition accuracy with additional training images.
- Add obstacle detection and avoidance.
- Improve turning accuracy using encoder or other feedback-based control.
- Optimize inference speed and reduce false predictions.
- Integrate the complete system into a more autonomous navigation platform.

## 👥 Contributors

Add the names of the project contributors here.

## 📄 License

Specify the license under which this project is distributed.
