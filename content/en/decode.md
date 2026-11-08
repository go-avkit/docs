---
title: "decode"
weight: 15
---

# decode {#decode}

`github.com/go-avkit/decode` (v0.1.0) decodes **H.264** to planar YUV, in pure Go
with no cgo. The decoding itself is
[`oops1/go.264`](https://github.com/oops1/go.264) (BSD-2-Clause), pinned and
forced onto its software path; `decode` is the avkit-facing layer around it and
the place where claims are measured. It is the only module in this organisation
that produces pictures, and H.264 is its only codec.

## Using it {#using-it}

```go
r, _ := container.NewReader(file)
frames, err := decode.DecodeTrack(r, trackID, decode.Config{})
for _, f := range frames { // display order
    use(f.Y, f.Cb, f.Cr, f.YStride, f.CStride, f.Width, f.Height, f.PTS)
}
```

- **Input:** an `avcC` record (`decode.NewAVCC`) or a `TrackConfig`
  (`decode.NewTrack`) with length-prefixed samples, fed with `DecodeSample` and
  ended with `Flush`; or a raw Annex B stream (`decode.New`, `DecodeAnnexB`).
- **Output:** frames in display order, with planes, strides, size, chroma
  format, bit depth and PTS.

## What it decodes {#what-it-decodes}

| Profile | State |
| --- | --- |
| Constrained Baseline; Baseline without FMO/ASO | bit-exact |
| Main, progressive (CAVLC, CABAC, B pictures, weighted prediction, multiple and long-term references, MMCO) | bit-exact |
| High, 4:2:0 8-bit, progressive | bit-exact on the streams measured; no JVT FRExt stream measured yet |
| Interlaced (PAFF, MBAFF), FMO/ASO, Extended | refused with `ErrUnsupported` |
| High 10, 4:2:2, 4:4:4, lossless | refused with `ErrUnsupported` |

## How it was measured {#measured}

Every picture's MD5 is compared with `ffmpeg -f framemd5 -pix_fmt yuv420p`,
single thread.

- **ITU-T H.264.1 conformance suite (AVCv1):** 81 streams tried, 73 give the same
  pictures as ffmpeg. 69 do with `go.264` v1.12.0 as pinned; 4 more
  (`MR2` to `MR5_TANDBERG`) only with the upstream fix
  [oops1/go.264#2](https://github.com/oops1/go.264/pull/2), which is not yet
  released. The other 8 are not claimed: 5 interlaced and 3 using FMO/ASO.
- **29 streams** in the repository run in CI against ffmpeg's MD5s.
- **5 real files** (MP4 and Matroska, 720x300 to 1920x1080, x264), 900 pictures
  each, recorded as hashes.

## Limits {#limits}

A stream is untrusted. `Config.MaxFrameMBs` (default 36864, a 4096x2304 picture)
and `MaxSampleBytes` (default 64 MiB) are checked before anything sized by the
stream is allocated. A bomb test asserts a refusal under a memory limit, and the
`avcC` parser and both sample decoders are fuzzed with a committed corpus.

## Speed {#speed}

Not yet measured against ffmpeg, so nothing is claimed about it. It is tracked in
[go-avkit/decode#2](https://github.com/go-avkit/decode/issues/2).
