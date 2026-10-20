---
title: "Overview"
weight: 10
---

# go-avkit {#go-avkit}

An audio/video toolkit. It reads and writes MP4/ISO-BMFF, Matroska/WebM and
MPEG-TS, and moves tracks between them without re-encoding. It also reads the
H.264, H.265, VP8 and VP9 bitstream syntax. It is not a decoder: nothing here
turns samples into pixels or sound.

Part of the **Graphics, images & media** family of the
[pure-Go ecosystem](https://go-desktop.github.io/) — 7 modules,
all `CGO_ENABLED=0`.

## What is here {#what-is-here}

This site is the organisation's reference index: what each module is, and where its
API documentation lives. The API itself is generated from the source and served by
[pkg.go.dev](https://pkg.go.dev/), which is always current with the released tags —
duplicating it here would only create a second copy to go stale.

- **[Modules]({{< relref "modules.md" >}})** — every module in go-avkit, with its source and its reference.

## What every module here is held to {#what-every-module-here-is-held-to}

- `CGO_ENABLED=0`: no cgo, and no shelling out to a command-line tool in place of a
  library.
- Built and tested on amd64, arm64, riscv64, loong64, ppc64le and s390x — the last
  being big-endian, which keeps every on-disk and on-wire encoding honest.
- 100% statement coverage as a CI gate, error branches included.
- BSD-3-Clause.

The standard is described in full on the
[ecosystem map](https://go-desktop.github.io/docs/latest/standards/).
