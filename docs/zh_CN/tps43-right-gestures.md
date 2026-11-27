# TPS43 右手手势固件

云端构建目标：`sweep_right_tps43_gestures.uf2`。适用于新右手 PCB、nice!nano v2 / nRF52840、TPS43；这是右手独立主控测试版，不等待左手配对。不要把它当作普通右手从机使用。

`sweep_right.uf2` 仍为原有普通右手从机，角色没有更改。旧左手 TPS 构建也保留。

## 手势

| 操作 | 输出 |
| --- | --- |
| 单指移动 | 鼠标移动 |
| 单指轻点 | 左键 |
| 双指轻点 | 右键 |
| 单指按住后移动 | 左键拖动，保持时间配置 250 ms |
| 单指轻点两下，第二下不抬手 | 左键拖动 |
| 双指纵向或横向移动 | 标准滚动事件，使用自然方向 |
| 双指从右边缘向左扫，随后抬手 | `F19`，可绑定为显示通知中心 |
| 三指上滑 / 下滑 | Ctrl + Up / Ctrl + Down |
| 三指左滑 / 右滑 | Ctrl + Right / Ctrl + Left |
| 三指轻点 | Ctrl + Command + D，查找指针下的词语 |

三指滑动阈值为 40，仅接受恰好三指。双指从中间起手的横滑保持标准横向滚动，不发送浏览器前进或后退快捷键。手势快捷键不会随按键层改成 Windows 方案。

## macOS 通知中心

在「系统设置 → 键盘 → 键盘快捷键 → 调度中心」勾选「显示通知中心」。双击该行的快捷键，两指在 TPS43 最右侧靠边落下，向左扫再抬手；看到 `F19` 后点「完成」。此设置每台 Mac 只需做一次，平时蓝牙使用无需桥接或 USB 连线。

## 引脚与角色

I2C 0x74 / 400 kHz；RDY=P0.17、NRST=P0.20（低有效）、SCL=P0.22、SDA=P0.24，3.3 V 供电。P0.24 原软关机唤醒输入及其引用已停用，以免与 SDA 冲突。保留原有按键、旋钮、USB 日志及 Studio 串口。

只刷右手，先接 USB 验证。BLE 名称为 `SweepR TPS43`。没有增加清空配对或动态键位的操作。

## 云端构建

推送到 `tps65` 或 `main`，或手动运行 Build firmware。下载成功运行中的 `sweep-pro-firmware`，选择 `sweep_right_tps43_gestures.uf2`。

`build.yaml` 为此目标加载 `config/sweep_right_tps43_gestures.overlay` 和 USB 手势诊断配置 `config/sweep_right_tps43_diag.conf`。基础接线和独立角色仍来自旧名 `sweep_right_tps65_test` 的配置，不代表硬件型号是 TPS65。

`config/west.yml` 固定引用包含当前手势的驱动版本，不需要额外应用本地测试补丁。手势开关和快捷键映射位于右手 overlay，不会自动启用在其他构建目标上。

驱动仓库保持私有，其组织策略禁止部署密钥。Actions 使用仓库密钥 `TPS43_DRIVER_READ_TOKEN`：必须是仅选择 `nxtkb/zmk-driver-azoteq-iqs5xx` 仓库、权限仅为 `Contents: Read-only` 的细粒度令牌（Metadata 的自动只读权限保留）。不使用本机已有的广权限登录令牌。需由有权限的用户创建；若组织要求审批，应先批准。

把令牌直接填入 Sweep-Pro 的 Settings / Secrets and variables / Actions，不要发到聊天或写进仓库。构建仅为这个依赖设置临时进程内 HTTPS 认证头，不写入 URL 或凭据文件，不改变全局 Git 配置。令牌和编码后的认证头均隐藏于日志。来自其他仓库的 PR 不运行这套需要私有依赖的构建。令牌到期后需更新同名 Actions 密钥。
