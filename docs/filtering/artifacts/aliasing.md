# Aliasing

Aliasing appears as stair steps,
jagged edges, broken diagonals,
or lines that shimmer during motion.
It occurs when the sampled image
cannot represent the original high-frequency shape correctly.

## Identify the cause first

### Poor scaling

An unsuitable resize kernel or scale factor
can turn smooth lines into regular steps.
If the source was upscaled from a known lower resolution,
a correct [descale](../common/descaling/theory.md)
may reverse the cause more faithfully than anti-aliasing.

### Rendering

3D elements rendered without sufficient anti-aliasing
can contain jagged high-contrast geometry.
The missing samples were never present,
so filtering can only make the result less distracting.

### Binarized line art

Scanned or digitally drawn lines may have been reduced
to hard black-and-white shapes
without a later smoothing pass.
This produces uneven edges throughout the artwork.

### Deliberate styling

Hard digital edges,
pixel art,
and rough line work can be intentional.
Consistent jaggedness that follows the design
should not be treated automatically.

## General anti-aliasing

For aliasing that cannot be reversed through descaling,
[`based_aa`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsaa/funcs/#vsaa.funcs.based_aa)
is a practical general treatment.
It supersamples the image,
applies edge-directed interpolation,
limits the result to areas where the interpolator changed the clip,
and merges it through an edge mask.

```py3
from vsaa import based_aa

antialiased = based_aa(clip)
```

!!! note "The function is already masked"

    [`based_aa`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsaa/funcs/#vsaa.funcs.based_aa)
    uses an internal edge mask by default.
    Do not add another generic edge mask merely because
    anti-aliasing is often described as a masked operation.
    Use `show_mask=True` to inspect its mask,
    or pass a custom mask when the default selection is unsuitable.

```py3
aa_mask = based_aa(clip, show_mask=True)
```

Additional limiting is still useful at the scene or frame-range level
when only part of a source needs anti-aliasing.
This avoids processing already-correct line art.

## Side effects to check

- Soft or thickened lines
- Lost intentional texture along edges
- New halos introduced by supersampling or interpolation
- Flicker where the mask changes between frames
- “Corrected” stylistic jaggedness

Strongly aliased rendering may not be repairable
without unacceptable line damage.
Partial improvement is often the correct goal.

## Related pages

- [Descaling theory](../common/descaling/theory.md)
- [Ringing and haloing](ringing-and-haloing.md)
- [Order of filtering operations](../general/order.md)
- [Comparison](../../misc/comparison.md)
