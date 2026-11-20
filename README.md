# Stamp-P4 SC035HGS flight camera recorder

Rocket-borne global-shutter camera recorder built around the M5Stack Stamp-P4
(ESP32-P4NRW32) and the SmartSens SC035HGS camera module. Everything except the
camera module sits on two rigid boards; there is no FFC between them.

| Subproject | Contents |
| --- | --- |
| `main-board/` | 72 × 72 mm C-109J board: Stamp-P4 (castellated SMT), microSD (SDMMC 4-bit), CAN (MCP2562, IL-G-3P, jumper 120 Ω termination), IL-G-4P power input |
| `cam-adapter/` | Plugs onto the Stamp-P4 top-side CSI BTB J2 (Panasonic A4S AXE616124D header, 0.8 mm mated) and carries the SC035HGS circuit and the 24-pin 0.5 mm module FPC socket |

The Stamp-P4 exposes MIPI-CSI only on its top-side J2, so the camera circuit
cannot live on the main board: it is a small board stacked on the module.

This project is separate from `stamp-p4-cam-recorder` (Stamp-P4 + ArduCam
B0335/OV9782), which stays unchanged. The SC035HGS circuit follows
`sc035hgs-esp32-p4x-camera-adapter-mod` (commit 6aea7f5), which remains the
ground board for the ESP32-P4 Function EV board.

## Design decisions (2026-11-20)

1. **TRIG and LED_STROBE are not routed to the main board.** J2 has no spare
   pin, and a second interconnect between the stacked adapter and the main
   board would either be a loose harness or a rigid second connector that
   over-constrains J2. In flight the sensor free-runs at a fixed frame rate and
   recording starts and stops on CAN commands. On the adapter TRIG is pulled
   down to GND and STROBE goes to a 1.8 V test pad; the reference board's
   TRIG/STROBE buffers (U5/U6), their resistors and the IL-G J3 are dropped.
   XSHUTDN is driven from J2 pin 2 (G54_CAM_RST) through the SN74LVC1G07, and
   the sleep push button is dropped.
2. **The camera is powered from J2's SOC_3.3V.** The Stamp-P4 3.3 V rail is an
   SY8089 buck marked "3.3V_600mA" on the module schematic. The SC035HGS draws
   at most 120 mW in total, so at most about 80 mA at 3.3 V through the three
   LDOs (the reference README's 67/80/43 mA are per-rail bounds and do not
   add up). Together with the ESP32-P4 (150 mA with all peripheral clocks on,
   datasheet table 5-7, plus PSRAM and MIPI traffic) the rail stays below
   about 450 mA. J2 pin 14 is rated 0.3 A. Measure SOC_3.3V while recording at
   bring-up.
3. **New repository, new adapter.** The OV9782 adapter of
   `stamp-p4-cam-recorder` is not reused or modified. The main board starts
   from a copy of that project's `main-board/` circuit.
4. **Adapter retention: tape, with M2 holes in reserve.** The adapter is held
   by J2 and by tape over the adapter and the FPC. A4S BTBs have no latch
   (minimum withdrawal force 0.165 N/pin, about 2.6 N for 16 pins), and
   Panasonic asks for screw fixing, so both boards keep two M2 clearance holes
   at the adapter's overhang corners. If vibration tests show tape is not
   enough, add M2 spacers there: the stack from the main board's top to the
   adapter's bottom is about 1.69 mm (castellation joint 0.05 + module PCB
   0.84 + mated height 0.80, roughly ±0.25 mm), so shim each unit until J2 is
   fully seated without bending the adapter.
5. **USB-C at the bottom edge.** The module's USB-C plug body reaches below the
   module, so the module's USB end sits on the main board's bottom edge. J2 and
   the adapter overhang then face the board's interior.
6. **MCLK through an SN74LVC1G34 powered from DOVDD.** The SC035HGS inputs are
   limited to DOVDD + 0.3 V (2.0 V at the 1.7 V DOVDD minimum). A resistive
   divider from the 3.3 V G53 clock leaves about 0.1 V of margin, draws DC
   current and back-powers the sensor while DOVDD is off; the buffer's output
   cannot exceed DOVDD and its Ioff input blocks back-powering. This replaces
   the reference board's 24 MHz oscillator.
