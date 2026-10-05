# Autonomous AI UGV: Hardware Design (KiCad)

Electrical schematics for an autonomous, AI-assisted unmanned ground vehicle (UGV) built around a **Raspberry Pi 4** (high-level AI / vision) and an **ESP32** (real-time control, sensors and radio). The repository contains the integrated system schematic plus the two sub-circuits it was merged from.

> Developed in the context of **TSYP14**. Edit this line if the repo targets something else.

---

## Repository contents

| File | Sheet | What it covers |
|------|-------|----------------|
| `UGV_AI_Robot_integrated.kicad_sch` | A2, rev **v10.0**, 2026-10-02 | **Full system**: power, Raspberry Pi 4, ESP32, motor drivers, sensors, LoRa, RFID, SD card, current monitor and servo driver. Every pin is wired through net labels. |
| `tsyp.kicad_sch` | A4 | **Motor and power sub-circuit**: ESP32-DevKitC-32E, four BTS7960 half-bridge modules driving two DC motors, INA219 current monitor, PCA9685 PWM driver and one servo, powered from a battery. |
| `tsyp_ras_circuit_commande.kicad_sch` | A4 | **Command link**: UART connection between the Raspberry Pi 4 and the ESP32-DevKitC-32E, with shared 5 V and GND. |

The integrated schematic is the one to use. Its title block states it merges `UGV_AI_Robot_wired`, `tsyp` (INA219, PCA9685, servo) and `tsyp_ras_circuit_commande` (RPi-ESP32 UART). The two smaller files are kept as references and as earlier iterations.

---

## System architecture

```
                 12 V battery (J1)
                        |
              INA219 shunt (IN+ / IN-)
                        |
        +---------------+-------------------+
        |                                   |
   LM2596 5 V buck                    2x BTS7960 (left / right)
        |                                   |
  +5 V rail                            2x DC motors (J2, J3)
  |  |  |  |
  |  |  |  +-- RPLiDAR A1M8, MQ-2, MQ-7, M6E Nano, SD module, 5 servos
  |  |  +----- ESP32 (5 V pin)  --> on-board 3V3 rail --> I2C sensors, LoRa
  |  +-------- Raspberry Pi 4B <== UART ==> ESP32
  +----------- BTS7960 logic supply
```

### Sections of the integrated sheet

1. Power regulation
2. High-level AI computer (Raspberry Pi 4B)
3. Motor drivers
4. ESP32 main controller
5. Sensors and communication buses
6. Current monitor and servo driver

---

## Main components

| Ref | Part | Role |
|-----|------|------|
| SBC1 | Raspberry Pi 4B | High-level AI, vision, mapping |
| CAM1 | Raspberry Pi Camera Module 3 | CSI camera on the Pi |
| SEN5 | RPLiDAR A1M8 | 2D LiDAR, USB to the Pi |
| U1 | ESP32-WROOM-32 (38-pin module) | Main real-time controller |
| REG1 | LM2596 5 V buck module | 12 V to 5 V regulation |
| U2 / U3 | BTS7960 modules (left / right) | High-current H-bridge motor drivers |
| J2 / J3 | Motor connectors | Left and right drive motors |
| U4 | INA219 | Battery voltage and current monitor |
| U5 | PCA9685 | 16-channel PWM driver (5 servos on LED0 to LED4) |
| M1 | Servo | Pan / actuator servo (LED0) |
| M2 to M5 | 4 additional servos | Extra actuators (LED1 to LED4) |
| SEN1 | MPU6050 | IMU |
| SEN2 | MLX90640 | Thermal camera array |
| SEN3 | MQ-2 | Combustible gas / smoke |
| SEN4 | MQ-7 | Carbon monoxide |
| MOD1 | RFM95W | LoRa radio (SPI) |
| MOD2 | M6E Nano | UHF RFID reader (UART) |
| MOD3 | Micro SD module | Data logging (SPI) |
| R1-R4 | 10 k / 18 k resistors | Voltage dividers for the MQ analog outputs |
| J1 | 2-pin connector | 12 V battery input |

---

## ESP32 pin map (integrated schematic)

| Function | ESP32 GPIO | Connected to |
|----------|-----------|--------------|
| I2C SDA | 21 | MPU6050, MLX90640, INA219, PCA9685 |
| I2C SCL | 22 | MPU6050, MLX90640, INA219, PCA9685 |
| SPI SCK / MISO / MOSI | 18 / 19 / 23 | RFM95W and SD module (shared bus) |
| LoRa chip select | 5 | RFM95W NSS |
| SD chip select | 4 | SD module CS |
| Left motor RPWM / LPWM | 32 / 33 | BTS7960 left (U2) |
| Right motor RPWM / LPWM | 25 / 26 | BTS7960 right (U3) |
| Motor enable | 27 | L_EN and R_EN of both BTS7960 |
| UART to Raspberry Pi (RX / TX) | 16 / 17 | Pi UART TXD / RXD (UART2) |
| UART to RFID reader (RX / TX) | 14 / 13 | M6E Nano TX / RX (UART1) |
| MQ-2 analog in | 34 | MQ-2 AO through 10 k / 18 k divider |
| MQ-7 analog in | 35 | MQ-7 AO through 10 k / 18 k divider |

### Servo channels (PCA9685)

| Servo | PCA9685 output | Signal net |
|-------|----------------|------------|
| M1 | LED0 | `SERVO_PWM` |
| M2 | LED1 | `SERVO2_PWM` |
| M3 | LED2 | `SERVO3_PWM` |
| M4 | LED3 | `SERVO4_PWM` |
| M5 | LED4 | `SERVO5_PWM` |

Each servo takes its signal from the PCA9685, +5 V from the rail and GND. The four new servos (M2 to M5) are documented here first; add them to `UGV_AI_Robot_integrated.kicad_sch` so the schematic matches. Net names for M2 to M5 are suggestions, so rename them if you prefer.

### I2C addresses

| Device | Address |
|--------|---------|
| PCA9685 | `0x40` |
| INA219 | `0x41` (A0 tied to VS) |
| MPU6050 | `0x68` |
| MLX90640 | `0x33` |

### Raspberry Pi connections

| Link | Details |
|------|---------|
| ESP32 <-> Pi | UART (`UART_RPI_TX` / `UART_RPI_RX`), 3.3 V logic |
| LiDAR | RPLiDAR A1M8 over USB |
| Camera | Camera Module 3 over CSI |
| Power | 5 V from the LM2596 rail |

---

## Power

| Rail | Source | Loads |
|------|--------|-------|
| `+12V` / `BATT+` | Battery via J1, measured by the INA219 shunt | BTS7960 motor supply (B+), LM2596 input |
| `+5V` | LM2596 buck | Raspberry Pi, ESP32 5 V pin, RPLiDAR, MQ-2, MQ-7, M6E Nano, SD module, BTS7960 logic, 5 servos |
| `+3V3` | ESP32 on-board regulator | MPU6050, MLX90640, RFM95W, INA219, PCA9685 logic |

`PWR_FLAG` symbols are placed on the battery rails so the ERC passes cleanly.

---

## Sub-circuit notes

### `tsyp.kicad_sch`: motor and power

Four BTS7960 modules are used as individual half-bridges, two per DC motor, each controlled with one `IN` and one `INH` line:

| Motor | Module | IN | INH |
|-------|--------|----|-----|
| M2 | U1 | IO23 | IO22 |
| M2 | U2 | IO19 | IO18 |
| M1 | U5 | IO5 | IO17 |
| M1 | U4 | IO16 | IO4 |

The INA219 and PCA9685 share I2C (SDA on IO21, SCL on IO0), with the PCA9685 `OE` on IO2 and the servo on `LED0`. The battery feeds the driver `VS` pins through the INA219 shunt.

### `tsyp_ras_circuit_commande.kicad_sch`: Pi to ESP32 link

The Pi UART pins (GPIO14 / GPIO15) go to ESP32 IO14 / IO12, with 5 V and GND shared between the boards. The integrated schematic supersedes this wiring and moves the link to ESP32 UART2 (GPIO16 / GPIO17).

---

## Opening the project

- Tool: **KiCad 10** (files generated by Eeschema, format version `20260306`).
- Open any `.kicad_sch` file directly in the Schematic Editor, or create a KiCad project in this folder and add `UGV_AI_Robot_integrated.kicad_sch` as the root sheet.
- Some symbols come from custom libraries (`tsyp`, `tsyp_new_library`, and project-embedded symbols such as the BTS7960 and RPLiDAR). They are embedded in the schematic files, so no extra library setup is needed to view them.

---

## Design notes and things to verify before building

- **5 V budget**: the Raspberry Pi 4, RPLiDAR, M6E Nano, MQ heaters, SD module and now five servos would share one LM2596 rail. Servo stall currents add up quickly, so power the servos from a dedicated 5 V regulator (grounds tied together) and keep the LM2596 for the logic loads.
- **MQ sensor outputs** are 5 V analog. The 10 k / 18 k dividers bring them to about 3.2 V for the ESP32 ADC pins (GPIO34 / 35, input-only).
- **I2C pull-ups** are not drawn on the integrated sheet. Make sure the breakout boards provide them, or add 4.7 k resistors on SDA and SCL.
- **Motor enable** is a single shared line (`M_EN`, GPIO27) for all four BTS7960 enable pins, so one pin cuts both drive motors.
- **ESP32 strapping pins**: the `tsyp.kicad_sch` sub-circuit uses IO0 for I2C SCL, which is a boot-strapping pin. The integrated design avoids this by using GPIO22.
- Unused PCA9685 outputs (`LED5` to `LED15`) are left free for further servos.

---

## Status

- Schematic capture: complete for the integrated system (v10.0).
- PCB layout: not included.
- Firmware: not included in this repository.
