# Building locally

CI builds every entry of [`build.yaml`](../build.yaml) on each push; this is for building on your
own machine.

## Prerequisites

- [Zephyr SDK](https://docs.zephyrproject.org/latest/develop/toolchains/zephyr_sdk.html)
- Python 3.10 with `west`, `protobuf` and `grpcio-tools`: `pip install west protobuf grpcio-tools`
- CMake and Ninja: `brew install cmake ninja`

## First-time setup

```bash
cd config
west init -l .
west update
```

## Build

From the repo root, with a shield name from the table below:

```bash
west build -s zmk/app -b nice_nano/nrf52840/zmk -p -- \
  -DSHIELD=pacane_left \
  -DZMK_CONFIG="$(pwd)/config" \
  -DBOARD_ROOT="$(pwd)"
```

The firmware lands in `build/zmk.uf2`. `-p` (pristine) is required when switching shields; drop
it to rebuild the same shield faster.

| Keyboard | `-DSHIELD=` |
|----------|-------------|
| Pacane | `pacane_left`, `pacane_right` |
| Pacane with the dongle as central | `pacane_dongle` (the dongle), `pacane_dongle_left`, `pacane_right` |
| Pacane Corne | `pacane_corne_left`, `pacane_corne_right` |
| Temper with nice!view | `"temper_left nice_view_adapter nice_view"`, same with `temper_right` |
| Pacino, hand-wired | `pacino_left`, `pacino_right` |
| Pacino, slim PCB | `pacino_pcb_left`, `pacino_pcb_right` |
| Trackball | `trackball` |
| Wipe a half's settings (bonds) | `settings_reset` |

### Another keymap than the shield's default

Pass the file explicitly; the pacino ships [two](pacino-keymaps.md):

```bash
  -DKEYMAP_FILE="$(pwd)/boards/shields/pacino/pacino_pacane.keymap"
```

### ZMK Studio transport

The pacane CI builds add the `studio-rpc-usb-uart` snippet; locally that is `-S studio-rpc-usb-uart`
before the `--`. See [zmk-studio.md](zmk-studio.md).
