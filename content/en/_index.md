---
title: "Overview"
weight: 10
---

# go-avkit {#go-avkit}

An audio/video toolkit. It reads and writes MP4/ISO-BMFF, Matroska/WebM and
MPEG-TS, reads ADTS, MP3, FLAC, Ogg, WAV and AVI, and moves tracks between them
without re-encoding, with their subtitles (WebVTT, SRT, TTML, 3GPP tx3g, and PGS
in Matroska), edit lists and metadata. It extracts one audio track as an
audio-only file (ADTS, MP3/MP2, FLAC, Ogg, WAV). It carries Apple Lossless
(ALAC) and Dolby TrueHD in MP4 and Matroska (not in MPEG-TS), and reads, writes
and carries chapters (MP4 `chpl` and QuickTime chapter track, Matroska
`Chapters`), moved by `Cut` and `Concat`. `container.Segment` (avkit v0.14.0)
cuts a file into the initialisation and media segments of an HLS or DASH
presentation — CMAF/fMP4 for video and audio, WebVTT documents for subtitles —
aligned across tracks at the video keyframes; the playlists and the MPD are
written by [go-streamkit](https://go-streamkit.github.io/)'s `hls` and `dash`
packages (streamkit v0.2.0), and the module `github.com/go-avkit/avkit/media`
(v0.2.0) joins the two: `media.HLS`, `media.DASH` and `media.WriteDir`. VOD
only, one video track at most: no live, no encoding ladder. Since avkit v0.15.0,
`container.EncryptSegments` encrypts those segments with Common Encryption —
`cenc` (AES-CTR) or `cbcs` (AES-CBC, 1:9 pattern for video) — from a key the
caller supplies, keeping NAL and slice headers clear, and `container.Encrypt`,
`Decrypt` and `DecryptFile` work on any fragmented MP4 (ffmpeg decrypts the
output). With `media.Keys(...)` (media v0.2.0), `media.HLS`, `media.DASH` and
`media.WriteDir` state how a player gets the key: `EXT-X-KEY` and
`EXT-X-SESSION-KEY` in the HLS playlists, `ContentProtection` in the DASH
manifest. There is no licence logic and no key server, and the content key
itself never enters `media`. It
also reads the H.264, H.265, VP8 and VP9 bitstream syntax. Only
[`decode`]({{< relref "decode.md" >}}) turns samples into pictures, for H.264 and
for HEVC Main and Main 10 (4:2:0), within the limits that page states. `h265`
carries a decoder of our own since v0.14.0 and reconstructs inter prediction
since v0.15.0: 7147 hashed conformance pictures verified (6638 inter), 0
mismatched; the 3091 pictures of the 59 hashless streams are unverified
(h265#48). Since v0.16.0 it has a public `Decoder` API (Decode and Flush, pictures in output
order, typed errors, limits on picture size and memory); v0.14.0 and v0.15.0 could
panic on a hostile SPS, fixed in v0.16.0. v0.20.0 is faster than the
previous third-party decoder on every stream measured on x86 (1.03 to 1.09 times on
8-bit, 3.07 on Main 10) and still 0.19 to 0.30 of ffmpeg's speed, an open defect;
v0.21.0 adds `Unit.Tag`, carried to `Output.Tag`. v0.22.0 decodes 8-bit pictures
natively into bytes, with the output unchanged (md5 equal to ffmpeg's): on one pinned
Zen 4 core, against v0.21.0, alien Main 91.1 to 107.6 fps (+18 %), fade Main 82.2 to
95.3 (+16 %), alien8 103.9 to 125.7 (+21 %), Main 10 unchanged; on an Apple M4
(indicative) +1 to +8 %. **Behaviour change in v0.22.0:** `Plane.Pix` is renamed
`Pix16`, with no alias, and an 8-bit picture carries `Pix8 []uint8` instead (exactly
one of the two is non-nil; `Is8`, `At` and `Len` read either). `decode` v0.11.0
uses it.
**Behaviour change in v0.19.0:** the planes of a picture from `Decoder.Output` are
read-only views, not copies, and `Stride` can exceed `Width`; a caller that modifies
samples must call `Clone()` first. Since `decode` v0.10.0, `decode` decodes HEVC with
`h265` (gen2brain/h265 is gone); v0.10.0 was 7 to 12 % slower than v0.9.0 on x86
8-bit streams, and since v0.11.0 no stream is slower than v0.9.0 (Main 10 is 2.5
times faster). Nothing
here decodes sound except `decode`'s FLAC, MP3, Opus and, since decode v0.6.0,
AAC-LC and HE-AAC v1 and, since v0.7.0, HE-AAC v2 (full profile since v0.8.0; not Vorbis or MP2).

`h265` speed (v0.20.0, from its
[README](https://github.com/go-avkit/h265#speed)): frames per second by CPU time,
one core (AMD Ryzen 9 7950X, `taskset -c 5`, ABBA of five rounds each, load 0.44 to
1.21, 2026-11-16). "Third-party" is gen2brain/h265, the decoder `decode` v0.9.0 used,
measured through `DecodeTrack`; ffmpeg is 6.1.1 with `-threads 1`. No parity is
claimed: it is far slower than ffmpeg, an open defect.

| stream (1080p) | v0.19.0 | v0.20.0 | third-party | ffmpeg | v0.20.0 / third-party | v0.20.0 / ffmpeg |
| --- | --- | --- | --- | --- | --- | --- |
| alien, Main | 75.9 | 91.2 | 87.2 | 392.6 | 1.05 | 0.23 |
| fade, Main | 68.7 | 82.6 | 79.9 | 428.2 | 1.03 | 0.19 |
| alien, Main 10 | 79.1 | 92.0 | 30.0 | 303.2 | 3.07 | 0.30 |
| alien8 | 85.6 | 104.0 | 95.2 | 431.0 | 1.09 | 0.24 |

`decode` v0.10.0 speed, `DecodeTrack` on the MP4 of four 1080p streams, frames per
second of CPU time, median of 7 rounds, order reversed every other round, against
ffmpeg 6.1.1 `-threads 1` (x86) and 9.0.2 (M4); "before" is decode v0.9.0 over
gen2brain/h265. Source: the [decode README](https://github.com/go-avkit/decode#readme).
The M4 rows are indicative (load 7.8 to 9.8).

| 1080p, one core | decode (h265) | before (gen2brain) | ffmpeg | decode / ffmpeg | decode / before |
| --- | --- | --- | --- | --- | --- |
| x86, Ryzen 9 7950X, `taskset -c 5`, load 0.4 to 1.1: Main (240 frames) | 77.2 | 87.1 | 390.2 | 0.20 | 0.89 |
| Main fade, weighted prediction (120) | 70.7 | 79.9 | 431.7 | 0.16 | 0.88 |
| Main 10 (240) | 75.2 | 30.0 | 295.6 | 0.25 | 2.51 |
| Main, second stream (240) | 88.9 | 95.5 | 430.9 | 0.21 | 0.93 |
| M4 Max, indicative: Main (240) | 92.8 | 72.8 | 431.7 | 0.21 | 1.27 |
| Main fade (120) | 99.2 | 67.6 | 485.8 | 0.20 | 1.47 |
| Main 10 (240) | 87.6 | 31.4 | 125.1 | 0.70 | 2.79 |
| Main, second stream (240) | 108.3 | 80.5 | 488.8 | 0.22 | 1.34 |

The table above is v0.10.0, where the x86 8-bit streams were 7 to 12 percent slower
than v0.9.0 because every 16-bit h265 frame was narrowed on the way out (decode#101,
closed). **`decode` v0.11.0** moves to h265 v0.22.0, which decodes 8-bit pictures into
bytes: through `DecodeTrack` on Zen 4 (pinned core), against v0.10.0 and against v0.9.0,
Main 1.23 and 1.10, fade 1.20 and 1.06, second Main 1.22 and 1.14, Main 10 1.00 and
2.51; on an Apple M4 (indicative) 1.00 to 1.10 against v0.10.0 and 1.43 to 2.84 against
v0.9.0. The picture md5 equals ffmpeg's. Correctness against v0.9.0 on the 196 JCT-VC streams: 148 identical, 0 lost, 4 more decode; the 44 both
refuse are 37 formats a `Frame` cannot carry and 7 using `cabac_bypass_alignment`.

**H.264 speed:** `decode` v0.12.0 moves to
[`h264`](https://github.com/go-avkit/h264) v0.11.0 (API unchanged), which runs more of
go-videodsp/kernels (AVX2 on amd64). Through `DecodeTrack`, one core, 1080p, Apple M4
(indicative, load 9 to 11: ratios hold, times do not): alien 5.7 to 5.9 s to 2.3 s, fade
(explicit weights) 2.0 s to 0.8 s, 2.5 times faster; ffmpeg is still 2.3 to 2.8 times
faster, an open defect. Zen 4 is not re-measured.

Part of the **Graphics, images & media** family of the
[pure-Go ecosystem](https://go-desktop.github.io/) — 9 modules,
all `CGO_ENABLED=0`.

## What is here {#what-is-here}

This site is the organisation's reference index: what each module is, and where its
API documentation lives. The API itself is generated from the source and served by
[pkg.go.dev](https://pkg.go.dev/), which is always current with the released tags —
duplicating it here would only create a second copy to go stale.

- **[Modules]({{< relref "modules.md" >}})** — every module in go-avkit, with its source and its reference.
- **[worker](https://github.com/go-avkit/worker)** (v0.1.2) — a `js/wasm` Web Worker that answers **poster** (a JPEG of the frame at a time) and **probe** (container, codec, size, duration) requests for a page that must not link a container reader or a decoder.
- **[decode]({{< relref "decode.md" >}})** — the H.264 and HEVC decoder and `PosterFrame`: what it claims, what it refuses, how it was measured.

## Hostile files {#hostile-files}

A media file is untrusted input. Since avkit v0.17.0, `NewReader`, `Demux` and
`OpenFile` walk a Matroska file's block headers before anything is allocated and
refuse it with `ErrReaderLimit` when a block states more laced frames than it has
bytes for, or a track or the file would hold more frames than the bounds allow
(defaults 1<<24 per track and 1<<25 per file, sized so that no file in a library
of 30 real Matroska files is refused). `ReaderLimits`, passed to
`NewReaderWithLimits`, `DemuxWithLimits` or `OpenFileWithLimits`, lowers them.
The worst case at the defaults is documented in avkit's README at about 380 bytes
allocated per input byte of one-byte laced frames; a caller with less memory
passes lower limits. The MP4 box bound is still open
([go-avkit/avkit#113](https://github.com/go-avkit/avkit/issues/113)).

## What every module here is held to {#what-every-module-here-is-held-to}

- `CGO_ENABLED=0`: no cgo, and no shelling out to a command-line tool in place of a
  library.
- Built and tested on amd64, arm64, riscv64, loong64, ppc64le and s390x — the last
  being big-endian, which keeps every on-disk and on-wire encoding honest.
- 100% statement coverage as a CI gate, error branches included.
- BSD-3-Clause.

The standard is described in full on the
[ecosystem map](https://go-desktop.github.io/docs/latest/standards/).
