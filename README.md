# zmk-config (fork)

> ### 👉 The firmware I actually build and flash lives on the **[`ataraxia_v2`](../../tree/ataraxia_v2)** branch.
>
> Go there for the keymap, the layout diagrams, and flashing instructions.

---

## What this branch is

`master` is the untouched fork of
[miketronic/zmk-config](https://github.com/miketronic/zmk-config) — upstream's
shared config covering all of their keyboards (`aeliux_4`, `avagrace`,
`capsule3`, `desolation`, `ledz`, `10x2`, `ataraxia`). Nothing here is mine, and
**nothing here is flashed to my keyboard.**

It is kept only to pull upstream changes.

## Why `ataraxia_v2` is a separate branch

That branch is not just a keymap change — it targets different hardware and a
different ZMK baseline:

| | `master` (upstream) | `ataraxia_v2` (mine) |
|---|---|---|
| Shield | `ataraxia nice_view_adapter nice_view_gem` | `ataraxia rgbled_adapter` |
| ZMK | `main` | pinned to `v0.3` |
| PMW3610 driver | `efogdev/zmk-pmw3610-driver` | `miketronic/zmk-pmw3610-driver-02` |
| Snippet | `zmk-usb-logging` | `studio-rpc-usb-uart` |
| Keymap | upstream's | my Lily58 layout, adapted to 42 keys |

Because the ZMK revision and the trackball driver differ, the two branches are
not interchangeable — building `master` would produce firmware for a
display-equipped board, not mine.

## Branches

| Branch | Purpose |
|---|---|
| **`ataraxia_v2`** | **My firmware.** Built by CI, flashed to my board. |
| `master` | Upstream fork, for syncing only. |
| others | Upstream's other boards. |
