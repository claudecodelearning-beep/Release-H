# RELEASE H — 解放双手

一条命令恢复 Mac 工作环境，并通过本地蓝牙按下配置好的 SwitchBot Bot。

## 功能

- 外接屏：Visual Studio Code、Terminal、备忘录、Codex 各占一个全屏空间。
- Mac 内屏：Google Chrome 中的 B 站番茄钟视频全屏、Claude 独立全屏。
- 复用已运行应用，恢复隐藏或最小化的窗口。
- 默认从配置的 B 站 UP 主视频来源随机选择番茄钟，网络不可用时使用内置列表。旧版哔哩哔哩 App 收藏夹坐标方案仍可选。
- 常驻 RELEASE H Agent.app 接收本地请求：先尝试已绑定 Bot 的按压，再在桌面解锁后执行窗口布局。
- 支持 Bot 蓝牙扫描、MAC/UUID 绑定、单次按压及回执记录，无需 Hub。

这是个人 Mac 自动化工具，目前通过命令行和本地 Agent 使用；独立 PWA 尚未实现。

## Benefits

- **替代部分 SwitchBot Hub 的使用场景。** 对于已接入的 Bot，Mac 可充当本地蓝牙控制端。配合现有的 Mac 远程访问通道，即可远程触发 Bot 按压，无需为这一用途额外购买 Hub。
- **电脑工作流与实体按钮一起启动。** 一条命令安排工作应用、番茄钟和已绑定的 Bot，减少重复操作。
- **复用已有设备，降低额外硬件成本。** 如果 Mac 本来就会保持运行，只需增加 Bot 即可实现相应的按键自动化。
- **本地执行和记录。** Bot 控制通过 Mac 蓝牙完成，设备绑定与执行记录保留在本机。

替代范围仅限本项目实现的 Bot 蓝牙按压。Mac 必须保持醒着、运行 Agent，并处于 Bot 的蓝牙范围内；远程触发还需要 Mac 在线和可用的远程通道。本项目不包含 Hub 的红外遥控、Matter 桥接等功能，也不能在 Mac 关机或睡眠时独立工作。

## 环境

- macOS 13 或更新版本，Python 3.10+。
- Xcode Command Line Tools（用于编译 Swift Agent）。
- Mac 内建屏幕和一块外接屏；外接屏不必设为系统主屏。
- 上述工作应用已安装；默认网页番茄钟使用 Google Chrome。
- 硬件功能需要 Mac 蓝牙开启，Bot 在范围内并设置为 Press 模式。

## 安装

下载或克隆仓库，在项目目录执行：

```sh
chmod +x bin/release-h scripts/*.sh
./scripts/install-local.sh
./scripts/install-agent.sh
.venv/bin/release-h start-work --dry-run
```

安装器在本机编译并临时签名 Agent，安装到 ~/Applications/RELEASE H Agent.app，注册当前用户的登录启动项。生成的 App 绑定当前项目路径；移动项目后需重新安装。安装器不需要管理员权限。

在系统设置中完成一次授权：

1. 隐私与安全性 → 辅助功能：允许 RELEASE H Agent。
2. 隐私与安全性 → 自动化：允许它控制 System Events 和目标应用。
3. 桌面与程序坞 → Mission Control：启用“显示器具有单独的空间”，按系统要求重新登录。
4. 使用 Bot 时，在首次扫描弹窗中允许蓝牙访问。

授权不会随 GitHub 源码同步。重建或更新 Agent 后，macOS 可能要求重新授权。

## 启动工作环境

发送给常驻 Agent：

```sh
.venv/bin/release-h request-start --json
```

返回 queued 仅表示请求写入成功；执行结果见 runtime/last-result.json。从其他工作目录或远程会话调用时，使用本机项目中 .venv/bin/release-h 的绝对路径。

Mac 醒着但锁屏时，Agent 可以尝试先执行蓝牙按压，桌面操作等解锁后执行。Mac 睡眠、关机、注销时无法保证处理请求。远程访问 Mac 的连接方式需另行配置。

直接执行完整工作流（Mac 已解锁）：

```sh
.venv/bin/release-h start-work
```

模拟运行，不操作桌面或硬件：

```sh
.venv/bin/release-h start-work --dry-run --json
```

## SwitchBot 设置

先在手机 SwitchBot App 添加 Bot、切换至 Press 模式并测试安装位置，然后执行：

```sh
.venv/bin/release-h bot scan --json
.venv/bin/release-h bot bind air_conditioner <实际MAC地址或扫描UUID> --json
.venv/bin/release-h bot status --json
```

角色支持 air_conditioner 和 coffee，只绑定拥有的设备。没有绑定的设备保持 pending_hardware，不会动作。升降桌暂未接入。

明确要立即按一次时使用：

```sh
.venv/bin/release-h bot press air_conditioner --json
```

同一启动请求的每个 Bot 只尝试一次。记录在发送前持久化，轮询、解锁和进程重启不会自动重试该请求；超时也不自动重试，因为按钮可能已按下。再次发送启动请求会创建新请求，可能再次按下按钮。

空调电源按钮通常切换开/关；Bot 不读取空调当前状态。成功回执说明 Bot 执行了动作，不代表空调一定被打开。本版本不支持需要密码认证的 Bot。

详细步骤见 [SwitchBot 设置说明](docs/switchbot-setup.md)。协议参考 [SwitchBot 官方 BLE 文档](https://github.com/OpenWonderLabs/SwitchBotAPI-BLE/blob/latest/devicetypes/bot.md)。

## 配置与运行记录

应用名称、B 站来源及旧版收藏夹参数位于 src/release_h/config.py。内置视频列表位于 src/release_h/data/fallback_videos.json。

项目中的 runtime/ 包含本机设备绑定、请求、结果和日志，已从 Git 排除：

- switchbot.json：本机 Bot 角色与设备地址。
- pending-start-work.json / running-start-work.json：等待及正在处理的请求。
- last-result.json：最近一次桌面工作流及硬件结果。
- bot-attempts/：每个请求的硬件尝试记录。
- agent.log：Agent 状态变化。

等待请求会合并；执行过程中提交的新请求可保留到下一轮。队列只接受固定 start-work 动作，不接受任意 shell 命令。

## 验证与限制

```sh
PYTHONPATH=src python3 -m unittest discover -s tests -v
```

测试覆盖队列、桌面流程、Bot 绑定、模拟运行、锁屏等待、失败处理和防重复按压。蓝牙权限、按钮实际受力和不同设备上的锁屏行为仍需实机验证。

macOS 空间排列和播放器全屏依赖辅助功能及 UI 操作，应用升级可能需要调整。缺少双屏时会跳过窗口布局并返回失败；单步失败会保留结果并允许其他步骤继续。

旧版哔哩哔哩 App 方案可用 start-work --pomodoro-mode app 启用；该方案依赖收藏夹名称和坐标，默认使用网页模式。

仓库只包含源码、测试和安装说明。虚拟环境、设备地址、运行日志、旧交付文档和本机生成的安装包均不上传。
