# Pacino: wiring the nice!view

The pacino wiring guide only draws the key matrix; the display is five more wires from the
nice!view's header (its pads are labelled on the board) to the nice!nano, on the pins ZMK's stock
`nice_view_adapter` expects. The nice!view is built to take an SSD1306 OLED header's four pins plus
one extra pin for chip select:

| nice!view pad | nice!nano pin | ZMK name | note |
|---------------|---------------|----------|------|
| VCC | VCC (the switched 3.3 V pin -- not RAW, not B+) | -- | ZMK powers it through ext-power |
| GND | GND | -- | |
| CS | P0.06 (silkscreen `006`) | `pro_micro 1` / D1 | the "bodge" pin the adapter assumes |
| MOSI (SDA) | P0.17 (`017`) | `pro_micro 2` / D2 | the OLED header's SDA |
| SCL / SCK | P0.20 (`020`) | `pro_micro 3` / D3 | the OLED header's SCL |

None of those are matrix pins on the hand-wired pacino (rows `pro_micro` 20/19/18/16/10, columns
5-9), so **row 3 stays on pin 16**. The pacino README's note about moving row 3 to 21 assumed the
display needs the AVR pro micro's SPI pins (14/15/16); the nRF52840 can put SPI on any pin, and the
adapter uses the OLED-header pins instead.

Firmware: add `nice_view_adapter nice_view` after the half's shield -- `build.yaml` has both halves,
locally it is `-DSHIELD="pacino_right nice_view_adapter nice_view"`. Nothing to add to
`pacino.conf`: the nice_view shield brings `CONFIG_ZMK_DISPLAY=y` itself (and the adapter disables
the nice!nano's I2C, which the pacino does not use).

The slim PCB build is the exception: its reversible board uses pins 2 and 3 for the matrix and keeps
1/14/15/16 free, so a display on `pacino_pcb` needs its own `nice_view_spi` definition on those pins
instead of the stock adapter.
