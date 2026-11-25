# Switching Circuit V3

Next-generation board for the battery-cell switching rig. One cell with four tabs (A+, B+, A−, B−) sits on a Neware BTS4000 cycler channel (5 V, 12 A max, 4-wire). The board switches each tab in and out at 5 kHz+ and measures every path's current at well over 1 kHz. A design requirement is **zero drain and zero extra charge** on the cell when the switches are off.

**Status (2026-11-25):** the schematic is complete and passes ERC except for 6 expected items (listed below). The PCB has not been started.

## Quick look

- **Schematic PDF:** [`exports/Switching_Circiut_V3_schematic.pdf`](exports/Switching_Circiut_V3_schematic.pdf)
- **Bill of materials:** [`exports/Switching_Circiut_V3_BOM.csv`](exports/Switching_Circiut_V3_BOM.csv)
- **Design reasoning and part choices:** [`docs/Switch_Charging_V3_Design_Notes.docx`](docs/Switch_Charging_V3_Design_Notes.docx)
- **Breadboard proof of concept:** [`docs/V3_Breadboard_Test_Report.docx`](docs/V3_Breadboard_Test_Report.docx). All six requirement tests passed, including 0 µA cell drain with the gates off.

## Opening it in KiCad

Use KiCad 10. Open `kicad/Switching_Circiut_V3/Switching_Circiut_V3.kicad_pro`. The custom parts library (`lib/V3_Parts`) is registered in the project, so nothing needs installing. Datasheet links in the library point into `datasheets/`. (The "Circiut" spelling in the file names is intentional; it's the original project name.)

## Architecture

| Block | Parts | Notes |
|---|---|---|
| Cycler input | J1 CYC+, J2 CYC−, F1 15 A ATO blade fuse, Q7 CSD25402Q3A reverse-polarity P-FET, D1 SMBJ6.0CA TVS | CYC− is board GND (charge-current return). J13/J14 take the cycler's 4-wire sense leads at +HV/GND. |
| High-side switches (A+, B+) | Q1–Q4 CSD17573Q5B as back-to-back common-source pairs | The back-to-back pair blocks both directions when off. This removes the V2 body-diode leakage path. |
| Low-side switches (A−, B−) | Q5, Q6 CSD17573Q5B | Gate pulldowns return to GND, not through the shunts. |
| Gate drive | U1–U4 Si8271GB isolated drivers. High side powered by PS1/PS2 RECOM R1S-0512 isolated 12 V; low side by a U16 MT3608 12 V boost. | |
| Current sense | R1–R4 5 mΩ Kelvin shunts (WSK2512); U18/U19 INA240A1 (bidirectional) on the high side; U9 ADS131M08 8-channel simultaneous 24-bit ADC | 12 A full scale. All four channels read charge as positive. |
| Voltage sense | +HV divider 100k/24k into ADS131M08 AIN4 | No dividers on the cell tabs, so there's no drain path. |
| Switching MCU | U7 RP2040 + U8 16 MB flash | Drives the gates (GPIO2–5) and reads the ADC (SPI1, CLKIN from GPIO21). |
| Server MCU | U5 RP2350A + U6 16 MB flash | Ethernet (U12 W5500 + RJ45), microSD logging, RTC (U17 RV-3028 + CR2032), OLED, rotary encoder, RGB LED. It can reflash the RP2040 over SWD. |
| USB | J7 USB-C, U13 USBLC6 ESD protection, U14 USB2512B 2-port hub | One cable programs both MCUs. The board is powered by its own isolated USB charger. |
| Power | U20 AP63203 buck 5 V → 3.3 V; Q8 + SW2 power switch, F2 polyfuse | |

ADC channel map: AIN0 P1 current, AIN1 P2, AIN2 N1, AIN3 N2, AIN4 +HV, AIN5–7 unused (grounded).

## Remaining ERC items (all expected)

- U1/U2 GND pins sit on the isolated high-side driver ground, not board GND. This is intentional.
- U1–U4 have VO+ and VO− tied together. That's normal with a single gate resistor.

## Open items

1. **Choose exact parts** for the 3.9 µH buck inductor (shielded, Isat ≥ 2.5 A) and the 15 A ATO fuse plus PCB holder (Littelfuse FLR 178.6165).
2. **First-board check:** measure the R1S gate-drive supply at light load. It must stay under the FETs' ±20 V gate rating.
3. **PCB layout** (not started). Requirements:
   - Match P1↔P2 and N1↔N2 path resistance and inductance (mirror-symmetric power stages).
   - Route sense traces only from shunt pads 2/3 (Kelvin connection). Load current must stay out of the sense loop; that was the lesson from the V2 current-sense errors.
   - Size the copper for 12 A.
   - Keep total loop resistance under about 67 mΩ, so the cycler's 5 V limit isn't reached with a full 4.2 V cell.

## Folder layout

```
kicad/Switching_Circiut_V3/   KiCad project (schematic, empty PCB, V3_Parts library)
exports/                      schematic PDF and BOM generated from the project
docs/                         design notes, ADC and driver/MOSFET comparisons, breadboard test report
datasheets/                   datasheets for parts on the board
datasheets/superseded/        datasheets for parts considered earlier and since replaced
```
