# A11G Final Submission

**Team Number:** 01

**Team Name:** Bubble

**GitHub Repository URL:** <https://github.com/ese5160/a11g-final-submission-s26-s26-t01-bubble>

**GitHub Pages URL:** `https://chennn0224.github.io/a11g-final-submission-s26-s26-t01-bubble/`

| Team Member Name | Email Address | GitHub Handle |
| --- | --- | --- |
| Zhiyuan Chen | chennn@seas.upenn.edu | chennn |
| Mianzhi Wu | mzwu098@seas.upenn.edu | Max098Wu |

## 1. Video Presentation

Final YouTube video link: <https://youtube.com/shorts/kgy-Gb3B5Gw?feature=share>

The final demo shows the tumbler-style prototype, the phone-based HTTP control page, live sensor telemetry, and motor response through the MCU-hosted Wi-Fi interface.

## 2. Project Summary

### Device Description

Team Bubble's final device is a self-balancing tumbler-style embedded prototype that uses IMU and barometric sensor data to observe motion and pressure/altitude changes while driving three motors. The device creates a Wi-Fi access point and serves a phone-friendly HTTP dashboard directly from the MCU for motor control and live sensor display.

The project was originally inspired by an airship concept, but the final prototype focuses on a smaller, more controllable physical platform that still integrates sensing, actuation, wireless networking, and a browser UI.

### Internet-Connected Functionality

The final project does **not** use Node-RED buttons or MQTT for sensor transport. According to the final burned code in `sl_si91x_i2c_driver_leader`, the SiWG917 creates a SoftAP named `Airship-917` and runs an HTTP server on port `8080`.

The browser page sends motor commands through:

- `/cmd?m=<motor_index>&s=<signed_speed_percent>`
- `/hold?e=<0_or_1>&t=<target_altitude_cm>`
- `/zero`

The page reads live sensor/control data through:

- `/state`

The `/state` JSON includes IMU validity, barometer validity, altitude, vertical speed, pressure, temperature, acceleration, gyro Z, altitude-hold state, target altitude, and lift output.

### Device Functionality

Final firmware project: [`sl_si91x_i2c_driver_leader`](https://github.com/ese5160/final-project-firmware-s26-t01-bubble/tree/main/sl_si91x_i2c_driver_leader)

Main firmware modules:

- `app.c`: initializes motors, sensor/control logic, and web control.
- `airship_sensors.c`: reads LSM6DSO IMU and BMP390 pressure sensor over I2C.
- `airship_control.c`: implements a simple pressure-based altitude hold path for motor C.
- `motor_test.c`: drives three motor channels with direction GPIO and PWM.
- `web_control.c`: starts Wi-Fi SoftAP and serves the HTTP dashboard plus JSON endpoints.

The directory and some symbols still use the word "airship" because the project started with the airship concept. The final implementation behavior is the tumbler/HTTP control prototype described here.

### Why We Pivoted Away From the Airship

The original plan was to build a custom airship with a shaped balloon. A custom balloon option cost about $500, so we tried lower-cost party balloons and nitrogen instead. After inflation, the party balloons did not form a regular shape; they behaved more like an uneven cone with a poorly distributed center of mass and buoyancy. This made stable flight control extremely difficult within the remaining time and budget, so we pivoted to a tumbler-style prototype that could still validate embedded sensing, motor actuation, Wi-Fi communication, and mobile control.

### Challenges

The biggest technical challenge was integration. Sensors, motors, Wi-Fi, and the web server all had to run together on the same embedded target. Earlier plans assumed Node-RED/MQTT, camera streaming, and airship flight dynamics, but the final code shows that the practical path became local Wi-Fi plus HTTP.

We overcame this by simplifying the network path. Instead of relying on an external dashboard for final control, the MCU hosts the control page and exposes a small JSON endpoint. That reduced moving parts and made the phone-to-device demo much more reliable.

### Prototype Learnings

We learned that mechanical feasibility can dominate embedded system design. The airship was not abandoned because the firmware idea was impossible; it was abandoned because the low-cost physical balloon did not provide a controllable platform.

If we rebuilt the project, we would validate the mechanical body and center-of-mass behavior much earlier, then name the firmware modules after the final device instead of carrying old "airship" labels into the final code.

### Next Steps & Takeaways

Next steps:

- Rename final firmware symbols and UI text from "airship" to the tumbler device name.
- Add a stronger IMU-based closed-loop balancing controller.
- Improve the mechanical mount so each motor's output maps repeatably to movement.
- Add SoftAP security or station-mode networking.

The main ESE5160 takeaway was that IoT edge projects are not just about one driver or one cloud feature. A successful prototype requires sensor validity, actuator reliability, mechanical design, real-time firmware, and networking behavior to work together.

## 3. Hardware & Software Requirements Review

The table below reviews the earlier Airship HRS/SRS against the final burned firmware. Requirements tied to Node-RED, MQTT, camera, servo tilt, and cloud OTA are marked honestly because the final implementation uses MCU-hosted Wi-Fi and HTTP instead.

### HRS Review

| ID | Requirement | Status | Validation / Evidence |
| --- | --- | --- | --- |
| HRS-1 | Use a SiWG917-family MCU for sensor interfacing, wireless communication, and actuator control. | Met | Final burned project is `sl_si91x_i2c_driver_leader`; code initializes I2C, PWM/GPIO motors, and Wi-Fi. |
| HRS-2 | Support Wi-Fi connectivity for cloud communication, remote control, and OTA updates. | Partial | Wi-Fi remote control is met through SoftAP + HTTP. Cloud/OTA is not part of the final control path. |
| HRS-3 | Include an IMU and magnetometer for motion/orientation. | Partial | Final code uses LSM6DSO accelerometer/gyroscope. It does not use a magnetometer in the final burned project. |
| HRS-4 | Include a pressure sensor to estimate altitude. | Met | BMP390 pressure is compensated and converted into relative altitude. |
| HRS-5 | Include an environmental sensor. | Partial | BMP390 reports temperature and pressure. Earlier BME680 work existed, but final burned code uses BMP390 rather than full humidity/gas sensing. |
| HRS-6 | Include a camera for remote visual monitoring. | Not met | Camera integration was removed from the final path after the airship pivot. |
| HRS-7 | Include motors for movement control. | Met | `motor_test.c` implements three motor channels with direction GPIO and PWM. |
| HRS-8 | Include a servo motor for camera tilt. | Not met | No final camera tilt servo path is used. |
| HRS-9 | Include a speaker for audio output or alerts. | Not met | Speaker output was not included in the final prototype. |
| HRS-10 | Support OTA firmware update through a cloud-hosted image. | Partial | OTA was explored earlier, but final demonstrated control firmware is the I2C/HTTP project. Claim only what the final video proves. |

### SRS Review

| ID | Requirement | Status | Validation / Evidence |
| --- | --- | --- | --- |
| SRS-1 | Initialize and manage sensing, actuation, and communication peripherals. | Met | `app_init()` starts motors, sensor/control logic, and web control. |
| SRS-2 | Connect the airship/device to Wi-Fi during startup. | Met | Final firmware starts its own SoftAP. |
| SRS-3 | Support MQTT communication between device and cloud. | Not met in final firmware | Final burned project uses HTTP, not MQTT. |
| SRS-4 | Subscribe to `airship/control/motion` and parse JSON motion commands. | Not met in final firmware | Final motion commands are HTTP query parameters at `/cmd?m=&s=`. |
| SRS-5 | Subscribe to camera tilt commands. | Not met | Camera tilt was removed from the final prototype. |
| SRS-6 | Control motion motors according to received commands. | Met | HTTP commands call `motor_test_set_signed_speed()`. |
| SRS-7 | Control camera tilt servo according to angle commands. | Not met | No final servo tilt command path is present. |
| SRS-8 | Publish system status information. | Partial | Status is not MQTT-published, but it is exposed as JSON through `/state`. |
| SRS-9 | Status data should include key system information. | Partial | Final JSON includes sensor validity, altitude, vertical speed, pressure, temperature, acceleration, gyro Z, hold state, target altitude, and lift output. It does not include camera angle. |
| SRS-10 | Support OTA firmware update using a cloud-hosted image. | Partial | Earlier OTA work exists, but it is separate from final HTTP control functionality unless demonstrated in the final video. |
| SRS-11 | Use multiple threads/tasks. | Met | Firmware creates a sensor task and a web server task with CMSIS-RTOS2/FreeRTOS. |
| SRS-12 | Use shared variables, queues, or event flags for inter-task communication. | Met | Sensor, control, and HTTP paths share structured state through `airship_sensor_state_t` and `airship_control_state_t`. |
| SRS-13 | Transmit live camera feed separately from MQTT. | Not met | Camera streaming was not implemented in the final prototype. |

## 4. Project Photos & Screenshots

The GitHub Pages site includes final prototype photos, the MCU-hosted HTTP control dashboard, Altium 2D/3D PCB screenshots, and the system block diagram.

## 5. Codebase

Final embedded C firmware:

- <https://github.com/ese5160/final-project-firmware-s26-t01-bubble/tree/main/sl_si91x_i2c_driver_leader>

Node-RED / earlier dashboard code:

- <https://github.com/ese5160/final-project-firmware-s26-t01-bubble/tree/main/Node-RED>

Important note: Node-RED was used in earlier course work, but it is **not** the final motor-control or sensor-transport path. The final device dashboard is hosted directly by the MCU over HTTP.

## Project Links

- A11G submission repository: <https://github.com/ese5160/a11g-final-submission-s26-s26-t01-bubble>
- GitHub Pages: `https://chennn0224.github.io/a11g-final-submission-s26-s26-t01-bubble/`
- Final firmware code: <https://github.com/ese5160/final-project-firmware-s26-t01-bubble/tree/main/sl_si91x_i2c_driver_leader>
