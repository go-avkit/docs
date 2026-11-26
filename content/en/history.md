---
title: "History and measurements"
weight: 30
---

# History and measurements {#history}

The per-version history and the speed measurements of the modules, moved here from
the [overview]({{< relref "_index.md" >}}) so that page can stay short. The detail of
the decoder is also on the [decode]({{< relref "decode.md" >}}) page.

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
for HEVC in 4:0:0, 4:2:0, 4:2:2 and 4:4:4 at 8 to 16 bits (4:2:0 at 8 and 10 bits before `decode` v0.15.0), within the limits that page states. `h265`
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
uses it. v0.23.0 decodes the 7 JCT-VC RExt streams that use
`cabac_bypass_alignment_enabled_flag` (all 31 of their picture hashes match) and the
base layer of the 9 MV-HEVC conformance streams (md5 of the shipped base-layer YUV on
371 pictures); speed is not re-measured. `decode` v0.13.0 (h265 v0.23.0) decodes 1 of the 7 streams and 6 are refused (4:4:4 with 16-bit samples, a format a `Frame` cannot carry), and decodes the MV-HEVC base layer (md5 equal to the shipped `_0.yuv`).
v0.24.0 adds an opt-in reuse of pictures: with `Options.RecyclePictures` set, a caller
that is done with a picture calls `(*DecodedPicture).Release()` and the decoder reuses
its planes once the decoded picture buffer no longer references it. The default is
unchanged (every picture is its own allocation, never reused); a picture that is not
released is never reused, and releasing twice, or with the option off, does nothing.
`h265.Decoder` alone, one pinned Zen 4 core, option off to on: alien Main 104.2 to
113.9 fps (+9 %), fade Main 93.2 to 101.2 (+8 %), alien Main 10 89.2 to 100.3 (+12 %),
alien8 121.1 to 124.9 (+3 %), with 3 to 5 times fewer page faults per frame. The
output is unchanged on all 196 JCT-VC conformance streams.
**Behaviour change in `decode` v0.15.0:** HEVC in 4:0:0, 4:2:2 and 4:4:4, at 8 to 16 bits
with luma and chroma depths that may differ, now decodes by default (until v0.14.0 only
4:2:0 at 8 or 10 bits did). A caller must read `Frame.Chroma`, `BitDepth` and `BitDepthC`
(`SampleBytes(plane)` and `ChromaSize()` help); the 47 JCT-VC range-extension streams the
test can run match the suite's md5. Still refused: separate colour planes, and a frame of
more than 2³¹−1 bytes (`ErrOverLimit`). `decode` v0.15.0 also uses h265 v0.24.0 (picture
reuse, 5 to 8 % faster through `Frames` and `DecodeTrack` on Zen 4) and h264 v0.12.0 (2.2
to 2.7 % faster); decode#108, a 32-bit overflow, is fixed. `decode` v0.15.1 uses h264
v0.13.0: H.264 decoding is 3.3 to 3.6 % faster on Zen 4 (alien 2.266 to 2.185 s, fade
0.881 to 0.852 s, from the h264 notes). `decode` v0.15.2 uses h264 v0.14.0: 5.9 % faster
on alien (2.207 to 2.076 s) and 3.5 % on fade (0.849 to 0.819 s) on Zen 4; ffmpeg is still
2.07 times faster on alien and 2.28 times on fade. `decode` v0.15.3 uses h264 v0.15.0
(CABAC residual decoding in one assembly kernel call, output identical to v0.14.0): alien
2.051 to 2.025 s (1.3 %) on Zen 4, ffmpeg about 2.02 times faster on alien (2.025 s
against 1.004 s). `decode` v0.16.1 runs the SBR filter banks of HE-AAC v2 in float32:
699 to 670 ms (-4.2 %) on a short clip, one Zen 4 core, 1.59 to 1.52 times ffmpeg. `decode` v0.17.1 drives the parametric-stereo all-pass plan
by tick (go-simd/floats v0.41.0): 2053 to 2044 us (-0.45 %, within a 0.3 % noise) on that
benchmark. Measured on 2026-11-25 (Zen 4, one pinned core, load about 1, 9 interleaved rounds, ffmpeg 6.1.1 with one thread): HE-AAC v2 decodes at 1.42 times ffmpeg's user time (was 1.74 when decode#77 opened); ffmpeg's real time also holds 209 ms of system time that our in-process benchmark does not pay, so user time is the comparable figure.
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
v0.9.0. The picture md5 equals ffmpeg's. Correctness against v0.9.0 on the 196 JCT-VC streams: 148 identical, 0 lost, 4 more decode (v0.10.0); against v0.12.0, `decode` v0.13.0 gives 152 identical, 0 lost, 1 more
(`WPP_AND_TILE_AND_CABAC_BYPASS_ALIGN_1_HIGH_TP_444_14BIT_RExt_Apple_2`) and 43 refused by both.

**H.264 speed:** `decode` v0.12.0 moves to
[`h264`](https://github.com/go-avkit/h264) v0.11.0 (API unchanged), which runs more of
go-videodsp/kernels (AVX2 on amd64). Through `DecodeTrack`, one core, 1080p, Apple M4
(indicative, load 9 to 11: ratios hold, times do not): alien 5.7 to 5.9 s to 2.3 s, fade
(explicit weights) 2.0 s to 0.8 s, 2.5 times faster; ffmpeg is still 2.3 to 2.8 times
faster, an open defect. Zen 4 is not re-measured.

