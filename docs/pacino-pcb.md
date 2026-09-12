# Pacino slim PCB: the controller sits on the other face

The pacino README's `pcb/*/README.md` says to put the controller sockets "on the same face as
everything else" and gives per-half pin tables and `L`/`R` jumper choices on that basis. The case
disagrees: the nano goes **on top of the board**, component side down, its back flush in the plate's
controller window (`z_mcu_bot = z_pcb_top + mcu_socket_h` in `keyboard.scad`; there is only 2.4 mm
under the board). The hot-swap sockets and diodes are on the underside, the controller sockets on the
top face.

Flipping which face the nano sits on exchanges its two pin rows, so relative to those tables:

| half | parts face | nano on | bridge JP1 / JP2 to | ZMK pins (this repo) |
|------|-----------|---------|---------------------|----------------------|
| right | front (`R` marks) | back / top, where the `L` marks are | **`L`** | the README's *left*-half list: rows `pro_micro` 21/20/19/18/10, cols 2/3/4/5/9 |
| left | back (`L` marks) | front / top, where the `R` marks are | **`R`** | the README's *right*-half list: rows 2/3/4/5/9, cols 10/18/19/20/21 |

Rule of thumb: **bridge to the letter printed on the face the nano is on.**

Get it wrong (nano on top, jumpers to the parts-face letter) and the battery goes into GPIO P0.06
through its protection diode and out of the nano's VCC pin: the nano heats up within seconds and
does not enumerate over USB. Check with a meter, nano out: `JP1` centre to pad 24 and `JP2` centre to
pad 4 closed on the right half (pad 1 / pad 21 on the left), and pad 24-to-pad 4 open. Pad 1 is the
square pad at the USB end; pads 13-24 run back along the other row, so pad 24 is opposite pad 1.

P0.06 (`pro_micro 1`) is the nice!view CS line and not a matrix pin, so a nano that survived the
short with a dead P0.06 still works as a keyboard.

The battery itself goes to `BAT +` / `-` (through the `PWR` slide switch and `JP1` to the nano's
RAW; `-` through `JP2` to GND). The nano's own B+ / B- pads stay empty.
