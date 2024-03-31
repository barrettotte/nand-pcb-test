# Laser Notes

I was going to do a laser silkscreen, but I didn't like the workflow
and was struggling to keep things aligned when flipping over a freshly routed board.

## Notes

### Preparing Silkscreen (optional)

- Open in `F.Silkscreen` PDF in Inkscape
  - Select silkscreen components, export
  - Export Selection tab, Export Selected Only, 300.0 DPI, PNG
- Open exported png in LaserGRBL
  - Conversion Tool - Passthrough
  - Click Next
  - Engraving Speed: 1000 mm/min
  - Laser Options, Laser Mode: M3, S-MIN: 0, S-MAX: 800
  - Image Autosize 300 DPI
  - Click Create
  - File > Save Laser GCODE

### Laser Etching Silkscreen (optional)

- Launch LaserGrbl
- Grbl > Settings
  - Jog control tab - Show Z up/down control
- Grbl > Grbl Configuration
  - $30 = 1000 RPM (configures laser power)
  - $32 = 1 (laser mode)
- Turn on laser low power, adjust Z-axis until dot is as small as possible (finding the focal point)

TODO: ...

## References

- [LaserGRBL](https://lasergrbl.com/)
- [3018 Pro - Setting up your laser; James Dean Designs](https://www.youtube.com/watch?v=YnFNFEdmPjU)
- [3018 Pro - Basics for Laser Engraving; James Dean Designs](https://www.youtube.com/watch?v=pCmazk-yTVo)
