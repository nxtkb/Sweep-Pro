# Sweep-Pro 本地编译

本文记录在本机编译 Sweep-Pro ZMK 固件的流程。先按你的实际 checkout 位置设置 `NXTKB_ROOT`：

```shell
export NXTKB_ROOT="/path/to/nxtkb"
cd "$NXTKB_ROOT/zmkfirmware/zmk"
```

后续命令默认从 ZMK workspace 根目录执行。`$NXTKB_ROOT/Sweep-Pro` 是键盘配置和 shield 仓库，不是 west workspace 根目录。不要在 `Sweep-Pro` 目录里直接执行 `west build`。

当前本地编译使用 `nxtkb/zmk` fork。该 fork 仅保留少量 NXTKB 必需 commit，
并持续 rebase 到官方 ZMK main。Sweep-Pro 的屏幕状态栏位于独立模块
`zmk-vfx-sweep-pro-display` 中，所以编译带屏幕的左手固件时需要同时加入该
module，并把 `sweep_display` 放进 `SHIELD` 列表。

Sweep-Pro 的 keymap 是所有硬件版本共用的一份 `config/sweep.keymap`。屏幕和触控板作为可选 shield 组合进构建，因此同一个仓库可以产出 4 个半边固件，用户只需要按自己的硬件版本选择对应 UF2。

## 依赖

本机需要先准备系统构建工具、Zephyr SDK 和 `uv`。

Arch Linux 示例：

```shell
sudo pacman -S git cmake ninja gperf ccache dfu-util dtc wget \
    tk xz file make uv
```

Zephyr SDK 可以通过 AUR 安装，或按 ZMK/Zephyr 官方文档手动安装：

```shell
paru -S zephyr-sdk
```

macOS（Apple Silicon）建议使用 Homebrew 安装宿主工具，并安装 `config/west.yml` 中固定的
ZMK revision 所要求的 Zephyr SDK 版本（当前为 0.17.0）：

```shell
brew install cmake ninja ccache dtc wget xz protobuf
```

如果 SDK 没有被自动识别，请在当前 shell 里指向你自己的安装目录：

```shell
export ZEPHYR_TOOLCHAIN_VARIANT=zephyr
export ZEPHYR_SDK_INSTALL_DIR="/path/to/zephyr-sdk-0.17.0"
```

## 初始化 Python 环境

在 ZMK workspace 根目录创建项目内虚拟环境：

```shell
export NXTKB_ROOT="/path/to/nxtkb"
cd "$NXTKB_ROOT/zmkfirmware/zmk"
uv venv --python 3.13
source .venv/bin/activate
uv pip install west
```

每次新开终端编译前，都需要重新激活虚拟环境：

```shell
export NXTKB_ROOT="/path/to/nxtkb"
cd "$NXTKB_ROOT/zmkfirmware/zmk"
source .venv/bin/activate
```

## 初始化 west workspace

第一次配置这个 checkout 时执行：

```shell
west init -l app/
west update
west zephyr-export
uv pip install -r zephyr/scripts/requirements-base.txt protobuf
```

`west zephyr-export` 会写入用户级 CMake package registry；`west update` 和 `uv pip install` 需要联网。

## 编译 Sweep-Pro

公共参数：

```shell
export NXTKB_ROOT="/path/to/nxtkb"
EXTRA_MODULES="$NXTKB_ROOT/Sweep-Pro;$NXTKB_ROOT/zmk-vfx-sweep-pro-display;$NXTKB_ROOT/cirque-input-module;$NXTKB_ROOT/zmk-behavior-report;$NXTKB_ROOT/zmk-behavior-send-string"
ZMK_CONFIG_DIR="$NXTKB_ROOT/Sweep-Pro/config"
```

推荐一次性编译 4 个半边固件：

| 固件 | Shield 组合 | 用途 |
| :--- | :--- | :--- |
| `sweep_left` | `sweep_left` | 左手基础版，不带屏幕 |
| `sweep_left_display` | `sweep_left sweep_left_display_hw sweep_display` | 左手带 e-ink 屏幕 |
| `sweep_right` | `sweep_right` | 右手基础版，不带触控板 |
| `sweep_right_trackpad` | `sweep_right sweep_right_trackpad` | 右手带 Cirque 触控板 |

四种整机版本对应关系：

| 整机版本 | 左手 UF2 | 右手 UF2 |
| :--- | :--- | :--- |
| Basic | `sweep_left` | `sweep_right` |
| E-ink | `sweep_left_display` | `sweep_right` |
| Trackpad | `sweep_left` | `sweep_right_trackpad` |
| Flagship | `sweep_left_display` | `sweep_right_trackpad` |

左手基础版。建议启用 Studio RPC over USB UART，方便用 ZMK Studio 改键：

```shell
west build -s app -p -d build/sweep_left -b nice_nano//zmk \
    -S studio-rpc-usb-uart -- \
    -DSHIELD=sweep_left \
    -DZMK_EXTRA_MODULES="$EXTRA_MODULES" \
    -DZMK_CONFIG="$ZMK_CONFIG_DIR"
```

左手带屏幕。`sweep_left_display_hw` 提供 e-ink 硬件节点，`sweep_display` 提供自定义状态栏 UI：

```shell
west build -s app -p -d build/sweep_left_display -b nice_nano//zmk \
    -S studio-rpc-usb-uart -- \
    -DSHIELD="sweep_left sweep_left_display_hw sweep_display" \
    -DZMK_EXTRA_MODULES="$EXTRA_MODULES" \
    -DZMK_CONFIG="$ZMK_CONFIG_DIR"
```

右手基础版：

```shell
west build -s app -p -d build/sweep_right -b nice_nano//zmk -- \
    -DSHIELD=sweep_right \
    -DZMK_EXTRA_MODULES="$EXTRA_MODULES" \
    -DZMK_CONFIG="$ZMK_CONFIG_DIR"
```

右手带触控板。`sweep_right_trackpad` 提供 Cirque I2C/Pinnacle 节点：

```shell
west build -s app -p -d build/sweep_right_trackpad -b nice_nano//zmk -- \
    -DSHIELD="sweep_right sweep_right_trackpad" \
    -DZMK_EXTRA_MODULES="$EXTRA_MODULES" \
    -DZMK_CONFIG="$ZMK_CONFIG_DIR"
```

这里显式使用 `-s app`，因为当前目录是 workspace 根目录，ZMK 应用源码在 `app/` 下。

构建成功后，固件位于：

```text
build/sweep_left/zephyr/zmk.uf2
build/sweep_left_display/zephyr/zmk.uf2
build/sweep_right/zephyr/zmk.uf2
build/sweep_right_trackpad/zephyr/zmk.uf2
```

第二次编译同一个 build 目录时，如果 CMake 参数没有变化，可以直接执行：

```shell
west build -d build/sweep_left
west build -d build/sweep_left_display
west build -d build/sweep_right
west build -d build/sweep_right_trackpad
```

修改了 shield、extra modules、snippets 或 `ZMK_CONFIG` 后，建议继续使用带 `-p` 的完整命令重新生成构建目录。

## Codex Micro USB 与蓝牙构建

该构建会增加第二个 vendor-defined USB HID interface，以及加密的 vendor-defined
HID-over-GATT report，同时保留 ZMK 普通 USB/蓝牙 HID 和 Studio USB UART。Codex RPC
响应发起请求的传输通道。

仓库的默认 GitHub Actions workflow 使用固定版本的 `nxtkb/zmk` 构建启用 Codex 的
central 固件。该 fork 已经包含所需的 USB HID interrupt-OUT 支持，不再需要单独应用
patch。本地构建时，只需把 Codex 模块加入 module 列表：

```shell
CODEX_EXTRA_MODULES="$EXTRA_MODULES;$NXTKB_ROOT/zmk-feature-codex-micro"
```

编译同时支持 USB 和蓝牙 Codex 通道的左手墨水屏固件：

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

模块包含可选的 `nxtkb-codex-micro-compat-identity` snippet，用于兼容当前 ChatGPT 桌面端
的发现机制。它只修改 USB VID/PID 和蓝牙 PnP VID/PID；产品名、厂商名、蓝牙名称与
Device Information 名称仍由键盘配置决定。Sweep Pro 的发布固件会启用该 snippet。

这些标识符不代表 OpenAI 认证，也不代表这些标识符已分配给 NXTKB 或第三方键盘厂商。
ChatGPT 的发现机制没有公开文档且可能变化；外部集成方需要自行确认是否有权发布使用这些
兼容标识符的固件。右手继续
使用原有 `sweep_right` 或 `sweep_right_trackpad` 固件。

首次刷入带 BLE 支持的固件后，需要先在主机蓝牙设置中忽略旧键盘、清除所选 ZMK
蓝牙 profile，再按键盘配置的名称重新配对。HID report map 会被缓存在旧 bond 中，普通
断开重连不会刷新。使用现有的 output toggle 键切换到蓝牙优先；USB 线仍可保留用于充电。
USB 和蓝牙键盘连接不会互斥或彼此断开。每个连接的电脑维持独立的 Codex 协议与 Agent
状态；Codex 按键和墨水屏状态只跟随当前 ZMK output。切换回来后会恢复该电脑最后上报的
Agent 状态。

## 常见问题

如果看到：

```text
west: unknown command "build"; do you need to run this inside a workspace?
```

说明当前目录不是 west workspace，回到：

```shell
export NXTKB_ROOT="/path/to/nxtkb"
cd "$NXTKB_ROOT/zmkfirmware/zmk"
source .venv/bin/activate
```

如果看到：

```text
source directory "." does not contain a CMakeLists.txt
```

说明命令从 workspace 根目录执行时缺少 `-s app`。

右手构建可能出现一些 Kconfig 提示，例如 USB 或 central battery proxy 的配置被 split peripheral 角色关闭。这类提示来自左右手角色差异；只要最终生成 `zmk.uf2`，构建就是成功的。触控板滚动的 smooth scrolling 配置放在左手 central 固件中，右手触控板固件只负责采集并转发输入事件。
