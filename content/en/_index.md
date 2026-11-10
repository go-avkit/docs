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
for HEVC Main and Main 10 (4:2:0), within the limits that page states; nothing
here decodes sound except `decode`'s FLAC, MP3 and Opus (not Vorbis, MP2 or AAC).

Part of the **Graphics, images & media** family of the
[pure-Go ecosystem](https://go-desktop.github.io/) — 8 modules,
all `CGO_ENABLED=0`.

## What is here {#what-is-here}

This site is the organisation's reference index: what each module is, and where its
API documentation lives. The API itself is generated from the source and served by
[pkg.go.dev](https://pkg.go.dev/), which is always current with the released tags —
duplicating it here would only create a second copy to go stale.

- **[Modules]({{< relref "modules.md" >}})** — every module in go-avkit, with its source and its reference.
- **[decode]({{< relref "decode.md" >}})** — the H.264 and HEVC decoder and `PosterFrame`: what it claims, what it refuses, how it was measured.

## What every module here is held to {#what-every-module-here-is-held-to}

- `CGO_ENABLED=0`: no cgo, and no shelling out to a command-line tool in place of a
  library.
- Built and tested on amd64, arm64, riscv64, loong64, ppc64le and s390x — the last
  being big-endian, which keeps every on-disk and on-wire encoding honest.
- 100% statement coverage as a CI gate, error branches included.
- BSD-3-Clause.

The standard is described in full on the
[ecosystem map](https://go-desktop.github.io/docs/latest/standards/).
