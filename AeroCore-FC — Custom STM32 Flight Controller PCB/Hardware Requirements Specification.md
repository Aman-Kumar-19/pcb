# AeroCore-FC — Hardware Requirements Specification

**Document:** Hardware Requirements Specification
**Project:** AeroCore-FC — Custom Flight Controller PCB
**Revision:** Rev 0.1
**Status:** Initial Requirements
**Design Scope:** Hardware / PCB
**Target PCB:** 4-Layer FR-4

---

## 1. Purpose

The AeroCore-FC project is the design of a custom, compact flight controller PCB intended for a quadcopter platform.

The objective is to develop the complete hardware design from system requirements through schematic capture, PCB layout, design verification, and manufacturing documentation.

The design will focus on:

* Embedded controller hardware
* Inertial sensing
* Power management
* Motor/ESC interfaces
* External communication interfaces
* Battery monitoring
* Debug and programming
* PCB signal and power integrity
* Manufacturability

Firmware, flight-control algorithms, and autonomous navigation are outside the scope of the initial hardware design.

---

## 2. Design Objectives

The flight controller shall:

1. Provide a high-performance microcontroller suitable for real-time flight-control applications.
2. Acquire inertial measurements using a 6-axis IMU.
3. Provide a barometric pressure sensor interface.
4. Provide interfaces for GPS and external peripherals.
5. Provide four independent ESC/motor control outputs.
6. Provide USB connectivity for development and configuration.
7. Provide a dedicated programming/debug interface.
8. Monitor battery voltage and support battery current measurement.
9. Provide regulated and filtered power rails for digital and sensor circuitry.
10. Maintain a PCB architecture suitable for low-noise sensor operation.
11. Use a manufacturable 4-layer PCB structure.
12. Provide sufficient expansion capability for future hardware revisions.

---

# 3. Functional Requirements

| ID     | Requirement                                                                                                             | Priority  |
| ------ | ----------------------------------------------------------------------------------------------------------------------- | --------- |
| FR-001 | The PCB shall contain a 32-bit microcontroller.                                                                         | Mandatory |
| FR-002 | The microcontroller shall provide sufficient processing performance for future real-time flight-control firmware.       | Mandatory |
| FR-003 | The PCB shall contain a 6-axis IMU providing 3-axis accelerometer and 3-axis gyroscope measurements.                    | Mandatory |
| FR-004 | The primary IMU interface shall use SPI where supported by the selected device.                                         | Mandatory |
| FR-005 | The PCB shall contain a barometric pressure sensor.                                                                     | Mandatory |
| FR-006 | The PCB shall provide a dedicated UART interface for a GNSS/GPS module.                                                 | Mandatory |
| FR-007 | The PCB shall provide at least four independent motor/ESC control outputs.                                              | Mandatory |
| FR-008 | Motor/ESC outputs shall be connected to hardware timer/PWM-capable MCU pins.                                            | Mandatory |
| FR-009 | The PCB shall provide USB connectivity to a host computer.                                                              | Mandatory |
| FR-010 | The PCB shall provide a dedicated SWD/JTAG-compatible programming and debug interface appropriate for the selected MCU. | Mandatory |
| FR-011 | The PCB shall provide battery-voltage measurement through an MCU ADC input.                                             | Mandatory |
| FR-012 | The PCB shall provide an interface for battery-current measurement.                                                     | Mandatory |
| FR-013 | The PCB shall provide regulated 5 V and 3.3 V power rails as required by the selected circuitry.                        | Mandatory |
| FR-014 | The PCB shall provide status indication through onboard LEDs.                                                           | Mandatory |
| FR-015 | The PCB shall provide at least one spare UART interface for future expansion.                                           | Mandatory |
| FR-016 | The PCB shall provide an I²C expansion interface.                                                                       | Mandatory |
| FR-017 | The PCB shall provide an SPI expansion capability where practical.                                                      | Desirable |
| FR-018 | The PCB shall provide spare GPIO/ADC resources for future hardware revisions.                                           | Desirable |
| FR-019 | The PCB shall provide appropriate external-interface protection and power filtering.                                    | Mandatory |
| FR-020 | The PCB shall provide test points for critical power rails and important debug signals.                                 | Mandatory |

---

# 4. Microcontroller Requirements

The selected MCU shall satisfy the following minimum requirements:

* 32-bit ARM-based MCU or equivalent high-performance architecture
* Sufficient CPU performance for real-time control applications
* Adequate Flash memory for future firmware
* Adequate RAM for control algorithms and data buffering
* Multiple hardware timers
* Multiple UART/USART interfaces
* SPI interfaces
* I²C interfaces
* ADC capability
* DMA capability
* USB capability
* Hardware debugging/programming interface
* Sufficient GPIO resources
* 3.3 V-compatible digital interfaces or appropriate level translation

The final MCU part number shall be selected during the **Component Selection** phase.

---

# 5. Inertial Measurement Requirements

The IMU is a critical subsystem and shall provide:

* 3-axis accelerometer
* 3-axis gyroscope
* Digital interface
* SPI interface preferred
* Suitable measurement range for multirotor applications
* Suitable sampling rate for future control-loop requirements
* Low-noise operation
* Low-power operation
* 3.3 V-compatible operation where possible

The IMU shall be positioned to minimize the influence of:

* Switching regulator noise
* High-current PCB traces
* Electromagnetic interference
* Mechanical vibration
* Thermal gradients

The final IMU shall be selected based on electrical performance, availability, package, lifecycle, and PCB-layout recommendations.

---

# 6. Barometric Sensor Requirements

The barometric subsystem shall:

* Measure atmospheric pressure
* Support altitude estimation in future firmware
* Provide a digital interface
* Support I²C and/or SPI
* Operate from an available regulated supply
* Include the manufacturer-recommended decoupling and layout provisions

The sensor shall be physically positioned away from major heat sources and airflow disturbances where practical.

---

# 7. GNSS/GPS Interface Requirements

The PCB shall provide an external GNSS/GPS module interface.

Minimum interface:

```text
3.3 V / 5 V* 
GND
UART_TX
UART_RX
```

* Final voltage shall depend on the selected GNSS module and interface requirements.

The interface shall include appropriate power filtering and signal protection where required.

---

# 8. ESC / Motor Interface Requirements

The PCB shall provide four independent ESC control outputs:

```text
Motor 1 / ESC 1
Motor 2 / ESC 2
Motor 3 / ESC 3
Motor 4 / ESC 4
```

Requirements:

* Each output shall be connected to a hardware timer-capable MCU pin.
* Outputs shall support hardware PWM.
* GPIO assignment shall allow future digital ESC protocols such as DShot where supported by the firmware architecture.
* Connector arrangement shall minimize wiring ambiguity.
* Ground reference shall be provided for external ESC connections.

The flight controller shall **not contain the ESC power stage** in the initial revision.

---

# 9. USB Requirements

The PCB shall provide a USB interface for development and future firmware functionality.

Requirements:

* USB connector suitable for the board form factor
* USB data lines routed according to the selected MCU and USB specification requirements
* USB ESD protection
* Controlled and continuous ground reference beneath high-speed USB routing
* Appropriate VBUS protection
* USB connector mechanical support

USB functionality shall be verified against the selected MCU's hardware reference design.

---

# 10. Power Input Requirements

The PCB shall accept power from an external battery/DC source suitable for the intended flight-controller application.

The power subsystem shall include, as applicable:

* Input protection
* Reverse-polarity protection
* Transient protection
* Input filtering
* Voltage regulation
* Local decoupling
* Power test points

Target power architecture:

```text
                 BATTERY INPUT
                       │
                       ▼
              Input Protection
                       │
                       ▼
                Input Filtering
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
             5 V              3.3 V
              │                 │
        Peripherals       MCU / Sensors
```

The final regulator topology and components shall be determined during the power-tree design.

---

# 11. Power Rail Requirements

The design shall provide the required regulated rails for all onboard components.

### Target rails

| Rail  | Intended Use                              |
| ----- | ----------------------------------------- |
| VBAT  | Battery input / battery monitoring        |
| 5 V   | External peripherals and selected devices |
| 3.3 V | MCU, IMU, sensors and digital peripherals |

Each rail shall be evaluated for:

* Maximum expected current
* Transient current
* Voltage tolerance
* Power dissipation
* Efficiency
* Thermal performance
* Noise
* Decoupling requirements

Power-budget calculations shall be completed before final regulator selection.

---

# 12. Battery Voltage Measurement

Battery voltage shall be monitored by the MCU.

Target architecture:

```text
VBAT
 │
 ▼
Resistive Divider
 │
 ▼
Filtering
 │
 ▼
MCU ADC
```

The voltage-divider ratio shall be selected based on:

* Maximum expected battery voltage
* MCU ADC input range
* ADC accuracy
* Resistor power rating
* Measurement resolution

Component values shall be calculated during schematic design.

---

# 13. Current Measurement

The PCB shall provide battery-current measurement capability.

The design shall support connection to an appropriate current-sensing circuit/device.

The selected architecture shall be evaluated for:

* Measurement range
* Accuracy
* Bandwidth
* Power dissipation
* ADC/interface compatibility
* PCB placement
* Grounding requirements

The current-sensing implementation shall be finalized during component selection and power-system design.

---

# 14. Protection Requirements

The PCB shall include appropriate protection for external interfaces and power inputs.

Protection shall be evaluated for:

### Power

* Reverse polarity
* Input transients
* Overvoltage where required
* ESD where applicable

### USB

* ESD protection
* VBUS protection

### External interfaces

Protection requirements shall be evaluated based on:

* Connector accessibility
* Cable length
* External module type
* Signal voltage
* Expected electrical environment

Protection components shall not be added without considering their effect on signal integrity and power loss.

---

# 15. PCB Requirements

The PCB shall use a **4-layer FR-4 stack-up** as the initial design target.

Proposed layer structure:

| Layer | Function                              |
| ----- | ------------------------------------- |
| L1    | Components + critical signals         |
| L2    | Solid GND plane                       |
| L3    | Power distribution + selected signals |
| L4    | Signals                               |

The final stack-up shall be confirmed against the selected PCB manufacturer.

---

# 16. Grounding Requirements

The PCB shall use a continuous ground-reference strategy.

Requirements:

* L2 shall preferably remain a continuous ground plane.
* High-current return paths shall be controlled.
* Sensitive sensor ground paths shall avoid unnecessary coupling with noisy switching-current paths.
* High-speed signals shall maintain appropriate return paths.
* Ground vias shall be used where required to maintain return continuity.
* Analog/digital ground separation shall only be implemented where technically justified.

The design shall avoid unnecessary split ground planes.

---

# 17. PCB Placement Requirements

Component placement shall prioritize electrical performance before aesthetic considerations.

Priority shall be:

1. IMU placement
2. MCU placement
3. Power architecture
4. High-speed interfaces
5. External connectors
6. Supporting components

The following shall be kept physically separated where practical:

```text
High-noise / high-current
        │
        │
        ▼
Power conversion
        │
        │
        X
        │
        │
        ▼
Sensitive sensing
IMU / Barometer
```

The IMU shall not be placed adjacent to switching regulators, inductors, high-current battery paths, or other major noise sources.

---

# 18. Decoupling Requirements

Every IC shall receive the manufacturer's recommended local bypass/decoupling network.

Decoupling design shall consider:

* Capacitor value
* Capacitor type
* Voltage rating
* ESR
* Placement
* Trace/via inductance
* Current return path

Decoupling capacitors shall be placed as close as practical to their respective power pins.

---

# 19. Signal Integrity Requirements

PCB routing shall consider:

* Signal return paths
* Trace length
* Crosstalk
* Ground reference
* Via transitions
* Differential routing where applicable
* High-speed interface requirements

USB and other high-speed interfaces shall follow the selected MCU/component manufacturer's layout recommendations.

---

# 20. Thermal Requirements

Thermal performance shall be evaluated for:

* Main voltage regulators
* Power-management components
* MCU
* Current-sensing components
* Any device with significant power dissipation

Copper area, thermal vias, and component placement shall be used where necessary to maintain acceptable operating temperatures.

---

# 21. Mechanical Requirements

The PCB shall be designed as a compact rectangular flight-controller board.

Initial target dimensions:

**Approximately 36 × 36 mm to 50 × 50 mm**

Final dimensions shall be determined after:

* Component selection
* Connector selection
* Mounting-hole definition
* Placement
* Routing

The design should target compatibility with commonly used flight-controller mounting patterns where practical.

---

# 22. Manufacturing Requirements

The PCB shall be designed for standard commercial PCB fabrication.

Target characteristics:

* 4-layer FR-4
* Standard copper thickness
* Standard through-hole vias
* Standard SMT manufacturing
* Lead-free assembly compatible
* Manufacturable trace widths and clearances
* No unnecessary exotic fabrication processes

The final design rules shall be based on the selected PCB manufacturer's capabilities.

---

# 23. Testability Requirements

The PCB shall include accessible test points for critical nodes.

Minimum recommended test points:

```text
VBAT
5V
3.3V
GND
NRST
SWDIO
SWCLK
USB VBUS
```

Additional test points shall be added for important subsystem signals where practical.

The PCB shall allow electrical validation before firmware integration.

---

# 24. Design Verification Requirements

Before manufacturing release, the following shall be completed:

* Schematic ERC
* PCB DRC
* Netlist verification
* Footprint verification
* 3D mechanical inspection
* Datasheet pinout verification
* Power-budget calculation
* Regulator thermal evaluation
* Decoupling review
* MCU boot configuration review
* SWD connectivity verification
* USB routing review
* IMU placement review
* Ground-plane review
* Manufacturing-rule verification
* BOM review
* Component availability review

---

# 25. Manufacturing Deliverables

The final hardware release shall contain:

```text
AeroCore-FC.kicad_sch
AeroCore-FC.kicad_pcb
BOM
Gerber files
Drill files
Pick-and-place files
Fabrication drawing
Assembly drawing
Schematic PDF
3D model/render
ERC report
DRC report
```

---

# 26. Scope Definition

## In Scope

* Hardware architecture
* Component selection
* Power-tree design
* MCU hardware
* IMU hardware
* Barometer
* GNSS interface
* USB interface
* ESC interfaces
* Battery monitoring
* Current sensing
* SWD
* PCB stack-up
* Component placement
* PCB routing
* Signal integrity considerations
* Power integrity considerations
* DRC/ERC
* BOM
* Manufacturing documentation

## Out of Scope — Revision 1

* Flight-control firmware
* PID implementation
* Sensor-fusion algorithms
* Autonomous navigation
* ESC power-stage design
* Motor design
* Drone frame
* Camera processing
* AI/ML
* Flight testing

---

# 27. Engineering Success Criteria

The AeroCore-FC PCB will be considered ready for fabrication when:

1. All mandatory functional requirements are satisfied.
2. The complete schematic has passed ERC.
3. The PCB has passed DRC.
4. All critical components and footprints have been verified against datasheets.
5. Power-tree calculations have been completed.
6. Critical power and signal routing has been reviewed.
7. IMU placement and grounding have been reviewed.
8. The PCB is manufacturable using the selected fabrication process.
9. BOM components have verified manufacturer part numbers and suitable availability.
10. Complete fabrication and assembly documentation has been generated.

---

# 28. Revision Strategy

The project will follow an incremental hardware-revision approach.

### Rev 0.1

Requirements and architecture

### Rev 0.2

Component selection and preliminary schematic

### Rev 0.3

Completed schematic and preliminary PCB placement

### Rev 0.4

Completed PCB routing and design verification

### Rev 1.0

Manufacturing release

Future revisions may address:

* Component substitutions
* PCB size optimization
* Additional interfaces
* Improved power architecture
* Improved sensor isolation
* Additional sensing capabilities
* Manufacturing optimization

---

## 29. Design Philosophy

The AeroCore-FC design shall prioritize:

**Correctness → Reliability → Signal/Power Integrity → Manufacturability → Compactness**

The PCB shall not be optimized for minimum size at the expense of electrical performance, thermal performance, serviceability, or manufacturability.

Component selection shall be based on electrical requirements, datasheet recommendations, availability, lifecycle, package constraints, and total system requirements rather than selecting components solely because they are commonly used in existing flight controllers.

---

**Document Status:** Requirements baseline for AeroCore-FC Rev 0.1

