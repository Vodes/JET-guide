# Blocking

Blocking appears as a grid of square regions
whose interiors or boundaries no longer match.
It is most visible in flat areas,
dark scenes,
motion,
and sources encoded at insufficient bitrate.

!!! example "Compression blocking"

    ![Blocking in an ending sequence from Guild no Uketsukejou desu ga](../img/artifacts/girumasu-blocking.png)

## Diagnosis

Codec transforms process rectangular regions.
When too much information is discarded,
neighboring blocks reconstruct differently
and their boundaries become visible.

Do not confuse blocking with:

- [Banding](banding.md), which follows gradients rather than a fixed grid
- large blotches from unstable [noise](noise-and-grain.md)
- point scaling, whose squares align with enlarged source pixels
- intentional tiled or pixel-art designs

View several consecutive frames.
Compression blocks commonly change with motion
and may become stronger around keyframe intervals.

## Treatment order

Mild blocking may be reduced as a side effect
of ordinary denoising or debanding.
Try that result before adding a dedicated filter.
Dedicated deblocking is more likely to soften real edges and texture.

### Deblock QED

[`deblock_qed`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/deblock/#vsdenoise.deblock.deblock_qed)
targets 8×8 block borders
and treats their interiors separately.

```py3
from vsdenoise import deblock_qed

deblocked = deblock_qed(clip)
```

It depends on the `deblock` plugin
and is best suited to clearly block-structured damage.
Tune `quant`,
`alpha`,
and `beta` only after checking the default result.

### DPIR

[`dpir.DEBLOCK`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/deblock/#vsdenoise.deblock.dpir)
can repair severe blocking,
but it is a neural restoration model
and can alter genuine image detail.
Treat it as a heavier option,
not the next automatic step.

```py3
from vsdenoise import dpir

deblocked = dpir.DEBLOCK(clip, strength=10)
```

Scene-specific strengths or zones are preferable
when only part of the source is badly damaged.

## Side effects to check

- Softened line art or text
- Removed texture inside blocks
- Temporal inconsistency between filtered and unfiltered scenes
- Neural detail that does not match adjacent frames
- Remaining block edges made more obvious by over-smoothing interiors

When the source is severely compressed,
complete reconstruction is impossible.
The least distracting compromise may retain some blocking.

## Related pages

- [Noise and grain](noise-and-grain.md)
- [Banding](banding.md)
- [Comparison](../../misc/comparison.md)
