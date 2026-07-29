# Noise and Grain

“Noise” and “grain” both describe pixel variation,
but they are not interchangeable diagnoses.
The important question is whether the variation is
intentional image structure,
useful protection,
or damage introduced by a source or encode.

## Recognizing the difference

### Grain

Grain may come from film,
be added for a creative look,
or be added to protect gradients during encoding.
It usually forms a coherent texture across the image.
Dynamic grain changes from frame to frame;
static grain repeats.

Film and creative grain are part of the intended appearance.
Protective grain may be nearly invisible at normal size
but still prevent [banding](banding.md).

### Compression noise

Compression noise often looks blotchy,
block-shaped,
or unstable around edges and existing grain.
It may pulse with keyframes or motion
and can differ strongly between releases.

### DCT and mosquito noise

DCT noise collects around sharp transitions
as small moving dots,
false edges,
or short ripples.
It overlaps visually with [ringing](ringing-and-haloing.md)
and often accompanies [blocking](blocking.md)
in bitrate-starved material.

### Texture and dither

Texture follows objects and surfaces.
Dither is a deliberately distributed pattern
used to disguise quantization.
Pause,
advance several frames,
and compare other releases before deciding either is noise.

## Should it be fixed?

Denoising trades unwanted variation for lost detail.
Removing all grain is rarely a good target:
it changes the image,
can make animation look plastic,
and may expose banding.

A useful goal is to suppress the part
that is ugly or disproportionately expensive to encode,
then restore a controlled grain pattern if needed.
Scenes with damaged encoded grain
may require stronger local filtering than the rest of the video.

## A common modern-anime setup

The top-level `vsdenoise` documentation provides
a chain that is commonly used
and sufficient for most modern anime sources:

References:
[`mc_degrain`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/funcs/#vsdenoise.funcs.mc_degrain),
[`Prefilter`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/prefilters/#vsdenoise.prefilters.Prefilter),
[`bm3d`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/blockmatch/#vsdenoise.blockmatch.bm3d),
and [`nl_means`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/nlm/#vsdenoise.nlm.nl_means).

```py3
from vsdenoise import MVToolsPreset, Prefilter, bm3d, mc_degrain, nl_means

# Build a motion-compensated reference.
ref = mc_degrain(
    clip,
    prefilter=Prefilter.DFTTEST(),
    preset=MVToolsPreset.HQ_SAD,
    thsad=100,
)

# Denoise luma with BM3D.
denoised = bm3d(
    clip,
    sigma=0.8,
    tr=2,
    profile=bm3d.Profile.NORMAL,
    ref=ref,
    planes=0,
)

# Denoise chroma with NLMeans.
denoised = nl_means(
    denoised,
    h=0.2,
    tr=2,
    ref=ref,
    planes=[1, 2],
)
```

This combines a motion-compensated reference,
BM3D for luma,
and NLMeans for chroma.
The values are a somewhat conservative starting point,
not a preset that makes comparison unnecessary.
Check dark gradients,
line shading,
and moving texture.

### Higher-quality chroma alternative

[`wnnm`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/blockmatch/#vsdenoise.blockmatch.wnnm)
is a slower block-matching denoiser
that generally produces less blocking and ringing than BM3D.
It can replace NLMeans as a higher-quality chroma option
when the extra processing cost is acceptable.

```py3
from vsdenoise import wnnm

# WNNM requires a 32-bit float clip and matching reference.
denoised = wnnm(
    denoised,
    sigma=0.5,
    tr=2,
    ref=ref,
    planes=[1, 2],
)
```

Do not switch only one input to float:
the clip and reference passed to WNNM
must have matching dimensions and format.

## MVTools and motion compensation

[`MVTools`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/mvtools/mvtools/#vsdenoise.mvtools.mvtools.MVTools)
estimates how blocks move between frames
so temporal filtering can use aligned neighboring content.
It is central to many serious denoising workflows,
but its motion search and compensation controls
are too broad for an artifact-recognition page.

[`mc_degrain`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/funcs/#vsdenoise.funcs.mc_degrain)
is the easier vsdenoise wrapper
for a common MVTools degraining setup.
It handles vector analysis,
refinement,
and degraining,
and accepts `MVToolsPreset.HQ_SAD`
or `MVToolsPreset.HQ_COHERENCE`.
`HQ_SAD` is the wrapper's default.
The presets are defined by
[`MVToolsPreset`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/mvtools/presets/#vsdenoise.mvtools.presets.MVToolsPreset).

```py3
from vsdenoise import MVToolsPreset, Prefilter, mc_degrain

denoised = mc_degrain(
    clip,
    prefilter=Prefilter.DFTTEST(),
    preset=MVToolsPreset.HQ_SAD,
    thsad=100,
)
```

Motion compensation is not automatically safer than spatial filtering.
Bad vectors can smear detail,
drag noise through motion,
or create discontinuities around scene changes.
Use the wrapper unless the source gives you a concrete reason
to configure `MVTools` directly.

## Other useful denoisers

[`nl_means`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/nlm/#vsdenoise.nlm.nl_means)
is also useful on its own,
especially for chroma or spatially repetitive noise.

```py3
from vsdenoise import nl_means

denoised = nl_means(clip, h=0.8, tr=1)
```

For frequency-shaped noise,
[`DFTTest.denoise`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/fft/#vsdenoise.fft.DFTTest.denoise)
can target frequency bands more deliberately,
but it requires source-specific tuning
and is not a drop-in “better” denoiser.

```py3
from vsdenoise import DFTTest

denoised = DFTTest().denoise(clip, sigma=8.0, tr=0)
```

## Chroma noise

Chroma noise appears as colored blotches or crawling color
and often benefits from processing only the chroma planes.
The native YUV implementation in
[`zsmooth`](https://github.com/adworacz/zsmooth)
avoids an RGB round trip.
Its direct `CCD` signature is documented in the
[`zsmooth` filter reference](https://github.com/adworacz/zsmooth#ccd).

```py3
# A spatial starting point. Raise threshold only after comparison.
denoised = core.zsmooth.CCD(clip, threshold=2.0, temporal_radius=0)
```

`zsmooth.CCD` and
[`vsdenoise.ccd`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/funcs/#vsdenoise.funcs.ccd)
implementation
do not use identical `scale` semantics.
Do not transfer a tuned scale value blindly.

## Restoring grain

If denoising leaves the clip too flat
or exposes quantization,
add new dynamic grain after destructive filtering.

[`Grainer`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdeband/noise/#vsdeband.noise.Grainer)
is a wrapper around
the [`vs-noise`](https://github.com/wwww-wwww/vs-noise)
grain generators.
It adds useful processing around the raw noise layer,
including:

- protection near legal sample-range limits
- optional protection for neutral chroma
- luma-adaptive masking through `luma_scaling`
- temporal averaging and grain-size controls
- separate luma/chroma strengths and post-processing hooks

These controls help prevent grain from being added blindly
to every sample at the same strength.

```py3
from vsdeband import Grainer

output = Grainer.GAUSS(
    denoised,
    strength=0.5,
    static=False,
    protect_edges=True,
    luma_scaling=4,
)
```

### Available grain types

| Grainer member | Generator |
| --- | --- |
| `Grainer.GAUSS` | Gaussian noise |
| `Grainer.PERLIN` | Perlin noise |
| `Grainer.SIMPLEX` | Simplex noise |
| `Grainer.FBM_SIMPLEX` | Fractional Brownian motion over simplex noise |
| `Grainer.POISSON` | Poisson noise, intended for intensity-correlated grain |
| `Grainer.PLACEBO` | libplacebo grain rather than a `vs-noise` generator |

Perlin,
simplex,
and fractional Brownian motion
allow more structured grain than Gaussian noise.
For these types,
use `size` to control their native grain scale:
the [`Grainer` reference](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdeband/noise/#vsdeband.noise.Grainer)
lists the controls supported by each generator.

```py3
output = Grainer.PERLIN(
    denoised,
    strength=0.5,
    size=3.0,
    luma_scaling=4,
)
```

`Grainer.PLACEBO` currently uses
the transitional `vsdeband.placebo_deband` wrapper internally.
Its availability may therefore change
if that wrapper is removed.

Try to match the source's size,
strength,
and brightness response.
Inspect the luma-adaptive mask indirectly
by comparing bright and dark regions,
and disable or retune protections
only when the source calls for it.
Do not use new grain to hide avoidable filtering damage.

## Side effects to check

- Loss of fine texture or line shading
- Wax-like flat areas
- Ghosting or trails from temporal filtering
- Grain that changes character between filtered and unfiltered scenes
- Newly visible banding

## Related pages

- [Banding](banding.md)
- [Blocking](blocking.md)
- [Ringing and haloing](ringing-and-haloing.md)
- [Comparison](../../misc/comparison.md)
