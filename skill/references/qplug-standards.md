# Q-SYS Qplug 官方编写规范 / Official Qplug Authoring Standards

> 来源: Q-SYS Developer Help (37 份官方 PDF)
> 核心文档: Reserved Functions (27页), Code Style Guide, Reserved Control Names, Plugin Compiler
> 本地路径: C:\Users\longe\Desktop\Q-SYS 文档\qplug\

---

## 1. 代码风格 / Code Style (官方 Code Style Guide)

### 命名规范 / Naming
| 类别 | 规范 | 示例 |
|------|------|------|
| 控件/函数/别名/全局对象/全局变量 | PascalCase / UpperCamelCase | `MyControl`, `MyFunction()` |
| 局部变量/局部对象 | camelCase / lowerCamelCase | `myVariable`, `localValue` |
| 仓库名/插件文件名 | PascalCase | `TriggerToMomentary.qplug` |
| 分支名 | main / develop / bugfix/xxx / feature/xxx | — |

### 格式 / Formatting
- 每行一条语句，每条语句一个赋值
- 缩进 2 个空格
- 控制结构 (if/while)、运算符、逗号后加空格
- 通用注释在代码块上方一行；细粒度注释在行内 `-- comment`
- 三段 `---***` 注释分隔代码区域

### 文件结构组织 / Organization
**Design Time (设计时):**
1. 单行注释: 插件名 / 开发者 / 开发年月
2. PluginInfo Header
3. Constants
4. Reserved functions
5. Custom functions (设计时用)

**Runtime (运行时, 在 `if Controls then` 内):**
1. `require` 声明 (顶部，字母序)
2. Control Aliases
3. Variables
4. Objects and tables
5. Constants
6. Reserved functions (运行时)
7. Custom functions
8. EventHandlers
9. Initialization function
10. Print statements (Socket 事件必须打印到 debug 窗口)

---

## 2. PluginInfo / 插件元信息

```lua
PluginInfo = {
  Name = "PluginName",           -- PascalCase
  Version = "1.0.0",             -- 语义化版本
  Author = "longwang",
  Id = "GUID",                   -- 唯一 GUID
  Description = "plugin description",
  BuildVersion = 1               -- 可选，编译自增
}
```

---

## 3. 保留函数 / Reserved Functions (官方完整定义)

### 设计时函数 / Design Time Functions

| 函数 | 必填 | 说明 |
|------|------|------|
| `GetPrettyName(props)` | 否 | 组件拖入设计时显示的名称，可含 `PluginInfo.Version` |
| `GetColor(props)` | 否 | 组件颜色 `{r, g, b}`，0-255 |
| `GetProperties()` | 是 | 属性定义表数组 |
| `RectifyProperties(props)` | 否 | 属性变更后调用，可设 `IsHidden` 显隐属性 |
| `GetPages(props)` | 否 | 多页面时返回 `{ {name="Page1"}, ... }` |
| `GetControls(props)` | 是 | 控件定义表数组 |
| `GetControlLayout(props)` | 是 | 返回 `layout, graphics` 两个表 |
| `GetComponents(props)` | 否 | 内嵌音频组件 |
| `GetPins(props)` | 否 | 添加音频/串口引脚（**不是控制引脚**） |
| `GetWiring(props)` | 否 | 内嵌组件与引脚的连线 |

### 运行时入口 / Runtime Entry
```lua
if Controls then
  -- 运行时代码
  -- Properties["name"].Value 读取属性
  -- Controls["name"] 访问控件
end
```

---

## 4. 属性定义 / Properties (GetProperties)

```lua
function GetProperties()
  return {
    { Name = "count",   Type = "integer", Min = 0, Max = 500, Value = 32 },
    { Name = "gain",    Type = "double",  Min = -100, Max = 20, Value = 0 },
    { Name = "ip",      Type = "string",  Value = "192.168.1.100" },
    { Name = "enabled", Type = "boolean", Value = true },
    { Name = "mode",    Type = "enum",    Choices = {"A","B","C"}, Value = "A" },
    { Name = "Header",  Type = "string",  Header = "Section Title" },  -- QDS 9.10+
    { Name = "info",    Type = "string",  Comment = "hint text" }       -- QDS 9.10+
  }
end
```

**属性类型 / Property Types:**
| Type | 说明 | 默认值要求 |
|------|------|-----------|
| `string` | 文本 | 可选 |
| `integer` | 有符号长整数 | 必填 |
| `double` | 双精度浮点 | 必填 |
| `boolean` | Yes/No 下拉，运行时返回 true/false | 必填 |
| `enum` | 下拉框，Choices 定义选项 | 可选（不选则空白） |

**RectifyProperties 显隐属性:**
```lua
function RectifyProperties(props)
  props["Advanced"].IsHidden = props["ShowAdvanced"].Value == "No"
  return props
end
```

**运行时读取:** `Properties["Property Name"].Value`（所有类型统一）

---

## 5. 控件定义 / Controls (GetControls)

### 通用属性 / Common Properties
| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `Name` | String | 是 | 控件名（PascalCase） |
| `ControlType` | String | 是 | `"Button"` / `"Knob"` / `"Indicator"` / `"Text"` |
| `DefaultValue` | varies | 否 | 首次编译时的默认值 |
| `UserPin` | Boolean | 否 | 默认 false。true 时在属性面板"Control Pins"下显示引脚 |
| `PinStyle` | String | 否 | `"Input"` / `"Output"` / `"Both"` / `"None"` |
| `Count` | Integer | 否 | 默认 1。>1 时运行时为 1-indexed 数组 `Controls["name"][i]` |

> **重要**: Count > 1 时，layout 中控件名为 `"Name 1"`, `"Name 2"`（Name + 空格 + 序号）

### Button 控件 / Button Control
| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `ButtonType` | String | 是 | `"Toggle"` / `"Momentary"` / `"Trigger"` / `"StateTrigger"` |
| `Icon` | String | 否 | QDS 内置图标名或本地文件（需 Plugin Compiler 编码） |
| `IconType` | String | 否 | `"Icon"`(默认,支持SVG/PNG/JPG) / `"Image"` |
| `Max` / `Min` | Integer | 否 | StateTrigger 的最大/最小值 |

**ButtonType 详解:**
- `Toggle` — 布尔保持型，引脚为 Boolean，EventHandler 中 `c.Boolean` 可靠
- `Momentary` — 瞬时型（按下 true，松开 false）
- `Trigger` — 触发型，**EventHandler 触发时 `c.Boolean` 为 false**（脉冲太快已复位），必须无条件执行
- `StateTrigger` — 带 Min/Max 范围的状态触发器

### Knob 控件 / Knob Control
| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `ControlUnit` | String | 是 | 见下表 |
| `Max` / `Min` | Integer | 否 | 超出 ControlUnit 默认范围时必填 |

**ControlUnit 选项与默认范围:**
| ControlUnit | 下限 | 上限 | 默认 Min | 默认 Max | Min/Max 必填 |
|-------------|------|------|----------|----------|-------------|
| `dB` | -100 | 20 | -100 | 20 | 是 |
| `Hz` | 20 | 20000 | 20 | 20000 | 是 |
| `Float` | -1e9 | 1e9 | 0 | 100 | 是 |
| `Integer` | -999,999,999 | 999,999,999 | 1 | 100 | 是 |
| `Pan` | -1 | 1 | -1 | 1 | 否 |
| `Percent` | 0 | 100 | 0 | 100 | 是 |
| `Position` | 0 | 1 | 0 | 1 | 否 |
| `Seconds` | 0 | 87400 | 0 | 1 | 是 |

> **关键**: Knob 必须设 `ControlUnit` 才会生成外部引脚！

### Indicator 控件 / Indicator Control
| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `IndicatorType` | String | 是 | `"Led"` / `"Meter"` / `"Text"` / `"Status"` |

### Text 控件 / Text Control
- 无特定属性，在 layout 中用 `Style` 决定是文本框/下拉框/列表框
- 运行时: `ctrl.String = "text"`

### 保留控件名 / Reserved Control Names
以下控件名有特殊含义，应按用途使用:
| 名称 | 用途 |
|------|------|
| `Status` | 连接状态指示 |
| `IPAddress` | IP 地址输入 |
| `MACAddress` | MAC 地址显示 |
| `Username` | 用户名输入 |
| `Password` | 密码输入 |
| `DeviceName` | 设备名 |
| `SerialNumber` | 序列号 |
| `DeviceFirmware` | 固件版本 |

---

## 6. 布局与图形 / Layout & Graphics (GetControlLayout)

### 返回值 / Return
```lua
function GetControlLayout(props)
  local layout = {}
  local graphics = {}
  -- ... 定义布局和图形 ...
  return layout, graphics
end
```

### Layout 控件属性 / Layout Control Properties
| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `Position` | Table `{x,y}` | 是 | 左上角坐标 |
| `Size` | Table `{w,h}` | 是 | 尺寸 |
| `Style` | String | 是 | 见下表 |
| `ClassName` | String | 否 | UCI CSS 类名 |
| `Color` | Table `{r,g,b,a}` | 否 | 控件颜色，alpha 可选 0-255 |
| `TextColor` | Table `{r,g,b,a}` | 否 | 文字颜色 |
| `Font` | String | 否 | 默认 "Roboto" |
| `FontSize` | Integer | 否 | 字号 |
| `FontStyle` | String | 否 | 见字体表 |
| `IsBold` | Boolean | 否 | 粗体 |
| `HTextAlign` | String | 否 | `"Center"`(默认) / `"Left"` / `"Right"` |
| `VTextAlign` | String | 否 | `"Center"`(默认) / `"Top"` / `"Bottom"` |
| `IsReadOnly` | Boolean | 否 | 运行时不可改（状态显示用） |
| `Margin` | Integer | 否 | 默认 0 |
| `Padding` | Integer | 否 | 默认 1 |
| `PrettyName` | String | 否 | 引脚别名，用 `~` 创建子层级 |
| `CornerRadius` / `Radius` | Integer | 否 | 圆角 |
| `StrokeColor` | Table | 否 | 边框颜色 |
| `StrokeWidth` | Integer | 否 | 边框宽度，默认 1 |
| `ZOrder` | Signed Integer | 否 | 垂直层级，越大越靠前（一旦使用建议所有对象都设） |

**Style 选项:**
`"Fader"`, `"Knob"`, `"Button"`, `"Text"`, `"Meter"`, `"Led"`, `"ListBox"`, `"ComboBox"`, `"Media"`, `"None"`（隐藏控件但保留引脚）

### Button-Only Layout 属性
| 属性 | 说明 |
|------|------|
| `ButtonStyle` | `"Toggle"` / `"Momentary"` / `"Trigger"` / `"StateTrigger"` / `"On"` / `"Off"` / `"Custom"` |
| `ButtonVisualStyle` | `"Flat"` / `"Gloss"`(默认) |
| `Legend` | 按钮上显示的文字 |
| `OffColor` | 仅 `UnlinkOffColor=true` 时生效，off 状态颜色 |
| `UnlinkOffColor` | 允许 on/off 完全不同颜色 |
| `IconColor` | 图标颜色 |
| `WordWrap` | Legend 文字是否换行 |
| `CustomButtonUp` / `CustomButtonDown` | Custom 风格按钮的上下文字 |

### Fader-Only: `ShowTextbox` (默认 false)
### Meter-Only: `MeterStyle` ("Level"/"Reduction"/"Gain"/"Standard"), `BackgroundColor`, `ShowTextbox`(默认 true)
### Text-Only: `TextBoxStyle` ("Normal"/"Meter"/"NoBackground"), `WordWrap`

### Graphics 图形 / Graphics Table
| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `Type` | String | 是 | `"Label"` / `"GroupBox"` / `"Header"` / `"Image"` / `"Svg"` |
| `Position` | Table `{x,y}` | 是 | 位置 |
| `Size` | Table `{w,h}` | 是 | 尺寸 |
| `ZOrder` | Signed Integer | 否 | 层级 |

**GroupBox 属性:** `Text`, `Font`, `FontSize`, `FontStyle`, `IsBold`, `HTextAlign`, `StrokeWidth`(0-64), `StrokeColor`, `Color`(文字色), `Fill`(背景色), `CornerRadius`

**Label 属性:** `Text`, `Color`, `Font`, `Fill`, `FontSize`, `FontStyle`, `IsBold`, `HTextAlign`, `VTextAlign`, `StrokeWidth`, `StrokeColor`, `Margin`, `Padding`, `CornerRadius`

**Header 属性:** `Text`, `Font`, `FontSize`, `FontStyle`, `IsBold`, `HTextAlign`, `Color`

**Image/Svg 属性:** `Image` (base64 编码字符串)

### 可用字体 / Fonts
| Font | FontStyle 选项 |
|------|---------------|
| Roboto (默认) | Thin, Thin Italic, Light, Light Italic, Regular, Italic, Medium, Medium Italic, Bold, Bold Italic, Black, Black Italic |
| Roboto Mono | Thin, Thin Italic, Light, Light Italic, Regular, Italic, Medium, Medium Italic, Bold, Bold Italic, Black, Black Italic |
| Open Sans | Light, Light Italic, Regular, Italic, Semibold, Semibold Italic, Bold, Bold Italic, Extrabold, Extrabold Italic |
| Lato | Light, Light Italic, Regular, Italic, Bold, Bold Italic, Black, Black Italic |
| Montserrat | Thin ... Black Italic (全系列) |
| Poppins | Light, Regular, Medium, SemiBold, Bold |
| Noto Serif | Regular, Italic, Bold, BoldItalic |
| Droid Sans | Regular, Bold |

---

## 7. 多页面 / Multiple Pages

`page_index` 是系统内置属性，在 `GetControlLayout` 中用 `props["page_index"].Value` 获取当前页（从 1 开始）。

```lua
-- 方式1: 用数字索引
if props["page_index"].Value == 1 then
  -- 第一页控件
elseif props["page_index"].Value == 2 then
  -- 第二页控件
end

-- 方式2: 用页面名
local currentPage = PageNames[props["page_index"].Value]
if currentPage == "Config" then ... end
```

> **注意**: `GetProperties()` 中对 PageNames 的修改不会传递到 `GetControls()`/`GetControlLayout()`。如需动态页面，在这两个函数内重新调用辅助函数构建页面列表。

---

## 8. 内嵌组件 / Embedded Components (GetComponents)

```lua
function GetComponents(props)
  return {
    { Name = "MainMixer", Type = "mixer", Properties = { ["n_inputs"] = 8, ["n_outputs"] = 1 } },
    { Name = "SignalDet", Type = "signal_presence", Properties = { ["multi_channel_type"] = 1 } }
  }
end
```
- 组件名不能含空格
- 运行时通过 `Components["Name"]` 访问
- 音频组件有使用上限（如 Audio Player 最大 32 通道）
- 可用 QDS 中 Tools > View Component Controls Info 查看组件引脚名

### GetPins (仅音频/串口引脚)
```lua
function GetPins(props)
  return {
    { Name = "Audio In",  Direction = "input" },
    { Name = "Audio Out", Direction = "output" },
    { Name = "Serial",    Direction = "input",  Domain = "serial" }
  }
end
```
> **注意**: 控制引脚由控件的 `UserPin=true` 生成，**不是**通过 GetPins！

### GetWiring (内嵌组件连线)
```lua
function GetWiring(props)
  return {
    { "In 1", "MainMixer Input 1" },        -- 插件输入 → 组件输入
    { "MainMixer Output 1", "Mix Output" }  -- 组件输出 → 插件输出
  }
end
```
- 插件输入可连多个组件输入
- 组件输入只能连一个插件输入
- 组件输出可连多个插件输出
- 插件输出只能连一个组件输出
- **未引用的组件引脚不会有音频通过**

---

## 9. 运行时 / Runtime

### 控件访问 / Controls Access
```lua
-- 单个控件
Controls["MyControl"].Value = 0.8
Controls.MyControl.Boolean = true

-- Count > 1 的控件数组 (1-indexed)
Controls["MyControl"][1].Value = 0.8

-- 别名
local myAlias = Controls["MyControl"]
myAlias.Value = 0.8
```

### 控件值属性 / Control Value Properties
| 控件类型 | 属性 |
|----------|------|
| Button | `.Boolean` (true/false), `.Value` (1/0) |
| Knob | `.Value` (数值) |
| Indicator | `.Value` (Led: 1/0, Meter: 数值) |
| Text | `.String` (文本) |

### EventHandler
```lua
Controls["MyButton"].EventHandler = function(c)
  -- c.Boolean, c.Value, c.String 等
end
```

### 推荐运行时函数 / Recommended Runtime Functions
| 函数 | 用途 |
|------|------|
| `Initialization()` | 插件启动时调用一次 |
| `SetupDebugPrint()` | 根据 Debug 属性设置打印级别 |
| `Send(cmd)` | 通过 socket 发送数据 |
| `ParseResponse()` | 处理 socket 接收数据 |
| `Connect()` | 连接建立后设置标志 |
| `Disconnected()` | 连接断开后重置标志 |
| `ClearVariables()` | 连接中断时清空控件/变量/表 |
| `GetDeviceInfo()` | 请求静态数据（序列号等） |
| `PollDevice()` | 定期轮询保持连接健康 |

---

## 10. Plugin Compiler / 编译工具 (官方)

### 9 文件框架
| 文件 | 职责 |
|------|------|
| `plugin.lua` | 主骨架，`--[[ #include "xxx.lua" ]]` 引入 |
| `info.lua` | PluginInfo |
| `properties.lua` | GetProperties + RectifyProperties |
| `controls.lua` | GetControls |
| `layout.lua` | GetControlLayout |
| `pages.lua` | GetPages |
| `model.lua` | GetComponents + GetPins + GetWiring + GetColor + GetPrettyName |
| `runtime.lua` | if Controls then ... end |
| `rectify_properties.lua` | RectifyProperties |

### 功能
- Image Encoder: PNG/JPG/SVG → base64
- GUID Generator
- Version Control: 每次构建自增版本
- Copy Qplug: 自动复制到 Plugins 目录
- 要求: VS Code + Git for Windows
- 快捷键: Ctrl+Shift+B (Run Build Task)
- `.qplug` 关联 Lua: VS Code Settings → files.associations → `*.qplug`: `lua`

---

## 11. 实践验证陷阱 / Verified Pitfalls

| 问题 | 原因 | 解决 |
|------|------|------|
| Trigger 输入无反应 | `ButtonType="Trigger"` 的 EventHandler 中 `c.Boolean=false` | 无条件执行，不判断 Boolean |
| Knob 无外部引脚 | 未设 `ControlUnit` | 必须设 `ControlUnit`（如 "Integer"） |
| `ButtonStyle` 无引脚 | ButtonStyle 只影响面板外观 | 用 `ButtonType` 控制引脚类型 |
| `GetAllControlPanels: <name> not defined` | layout 引用了 GetControls 未定义的控件名 | 检查 layout key 与 controls Name 完全一致（含 Count 时的 `"Name 1"` 格式） |
| `unexpected symbol near ')'` | `layout["x"] = { ... })` 多了 `)` | 赋值用 `}`，函数调用才用 `})` |
| 修改 count 后引脚不变 | 组件未重建 | 删除旧组件重新拖入 |
| Core 上定时器不准 | `os.clock()` 是 CPU 时间 | 看门狗用 `os.time()` |
| 面板元素超框 | GroupBox/面板尺寸不够 | 画大留足边距，宁大勿小 |
| TCP 假连接 | 连接成功但无数据 | 两级看门狗：无数据超时 + 注册后无信号超时 |
| Count>1 layout 名错误 | 用了 `"Name1"` 而非 `"Name 1"` | 控件名 + 空格 + 序号 |

---

## 12. wxl_personal_plug 系列规范 / House Standards

- 分类名: `wxl_personal_plug~PluginName`
- 作者: `longwang`
- 面板文字: 全英文
- Notes: 中英文双语，放在面板底部
- 每个插件底部必须有 Notes 说明区域
- GitHub: 每个模块独立仓库，README 中英文双语
- 坐标: 面板宽建议 340px，所有框留足边距（上下左右 ≥8px，元素间距 ≥4px）
- 输入触发按钮: Toggle+自动复位（点一下触发一次）
