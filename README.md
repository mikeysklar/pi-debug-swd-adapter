# Pi Debug → SWD 2x5 Adapter

A tiny passive adapter that connects the $12 [Raspberry Pi Debug Probe](https://www.raspberrypi.com/products/debug-probe/)
"D" port (JST-SH 3-pin) to the standard **2x5 1.27 mm ARM Cortex Debug (SWD) header**
found on many Adafruit Metro and Feather boards.

No logic, no level shifting — the Debug Probe is fixed 3.3 V, so this is just three
signals routed across a 26.6 × 11 mm single-sided board:

| Debug Probe "D" (J1, JST-SH) | Cortex Debug 2x5 (J2) |
| --- | --- |
| 1 — SC (SWCLK) | 4 — SWCLK |
| 2 — GND | 3, 5 — GND |
| 3 — SD (SWDIO) | 2 — SWDIO |

J2 pins 1 (VTref), 6 (SWO), 7 (KEY), 8 (NC), 9 (GNDDetect) and 10 (nRESET) are
left unconnected.

**The whole thing was dictated to Claude as an idea and built from there.** Claude wrote
the KiCad schematic and PCB out as raw s-expression files by hand — no KiCad MCP server,
no skill, no reference design — sourced the components and dropped them in my shopping
cart. I did the fabrication: a [Bantam CNC](https://www.bantamtools.com/) to cut the
copper-clad blanks and an xTool fiber laser to etch the copper, burn the silkscreen, and
expose the solder mask over a manually applied resin layer. The stainless solder-paste
stencil was cut on the same fiber laser.

---

## Stencil, Boards and Components

![Laser-etched stencil with finished boards and parts](pics/pcb-stencil-components.jpeg)

The finished boards with solder mask and silkscreen, next to the stainless steel paste
stencil cut on the xTool fiber laser. Alongside are the CNC Tech 2x5 1.27 mm shrouded
box headers and the JST-SH 3-pin vertical receptacles.

## Etched Copper — Trace Check

![Backlit etched board showing the copper traces](pics/pcb-etched-tracecheck.JPG)

Freshly etched board held up to a light to check trace continuity and that nothing
bridged. All copper is on F.Cu only — single sided, no vias — with the GND pour doing
all the ground routing.

## CNC-Cut Blank

![Blank copper-clad PCB cut on the Bantam CNC](pics/pcb-blank.JPG)

The raw blank: 26.6 × 11 mm of copper-clad FR-4 milled on the Bantam CNC, with 3 mm
rounded corners and two 3.3 mm M2.5 mounting holes on a 20 mm spacing.

## KiCad 3D Render

![KiCad 3D render of the assembled board](pics/pcb-3d.png)

KiCad's ray-traced view of the finished design, showing the JST-SH receptacle, the 2x5
shrouded header, and the Data 70 silkscreen. This is what the board was supposed to
look like before any copper got cut.

## KiCad Board Layout

![KiCad PCB editor view of the F.Cu layer](pics/pcb-kicad-brd.png)

The original layout in the KiCad PCB editor — the oldest artifact here. Everything on
F.Cu, 0.2 mm tracks and clearance, GND filled zone across the whole board, and the
board name in copper so it etches as readable text.

---

## Repo Contents

```
kicad/
  pico-debug-swd-adapter.kicad_sch    schematic (hand-written s-expression)
  pico-debug-swd-adapter.kicad_pcb    board layout
  pico-debug-swd-adapter.kicad_pro    project
  pico-debug-swd-adapter.step         exported 3D model
  pi-debug-swd_recovered.xcs          xTool Creative Space laser job
  3dmodels/                           project-local STEP models
  etch/
    etch-FCu-MIRRORED-1to1.pdf        print at 100% for toner transfer / laser
    etch-FCu-reference.pdf            same, unmirrored, for visual checking
pics/                                 photos and renders above
```

## Bill of Materials

| Ref | Part | Description |
| --- | --- | --- |
| J1 | JST `BM03B-SRSS-TB` | 3-pin 1.00 mm vertical SMT receptacle — Debug Probe "D" port |
| J2 | CNC Tech `3220-10-0300-00` | 2x5 1.27 mm shrouded keyed SWD box header, SMT |

Plus two M2.5 screws if you want to mount it.

## License

[GPL v3](LICENSE)
