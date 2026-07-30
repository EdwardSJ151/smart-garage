## Smart Garage System

This repository contains the implementation of an IoT-based Smart Garage system. The project integrates an ESP32 microcontroller, MQTT (via Mosquitto), and a Darknet/EasyOCR license-plate pipeline to automate garage door operations and vehicle entry logging.

<video src="https://github.com/mendesLet/smart-garage/releases/download/demo-assets/iot.mp4" controls muted playsinline width="720"></video>

### Directory Structure

```
smart-garage/
├── main.py                         # MQTT orchestrator (triggers plate detection, opens garage)
├── esp32_clients/
│   ├── iot_final.ino               # Main Arduino sketch for ESP32
│   └── PinDefinitionsAndMore.h     # Pin definitions and helper functions
├── raspi_clients/
│   ├── client_pub.py               # MQTT Publisher (Garage door control)
│   ├── client_sub.py               # MQTT Subscriber (Ultrasonic sensor)
├── plate_model/
│   ├── darknet_video_full_detect.py # Full-plate YOLO detection
│   ├── darknet_video_ocr.py         # Plate crop + EasyOCR
│   ├── FullPlates/                  # Full-plate model config/names
│   └── DiffPlates/                  # Diff-plate model config/names
└── README.md
```

### Setup Instructions

**1. Install Mosquitto on Linux**

```
sudo apt install mosquitto mosquitto-clients
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

<details>
<summary>Make sure to add this to the Mosquitto config files</summary>

```
listener 1883
allow_anonymous true
```

</details>


**2. ESP32 Setup**
- Upload the iot_final.ino sketch to your ESP32:
- Open esp32_clients/iot_final.ino in Arduino IDE.
- Set WiFi SSID/password and the MQTT broker IP in the sketch.
- Connect your ESP32 device and select the correct port under Tools > Port.
- Compile and upload the sketch.

**3. Raspberry Pi Setup**
- Install the dependencies
```
pip install paho-mqtt
```

- Run the main orchestrator (ultrasonic trigger → plate detection → open garage):
```
python main.py --fullplate
# or
python main.py --ocr
```

- Optionally run the standalone MQTT helpers:
```
python raspi_clients/client_pub.py
python raspi_clients/client_sub.py
```

### Usage

![Diagramas(1)](https://github.com/user-attachments/assets/f4a617de-56ce-4d62-abf9-58d418862c58)

1. Start the ESP32 with the uploaded iot_final.ino sketch.

2. Ensure Mosquitto is running and start `main.py` with `--fullplate` or `--ocr`.

3. Flow over MQTT topics:
    - ESP32 publishes to `ultrasonic/detection` when a vehicle is nearby.
    - `main.py` runs plate detection; on a valid plate it publishes to `garage/open_garage`.
    - ESP32 subscribes to `garage/open_garage` and sends the stored IR code to open the door.

4. For manual testing without the vision pipeline, use `raspi_clients/client_pub.py` to publish to `garage/open_garage`.
