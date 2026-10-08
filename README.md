# crestronCIP — Q-SYS Crestron CIP Protocol Gateway

Q-SYS Designer plugin that connects Q-SYS Core to Crestron control systems via the CIP (Crestron Internet Protocol) over TCP (port 41794). Supports configurable Digital / Analog / Serial signal counts, auto-reconnect with watchdog, and real-time monitoring.

Q-SYS Designer 插件，通过 CIP 协议（TCP 端口 41794）连接 Q-SYS Core 与 Crestron 中控系统。支持可配置的数字量/模拟量/串量数量、带看门狗的自动重连、实时监控日志。

---

## Features / 功能特性

- **Configurable signal counts / 可配置信号数量**: 0–500 Digital, Analog, Serial signals independently, 50 per panel page
- **Auto-connect on startup / 启动自动连接**: No manual Connect click required; connects immediately when Core starts
- **Auto-reconnect / 自动重连**: Reconnects after TCP disconnect, error, or Crestron program reload (0x03)
- **Two-level watchdog / 两级看门狗**: Detects dead TCP connections (20s no data) and fake CIP sessions (15s registered but no signal)
- **Digital pulse stretching / 数字量脉冲拉伸**: Short trigger pulses (10ms) are stretched to configurable width (default 0.3s) for reliable Crestron reception
- **Analog value display / 模拟量数值显示**: Each Analog fader shows its numeric value beside the slider
- **Real-time monitor log / 实时监控日志**: All signal changes and connection events logged with timestamp, flow direction, and value
- **Heartbeat handling / 心跳处理**: Replies to 0x0D heartbeat requests, sends 0x0D every 15s
- **Multi-subpacket parsing / 多子包解析**: Correctly parses 0x05 data frames containing multiple Cresnet subpackets
- **Panel IP/IPID override / 面板 IP/IPID 覆盖**: Default IP/IPID from Properties, editable on the Connection page

---

## Installation / 安装

1. Copy `crestronCIP.qplug` to your Q-SYS Designer Plugins folder:
   - Windows: `C:\Users\<username>\Documents\QSC\Q-Sys Designer\Plugins\`
2. Restart Q-SYS Designer (or refresh the plugin list)
3. Drag the **crestronCIP** component from the `wxl_personal_plug` category into your schematic
4. Configure the Properties (see below)
5. The plugin auto-connects on startup — no manual Connect needed

1. 将 `crestronCIP.qplug` 复制到 Q-SYS Designer 插件目录：
   - Windows: `C:\Users\<用户名>\Documents\QSC\Q-Sys Designer\Plugins\`
2. 重启 Q-SYS Designer（或刷新插件列表）
3. 从 `wxl_personal_plug` 分类中拖入 **crestronCIP** 组件
4. 配置属性参数（见下文）
5. 插件启动时自动连接，无需手动点击 Connect

---

## Properties / 属性参数

| Property | Type | Range | Default | Description / 说明 |
|----------|------|-------|---------|-------------------|
| `cip_ip` | string | — | `192.168.1.100` | Crestron processor IP address (panel IP_Input overrides) / Crestron 中控 IP 地址（面板可覆盖） |
| `cip_ipid` | string | hex `03`–`FF` | `03` | Device IP-ID in hex / 设备 IP-ID（十六进制） |
| `digital_count` | double | 0–500 | `32` | Number of Digital signals / 数字量数量 |
| `analog_count` | double | 0–500 | `32` | Number of Analog signals / 模拟量数量 |
| `serial_count` | double | 0–500 | `32` | Number of Serial signals / 串量数量 |
| `digital_pulse_width` | double | 0.1–2.0s | `0.3` | Stretch duration for short digital trigger pulses / 短触发脉冲拉伸时长 |
| `auto_reconnect` | string | `true`/`false` | `true` | Auto-reconnect on disconnect / 断开后自动重连 |
| `reconnect_interval` | double | 1–60s | `5` | Seconds between reconnect attempts / 重连间隔（秒） |

> **Note / 注意**: After changing `*_count` properties, delete and re-drag the component to regenerate pins.
> 修改 `*_count` 属性后，需删除并重新拖入组件以重新生成引脚。

---

## Control Pins / 控制引脚

### Connection / 连接控制

| Pin | Direction | Type | Description / 说明 |
|-----|-----------|------|-------------------|
| `CIP_Connect` | Input | Toggle | Connect/disconnect (auto-set ON on startup) / 连接/断开（启动自动 ON） |
| `CIP_Status` | Output | Toggle | Connection status indicator / 连接状态指示灯 |
| `monitor_log` | Output | Text | Real-time log output (external pin available) / 实时日志输出 |

### Digital Signals / 数字量 (1–N)

| Pin | Direction | Type | Description / 说明 |
|-----|-----------|------|-------------------|
| `Digital_In_N` | Output | Toggle | Crestron → Q-SYS boolean signal / Crestron 发 Q-SYS 开关量 |
| `Digital_Out_N` | Input | Toggle | Q-SYS → Crestron boolean signal (pulse stretched) / Q-SYS 发 Crestron 开关量（脉冲拉伸） |

### Analog Signals / 模拟量 (1–N)

| Pin | Direction | Type | Description / 说明 |
|-----|-----------|------|-------------------|
| `Analog_In_N` | Output | Knob (0–65535) | Crestron → Q-SYS 16-bit unsigned integer / Crestron 发 Q-SYS 16位无符号整数 |
| `Analog_Out_N` | Input | Knob (0–65535) | Q-SYS → Crestron 16-bit unsigned integer / Q-SYS 发 Crestron 16位无符号整数 |

> `Analog_*_N_Val` are panel-only text displays — **no external pin**.
> `Analog_*_N_Val` 仅为面板数值显示，**无外部引脚**。

### Serial Signals / 串量 (1–N)

| Pin | Direction | Type | Description / 说明 |
|-----|-----------|------|-------------------|
| `Serial_In_N` | Output | Text | Crestron → Q-SYS text string / Crestron 发 Q-SYS 字符串 |
| `Serial_Out_N` | Input | Text | Q-SYS → Crestron text string / Q-SYS 发 Crestron 字符串 |

---

## Panel Pages / 面板页面

### Connection
- IP Address and IPID input fields (override Properties)
- Connect button and Status indicator
- Configured signal counts display
- Bilingual Notes with full parameter and pin reference

### Monitor / 监控
- Real-time log of all CIP activity (max 50 lines, auto-scroll)
- Clear Log button
- Log format: `HH:MM:SS [Flow] [Type Dir N] = value`
  - Flow: `[QSC->Crestron]` / `[Crestron->QSC]` / `[System]`
  - Example: `14:30:05 [QSC->Crestron] [Digital Out 5] = true`

### Digital / Analog / Serial Pages
- 50 signal pairs per page (In from Crestron | Out to Crestron)
- Analog pages show numeric value next to each fader
- Numbered labels for easy identification

---

## CIP Protocol Details / CIP 协议细节

### Connection Flow / 连接流程

```
1. TCP connect to <ip>:41794
2. Wait for 0x0F Program Ready (timeout 3s, then register anyway)
3. Send IPID Registration: 01 00 0B 00 00 00 00 00 <IPID> 40 FF FF F1 01
4. Receive 0x02 Registration Response
5. Send Update Request: 05 00 05 00 00 02 03 00 (get initial join values)
6. Start 15s heartbeat (0x0D)
7. Normal operation — data flows both ways
```

### Packet Types / 包类型

| Type | Name / 名称 | Handling / 处理 |
|------|-------------|----------------|
| `0x01` | Registration / 注册 | Sent by Q-SYS / Q-SYS 发送 |
| `0x02` | Registration Response / 注册响应 | Sets registered flag / 设置注册标志 |
| `0x03` | Disconnect / 断开 | Force reconnect (not user disconnect) / 强制重连 |
| `0x05` | Data Frame / 数据帧 | Parse subpackets (Digital 0x00, Analog 0x14, Command 0x03) / 解析子包 |
| `0x0D` | Heartbeat Request / 心跳请求 | Reply with 0x0E / 回复 0x0E |
| `0x0E` | Heartbeat Response / 心跳响应 | Ignore (confirms liveness) / 忽略（确认存活） |
| `0x0F` | Program Status / 程序状态 | 0=loading, 1=stopped, 2=ready -> register / 2=就绪->注册 |
| `0x12` | Extended Data / 扩展数据 | Parse Serial (0x34) / 解析串量 |

### Watchdog Logic / 看门狗逻辑

- **Level 1 (20s)**: No TCP data received at all -> Crestron unreachable -> force reconnect
  无任何 TCP 数据 -> Crestron 不可达 -> 强制重连
- **Level 2 (15s)**: CIP registered but no real signal data -> fake session -> force reconnect
  CIP 已注册但无真实信号数据 -> 假会话 -> 强制重连
- Uses `os.time()` (wall clock), not `os.clock()` (CPU time, unreliable on Core)
  使用 `os.time()`（墙上时钟），而非 `os.clock()`（CPU 时间，Core 上不可靠）

---

## Usage Example / 使用示例

### Basic Digital Signal / 基础数字量

```
Crestron Digital Join 1 (button press) -> crestronCIP.Digital_In_1 -> Q-SYS Logic
Q-SYS Trigger -> crestronCIP.Digital_Out_1 -> Crestron Digital Join 1 (LED on)
```

### Analog Level / 模拟量

```
Crestron Analog Join 5 (volume 0-65535) -> crestronCIP.Analog_In_5 -> Q-SYS Gain
Q-SYS Knob -> crestronCIP.Analog_Out_5 -> Crestron Analog Join 5 (level display)
```

### Serial Text / 串量文本

```
Crestron Serial Join 3 ("Hello") -> crestronCIP.Serial_In_3 -> Q-SYS Text Display
Q-SYS Text -> crestronCIP.Serial_Out_3 -> Crestron Serial Join 3 (string)
```

---

## Changelog / 版本历史

### v1.1.2 (2026-10-08)
- **Fixed / 修复**: Watchdog now uses `os.time()` instead of `os.clock()` (was unreliable on Q-SYS Core, causing no auto-reconnect after Crestron reboot)
  看门狗改用 `os.time()` 代替 `os.clock()`（Core 上不可靠，导致 Crestron 重启后不自动重连）
- **Added / 新增**: Level 2 watchdog — detects fake CIP sessions (registered but no signal data for 15s)
  二级看门狗——检测假 CIP 会话（注册后 15 秒无信号数据）
- **Added / 新增**: `forceReconnect()` unified function, eliminates duplicate cleanup code
  统一 `forceReconnect()` 函数，消除重复清理代码
- **Added / 新增**: `lastSignalTime` tracking for real signal data (digital/analog/serial), heartbeats excluded
  `lastSignalTime` 跟踪真实信号数据（数字/模拟/串量），排除心跳
- Watchdog timeout reduced from 30s to 20s for faster recovery
  看门狗超时从 30 秒降至 20 秒，恢复更快

### v1.1.1
- **Added / 新增**: Watchdog timeout (30s no data -> force reconnect) to catch dead TCP after Crestron reboot/power loss
  心跳超时检测（30 秒无数据->强制重连），解决 Crestron 重启/断电后 TCP 假连接

### v1.1.0
- **Added / 新增**: Default auto-connect on startup (no manual Connect click needed)
  启动默认自动连接，无需手动点击 Connect
- **Added / 新增**: Auto-reconnect on TCP CLOSED/ERROR and 0x03 processor disconnect
  TCP 断开/错误和 0x03 中控断开时自动重连
- **Added / 新增**: 0x0F Program Status handling (wait for ready before register)
  0x0F 程序状态处理（就绪后再注册）
- **Added / 新增**: 0x0D heartbeat request reply (0x0E)
  0x0D 心跳请求回复（0x0E）
- **Added / 新增**: Multi-subpacket parsing in 0x05 data frames
  0x05 数据帧多子包解析
- **Added / 新增**: Digital pulse stretching (configurable width, default 0.3s)
  数字量脉冲拉伸（可配置宽度，默认 0.3 秒）
- **Added / 新增**: Analog value display text next to each fader
  每个模拟量滑条旁显示数值
- **Added / 新增**: Monitor page with real-time structured log
  Monitor 页面实时结构化日志

### v1.0.0
- Initial release / 初始版本

---

## Troubleshooting / 故障排查

| Symptom / 现象 | Cause / 原因 | Solution / 解决 |
|----------------|-------------|----------------|
| Connected light on but no signals / 连接灯亮但无信号 | Fake CIP session / 假 CIP 会话 | v1.1.2 Level 2 watchdog auto-reconnects / v1.1.2 二级看门狗自动重连 |
| No reconnect after Crestron reboot / Crestron 重启后不重连 | `os.clock()` not advancing / `os.clock()` 不递增 | Fixed in v1.1.2 (uses `os.time()`) / v1.1.2 已修复（用 `os.time()`） |
| Trigger pulses sometimes missed / 触发脉冲偶尔丢失 | Pulse too short for CIP / 脉冲太短 | Increase `digital_pulse_width` property / 增大 `digital_pulse_width` 属性 |
| Pins don't appear after changing count / 修改数量后引脚不显示 | Component not refreshed / 组件未刷新 | Delete and re-drag component / 删除并重新拖入组件 |
| `Too many Telnet connections` / Telnet 连接过多 | Multiple plugin instances / 多个插件实例 | Use only one crestronCIP per Crestron IPID / 每个 Crestron IPID 只用一个 crestronCIP |

---

# TriggerToMomentary — Trigger to Momentary Pulse Converter

Q-SYS Designer plugin that converts trigger button inputs into momentary (pulsed) outputs. Each trigger fires its corresponding output `true` for a configurable duration, then automatically returns to `false`. Retriggerable — firing again while active resets the timer.

Q-SYS Designer 插件，将触发按钮输入转换为瞬时（脉冲）输出。每次触发使对应输出保持 `true` 一段可配置时长，然后自动恢复为 `false`。可重触发——输出激活时再次触发会重置计时器。

---

## Features / 功能特性

- **Configurable channel count / 可配置通道数**: 0–500 channels, default 32, 50 per panel page
- **Configurable pulse duration / 可配置脉冲时长**: 0.1–10.0 seconds, default 0.5s (Properties + panel knob override)
- **Retriggerable / 可重触发**: Re-firing while output is active resets the countdown
- **Trigger-type input pins / Trigger 类型输入引脚**: `ButtonType = "Trigger"` — accepts external Trigger signals directly
- **Simultaneous channels / 多通道并行**: Multiple channels can be active at the same time
- **Single polling timer / 单定时器驱动**: One 50ms timer drives all channels for efficiency
- **Active count display / 激活数量显示**: Panel shows how many outputs are currently active
- **Panel duration override / 面板时长覆盖**: Knob on Config page overrides property value in real time

---

## Properties / 属性参数

| Property | Type | Range | Default | Description / 说明 |
|----------|------|-------|---------|-------------------|
| `trigger_count` | double | 0–500 | `32` | Number of trigger/momentary channel pairs / 触发-瞬时通道对数量 |
| `momentary_duration` | double | 0.1–10.0s | `0.5` | Default pulse duration in seconds / 默认脉冲时长（秒） |

> **Note / 注意**: After changing `trigger_count`, delete and re-drag the component to regenerate pins.
> 修改 `trigger_count` 后，需删除并重新拖入组件以重新生成引脚。

---

## Control Pins / 控制引脚

| Pin | Direction | Type | Description / 说明 |
|-----|-----------|------|-------------------|
| `Trigger_In_N` | Input | Trigger | Trigger input — fires momentary output on rising edge / 触发输入，上升沿触发瞬时输出 |
| `Momentary_Out_N` | Output | Toggle | Momentary output — `true` for duration, then `false` / 瞬时输出，保持 true 设定时长后 false |

### Panel-Only Controls / 仅面板控件（无外部引脚）

| Control | Type | Description / 说明 |
|---------|------|-------------------|
| `Duration_Input` | Knob (1–100) | Pulse duration override in 0.1s units (5 = 0.5s) / 脉冲时长覆盖，单位 0.1 秒 |
| `Active_Count` | Text | Number of currently active outputs / 当前激活输出数量 |

---

## Panel Pages / 面板页面

### Config
- Duration knob (0.1–10.0s, overrides property)
- Active outputs count display
- Channel count display
- Bilingual Notes with full parameter and pin reference

### Signals (50 per page)
- Two columns: Trigger In (orange) | Momentary Out (green)
- Numbered row labels for easy identification
- Click TRIG button on panel to manually fire a channel

---

## Behavior / 行为逻辑

```
Trigger_In_N fires (rising edge)
  → Momentary_Out_N = true
  → remainingTime[N] = duration
  → channel added to active list
  → 50ms polling timer starts (if not running)

Every 50ms:
  → remainingTime[N] -= 0.05
  → if remainingTime[N] <= 0:
      Momentary_Out_N = false
      channel removed from active list
  → if no active channels: timer stops

If Trigger_In_N fires again while active:
  → remainingTime[N] = duration (reset)
  → output stays true
```

---

## Usage Example / 使用示例

### Projector Power Trigger / 投影机电源触发

```
Q-SYS Schedule Trigger → TriggerToMomentary.Trigger_In_1
TriggerToMomentary.Momentary_Out_1 → Projector Power Input (needs 0.5s pulse)
```

### Screen Lift / 幕布升降

```
Touch Panel Button → TriggerToMomentary.Trigger_In_5
TriggerToMomentary.Momentary_Out_5 → Relay (momentary closure for 1s)
```

### Use with crestronCIP / 与 crestronCIP 配合

```
Crestron Digital Join (momentary) → crestronCIP.Digital_In_1
crestronCIP.Digital_In_1 → TriggerToMomentary.Trigger_In_1
TriggerToMomentary.Momentary_Out_1 → Q-SYS device requiring longer pulse
```

> Set `momentary_duration` to 0.5s or more to ensure downstream devices detect the pulse.
> 将 `momentary_duration` 设为 0.5 秒或更长，确保下游设备能检测到脉冲。

---

## Changelog / 版本历史

### v1.0.0 (2026-10-08)
- Initial release / 初始版本
- 0–500 configurable channels / 0-500 可配置通道
- Configurable 0.1–10.0s pulse duration / 可配置 0.1-10.0 秒脉冲时长
- Retriggerable operation / 可重触发
- Trigger-type input pins / Trigger 类型输入引脚
- Single 50ms polling timer / 单个 50ms 轮询定时器

---

## Demo Files / 演示文件

The `demo/` folder contains test files for CIP integration:

| File | Description / 说明 |
|------|-------------------|
| `demo/CIP.qsys` | Q-SYS Designer test design with crestronCIP and signal routing / 含 crestronCIP 和信号路由的 Q-SYS 测试设计 |
| `demo/CIP_archive.zip` | Compiled Q-SYS design archive for Core deployment / 用于 Core 部署的编译后 Q-SYS 设计归档 |

---

# qsc-qplug-dev — Q-SYS Plugin Development Skill / Q-SYS 插件开发技能

A reusable AI Agent skill that standardizes Q-SYS Designer qplug plugin development. It encapsulates the official Q-SYS Developer Help API reference (37 documents), verified debugging experience, and the `wxl_personal_plug` house style. When loaded, the agent automatically follows these standards for every plugin task — no need to re-explain rules each time.

一个可复用的 AI Agent 技能，标准化 Q-SYS Designer qplug 插件开发。它封装了 Q-SYS 官方开发者帮助 API 参考（37 份文档）、实践验证的调试经验以及 `wxl_personal_plug` 系列风格。加载后，Agent 在每次插件任务中自动遵循这些规范——无需每次重复说明规则。

## Skill Files / 技能文件

```
skill/
├── SKILL.md                          # Skill entry: triggers, workflow, critical rules
└── references/
    └── qplug-standards.md            # Full API reference + pitfalls (12 sections)
```

## What It Covers / 涵盖内容

### 1. Official API Reference / 官方 API 参考
Distilled from all 37 Q-SYS Developer Help PDFs:
- **Reserved Functions** (27-page spec): `GetProperties`, `RectifyProperties`, `GetPages`, `GetControls`, `GetControlLayout`, `GetComponents`, `GetPins`, `GetWiring`, `GetColor`, `GetPrettyName`
- **Control Types**: Button (Toggle/Momentary/Trigger/StateTrigger), Knob (8 ControlUnit types), Indicator (Led/Meter/Text/Status), Text
- **Layout & Graphics**: 10 Style types, 5 Graphics types (Label/GroupBox/Header/Image/Svg), ZOrder layering, 11 font families
- **Properties**: 5 types (string/integer/double/boolean/enum), `IsHidden` dynamic visibility
- **Embedded Components**: `GetComponents` + `GetPins` + `GetWiring` for audio processing
- **Plugin Compiler**: 9-file framework, VS Code + Git workflow

### 2. Code Style / 代码风格
- PascalCase for controls/functions/globals; camelCase for locals
- 2-space indentation, one statement per line
- Design-time vs Runtime organization (`if Controls then`)
- Socket events must print to debug window

### 3. Critical Rules (Non-Negotiable) / 关键规则（必须遵守）
| Rule / 规则 | Why / 原因 |
|-------------|-----------|
| `ButtonType="Trigger"` EventHandler must fire unconditionally | `c.Boolean` is `false` at callback time (pulse already reset) |
| Knob must set `ControlUnit` (e.g. `"Integer"`) | No external pin generated without it |
| `ButtonStyle` only affects panel appearance | Does NOT control pin type — use `ButtonType` for pins |
| Count>1 layout name = `"Name 1"` (with space) | Not `"Name1"` — causes `GetAllControlPanels` error |
| Delete & re-drag component after changing `*_count` | Pins don't regenerate automatically |
| Watchdog uses `os.time()`, not `os.clock()` | `os.clock()` is CPU time, unreliable on Q-SYS Core |
| Panel boxes: bigger is better, leave margins | Avoid clipping/overflow issues |
| Every plugin needs bilingual Notes at bottom / 每个插件底部必须有中英文 Notes | House standard for `wxl_personal_plug` |

### 4. Verified Pitfalls / 实践验证陷阱
10+ real bugs encountered and solved:
- `GetAllControlPanels: <name> not defined` — layout/control name mismatch
- `unexpected symbol near ')'` — extra `)` in layout assignment
- Trigger inputs not firing — `if c.Boolean then` guard blocks all triggers
- Knob pins missing — missing `ControlUnit`
- Fake TCP connection after Crestron reboot — two-level watchdog needed
- `Too many Telnet connections` — multiple plugin instances

### 5. Development Workflow / 开发工作流
1. **Requirements gathering** — list I/O pins, properties, pages, behavior, confirm with user
2. **Author qplug** — single-file structure in official order
3. **Archive** — copy to `Q-Sys Designer\Plugins\`
4. **Test** — drag into Q-SYS Designer, verify pins, panel, runtime logs
5. **Publish** — independent GitHub repo per module, bilingual README

### 6. House Standards / 系列规范
- Category: `wxl_personal_plug~PluginName`
- Author: `longwang`
- Panel text: English only (compatibility)
- Notes: bilingual (English + Chinese)
- GitHub: one repo per plugin, bilingual README
- Coordinates: panel width ~340px, ≥8px margins, ≥4px element spacing

## How to Install / 如何安装

Copy the `skill/` folder to your agent's user skills directory:

```
<agent-workspace>/.user_skills/qsc-qplug-dev/
├── SKILL.md
└── references/qplug-standards.md
```

The skill auto-triggers when you ask to create, edit, or debug any `.qplug` file.

## How to Use / 如何使用

Once installed, simply describe what plugin you need:

> "做一个新的 qplug，输入 3 个 toggle，输出 3 个 toggle，当输入变化时同步输出"

The agent will automatically:
- Follow official API conventions (correct ButtonType, ControlUnit, PinStyle)
- Apply house style (English panel text, bilingual Notes, proper margins)
- Avoid known pitfalls (Trigger Boolean issue, Knob pin issue)
- Output a complete, testable `.qplug` file
- Archive to your Plugins folder
- Offer GitHub publishing with bilingual README

## Version / 版本

- **Skill version**: 1.0.0
- **Based on**: Q-SYS Developer Help (37 documents, 2026)
- **Author**: longwang
- **Part of**: wxl_personal_plug series

---

## Author / 作者

**longwang** — wxl_personal_plug series

Part of the [wxl_personal_plug](https://github.com/xlooong) Q-SYS plugin collection.

---

## License / 许可证

MIT License — free to use and modify.
