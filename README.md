# ESP32-CAM Web Server

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/Platform-ESP32--CAM-blue.svg)](https://www.espressif.com/)
[![Arduino](https://img.shields.io/badge/Arduino-Compatible-green.svg)](https://www.arduino.cc/)

A simple and lightweight video streaming server for ESP32-CAM with web interface. Access your camera from any device on your local network or remotely via internet using ngrok.

## Features

- MJPEG video streaming via HTTP
- Responsive web interface (works on mobile and desktop)
- Easy WiFi configuration
- Remote access support via ngrok tunnel
- Compatible with AI-Thinker ESP32-CAM module
- Works with Arduino IDE (Windows) and Arduino CLI (Linux)

## Hardware Requirements

- ESP32-CAM (AI-Thinker model)
- USB cable (if board has integrated USB) or FTDI adapter
- Computer with Windows or Linux

## Quick Start

### 1. Configure WiFi

Edit `ESP32CAM_WebServer.ino` and set your WiFi credentials:

```cpp
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";
```

### 2. Upload the Code

**Using Arduino IDE (Windows):**
1. Install [Arduino IDE](https://www.arduino.cc/en/software)
2. Add ESP32 board support (see [Installation Guide](#arduino-ide-windows))
3. Select **AI Thinker ESP32-CAM** board
4. Click Upload

**Using Arduino CLI (Linux):**
```bash
arduino-cli compile --fqbn esp32:esp32:esp32cam ESP32CAM_WebServer
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32cam ESP32CAM_WebServer
```

### 3. Access the Camera

Open Serial Monitor (115200 baud) to see the IP address:
```
Camera inicializada!
WiFi conectado!
Acesse: http://192.168.x.x
Servidor iniciado!
```

Open the IP in your browser to view the stream.

## Limitations

- **Single connection only**: ESP32-CAM supports only one client at a time for streaming
- **2.4GHz WiFi only**: ESP32 does not support 5GHz networks
- Streaming may be unstable on long sessions due to memory constraints

---

## Installation Guide

### Arduino IDE (Windows)

1. **Install Arduino IDE**
   - Download from: https://www.arduino.cc/en/software

2. **Add ESP32 Board Support**
   - Go to **File > Preferences**
   - Add to **Additional Board Manager URLs**:
     ```
     https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
     ```
   - Go to **Tools > Board > Boards Manager**
   - Search for **esp32** and install the Espressif Systems package

3. **Install USB Driver (if needed)**
   - **CH340**: Search "CH340 driver download"
   - **CP210x**: Download from Silicon Labs website

4. **Upload**
   - Select **Tools > Board > AI Thinker ESP32-CAM**
   - Select **Tools > Port > COMx**
   - Click **Upload**

5. **Serial Monitor**
   - Open **Tools > Serial Monitor**
   - Set baud rate to **115200**

### Arduino CLI (Linux)

1. **Install Arduino CLI**
   ```bash
   curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | BINDIR=~/.local/bin sh
   ```

2. **Configure ESP32 Support**
   ```bash
   arduino-cli config init
   arduino-cli config add board_manager.additional_urls https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   arduino-cli core update-index
   arduino-cli core install esp32:esp32
   ```

3. **Serial Port Permission**
   ```bash
   sudo usermod -a -G dialout $USER
   # Logout and login again
   ```

4. **Compile and Upload**
   ```bash
   arduino-cli compile --fqbn esp32:esp32:esp32cam ESP32CAM_WebServer
   arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32cam ESP32CAM_WebServer
   ```

5. **Serial Monitor**
   ```bash
   # IMPORTANT: use dtr=off and rts=off to prevent board reset
   arduino-cli monitor -p /dev/ttyUSB0 -c baudrate=115200 -c dtr=off -c rts=off
   ```

---

## Remote Access (ngrok)

Access your camera from anywhere using ngrok:

1. **Create account** at https://ngrok.com

2. **Install ngrok**
   - Windows: Download from https://ngrok.com/download
   - Linux:
     ```bash
     curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
     echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
     sudo apt update && sudo apt install ngrok
     ```

3. **Configure token**
   ```bash
   ngrok config add-authtoken YOUR_TOKEN_HERE
   ```

4. **Create tunnel**
   ```bash
   ngrok http CAMERA_IP:80
   # Example: ngrok http 192.168.0.23:80
   ```

5. **Access remotely**

   Use the generated URL (e.g., `https://xxxxx.ngrok-free.app`) from any device.

---

## Configuration

### Image Orientation

If the image is upside down, adjust these lines after `esp_camera_init`:

```cpp
sensor_t *s = esp_camera_sensor_get();
s->set_vflip(s, 1);    // 1 = flipped, 0 = normal
s->set_hmirror(s, 1);  // 1 = mirrored, 0 = normal
```

### Available Endpoints

| Endpoint | Description |
|----------|-------------|
| `/` | Web page with embedded stream |
| `/stream` | Direct MJPEG stream |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Garbled characters in Serial Monitor | Set baud rate to **115200** |
| Serial Monitor resets the board | Use `-c dtr=off -c rts=off` flags |
| Camera not initializing | Check if flat cable is properly connected (gold contacts facing the board) |
| WiFi not connecting | Verify SSID/password; ESP32-CAM only supports **2.4GHz** networks |
| Streaming unstable | Only 1 device can watch at a time; close other connections |
| ngrok not connecting | Verify camera is accessible on local network first |

---

## Project Structure

```
ESP32CAM_WebServer/
├── ESP32CAM_WebServer.ino    # Main code
├── README.md                  # Documentation
├── LICENSE                    # MIT License
└── .gitignore                 # Git ignore rules
```

---

## Contributing

Contributions are welcome! Feel free to:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- [Espressif Systems](https://www.espressif.com/) for ESP32 Arduino Core
- [Arduino](https://www.arduino.cc/) for the development platform
- [ngrok](https://ngrok.com/) for tunneling service
