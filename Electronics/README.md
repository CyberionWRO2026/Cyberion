<img width="255" height="324" alt="image" src="https://github.com/user-attachments/assets/98048663-94a7-45fe-b655-b2314c17eff2" />⚡️ Electronic Systems

Our robot's electronic architecture is built around a clear separation of responsibilities: a Raspberry Pi 5 handles all vision processing and high-level navigation, while an ESP32 microcontroller manages every time-critical control task — motor drive, steering, and sensor reading — in real time. These two processors communicate over a dedicated UART serial link, keeping vision latency completely isolated from motion control.

Power is distributed across three independent paths: the drive motor is fed directly from the 4S Li-ion battery through a BTS7960 H-bridge driver, the steering servo runs from a dedicated regulated Rail A, and all computing and sensing electronics share a separate regulated Rail B. This three-way separation ensures that motor switching transients and servo current spikes never reach the Raspberry Pi 5, ESP32, or sensors. A full Fritzing wiring diagram documenting every power and signal connection is included in the repository.
Components

The diagram below shows the full wiring architecture — power distribution paths, signal connections, and communication links between all major components. A complete Fritzing source file is included in the /fritzing directory.
<img width="1135" height="850" alt="image" src="https://github.com/user-attachments/assets/1b96a7ca-cd71-429d-afea-c8e438d6c064" />
🔋 Battery — Li-ion 18650 (×4, 4S Pack)

The robot is powered by a 4S Li-ion battery pack consisting of four 3.7 V, 2200 mAh 18650 cells connected in series. The pack provides 14.8 V nominal voltage and reaches 16.3 V when fully charged. It stores 32.56 Wh, with approximately 22.79 Wh usable at 70% DoD. The measured runtime under full competition load is 95 minutes. The cells are rated for 2.2 A continuous discharge (1C) and 4.4 A burst discharge (2C), with a system low-voltage cut-off at 12.0 V.

⚙️ Power Regulation — LM2596 Buck Converters

Two LM2596 HW-411 buck converters regulate the battery voltage to 5 V. Rail A is dedicated to the MG996R steering servo, providing isolation from servo current spikes, with a measured efficiency of 84.5%. Rail B supplies the Raspberry Pi 5, ESP32, and all sensors, with a measured efficiency of 83.1%. Both rails operate from the 16.3 V maximum battery voltage and provide a regulated 5.00 V output.<img width="373" height="388" alt="image_2026-08-13_05-23-46" src="https://github.com/user-attachments/assets/05bbf027-07f7-4378-82e1-0deba515de37" />


🖥 Raspberry Pi 5 (8GB)

The Raspberry Pi 5 is the main computing unit and runs the Python/OpenCV vision and navigation pipeline on its quad-core ARM Cortex-A76 processor. It connects to the camera through CSI-2 and communicates with the ESP32 through UART. Its typical and peak current consumption is approximately 0.80 A and 1.80 A, respectively, and it is powered from Rail B.<img width="410" height="311" alt="image" src="https://github.com/user-attachments/assets/351d0768-748f-437a-afda-519403f3158d" />


📷 Raspberry Pi Camera Module 2

The Raspberry Pi Camera Module 2, based on the Sony IMX219 8 MP sensor, is connected to the Pi 5 through a CSI-2 ribbon cable. It captures colour images for real-time HSV-based obstacle classification. During validation, the system achieved 95% detection accuracy (19/20 correct detections) under approximately 500 lux fluorescent lighting.<img width="242" height="187" alt="image" src="https://github.com/user-attachments/assets/7ec3a039-d3a6-41e3-af06-708d382eae6d" />


🧠 ESP32-WROOM-32

The ESP32-WROOM-32 DevKit v1 handles all real-time control tasks, including motor PWM, steering control, and continuous I²C sensor polling. Its dual-core 240 MHz processor keeps motion control independent of the Raspberry Pi's Linux scheduling. The ESP32 operates at a sensor polling rate of 100 Hz, with typical and peak current consumption of approximately 0.080 A and 0.250 A, respectively.<img width="395" height="396" alt="image" src="https://github.com/user-attachments/assets/a43374c2-ab56-4acf-a9ce-2caf15ccc5a7" />


🚗 JSumo Titan DC Motor — 12 V / 1000 RPM

The JSumo Titan DC motor is powered directly from the battery through the BTS7960 driver. At a 72% PWM duty cycle, the effective motor voltage is approximately 11.20 V. During testing, the robot achieved a measured field speed of 1.5 m/s, with a typical motor load current of approximately 1.20 A and a measured stopping distance of 0.56 m.<img width="255" height="324" alt="image" src="https://github.com/user-attachments/assets/88be0846-46eb-45a7-b458-977fd0dbfba8" />



🔌 BTS7960B Motor Driver

The BTS7960B is a high-current MOSFET H-bridge used to control the drive motor in both directions through PWM. It is rated up to 43 A absolute maximum and accepts the ESP32's 3.3 V logic signals directly. The driver is connected directly to the battery and operates over the 12.0–16.8 V supply range.<img width="405" height="369" alt="image" src="https://github.com/user-attachments/assets/9bbb7c56-8c7b-47f9-83fb-5fe9a3c227b2" />


🔩 TowerPro MG996R Servo

The MG996R provides steering control and is powered separately from Rail A to isolate its current spikes from the logic electronics. It uses a 50 Hz PWM control signal, with a typical current of approximately 0.35 A and a measured stall current of 2.50 A. Its rated stall torque is approximately 9.4 kgf·cm at 4.8 V.<img width="494" height="362" alt="image" src="https://github.com/user-attachments/assets/e1ea97e1-f7fc-407f-a47f-6c59424287b5" />


🧭 BNO086 IMU

The BNO086 IMU is mounted near the geometric centre of the robot and provides heading and angular-rate information through the I²C bus. After zero-rate bias calibration to 0.018°/s, controlled 90° turns showed a validated heading error of 1.8°, while the total drift during a full competition run was approximately 3.2°. The IMU is powered from the ESP32's 3.3 V supply.<img width="447" height="447" alt="image" src="https://github.com/user-attachments/assets/f96e4599-1f87-4014-a026-dd00168dc6ff" />


📡 VL53L0X ToF Sensors (×4)

Four VL53L0X time-of-flight sensors provide distance measurements in the front, rear, left, and right directions. They share a single I²C bus, with individual XSHUT control used during startup to assign unique addresses from 0x30 to 0x33. The four sensors consume approximately 64 mA combined, and the measured distance error remains within ±12 mm across the specified operating range. They are powered from Rail B.<img width="371" height="371" alt="image" src="https://github.com/user-attachments/assets/20f0e167-cfc8-436b-be7a-b18961e4089d" />
