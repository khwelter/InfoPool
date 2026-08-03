# Juki or "Yuki" Nozzle Vision Notes

## Naming note

I did not find strong public OpenPnP references for `Yuki` nozzles as a separate category. The public references I could verify are for `Juki` nozzles, especially Juki 50x style nozzles.

If `Yuki` was intended literally, treat the rest of this page as a likely spelling-adjacent result, not a guarantee. If you meant `Juki`, this is the relevant material.

## What OpenPnP officially documents for Juki nozzles

OpenPnP has explicit nozzle-tip calibration guidance for Juki nozzles.

The official `Nozzle Tip Calibration Setup` wiki page includes a dedicated `Juki-Nozzles` section in the pipeline notes. For older manual pipeline tuning it specifically calls out:

- `Threshold`
- `DetectCirclesHough`

and says the most relevant parameters are:

- threshold value
- mask-circle diameter
- Hough-circle diameter min/max

That is the clearest official OpenPnP guidance I found for Juki nozzle tip detection.

## What modern OpenPnP prefers

The same OpenPnP wiki page also says that newer OpenPnP versions now use a self-tuning Circular Symmetry pipeline for nozzle-tip calibration, and recommends using `Issues & Solutions` instead of manually hand-editing the old pipeline unless you specifically need manual control.

So the decision tree is:

1. On current OpenPnP, prefer `Issues & Solutions` and the stock nozzle-tip calibration pipeline.
2. Only drop to manual pipeline tuning if the default calibration is not stable enough for your nozzle geometry or lighting.

## The strongest Juki-specific OpenPnP hint

OpenPnP's own `NozzleTipSolutions` source contains a Juki-specific background calibration recommendation:

- `BrightnessAndKeyColor` is described as the right choice when the nozzle tip and/or shade is color-keyed
- the in-code description explicitly says: `Use for green Juki style nozzles.`

That is stronger than a generic wiki note, because it is the behavior OpenPnP presents in the actual `Issues & Solutions` flow.

## Practical calibration guidance for Juki nozzles

### 1. Enable nozzle-tip calibration first

Background calibration in OpenPnP depends on nozzle-tip calibration already being enabled and functioning.

The official wiki says nozzle-tip calibration is used to determine:

- nozzle/tip runout
- tool-specific bottom-camera offset
- background characteristics for vision masking

### 2. Prefer `BrightnessAndKeyColor` for green Juki-style nozzles

If your Juki nozzle or shade provides a strong dominant green key color, use:

- `Background Calibration Method = BrightnessAndKeyColor`

Why:

- the OpenPnP wiki says Juki-like nozzle tips benefit from color-key analysis in HSV space
- OpenPnP source explicitly labels this method as appropriate for green Juki-style nozzles

### 3. Fall back to `Brightness` if color-keying is weak

If the nozzle or background is not consistently color-keyed, use:

- `Background Calibration Method = Brightness`

This keeps the brightness cutoff logic without requiring a stable hue signature.

### 4. Keep `MaskHsv` in the pipeline

OpenPnP's background calibration only controls the pipeline correctly if the bottom-vision pipeline still contains a `MaskHsv` stage. The official background-calibration wiki calls this out explicitly.

If you remove `MaskHsv`, the automatic background calibration will no longer be able to drive those HSV bounds for you.

### 5. Let OpenPnP control the pipeline properties it is designed to own

When background calibration is enabled, OpenPnP automatically controls key parts of the pipeline, including:

- `BlurGaussian.kernelSize` via `Minimum Detail Size`
- `MaskCircle.diameter` via nozzle-tip part-diameter settings
- `MaskHSV` hue, saturation, and value bounds via the calibrated background box

This matters because manual edits to those same fields may appear to "not stick" when calibration is active. That is expected behavior.

### 6. Use the nozzle-tip calibration pipeline editor for manual tuning only when needed

If you do need manual tuning, OpenPnP's nozzle-tip calibration wizard exposes the calibration pipeline editor directly, and the pipeline receives useful bound properties including:

- `nozzleTip.diameter`
- `nozzleTip.maxDistance`
- `nozzleTip.center`
- `MaskCircle.center`

Those properties are injected by the calibration code and are part of why the stock pipeline adapts reasonably well.

## Lighting and image-quality guidance

OpenPnP's background diagnostics are unusually concrete here. The source-generated diagnostic messages recommend:

- eliminate highlights and reflections
- use a shade behind the nozzle
- renew blackening on dark nozzle-tip areas when needed
- clean the nozzle tip
- if the tip is shiny, make it dull
- check camera white balance when using color-keying
- check exposure if the image is too dark

For Juki nozzles this is especially relevant because metallic reflections and worn surfaces can make circle detection unstable even when geometry is fine.

There is also a public OpenPnP issue about camera light control during vision operations, with example scripting to switch top and bottom camera lights during captures. That is relevant whenever nozzle-tip or bottom-vision images are being contaminated by the wrong light source.

## If you are using an older manual Hough-circle style setup

The legacy OpenPnP wiki guidance for Juki nozzles boils down to this:

- set the threshold only as high as needed to keep the nozzle tip circular without bleeding
- constrain Hough-circle min/max diameters tightly around the real nozzle feature size
- keep the mask-circle diameter appropriate so irrelevant periphery is removed

That is still useful when a stock current pipeline needs interpretation during debugging.

## Public user references I could verify

These are public, user-maintained references that are directly relevant to Juki nozzles in OpenPnP-adjacent builds.

### Genie Kobayashi

Repository:

- https://github.com/geniekobayashi/juki_nozzle_changer

What it shows:

- a Juki 50x automatic nozzle changer for OpenPnP
- use with Ray Kholodovsky's custom holder
- mechanical nozzle holding / release sequence
- printable changer geometry and gang mounting ideas

This is a hardware reference rather than a pipeline preset, but it is a strong public user example for Juki nozzle use in OpenPnP.

### crono2250

Repository:

- https://github.com/crono2250/mounter_ZCaxis_head_4juki-nozzles

What it shows:

- an OpenPnP head assembly using four Juki nozzles
- design notes about XY tolerance, magnetic pickup, and nozzle-head mechanics

Again, this is primarily a mechanical reference, but it is a verified public user build using Juki nozzles in an OpenPnP context.

### Karl Ekdahl reference chain

Genie Kobayashi's repository also cites Karl Ekdahl's automatic nozzle changer video as inspiration:

- https://vimeo.com/144454866

That is not a pipeline guide, but it is part of the public user knowledge trail around Juki-style nozzle-change hardware in OpenPnP builds.

## What I did not find

I did not find a strong public repo or wiki page that publishes a ready-to-import OpenPnP pipeline XML specifically labeled for Juki nozzles by a user.

So the strongest defensible documentation is:

- official OpenPnP nozzle-tip calibration and background-calibration guidance
- OpenPnP's own Juki-specific in-app recommendation for `BrightnessAndKeyColor`
- public user hardware references showing Juki nozzles are used in real OpenPnP builds

## Practical recommendation

For current OpenPnP versions, the most defensible starting point for Juki nozzles is:

1. Use the stock nozzle-tip calibration flow from `Issues & Solutions`.
2. Set background calibration to `BrightnessAndKeyColor` if your nozzle or shade is green and consistent.
3. Keep `MaskHsv` in the bottom-vision pipeline.
4. Only tune threshold / Hough-circle details manually if the stock self-tuning path is not stable.
5. Fix lighting and reflections before over-tuning the pipeline.

That ordering matches both the official wiki guidance and what OpenPnP's own code is designed to do.