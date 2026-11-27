# Sweep-Pro for e-ink vX.Y.Z

## ✨ Sweep-Pro Firmware Update

This firmware is for the Sweep-Pro, compatible with the version featuring an e-ink screen and touchpad.

## 🚀 Highlights

- Merged latest upstream changes from ZMK.
- TODO: Add feature or upstream update summary.

## 🖱️ Touchpad Improvements

- TODO: Add touchpad improvement summary.

## 📦 Firmware Selection

- ⌨️ Basic: flash `sweep_left.uf2` and `sweep_right.uf2`.
- 🖥️ E-ink: flash `sweep_left_display.uf2` and `sweep_right.uf2`.
- 🖱️ Left TPS65: flash `sweep_left_tps65.uf2` and `sweep_right.uf2`. Do not combine TPS65 and the display on the left half.
- Right TPS43 standalone gesture test: flash `sweep_right_tps43_gestures.uf2` to the new right PCB only. This is not a split peripheral; do not pair it as the ordinary right half. Includes two-finger page swipes and three-finger directional shortcuts.

## 📥 How to Flash

1. Download the matching `.uf2` files from the Assets section below.
2. Put your nice!nano v2 into bootloader mode by double-tapping reset within 0.5s.
3. Drag and drop the left and right firmware files into the corresponding drives.

## ✅ Build Verification

- TODO: Add successful GitHub Actions build number and commit.
- Release assets were downloaded from that successful workflow run.
