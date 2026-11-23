# tailscale-slim-mipsle

A size-reduced build of [Tailscale](https://github.com/tailscale/tailscale) for small
MIPS little-endian routers (for example the MediaTek MT7628, 16 MB flash, 128 MB RAM).

The official mipsle release ships two files of about 76 MB together. This build is one
combined file (`tailscaled` with the `tailscale` CLI built in) of about 15.7 MB, and 5.2 MB
compressed. It is small enough to download into RAM at boot on a router that cannot hold it
in flash.

This is an unofficial build. It is not made or supported by Tailscale Inc.

## Files (in the release, not in the repo tree)

| File | SHA-256 |
|---|---|
| `tailscaled-1.104.7-mipsle-slim.gz` | `f10e4e25ce4e51d2a8a2d60b0ee2d647eb6facfdcdbf45213b88cad8f0017c64` |
| the same file unpacked (`gunzip`) | `9c7c2bc530682503fd05c307fc96cc9dcc1bc9a59ff1f389df4abda96df6142c` |
| `tailscaled-1.104.7-mipsle-slim2.gz` (release `v1.104.7-slim2`) | `1a2a6a0e188aa537b7ea57f3cdcbcceeb7207a4b39b15db58023130e0bbf58bc` |
| slim2 unpacked (`gunzip`) | `fa644b3108bf3128ff6f7f1cec5da5ec8679061702369859b4c5fb423cdf56b7` |

`slim2` is the same build with one feature added back: using an exit node (`useexitnode`), 15.79 MB unpacked.

A device should pin the unpacked SHA-256 and reject any download that does not match.
Then the download host does not need to be trusted.

## Use

The one binary acts by the name it is called by:

```sh
ln -s tailscaled.box tailscaled
ln -s tailscaled.box tailscale
./tailscaled --state=/etc/tailscale/tailscaled.state --tun=tailscale0 --no-logs-no-support
./tailscale up
```

## What is left out

Built with Tailscale's own feature tags: everything optional is removed except kernel
routing (`osrouter`), NAT traversal (`nattraversal`), the notification bus (`ipnbus`,
needed for `tailscale up` to print the sign-in link) and the embedded CLI (`cli`).
So there is no Tailscale SSH, no Taildrop, no exit-node serving, no web client, no
MagicDNS server, no auto-update, and no log upload.

## How it was built

- Source: tag `v1.104.7`, commit `cbc74fe48ef1b1649f69a4d9a602319427e25102`
- Go 1.27.4 (as required by the source's `go.mod`), `GOTOOLCHAIN=local`

```sh
git clone --depth 1 --branch v1.104.7 https://github.com/tailscale/tailscale.git
cd tailscale
# for slim2, add ,useexitnode to the --add list
TAGS="$(go run ./cmd/featuretags --min --add=osrouter,nattraversal,ipnbus,cli)" \
  GOOS=linux GOARCH=mipsle GOMIPS=softfloat \
  ./build_dist.sh --strip -o tailscaled.box ./cmd/tailscaled
gzip -9 -c tailscaled.box > tailscaled-1.104.7-mipsle-slim.gz
```

## Licence

Tailscale is licensed under the BSD 3-Clause License. Its licence is in [LICENSE](LICENSE).
