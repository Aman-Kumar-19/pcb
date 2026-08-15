# AeroCore-FC — System Architecture

**Document:** System Architecture
**Project:** AeroCore-FC — Custom Flight Controller PCB
**Revision:** Rev 0.1
**Status:** Preliminary Architecture
**Design Scope:** Hardware / PCB

---

# 1. Purpose

This document defines the electrical architecture of the AeroCore-FC custom flight controller.

The architecture translates the hardware requirements into functional subsystems and defines the interfaces between them.

The architecture shall be established before detailed component selection and schematic capture.

---

# 2. System Overview

AeroCore-FC is structured around a central high-performance microcontroller connected to:

* Inertial measurement hardware
* Barometric sensor
* GNSS/GPS interface
* USB interface
* ESC/motor control outputs
* Battery monitoring
* Current measurement
* Power-management circuitry
* Debug/programming interface
* Expansion interfaces

High-level architecture:

```text
                         ┌─────────────────────┐
                         │     GNSS / GPS      │
                         │       Module        │
                         └──────────┬──────────┘
                                    │ UART
                                    │
                                    ▼
┌──────────────┐              ┌──────────────────┐
│              │              │                  │
│    USB-C     │──── USB ────►│                  │
│              │              │                  │
└──────────────┘              │                  │
                              │                  │
┌──────────────┐              │    MCU           │
│              │    SPI       │                  │
│     IMU      │─────────────►│                  │
│              │              │                  │
└──────────────┘              │                  │
                              │                  │
┌──────────────┐      I²C/SPI │                  │
│  Barometer   │─────────────►│                  │
└──────────────┘              │                  │
                              │                  │
                              ├──── PWM/DShot ──► ESC 1
                              │
                              ├──── PWM/DShot ──► ESC 2
                              │
                              ├──── PWM/DShot ──► ESC 3
                              │
                              └──── PWM/DShot ──► ESC 4
                                     
                              ┌──────────────────┐
                              │ Battery Monitor  │
                              │ Voltage/Current  │
                              └────────┬─────────┘
                                       │
                                      ADC
                                       │
                                       ▼
                                      MCU


             ┌─────────────────────────────────┐
             │          POWER SYSTEM            │
             │                                 │
Battery ────►│ Protection → Regulation → Rails │
             │                  │              │
             │                  ├── 5 V        │
             │                  └── 3.3 V      │
             └─────────────────────────────────┘


                         ┌──────────────┐
                         │ SWD / DEBUG  │
                         └──────┬───────┘
                                │
                                ▼
                               MCU
```

---

# 3. Major Hardware Subsystems

The PCB shall be divided into the following functional blocks:

| Block            | Function                                  |
| ---------------- | ----------------------------------------- |
| MCU              | Central processing and peripheral control |
| IMU              | Accelerometer and gyroscope measurement   |
| Barometer        | Atmospheric pressure measurement          |
| GNSS Interface   | External GPS/GNSS communication           |
| USB              | PC connection and future data interface   |
| ESC Interface    | Four motor/ESC control outputs            |
| Power Input      | Battery/DC power entry                    |
| Power Management | Voltage conversion and filtering          |
| Battery Monitor  | Battery voltage measurement               |
| Current Monitor  | Battery current measurement               |
| Debug Interface  | MCU programming and debugging             |
| Expansion        | Future peripheral connectivity            |
| Status           | LEDs and basic hardware indication        |

---

# 4. Central Processing Architecture

The MCU is the central device of the system.

It shall interface directly with the majority of digital peripherals.

Conceptual architecture:

```text
                    ┌─────────────────────┐
                    │         MCU         │
                    │                     │
          ┌────────►│ SPI             TIM │──────► ESC 1
          │         │                     │──────► ESC 2
          │         │ I²C             TIM │──────► ESC 3
          │         │                     │──────► ESC 4
          │         │ UART                │
          │         │                     │
          │         │ USB                 │
          │         │                     │
          │         │ ADC                 │
          │         │                     │
          │         │ SWD                 │
          │         └─────────────────────┘
          │
          │
       ┌──┴───┐
       │  IMU │
       └──────┘
```

The MCU shall provide sufficient peripheral resources to prevent resource conflicts between mandatory interfaces.

Peripheral allocation shall be performed during the pin-mapping stage.

---

# 5. IMU Architecture

The IMU shall use SPI as the primary interface where supported.

```text
                 MCU
                  │
           ┌──────┼──────┐
           │      │      │
          SCK    MOSI   MISO
           │      │      │
           └──────┼──────┘
                  │
                 CS
                  │
                  ▼
            ┌───────────┐
            │    IMU    │
            │ ACC + GYRO│
            └───────────┘
```

### Architecture decisions

* Dedicated chip-select signal
* Short SPI routing
* Local decoupling
* Clean ground reference
* Placement away from switching power circuitry
* Optional interrupt connection to MCU

An IMU interrupt line shall be considered during pin allocation to support deterministic data acquisition.

---

# 6. Barometer Architecture

The barometer shall connect to the MCU using I²C or SPI depending on the selected device.

Preferred architecture:

```text
MCU
 │
 ├── SCL
 ├── SDA
 │
 ▼
Barometer
```

If SPI is selected:

```text
MCU
 │
 ├── SCK
 ├── MOSI
 ├── MISO
 └── CS
 │
 ▼
Barometer
```

The final interface shall be selected based on component availability and overall MCU peripheral allocation.

---

# 7. GNSS/GPS Architecture

The GNSS module shall be an external device.

Communication shall use UART.

```text
              AeroCore-FC
                  │
             ┌────┴────┐
             │   UART  │
             └────┬────┘
                  │
        ┌─────────┴─────────┐
        │                   │
      TX →                 ← RX
        │                   │
        └─────────┬─────────┘
                  │
              GNSS Module
```

The connector shall provide:

* TX
* RX
* GND
* Appropriate supply voltage

Optional GNSS control/reset signals may be provided if MCU resources permit.

---

# 8. USB Architecture

USB shall provide a direct interface between the MCU and host PC.

```text
                 USB-C
                   │
             ESD Protection
                   │
                   ▼
             USB D+ / D-
                   │
                   ▼
                  MCU
```

The USB section shall include:

* USB connector
* ESD protection
* VBUS protection
* Required pull-up/pull-down components
* Local decoupling
* Appropriate routing

USB differential routing shall remain referenced to a continuous ground plane.

---

# 9. ESC Interface Architecture

Four independent control channels shall be provided.

```text
                       MCU
                        │
            ┌───────────┼───────────┐
            │           │           │
           TIM         TIM         TIM
            │           │           │
            ▼           ▼           ▼

          ESC 1       ESC 2       ESC 3       ESC 4
            ▲           ▲           ▲           ▲
            └───────────┴───────────┴───────────┘
                         GND
```

Each ESC interface shall contain:

* Motor control signal
* Ground reference

Power for the ESC/motor system shall **not** be routed through the flight-controller logic power rails.

The battery/ESC power path shall remain electrically separate from low-current logic regulation except for the required common electrical reference.

---

# 10. Power Architecture

The power system shall be treated as a dedicated subsystem.

```text
                 BATTERY
                    │
                    ▼
          ┌───────────────────┐
          │ Input Protection  │
          └─────────┬─────────┘
                    │
                    ▼
          ┌───────────────────┐
          │ Input Filtering   │
          └─────────┬─────────┘
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
      5 V Regulator       3.3 V Regulator
          │                   │
          ▼                   ▼
     Peripherals        MCU / Sensors
```

The power architecture shall be designed around calculated load requirements rather than selecting regulators based only on nominal voltage.

---

# 11. Power-Domain Strategy

The architecture shall distinguish between:

### Battery domain

```text
VBAT
```

Used for:

* Battery measurement
* External battery-powered interfaces where applicable
* Input to regulators

### 5 V domain

Used for:

* Selected external peripherals
* GPS/GNSS where applicable
* USB-related power requirements
* Expansion

### 3.3 V domain

Used primarily for:

* MCU
* IMU
* Barometer
* Digital sensors
* Logic interfaces

The final power-domain assignment shall be verified against the selected component datasheets.

---

# 12. Battery Voltage Measurement

Battery voltage shall be measured through an ADC channel.

```text
VBAT
 │
 ▼
Protection / Filtering
 │
 ▼
Voltage Divider
 │
 ▼
ADC Input
 │
 ▼
MCU
```

The divider shall be designed so that the maximum expected battery voltage remains within the MCU ADC input range.

The ADC input shall include appropriate filtering where required.

---

# 13. Current Measurement Architecture

The current measurement subsystem shall measure battery current.

Conceptual architecture:

```text
Battery
   │
   ▼
Current Sensor
   │
   ├──────────────► Power System
   │
   └── Measurement Output
            │
            ▼
           ADC
            │
            ▼
           MCU
```

The exact implementation may use:

* Shunt + current-sense amplifier
* Integrated current-sense IC
* External current-sensor interface

Final selection will be made during component selection.

---

# 14. Debug Architecture

The MCU shall expose a dedicated SWD interface.

```text
             Debugger
                │
       ┌────────┼────────┐
       │        │        │
     SWCLK    SWDIO    NRST
       │        │        │
       └────────┼────────┘
                │
                ▼
               MCU
```

Required connections:

* SWCLK
* SWDIO
* NRST
* 3.3 V
* GND

The debug interface shall remain accessible after PCB assembly.

---

# 15. Expansion Architecture

The board shall expose selected unused MCU peripherals through expansion connectors/test points.

Target interfaces:

```text
┌───────────────────────┐
│ Expansion Interface   │
├───────────────────────┤
│ 3.3 V                 │
│ GND                   │
│ UART TX               │
│ UART RX               │
│ I²C SDA               │
│ I²C SCL               │
│ SPI SCK               │
│ SPI MOSI              │
│ SPI MISO              │
│ GPIO                  │
│ ADC                   │
└───────────────────────┘
```

Exact signals shall be finalized after MCU pin allocation.

---

# 16. Clock Architecture

The MCU clock architecture shall be based on the selected MCU manufacturer's recommended reference design.

The design shall consider:

* Main system clock
* External crystal/oscillator requirements
* USB clock requirements
* RTC requirements
* Clock accuracy
* Layout of crystal/oscillator circuitry

If an external crystal is required, it shall be placed close to the MCU with short connections and appropriate grounding considerations.

---

# 17. Reset and Boot Architecture

The MCU shall include:

* Hardware reset
* Required boot configuration
* Reset pull-up/pull-down components as specified by the MCU
* SWD access

Conceptual structure:

```text
                 3.3V
                  │
               Reset
                  │
                  ▼
                 MCU
                  │
               Boot
                  │
                  ▼
           Boot Configuration
```

The exact boot configuration shall follow the selected MCU reference design.

---

# 18. Power-On Architecture

The expected power-up sequence is:

```text
Battery Connected
       │
       ▼
Input Protection
       │
       ▼
Power Regulation
       │
       ├────► 5 V
       │
       └────► 3.3 V
                │
                ▼
             MCU Reset
                │
                ▼
             MCU Start
                │
                ▼
       Peripheral Initialization
```

Power-good monitoring shall be considered if required by the selected regulators or system architecture.

---

# 19. Ground and Return-Path Architecture

A continuous ground plane shall form the primary return path.

```text
L1 — Signals / Components
│
├── Signal
│
▼
L2 — Continuous GND
│
▼
L3 — Power
│
▼
L4 — Signals
```

High-current switching paths shall be physically minimized.

Sensitive sensor circuitry shall be placed away from noisy power-conversion loops.

No unnecessary ground-plane splits shall be introduced.

---

# 20. Noise-Sensitive vs Noisy Zones

The PCB shall be divided conceptually into:

### Sensitive zone

```text
IMU
Barometer
MCU
Low-level sensor signals
```

### Digital zone

```text
MCU
USB
UART
SPI
I²C
Debug
```

### Power/noisy zone

```text
Battery input
Protection
Switching regulator
Inductor
High-current paths
```

Preferred physical arrangement:

```text
┌───────────────────────────────────────┐
│                                       │
│        SENSITIVE / CONTROL            │
│                                       │
│     IMU       MCU       BARO          │
│                                       │
├───────────────────────────────────────┤
│                                       │
│        DIGITAL INTERFACES             │
│                                       │
│ USB     UART     SWD     EXPANSION    │
│                                       │
├───────────────────────────────────────┤
│                                       │
│          POWER SECTION                │
│                                       │
│ BATTERY → PROTECTION → REGULATORS     │
│                                       │
└───────────────────────────────────────┘
```

This is a conceptual placement strategy. Final placement shall be determined during PCB layout.

---

# 21. Interface Summary

| Subsystem       | Interface     | Direction      | Target    |
| --------------- | ------------- | -------------- | --------- |
| IMU             | SPI           | MCU ↔ IMU      | Mandatory |
| IMU Interrupt   | GPIO/EXTI     | IMU → MCU      | Preferred |
| Barometer       | I²C/SPI       | MCU ↔ Sensor   | Mandatory |
| GNSS            | UART          | MCU ↔ GNSS     | Mandatory |
| ESC 1           | Timer/PWM     | MCU → ESC      | Mandatory |
| ESC 2           | Timer/PWM     | MCU → ESC      | Mandatory |
| ESC 3           | Timer/PWM     | MCU → ESC      | Mandatory |
| ESC 4           | Timer/PWM     | MCU → ESC      | Mandatory |
| USB             | USB           | MCU ↔ Host     | Mandatory |
| Battery Voltage | ADC           | Monitor → MCU  | Mandatory |
| Current Sense   | ADC/Interface | Sensor → MCU   | Mandatory |
| SWD             | SWD           | Debugger ↔ MCU | Mandatory |
| Expansion UART  | UART          | MCU ↔ External | Mandatory |
| Expansion I²C   | I²C           | MCU ↔ External | Mandatory |
| Expansion SPI   | SPI           | MCU ↔ External | Desirable |

---

# 22. Preliminary Power Tree

The preliminary power tree is:

```text
                           VBAT
                             │
                             ▼
                    ┌────────────────┐
                    │ Input Protection│
                    └───────┬────────┘
                            │
                            ▼
                    ┌────────────────┐
                    │ Input Filtering │
                    └───────┬────────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
          ┌──────────────┐      ┌──────────────┐
          │   5 V Rail   │      │  3.3 V Rail  │
          └──────┬───────┘      └──────┬───────┘
                 │                     │
                 ▼                     ├── MCU
          External Devices             ├── IMU
                                       ├── Barometer
                                       └── Logic
```

The final topology may change after power-budget analysis.

---

# 23. Preliminary Data Architecture

```text
                    ┌──────────────┐
                    │    GNSS      │
                    └──────┬───────┘
                           │ UART
                           │
                           ▼
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│     IMU      │─SPI─►│              │◄─I²C─│  Barometer   │
└──────────────┘      │              │      └──────────────┘
                      │     MCU      │
┌──────────────┐      │              │
│ USB Host     │◄USB─►│              │
└──────────────┘      │              │
                      │              │
Battery Monitor ─ADC─►│              │
                      │              │
                      └──────┬───────┘
                             │
                     Timer/PWM/DShot
                             │
                  ┌──────────┼──────────┐
                  ▼          ▼          ▼
                 ESC1       ESC2       ESC3       ESC4
```

---

# 24. Architecture Decisions

The following decisions are established for the initial design:

| Decision            | Selection                        |
| ------------------- | -------------------------------- |
| PCB layers          | 4                                |
| Main controller     | High-performance STM32-class MCU |
| IMU interface       | SPI preferred                    |
| GNSS interface      | UART                             |
| ESC outputs         | 4                                |
| USB                 | Required                         |
| Debug               | SWD                              |
| Battery measurement | ADC                              |
| Current measurement | Required                         |
| Ground strategy     | Continuous ground plane          |
| ESC power stage     | External                         |
| PCB scope           | Flight controller only           |
| Firmware            | Out of scope for Rev 1           |

---

# 25. Items Pending Component Selection

The following shall remain open until the component-selection phase:

* Exact MCU part number
* MCU package
* Exact IMU
* Exact barometer
* Voltage regulators
* Current-sense architecture
* USB connector
* ESD protection devices
* GNSS connector
* ESC connectors
* Expansion connector
* Crystal/oscillator
* Flash/EEPROM, if required
* Battery connector
* Mounting-hole dimensions
* Final PCB dimensions
* Final power-tree topology

These shall be resolved using datasheets, electrical calculations, availability, lifecycle, package constraints, and PCB requirements.

---

