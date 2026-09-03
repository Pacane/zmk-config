# ZMK Studio

[ZMK Studio](https://zmk.dev/docs/features/studio) lets you edit the keymap live from a browser.
It is enabled on the pacane; that took three things:

1. A physical layout with key positions for Studio's visual editor:
   [`boards/shields/pacane/pacane-layouts.dtsi`](../boards/shields/pacane/pacane-layouts.dtsi).
2. `chosen` in [`pacane.dtsi`](../boards/shields/pacane/pacane.dtsi) pointing at
   `zmk,physical-layout` instead of `zmk,matrix-transform`.
3. `CONFIG_ZMK_STUDIO=y` in [`config/pacane.conf`](../config/pacane.conf), and the
   `studio-rpc-usb-uart` snippet on the builds (see [`build.yaml`](../build.yaml)).

The pacane, temper and pacino keymaps carry `&studio_unlock` on their `KBM` layer (reached from
`LOW`); Studio needs that press before it will edit anything. The other shields only have a matrix transform, so Studio
is not enabled for them -- the pacane layout file is the template if you want to add one.
