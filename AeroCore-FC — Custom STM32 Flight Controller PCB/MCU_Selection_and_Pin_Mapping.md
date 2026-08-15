# 05_MCU_Selection_and_Pin_Mapping.md

# AeroCore-FC — MCU Selection & Pin Mapping

**Project:** AeroCore-FC — Custom Flight Controller PCB  
**Document:** MCU Selection & Pin Mapping  
**Revision:** Rev 0.1  
**Status:** Preliminary  
**Design Scope:** MCU Selection, Peripheral Allocation and Pin Mapping

---

# 1. Purpose

This document defines the selected MCU and the preliminary pin assignment for the AeroCore-FC flight controller PCB.

The purpose is to ensure that all mandatory hardware interfaces can be implemented without:

- Peripheral conflicts
- Timer conflicts
- ADC conflicts
- Communication-interface conflicts
- Debug conflicts
- Boot conflicts
- PCB routing problems

The pin map shall be verified against the latest MCU datasheet before schematic freeze.

---

# 2. Selected MCU

## 2.1 MCU

**STM32H743VI**

Target ordering part:

**STM32H743VIT6**

Package:

**LQFP100**

---

# 3. Why STM32H743VI

The STM32H743VI provides significantly more processing and peripheral capability than the minimum required for this flight controller.

Key capabilities include:

- ARM Cortex-M7
- Up to 480 MHz CPU
- 2 MB Flash
- 1 MB RAM
- Multiple SPI interfaces
- Multiple UART/USART interfaces
- Multiple I²C interfaces
- Multiple ADC channels
- Multiple timers
- Motor-control timers
- USB
- FDCAN
- Ethernet MAC
- SWD/JTAG

This provides sufficient processing and peripheral margin for future development.

---

# 4. Why LQFP100

The LQFP100 package is selected for Rev 1.

Reasons:

- Easier inspection
- Easier probing
- Easier rework
- Easier debugging
- Lower routing complexity than BGA
- Sufficient GPIO and peripheral resources
- Suitable for prototype assembly

The objective of Rev 1 is successful design, manufacturing, assembly and bring-up rather than minimum package size.

---

# 5. MCU Architecture

```text
                         STM32H743VI
                              │
          ┌───────────────────┼────────────────────┐
          │                   │                    │
          ▼                   ▼                    ▼
         SPI                 UART                 USB
          │                   │                    │
          ▼                   ▼                    ▼
         IMU                 GNSS                 PC

          │
          ▼
      I²C / SPI
          │
          ▼
      Barometer

          │
          ├──── Timer Output ────► ESC1
          ├──── Timer Output ────► ESC2
          ├──── Timer Output ────► ESC3
          └──── Timer Output ────► ESC4

          │
          ├──── ADC ──────────► Battery Voltage
          │
          └──── ADC ──────────► Current Sense

          │
          └──── SWD ──────────► Debugger

```

---
# Required MCU Interfaces

| Interface | Quantity | Requirement |
|---|---:|---|
| IMU SPI | 1 | Mandatory |
| IMU Interrupt | 1 | Mandatory |
| Barometer I²C | 1 | Mandatory |
| GNSS UART | 1 | Mandatory |
| USB | 1 | Mandatory |
| ESC Outputs | 4 | Mandatory |
| Battery ADC | 1 | Mandatory |
| Current ADC | 1 | Mandatory |
| SWD | 1 | Mandatory |
| Expansion UART | 1 | Desired |
| Expansion I²C | 1 | Desired |
| Expansion SPI | 1 | Desired |
| Status LEDs | 2–3 | Desired |
| Reset | 1 | Mandatory |
| Boot Configuration | 1 | Mandatory |

---

# 7. Pin-Mapping Strategy
The following priorities shall be used:

1. USB
2. SWD
3. IMU SPI
4. ESC timer outputs
5. GNSS UART
6. ADC channels
7. Barometer I²C
8. Reset and boot
9. Status LEDs
10. Expansion interfaces

Timer channels shall be selected carefully so that all four ESC outputs can operate simultaneously.

---

# 8. Preliminary IMU SPI Allocation
Initial allocation:
```
SPI1

PA5  → SPI1_SCK
PA6  → SPI1_MISO
PA7  → SPI1_MOSI
PC4  → IMU_CS
PB0  → IMU_INT
```
Connection:
```
MCU                    IMU

PA5  ────────────────► SCK
PA6  ◄──────────────── MISO
PA7  ────────────────► MOSI
PC4  ────────────────► CS
PB0  ◄──────────────── INT
```
SPI shall be routed as a short local interface.

---

# 9. Barometer Interface
Initial allocation:
```
I2C1

PB8 → I2C1_SCL
PB9 → I2C1_SDA
```
Connection:
```
MCU                    BAROMETER

PB8  ────────────────► SCL
PB9  ◄───────────────► SDA
```
External pull-up resistors shall be included as required by the final bus design.

---
# 10. GNSS UART
The external GNSS module shall use a dedicated UART.

Initial allocation:
```
USART1

PA9  → USART1_TX
PA10 → USART1_RX
```
Connection:
```
MCU                    GNSS

PA9  ────────────────► RX
PA10 ◄──────────────── TX
```
Optional GNSS reset/control signals may be added later.

---
# 11. USB

USB shall use the dedicated STM32 USB pins.

Required signals:
```
USB_DP
USB_DM
VBUS
GND
```
The exact USB pin mapping shall be verified against the STM32H743VI alternate-function table during schematic capture.

USB ESD protection shall be placed close to the USB connector.

---
# 12. ESC Outputs

Four independent timer outputs are required.

Initial candidate allocation:
```
ESC1 → PE9
ESC2 → PE11
ESC3 → PE13
ESC4 → PE14
```
Conceptual connection:
```
MCU

PE9  ─────────► ESC1
PE11 ─────────► ESC2
PE13 ─────────► ESC3
PE14 ─────────► ESC4
```
The exact timer/channel/alternate-function assignment shall be verified against the STM32H743VI datasheet before schematic freeze.

The final timer allocation shall allow all four outputs to operate simultaneously.

----
# 13. Battery Voltage ADC

Initial allocation:
```
PC0 → Battery Voltage ADC
```
Architecture:
```
VBAT
 │
Voltage Divider
 │
RC Filter
 │
 ▼
PC0 / ADC
 │
 ▼
MCU
```
The voltage divider shall be designed for the maximum expected 6S battery voltage.

---
# 14. Current-Sense ADC

Initial allocation:
```
PC1 → Current Sense ADC
```
Architecture:
```
Current Sensor
      │
      ▼
Signal Conditioning
      │
      ▼
PC1 / ADC
      │
      ▼
     MCU
```
The final ADC channel shall be confirmed after the current-sense circuit is finalized.

---
# 15. SWD Interface

SWD shall remain dedicated to debugging.
```
PA13 → SWDIO
PA14 → SWCLK
NRST → Reset
```
Required connector signals:
```
SWDIO
SWCLK
NRST
3.3V
GND
```
The SWD interface shall remain accessible after PCB assembly.

---
# 16. Reset
```
NRST → MCU Reset
```
The reset circuit shall follow the STM32H743 reference design.

NRST shall also be connected to the debug interface.

---
# 17. BOOT Configuration

The MCU shall normally boot from internal Flash.

BOOT0 shall therefore have a defined default state.

A method shall be provided to force the MCU into the required system boot mode during development or recovery.

Conceptual architecture:
```
BOOT0
  │
  └──── Boot Configuration
```
The exact circuit shall be finalized during schematic capture.

---
# 18. Status LEDs

Initial allocation:
```
LED1 → PG0
LED2 → PG1
LED3 → PG2
```
Suggested functions:
```
LED1 → Power / Status
LED2 → MCU Activity
LED3 → User / Debug
```
The exact functions can be assigned in firmware.

Each LED shall have an appropriate current-limiting resistor.

---
# 19. Expansion UART

A second UART shall be reserved for future expansion.

Initial allocation:
```
UART4


PC10 → UART4_TX
PC11 → UART4_RX
```
Potential applications:
```
Telemetry
External peripheral
Debug console
Radio interface
Future expansion
```
---
# 20. Expansion SPI

A second SPI interface shall be reserved.

Initial allocation:
```
SPI2


PB13 → SPI2_SCK
PB14 → SPI2_MISO
PB15 → SPI2_MOSI
PB12 → SPI2_CS
```
This interface may be connected to the expansion connector.

---
# 21. Expansion I²C

A second I²C interface shall be reserved.

Initial allocation:
```
I2C2


PB10 → I2C2_SCL
PB11 → I2C2_SDA
```
Potential applications:
```
External sensors
Compass
Additional peripheral
Future expansion
```
---

# 22. Preliminary Pin Allocation Table

| MCU Pin | Function | Peripheral | Connected To | Priority |
|---|---|---|---|---|
| PA5 | SPI1_SCK | SPI1 | IMU | Mandatory |
| PA6 | SPI1_MISO | SPI1 | IMU | Mandatory |
| PA7 | SPI1_MOSI | SPI1 | IMU | Mandatory |
| PC4 | GPIO | CS | IMU | Mandatory |
| PB0 | GPIO/EXTI | Interrupt | IMU | Mandatory |
| PB8 | I2C1_SCL | I2C1 | Barometer | Mandatory |
| PB9 | I2C1_SDA | I2C1 | Barometer | Mandatory |
| PA9 | USART1_TX | USART1 | GNSS | Mandatory |
| PA10 | USART1_RX | USART1 | GNSS | Mandatory |
| USB pins | USB | USB | USB-C | Mandatory |
| PE9 | Timer Output | TIM | ESC1 | Mandatory |
| PE11 | Timer Output | TIM | ESC2 | Mandatory |
| PE13 | Timer Output | TIM | ESC3 | Mandatory |
| PE14 | Timer Output | TIM | ESC4 | Mandatory |
| PC0 | ADC | ADC | Battery Voltage | Mandatory |
| PC1 | ADC | ADC | Current Sense | Mandatory |
| PA13 | SWDIO | Debug | SWD | Mandatory |
| PA14 | SWCLK | Debug | SWD | Mandatory |
| NRST | Reset | Reset | SWD / Reset | Mandatory |
| PC10 | UART TX | UART4 | Expansion | Desired |
| PC11 | UART RX | UART4 | Expansion | Desired |
| PB13 | SPI2_SCK | SPI2 | Expansion | Desired |
| PB14 | SPI2_MISO | SPI2 | Expansion | Desired |
| PB15 | SPI2_MOSI | SPI2 | Expansion | Desired |
| PB12 | SPI2_CS | GPIO | Expansion | Desired |
| PB10 | I2C2_SCL | I2C2 | Expansion | Desired |
| PB11 | I2C2_SDA | I2C2 | Expansion | Desired |
| PG0 | GPIO | GPIO | LED1 | Desired |
| PG1 | GPIO | GPIO | LED2 | Desired |
| PG2 | GPIO | GPIO | LED3 | Desired |

---

# 23. Pin Conflict Review

Before schematic freeze, the following items shall be verified:

- [ ] All four ESC outputs use valid timer channels.
- [ ] All four ESC timer channels can operate simultaneously.
- [ ] IMU pins support the selected SPI peripheral.
- [ ] IMU chip-select pin is available as GPIO.
- [ ] IMU interrupt pin supports EXTI functionality.
- [ ] Barometer pins support I²C1.
- [ ] GNSS pins support USART1.
- [ ] USB pins are correctly assigned.
- [ ] Battery voltage pin supports the selected ADC.
- [ ] Current-sense pin supports the selected ADC.
- [ ] SWDIO and SWCLK remain available for debugging.
- [ ] NRST is correctly connected to the reset circuit and SWD interface.
- [ ] BOOT0 is correctly configured.
- [ ] HSE crystal pins remain available.
- [ ] All MCU power pins are correctly connected.
- [ ] VDDA and VREF+ requirements are correctly handled.
- [ ] VCAP requirements are correctly handled.
- [ ] Expansion UART does not conflict with mandatory peripherals.
- [ ] Expansion I²C does not conflict with mandatory peripherals.
- [ ] Expansion SPI does not conflict with mandatory peripherals.
- [ ] Status LED GPIOs do not conflict with mandatory functions.
- [ ] No unintended alternate-function conflicts exist.
- [ ] Pin assignments are compatible with the selected LQFP100 package.
- [ ] All assigned pins are verified against the latest STM32H743VI datasheet.
- [ ] Final timer/channel assignments are verified before schematic freeze.

---
# 23. Pin Conflict Review

Before schematic freeze, verify:

 - [ ] All four ESC outputs use valid timer channels
 - [ ] Timer channels can operate simultaneously
 - [ ] IMU pins support SPI1
 - [ ] IMU CS is available as GPIO
 - [ ] IMU interrupt supports EXTI
 - [ ] Barometer pins support I²C1
 - [ ] GNSS pins support USART1
 - [ ] USB pins are correctly assigned
 - [ ] Battery ADC pin supports the selected ADC
 - [ ] Current ADC pin supports the selected ADC
 - [ ] SWD pins remain available
 - [ ] BOOT0 is correctly configured
 - [ ] Expansion interfaces do not conflict
 - [ ] Oscillator pins remain available
 - [ ] All MCU power pins are connected
 - [ ] VREF+ and VDDA requirements are handled
 - [ ] VCAP requirements are handled
 - [ ] No unintended alternate-function conflicts exist

---

# 24. MCU Power Pins

All MCU supply pins shall be connected according to the STM32H743VI datasheet.

The schematic shall include the required connections for:

- VDD
- VSS
- VDDA
- VSSA
- VREF+
- VCAP
- VBAT
- USB supply pins

Each supply shall receive the appropriate decoupling components.

Exact capacitor values shall be taken from the latest STM32H743 documentation.

---
# 25. Clock

The initial architecture shall use the MCU manufacturer's recommended HSE configuration.

Conceptual architecture:
```
        Crystal
       ┌───────┐
       │       │
PH0 ───┤       ├─── PH1
       │       │
       └───────┘
```
The final crystal frequency and load capacitors shall be selected during schematic design.

---
# 26. Unused Pins

Unused MCU pins shall be reviewed individually.

Considerations:

- Future expansion
- Test points
- EMI
- Default state
- Firmware configuration

Unused pins shall not be assigned randomly.

---
# 27. Pin-Mapping Design Rule

A pin assignment shall not be considered final until all of the following are verified:
```
Pin Function
     +
Alternate Function
     +
Peripheral Availability
     +
Timer Compatibility
     +
ADC Capability
     +
PCB Routing
     +
Connector Requirement
     ↓
FINAL PIN
```

---
# 28. MCU Selection Decision

Current MCU selection:

STM32H743VI

Target package:

LQFP100

Target part:

STM32H743VIT6

The MCU is considered suitable for the AeroCore-FC Rev 1 architecture.

The final part shall be frozen after:

- Datasheet verification
- Alternate-function verification
- Timer verification
- Package verification
- Power-pin verification
- Availability verification

---

# 29. Important Engineering Note

The pin assignments in this document are a preliminary engineering allocation.

Before schematic release, every pin shall be checked against the latest:

- STM32H743VI datasheet
- STM32H743 reference manual
- STM32H743VI alternate-function table
- STM32H743 hardware design guidelines

Particular attention shall be given to:

- Timer channels
- USB pins
- ADC channels
- Alternate functions
- VCAP
- VDDA/VREF+
- Boot configuration
- HSE/LSE pins

 ----
No PCB routing shall be performed using an unverified pin assignment.



