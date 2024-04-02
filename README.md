# nand-pcb-test

Homemade PCB test using a CNC router.

Hopefully this is useful to someone else wanting a single place to see the whole CNC PCB building process.
These notes assume you have a basic understanding of Kicad and Candle (grblcontrol).

TODO: front and back finished PCB

## Why?

There are plenty of companies that offer cheap PCB manufacturing, but I'm impatient and bad at designing things correctly on the first try.
Imagine ordering a PCB, waiting a few days, and figuring out you messed up your design.
Manufacturing with a CNC router helps to eliminate waiting time between idea and implementation.

Alternatively, plenty of people use the chemical etching process (example: thermal transfer paper and ferric chloride) to build their PCBs.
I just don't have a good workspace to mess with nasty chemicals, so the CNC approach is more doable for me.
Maybe I'll also add a chemical etching bonus section in the future.

## Equipment

- [Genmitsu 3020-PRO MAX V2 CNC Router](https://www.sainsmart.com/collections/new-genmitsu-collection/products/3020-pro-max-v2)
  - [3040 Y-Axis Extension Kit](https://www.sainsmart.com/collections/genmitsu-cnc-replacement-upgrade-parts/products/3020-yaxis-extension-kit) (optional)
- [FR-4 Copper Clad PCB Laminate Board, Single Side, 4 x 2.7 inch (10 piece)](https://www.amazon.com/dp/B01MCVLDDZ)
- Bits
  - [0.5mm PCB End Mill Bit](https://www.amazon.com/gp/product/B073RJ3SYZ)
  - [0.8mm PCB Drill Bit](https://www.amazon.com/gp/product/B091FD44Y3)
  - [2.5mm End Mill Bit](https://www.amazon.com/gp/product/B07P84V888)
  - [Assorted PCB Drill Bit Set](https://www.amazon.com/gp/product/B08M5PLF48)
- [Mini Dry Iron](https://www.amazon.com/gp/product/B09YXVWSTQ)
- [Thermal Transfer Paper](https://www.amazon.com/gp/product/B07GLGL2CX)
- [Monochrome Laser Printer](https://www.amazon.com/gp/product/B00NQ1CLTI)
- [Mini Drill Press](https://www.amazon.com/gp/product/B0BKZRD2P7)

<img src="images/cnc.jpg" alt="cnc router" width="50%" height="50%"/>

## Build Process

### Circuit Design and Prototyping

- Design circuit in Kicad Schematic Editor
- Breadboard implementation
- Protoboard implementation (optional)
- Kicad Schematic Editor
  - Inspect > Electrical Rules Checker
  - Tools > Annotate Schematic
  - Tools > Assign Footprints
  - File > Plot > PDF, Plot All Pages

<img src="images/breadboard.jpg" alt="breadboard" width="50%" height="50%"/>

<img src="images/protoboard.jpg" alt="protoboard" width="50%" height="50%"/>

<img src="images/kicad-schematic.png" alt="kicad schematic" width="50%" height="50%"/>

### PCB Design

- Open Kicad PCB Editor (or from Kicad Schematic Editor: Tools > Update PCB From Schematic)
- Generally only use back copper and front silkscreen
- Rearrange rats nest of components to eliminate overlap
- Set design rules (File > Board Setup)
  - Minimum clearance: 0.5mm
  - Minimum track width: 0.5mm
  - Copper to hole clearance: 0.5mm
  - Copper to edge clearance: 0.5mm
  - Minimum through hole: 0.7mm
  - Hole to hole clearance: 0.5mm
- Route tracks on `B.Cu` (back copper) layer. Ignore ground pins, will be filled later
- Select `Edge.Cuts` layer, draw a rectangle to trace board boundary, thickness 1mm
- Select `B.Cu` layer and add fill for ground
- Right click fill edge, Zones > Fill Zone
- Add orthogonal dimensions to `User.Drawings` layer to double check board width/height
- Place origin at top left of board
- Run Design Rules Checker
- If exporting `F.Silkscreen`, I recommend setting text thickness to 0.1mm 
- Export Gerber files (File > Plot)
  - Check Use drill/place file origin
  - Export `B.Cu`, `Edge.Cuts` (used in router step)
  - Export `B.Mask` (used in solder masking step) as PDF
  - Generate Drill Files (`*-NPTH.drl` and `*-PTH.drl`)
    - Drill Origin = Drill/place file origin
    - Drill Units = Millimeters
- Print (File > Print) `F.Silkscreen` and `F.Mask` layers to PDF (used in silkscreen step)
  - Verify Print One Page Per Layer is unchecked
  - Scale 1:1
  - Select Print Mirrored

<img src="images/kicad-pcb.png" alt="kicad pcb" width="50%" height="50%"/>

<img src="images/kicad-3d-front.png" alt="kicad pcb 3d front" width="50%" height="50%"/>

<img src="images/kicad-3d-back.png" alt="kicad pcb 3d back" width="50%" height="50%"/>

### Bits

Refer to bits listed in Equipment section above.

- Isolation routing: 0.5mm pcb milling bit
- Holes: 0.8mm drill bit
- Edge cut: 2.5mm end mill bit

I know a lot of people seem to recommend v-bits, but I found them to cut too
inconsistent to quickly throw together boards. 

I did manage to get a couple boards cut with a 0.1mm 60deg bit after manually offsetting my heightmap by 0.05 increments, but I could not replicate this every time. 
Switching to the pcb milling bit removed ~5-10 minutes of manual adjustment.

### Computer Aided Manufacturing (CAM)

- Launch FlatCAM
- Load `B.Cu` and `Edge.Cuts` gerber files (File > Open Gerber)
- Add drill files (`*.drl`) (File > Open Excellon)
  - Note: Ignore "No geometry found in file" error if no NPTH holes needed
- Options tab
  - Set units to mm
- Mirror on y-axis since we're cutting the back copper
  - Tool > Double-Sided PCB Tool
  - Mirror `B.Cu` and `*.drl` files on y-axis using Point/Box
- Select `*-B_Cu.gbr`
  - Isolation Routing section
  - Tool dia: 0.45 (note: 0.05 less than clearance set in Kicad)
  - Width (# passes): 2
  - Pass overlap: 0.10
  - Combine Passes: yes
  - Click Generate Geometry
- Select `*-B_Cu.gbr_iso`
  - Create CNC Job section
  - Cut Z: -0.015
  - Travel Z: 2
  - Feed Rate: 85.0
  - Tool dia: 0.45 (note: 0.05 less than clearance set in Kicad)
  - Spindle speed: 10000
  - Multi-Depth: yes
  - Depth/Pass: 0.08
  - Click Generate
- Select `*-B_Cu.gbr_iso_cnc`, export G-Code to `B_Cu.nc`
- Select `*-PTH.drl`
  - If too many sizes of drills, go back to Kicad and change hole sizes of individual components (we can always redrill by hand later if holes too small)
  - Create CNC Job section
  - Cut Z: -2
  - Travel Z: 2
  - Feed Rate: 100.0
  - Tool Change: no (unless using multiple drill sizes)
  - Spindle Speed: 10000
  - Click Generate
- Select `*-PTH.drl_cnc`, set tool dia to 0.8, export G-Code to `PTH.drl.nc`
- Select `*.Edge_Cuts.gbr`
  - Board cutout section
  - Tool dia: 2.5
  - Margin: 0.1
  - Gap size: 1.0
  - Gaps: 4
  - Click Generate Geometry
- Select `*-Edge_Cuts.gbr_cutout`
  - Create CNC Job section
  - Cut Z: -2
  - Travel Z: 2.0
  - Feed Rate: 50.0
  - Tool dia: 2.5
  - Spindle speed: 10000
  - Multi-Depth: yes
  - Depth/pass: 0.25
  - Click Generate
- Select `*-Edge_Cuts.gbr_cutout_cnc`, export G-Code to `Edge_Cuts.nc`

<img src="images/pcb-routed.jpg" alt="routed pcb" width="50%" height="50%"/>

### Preparing Silkscreen

- Open `F.Silkscreen` PDF in Inkscape (to remove blank space of PDF document)
  - Select silkscreen components, export
  - Export Selection tab, Export Selected Only, 300.0 DPI, SVG

### CNC Routing

Refer to bits in "Computer Aided Manufacturing (CAM)" section

To secure the PCB to the bed, 
I used 3D printed M6 Plate Clamps from https://www.printables.com/model/250450-enhancements-for-sainsmartgenmitsu-3020-pro-max-cn/files. 
Alternatively, I saw a lot of people using carpet tape.

Probe setup using aluminum tape and z-probe that came with CNC router.

<img src="images/cnc-probe.jpg" alt="cnc probe setup" width="50%" height="50%"/>

- Jog machine to upper left corner, z probe, zero XY
- Height map
  - Load nc file, click auto
  - Zero XYZ, set bit at origin
  - Probe grid, set to be ~10mm apart
  - Interpolation grid: 20, 20
  - zt = 1.0, zb = -1.0
  - Click Probe
  - Save as `height.map`
  - Click edit mode to leave edit mode
  - Verify use heightmap is checked
- Remove probes
- Load `B.Cu.nc`
- Swap to drill bit, re-zero Z
- Load `PTH.drl.nc`
- Swap to edge cut bit, re-zero Z
- Load `Edge_Cuts.nc`

### Silkscreen

- Load thermal transfer paper into printer
- Print `F.Silkscreen` SVG onto thermal transfer paper (smooth side up)
  - Ensure scale is 100%
- Tape thermal paper to front of PCB (toner side down)
- Use highest setting of dry iron to heat evenly for 2-3 minutes
- Apply polycrylic to preserve silkscreen

<img src="images/silkscreen.png" alt="silkscreen" width="50%" height="50%"/>

### Final Steps

- Use mini drill press to adjust any holes for larger through hole parts
- Use 400-600 grit sandpaper and/or steel wool to remove any small burrs on back copper
- File down tabs on sides of PCB
- Clean off with a bit of isopropyl alcohol
- Use multimeter to check all tracks for shorts
- Solder components
- Clean off any flux with isopropyl

TODO: front and back finished PCB

## References

- [Candle](https://github.com/trasz/grblControl)
- [FlatCAM](http://flatcam.org/)
- [GCODE Reference](https://marlinfw.org/meta/gcode/)
- Youtube
  - [Homemade custom PCB guide using free KiCAD software; Teaching Tech](https://www.youtube.com/watch?v=NgDXPWaA5Ic)
  - [Homemade PCB using thermal transfer paper](https://www.youtube.com/watch?v=aemG_4sDz_Q)
  - [Intro to Kicad; Shawn Hymel](https://www.youtube.com/playlist?list=PL3bNyZYHcRSUhUXUt51W6nKvxx2ORvUQB)
  - [Machining a PCB on the 3-axis CNC Bridgeport Mill; Usagi Electric](https://www.youtube.com/watch?v=AB84_vbH_e8)
  - [Milling PCBs on a Homemade CNC (Part 2): Masking](https://www.youtube.com/watch?v=qIxvXU7KDmE)
  - [Milling Printed Circuit Boards (PCBs) on a Cheap CNC Machine; Matt's Electronics & RC](https://www.youtube.com/watch?v=bQ6_oYZrsjk)
  - [Relay Calculators: Episode 8 - Using FlatCAM and a CNC mill to make PCBs; Usagi Electric](https://www.youtube.com/watch?v=F2FRN5z2S78)
  - [Sainsmart Genmitsu 3020 Pro Max - 300W Spindle & Linear Rails - Build, Test & Review; techydiy](https://www.youtube.com/watch?v=5vaAthrJQm0)
