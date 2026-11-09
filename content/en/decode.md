---
title: "decode"
weight: 15
---

# decode {#decode}

`github.com/go-avkit/decode` (v0.3.0) decodes **H.264** and **HEVC / H.265**
(Main and Main 10, 4:2:0) to planar YUV, in pure Go with no cgo. H.264 is
decoded by [`oops1/go.264`](https://github.com/oops1/go.264) (BSD-2-Clause),
pinned and forced onto its software path; HEVC by
[`gen2brain/h265`](https://github.com/gen2brain/h265) v0.2.4 (MIT). `decode` is
the avkit-facing layer around them and the place where claims are measured. It
is the only module in this organisation that produces pictures.

## Using it {#using-it}

```go
r, _ := container.NewReader(file)
frames, err := decode.DecodeTrack(r, trackID, decode.Config{})
for _, f := range frames { // display order
    use(f.Y, f.Cb, f.Cr, f.YStride, f.CStride, f.Width, f.Height, f.PTS)
}
```

- **H.264 input:** an `avcC` record (`decode.NewAVCC`) or a `TrackConfig`
  (`decode.NewTrack`) with length-prefixed samples, fed with `DecodeSample` and
  ended with `Flush`; or a raw Annex B stream (`decode.New`, `DecodeAnnexB`).
- **HEVC input:** the same calls choose HEVC for a track whose sample entry is
  `hvc1` or `hev1`; `decode.NewHVCC` takes an hvcC record, `decode.NewHEVC` takes
  in-band parameter sets, and `DecodeAnnexB` a raw `.265` stream. Main 10 frames
  hold little-endian 16-bit samples.
- **Output:** frames in display order, with planes, strides, size, chroma
  format, bit depth and PTS.

### One picture {#poster-frame}

```go
id, _ := decode.FirstVideoTrack(r)
f, err := decode.PosterFrame(r, id, 12.5, decode.PosterConfig{})
```

`PosterFrame` (since decode v0.2.0) returns the picture with the greatest
presentation time at or before `t`, the very bytes `DecodeTrack` returns for it,
without decoding the whole track: decoding starts at the last sync sample at or
before the picture's sample and stops as soon as the picture is out. It takes
H.264 (`avc1`, `avc3`) and HEVC (`hvc1`, `hev1`); another codec is
`ErrUnsupported`. A negative or NaN `t` is `ErrTime`.

## What it decodes {#what-it-decodes}

| Codec and profile | State |
| --- | --- |
| H.264 Constrained Baseline; Baseline without FMO/ASO | bit-exact |
| H.264 Main, progressive (CAVLC, CABAC, B pictures, weighted prediction, multiple and long-term references, MMCO) | bit-exact |
| H.264 High, 4:2:0 8-bit, progressive | bit-exact on the streams measured; no JVT FRExt stream measured yet |
| H.264 interlaced (PAFF, MBAFF), FMO/ASO, Extended | refused with `ErrUnsupported` |
| H.264 High 10, 4:2:2, 4:4:4, lossless | refused with `ErrUnsupported` |
| HEVC Main, Main Still Picture, 4:2:0 8-bit | bit-exact against ffmpeg |
| HEVC Main 10, 4:2:0 10-bit | bit-exact against ffmpeg |
| HEVC other chroma formats, other bit depths, unequal luma and chroma depths | refused with `ErrUnsupported` |

## How it was measured {#measured}

Every picture's MD5 is compared with ffmpeg's `-f framemd5`, single thread.

- **ITU-T H.264.1 conformance suite (AVCv1):** 81 streams tried, 73 give the same
  pictures as ffmpeg. 69 do with `go.264` v1.12.0 as pinned; 4 more
  (`MR2` to `MR5_TANDBERG`) only with the upstream fix
  [oops1/go.264#2](https://github.com/oops1/go.264/pull/2), which is not yet
  released. The other 8 are not claimed: 5 interlaced and 3 using FMO/ASO.
- **JCT-VC conformance suite HEVC_v1 (ITU-T H.265.1):** 147 streams. 143 give the
  same pictures as ffmpeg. Three are decided by the suite's own MD5 because
  ffmpeg disagrees with it and `decode` agrees with it (`SAODBLK_A_MainConcept_4`,
  `SAODBLK_B_MainConcept_4`, `VPSSPSPPS_A_MainConcept_1`). One,
  `TSUNEQBD_A_MAIN10_Technicolor_2` (luma and chroma bit depths differ), is
  refused by `decode` and by ffmpeg. The bitstreams are not in the repository,
  so the conformance test runs only when pointed at them.
- **Streams and real files in the repository:** 29 H.264 and 5 HEVC streams run
  in CI against ffmpeg's MD5s; real MP4 and Matroska files (5 H.264, 7 HEVC) are
  recorded as hashes.

## Limits {#limits}

A stream is untrusted. `Config.MaxFrameMBs` (default 36864, a 4096x2304 picture)
and `MaxSampleBytes` (default 64 MiB) are checked before anything sized by the
stream is allocated. Bomb tests assert a refusal under a memory limit for both
codecs, and the parsers and sample decoders are fuzzed with a committed corpus.
`PosterFrame` has bounds of its own in `PosterConfig`: `MaxSamples` (default 600)
caps the samples decoded for one picture, past it the error is `ErrPosterLimit`,
and `MaxTrackSamples` (default 1<<20) refuses a longer sample list.

## Speed {#speed}

Not yet measured against ffmpeg, so nothing is claimed about it. It is tracked in
[go-avkit/decode#2](https://github.com/go-avkit/decode/issues/2).
