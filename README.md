# ZMK Config

Personal ZMK firmware configuration for custom keyboards.

## Keyboards

| Shield | Keys | Type | Features |
|--------|------|------|----------|
| Pacane | 58 | Split (pro_micro) | Mouse, ZMK Studio |
| Pacane Corne | 42 | Split (pro_micro) | Pointing |
| Temper | 36 | Split (pro_micro) | Pointing, OLED, RGB underglow |
| Pacane Macro | 16 | Single (pro_micro) | Sleep |

All split keyboards use `nice_nano_v2` boards.

## ZMK Studio

The Pacane shield has ZMK Studio support enabled. This required:

1. A physical layout file (`boards/shields/pacane/pacane-layouts.dtsi`) defining key positions for Studio's visual editor.
2. Switching `chosen` in `pacane.dtsi` from `zmk,matrix-transform` to `zmk,physical-layout`.
3. Adding `CONFIG_ZMK_STUDIO=y` to `config/pacane.conf`.

## Building Locally

### Prerequisites

- [Zephyr SDK](https://docs.zephyrproject.org/latest/develop/toolchains/zephyr_sdk.html)
- Python 3.10.x with `west`, `protobuf`, and `grpcio-tools`:
  ```
  pip install west protobuf grpcio-tools
  ```
- CMake and Ninja:
  ```
  brew install cmake ninja
  ```

### Setup (first time)

```bash
cd config
west init -l .
west update
```

### Build

From the repo root:

```bash
# Pacane left
west build -s zmk/app -b nice_nano/nrf52840/zmk -- \
  -DSHIELD=pacane_left \
  -DZMK_CONFIG="$(pwd)/config" \
  -DBOARD_ROOT="$(pwd)"

# Pacane right (use -p for pristine rebuild when switching shields)
west build -s zmk/app -b nice_nano/nrf52840/zmk -p -- \
  -DSHIELD=pacane_right \
  -DZMK_CONFIG="$(pwd)/config" \
  -DBOARD_ROOT="$(pwd)"
```

Output firmware is at `build/zmk.uf2`.

### Other shields

```bash
# Temper (with nice_view display)
west build -s zmk/app -b nice_nano/nrf52840/zmk -p -- \
  -DSHIELD="temper_left nice_view_adapter nice_view" \
  -DZMK_CONFIG="$(pwd)/config" \
  -DBOARD_ROOT="$(pwd)"

# Pacane Corne
west build -s zmk/app -b nice_nano/nrf52840/zmk -p -- \
  -DSHIELD=pacane_corne_left \
  -DZMK_CONFIG="$(pwd)/config" \
  -DBOARD_ROOT="$(pwd)"

# Pacane Macro
west build -s zmk/app -b nice_nano/nrf52840/zmk -p -- \
  -DSHIELD=pacane_macro \
  -DZMK_CONFIG="$(pwd)/config" \
  -DBOARD_ROOT="$(pwd)"
```

## CI

Pushes and pull requests automatically build all shields via GitHub Actions (see `.github/workflows/build.yml` and `build.yaml`).
