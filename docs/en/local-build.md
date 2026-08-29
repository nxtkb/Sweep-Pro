# Sweep-Pro Local Build

This guide describes how to build the Sweep-Pro ZMK firmware locally. First set `NXTKB_ROOT` to your local checkout path:

```shell
export NXTKB_ROOT="/path/to/nxtkb"
cd "$NXTKB_ROOT/zmkfirmware/zmk"
```

The following commands assume you are running them from the ZMK west workspace root. `$NXTKB_ROOT/Sweep-Pro` is the keyboard config and shield repository, not the west workspace root. Do not run `west build` directly inside the `Sweep-Pro` directory.

The local build uses the `nxtkb/zmk` fork, which is maintained as a small set of
NXTKB commits rebased on current official ZMK main. The Sweep-Pro display status
screen lives in the standalone `zmk-vfx-sweep-pro-display` module, so builds
with the display need that module in `ZMK_EXTRA_MODULES` and `sweep_display` in
the `SHIELD` list.

All Sweep-Pro hardware variants share one `config/sweep.keymap`. The display and trackpad are optional shields that get composed into the build, so one repository can produce 4 half-keyboard firmware files. Users only need to pick the UF2 files matching their hardware.

## Dependencies

You need system build tools, the Zephyr SDK, and `uv`.

Arch Linux example:

```shell
sudo pacman -S git cmake ninja gperf ccache dfu-util dtc wget \
    tk xz file make uv
```

Install the Zephyr SDK from AUR, or install it manually by following the ZMK/Zephyr documentation:

```shell
paru -S zephyr-sdk
```

On Apple Silicon macOS, install the host tools with Homebrew and install the Zephyr SDK version
required by the ZMK revision pinned in `config/west.yml` (currently 0.17.0):

```shell
brew install cmake ninja ccache dtc wget xz protobuf
```

If the SDK is not detected automatically, point Zephyr at your installation in the current shell:

```shell
export ZEPHYR_TOOLCHAIN_VARIANT=zephyr
export ZEPHYR_SDK_INSTALL_DIR="/path/to/zephyr-sdk-0.17.0"
```

## Initialize Python

Create a project-local virtual environment from the ZMK workspace root:

```shell
export NXTKB_ROOT="/path/to/nxtkb"
cd "$NXTKB_ROOT/zmkfirmware/zmk"
uv venv --python 3.13
source .venv/bin/activate
uv pip install west
```

Reactivate the virtual environment in every new terminal before building:

```shell
export NXTKB_ROOT="/path/to/nxtkb"
cd "$NXTKB_ROOT/zmkfirmware/zmk"
source .venv/bin/activate
```

## Initialize West

Run this once for a fresh checkout:

```shell
west init -l app/
west update
west zephyr-export
uv pip install -r zephyr/scripts/requirements-base.txt protobuf
```

`west zephyr-export` writes to the user-level CMake package registry. `west update` and `uv pip install` need network access.

## Build Sweep-Pro

Common parameters:

```shell
export NXTKB_ROOT="/path/to/nxtkb"
EXTRA_MODULES="$NXTKB_ROOT/Sweep-Pro;$NXTKB_ROOT/zmk-vfx-sweep-pro-display;$NXTKB_ROOT/cirque-input-module;$NXTKB_ROOT/zmk-behavior-report;$NXTKB_ROOT/zmk-behavior-send-string"
ZMK_CONFIG_DIR="$NXTKB_ROOT/Sweep-Pro/config"
```

The recommended output set is 4 half-keyboard firmware files:

| Firmware | Shield combination | Use |
| :--- | :--- | :--- |
| `sweep_left` | `sweep_left` | Base left half, no display |
| `sweep_left_display` | `sweep_left sweep_left_display_hw sweep_display` | Left half with e-ink display |
| `sweep_right` | `sweep_right` | Base right half, no trackpad |
| `sweep_right_trackpad` | `sweep_right sweep_right_trackpad` | Right half with Cirque trackpad |

Use these combinations for the 4 keyboard variants:

| Keyboard variant | Left UF2 | Right UF2 |
| :--- | :--- | :--- |
| Basic | `sweep_left` | `sweep_right` |
| E-ink | `sweep_left_display` | `sweep_right` |
| Trackpad | `sweep_left` | `sweep_right_trackpad` |
| Flagship | `sweep_left_display` | `sweep_right_trackpad` |

Base left half. Studio RPC over USB UART is useful for ZMK Studio remapping:

```shell
west build -s app -p -d build/sweep_left -b nice_nano//zmk \
    -S studio-rpc-usb-uart -- \
    -DSHIELD=sweep_left \
    -DZMK_EXTRA_MODULES="$EXTRA_MODULES" \
    -DZMK_CONFIG="$ZMK_CONFIG_DIR"
```

Left half with display. `sweep_left_display_hw` provides the e-ink hardware node, and `sweep_display` provides the custom status screen UI:

```shell
west build -s app -p -d build/sweep_left_display -b nice_nano//zmk \
    -S studio-rpc-usb-uart -- \
    -DSHIELD="sweep_left sweep_left_display_hw sweep_display" \
    -DZMK_EXTRA_MODULES="$EXTRA_MODULES" \
    -DZMK_CONFIG="$ZMK_CONFIG_DIR"
```

Base right half:

```shell
west build -s app -p -d build/sweep_right -b nice_nano//zmk -- \
    -DSHIELD=sweep_right \
    -DZMK_EXTRA_MODULES="$EXTRA_MODULES" \
    -DZMK_CONFIG="$ZMK_CONFIG_DIR"
```

Right half with trackpad. `sweep_right_trackpad` provides the Cirque I2C/Pinnacle node:

```shell
west build -s app -p -d build/sweep_right_trackpad -b nice_nano//zmk -- \
    -DSHIELD="sweep_right sweep_right_trackpad" \
    -DZMK_EXTRA_MODULES="$EXTRA_MODULES" \
    -DZMK_CONFIG="$ZMK_CONFIG_DIR"
```

The commands explicitly use `-s app` because the current directory is the workspace root, while the ZMK application source is under `app/`.

After a successful build, the firmware files are:

```text
build/sweep_left/zephyr/zmk.uf2
build/sweep_left_display/zephyr/zmk.uf2
build/sweep_right/zephyr/zmk.uf2
build/sweep_right_trackpad/zephyr/zmk.uf2
```

For later builds with the same CMake parameters, you can usually reuse the build directories:

```shell
west build -d build/sweep_left
west build -d build/sweep_left_display
west build -d build/sweep_right
west build -d build/sweep_right_trackpad
```

If you change the shield, extra modules, snippets, or `ZMK_CONFIG`, rerun the full command with `-p` to regenerate the build directory.

## Codex Micro USB and Bluetooth build

This build adds a second vendor-defined USB HID interface and an encrypted vendor-defined
HID-over-GATT report while preserving the normal ZMK USB/Bluetooth HID and Studio USB UART
interfaces. Codex RPC follows ZMK's selected output endpoint, including the active Bluetooth
profile. Inactive USB and Bluetooth hosts cannot take over the Codex session by retrying.

The repository's default GitHub Actions workflow builds Codex-enabled central
firmware with the pinned `nxtkb/zmk` revision. That fork already contains the
required USB HID interrupt-OUT support, so no separate patch step is needed.
For a local checkout, add the Codex module to the module list:

```shell
CODEX_EXTRA_MODULES="$EXTRA_MODULES;$NXTKB_ROOT/zmk-feature-codex-micro"
```

Build the left half with display and both Codex transports:

```shell
west build -s app -p -d build/sweep_left_display_codex -b nice_nano//zmk \
    -S studio-rpc-usb-uart \
    -S nxtkb-codex-micro-usb \
    -S nxtkb-codex-micro-ble \
    -S nxtkb-codex-micro-compat-identity -- \
    -DSHIELD="sweep_left sweep_left_display_hw sweep_display" \
    -DZMK_EXTRA_MODULES="$CODEX_EXTRA_MODULES" \
    -DZMK_CONFIG="$ZMK_CONFIG_DIR"
```

The module includes the optional `nxtkb-codex-micro-compat-identity` snippet for interoperability
with the current ChatGPT desktop discovery. It changes only the USB VID/PID and Bluetooth PnP
VID/PID. Product, manufacturer, Bluetooth, and Device Information names remain owned by the
keyboard configuration. Sweep Pro release builds enable this snippet.

These identifiers do not imply OpenAI certification or an identifier assignment to NXTKB or
third-party keyboard makers. The discovery behavior is undocumented and may change. External
integrators are responsible for determining whether they are authorized to distribute firmware
using the compatibility identifiers.
The right half continues to use the normal `sweep_right` or `sweep_right_trackpad` firmware.

After first flashing a BLE-enabled build, forget the old keyboard in the host Bluetooth settings,
clear the selected ZMK Bluetooth profile, and pair with the keyboard's configured name again. HID report maps are
cached in the bond. Use the existing output-toggle key to prefer Bluetooth; a connected USB cable
may still be used for charging. USB and Bluetooth keyboard connections remain active instead of
disconnecting each other. Every connected computer keeps an independent Codex protocol and Agent
state session; Codex keys and the display follow only the currently selected ZMK output. Switching
back restores the last Agent state reported by that computer.

## Troubleshooting

If you see:

```text
west: unknown command "build"; do you need to run this inside a workspace?
```

you are not in the west workspace. Return to:

```shell
export NXTKB_ROOT="/path/to/nxtkb"
cd "$NXTKB_ROOT/zmkfirmware/zmk"
source .venv/bin/activate
```

If you see:

```text
source directory "." does not contain a CMakeLists.txt
```

you ran the command from the workspace root without `-s app`.

The right-half build may show Kconfig warnings about USB or central battery proxy options being disabled for the split peripheral role. Those warnings come from the left/right role difference. If `zmk.uf2` is generated, the build succeeded. Trackpad smooth scrolling is configured in the left central firmware; the right trackpad firmware only captures and forwards input events.
