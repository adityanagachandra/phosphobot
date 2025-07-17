# Phosphobot Local Deployment Guide

**Phosphobot** is a comprehensive robotics framework for controlling AI robots, recording datasets, training Vision Language Action (VLA) models, and implementing teleoperation. This guide covers everything you need to deploy and customize your own Phosphobot server locally.

## Table of Contents

1. [Quick Start](#quick-start)
2. [System Requirements](#system-requirements)
3. [Installation Methods](#installation-methods)
4. [Server Configuration](#server-configuration)
5. [Supported Hardware](#supported-hardware)
6. [API Reference](#api-reference)
7. [Custom Integrations](#custom-integrations)
8. [Examples and Use Cases](#examples-and-use-cases)
9. [Development Setup](#development-setup)
10. [Troubleshooting](#troubleshooting)

## Quick Start

### 1. Automated Installation (Recommended)

```bash
# One-line installation
curl -fsSL https://raw.githubusercontent.com/phospho-app/phosphobot/main/install.sh | bash

# Start the server
phosphobot run

# Access the dashboard
open http://localhost:80
```

### 2. Manual Installation from Source

```bash
# Prerequisites: Install uv and npm
curl -LsSf https://astral.sh/uv/install.sh | sh
# Install Node.js from https://nodejs.org/

# Clone the repository
git clone https://github.com/phospho-app/phosphobot.git
cd phosphobot

# Build and run (macOS/Linux)
make

# For Windows (run commands directly)
cd dashboard && npm i && npm run build && mkdir -p ../phosphobot/resources/dist/ && cp -r dist/* ../phosphobot/resources/dist/
cd ../phosphobot && uv run --python 3.10 phosphobot run --simulation=headless
```

## System Requirements

### Minimum Requirements
- **OS**: Linux (Ubuntu 18.04+), macOS (10.15+), or Windows 10+
- **Python**: 3.10 or higher
- **Node.js**: 18+ (for dashboard)
- **Memory**: 4GB RAM minimum, 8GB recommended
- **Storage**: 2GB available space
- **Network**: Internet connection for model downloads

### Hardware Support
- **Cameras**: USB webcams, RealSense cameras
- **Robots**: SO-100, SO-101, Koch v1.1, WX-250, AgileX Piper, Unitree Go2, LeCabot
- **Controllers**: Keyboard, leader arms, Meta Quest VR headsets

### GPU Support (Optional)
- **NVIDIA GPU** with CUDA 12.4+ for AI model training/inference
- **8GB+ VRAM** recommended for VLA models

## Installation Methods

### Method 1: Package Manager Installation

**macOS (Homebrew):**
```bash
brew install phosphobot
phosphobot run
```

**Linux (APT):**
```bash
sudo apt update && sudo apt install phosphobot
phosphobot run
```

**Windows (PowerShell):**
```powershell
powershell -ExecutionPolicy ByPass -Command "irm https://raw.githubusercontent.com/phospho-app/phosphobot/main/install.ps1 | iex"
phosphobot run
```

### Method 2: Python Package Installation

```bash
pip install phosphobot
phosphobot run
```

### Method 3: Development Installation

```bash
git clone https://github.com/phospho-app/phosphobot.git
cd phosphobot
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv pip install -e .
phosphobot run --simulation=gui --port=8080 --host=127.0.0.1 --no-telemetry
```

## Server Configuration

### Configuration File

Phosphobot uses a YAML configuration file located at `~/.phosphobot/config.yaml`:

```yaml
# Server settings
port: 80
host: "0.0.0.0"
telemetry: false

# Hardware settings
enable_realsense: true
enable_cameras: true
main_camera_id: 0

# Simulation settings
sim_mode: "headless"  # Options: "headless", "gui"
only_simulation: false
simulate_cameras: false

# Recording defaults
default_dataset_name: "my_dataset"
default_freq: 30
default_episode_format: "lerobot_v2.1"
default_video_codec: "avc1"
default_video_size: [320, 240]
default_task_instruction: "Pick and place task"

# Camera configuration
default_cameras_to_disable: []  # List of camera IDs to disable
default_cameras_to_record: null  # null = all cameras, or specify list [0, 1]
```

### Command Line Options

```bash
# Basic options
phosphobot run --port 8080 --host localhost --no-telemetry

# Simulation modes
phosphobot run --simulation=gui        # With PyBullet GUI
phosphobot run --simulation=headless   # Headless mode
phosphobot run --only-simulation       # Simulation only, no hardware

# Development mode
phosphobot run --simulation=gui --port=8080 --host=127.0.0.1 --no-telemetry

# Check system info
phosphobot info --opencv --servos
```

### Environment Variables

```bash
# Hugging Face token for model access
export HF_TOKEN="your_huggingface_token"

# OpenAI API key for LLM integration
export OPENAI_API_KEY="your_openai_key"

# ElevenLabs for text-to-speech
export ELEVENLABS_API_KEY="your_elevenlabs_key"
```

## Supported Hardware

### Robot Arms

| Robot | Status | Connection | Notes |
|-------|--------|------------|-------|
| SO-100 | ✅ Full | USB/Serial | Starter kit robot |
| SO-101 | ✅ Full | USB/Serial | Advanced version |
| Koch v1.1 | 🟡 Beta | USB/Serial | Community supported |
| WX-250 | 🟡 Beta | USB/Serial | Trossen Robotics |
| AgileX Piper | 🟡 Beta | Ethernet | Linux only |
| Unitree Go2 | 🟡 Beta | WiFi | Air/Pro/Edu models |

### Cameras

| Camera Type | Status | Interface | Notes |
|-------------|--------|-----------|-------|
| USB Webcams | ✅ Full | USB | Standard UVC cameras |
| RealSense D435i | ✅ Full | USB 3.0 | Depth + RGB |
| RealSense D455 | ✅ Full | USB 3.0 | Depth + RGB |
| IP Cameras | 🟡 Beta | Network | RTSP streams |

### Controllers

| Controller | Status | Connection | Notes |
|------------|--------|------------|-------|
| Keyboard | ✅ Full | USB/Wireless | Web interface |
| Leader Arm | ✅ Full | USB/Serial | Teleoperation |
| Meta Quest | ✅ Full | WiFi | VR teleoperation |
| Joystick/Gamepad | 🟡 Beta | USB | Custom mapping |

## API Reference

### Core Endpoints

The Phosphobot server exposes a comprehensive REST API and WebSocket interface:

**Server Status:**
```bash
GET /status                    # Server health and info
GET /docs                      # Interactive API documentation
```

**Robot Control:**
```bash
POST /move/init                # Initialize robot
POST /move/absolute            # Absolute positioning
POST /move/relative            # Relative movement
POST /joints/read              # Read joint angles
POST /joints/write             # Write joint angles
POST /move/teleop              # Teleoperation mode
```

**Camera Operations:**
```bash
GET /cameras                   # List available cameras
GET /camera/{id}/frame         # Get camera frame
POST /cameras/config           # Configure cameras
```

**AI Control:**
```bash
POST /ai/start                 # Start AI control
POST /ai/stop                  # Stop AI control
GET /ai/status                 # AI control status
POST /ai/model/load            # Load AI model
```

**Recording & Datasets:**
```bash
POST /recording/start          # Start recording
POST /recording/stop           # Stop recording
GET /recording/status          # Recording status
GET /datasets                  # List datasets
POST /datasets/upload          # Upload dataset
```

### Python Client API

```python
import httpx
import numpy as np
from phosphobot.camera import AllCameras

# Basic robot control
client = httpx.Client(base_url="http://localhost:80")

# Initialize robot
client.post("/move/init")

# Move robot to absolute position
client.post("/move/absolute", json={
    "x": 10.0, "y": 0.0, "z": 15.0,
    "rx": 0.0, "ry": 0.0, "rz": 0.0,
    "open": 0.5
})

# Read joint angles
response = client.get("/joints/read")
angles = response.json()["angles_rad"]

# Camera access
cameras = AllCameras()
frame = cameras.get_rgb_frame(camera_id=0, resize=(320, 240))
```

### WebSocket Interface

```javascript
// Real-time teleoperation
const ws = new WebSocket('ws://localhost:80/ws/teleop');

ws.onopen = function() {
    // Send control commands
    ws.send(JSON.stringify({
        type: 'control',
        data: {
            x: 0.1, y: 0.0, z: 0.0,
            rx: 0.0, ry: 0.0, rz: 0.0
        }
    }));
};

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log('Robot feedback:', data);
};
```

## Custom Integrations

### Adding a New Robot

1. **Create a new robot class** in `phosphobot/hardware/`:

```python
from phosphobot.hardware.base import BaseManipulator

class MyCustomRobot(BaseManipulator):
    def __init__(self, port="/dev/ttyUSB0", **kwargs):
        super().__init__(**kwargs)
        self.port = port
        # Initialize your robot connection
        
    def move_to_position(self, position, orientation, gripper_openness):
        # Implement robot movement
        pass
        
    def read_position(self):
        # Return current position
        return {"position": [0, 0, 0], "orientation": [0, 0, 0]}
        
    def set_torque_enable(self, enable):
        # Enable/disable motors
        pass
```

2. **Register the robot** in `phosphobot/hardware/__init__.py`:

```python
from .my_custom_robot import MyCustomRobot

ROBOT_CLASSES = {
    "so100": SO100Hardware,
    "custom": MyCustomRobot,  # Add your robot
}
```

### Custom AI Models

1. **Create a model interface** in `phosphobot/am/`:

```python
from phosphobot.am.base import BaseActionModel

class MyCustomModel(BaseActionModel):
    def __init__(self, model_path):
        self.model = self.load_model(model_path)
    
    def sample_actions(self, observations):
        # Process observations and return actions
        return self.model.predict(observations)
    
    def load_model(self, path):
        # Load your trained model
        pass
```

2. **Register the model**:

```python
from phosphobot.am import register_model
register_model("custom_model", MyCustomModel)
```

### Custom Camera Integration

```python
from phosphobot.camera import BaseCamera

class MyCustomCamera(BaseCamera):
    def __init__(self, camera_id):
        super().__init__(camera_id)
        # Initialize camera connection
    
    def get_frame(self):
        # Return frame as numpy array
        return frame
    
    def get_rgb_frame(self, resize=None):
        frame = self.get_frame()
        if resize:
            frame = cv2.resize(frame, resize)
        return frame
```

### Voice Control Integration

```python
import speech_recognition as sr
from gtts import gTTS
import pygame

class VoiceController:
    def __init__(self, phosphobot_url="http://localhost:80"):
        self.client = httpx.Client(base_url=phosphobot_url)
        self.recognizer = sr.Recognizer()
        self.microphone = sr.Microphone()
    
    def listen_and_execute(self):
        with self.microphone as source:
            audio = self.recognizer.listen(source)
        
        try:
            command = self.recognizer.recognize_google(audio)
            self.execute_voice_command(command)
        except sr.UnknownValueError:
            print("Could not understand audio")
    
    def execute_voice_command(self, command):
        if "move up" in command.lower():
            self.client.post("/move/relative", json={"z": 5.0})
        elif "open gripper" in command.lower():
            self.client.post("/move/relative", json={"open": 1.0})
```

### Custom Training Pipeline

```python
from phosphobot.training import BaseTrainer

class MyCustomTrainer(BaseTrainer):
    def __init__(self, dataset_path, model_config):
        self.dataset_path = dataset_path
        self.config = model_config
    
    def prepare_data(self):
        # Load and preprocess dataset
        pass
    
    def train(self, epochs=100):
        # Implement training loop
        pass
    
    def save_model(self, path):
        # Save trained model
        pass
```

## Examples and Use Cases

### 1. Simple Movement Control

```python
# examples/01-simple-movement/main.py
import time
import httpx

client = httpx.Client(base_url="http://localhost:80")

# Initialize robot
client.post("/move/init")
time.sleep(2)

# Move in a square pattern
positions = [
    {"x": 10, "y": 10, "z": 0},
    {"x": 10, "y": -10, "z": 0},
    {"x": -10, "y": -10, "z": 0},
    {"x": -10, "y": 10, "z": 0}
]

for pos in positions:
    client.post("/move/absolute", json={**pos, "rx": 0, "ry": 0, "rz": 0, "open": 0})
    time.sleep(1)
```

### 2. Data Recording for Training

```python
# examples/02-data-recording/record_task.py
import time
import httpx

client = httpx.Client(base_url="http://localhost:80")

# Start recording
client.post("/recording/start", json={
    "dataset_name": "pick_and_place_demo",
    "task_description": "Pick up red block and place in box",
    "frequency": 30
})

print("Recording started. Perform the task now...")
input("Press Enter when done...")

# Stop recording
result = client.post("/recording/stop")
print(f"Recording saved: {result.json()}")
```

### 3. AI Model Inference

```python
# examples/03-ai-control/inference.py
import cv2
import numpy as np
from phosphobot.camera import AllCameras
from phosphobot.am import Gr00tN1
import httpx

# Setup
cameras = AllCameras()
model = Gr00tN1(server_url="localhost", server_port=5555)
client = httpx.Client(base_url="http://localhost:80")

while True:
    # Get camera frames
    images = [
        cameras.get_rgb_frame(camera_id=0, resize=(320, 240)),
        cameras.get_rgb_frame(camera_id=1, resize=(320, 240))
    ]
    
    # Get robot state
    response = client.get("/joints/read")
    joint_angles = response.json()["angles_rad"]
    
    # Prepare observation
    obs = {
        "video.image_cam_0": np.expand_dims(images[0], axis=0),
        "video.image_cam_1": np.expand_dims(images[1], axis=0),
        "state.arm": np.array(joint_angles[:6]).reshape(1, 6),
        "annotation.human.action.task_description": ["Pick up the red cube"]
    }
    
    # Get model prediction
    action = model.sample_actions(obs)
    
    # Execute action
    client.post("/joints/write", json={"angles": action[0].tolist()})
    time.sleep(1/30)  # 30 Hz control loop
```

### 4. Custom Web Interface

```html
<!-- examples/04-web-interface/index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Custom Robot Control</title>
</head>
<body>
    <div id="control-panel">
        <button onclick="initRobot()">Initialize</button>
        <button onclick="moveUp()">Move Up</button>
        <button onclick="moveDown()">Move Down</button>
        <button onclick="openGripper()">Open Gripper</button>
        <button onclick="closeGripper()">Close Gripper</button>
    </div>
    
    <script>
        const API_BASE = 'http://localhost:80';
        
        async function initRobot() {
            await fetch(`${API_BASE}/move/init`, {method: 'POST'});
        }
        
        async function moveUp() {
            await fetch(`${API_BASE}/move/relative`, {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({z: 5.0})
            });
        }
        
        async function openGripper() {
            await fetch(`${API_BASE}/move/relative`, {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({open: 1.0})
            });
        }
    </script>
</body>
</html>
```

## Development Setup

### Frontend Development

The dashboard is built with React and Vite:

```bash
cd dashboard

# Install dependencies
npm install

# Development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Backend Development

```bash
cd phosphobot

# Install development dependencies
uv add --dev pytest mypy ruff

# Run tests
uv run pytest

# Type checking
uv run mypy phosphobot/

# Code formatting
uv run ruff check phosphobot/
uv run ruff format phosphobot/
```

### Building Executables

```bash
# Build standalone executable
make build_pyinstaller

# Run the executable
./phosphobot/dist/main.bin run --simulation=headless
```

### Docker Development

```bash
# Build custom image
docker build -t my-phosphobot .

# Run container
docker run -p 80:80 --device=/dev/ttyUSB0 my-phosphobot
```

## Advanced Configuration

### Multi-Robot Setup

```yaml
# config.yaml
robots:
  - id: 0
    type: "so100"
    port: "/dev/ttyUSB0"
    name: "Left Arm"
  - id: 1
    type: "so100"
    port: "/dev/ttyUSB1"
    name: "Right Arm"

# Enable dual-arm control
dual_arm_mode: true
```

### Cloud Integration

```yaml
# config.yaml
cloud:
  supabase_url: "your_supabase_url"
  supabase_key: "your_key"
  enable_cloud_sync: true
  auto_upload_datasets: true

ai_providers:
  modal:
    enabled: true
    environment: "production"
  huggingface:
    auto_upload_models: true
```

### Network Configuration

```yaml
# config.yaml
network:
  host: "0.0.0.0"  # Listen on all interfaces
  port: 80
  cors_origins: ["*"]
  max_connections: 100
  
  # HTTPS configuration
  ssl:
    enabled: false
    cert_file: "/path/to/cert.pem"
    key_file: "/path/to/key.pem"
```

## Troubleshooting

### Common Issues

**Port 80 already in use:**
```bash
# Use different port
phosphobot run --port 8080

# Or kill process using port 80
sudo lsof -i :80
sudo kill -9 <PID>
```

**Camera not detected:**
```bash
# Check available cameras
phosphobot info --opencv

# Test camera access
python -c "import cv2; print(cv2.VideoCapture(0).read())"

# Disable camera if needed
echo "enable_cameras: false" >> ~/.phosphobot/config.yaml
```

**Serial port permission denied:**
```bash
# Add user to dialout group (Linux)
sudo usermod -a -G dialout $USER
# Logout and login again

# Or change permissions
sudo chmod 666 /dev/ttyUSB0
```

**AI model errors:**
```bash
# Check GPU availability
python -c "import torch; print(torch.cuda.is_available())"

# Install CUDA support
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124

# Use CPU-only mode
phosphobot run --ai-device cpu
```

### Debug Mode

```bash
# Enable debug logging
export PHOSPHOBOT_LOG_LEVEL=DEBUG
phosphobot run --simulation=gui --no-telemetry

# Profile performance
phosphobot run --profile
# Creates profile.html in root directory
```

### Getting Help

- **Documentation**: https://docs.phospho.ai
- **Discord Community**: https://discord.gg/cbkggY6NSK
- **GitHub Issues**: https://github.com/phospho-app/phosphobot/issues
- **API Documentation**: http://localhost:80/docs (when server is running)

### Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Make your changes and add tests
4. Run the test suite: `uv run pytest`
5. Submit a pull request

For major changes, please open an issue first to discuss your proposed changes.

---

**Made with 💚 by the Phospho community**

For commercial support and custom integrations, contact: contact@phospho.ai