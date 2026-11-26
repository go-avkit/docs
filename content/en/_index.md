---
title: "Overview"
weight: 10
---

# go-avkit {#go-avkit}

An audio/video toolkit in pure Go (`CGO_ENABLED=0`, no ffmpeg, no subprocesses). It
reads and writes MP4, Matroska/WebM and MPEG-TS, reads the common audio formats, and
moves tracks between containers without re-encoding, subtitles, chapters and metadata
included. It cuts a file into HLS/DASH segments, optionally encrypted with a key you
supply, reads the H.264, H.265, VP8 and VP9 bitstream syntax, and decodes H.264 and HEVC
to planar YUV and FLAC, MP3, Opus, AAC and linear PCM to PCM. Only
[`decode`]({{< relref "decode.md" >}}) turns samples into pictures or sound.

Part of the **Graphics, images & media** family of the
[pure-Go ecosystem](https://go-desktop.github.io/) — 9 modules.

## What is here {#what-is-here}

This site is the organisation's reference index. The API itself is generated from the
source and served by [pkg.go.dev](https://pkg.go.dev/), always current with the
released tags.

- **[Modules]({{< relref "modules.md" >}})**: every module, with its source, its
  reference and a note on what it does.
- **[decode]({{< relref "decode.md" >}})**: the H.264, HEVC and audio decoder and
  `PosterFrame`: what it claims, what it refuses, how it was measured.
- **[History and measurements]({{< relref "history.md" >}})**: per-version changes and
  speed tables, including `h265` (open defect: about 0.2 to 0.3 of ffmpeg's speed on one
  core) and H.264.

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
