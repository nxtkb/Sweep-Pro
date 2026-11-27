# TPS43 右手三指手势固件

云端构建目标：`sweep_right_tps43_gestures.uf2`。适用于新右手 PCB、nice!nano v2 / nRF52840、TPS43；这是右手独立主控测试版，不等待左手配对。不要把它当作普通右手从机使用。

`sweep_right.uf2` 仍为原有普通右手从机，角色没有更改。旧左手 TPS 构建也保留。

## 手势

| 操作 | 输出 |
| --- | --- |
| 单指移动 | 鼠标移动 |
| 单指轻点 | 左键 |
| 双指轻点 | 右键 |
| 单指按住后移动 | 左键拖动，保持时间配置 250 ms |
| 双指纵向移动 | 自然方向滚动 |
| 双指左滑 / 右滑 | Command + ] / Command + [ |
| 三指上滑 / 下滑 | Ctrl + Up / Ctrl + Down |
| 三指左滑 / 右滑 | Ctrl + Right / Ctrl + Left |

三指滑动仅接受恰好三指，四指不触发这组快捷键；实际放指和抬指的误触情况仍需实机验证。翻页和三指滑动阈值为 60。双指横滑优先用于翻页，不再是普通水平滚动。快捷键沿用原映射，不随按键层自动改成 Windows 方案。

## 引脚与角色

I2C 0x74 / 400 kHz；RDY=P0.17、NRST=P0.20（低有效）、SCL=P0.22、SDA=P0.24，3.3 V 供电。P0.24 原软关机唤醒输入及其引用已停用，以免与 SDA 冲突。保留原有按键、旋钮、USB 日志及 Studio 串口。

只刷右手，先接 USB 验证。BLE 名称为 `SweepR TPS43`。没有增加清空配对或动态键位的操作。

## 云端构建

推送到 `tps65` 或 `main`，或手动运行 Build firmware。下载成功运行中的 `sweep-pro-firmware`，选择 `sweep_right_tps43_gestures.uf2`。

`build.yaml` 为此目标额外加载 `config/sweep_right_tps65_test.conf` 和 `.overlay`；保留旧配置文件名以便追踪历史，不代表硬件型号是 TPS65。

`config/west.yml` 已固定引用驱动提交 `4b74d4eaeab70114289a65e98bdac7d08996b343`。三指限定已经包含在驱动中，云端无需额外应用临时补丁 `iqs5xx-three-finger-only.patch`。手势开关和快捷键映射位于右手 overlay，仅有驱动支持并不会自动启用。

驱动仓库保持私有，其组织策略禁止部署密钥。Actions 使用仓库密钥 `TPS43_DRIVER_READ_TOKEN`：必须是仅选择 `nxtkb/zmk-driver-azoteq-iqs5xx` 仓库、权限仅为 `Contents: Read-only` 的细粒度令牌（Metadata 的自动只读权限保留）。不使用本机已有的广权限登录令牌。需由有权限的用户创建；若组织要求审批，应先批准。

把令牌直接填入 Sweep-Pro 的 Settings / Secrets and variables / Actions，不要发到聊天或写进仓库。构建仅为这个依赖设置临时进程内 HTTPS 认证头，不写入 URL 或凭据文件，不改变全局 Git 配置。令牌和编码后的认证头均隐藏于日志。来自其他仓库的 PR 不运行这套需要私有依赖的构建。令牌到期后需更新同名 Actions 密钥。
