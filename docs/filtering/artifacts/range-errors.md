# Levels and Range Errors

A range error occurs when sample values
and their interpretation disagree,
or when processing maps values into the wrong interval.
The result may look washed out,
over-contrasted,
too dark,
too bright,
or clipped.

## Three different problems

### Incorrect metadata

The pixel values are correct,
but `_ColorRange` tells the renderer to interpret them incorrectly.
Fixing this case means retagging metadata only.

### Incorrect numerical range

The samples were expanded or compressed incorrectly.
The image must be converted numerically
from the range it actually uses to the intended range.
Changing only the tag leaves the wrong values untouched.

### Clipping

Values beyond a limit were clamped together.
Once distinct highlights or shadows become the same sample value,
a range conversion cannot reconstruct the missing detail.

## Diagnose before converting

Check all of the following:

- Frame properties and container metadata
- A waveform, histogram, or pixel-value inspection
- Whether black and white detail is merely displayed incorrectly or absent
- Other releases of the same material
- Whether only some scenes or frames use the bad range

Legal-range excursions are not automatically errors.
Filters,
overshoot,
and real mastering values can extend beyond nominal black and white.
Do not clamp them merely because they cross a legal-range marker.

## Retag metadata

Retagging changes frame properties without changing pixels.
Use it only when the stored values are already correct.
Use
[`ColorRange`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vstools/enums/color/#vstools.enums.color.ColorRange)
or VapourSynth's
[`SetFrameProps`](https://www.vapoursynth.com/doc/functions/video/setframeprops.html).

=== "vs-tools"

    ```py3
    from vstools import ColorRange

    # The samples already contain full-range values.
    retagged = ColorRange.FULL.apply(clip)
    ```

=== "Vanilla VapourSynth"

    ```py3
    # In the _ColorRange frame property, 0 means full range.
    retagged = clip.std.SetFrameProps(_ColorRange=0)
    ```

The preview will change because the renderer now interprets
the same values using different metadata.

## Convert sample values

When limited-range values need to become full-range values,
tag the actual input correctly
and request a numerical conversion:
[`ColorRange`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vstools/enums/color/#vstools.enums.color.ColorRange)
and VapourSynth's [`resize`](https://www.vapoursynth.com/doc/functions/video/resize.html)
use different numeric range conventions.

```py3
from vstools import ColorRange, core

limited = ColorRange.LIMITED.apply(clip)
converted = core.resize.Point(
    limited,
    range=ColorRange.FULL.value_zimg,
)
```

Use the inverse source and destination
when converting full range to limited range.
For a longer filter chain,
convert to a high-precision working format first
and dither only when reducing precision at the end.

!!! danger "Range enums use different numeric conventions"

    The `_ColorRange` frame property uses `0` for full
    and `1` for limited.
    zimg's `range` argument uses the opposite numeric convention.
    Prefer named enums and `.value_zimg`
    rather than raw integers.

Do not use `range_in` to override contradictory frame properties:
properties take precedence.
Explicitly apply the correct source property before conversion.

## Irregular levels

Some damaged sources use a nonstandard mapping,
such as luma values spanning 0–235.
These require a deliberate levels transform
based on measured input and intended output.
Perform the calculation at high precision
and handle luma and chroma independently when their ranges differ.

Avoid publishing a copied `std.Levels` recipe
without its exact source assumptions.
A wrong transform can create clipping,
rounding banding,
and color shifts.

## Side effects to check

- Clipped highlight or shadow detail
- Raised blacks or reduced contrast
- A double conversion caused by incorrect source metadata
- Banding from low-precision level changes
- Different range behavior in only part of the source
- Luma corrected while chroma remains mis-scaled

## Related pages

- [How do I retag or convert color information?](../../basics/how-do-i.md#how-do-i-retag-a-clips-color-matrixcolor-rangeetc)
- [Banding](banding.md)
- [Comparison](../../misc/comparison.md)
