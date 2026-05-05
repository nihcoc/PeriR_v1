# PeriR

A custom Software Defined Radio (SDR) carrier board built around the **Xilinx Zynq-7020 SoC** and **Analog Devices AD9364** RF transceiver. The board is designed to run **Kuiper Linux** (Analog Devices' Raspberry Pi OS-based distro) out of the box, with native support for GNU Radio and SDR++ applications.

---

## Overview

This project is inspired by the [FreeSRP](http://electronics.kitchen/misc/freesrp/) open-source SDR by Lukas Lao Beyer and borrows design references from the **FMCOMMS4**, **Zedboard**, **ADRV9364-Z7020**, and **PlutoSDR**.

The goal is a powerful, Linux-capable SDR transceiver with a compact form factor and eventually, a handheld device like the HackRF Portapack.

---

## Key Specifications

| Feature | Detail |
|---|---|
| **SoC / FPGA** | Xilinx Zynq-7020 (ARM Cortex-A9, ~85K LUTs) |
| **RF Transceiver** | Analog Devices AD9364 |
| **Frequency Range** | 70 MHz – 6.0 GHz |
| **Bandwidth** | 200 kHz – 56 MHz |
| **RAM** | 2× 128MB DDR3 (256MB total, Zedboard config) |
| **OS** | Kuiper Linux (Analog Devices / Raspberry Pi OS based) |
| **Primary Storage** | MicroSD card |

---

## Block Diagram

```
Antenna
   │
   ▼
AD9364 RF Transceiver (70MHz–6GHz)
   │  LVDS (max throughput)
   ▼
Zynq-7020 (PS + PL)
   ├── ARM Cortex-A9 (PS) ──► DDR3 RAM, SD Card, USB-OTG, Ethernet, HDMI
```

---

## Hardware Design

### RF Transceiver — AD9364
- **Signalling standard:** LVDS (chosen over CMOS for lower EMI and higher throughput)
- **Interface:** Connected to FPGA Bank 34 (VCCO = 2.5V, `LVDS_25` standard)
- **Balun:** TCM1-63AX+ wideband RF transformer (≤1.8 dB insertion loss @ 6 GHz)
- **Clock:** 40 MHz, 10 pF load capacitance (tested reference from AD)
- **Power rail:** 1.3V via ADP1754ACPZ-1.3-R7 (AD-recommended for stability)
- **TX Power Amp:** PGA-102 (dedicated clean 3.3V supply)

### Zynq-7020 SoC
- **PS Clock:** 33.33 MHz  
- **PL Clock:** 100 MHz (EPSON SG-8018CA oscillator)
- **Boot:** SD Card (primary), QSPI Flash (fallback)
- **Configuration:** Boot mode selectable via header pins/switches
- **RAM interface:** Signal lines terminated at 40 Ω

### Power Architecture

Power sequencing follows **DS187** requirements:
1. VCCPINT (1.0V)
2. VCCPAUX + VCCPLL (1.8V) — simultaneously
3. PS VCCO supplies: VCCOMIO0, VCCOMIO1, VCCO_DDR (3.3V / 1.5V / 0.75V)

| Rail | IC | Current | Usage |
|---|---|---|---|
| 1.0V | TPS62130A | 3A | Zynq VCCINT, VCCBRAM |
| 1.8V | TPS62130A | 3A | Zynq VCCAUX, VCCADC, MIO logic |
| 3.3V | TPS65251-3RHAR (Ch1) | 3A | I/O, SD card, interfaces |
| 2.5V | TPS65251-3RHAR (Ch2) | 2A | FPGA Bank 34 (LVDS) |
| 1.5V | TPS65251-3RHAR (Ch3) | 2A | DDR3 VCCIO |
| 0.75V | MAX1510ETB+ | — | DDR3 VTT termination |
| 1.3V | ADP1754ACPZ-1.3-R7 | — | AD9364 primary supply |
| 5V | External connector | — | Board input power |

> Sequencing is managed using EN (Enable) and PG (Power Good) pins chained between power ICs.
> AD7291BCPZ added for full-board power monitoring.

### Interfaces

| Interface | IC / Notes |
|---|---|
| **HDMI** | ADV7511 transmitter — follows Zedboard design for Kuiper Linux compatibility |
| **USB-OTG** | Standard USB 2.0 OTG with ESD protection |
| **Ethernet** | GbE — backup interface for setup and testing |
| **SD Card** | Level-shifted (1.8V MIO → 3.3V SD logic) |
| **QSPI Flash** | MT25QU128ABA — 128Mb, all data lines active |


## Software

- **OS:** [Kuiper Linux](https://wiki.analog.com/resources/tools-software/linux-software/kuiper-linux) by Analog Devices
- **SDR Applications:** GNU Radio, SDR++
- **FPGA toolchain:** Xilinx Vivado + Analog Devices HDL reference designs
- **Reference HDL:** [Analog Devices HDL](https://github.com/analogdevicesinc/hdl) (FMCOMMS4 / ADRV9364 compatible)

---

## Design References

- [FreeSRP](http://electronics.kitchen/misc/freesrp/) — Lukas Lao Beyer
- Analog Devices FMCOMMS4
- Xilinx Zedboard
- ADRV9364-Z7020
- PlutoSDR
- **Key Datasheets:** UG585, UG933, UG482, UG470, UG483, DS187, AD9364 Reference Manual

---

## Project Status

- [x] Component selection
- [x] Schematic capture
- [x] Footprint assignment
- [x] Schematic verification (round 1)
- [ ] Schematic verification (round 2)
- [ ] PCB layout & routing
- [ ] DFM review
- [ ] Fabrication & assembly
- [ ] Bring-up & firmware development
- [ ] HAT board design

---


*This is a personal learning project. Feedback and suggestions welcome — especially on the RF and LVDS sections!*
