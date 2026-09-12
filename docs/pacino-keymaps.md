# Pacino keymaps

The pacino comes from [Pacane/pacino](https://github.com/Pacane/pacino). It is the temper's 36 keys
(3x5 + 3 thumbs per half) plus two extra keys per half, under X/C and under ,/. -- and two shields:

- [`pacino`](../boards/shields/pacino) -- the hand-wired build
- [`pacino_pcb`](../boards/shields/pacino_pcb) -- the reversible slim PCB (same layout, but the
  flipped board puts the matrix on different pins on each half)

Both shields share the keymaps in `boards/shields/pacino/`.

## The two variants

| Keymap | Based on | Extras (left / right) |
|--------|----------|-----------------------|
| [`pacino_temper.keymap`](../boards/shields/pacino/pacino_temper.keymap) | the [temper keymap](../boards/shields/temper/temper.keymap)'s finger keys and combos, verbatim | shift/caps_word, ctrl·TAB / è, LALT |
| [`pacino_pacane.keymap`](../boards/shields/pacino/pacino_pacane.keymap) | the [pacane keymap](../boards/shields/pacane/pacane.keymap): its 3x5 block, no combos | shift/caps_word, ctrl·TAB / MEDIA, LALT |

Both use the **pacane disposition on the right thumbs** -- DEL·NUM, RET·RAI, BSPC·MOU -- because
the pacino's thumb fan sits further out than the temper's, which places the middle and inner keys
better. On the left, the temper variant keeps the temper cluster (ESC·NMP, TAB·LOW, SPACE·FUN);
the pacane variant uses the pacane's (ESC·NMP, SPACE·LOW, RET·FUN).

The header comment of each file has the full per-layer table for the extras and lists what was
relocated from the pacane's number row and outer columns. Both put `&bootloader` / `&sys_reset` on
each half's extras in the keyboard layer, so either half can be flashed from the keymap.

## Picking one

[`pacino.keymap`](../boards/shields/pacino/pacino.keymap) is a one-line `#include` choosing the
default (temper); `pacino_pcb.keymap` follows it. Change the include to switch.

CI builds both variants for both shields: the plain `pacino_*` artifacts use the default keymap, the
`*_pacane` ones select the other file with `-DKEYMAP_FILE=...` in [`build.yaml`](../build.yaml)
(same flag [locally](building.md#another-keymap-than-the-shields-default)).

## Testing a single half

The shield's `Kconfig.defconfig` makes the left half the central. To run the right half on its own
(only one half wired yet), give it the central role in a `config/pacino_right.conf`:

```
CONFIG_ZMK_SPLIT_ROLE_CENTRAL=y
CONFIG_ZMK_KEYBOARD_NAME="Pacino"
```

It then shows up over USB and BLE by itself. To go back to the normal split later: delete the file,
flash `settings_reset` on that half (it stored bonds as a central), flash the regular `pacino_right`
firmware on it and `pacino_left` on the left, and pair the host again with the left half.

(While the pacino's left half did not exist yet, a throwaway `temper_pacino_left` shield -- temper-left
hardware with the pacino's position numbering, peripheral role -- let the temper's left half stand in;
it is in git history before the commit "pacino: the left half is the central again".)
