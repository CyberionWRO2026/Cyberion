# Cyberion
Autonomous Ackermann-steering vehicle developed for the WRO Future Engineers 2026 competition. Includes mechanical design, electronics, control software, computer vision, sensor integration, and engineering documentation.
We are a team of three students who share more than just a passion for robotics — we share three years of friendship, late nights debugging code, and the relentless drive to build something extraordinary. What started as a shared curiosity for technology has evolved into a serious engineering partnership, where each of us brings a unique and irreplaceable skill set to the table.

<img width="1000" height="850" alt="image" src="https://github.com/user-attachments/assets/fb4b2cf9-931a-49ba-a13e-d00ef85702c0" />
<p align="center">

Ali Essa serves as our software architect and programming lead. He is responsible for the full software stack of our self-driving vehicle — from low-level motor control to high-level computer vision algorithms. His ability to translate complex engineering problems into elegant, efficient code is the brain behind our robot's autonomous decision-making.

Ibrahim Jneed & Lujain Alahmad form our hardware and electrical engineering duo. Ibrahim and Lujain are responsible for the complete 3D mechanical design, chassis engineering, and all electrical wiring and circuit architecture. Their precision in design and deep understanding of electromechanical systems ensure that every component fits, functions, and performs under competition conditions.

Together, we do not just build robots — we engineer solutions. Every decision we make is driven by data, testing, and a shared obsession with pushing our vehicle's performance to its absolute limit.


## ⚡️ Electronic Systems

Our robot's electronic architecture is built around a clear separation of responsibilities: a Raspberry Pi 5 handles all vision processing and high-level navigation, while an ESP32 microcontroller manages every time-critical control task — motor drive, steering, and sensor reading — in real time. These two processors communicate over a dedicated UART serial link, keeping vision latency completely isolated from motion control.

Power is distributed across three independent paths: the drive motor is fed directly from the 4S Li-ion battery through a BTS7960 H-bridge driver, the steering servo runs from a dedicated regulated Rail A, and all computing and sensing electronics share a separate regulated Rail B. This three-way separation ensures that motor switching transients and servo current spikes never reach the Raspberry Pi 5, ESP32, or sensors. A full Fritzing wiring diagram documenting every power and signal connection is included in the repository.

### Components

The diagram below shows the full wiring architecture — power distribution paths, signal connections, and communication links between all major components. A complete Fritzing source file is included in the /fritzing directory.

<img width="2955" height="2922" alt="fritzing_bb" src="https://github.com/user-attachments/assets/d4efde9e-e6ad-4282-b579-da74d1c84d9d" />

#### Battery — Li-ion 18650 (×4, 4S Pack)

The robot is powered by a 4S Li-ion battery pack consisting of four 3.7 V, 2200 mAh 18650 cells connected in series. The pack provides 14.8 V nominal voltage and reaches 16.3 V when fully charged. It stores 32.56 Wh, with approximately 22.79 Wh usable at 70% DoD. The measured runtime under full competition load is 95 minutes. The cells are rated for 2.2 A continuous discharge (1C) and 4.4 A burst discharge (2C), with a system low-voltage cut-off at 12.0 V.

#### Power Regulation — LM2596 Buck Converters

Two LM2596 HW-411 buck converters regulate the battery voltage to 5 V. Rail A is dedicated to the MG996R steering servo, providing isolation from servo current spikes, with a measured efficiency of 84.5%. Rail B supplies the Raspberry Pi 5, ESP32, and all sensors, with a measured efficiency of 83.1%. Both rails operate from the 16.3 V maximum battery voltage and provide a regulated 5.00 V output.

<img width="329" height="325" alt="image" src="https://github.com/user-attachments/assets/228ba31f-159d-4124-8608-66afbf0272b6" />


#### Raspberry Pi 5 (8GB)

The Raspberry Pi 5 is the main computing unit and runs the Python/OpenCV vision and navigation pipeline on its quad-core ARM Cortex-A76 processor. It connects to the camera through CSI-2 and communicates with the ESP32 through UART. Its typical and peak current consumption is approximately 0.80 A and 1.80 A, respectively, and it is powered from Rail B.

<img width="320" height="250" alt="image" src="https://github.com/user-attachments/assets/10b350a2-c105-4e72-92a9-31ba2cd73d26" />

#### Raspberry Pi Camera Module 2

The Raspberry Pi Camera Module 2, based on the Sony IMX219 8 MP sensor, is connected to the Pi 5 through a CSI-2 ribbon cable. It captures colour images for real-time HSV-based obstacle classification. During validation, the system achieved 95% detection accuracy (19/20 correct detections) under approximately 500 lux fluorescent lighting.

<img width="237" height="180" alt="image" src="https://github.com/user-attachments/assets/39177b21-8617-403b-a27a-d3f1a05ef465" />

#### ESP32-WROOM-32

The ESP32-WROOM-32 DevKit v1 handles all real-time control tasks, including motor PWM, steering control, and continuous I²C sensor polling. Its dual-core 240 MHz processor keeps motion control independent of the Raspberry Pi's Linux scheduling. The ESP32 operates at a sensor polling rate of 100 Hz, with typical and peak current consumption of approximately 0.080 A and 0.250 A, respectively.

<img width="320" height="320" alt="image" src="https://github.com/user-attachments/assets/7d05d40e-3b38-4230-8a39-b2a34d901fd7" />


#### JSumo Titan DC Motor — 12 V / 1000 RPM

The JSumo Titan DC motor is powered directly from the battery through the BTS7960 driver. At a 72% PWM duty cycle, the effective motor voltage is approximately 11.20 V. During testing, the robot achieved a measured field speed of 1.5 m/s, with a typical motor load current of approximately 1.20 A and a measured stopping distance of 0.56 m.

<img width="220" height="300" alt="image" src="https://github.com/user-attachments/assets/b7747563-954e-4436-87b2-9739f1062fd4" />


#### BTS7960B Motor Driver

The BTS7960B is a high-current MOSFET H-bridge used to control the drive motor in both directions through PWM. It is rated up to 43 A absolute maximum and accepts the ESP32's 3.3 V logic signals directly. The driver is connected directly to the battery and operates over the 12.0–16.8 V supply range.

<img width="295" height="299" alt="image" src="https://github.com/user-attachments/assets/d45d7a2c-4a97-4af1-88d4-65481a1b2e9b" />


#### TowerPro MG996R Servo

The MG996R provides steering control and is powered separately from Rail A to isolate its current spikes from the logic electronics. It uses a 50 Hz PWM control signal, with a typical current of approximately 0.35 A and a measured stall current of 2.50 A. Its rated stall torque is approximately 9.4 kgf·cm at 4.8 V.

<img width="320" height="250" alt="image" src="https://github.com/user-attachments/assets/53c120ca-5f9a-4c2d-9b5c-7deb4e75d998" />

#### BNO086 IMU

The BNO086 IMU is mounted near the geometric centre of the robot and provides heading and angular-rate information through the I²C bus. After zero-rate bias calibration to 0.018°/s, controlled 90° turns showed a validated heading error of 1.8°, while the total drift during a full competition run was approximately 3.2°. The IMU is powered from the ESP32's 3.3 V supply.

<img width="320" height="320" alt="image" src="https://github.com/user-attachments/assets/7d0600bf-1e36-4eaf-8d36-98aac0b73a9a" />


#### VL53L0X ToF Sensors (×4)

Four VL53L0X time-of-flight sensors provide distance measurements in the front, rear, left, and right directions. They share a single I²C bus, with individual XSHUT control used during startup to assign unique addresses from 0x30 to 0x33. The four sensors consume approximately 64 mA combined, and the measured distance error remains within ±12 mm across the specified operating range. They are powered from Rail B.

### Power Consumption Summary
|        Component       |      Voltage      | Typical Current | Peak Current |   Power Rail   |
|:----------------------:|:-----------------:|:---------------:|:------------:|:--------------:|
| Raspberry Pi 5 (8GB)   | 5 V               | 0.80 A          | 1.80 A       | Rail B         |
| RPi Camera Module 2    | 3.3 V (via Pi 5)  | 0.13 A          | 0.25 A       | via Pi 5       |
| ESP32-WROOM-32         | 5 V → 3.3 V       | 0.080 A         | 0.250 A      | Rail B         |
| BNO086 IMU             | 3.3 V (via ESP32) | 10.7 mA         | 16 mA        | via ESP32      |
| VL53L0X ToF ×4         | 5 V → 2.8 V       | 64 mA total     | 40 mA each   | Rail B         |
| TowerPro MG996R Servo  | 5 V               | 0.35 A          | 2.50 A       | Rail A         |
| JSumo Titan DC Motor   | 12 V (PWM)        | 1.20 A          | 5.00 A       | Battery direct |
| BTS7960B Motor Driver  | 12 – 16.8 V       | 1.14 mA         | —            | Battery direct |
| LM2596 Rail A          | 16.3 V → 5 V      | 0.128 A         | —            | Battery        |
| LM2596 Rail B          | 16.3 V → 5 V      | 0.400 A         | —            | Battery        |
| 4S Li-ion Pack (total) | 14.8 V nominal    | 0.744 A         | —            | Source         |


### Power Budget

Every watt counts in competition. Here's exactly where the power goes — calculated from measured values, not estimates.

The brain (Rail B — Logic & Sensors)
The Pi 5 with camera draws 0.913 A, the ESP32 adds 0.083 A, and four ToF sensors contribute 0.064 A — totalling 1.060 A at 5.03 V. That's 5.332 W at the rail, which means the battery has to supply 6.441 W to cover the LM2596's 83.1% conversion loss.

The muscles (Rail A — Servo)
The steering servo runs on its own isolated rail. At 0.347 A × 5.02 V = 1.742 W out, the battery delivers 2.061 W through the LM2596 at 84.5% efficiency — completely isolated from everything else.

The drive (Motor — Direct)
The motor bypasses regulation entirely, fed straight from the pack at 72% PWM. Effective voltage: 11.20 V, current: 0.300 A — 3.36 W of mechanical output, drawing 0.216 A directly from the battery.

Total Battery Load
|           Path           | Battery Current | Battery Power |
|:------------------------:|:---------------:|:-------------:|
| Rail B — Logic & Sensors | 0.400 A         | 6.441 W       |
| Rail A — Servo           | 0.128 A         | 2.061 W       |
| Motor — Direct           | 0.216 A         | 3.370 W       |
| Total                    | 0.744 A         | 11.872 W      |

Runtime — the bottom line

With 22.79 Wh of usable energy and an average draw of 12.45 W, the math gives 110 minutes. Real-world testing came in at 95 minutes  — a healthy margin that accounts for acceleration peaks and steering transients the steady-state model doesn't capture.
Calculated: 22.79 ÷ 12.45 = 110 min · Measured: 95 min 
We run two identical 4S packs in rotation: one powers the robot, the other charges. A full competition run takes well under 95 minutes, so a fresh pack is always ready before the other is depleted.

### Connection & Pin Assignment
|             From            |               To              |          Signal         |
|:---------------------------:|:-----------------------------:|:-----------------------:|
| ESP32 GPIO 25               | BTS7960 RPWM                  | Motor PWM — forward     |
| ESP32 GPIO 26               | BTS7960 LPWM                  | Motor PWM — reverse     |
| ESP32 GPIO 27 + 14          | BTS7960 R_EN + L_EN           | Driver enable           |
| ESP32 GPIO 13               | MG996R signal                 | Steering PWM 50 Hz      |
| ESP32 GPIO 21 (SDA)         | BNO086 SDA + VL53L0X ×4 SDA   | Shared I²C data 400 kHz |
| ESP32 GPIO 22 (SCL)         | BNO086 SCL + VL53L0X ×4 SCL   | Shared I²C clock        |
| ESP32 GPIO 32 / 33 / 4 / 15 | VL53L0X #1–#4 XSHUT           | Address assignment      |
| ESP32 GPIO 16 (RX2)         | Pi 5 GPIO 14 (TXD)            | UART Pi → ESP32         |
| ESP32 GPIO 17 (TX2)         | Pi 5 GPIO 15 (RXD)            | UART ESP32 → Pi         |
| Rail B 5 V                  | ESP32 VIN + Pi 5 + VL53L0X ×4 | Logic & sensor supply   |
| ESP32 3V3                   | BNO086 VIN                    | IMU supply              |
| Battery +                   | BTS7960 B+                    | Motor power             |
| Common GND                  | All components                | Single-point ground     |


and the next Figure presents the complete electrical and electronic architecture of the robot, showing the power distribution paths, regulated voltage rails, communication interfaces, motor and steering control, and connections between the Raspberry Pi 5, ESP32, and all onboard sensors.

<img width="1402" height="1122" alt="image" src="https://github.com/user-attachments/assets/dbc7804b-9491-4442-af88-4420a9d2f3ac" />

## src

### Simulation

1. Extract `wro2026_sim.zip` to any directory on your Ubuntu 24.04 system.
2. Open a terminal and navigate to the extracted folder:
```bash
   cd ~/<path-to-extracted-folder>/wro2026_sim
```
3. Set the Gazebo resource path:
```bash
   export GZ_SIM_RESOURCE_PATH=$PWD/models:$PWD/worlds
```
4. Launch the simulator:
```bash
   gz sim -r worlds/world.sdf
```
5. The simulator should now open successfully.

### Codes

