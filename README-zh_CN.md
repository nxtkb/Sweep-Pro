# NXTKB Ferris Sweep Pro ZMK 配置

[![Firmware Build](https://img.shields.io/github/actions/workflow/status/nxtkb/Sweep-Pro/build.yml?branch=main&label=firmware%20build&style=flat-square)](https://github.com/nxtkb/Sweep-Pro/actions/workflows/build.yml)
[![Official Docs](https://img.shields.io/badge/docs-nxtkb.com-007c78?style=flat-square)](https://nxtkb.com/zh/docs/setup/ferris-sweep-pro/)
[![Keymap Source](https://img.shields.io/badge/keymap-sweep.keymap-5f6fbf?style=flat-square)](./config/sweep.keymap)
[![ZMK Studio](https://img.shields.io/badge/ZMK-Studio-8b5cf6?style=flat-square)](https://zmk.studio/)

中文 | [English](./README.md)

34 键矮轴分体键盘，支持 USB、蓝牙和 ZMK Studio，可选左侧墨水屏与左侧 Azoteq TPS65 触控板。
这个仓库保存键盘配置、默认键位和固件构建流程；上手与日常使用请从[官方使用指南](https://nxtkb.com/zh/docs/setup/ferris-sweep-pro/)开始。
产品规格、轴体、配件和配色见[官网产品页](https://nxtkb.com/zh/products/ferris-sweep-pro/)。

## 官方文档

- [Ferris Sweep Pro 使用指南](https://nxtkb.com/zh/docs/setup/ferris-sweep-pro/)
- [默认键位与层级](https://nxtkb.com/zh/docs/setup/keymap/ferris-sweep-pro-keymap/)
- [连接设备](https://nxtkb.com/zh/docs/setup/connect/)
- [使用 ZMK Studio 或源码改键](https://nxtkb.com/zh/docs/setup/keymap/how-to-update-keymaps/)
- [固件下载与刷写](https://nxtkb.com/zh/docs/firmware/how-to-flash-a-firmware/)
- [固件配置文件](https://nxtkb.com/zh/docs/firmware/sweep-pro-configuration/)
- [Codex / ChatGPT 按键使用](https://nxtkb.com/zh/blog/codex-micro-chatgpt-protocol/)

## 固件与改键

[下载发布固件](https://github.com/nxtkb/Sweep-Pro/releases)，按使用指南选择对应硬件的左右半固件。
日常改键可使用 [ZMK Studio](https://zmk.studio/)；需要修改固件功能时：

1. Fork 本仓库，修改键位或配置文件。
2. 等待 [GitHub Actions](https://github.com/nxtkb/Sweep-Pro/actions/workflows/build.yml) 构建固件。
3. 下载 UF2，按[刷写指南](https://nxtkb.com/zh/docs/firmware/how-to-flash-a-firmware/)更新对应半边。

本地开发见[本地编译固件](docs/zh_CN/local-build.md)。

新右手 PCB 的 TPS43 独立手势测试版为 `sweep_right_tps43_gestures.uf2`，包含双指翻页和三指四方向操作；与普通右手从机固件不同。引脚、操作及云端构建见 [TPS43 右手手势说明](docs/zh_CN/tps43-right-gestures.md)。

## 仓库结构

| 路径 | 用途 |
| :--- | :--- |
| [`config/sweep.keymap`](./config/sweep.keymap) | 默认层、按键与行为绑定。 |
| [`config/sweep.conf`](./config/sweep.conf) | USB、蓝牙、ZMK Studio 等固件选项。 |
| [`config/west.yml`](./config/west.yml) | 固定的 ZMK 与模块依赖。 |
| [`build.yaml`](./build.yaml) | 左右半及硬件版本的构建目标。 |
| [`.github/workflows/build.yml`](./.github/workflows/build.yml) | GitHub Actions 固件构建入口。 |

## 支持

[产品与购买](https://nxtkb.com/zh/products/ferris-sweep-pro/) · [键鼠测试](https://nxtkb.com/zh/docs/setup/keymap/input-tester/)

Discord：`@lilin0522` · 微信：`nxtkb888`
