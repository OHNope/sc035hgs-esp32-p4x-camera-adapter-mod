# Stamp-P4 SC035HGS flight camera recorder

Rocket-borne global-shutter camera recorder built around the M5Stack Stamp-P4
(ESP32-P4NRW32) and the SmartSens SC035HGS camera module. Everything except the
camera module sits on two rigid boards; there is no FFC between them.

| Subproject | Contents |
| --- | --- |
| `main-board/` | 72 × 72 mm C-109J SELFY board: Stamp-P4 (castellated SMT), microSD (SDMMC 4-bit, RT9080 3.3 V + XC8107 switch), CAN (MCP2562, JST XH 3P, jumper 120 Ω termination), JST VH 2P 5 V power input |
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
   from `stamp-p4-demo-rec11` (9dddc31), the Stamp-P4 main board already redone
   to the C-109J electrical spec.
4. **Adapter retention: M2 screws and spacers on flight units.** A4S BTBs
   have no latch (minimum withdrawal force 0.165 N/pin, about 2.6 N for 16
   pins), and Panasonic asks for screw fixing, so both boards carry two M2
   clearance holes at the adapter's overhang corners (adapter H1/H2, main
   board H5/H6) and flight units are screwed down through spacers. Ground and
   bench units may use tape over the adapter and the FPC instead. See
   "Assembly and bring-up" for the spacer height.
5. **USB-C at the bottom edge.** The module's USB-C plug body reaches below the
   module, so the module's USB end sits on the main board's bottom edge. J2 and
   the adapter overhang then face the board's interior.
6. **MCLK through an SN74LVC1G34 powered from DOVDD.** The SC035HGS inputs are
   limited to DOVDD + 0.3 V (2.0 V at the 1.7 V DOVDD minimum). A resistive
   divider from the 3.3 V G53 clock leaves about 0.1 V of margin, draws DC
   current and back-powers the sensor while DOVDD is off; the buffer's output
   cannot exceed DOVDD and its Ioff input blocks back-powering. This replaces
   the reference board's 24 MHz oscillator.
7. **C-109J electrical spec (SELFY, board 1).** The positions are mandatory:
   - J1 JST VH 2P power on the left edge (1: 5 V logic supply, 2: GND) from
     POWER;
   - J3 JST XH 3P CAN on the top edge (1: CAN_H, 2: CAN_L, 3: GND) to and from
     RT_VIDEO;
   - other cables go toward the bottom when in doubt ("迷ったら下"); the camera
     FPC socket on the adapter faces the camera instead (user decision);
   - the board is 72 × 72 mm, with four 4 mm holes on a 64 mm square (4 mm from
     each edge). These are the spec's current values; 66 mm and 3 mm are struck
     through, which replaces the 60 mm / 3 mm used by rec11.

## PCB results (2026-11-21)

Both boards use JLCPCB's 4-layer JLC04161H-7628 stack (1.6 mm, ENIG): F.Cu and
B.Cu signal, In1.Cu and In2.Cu solid GND, GND pours on the outer layers with
stitching vias. Each board was generated in a kicad-auto workspace, reviewed by
pcb-review, and published to the project's `.kicad_pcb` with its rules.

### Adapter (`cam-adapter/`)

- Workspace `pcb-sp4sc-adapter-r2`; review `pcbrev-sp4sc-adapter-r3`: pass,
  release allowed. DRC 0 errors, 0 warnings, 0 unconnected.
- **J1 fan-out (user decision 2026-11-21).** Every J1 CSI pin gets its own
  0.3/0.15 mm via, placed between the two 0.4 mm pad rows, as on the OV9782
  adapter. A J1-courtyard clearance rule of 0.15 mm (the pads themselves are
  0.17 mm apart) lives in `stamp-p4-sc035hgs-adapter.kicad_dru`. Everywhere
  else the netclass clearance stays 0.2 mm and the fab floor 0.127 mm.
- **CSI pairs.** 0.2213 mm at 0.45 mm pitch on F.Cu over In1 (about 102 Ω on
  JLC's calculator), necked to 0.1779 mm inside the J1 courtyard. The pin
  order forces one crossing (J1 west to east: D1, CLK, D0; J2: CLK, D1, D0), so
  CLK runs under D1 on B.Cu at 0.25/0.5 mm (100 Ω over In2) and rises beside
  J2's GND pins. D0 and D1 each make a short B.Cu hop as well, so all six
  lanes carry three vias: kicad-auto's match-group check gives every via
  ±5.9 ps, and only equal via counts fit the 4.5 ps inter-pair budget.
  Measured: intra-pair skew ≤ 0.07 mm, pair-to-pair delay spread about 0.1 mm
  equivalent, every layer change within 2 mm of a GND via.
- **Deviations from a router-only board.** The J1 fan-out, the three pairs,
  the low-speed J1 escapes and two GND stitching vias are recorded manual
  copper (`.kicad-auto/manual-copper/`): the pair router cannot plan the
  crossing or a pin-field fan-out, and the stitching planner found no site at
  two layer changes. The other nets come from the negotiated router.
- Open `should` items: GND plane slots where the J1 via field and clustered
  power vias chain their antipads, the R8/R9 I2C pull-up stubs (7–9 mm), the
  MCLK route (1.66× its straight span), and layer changes of low-speed nets
  without a GND via within 2 mm.
- H1/H2 are excluded from the BOM and position files in the schematic, so
  board updates keep the exclusion.

### Main board (`main-board/`)

- Routed in workspace `pcb-sp4sc-main-r1` (checkpoint `routed-r3-keepout`),
  then `pcb-sp4sc-main-r2` after the H1–H6 attribute update; review
  `pcbrev-sp4sc-main-r5`: pass, release allowed, schematic parity pass. DRC 0
  errors, 0 unconnected. PWR_5V is 0.5 mm.
- Open `should` items: eight SD/CD signal layer changes without a GND via
  within 2 mm, the GND planes necking between clustered SD vias, the SD
  pull-up stubs (12–15 mm), the DM3D-SF courtyard leaving part of
  its body out, and the Stamp-P4 footprint having no courtyard.
- H1–H6 are excluded from the BOM and position files in the schematic. The
  first sync had dropped the exclusion because the generated mounting-hole
  symbols were `in_bom yes`; fixing the symbols and updating the routed board
  from the schematic restored it without touching copper.

## Assembly and bring-up

- **Fab options.** 4-layer JLC04161H-7628, 1.6 mm, ENIG. The adapter uses
  0.3/0.15 mm vias; 0.15 mm holes are a paid JLCPCB option, so confirm the via
  size and the price on the order form. The main board's smallest via is the
  standard 0.45/0.2 mm.
- **Adapter retention.** Flight units: M2 screws through adapter H1/H2 and main
  board H5/H6 with spacers. The stack from the main board's top to the
  adapter's bottom is about 1.69 mm (castellation joint 0.05 + module PCB
  0.84 + A4S mated height 0.80, roughly ±0.25 mm); shim each unit until J2 is
  fully seated without bending the adapter. Ground units may use tape.
- **CAN termination.** JP1 (CAN_TERM) is an open solder jumper for the 120 Ω
  terminator. Bridge it only when this board is an end of the CAN bus (SELFY
  is at the end in the C-109J harness).
- **Stamp-P4 supply.** The module carries v1.x ESP32-P4 silicon, which is on
  its end-of-life path: buy spare modules now, pin the ESP-IDF version and
  build for the v1.x chip revision. Measure the module's 3.3 V (SOC_3.3V)
  current while recording at bring-up (decision 2).
