# Modules

The 7 modules of **go-avkit**. Each links to its source and to
its generated API reference.

| Module | Reference |
| --- | --- |
| [`avkit`](https://github.com/go-avkit/avkit) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/avkit) |
| [`bitstream`](https://github.com/go-avkit/bitstream) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/bitstream) |
| [`boolcoder`](https://github.com/go-avkit/boolcoder) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/boolcoder) |
| [`h264`](https://github.com/go-avkit/h264) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/h264) |
| [`h265`](https://github.com/go-avkit/h265) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/h265) |
| [`vp8`](https://github.com/go-avkit/vp8) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/vp8) |
| [`vp9`](https://github.com/go-avkit/vp9) | [pkg.go.dev](https://pkg.go.dev/github.com/go-avkit/vp9) |

Descriptions live on the repositories themselves rather than being copied here, so
there is one place to correct when one changes.

This table is kept by hand, and it had fallen six modules behind — it listed
`avkit` alone while the organisation held seven. To check it, ask the
organisation which of its repositories are Go modules:

```sh
for r in $(gh repo list go-avkit --limit 60 --json name --jq '.[].name'); do
  gh api repos/go-avkit/$r/contents/go.mod --jq '.name' >/dev/null 2>&1 && echo "$r"
done | sort
```
