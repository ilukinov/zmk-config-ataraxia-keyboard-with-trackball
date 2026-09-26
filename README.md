# Ataraxia v2 — ZMK config

Firmware for the **Ataraxia v2**: a 42-key unibody split with a PMW3610 trackball,
running a layout adapted from my Lily58.

> **This is the active firmware branch.** `master` is the untouched fork of
> [miketronic/zmk-config](https://github.com/miketronic/zmk-config) and is not built or flashed.

---

## Flashing

The Ataraxia is unibody — **one controller, one flash**, unlike a split.

1. Plug in over USB-C (a cable that carries data, not charge-only).
2. Enter the bootloader — either:
   - press **`W` + `E` + `I` + `O`** together (see [combos](#combos)), or
   - **double-tap the reset button**.
3. A disk named `NICENANO` mounts.
4. Copy `ataraxia.uf2` onto it.
5. The board reboots itself and the disk disappears. macOS may say *"Disk Not
   Ejected Properly"* — that is expected, not an error.

Firmware comes from the `firmware` artifact on the latest green
[Actions run](../../actions), or from a [local build](#building-locally).

### Two different "resets"

These are easy to confuse and do completely different things:

| | What it does | How |
|---|---|---|
| **Bootloader** | Puts the board in UF2 mode so you can flash it. Changes nothing. | `W`+`E`+`I`+`O`, or double-tap reset |
| **Settings reset** | Wipes stored settings — Bluetooth pairings, last active layer. | Flash `settings_reset_ataraxia.uf2`, then flash the real firmware again |

Reach for the settings reset only if the board misbehaves after a keymap change
that moved layers around — stale layer state is the usual culprit.

---

## Layout

Three layers you'll actually touch. `─` means the key falls through to the layer below.

### Base

```
 TAB    Q    W    E    R    T   │   Y     U     I     O     P     -
 ESC    A    S    D    F    G   │   H     J     K     L     ;     '
 SHFT   Z    X    C    V    B   │   N     M     ,     .     /    SHFT
              RALT RCTL  ⌘/BSPC │ RAISE/SPC  LOWER/RET  RGUI
```

The bottom three rows are the Lily58's, unchanged. Thumbs condense eight keys
into six: `⌘/BSPC`, `RAISE/SPC` and `LOWER/RET` are tap/hold.

### Lower — hold the **right middle** thumb

```
  `     !    @    #    $    %   │   ^     &     *     (     )     ~
 F1    F2   F3   F4   F5   F6   │  F7    F8    F9    F10   F11   F12
 HYP-L BT0  BT1  BT2  BT3  BT4  │   _     +     {     }     |    HYP-R
              SCRL SNIPE BT_CLR │ LC(SPC)   ─     ─
```

`HYP-L` / `HYP-R` are the two hyper keys from the Lily58
(`LS(LC(LG(LALT)))` and `RS(RA(RC(RGUI)))`). `LC(SPACE)` sits on **Lower + Space**.

### Raise — hold the **right inner** thumb

```
  `     1    2    3    4    5   │   6     7     8     9     0     =
  ─     ─    ─    ─    ─    ─   │  LEFT  DOWN  UP   RIGHT  HOME  END
  ─     ─    ─    ─    ─    ─   │   [     ]     {     }     \    PSCRN
              ─    ─    ─       │   ─     ─     ─
```

The number row keeps its original Lily58 columns, so the digits are exactly
where your fingers expect them — one thumb hold away.

---

## Trackball

Moving the ball temporarily activates the **Mouse** layer, putting the buttons
under your right hand:

```
  ─     ─    ─    ─    ─    ─   │ SNIPE  LCLK  MCLK  RCLK  SCRL   ─
```

It will not fire until you have paused **150 ms** since your last keystroke
(`require-prior-idle-ms`), and it releases **350 ms** after the ball stops.
Both knobs live at the top of `config/ataraxia.keymap`.

### Layer indices are not free

Scroll, sniping and ball-as-arrows are implemented by the **PMW3610 driver**,
not by ZMK input processors, and the shield hard-codes which layer index
triggers each one:

```c
// boards/shields/ataraxia/boards/nice_nano_v2.overlay
snipe-layers  = <7>;
scroll-layers = <4 5>;
arrows        { layers = <2>; ... };
```

So the keymap's layer order is deliberate, not arbitrary:

| Index | Layer | Why |
|---:|---|---|
| 0 | Base | |
| 1 | Lower | |
| 2 | *reserved* | driver would turn the ball into arrow keys here |
| 3 | Raise | |
| 4 | Scroll | driver scrolls on this index |
| 5 | *reserved* | driver treats this as a second scroll layer |
| 6 | Mouse | auto-activated by ball movement |
| 7 | Snipe | driver slows the ball on this index |

**Renumbering these silently changes what the trackball does.**

---

## Combos

| Keys | Positions | Action |
|---|---|---|
| `W` `E` `I` `O` | `2 3 8 9` | Enter UF2 bootloader |

Key positions, for reference:

```
  0   1   2   3   4   5   │   6   7   8   9  10  11
 12  13  14  15  16  17   │  18  19  20  21  22  23
 24  25  26  27  28  29   │  30  31  32  33  34  35
          36  37  38      │  39  40  41
```

---

## Building locally

CI builds on every push to `config/**` or `build.yaml`. To build on your own
machine with Docker instead:

```bash
docker volume create zmk-ataraxia-ws
docker run --rm \
  -v zmk-ataraxia-ws:/ws \
  -v "$PWD/config":/src/config:ro \
  -v "$PWD/out":/out \
  -w /ws zmkfirmware/zmk-build-arm:stable bash -c '
    cp -R /src/config /ws/config
    cd /ws && west init -l config && west update --narrow -o=--depth=1 && west zephyr-export
    west build -s zmk/app -b nice_nano_v2 -S studio-rpc-usb-uart -d build/ataraxia -p \
      -- -DSHIELD="ataraxia rgbled_adapter" -DZMK_CONFIG=/ws/config -DCONFIG_ZMK_STUDIO=y
    cp /ws/build/ataraxia/zephyr/zmk.uf2 /out/ataraxia.uf2
  '
```

The volume keeps the west workspace off the bind mount — cloning Zephyr's file
tree through virtiofs is painfully slow otherwise. Re-running reuses it.

## Editing the keymap

`config/info.json` defines the 42-key physical layout for
[keymap-editor](https://nickcoutsos.github.io/keymap-editor), so the keymap can
be edited visually rather than by hand.
