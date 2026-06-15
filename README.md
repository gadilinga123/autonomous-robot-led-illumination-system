# Autonomous Robot LED Illumination System
### Hardware Design Assignment | Completed in 6 days alongside full-time internship

---

## Background

This project was a take-home hardware design assignment provided by a US-based autonomous 
robotics startup. The task was to design a complete high-intensity LED illumination subsystem 
for an autonomous drywall finishing robot — covering component selection, electrical design, 
PCB architecture, schematic design, and wire harness.

I completed this entirely in 6 days while simultaneously working full-time as a research 
intern at WiLab, University of Bologna.

---

## The Problem

An autonomous sanding robot needs to image the drywall surface during each sanding pass 
using a multi-camera vision system. The cameras use global shutter sensors at 30 fps with 
sub-millisecond exposure times (200–800 µs). Ambient construction site lighting is 
inconsistent — so the robot carries its own high-intensity LED ring that fires only during 
each camera exposure window, synchronised to the shutter trigger.

The core challenge: how do you switch 1400 mA of LED current on and off in under 200 µs, 
reliably, from a 3.3V logic signal, with a 48V unregulated battery supply?

---

## System Overview

| Parameter | Value |
|-----------|-------|
| LED model | Cree XLamp XHP70.3 |
| Number of LEDs | 20 |
| Configuration | 6V, 5 LEDs per string |
| Drive current | 1400 mA per string |
| Number of drivers | 4 × Mean Well NLDD-1400H |
| Peak system power | ~161W |
| Average power (2.4% duty cycle) | ~4W |
| Total switching delay | 57.2 ns |
| Illumination angle | 10° grazing (raking light) |
| Power source | 48V unregulated battery via USB-C EPR |

---

## Key Design Decisions

**1. 6V LED configuration over 12V**

At 1400 mA, 6V config gives a string voltage of ~28.75V (5 LEDs × 5.75V) — divides 
20 LEDs into exactly 4 equal strings of 5, one driver per string. 8.5V headroom at 
minimum battery voltage (42V). 12V config would need 7 unbalanced drivers with only 
0.6V headroom at worst case — rejected.

**2. MOSFET bypass instead of PWM input**

NLDD-1400H PWM input is rated only to 1 kHz. Camera sync needs sub-millisecond 
switching — minimum 5 kHz equivalent. Solution: logic-level N-channel MOSFET in series 
with each string cathode, driven via TC4426A gate driver from Orin Nano GPIO. 
Total propagation delay: 57.2 ns. NLDD-1400H runs continuously in CC mode — 
only the current path is switched.

**3. XOR safety interlock**

STM32F103 and Jetson/Orin GPIO both feed an XOR gate (SN74AHC1G86) per string:
- Normal operation: Jetson high + STM32 low → LED on
- Over-temperature: STM32 drives high → LED off (hardware enforced)
- Bench test without Jetson: STM32 high alone → LED on

**4. Flex PCB ring**

20 LEDs at 18° spacing on a ~707mm × 14mm polyimide flex ring. At 2.4% duty cycle 
average power per LED is only ~0.19W — thermal objection to flex does not apply. 
Flex tail carries all 4 string pairs and GPIO signal in a single exit point into a 
ZIF socket on the driver board. No separate harness needed.

**5. Raking light geometry**

LEDs aimed inward at ~10° above surface — arctan(20mm / 112.5mm). At this grazing 
angle a 0.2mm surface defect casts a visible shadow for the camera. Same principle 
used by Festool Planex LHS 2 225 EQI professional drywall sander.

---

## Schematic Overview

7-sheet driver module schematic (Altium Designer, V1.0):

| Sheet | File | Description |
|-------|------|-------------|
| 1 | BLOCK_DIAGRAM.SchDoc | System architecture block diagram |
| 2 | MICROCONTROLLER.SchDoc | STM32F103CBT6TR — GPIO, USB, I2C, USART, boot config |
| 3 | GATE_DRVR_BFR_HDR.SchDoc | SN74LVC125A buffer, TC4426A gate drivers, expansion header |
| 4 | PERIPHERAL.SchDoc | XOR interlock (SN74AHC1G86), TMP75 temp sensor, M24C02 EEPROM |
| 5 | LED_DRIVER_NMOS.SchDoc | 4 × NLDD-1400H drivers, dual N-MOSFET switching array, timing budget |
| 6 | USBC.SchDoc | USB-C EPR connector, RT7209 PD controller, VBUS protection |
| 7 | SMPS_PROTECTION.SchDoc | MAX17504 SMPS (48V→5V), differential choke, LED module edge connector |

Power delivery: USB-C EPR via RT7209 supporting up to 48V/5A negotiation.

> **Note:** PCB layout is not included in this submission.
> Schematics, BOM, and datasheets are attached.

---

## Files

| File | Description |
|------|-------------|
| `LED_DRIVING_MODULE_SCHEMATICS.pdf` | Full 7-sheet schematic — V1.0 |
| `solution_report.pdf` | Complete design report (Parts 1–4) |
| `bom.csv` | Bill of materials with part numbers |
| `datasheet_XHP70.3.pdf` | Cree XLamp XHP70.3 LED datasheet |
| `datasheet_NLDD1400H.pdf` | Mean Well NLDD-1400H driver datasheet |

---

## Tools Used

- Altium Designer — schematic capture
- Saturn PCB Toolkit — trace width and current capacity verification
- JLCPCB stackup calculator — layer construction selection
- Cree XHP70.3 datasheet — I-V curve analysis at 1400 mA
- Mean Well NLDD-H datasheet — PWM frequency limitation analysis

---

## About

**Gadilingareddy G M**
M.Sc. Electronics Engineering for Intelligent Vehicles — MUNER, Modena, Italy
Research Intern — WiLab, University of Bologna
Previously: Embedded Hardware Engineer — Eviden (Atos), Bengaluru (3.5 years)

[LinkedIn](https://linkedin.com/in/gadilinga-reddy) |
[GitHub](https://github.com/gadilinga123) |
gadilinga.reddy.gm@gmail.com
