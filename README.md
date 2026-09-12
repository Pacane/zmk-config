# ZMK Config

Personal [ZMK](https://zmk.dev) firmware for my split keyboards. Every push builds all of them on
[GitHub Actions](https://github.com/Pacane/zmk-config/actions): download the `.uf2` from the run's artifacts, double-tap reset on
the nice!nano, drag the file onto the drive that appears.

## Keyboards

| Keyboard | Keys | Shield | Keymap | Notes |
|----------|------|--------|--------|-------|
| Pacane | 58 | [`pacane`](boards/shields/pacane) | [pacane.keymap](boards/shields/pacane/pacane.keymap) | [ZMK Studio](docs/zmk-studio.md); optional [dongle](boards/shields/pacane_dongle) as the central |
| Pacane Corne | 42 | [`pacane_corne`](boards/shields/pacane_corne) | [pacane_corne.keymap](boards/shields/pacane_corne/pacane_corne.keymap) | |
| Temper | 36 | [`temper`](boards/shields/temper) | [temper.keymap](boards/shields/temper/temper.keymap) | nice!view display |
| Pacino | 40 | [`pacino`](boards/shields/pacino), [`pacino_pcb`](boards/shields/pacino_pcb) | [two variants](docs/pacino-keymaps.md) | from [Pacane/pacino](https://github.com/Pacane/pacino); [nice!view wiring](docs/pacino-nice-view.md) |
| Trackball | | [`trackball`](boards/shields/trackball) | [trackball.keymap](boards/shields/trackball/trackball.keymap) | PMW3610, [build notes](docs/trackball.md) |

All on nice!nano v2 (`nice_nano/nrf52840/zmk`); all keyboard keymaps have a mouse layer.

## Docs

- [Building locally](docs/building.md)
- [Pacino keymaps](docs/pacino-keymaps.md) -- the two variants, and running one half alone
- [Pacino nice!view wiring](docs/pacino-nice-view.md)
- [ZMK Studio](docs/zmk-studio.md)
- [Trackball build notes](docs/trackball.md)

## Layout of the repo

- `boards/shields/<name>/` -- shield definition and its keymap
- `config/<name>.conf` -- per-shield Kconfig options
- [`build.yaml`](build.yaml) -- the CI build matrix ([workflow](.github/workflows/build.yml))
