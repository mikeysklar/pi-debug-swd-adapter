# CLAUDE.md — home-etched single-sided PCB recipe

How to design boards like this one for me. Component choices come from a BOM doc
(usually an Obsidian note); everything below is about how the board is built,
independent of the specific parts.

## Toolchain (no KiCad MCP server needed)

- KiCad 9 on macOS. Stock libraries live under
  `/Applications/KiCad/KiCad.app/Contents/SharedSupport/{symbols,footprints}`.
- **Schematic**: write the `.kicad_sch` s-expression directly. Embed the needed
  symbols in `lib_symbols` copied verbatim from the stock libs. Use local net
  labels + wire stubs (net names come out as `/NAME`). Symbol instance `path`
  UUIDs matter — the PCB footprints link back to them.
- **PCB**: script it with KiCad's bundled Python:
  `/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3`
  (imports `pcbnew` headlessly; ignore the wxApp assert noise). Keep the build
  script idempotent: reset the `.kicad_pcb` to a 4-line stub, then regenerate
  everything. Link footprints to the schematic with
  `fp.SetPath(pcbnew.KIID_PATH("/<symbol-uuid>"))` and name unconnected-pin nets
  `unconnected-(REF-PinName-PadN)` to match KiCad's netlister.
- **Verify every iteration** with `kicad-cli` (in the app bundle): `sch erc`,
  `pcb drc --exit-code-violations` (must be 0 violations, 0 unconnected),
  `pcb render --side top` for a visual, and check pad coordinates by printing
  them from the script rather than trusting rotation math.
- If KiCad has the file open, ask me to close without saving before writing it.

## Board rules (single-sided, home etch)

- All copper on **F.Cu only** — SMT parts only, no vias, no through-hole
  headers. Plan routing so nets are planar; tricks that work: run a trace down
  the center channel between the pad columns of a 1.27 mm box header
  (0.6 mm trace fits with ≥0.45 mm clearance), and let the GND pour do all GND
  routing.
- Clearance 0.2 mm, tracks 0.2 mm, copper-to-edge 0.1 mm (set on the default
  netclass + `m_CopperEdgeClearance`). Wide is good; this gets etched with
  toner transfer.
- **GND is a filled zone over the whole board** (less copper to etch away):
  thermal reliefs (gap 0.25, spoke 1, min width 0.2, clearance 0.2), island
  removal = always. Add a short GND stub track from any GND pad the pour can
  only approach from one side, and set pads to solid zone connection
  (`SetLocalZoneConnection(ZONE_CONNECTION_FULL)`) if DRC reports starved
  thermals. Fill with `ZONE_FILLER` before saving.
- **Mounting holes: plain 3.3 mm circles on Edge.Cuts** — no footprint, no
  copper rings, nothing else. Sized for M2.5 screws. Keep centers ~2.6 mm clear
  of part bodies (screw head room) and ≥1.6 mm of board material to the edge.
  Place hole centers on a 10 mm grid (both hole position and hole-to-hole
  spacing land on multiples of 10 mm).
- Component spacing: ~4 mm clear between connector bodies is enough for
  plugging/unplugging.
- Keep the board a compact rectangle; hole – parts – hole in a line works well.
- **Round the outer edge cuts** — no sharp corners. Replace the board outline's
  `gr_rect` with 4 `gr_line` edges (each shortened by the corner radius) plus 4
  `gr_arc` corners, all still on Edge.Cuts. Build each arc's `mid` point via the
  bisector of the two center→endpoint vectors (`SetArcGeometry(start, mid,
  end)` in pcbnew's API) rather than `SetArcAngleAndEnd` — the angle/end
  variant is ambiguous about winding direction and produces a self-intersecting
  outline half the time. ~3 mm radius suits boards in this size range; scale to
  taste. After changing the outline, also update the GND zone's own boundary
  `polygon` (not the `filled_polygon` — that's derived) to the same 4-arc shape
  so the zone's editable outline matches the board instead of overhanging past
  the rounded corners; refill with `ZONE_FILLER` and re-save so KiCad
  re-serializes both consistently. Verify with `pcb drc` (0 new violations)
  and `pcb render` before/after.

## 3D models

- Every footprint must have a working 3D STEP model so `pcb render` shows the
  whole board, not just some parts. After placing footprints, render and check
  visually for anything missing.
- If a footprint's `model` path (e.g. `${KICAD9_3DMODEL_DIR}/...`) doesn't
  resolve locally, the part's model is simply newer than what's bundled with
  the installed KiCad version — check the official
  `KiCad/kicad-packages3D` GitHub repo (and its distro mirrors, e.g.
  `deepin-community/kicad-packages3d`) for the exact same filename; verify the
  downloaded STEP file's header carries KiCad's own CC-BY-SA 4.0 license block
  before using it. Drop it in a project-local `3dmodels/<Library>.3dshapes/`
  folder and point the footprint's `model` entry at
  `${KIPRJMOD}/3dmodels/<Library>.3dshapes/<file>.step` so it travels with the
  project instead of depending on the local KiCad install.
- Don't touch the system KiCad library folders — only add files under the
  project's own `3dmodels/`.

## Labels

- Hide all reference designators and value fields
  (`fp.Reference().SetVisible(False)`, `fp.Value().SetVisible(False)`).
- Silkscreen = signal names only (SWDIO/SWCLK/GND style), one label per signal
  placed along its trace when space is tight, 0.8 mm text.
- Silk font is **Data 70** (installed system font, family name "Data 70").
  pcbnew's Python API can't set faces, so after `SaveBoard` patch the file:
  replace `(font\n\t\t\t\t(size 0.8 0.8)` with
  `(font\n\t\t\t\t(face "Data 70")\n\t\t\t\t(size 0.8 0.8)`. Keep any other
  text at a size ≠ 0.8 so the patch doesn't catch it. The font is referenced,
  not embedded — fine locally; embed or outline it before sharing the project.
- Board name goes in **copper**: stroke-font text placed in the pour (the zone
  auto-clears around it, so it etches as readable copper-on-substrate).

## Deliverables

- `etch/etch-FCu-MIRRORED-1to1.pdf` — print this at 100% for toner transfer
  (`kicad-cli pcb export pdf --layers F.Cu,Edge.Cuts --black-and-white --mirror`).
- `etch/etch-FCu-reference.pdf` — same, unmirrored, for visual checking.
- Regenerate both after any copper change.
