![Laser-etched stencil with finished boards and parts](pics/pcb-stencil-components.jpeg)

# pi-debug-swd-adapter

Connects the $12 Raspberry Pi Debug Probe to a standard 2x5 SWD connector.

## How it was made

Voice dictated to Claude, which sourced the parts and wrote out the KiCad 
schematic and PCB board file. No MCP server, no skill, no reference 
design.

See [`CLAUDE.md`](CLAUDE.md) for what it was used. This file represents the on-going changes I had Claude 
make after the first pass.

Fabrication: a [Bantam CNC](https://www.bantamtools.com/) to cut the single sided PCB blanks, 
and an xTool F1 Ultra fiber laser to etch, silkscreen, and expose the pads under a green 
resin layer was applied. The steel business card stencil was cut on the same laser.

## About

The [Raspberry Pi Debug Probe](https://www.raspberrypi.com/products/debug-probe/) ends in a
3-pin JST-SH connector. Many Adafruit Metro and Feather boards expose debug on the standard
2x5 1.27 mm SWD header. This board bridges the two.

No active parts — just three signals and a ground pour on a 26.6 × 11 mm single-sided board.

Design files are in [`kicad/`](kicad/).

---

## Assembled board

![Assembled adapter with both connectors soldered](pics/pcb-asm.JPG)

The finished prototype, backlit. Both connectors reflowed onto the laser-exposed pads, with
the copper silkscreen and board name still readable through the green resin.

## Working debugger setup

![Adapter connecting a Raspberry Pi Debug Probe to an Adafruit Metro M0 Express](pics/hookup.jpeg)

The whole chain in use: Debug Probe on the left, its 3-pin JST-SH lead into the adapter at
bottom right, and a 2x5 1.27 mm ribbon up to the SWD header on an Adafruit Metro M0 Express.

## KiCad 3D render

![KiCad 3D render of the assembled board](pics/pcb-3d.png)

What the board was supposed to look like before any copper got cut.

## KiCad board layout

![KiCad PCB editor view of the F.Cu layer](pics/pcb-kicad-brd.png)

The original layout, and the oldest artifact here. The board name is set in copper so it
etches as readable text.

## Etched copper — trace check

![Backlit etched board showing the copper traces](pics/pcb-etched-tracecheck.JPG)

A freshly etched board held up to a light to check the traces. All copper is on one side —
no vias.

## CNC-cut blank

![Blank copper-clad PCB cut on the Bantam CNC](pics/pcb-blank.JPG)

The raw blank milled on the Bantam CNC, with rounded corners and two M2.5 mounting holes.

---

## Design files

Designed in [KiCad](https://www.kicad.org/) 9. The project lives in [`kicad/`](kicad/):

- `pico-debug-swd-adapter.kicad_pro` — project file
- `pico-debug-swd-adapter.kicad_sch` — schematic
- `pico-debug-swd-adapter.kicad_pcb` — PCB layout
- `pico-debug-swd-adapter.step` — 3D model export
- `pi-debug-swd_recovered.xcs` — xTool Creative Space laser job
- `3dmodels/` — project-local STEP models
- `etch/etch-FCu-MIRRORED-1to1.pdf` — print at 100% for toner transfer
- `etch/etch-FCu-reference.pdf` — same artwork unmirrored, for checking

## Bill of materials

| Ref | Part | Description |
|---|---|---|
| J1 | JST `BM03B-SRSS-TB` | 3-pin 1.00 mm vertical SMT receptacle |
| J2 | CNC Tech `3220-10-0300-00` | 2x5 1.27 mm shrouded SWD box header, SMT |

Plus two M2.5 screws and a 2x5 1.27 mm ribbon cable.

## Pinout

| Debug Probe "D" (J1) | Cortex Debug 2x5 (J2) |
|---|---|
| 1 — SC | 4 — SWCLK |
| 2 — GND | 3, 5 — GND |
| 3 — SD | 2 — SWDIO |

Everything else on J2 is left unconnected. Use the **"D"** port on the probe, not "U".

## Using it

The Debug Probe shows up as CMSIS-DAP, so no special driver is needed. Connecting to the
Adafruit Metro M0 Express in the photo above, through this adapter:

```console
$ openocd -f interface/cmsis-dap.cfg -f target/at91samdXX.cfg \
    -c "adapter speed 400" -c "init; reset halt; flash probe 0; targets; shutdown"

Open On-Chip Debugger 0.12.0
adapter speed: 400 kHz

Info : Using CMSIS-DAPv2 interface with VID:PID=0x2e8a:0x000c, serial=E6614103E7687D25
Info : CMSIS-DAP: SWD supported
Info : CMSIS-DAP: FW Version = 2.0.0
Info : CMSIS-DAP: Interface Initialised (SWD)
Info : CMSIS-DAP: Interface ready
Info : clock speed 400 kHz
Info : SWD DPIDR 0x0bc11477
Info : [at91samd.cpu] Cortex-M0+ r0p1 processor detected
Info : [at91samd.cpu] target has 4 breakpoints, 2 watchpoints
Info : starting gdb server for at91samd.cpu on 3333
Info : Listening on port 3333 for gdb connections
[at91samd.cpu] halted due to debug-request, current mode: Thread
xPSR: 0x71000000 pc: 0x00000264 msp: 0x20002de0
Info : SAMD MCU: SAMD21G18A (256KB Flash, 32KB RAM)
shutdown command invoked
```

`SWD DPIDR 0x0bc11477` is the handshake landing. After that the probe halts the core and
reads the chip back as a SAMD21G18A, which is the part on the Metro M0.

The Debug Probe does not power the target, so bring the target up on its own supply and leave
it powered before starting OpenOCD. A target that is off gives
`Error: Error connecting DP: cannot read IDR`.

---

## License

[GPLv3](LICENSE)
