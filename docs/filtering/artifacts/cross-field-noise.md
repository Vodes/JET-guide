# Cross-Field Noise

Cross-field noise is a comb-like compression defect
found most often in hard-telecined MPEG-2 material.
Noise and edge remnants from one field
leak across the lines belonging to the other field.
It can resemble ordinary combing or aliasing,
but its structure follows the field layout
and may remain after an otherwise correct field match.

## Diagnose the field structure

First determine whether the source is interlaced,
telecined,
or progressive.
Read [Field-Based Video](../situational/fieldbased.md)
before changing field order or cadence.

Use VapourSynth's
[`SeparateFields`](https://www.vapoursynth.com/doc/functions/video/separatefields.html)
for inspection:

```py3
# Set tff=False for a bottom-field-first source.
fields = clip.std.SeparateFields(tff=True)
```

Advance through the separated fields
and look for short horizontal remnants
or compression noise that alternates with field parity.
Compare the same scene in another release when possible.

Do not diagnose every combed frame as cross-field noise.
Normal telecine combing disappears with a correct field match;
cross-field contamination is damage inside the encoded fields.

## Treatment

Perform IVTC or deinterlacing in the correct place
for the source's cadence.
Do not apply an ordinary progressive spatial smoother
across the woven scanlines before understanding the fields.

For cross-field compression artifacts in hard-telecined MPEG-2,
vs-jetpack provides
[`mpeg2stinx`](https://jaded-encoding-thaumaturgy.github.io/vs-jetpack/api/vsdenoise/deblock/#vsdenoise.deblock.mpeg2stinx):

```py3
from vsdenoise import mpeg2stinx

# The input is the progressive-framed, hard-telecined source.
# Set tff=False if the source is bottom-field-first.
repaired = mpeg2stinx(clip, tff=True)
```

The filter bobs the source to estimate clean lines,
repairs the cross-field contamination,
and temporally limits the changes.
Its default two-pass repair is source-specific machinery,
not a general-purpose denoiser.

!!! warning "Do not hide cadence errors"

    If the real problem is an incorrect field order,
    a bad field match,
    orphaned fields,
    or blended fields,
    repair the field workflow instead.
    Filtering a cadence error can make later IVTC less reliable.

## Side effects to check

- Vertical detail changed by the bobbed estimate
- Genuine horizontal texture mistaken for field noise
- Remaining artifacts on pattern breaks
- Motion differences between repaired and unrepaired sections
- Incorrect behavior caused by the wrong `tff` value

## Related pages

- [Field-Based Video](../situational/fieldbased.md)
- [Aliasing](aliasing.md)
- [Noise and grain](noise-and-grain.md)
- [Order of filtering operations](../general/order.md)
