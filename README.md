# Sluice

**A 12 V / 15 A smart amp-hour meter with independent charge and discharge control.**

Sluice measures the amp-hours flowing through a 12 V battery and can cut off charging and discharging separately. A shunt in the negative leg feeds an ESP32 coulomb counter, and a BQ76200 gate driver switches a high-side back-to-back MOSFET pair.

**Status:** 🚧 Design in progress. Not yet built. Not yet tested.
**Target for tested v1.0 release:** 14 February 2027

---

> [!CAUTION]
> **This is an untested, experimental design. Do not connect it to a real battery.**
> It switches up to 15 A on a lithium battery. A wrong design, bad solder joint, or missing fuse can overheat the board or start a fire.
>
> - Do not build or use this design until the repository shows a **tested v1.0 release**.
> - Always use a correctly rated fuse in series with the battery.
> - Use keyed connectors. The driver's BAT and PACK pins are **not** reverse-polarity tolerant (absolute maximum −0.3 V). A reversed battery will destroy it.
> - Do not leave a prototype running unattended.
> - Sluice is **not a BMS.** It has no per-cell voltage or temperature protection. Use it alongside a proper BMS, not instead of one.
> - Provided as-is, with no warranty. You are responsible for anything you build.

---

## What it does (planned)

- Counts amp-hours in and out of a 12 V battery (coulomb counting).
- Switches charge and discharge paths **independently** using two MOSFETs and a BQ76200 gate driver.
- Shows live voltage, current and Ah on an onboard display.
- Adds a hardware overcurrent trip so protection does not depend on firmware speed.

## Target specifications

| Item | Target |
|---|---|
| System voltage | 12 V (LiFePO4) |
| Continuous current | 15 A |
| Current sensing | Shunt in the negative leg (amplifier/ADC part to be decided) |
| Switching | High-side, back-to-back N-channel MOSFETs, BQ76200 driver |
| Controller | ESP32-WROOM-32 |
| Board | 100 × 60 mm, 2-layer, 2 oz copper; MOSFETs on bottom, control and display on top |

## How the switch works

```
BAT+ ──[fuse]──[CHG FET]──┬──[DSG FET]── PACK+
                          │
                 (drains joined, no connection
                  to the driver)

BAT− ──[shunt]──────────────────────────── PACK−
```

The BQ76200 senses BAT and PACK and drives both gates. The ESP32 controls it through three pins: `CHG_EN`, `DSG_EN` and `CP_EN`.

| CHG FET | DSG FET | Result |
|---|---|---|
| On | On | Current flows both ways |
| Off | On | Charging blocked; discharge passes through the CHG FET body diode |
| On | Off | Discharge blocked; charging passes through the DSG FET body diode |
| Off | Off | Fully isolated |

> [!WARNING]
> In the two middle states, current flows through a MOSFET body diode (about 0.7 V). At 15 A that is roughly 10 W of heat. Do not hold these states under full load.

## Known limitations

- Coulomb counting drifts over time. The count needs periodic recalibration at a known full or empty point.
- No per-cell monitoring, balancing or temperature protection.
- Only the positive side is switched. The negative leg is sensing only.
- The BQ76200's minimum BAT voltage versus the LFP low cutoff (about 10 V) is still to be verified against the datasheet.

## Roadmap

| Stage | Target date |
|---|---|
| Schematic and review | 22 Nov 2026 |
| Layout and PCB order | 2 Dec 2026 |
| Fabrication and shipping | by 24 Dec 2026 |
| Power-stage bring-up (no firmware) | by 10 Jan 2027 |
| Firmware, load tests, calibration | by 7 Feb 2027 |
| **Tested v1.0 release** | **14 Feb 2027** |

A board respin would move the release by roughly 4-6 weeks.

**Release criteria (proposed):**

- Both gates verified in all four states.
- Hardware overcurrent trip verified.
- Supervised 15 A run with measured temperature rise.
- Ah count checked against a reference meter over a full cycle.

## Repository layout (planned)

```
hardware/   schematic, PCB, BOM, gerbers
firmware/   ESP32 source
docs/       design notes and test results
```

## License

Hardware: CERN-OHL (version to be confirmed). Firmware: MIT.

## Author

NITHIN MATHEW JOJI
