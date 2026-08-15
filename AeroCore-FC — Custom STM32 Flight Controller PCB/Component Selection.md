# AeroCore-FC — Component Selection

**Document:** Component Selection
**Project:** AeroCore-FC — Custom Flight Controller PCB
**Revision:** Rev 0.1
**Status:** Preliminary Component Selection
**Design Scope:** Hardware / PCB

---

# 1. Purpose

This document defines the preliminary component selection for the AeroCore-FC flight controller PCB.

Components shall be selected based on:

* Electrical requirements
* Interface compatibility
* Package availability
* PCB layout requirements
* Power consumption
* Thermal performance
* Component lifecycle
* Availability
* Manufacturer documentation
* Recommended application circuits
* Recommended PCB layout
* Cost
* Manufacturing suitability

The components listed in this document shall be verified against the latest manufacturer datasheets before schematic release.

---

# 2. Component Selection Philosophy

Component selection shall follow the order:

```text
System Requirement
       ↓
Electrical Requirement
       ↓
Candidate Components
       ↓
Datasheet Review
       ↓
Package / Layout Review
       ↓
Availability Review
       ↓
Design Calculation
       ↓
Final Selection
```

A component shall not be selected solely because it is commonly used in existing flight controllers.

---

# 3. Main MCU

## 3.1 Target Component

**STM32H743**

The exact ordering part number shall be finalized after package, memory, GPIO, peripheral, and availability analysis.

### Required characteristics

* ARM Cortex-M7
* High clock performance
* Hardware timers
* Multiple UART/USART interfaces
* SPI
* I²C
* ADC
* DMA
* USB
* SWD
* Sufficient GPIO
* Sufficient Flash
* Sufficient RAM

### Why STM32H7

The STM32H7 family provides significantly more processing and peripheral capability than required for the first PCB revision.

This provides margin for future firmware development and additional interfaces without requiring an immediate MCU redesign.

### MCU selection criteria

| Parameter        | Requirement                            |
| ---------------- | -------------------------------------- |
| CPU architecture | ARM Cortex-M class                     |
| Performance      | High-performance MCU                   |
| Flash            | Sufficient for future firmware         |
| RAM              | Sufficient for control/data processing |
| UART             | ≥ 3 preferred                          |
| SPI              | ≥ 2 preferred                          |
| I²C              | ≥ 2 preferred                          |
| ADC              | Required                               |
| Timer channels   | ≥ 4 motor outputs                      |
| USB              | Required                               |
| Debug            | SWD                                    |
| Supply           | 3.3 V logic                            |
| Package          | Suitable for compact 4-layer PCB       |

---

# 4. IMU

## 4.1 Target Component

**TDK InvenSense ICM-42688-P**

### Function

Provides:

* 3-axis accelerometer
* 3-axis gyroscope

### Interface

**SPI**

### Selection rationale

The ICM-42688-P is a modern low-power IMU suitable for high-rate inertial sensing applications.

SPI is preferred because the IMU is a high-priority sensor and should have a fast, deterministic communication path to the MCU.

### PCB requirements

The IMU layout shall follow the manufacturer's recommendations.

Particular attention shall be given to:

* Decoupling
* Grounding
* SPI routing
* Mechanical placement
* Vibration
* Thermal isolation
* Distance from switching regulators

### Status

**Candidate — requires final datasheet/package verification.**

---

# 5. Barometric Pressure Sensor

## 5.1 Target Component

**Bosch BMP390**

### Function

Atmospheric pressure measurement.

### Interface

* I²C
* SPI

### Preferred interface

I²C initially.

SPI may be used if required by the final MCU peripheral allocation.

### Selection rationale

The BMP390 provides a compact digital pressure-sensing solution suitable for the intended board architecture.

### PCB requirements

The barometer shall:

* Be located away from switching regulators.
* Avoid unnecessary heat sources.
* Have appropriate decoupling.
* Follow the manufacturer's recommended pressure-port/layout requirements.

### Status

**Candidate — requires final datasheet/package verification.**

---

# 6. GNSS/GPS Interface

The GNSS module will initially be **external to the PCB**.

The flight controller will provide an electrical interface rather than integrating the GNSS receiver.

### Interface

UART.

### Connector signals

```text
1. VCC
2. GND
3. UART_TX
4. UART_RX
5. Optional RESET
```

The final connector shall be selected based on:

* Current requirement
* Cable type
* Mechanical reliability
* Board space
* Pitch
* Availability

---

# 7. USB Connector

## Target

**USB Type-C receptacle**

USB-C is preferred because it is mechanically robust and widely available.

The final connector shall be selected based on:

* Through-hole vs SMT mounting
* Mechanical retention
* USB 2.0 compatibility
* Board-edge placement
* Availability
* Assembly capability

### USB subsystem

```text
USB-C
  │
  ├── VBUS
  ├── D+
  ├── D-
  └── GND
```

The USB design shall include appropriate ESD protection and configuration components according to the selected MCU/USB implementation.

---

# 8. USB ESD Protection

A dedicated low-capacitance USB ESD protection device shall be used.

### Selection requirements

The device shall:

* Support USB 2.0 signals
* Have low parasitic capacitance
* Protect D+ and D-
* Be placed close to the USB connector

The exact part number shall be selected after confirming the USB implementation.

---

# 9. Power Architecture

The power system shall generate:

```text
VBAT
 │
 ├── Battery Monitoring
 │
 └── Regulation
       │
       ├── 5 V
       │
       └── 3.3 V
```

The regulators shall be selected after calculating:

* Maximum load current
* Typical load current
* Peak transient current
* Thermal dissipation
* Efficiency
* Input voltage range

---

# 10. 3.3 V Regulator

The 3.3 V rail supplies the main digital and sensor electronics.

### Target requirements

* Suitable input voltage range for the chosen power architecture
* Output: 3.3 V
* Adequate load current
* Low output noise
* Good transient response
* Thermal margin
* Appropriate protection
* Available in a PCB-friendly package

### Preferred topology

A switching regulator may be used if the input-to-output voltage difference and current requirements justify it.

If the current requirement is sufficiently low and noise is a greater concern, an LDO may be considered.

The final architecture shall be determined using power calculations.

---

# 11. 5 V Regulator

The 5 V rail will primarily support external peripherals and selected interfaces.

### Requirements

* Regulated 5 V output
* Adequate current capability
* Suitable input voltage range
* Thermal margin
* Good efficiency
* Appropriate protection

The regulator topology shall be selected based on the battery-input range and calculated power requirements.

---

# 12. Power Input Protection

The battery input shall include appropriate protection.

Potential protection functions:

```text
Battery
  │
  ▼
Reverse Polarity Protection
  │
  ▼
Transient Protection
  │
  ▼
Input Filtering
  │
  ▼
Regulators
```

The exact protection topology shall be determined after the target battery voltage range is established.

---

# 13. Reverse Polarity Protection

A MOSFET-based reverse-polarity protection circuit is preferred over a simple series diode where efficiency and voltage drop are important.

The final implementation shall consider:

* Maximum battery voltage
* Maximum input current
* MOSFET RDS(on)
* Power dissipation
* Gate-drive requirements
* Failure behavior

---

# 14. Transient Protection

A TVS diode or equivalent transient-protection strategy shall be considered at the battery input.

The selected device shall be based on:

* Maximum operating voltage
* Battery voltage range
* Expected transient environment
* Clamping voltage
* Peak pulse power

The final device shall be selected after defining the battery-input operating range.

---

# 15. Battery Voltage Measurement

The battery voltage will be measured using a resistor divider connected to an MCU ADC.

### Architecture

```text
VBAT
 │
 R1
 │
 ├──────► ADC
 │
 R2
 │
GND
```

The resistor ratio shall be calculated based on the maximum battery voltage and MCU ADC input range.

A capacitor may be added at the ADC node to form an RC low-pass filter.

---

# 16. Current Measurement

The initial architecture will use a dedicated current-sensing solution rather than routing the high-current battery path through the MCU PCB.

Preferred options:

### Option A — External current sensor

```text
Battery
   │
   ▼
External Current Sensor
   │
   ├──── Power
   │
   └──── Measurement
             │
             ▼
            ADC
```

### Option B — On-board shunt

```text
Battery
   │
   ▼
Shunt
   │
   ▼
Power System

Shunt Voltage
   │
   ▼
Current Sense Amplifier
   │
   ▼
MCU ADC
```

Option B will be considered only if the PCB power architecture can safely accommodate the required current.

---

# 17. Debug Connector

The PCB shall provide an SWD interface.

### Required signals

```text
SWDIO
SWCLK
NRST
3.3V
GND
```

A compact connector or test-point arrangement may be used.

The connector shall remain accessible during development and initial board validation.

---

# 18. Clock Components

The MCU clock architecture shall use the manufacturer's recommended oscillator configuration.

Potential components:

* External crystal
* Load capacitors
* RTC crystal, if required

Final oscillator frequency and components shall be determined from the selected MCU configuration and USB/clock requirements.

---

# 19. Decoupling Capacitors

Decoupling capacitors shall be selected according to the requirements of each IC.

Typical categories:

| Location      | Typical Function                |
| ------------- | ------------------------------- |
| MCU VDD       | High-frequency local decoupling |
| MCU bulk rail | Local energy storage            |
| IMU           | Sensor supply filtering         |
| Barometer     | Sensor supply filtering         |
| Regulators    | Input/output stability          |
| USB           | Local supply filtering          |

Actual capacitor values shall be taken from manufacturer recommendations and regulator stability requirements.

---

# 20. Resistors

Resistors shall be selected for:

* Pull-up/pull-down
* Voltage divider
* Current limiting
* Configuration
* Termination where required
* LED drive

Precision requirements shall be determined per circuit.

For the battery divider, resistor tolerance shall be included in the voltage-measurement error calculation.

---

# 21. Status LEDs

The PCB shall provide basic hardware status indication.

Target LEDs:

```text
LED 1 — Power
LED 2 — MCU / Status
LED 3 — User / Debug
```

LED current shall be kept low to minimize unnecessary power consumption.

---

# 22. ESC Connectors

Four ESC interfaces shall be provided.

Initial connector requirement:

```text
ESC1 — Signal + GND
ESC2 — Signal + GND
ESC3 — Signal + GND
ESC4 — Signal + GND
```

Connector selection shall consider:

* Pitch
* Mechanical strength
* Cable compatibility
* Assembly method
* Board area
* Availability

---

# 23. Expansion Connector

An expansion connector shall expose selected MCU resources.

Target signals:

```text
3.3V
GND
UART_TX
UART_RX
I2C_SDA
I2C_SCL
SPI_SCK
SPI_MOSI
SPI_MISO
GPIO
ADC
```

Exact pin assignment shall be defined after MCU pin mapping.

---

# 24. Mounting Hardware

The PCB shall include four mechanical mounting holes.

Target mounting pattern:

**To be finalized during mechanical design.**

The mounting-hole diameter and pattern shall be selected based on the intended flight-controller mounting ecosystem and mechanical constraints.

Mounting holes shall include appropriate copper clearance and mechanical keep-outs.

---

# 25. Preliminary Component List

| Ref   | Function           | Target Component | Status    |
| ----- | ------------------ | ---------------- | --------- |
| U1    | Main MCU           | STM32H743 family | Candidate |
| U2    | IMU                | ICM-42688-P      | Candidate |
| U3    | Barometer          | BMP390           | Candidate |
| U4    | 3.3 V regulator    | TBD              | Open      |
| U5    | 5 V regulator      | TBD              | Open      |
| U6    | USB ESD            | TBD              | Open      |
| U7    | Current sensor     | TBD              | Open      |
| J1    | USB                | USB-C receptacle | Candidate |
| J2    | GNSS               | TBD              | Open      |
| J3    | ESC1               | TBD              | Open      |
| J4    | ESC2               | TBD              | Open      |
| J5    | ESC3               | TBD              | Open      |
| J6    | ESC4               | TBD              | Open      |
| J7    | SWD                | TBD              | Open      |
| J8    | Expansion          | TBD              | Open      |
| D1-D3 | Status LEDs        | TBD              | Open      |
| Q1    | Reverse protection | TBD              | Open      |
| D4    | TVS                | TBD              | Open      |

---

# 26. Component Selection Verification

Before a component is marked **Final**, the following shall be checked:

* [ ] Manufacturer datasheet reviewed
* [ ] Absolute maximum ratings verified
* [ ] Operating conditions verified
* [ ] Supply voltage verified
* [ ] Interface compatibility verified
* [ ] Package availability verified
* [ ] Recommended PCB layout reviewed
* [ ] Recommended application circuit reviewed
* [ ] Decoupling requirements verified
* [ ] Thermal requirements verified
* [ ] Manufacturer lifecycle checked
* [ ] Distributor availability checked
* [ ] Alternate component identified where practical

---

# 27. Component Lifecycle and Availability

For critical components, preference shall be given to parts with:

* Active manufacturer status
* Long-term availability
* Multiple authorized distribution channels
* Established manufacturer documentation
* Suitable second-source alternatives where practical

Critical components shall not be finalized based solely on current stock availability.

---

# 28. Design-for-Assembly Considerations

Component selection shall consider the complete manufacturing process.

Preferred characteristics:

* Standard SMT packages
* Reasonable component density
* Avoid unnecessary ultra-small packages
* Minimize unique component types where practical
* Avoid obsolete packages
* Maintain sufficient PCB assembly clearance
* Provide accessible test points

Fine-pitch components shall only be used where their electrical or functional benefits justify the additional assembly complexity.

---

# 29. Final Component Selection Criteria

A component shall be considered **Final** only when all of the following are satisfied:

```text
Electrical Compatibility
        +
Mechanical Compatibility
        +
PCB Layout Compatibility
        +
Thermal Compatibility
        +
Availability
        +
Manufacturability
        +
Cost
        ↓
     FINAL
```

---

