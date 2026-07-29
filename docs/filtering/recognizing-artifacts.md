# Recognizing Artifacts

An artifact is an unintended defect in a video.
It may come from production,
mastering,
format conversion,
or compression.
Before filtering anything,
identify both what the defect looks like
and what probably caused it.
Different causes can look similar
but require very different treatment.

!!! warning "Recognition comes before filtering"

    Do not filter an artifact merely because it is present.
    Every filter can remove wanted detail
    or introduce a new defect.
    Compare the result at normal playback speed
    as well as frame-by-frame,
    and leave the source alone
    when the trade-off is not worthwhile.

## Quick reference

| What you see | Likely issue | Where to look |
| --- | --- | --- |
| Steps or contours in a smooth gradient | Banding | [Banding](artifacts/banding.md) |
| Random, unstable, or blotchy texture | Compression noise | [Noise and grain](artifacts/noise-and-grain.md) |
| A regular grid of square regions | Blocking | [Blocking](artifacts/blocking.md) |
| Stair-stepped or broken diagonal lines | Aliasing | [Aliasing](artifacts/aliasing.md) |
| Ripples or bright/dark outlines near edges | Ringing or haloing | [Ringing and haloing](artifacts/ringing-and-haloing.md) |
| Comb-like noise that changes between fields | Cross-field noise | [Cross-field noise](artifacts/cross-field-noise.md) |
| Crushed, washed-out, clipped, or incorrectly contrasted video | Range or levels error | [Levels and range errors](artifacts/range-errors.md) |

This table is a starting point,
not a substitute for examining the source.
Artifacts often overlap:
a bitrate-starved source may contain
blocking,
noise,
ringing,
and banding at the same time.

## Where defects enter the video

### Mastering defects

Mastering defects are baked into the supplied master.
They may originate in production software,
scanning or capture equipment,
compositing,
or later remastering.
Upscaling artifacts found in every consumer release
are likely to be mastering defects.

Comparing independent releases is often the best test.
If a defect occurs in the same place in every source,
it probably predates their individual encodes.

### Authoring defects

Authoring defects are introduced while preparing
a disc,
broadcast,
or stream.
Common examples include compression artifacts,
incorrect range metadata,
and poor deinterlacing.
They may differ between releases of the same material,
which is why source comparison matters.

!!! note "The categories are not absolute"

    A studio can bake compression or deinterlacing into a master,
    while a distributor can introduce scaling artifacts during authoring.
    The origin matters more than the label.

## Read the frame

Different image regions respond differently to filtering.

!!! example "Common regions within an animation frame"

    ![Screenshot of Hibiki from The iDOLM@STER](img/artifacts/imas-frame.png)

### Flat areas

Flat areas contain little pixel-to-pixel variation:
skies,
walls,
skin,
soft shadows,
and out-of-focus backgrounds.
They make banding,
blocking,
and low-frequency noise easy to see.

### Hard edges

Line art,
text,
geometry,
and strong shadow boundaries
contain high-frequency information.
They reveal aliasing,
ringing,
haloing,
and poor resampling,
but are also easily damaged by smoothing.

### Textures

Fabric,
foliage,
water,
and patterned backgrounds
contain structured detail across many frequencies.
An aggressive denoiser or debander
may mistake this detail for an artifact.

### Dither

Dither is controlled noise used to make quantization steps less visible.
It is not itself an artifact.
Dither is usually deliberately distributed,
while texture is structured
and compression noise often follows blocks,
edges,
or unstable source grain.

## Spatial and temporal clues

A spatial artifact is visible in a still frame.
Banding,
blocking,
ringing,
and most aliasing fit this description.

A temporal artifact is identified through motion or change over time.
Shimmering,
unstable noise,
and field-related defects may be subtle or invisible
in a single screenshot.
Always preview several frames around the problem.

## Things commonly mistaken for artifacts

### Grain and deliberate dither

Film grain,
creative grain,
and protective grain are intentional image components.
Removing them can change the visual character
and expose banding during re-encoding.
See [Noise and grain](artifacts/noise-and-grain.md)
before deciding to denoise.

### Deliberately jagged line art

Pixel-art styling,
rough animation lines,
and deliberately hard digital edges
should not automatically be anti-aliased.
Filtering cannot reconstruct artistic information
that was never present.

### Chromatic aberration

Colored fringes may be a lens effect
or a deliberate compositing choice.
They differ from chroma shift
because the separation varies with position and optical context
rather than moving an entire chroma plane uniformly.

### Bad drawings

An off-model or poorly drawn line is not a sampling defect.
Warping or anti-aliasing it may only produce
a smoother version of the wrong shape.

## Choosing a treatment

Prefer operations that reverse a known process.
A correct descale,
field reconstruction,
or range conversion
usually has a stronger justification
than generic smoothing.
Apply inverse operations early,
then follow the [order of filtering operations](general/order.md).

When no inverse is possible:

1. Start with the least destructive treatment.
2. Inspect masks and intermediate clips where available.
3. Limit filtering by plane, scene, or range when necessary.
4. Use the [comparison workflow](../misc/comparison.md)
   to check detail retention and temporal behavior.
