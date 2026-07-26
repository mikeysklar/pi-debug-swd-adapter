![Laser-etched stencil with finished boards and parts](pics/pcb-stencil-components.jpeg)

# pi-debug-swd-adapter

A passive adapter that connects the $12 Raspberry Pi Debug Probe to a standard 2x5 SWD connector.

## About

The [Raspberry Pi Debug Probe](https://www.raspberrypi.com/products/debug-probe/) is the
cheapest good CMSIS-DAP probe you can buy, but its **"D" port** is a 3-pin JST-SH
connector — while most target boards, including many Adafruit Metro and Feather boards,
expose debug on the standard **2x5 1.27 mm ARM Cortex Debug (SWD) header**. This board is
the 26.6 × 11 mm piece of copper that bridges the two.

**Zero active components.** The Debug Probe is fixed 3.3 V logic, so there's nothing to
level-shift and no VTref to sense. It's three signals and a ground pour: JST-SH receptacle
on one side, keyed shrouded box header on the other. The shroud key means you can't plug
the ribbon in backwards.

Design files are in [`kicad/`](kicad/).

### How it was made

The idea was **dictated to Claude**, which wrote the KiCad schematic and PCB out as raw
s-expression files by hand — no KiCad MCP server, no skill, no reference design provided —
then sourced the components and added them to my shopping cart. I did the fabrication: a
[Bantam CNC](https://www.bantamtools.com/) to cut the copper-clad blanks, and an xTool
fiber laser to etch the copper, burn the silkscreen, and expose the solder mask over a
manually applied resin layer. The stainless steel paste stencil was cut on the same fiber
laser.

## Pinout

| Debug Probe "D" (J1) | Cortex Debug 2x5 (J2) | Notes |
|---|---|---|
| 1 — SC | 4 — SWCLK | clock |
| 2 — GND | 3, 5 — GND | both grounds tied to the pour |
| 3 — SD | 2 — SWDIO | bidirectional data |
| — | 1 — VTref | **NC** — probe is fixed 3.3 V, no sensing |
| — | 6 — SWO | **NC** — Debug Probe "D" port has no trace pin |
| — | 7 — KEY | **NC** |
| — | 8 — NC | **NC** |
| — | 9 — GNDDetect | **NC** |
| — | 10 — nRESET | **NC** — reset the target yourself |

Use the **"D"** port on the probe, not "U" — that one's the USB-serial UART.

## Using it

The Debug Probe enumerates as CMSIS-DAP, so nothing here needs a custom driver.

```bash
# probe-rs
probe-rs list
probe-rs info --protocol swd
probe-rs run --chip RP2350 target.elf
```

```bash
# OpenOCD
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg -c "adapter speed 5000"
```

Swap the chip/target config for whatever you've actually got on the other end of the
ribbon cable.

---

## Assembled boards and stencil

![Laser-etched stencil with finished boards and parts](pics/pcb-stencil-components.jpeg)

The finished boards with solder mask and silkscreen, next to the stainless steel paste
stencil cut on the xTool fiber laser. Alongside are the CNC Tech 2x5 1.27 mm shrouded box
headers and the JST-SH 3-pin vertical receptacles.

## Etched copper — trace check

![Backlit etched board showing the copper traces](pics/pcb-etched-tracecheck.JPG)

A freshly etched board held up to a light to check trace continuity and that nothing
bridged. All copper is on F.Cu only — single sided, no vias — with the GND pour doing all
the ground routing.

## CNC-cut blank

![Blank copper-clad PCB cut on the Bantam CNC](pics/pcb-blank.JPG)

The raw blank: 26.6 × 11 mm of copper-clad FR-4 milled on the Bantam CNC, with 3 mm
rounded corners and two 3.3 mm M2.5 mounting holes on 20 mm centers.

## KiCad 3D render

![KiCad 3D render of the assembled board](pics/pcb-3d.png)

KiCad's view of the finished design — the JST-SH receptacle, the 2x5 shrouded header, and
the Data 70 silkscreen. This is what the board was supposed to look like before any copper
got cut.

## KiCad board layout

![KiCad PCB editor view of the F.Cu layer](pics/pcb-kicad-brd.png)

The original layout in the KiCad PCB editor, and the oldest artifact here. Everything on
F.Cu, 0.2 mm tracks and clearance, a GND filled zone across the whole board, and the board
name set in copper so it etches as readable text.

---

## Design files

Designed in [KiCad](https://www.kicad.org/) 9. The project lives in [`kicad/`](kicad/):

- `pico-debug-swd-adapter.kicad_pro` — project file
- `pico-debug-swd-adapter.kicad_sch` — schematic
- `pico-debug-swd-adapter.kicad_pcb` — PCB layout
- `pico-debug-swd-adapter.step` — 3D model export
- `pi-debug-swd_recovered.xcs` — xTool Creative Space laser job
- `3dmodels/` — project-local STEP models so renders work anywhere
- `etch/etch-FCu-MIRRORED-1to1.pdf` — **print this at 100%** for toner transfer
- `etch/etch-FCu-reference.pdf` — same artwork unmirrored, for visual checking

Board rules are single-sided home-etch friendly: 0.2 mm tracks and clearance, 0.1 mm
copper-to-edge, everything on F.Cu, no vias, no through-hole parts.

## Bill of materials

| Ref | Part | Description |
|---|---|---|
| J1 | JST `BM03B-SRSS-TB` | 3-pin 1.00 mm vertical SMT receptacle — Debug Probe "D" port |
| J2 | CNC Tech `3220-10-0300-00` | 2x5 1.27 mm shrouded keyed SWD box header, SMT |

Plus two M2.5 screws if you want to bolt it down, and a 2x5 1.27 mm ribbon cable to reach
the target.

## License

[GPLv3](LICENSE)
