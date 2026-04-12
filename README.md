# KarmaPad

> "If humans made it, I can do it too." — 1801 Collective

6-key wireless macropad with 1.3" SH1106 OLED, WS2812B ARGB underglow, and dual
EC11 rotary encoders. Built on ZMK firmware for the Pro Micro NRF52840.

## Features

- **6 keys** mapped to F13–F18 on the base layer (invisible to normal apps, perfect for OBS/Streamlabs hotkeys)
- **2 rotary encoders** — left for mic volume/mute, right for desktop volume/mute
- **1.3" SH1106 OLED** showing layer name, BT profile, battery %
- **WS2812B underglow** with 7+ built-in effects (solid/breathe/spectrum/swirl)
- **3 layers**: STREAM (default), MEDIA, SETTINGS
- **BLE + USB** — pair to 5 devices, swap with a keypress
- **15 min idle sleep** for LiPo longevity

---

## Wiring (Pro Micro NRF52840)

### Keys — direct wire to GND
| Key | Pin  | Function                              |
|-----|------|---------------------------------------|
| K1  | D4   | F13                                   |
| K2  | D5   | F14                                   |
| K3  | D6   | F15                                   |
| K4  | D7   | F16                                   |
| K5  | D8   | F17                                   |
| K6  | D9   | F18                                   |
| K7  | D10  | Left knob press — tap mute / hold MEDIA    |
| K8  | D14  | Right knob press — tap mute / hold SETTINGS |

### Encoders
| Signal        | Pin |
|---------------|-----|
| Left A        | D21 |
| Left B        | D20 |
| Left Switch   | D10 (wired as K7) |
| Right A       | D19 |
| Right B       | D18 |
| Right Switch  | D14 (wired as K8) |

### OLED (I2C)
| OLED | Pro Micro |
|------|-----------|
| SDA  | D2        |
| SCL  | D3        |
| VCC  | 3.3V      |
| GND  | GND       |

### WS2812B ARGB
| Strip | Pro Micro |
|-------|-----------|
| DIN   | D16 (MOSI)|
| +5V   | RAW (USB) or boost from LiPo |
| GND   | GND       |

> ⚠️ **Battery warning**: WS2812B is rated 5V. On LiPo (3.7–4.2V) they *usually*
> still work at reduced brightness, but for reliability swap to **SK6812 MINI-E**
> (3.3V tolerant, same protocol, same library). Keep `RGB_BRT_START` low (40)
> and toggle RGB off via the SETTINGS layer when running on battery.

---

## Streamlabs setup

ZMK is an HID keyboard — it can't call Streamlabs APIs directly. Instead the knobs
send unused modifier+Fkey combos that you bind as hotkeys inside Streamlabs:

| Action            | Sent keycode                    |
|-------------------|---------------------------------|
| Mic vol up        | `Ctrl+Shift+Alt+F23`            |
| Mic vol down      | `Ctrl+Shift+Alt+F24`            |
| Desktop vol up    | `C_VOL_UP` (OS native)          |
| Desktop vol down  | `C_VOL_DN` (OS native)          |

In **Streamlabs → Settings → Hotkeys**, bind "Mute Mic" and "Mute Desktop Audio"
to whatever unused combos you want, then edit `karmapad.keymap` to match.

Left knob rotate = mic volume, right knob rotate = system volume. Press the
encoder shafts to mute (wire encoder switches as keys K7/K8 if you want this —
current config has 6 keys only).

---

## Layers

1. **STREAM** (default) — F13–F18 + mic/desktop volume
2. **MEDIA** — prev/play/next/mute/stop/eject + volume + brightness
3. **SETTINGS** — BT profile 0/1/2, BT clear, RGB toggle, RGB effect cycle. Knobs control RGB brightness + hue.

To switch layers: add a `&mo 1` or `&to 2` binding to any key. Currently all 18
slots are used for hotkeys — add a tap-dance or hold-tap on one key if you want
layer switching without losing a hotkey.

---

## Build

This is a **ZMK user config**, not a VSCode/PlatformIO project. You have two paths:

### Path A — GitHub Actions (recommended)
1. Push this repo to GitHub
2. Actions auto-builds `karmapad.uf2`
3. Download from the Actions artifacts tab
4. Double-tap reset on the Pro Micro → it mounts as USB drive → drag the `.uf2` over

### Path B — Local build in VSCode
1. Install the [ZMK toolchain](https://zmk.dev/docs/development/setup)
2. Open this folder in VSCode with the Dev Containers extension
3. `west build -s zmk/app -b karmapad -- -DZMK_CONFIG="$(pwd)/config"`
4. Flash `build/zephyr/zmk.uf2`

---

## File layout

```
karmapad/
├── build.yaml                 # which boards to build
├── .github/workflows/build.yml
└── config/
    ├── west.yml               # ZMK manifest
    ├── karmapad.keymap        # ← edit this for key bindings
    └── boards/arm/karmapad/
        ├── karmapad.dts       # hardware definition (pins, OLED, WS2812)
        ├── karmapad_defconfig # feature flags
        ├── Kconfig.board
        └── Kconfig.defconfig
```

---

## Known gotchas

- **SH1106 vs SSD1306**: The 1.3" white OLED is almost always SH1106. The
  `segment-offset = <2>` line in `karmapad.dts` is critical — without it you get
  a 2-pixel garbage strip on the left edge.
- **Encoder direction reversed**: swap `a-gpios` and `b-gpios` in the dts.
- **Encoder skips steps**: change `steps = <80>` to `<40>` (for 20-detent encoders) or `<24>`.
- **WS2812 ghost LED at end**: increase `chain-length` by 1 and ignore the last LED.
- **BLE won't pair**: on SETTINGS layer press K4 (`&bt BT_CLR`) then re-pair.
