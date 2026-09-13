# Pacino slim PCB: the controller sits on the other face

The nano goes **on top of the board** -- the face opposite the hot-swap sockets and diodes --
component side down, its back flush in the plate's controller window (`z_mcu_bot = z_pcb_top +
mcu_socket_h` in `keyboard.scad`; there is only 2.4 mm under the board). Until September 2026 the
pacino PCB generator assumed the controller sat on the parts face instead, so its per-half pin tables
and `L`/`R` jumper letters were the wrong way round; the generator, its READMEs and the overlays in
both repos now agree with the case. Flipping which face the nano sits on exchanges its two pin rows.

**Boards from before that fix** (the first batch): the letter printed on the controller's face is the
*other* half's, but it is still the pad to bridge --

| half | parts face | nano on | bridge JP1 / JP2 to the pad marked | ZMK pins |
|------|-----------|---------|------------------------------------|----------|
| right | front | back / top | **`L`** (the mark you can see from the top) | rows `pro_micro` 21/20/19/18/10, cols inner-first 9/5/4/3/2 |
| left | back | front / top | **`R`** | rows 2/3/4/5/9, cols pinky-first 21/20/19/18/10 |

Those boards also have the reset footprint's nets on the wrong leg pairs (a 12 mm tactile links the
legs 12.5 mm apart, the footprint linked the 5 mm ones), so fitted normally the switch shorts RST to
GND and the nano never boots: fit it by **two diagonal legs only**. And their jumpers sit under the
bay's corner boss, where a solder bridge stops the case closing: bypass them with two wires instead
(`PWR` middle pin to the nano's RAW hole, `BAT -` to its GND hole) and leave the pads bare.

Rule of thumb on any board: **bridge the marked pad on the face the nano is on.**

Get it wrong (nano on top, jumpers to the parts-face letter) and the battery goes into GPIO P0.06
through its protection diode and out of the nano's VCC pin: the nano heats up within seconds and
does not enumerate over USB. Check with a meter, nano out: `JP1` centre to pad 24 and `JP2` centre to
pad 4 closed on the right half (pad 1 / pad 21 on the left), and pad 24-to-pad 4 open. Pad 1 is the
square pad at the USB end; pads 13-24 run back along the other row, so pad 24 is opposite pad 1.

P0.06 (`pro_micro 1`) is the nice!view CS line and not a matrix pin, so a nano that survived the
short with a dead P0.06 still works as a keyboard.

The battery itself goes to `BAT +` / `-` (through the `PWR` slide switch and `JP1` to the nano's
RAW; `-` through `JP2` to GND). The nano's own B+ / B- pads stay empty.
