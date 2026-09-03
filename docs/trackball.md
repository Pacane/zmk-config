# PMW3610 Trackball Build Notes

## Hardware

- **Controller:** nice!nano v2 (nRF52840)
- **Sensor board:** [Skree PMW3610 board](https://skree.us/products/zmk-compatible-pmw3610-board) (based on [siderakb/pmw3610-pcb](https://github.com/siderakb/pmw3610-pcb))
- **Keyboard designer:** [Cosmos](https://ryanis.cool/cosmos)
- **Firmware:** ZMK

---

## Wiring: PMW3610 Board → nice!nano

The PMW3610 uses a **single bidirectional data line** (SDIO) instead of separate MOSI/MISO.

| PMW3610 Pad | Function                  | nice!nano Pin (example) | nRF52840 GPIO |
|-------------|---------------------------|-------------------------|---------------|
| SCLK        | SPI Clock                 | Any free GPIO           | e.g. P0.08    |
| SDIO        | SPI Data (MOSI + MISO)    | Any free GPIO           | e.g. P0.17    |
| NCS         | Chip Select (active low)  | Any free GPIO           | e.g. P0.20    |
| MOT         | Motion interrupt (act low)| Any free GPIO           | e.g. P0.06    |
| GND         | Ground                    | GND                     | —             |
| VDD         | Power (3.3V)              | VCC                     | —             |

**Set the board's solder jumper to 3.3V** (nice!nano runs 3.3V logic).

### nice!nano Pins to Avoid

- **P0.04** — reserved for battery voltage ADC
- **P0.13** — controls VCC power (LED power saving)
- Any pins already used by your key matrix

### nice!nano v2 Full Pinout Reference

Left side (top to bottom): RAW, GND, GND, VCC, P0.04, P0.06, P0.08, P1.09, P0.20, P0.22, P0.24, P1.00, P0.11

Right side (top to bottom): VCC, P0.05, P0.07, P0.09, P0.10, P0.26, P0.27, P0.29, P0.31, P1.13, P1.11, P1.15, GND

Ref: https://nicekeyboards.com/docs/nice-nano/pinout-schematic/

---

## Driver Setup (badjeff's driver)

ZMK does **not** have native PMW3610 support. You need an external driver module.

### west.yml

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: badjeff
      url-base: https://github.com/badjeff
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: zmk-pmw3610-driver
      remote: badjeff
      revision: main
  self:
    path: config
```

### Key differences from inorichi's driver

- Compatible string: `"pixart,pmw3610-alt"` (not `"pixart,pmw3610"`)
- Config prefix: `CONFIG_PMW3610_ALT_*` (not `CONFIG_PMW3610_*`)
- Uses `CONFIG_ZMK_POINTING` (not `CONFIG_ZMK_MOUSE`)
- Axis options (swap-xy, invert-x, invert-y, cpi) are set in device tree, not Kconfig
- Supports split peripheral shields

### For split keyboards

You may also need [badjeff/zmk-split-peripheral-input-relay](https://github.com/badjeff/zmk-split-peripheral-input-relay) to forward trackball input from peripheral to central half.

---

## Standalone Trackball Build (no keyboard)

ZMK still requires a kscan driver even with no keys. Use `zmk,kscan-mock`.

### Shield file structure: `config/boards/shields/trackball/`

#### trackball.zmk.yml

```yaml
file_format: "2"
id: trackball
name: Trackball
type: shield
features:
  - keys: false
  - pointing: true
```

#### trackball.conf

```conf
CONFIG_SPI=y
CONFIG_INPUT=y
CONFIG_ZMK_POINTING=y
CONFIG_PMW3610_ALT=y
CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS=1000
```

#### trackball.keymap

```dts
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>

/ {
    keymap {
        compatible = "zmk,keymap";
        default_layer {
            bindings = <>;
        };
    };
};
```

#### trackball.overlay

```dts
#include <zephyr/dt-bindings/input/input-event-codes.h>

/ {
    chosen {
        zmk,kscan = &mock_kscan;
    };

    mock_kscan: mock_kscan_0 {
        compatible = "zmk,kscan-mock";
        columns = <0>;
        rows = <0>;
        events = <0>;
    };

    trackball_listener {
        compatible = "zmk,input-listener";
        device = <&trackball>;
    };
};

&pinctrl {
    spi0_default: spi0_default {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 0, 8)>,
                    <NRF_PSEL(SPIM_MOSI, 0, 17)>,
                    <NRF_PSEL(SPIM_MISO, 0, 17)>;
        };
    };
    spi0_sleep: spi0_sleep {
        group1 {
            psels = <NRF_PSEL(SPIM_SCK, 0, 8)>,
                    <NRF_PSEL(SPIM_MOSI, 0, 17)>,
                    <NRF_PSEL(SPIM_MISO, 0, 17)>;
            low-power-enable;
        };
    };
};

&spi0 {
    status = "okay";
    compatible = "nordic,nrf-spim";
    pinctrl-0 = <&spi0_default>;
    pinctrl-1 = <&spi0_sleep>;
    pinctrl-names = "default", "sleep";
    cs-gpios = <&gpio0 20 GPIO_ACTIVE_LOW>;

    trackball: trackball@0 {
        compatible = "pixart,pmw3610-alt";
        reg = <0>;
        spi-max-frequency = <2000000>;
        irq-gpios = <&gpio0 6 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
        cpi = <600>;
        evt-type = <INPUT_EV_REL>;
        x-input-code = <INPUT_REL_X>;
        y-input-code = <INPUT_REL_Y>;
    };
};
```

### Build command

```
west build -b nice_nano_v2 -- -DSHIELD=trackball
```

### Adding mouse buttons

Replace `zmk,kscan-mock` with `zmk,kscan-gpio-direct` and wire buttons to GPIOs:

```dts
kscan: kscan {
    compatible = "zmk,kscan-gpio-direct";
    input-gpios = <&gpio0 22 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
};
```

---

## Optional Device Tree Properties

| Property              | Purpose                                  |
|-----------------------|------------------------------------------|
| `cpi = <600>`        | Counts per inch sensitivity              |
| `swap-xy`            | Swap X/Y axes                            |
| `invert-x`           | Flip X direction                         |
| `invert-y`           | Flip Y direction                         |
| `force-awake`        | Keep sensor active during ZMK ACTIVE     |
| `force-awake-4ms-mode` | 4ms sampling (250Hz) instead of 8ms    |

## Optional Kconfig Parameters

| Parameter                                        | Purpose                    |
|--------------------------------------------------|----------------------------|
| `CONFIG_PMW3610_ALT_REPORT_INTERVAL_MIN=12`      | Min report interval (ms)   |
| `CONFIG_PMW3610_ALT_LOG_LEVEL_DBG=y`             | Debug logging              |
| `CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS=1000` | Fix "product id 0xFF" error |

---

## Sources

- [Skree PMW3610 Board](https://skree.us/products/zmk-compatible-pmw3610-board)
- [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)
- [inorichi/zmk-pmw3610-driver](https://github.com/inorichi/zmk-pmw3610-driver)
- [nice!nano Pinout](https://nicekeyboards.com/docs/nice-nano/pinout-schematic/)
- [siderakb/pmw3610-pcb](https://github.com/siderakb/pmw3610-pcb)
- [ZMK Kscan Config](https://zmk.dev/docs/config/kscan)
- [ZMK Pointing Devices](https://zmk.dev/docs/features/pointing)
