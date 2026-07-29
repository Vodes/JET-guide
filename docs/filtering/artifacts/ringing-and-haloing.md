# Ringing and Haloing

Ringing and haloing both appear near strong edges,
but their shapes and best treatments differ.

- **Ringing** forms repeated ripples or narrow bright/dark echoes.
- **Haloing** forms a broader outline or glow around an edge.

Both can affect luma or chroma.
Inspect individual planes when the color of the defect
does not match the line art.

!!! example "Ringing"

    ![Ringing affecting multiple planes in Little Busters! OVA](../img/artifacts/lbova-ringing.png)

## Common causes

Ringing may come from sharp resampling kernels,
lowpass filtering,
compression,
or sharpening overshoot.
Haloing is commonly associated with sharpening
or a poor upscale,
but a soft glow can also be an intentional compositing effect.

!!! example "Haloing"

    ![Haloing in Shakugan no Shana](../img/artifacts/shana-haloing.png)

Do not remove a consistent glow
that belongs to lighting or compositing.
An unwanted halo usually tracks high-contrast edges mechanically;
an artistic glow more often has scene-aware color,
falloff,
and direction.

## Prefer reversing the cause

If ringing came from an identifiable resize,
a correct [descale](../common/descaling/theory.md)
may remove it while restoring the earlier sampling grid.
This is preferable to blurring the rings after the fact.

Similarly,
a clean alternate source may let you reconstruct damage
from a horizontal lowpass.
Generic deringing cannot recreate frequencies that were discarded.

## Dehaloing

[`fine_dehalo`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdehalo/mask/#vsdehalo.mask.fine_dehalo)
is the recommended general starting point for true halos.
It uses a halo-oriented mask around strong edges
and can treat bright and dark halos separately.

```py3
from vsdehalo import fine_dehalo

dehaloed = fine_dehalo(clip)
```

Its [generated masks](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdehalo/mask/#vsdehalo.mask.FineDehalo.Masks)
can be inspected while tuning:

```py3
dehaloed = fine_dehalo(clip)
halo_mask = fine_dehalo.masks.MAIN
```

Increase strength or radius only after checking
line interiors,
thin details,
and intentional glows.

## Deringing and edge cleaning

[`hq_dering`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdehalo/denoise/#vsdehalo.denoise.hq_dering)
is aimed at repeated ringing near edges.
It smooths the ring region,
repairs and limits the filtered result,
and merges it through a generated ring mask.

```py3
from vsdehalo import hq_dering

deringed = hq_dering(clip)
```

[`edge_cleaner`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdehalo/warp/#vsdehalo.warp.edge_cleaner)
uses warping and repair
to clean disturbed edge neighborhoods.
It is worth testing for suitable edge damage,
but it is not the default choice for broad halos.

```py3
from vsdehalo import edge_cleaner

cleaned = edge_cleaner(clip, strength=5)
```

Choose by morphology:
`fine_dehalo` for clear halos,
`hq_dering` for repeated rings,
and `edge_cleaner` for edge-local damage
that responds well to controlled warping.

## Border ringing

Border ringing occurs along the image boundary
or along a hard letterbox border.
Resampling algorithms may have extrapolated black pixels
beyond the active image,
creating a false high-contrast edge.

!!! example "Image-border ringing"

    ![Border ringing in the Occultic;Nine ending](../img/artifacts/o9-border-ringing.png)

!!! example "Letterbox-border ringing"

    ![Letterbox ringing in the Occultic;Nine ending](../img/artifacts/o9-letterbox-ringing.png)

When this comes from an upscale,
descale with the correct border handling.
Be aware that chroma and luma borders
may not align after subsampling or padding.

## Horizontal lowpass damage

Lowpass filtering removes high-frequency information
and can introduce horizontal ringing around vertical edges.
It is common in older distribution chains.

!!! example "Horizontal lowpass"

    ![Horizontal lowpass damage in the Triangle Heart OVA opening](../img/artifacts/th3-lowpassing.png)

An FFT spectrum can support the diagnosis:
horizontal frequency loss creates a characteristic gap.

!!! example "FFT spectrum"

    ![FFT spectrum of the Triangle Heart OVA frame](../img/artifacts/th3-lowpassing-dft.png)

If a clean reference exists,
a source-specific difference reconstruction may restore
some of the lost line information.
Without a reference,
ordinary deringing can only suppress the visible ripples.

## HDCAM SR resampling

HDCAM SR workflows commonly resampled
1920×1080 material through 1440×1080 storage.
The horizontal downscale and upscale can create
strong horizontal-only ringing and frequency loss.

!!! example "HDCAM SR damage"

    ![Horizontal resampling damage in Little Busters! Refrain](../img/artifacts/lbrefrain-hdcam.png)

!!! example "HDCAM SR spectrum"

    ![FFT spectrum of the Little Busters! Refrain frame](../img/artifacts/lbrefrain-hdcam-dft.png)

This damage is often present in the master itself.
When an independent clean source exists,
it may serve as a reconstruction reference;
otherwise accept that some information is unrecoverable.

## Side effects to check

- Thin lines erased or reshaped
- Edge contrast reduced
- Intentional glows removed
- New aliasing from warping or supersampling
- A mask that mistakes rings for real edges
- Temporal changes in halo width or strength

## Related pages

- [Aliasing](aliasing.md)
- [Noise and grain](noise-and-grain.md)
- [Descaling theory](../common/descaling/theory.md)
- [Comparison](../../misc/comparison.md)
