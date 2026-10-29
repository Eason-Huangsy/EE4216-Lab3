# EE4216 Lab 3: Bidirectional IoT Gateway Using HTTP

Final Arduino sketches for Tasks 1–4 using two ESP32-S3 boards.

## Boards and libraries

- **Board B — Gateway:** open `Gateway_B_Final/Gateway_B_Final.ino`. It runs a Wi-Fi AP and connects to the upstream router as a station.
- **Board A — Sensor node:** open `Sensor_Node_A_Final/Sensor_Node_A_Final.ino`. It joins the gateway AP, reads the HC-SR04 and controls eight NeoPixels.
- In Arduino IDE, select **ESP32S3 Dev Module** from the ESP32 board package. Install **ArduinoJson** (v6 or v7) and **Adafruit NeoPixel**. The other libraries come with the ESP32 board package. Use Serial Monitor at **115200 baud**.

## Wiring on Board A

| Connection | GPIO |
| --- | --- |
| HC-SR04 TRIG | 4 |
| HC-SR04 ECHO | 5, through the Lab 1 voltage divider |
| NeoPixel DIN | 7 |

Use a common ground. The HC-SR04 ECHO is 5 V and must be reduced to 3.3 V for the ESP32 input. Confirm `LED_PIN` matches the actual wiring.

## Configuration

Replace the `YOUR_...` placeholders before uploading:

- Set the same gateway AP password in both sketches. The AP SSID is `A0309529W`.
- Set the upstream 2.4 GHz Wi-Fi SSID and password in the gateway sketch.
- Set `SENSOR_DATA_WRITE_KEY`, `LED_CONTROL_CHANNEL_ID` and `LED_CONTROL_READ_KEY` using two separate ThingSpeak channels: Sensor Data and LED Control.
- Keep private credentials out of GitHub. The LED Control write key is used when submitting a command to ThingSpeak; it is not needed in either sketch.

## Data and control flow

1. Board A sends a valid distance as JSON to gateway `POST /api/sensor` at `192.168.4.1`, about every 5 s.
2. Board B learns the node's IP from that request, prints the reading and uploads the latest new reading to Sensor Data field 1 by HTTPS POST, about every 20 s. NTP provides Singapore time and supports TLS certificate validation.
3. Board B polls the latest LED Control entry by HTTPS GET, about every 5 s. Field 1 selects LED 0–7; field 2 selects 0=off, 1=red, 2=green or 3=blue.
4. Board B forwards the command as JSON to Board A's `POST /api/led`. Board A returns an acknowledgement; the gateway retries until acknowledged and skips completed entries.

This repository contains the implementation. It does not include measurement logs or test screenshots.
