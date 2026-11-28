# Hardware discovery

Add the `zmk-feature-hardware-discovery` checkout to `ZMK_EXTRA_MODULES` in the existing Sweep Pro build. Append `discovery.conf` to `EXTRA_CONF_FILE` and the overlays below to `EXTRA_DTC_OVERLAY_FILE` on the central build. Keep the shields and snippets used by the keyboard configuration.

| Hardware | Central discovery overlays | Matching right-half shields |
| --- | --- | --- |
| Base | `base.overlay` | `sweep_right` |
| Display | `base.overlay;display.overlay` | `sweep_right` |
| Trackpad | `base.overlay;trackpad.overlay` | `sweep_right sweep_right_trackpad` |
| Display and trackpad | `base.overlay;display.overlay;trackpad.overlay` | `sweep_right sweep_right_trackpad` |

Use absolute paths for the overlays and configuration fragment. Keep the display shields when adding `display.overlay`. These overlays describe the hardware. The App continues to discover encoder editing and display configuration through their respective interfaces.

The central reports the remote components declared by its build. The right encoder and trackpad share ZMK split source 0. The App updates their link state as the right half disconnects and reconnects. Use the matching central build when changing the right half.
