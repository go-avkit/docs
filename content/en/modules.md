---
title: "Modules"
weight: 20
---

# Modules {#modules}

The 9 modules of **go-avkit**. Each links to its source and to
its generated API reference.

| Module | Reference |
| --- | --- |
| [`avkit`](https://github.com/go-avkit/avkit) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/avkit) |
| [`bitstream`](https://github.com/go-avkit/bitstream) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/bitstream) |
| [`boolcoder`](https://github.com/go-avkit/boolcoder) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/boolcoder) |
| [`decode`](https://github.com/go-avkit/decode) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/decode) |
| [`h264`](https://github.com/go-avkit/h264) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/h264) |
| [`h265`](https://github.com/go-avkit/h265) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/h265) |
| [`vp8`](https://github.com/go-avkit/vp8) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/vp8) |
| [`vp9`](https://github.com/go-avkit/vp9) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/vp9) |
| [`worker`](https://github.com/go-avkit/worker) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/worker) |

Descriptions live on the repositories themselves rather than being copied here, so
there is one place to correct when one changes.

## Module notes {#module-notes}

The landing page gives each module one line; the detail is here.

- **`avkit`** reads and writes MP4/ISO-BMFF, Matroska/WebM and MPEG-TS; reads ADTS, MP3,
  FLAC, Ogg, WAV and AVI; extracts an audio track as ADTS, MP3, FLAC, Ogg or WAV. It
  copies, cuts, joins and concatenates tracks with no re-encoding, carrying WebVTT, SRT,
  TTML, 3GPP tx3g and PGS subtitles, ALAC and TrueHD audio, chapters, edit lists,
  encoder priming, metadata and cover art. `container.Segment` cuts CMAF/fMP4 and WebVTT
  segments for HLS and DASH (VOD; the playlists are written by go-streamkit),
  `container.EncryptSegments` encrypts them with Common Encryption (`cenc`, `cbcs`) from a
  key you supply, and `media.Keys` states how a player gets the key (`EXT-X-KEY` and
  `EXT-X-SESSION-KEY` in HLS, `ContentProtection` in DASH). `ConformHVC1` makes an HEVC
  file one macOS will show. Since v0.17.0 a Matroska file that would unlace more frames
  than a track or the file may hold is refused with `ErrReaderLimit` before anything is
  allocated; see [hostile files]({{< relref "_index.md#hostile-files" >}}).
- **`bitstream`**: the bit reader with the formats' Exp-Golomb codes, both NAL framings,
  and the unescaping a payload is meaningless without.
- **`h264`**: H.264/AVC syntax and the derivations of clause 8.2: parameter sets and
  slice headers, then picture order counts, the reference picture set, the reference
  lists and the prediction weights. Since v0.11.0 the decoder runs more of
  go-videodsp/kernels (AVX2 on amd64) with the API unchanged. Whole decode of 1080p clips,
  one core, Apple M4 Max (indicative, load 11 to 17): alien 5.75 to 6.05 s down to 2.28 to
  2.39 s, fade 2.01 to 2.13 s down to 0.82 to 0.89 s (about 2.5 times faster), framemd5
  equal to ffmpeg's. Still 2.3 to 2.9 times slower than ffmpeg, an open defect; Zen 4
  is not re-measured. Later speed steps are on the [decode]({{< relref "decode.md" >}}) page.
- **`h265`**: the NAL layer, parameter sets, slice segment header and picture
  boundaries, and since v0.14.0 an HEVC decoder of our own. All 52215 conformance slices
  walk to their last bit; 509 of 509 hashed intra pictures match their decoded picture
  hash SEI (v0.14.0); 7147 hashed pictures, 6638 of them inter, 0 mismatched (v0.15.0).
  Speed and the per-version changes are on the [home page]({{< relref "_index.md" >}}).
- **`boolcoder`**: the boolean arithmetic decoder VP8 and VP9 read their compressed
  partitions with.
- **`vp8`**: the VP8 frame tag and key-frame header, the parts of a frame that are plain
  bytes and bit fields.
- **`vp9`**: the uncompressed frame header, and the superframe index that packs several
  frames into one sample.
- **`worker`** (v0.1.2): a `js/wasm` Web Worker over the container reader and the H.264
  decoder. **poster** returns a JPEG of the frame at a time, **probe** returns container,
  codec, size and duration. A request can only lower the worker's limits, and a hostile
  file is refused rather than decoded.

This table is kept by hand, and it had fallen six modules behind — it listed
`avkit` alone while the organisation held seven. To check it, ask the
organisation which of its repositories are Go modules:

```sh
for r in $(gh repo list go-avkit --limit 60 --json name --jq '.[].name'); do
  gh api repos/go-avkit/$r/contents/go.mod --jq '.name' >/dev/null 2>&1 && echo "$r"
done | sort
```
