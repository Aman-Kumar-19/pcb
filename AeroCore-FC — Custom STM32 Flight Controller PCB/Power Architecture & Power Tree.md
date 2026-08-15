# AeroCore-FC — Power Architecture & Power Tree

**Document:** Power Architecture & Power Tree
**Project:** AeroCore-FC — Custom Flight Controller PCB
**Revision:** Rev 0.1
**Status:** Preliminary Electrical Design
**Design Scope:** Power Input, Protection, Regulation and Monitoring

---

# 1. Purpose

This document defines the power architecture for the AeroCore-FC flight controller PCB.

The power subsystem shall convert the external battery input into stable, low-noise supply rails required by the MCU, sensors, USB interface, and external peripherals.

The design shall prioritize:

* Electrical safety
* Stable regulation
* Low noise
* Transient response
* Thermal performance
* Efficient power conversion
* Sensor power integrity
* Protection
* Manufacturability

---

# 2. Power Architecture Overview

The flight controller shall be powered from an external LiPo battery.

### Target input range

**2S–6S LiPo**

Nominal voltage:

```text
2S = 7.4 V
3S = 11.1 V
4S = 14.8 V
5S = 18.5 V
6S = 22.2 V
```

Maximum fully charged 6S voltage:

```text
6S × 4.2 V = 25.2 V
```

Therefore:

> **The power-input stage shall be designed for at least 25.2 V operating input, with additional transient margin.**

---

# 3. Overall Power Tree

```text
                         LiPo Battery
                              │
                              │ VBAT
                              ▼
                    ┌──────────────────┐
                    │ Reverse Polarity │
                    │    Protection    │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Transient / TVS  │
                    │    Protection    │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Input Filtering  │
                    └────────┬─────────┘
                             │
                       Protected VBAT
                             │
               ┌─────────────┴─────────────┐
               │                           │
               ▼                           ▼
        ┌──────────────┐            ┌──────────────┐
        │ 5 V Buck     │            │ 3.3 V Buck   │
        │ Regulator    │            │ Regulator    │
        └──────┬───────┘            └──────┬───────┘
               │                           │
               │                           │
               ▼                           ▼
          5 V RAIL                    3.3 V RAIL
               │                           │
        ┌──────┼──────┐              ┌─────┼────────┐
        │      │      │              │     │        │
       GPS   USB   Expansion         MCU   IMU    Barometer
```

---

# 4. Power Domains

The design shall contain three primary electrical power domains.

## 4.1 VBAT

Battery-derived voltage.

Used for:

* Input power
* Battery voltage measurement
* Power-system monitoring

The VBAT domain shall not directly power 3.3 V digital components.

---

## 4.2 5 V Rail

Used for:

* Selected external peripherals
* GNSS modules where required
* Expansion interfaces
* Other 5 V-compatible devices

The exact external load shall be determined during schematic design.

---

## 4.3 3.3 V Rail

Primary logic and sensor supply.

Target loads:

* MCU
* IMU
* Barometer
* Digital sensors
* Logic interfaces
* Status indicators

This rail is the most important low-voltage rail on the PCB.

---

# 5. Input Voltage Requirement

The power input shall support:

| Parameter                 |                    Target |
| ------------------------- | ------------------------: |
| Minimum operating input   | 2S fully discharged range |
| Nominal maximum battery   |                        6S |
| Maximum fully charged     |                    25.2 V |
| Design input rating       |                  ≥ 25.2 V |
| Recommended design margin |                    ≥ 30 V |

The exact regulator voltage ratings shall provide adequate margin above the maximum battery voltage.

---

# 6. Battery Connector

The PCB shall provide a dedicated battery input connector.

Minimum connections:

```text
BAT+
BAT-
```

The connector shall be rated for the expected voltage and current.

The flight controller does not carry the motor/ESC current.

Therefore, the connector current requirement is primarily based on:

* Flight-controller consumption
* External peripheral consumption
* GNSS consumption
* Transient loads

---

# 7. Reverse-Polarity Protection

A MOSFET-based reverse-polarity protection circuit is preferred.

Conceptual architecture:

```text
BAT+
 │
 ▼
┌──────────────┐
│ Protection   │
│ MOSFET       │
└──────┬───────┘
       │
       ▼
Protected VBAT
```

A MOSFET solution is preferred over a series diode because it can significantly reduce voltage drop and power loss.

The final MOSFET shall be selected based on:

* VDS rating
* RDS(on)
* Gate threshold/drive
* Package
* Power dissipation
* Transient capability

---

# 8. Transient Protection

A TVS diode shall be evaluated at the battery input.

Conceptual connection:

```text
Protected Input
      │
      ├───────────────► Power System
      │
     TVS
      │
     GND
```

The TVS shall be selected according to:

* Normal maximum battery voltage
* Maximum standoff voltage
* Clamping voltage
* Pulse power
* Expected transient environment

The TVS shall not conduct during normal 6S battery operation.

---

# 9. Input Filtering

The battery input shall contain local bulk capacitance.

Conceptual structure:

```text
VBAT
 │
 ├──── Bulk Capacitor
 │
 ├──── Ceramic Capacitor
 │
 ▼
Power Regulators
```

The final capacitor values shall be selected based on:

* Regulator datasheet requirements
* Input transient requirements
* Battery wiring characteristics
* PCB layout

---

# 10. 5 V Regulation

The 5 V rail shall be generated from the protected battery input.

Because the input may reach 25.2 V, a **buck converter** is preferred.

Architecture:

```text
VBAT
 │
 ▼
5 V Buck
 │
 ▼
5 V
```

The regulator shall provide adequate current for:

* GNSS
* USB-related loads
* Expansion peripherals
* Future external devices

### Initial design target

**5 V / 1 A minimum**

A higher-current regulator may be selected if the power budget requires it.

---

# 11. 3.3 V Regulation

The 3.3 V rail shall power the primary digital and sensor circuitry.

Architecture:

```text
VBAT
 │
 ▼
3.3 V Buck
 │
 ▼
3.3 V
```

A direct high-voltage-to-3.3 V buck is preferred for the initial architecture because it avoids unnecessary intermediate conversion losses.

An LDO may be added later for particularly noise-sensitive analog/sensor loads if required by measurement results.

---

# 12. Initial 3.3 V Power Budget

A preliminary budget shall be established before regulator selection.

| Load                | Estimated Current |
| ------------------- | ----------------: |
| MCU                 |            200 mA |
| IMU                 |             20 mA |
| Barometer           |             10 mA |
| LEDs                |             15 mA |
| USB/logic           |             50 mA |
| Miscellaneous       |             50 mA |
| Expansion margin    |            150 mA |
| **Estimated total** |        **495 mA** |

Therefore:

> Initial design target: **3.3 V / ≥ 1 A**

The additional margin allows for startup current, future peripherals, and estimation error.

The final current budget shall use actual maximum values from the selected component datasheets.

---

# 13. Initial 5 V Power Budget

Preliminary target:

| Load                 | Estimated Current |
| -------------------- | ----------------: |
| GNSS                 |        100–200 mA |
| External peripherals |            200 mA |
| Expansion            |            300 mA |
| Miscellaneous        |            100 mA |
| Margin               |            200 mA |
| **Target**           |         **≥ 1 A** |

Therefore:

> Initial design target: **5 V / ≥ 1 A**

The final regulator shall be selected after the exact external-module requirements are defined.

---

# 14. Regulator Selection Requirements

Both regulators shall be evaluated for:

* Input voltage rating
* Output voltage accuracy
* Maximum output current
* Efficiency
* Switching frequency
* Thermal performance
* Inductor requirements
* Output capacitor requirements
* Input capacitor requirements
* Feedback network
* Protection features
* Package
* Availability

The selected regulator shall have sufficient voltage margin for 6S battery operation.

---

# 15. Power Dissipation

For a linear regulator:

```text
PLOSS = (VIN - VOUT) × IOUT
```

For example, directly converting 25 V to 3.3 V using an LDO at 0.5 A would result in:

```text
PLOSS = (25 - 3.3) × 0.5
      = 10.85 W
```

This is unacceptable for the intended compact PCB.

Therefore:

> **A direct LDO from 6S battery to 3.3 V shall not be used.**

A switching regulator is required for the main conversion.

---

# 16. Optional Sensor LDO

If PCB testing shows excessive switching noise on the 3.3 V rail, a dedicated low-noise LDO may be considered for the IMU/sensor domain.

Potential architecture:

```text
3.3 V Buck
    │
    ▼
Low-Noise LDO
    │
    ▼
3.3 V_SENSOR
    │
 ┌──┴──────┐
 │         │
 IMU     Barometer
```

This is **not mandatory for Rev 1**.

It shall only be added if justified by:

* Sensor datasheet requirements
* Power-integrity analysis
* Noise measurements
* PCB area
* Power budget

---

# 17. MCU Power Distribution

The MCU shall receive power through a low-impedance 3.3 V distribution network.

Conceptual architecture:

```text
3.3 V Buck
     │
     ▼
3.3 V Plane
     │
     ├── MCU VDD
     ├── MCU VDDA
     └── Other MCU supply pins
```

Each MCU supply pin shall receive the manufacturer's recommended local decoupling.

---

# 18. IMU Power Distribution

The IMU shall receive a clean local supply.

```text
3.3 V
 │
 ▼
Local Decoupling
 │
 ▼
IMU
```

The decoupling network shall be placed immediately adjacent to the IMU power pins.

The IMU supply path shall avoid unnecessary routing through noisy/high-current areas.

---

# 19. Barometer Power

The barometer shall have local decoupling according to its datasheet.

```text
3.3 V
 │
 ▼
Decoupling
 │
 ▼
Barometer
```

The barometer shall be physically separated from hot switching components where practical.

---

# 20. USB Power

USB VBUS shall be treated separately from the battery input.

Conceptual architecture:

```text
USB-C VBUS
     │
     ▼
USB Protection
     │
     ▼
USB Power Detection / Supply
```

The final USB power arrangement shall ensure that:

* USB power does not unintentionally back-feed the battery system.
* External USB power does not create an unsafe condition when battery power is connected.
* VBUS is correctly handled according to the selected MCU USB implementation.

This shall be finalized during USB schematic design.

---

# 21. Battery Voltage Monitoring

The battery voltage shall be monitored independently of the regulated rails.

Architecture:

```text
VBAT
 │
 R1
 │
 ├──────────► ADC
 │
 R2
 │
GND
```

The resistor ratio shall be calculated for the maximum battery voltage.

For an ADC with a 3.3 V maximum input:

```text
VADC < 3.3 V
```

At maximum battery voltage:

```text
VADC_MAX = VBAT_MAX × R2 / (R1 + R2)
```

The divider shall include sufficient margin below the ADC maximum input.

---

# 22. ADC Filtering

A small capacitor may be placed across the lower resistor.

```text
          R1
VBAT ───/\/\/───┬──── ADC
                │
               R2
                │
               GND

                │
                C
                │
               GND
```

This creates a low-pass filter.

The RC values shall be selected based on:

* Required measurement bandwidth
* ADC sampling rate
* Noise level
* Divider impedance
* ADC acquisition requirements

---

# 23. Current Measurement Architecture

The current measurement architecture shall be finalized after deciding whether current sensing is:

1. On-board
2. External

For Rev 1, the preferred approach is:

> **Provide an external current-sense input capability if an on-board high-current shunt complicates the PCB power architecture.**

This keeps the flight-controller PCB focused on logic and sensing rather than high-current power distribution.

---

# 24. Power Test Points

The following test points shall be included:

```text
TP1 — VBAT
TP2 — GND
TP3 — 5V
TP4 — 3.3V
TP5 — NRST
```

Additional test points may be provided for:

* VDDA
* Sensor supply
* USB VBUS
* Current-sense output

---

# 25. Power Layout Strategy

The power section shall be placed away from the IMU.

Preferred arrangement:

```text
┌─────────────────────────────────────────┐
│                                         │
│             IMU / SENSOR                │
│                                         │
│          MCU / DIGITAL                  │
│                                         │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│             POWER SECTION               │
│                                         │
│ BAT → PROTECTION → BUCK → REGULATORS   │
│                                         │
└─────────────────────────────────────────┘
```

The switching regulator:

* Inductor
* High-side switching node
* Diode/synchronous path
* Input capacitors
* Output capacitors

shall be kept physically compact.

The switching node shall be minimized.

---

# 26. High-Current Switching Loop

The regulator's high-current switching loop shall be minimized.

Conceptually:

```text
        ┌─────────────┐
        │   Switch    │
        └──────┬──────┘
               │
             Inductor
               │
               ▼
             Output

Input Capacitor
      │
      └──────► Switch
```

The input capacitor shall be located as close as possible to the regulator input and ground pins.

This reduces parasitic inductance and switching-loop area.

---

# 27. Grounding Strategy

The power system shall connect to the main ground plane.

The design shall avoid creating unnecessary isolated ground islands.

However, high-current switching return paths shall be physically controlled so that switching currents do not pass through sensitive sensor areas.

---

# 28. Preliminary Power Requirements

| Parameter               |           Target |
| ----------------------- | ---------------: |
| Battery                 |       2S–6S LiPo |
| Maximum battery voltage |           25.2 V |
| Design input rating     | ≥ 30 V preferred |
| Main 5 V rail           |     ≥ 1 A target |
| Main 3.3 V rail         |     ≥ 1 A target |
| Main conversion         |             Buck |
| Reverse protection      |         Required |
| Transient protection    |         Required |
| Input filtering         |         Required |
| Battery voltage sensing |         Required |
| Current sensing         |         Required |
| Power test points       |         Required |
| Sensor LDO              |         Optional |

---

# 29. Power Design Verification

Before selecting final regulator components, the following calculations shall be completed:

* [ ] Maximum battery voltage
* [ ] Minimum battery voltage
* [ ] 3.3 V maximum current
* [ ] 5 V maximum current
* [ ] Regulator efficiency
* [ ] Regulator thermal dissipation
* [ ] Inductor current rating
* [ ] Inductor saturation current
* [ ] Input capacitor RMS current
* [ ] Output capacitor requirements
* [ ] TVS selection
* [ ] Reverse-polarity MOSFET losses
* [ ] Battery divider maximum voltage
* [ ] ADC divider error
* [ ] Power-up behavior

---

# 30. Preliminary Power Architecture Decision

The current architecture is:

```text
              2S–6S LiPo
                   │
                   ▼
          Reverse Protection
                   │
                   ▼
            TVS Protection
                   │
                   ▼
            Input Filtering
                   │
            Protected VBAT
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
       5 V Buck          3.3 V Buck
          │                 │
          ▼                 ▼
        5 V Rail          3.3 V Rail
          │                 │
       External       ┌─────┼──────────┐
      Peripherals     │     │          │
                     MCU    IMU     Barometer
```

This architecture is the baseline for schematic development.

---






