# crestronCIP / Crestron CIP 协议网关

**Version / 版本**: 1.0.0
**Author / 作者**: longwang
**Category / 分类**: `wxl_personal_plug~crestronCIP`

Crestron CIP (Control over IP) protocol gateway over TCP (port 41794). Bidirectional Digital/Analog/Serial signal exchange with Crestron control systems. Configurable IP, IPID, and signal counts.

Crestron CIP 协议网关，通过 TCP（端口41794）与 Crestron 控制系统双向交换数字量/模拟量/串量信号。可配置 IP、IPID 和信号数量。

---

## Features / 功能特性

- **Configurable Signal Counts / 信号数量可配**: Digital, Analog, Serial each 0-500 (default 32), 50 signals per page
  数字量、模拟量、串量各0-500（默认32），每页50个信号
- **Configurable IP & IPID / IP和IPID可配**: Default in properties, override on Connection page panel
  属性中设默认值，连接页面板可覆盖修改
- **Bidirectional Signals / 双向信号**: Digital_In/Out, Analog_In/Out, Serial_In/Out — all exposed as control pins for wiring
  所有信号都暴露为控制引脚可接线
- **Analog Range / 模拟量范围**: 0-65535 (16-bit unsigned integer)
  0-65535（16位无符号整数）
- **Auto Registration / 自动注册**: Sends IPID registration packet on connect, handles sync completion handshake
  连接时自动发送IPID注册包，处理同步完成握手
- **Heartbeat / 心跳**: 15-second heartbeat to maintain connection
  每15秒心跳维持连接
- **Sticky Packet Handling / 粘包处理**: 50ms poll timer with buffer reassembly for fragmented CIP packets
  50ms轮询定时器，带缓冲区重组处理CIP粘包
- **Debug Log Server / 调试日志服务器**: TCP client mode to stream all protocol logs to remote log server for analysis
  TCP客户端模式，将所有协议日志发送到远程日志服务器进行分析
- **Hex Packet Logging / 十六进制报文日志**: All non-heartbeat packets logged in hex for debugging
  所有非心跳报文以十六进制记录便于调试

---

## Pages / 页面

| Page / 页面 | Content / 内容 |
|---|---|
| **Connection** | IP/IPID config, connect button, status, signal count info, debug log server config 连接配置、连接按钮、状态、信号数量信息、调试日志服务器配置 |
| **Digital 1-50, 51-100, ...** | Digital_In and Digital_Out toggles with number labels 数字量输入输出开关，带编号标签 |
| **Analog 1-50, 51-100, ...** | Analog_In and Analog_Out faders (0-65535) with number labels 模拟量输入输出滑条，带编号标签 |
| **Serial 1-50, 51-100, ...** | Serial_In and Serial_Out text fields with number labels 串量输入输出文本框，带编号标签 |

Pages are dynamically generated based on configured signal counts (50 per page).
页面根据配置的信号数量动态生成（每页50个）。

---

## Input Pins / 输入引脚

### Connection / 连接

| Pin Name / 引脚名 | Type / 类型 | Description / 说明 |
|---|---|---|
| `CIP_Connect` | Toggle (Input) | Connect/disconnect to Crestron CIP server. ON = connect, OFF = disconnect. 连接/断开Crestron CIP服务器 |

### Signals to Crestron / 发送到Crestron的信号

| Pin Name Pattern / 引脚名格式 | Type / 类型 | Count / 数量 | Description / 说明 |
|---|---|---|---|
| `Digital_Out_{1..N}` | Toggle (Input) | `digital_count` | Digital signals sent TO Crestron. ON = join press, OFF = join release. 发送给Crestron的数字量 |
| `Analog_Out_{1..N}` | Integer Knob (Input) | `analog_count` | Analog signals sent TO Crestron. Range 0-65535. 发送给Crestron的模拟量 |
| `Serial_Out_{1..N}` | Text Edit (Input) | `serial_count` | Serial/string signals sent TO Crestron. 发送给Crestron的串量 |

### Debug / 调试

| Pin Name / 引脚名 | Type / 类型 | Description / 说明 |
|---|---|---|
| `Debug_Connect` | Toggle (Input) | Connect/disconnect to remote debug log server. 连接/断开远程调试日志服务器 |

---

## Output Pins / 输出引脚

### Connection / 连接

| Pin Name / 引脚名 | Type / 类型 | Description / 说明 |
|---|---|---|
| `CIP_Status` | Toggle (Output) | Connection status. ON = connected and IPID registered. 连接状态，ON=已连接并注册 |

### Signals from Crestron / 从Crestron接收的信号

| Pin Name Pattern / 引脚名格式 | Type / 类型 | Count / 数量 | Description / 说明 |
|---|---|---|---|
| `Digital_In_{1..N}` | Toggle (Output) | `digital_count` | Digital signals received FROM Crestron. 从Crestron接收的数字量 |
| `Analog_In_{1..N}` | Integer Knob (Output) | `analog_count` | Analog signals received FROM Crestron. Range 0-65535. 从Crestron接收的模拟量 |
| `Serial_In_{1..N}` | Text Display (Output) | `serial_count` | Serial/string signals received FROM Crestron. 从Crestron接收的串量 |

### Debug / 调试

| Pin Name / 引脚名 | Type / 类型 | Description / 说明 |
|---|---|---|
| `Debug_Status` | Toggle (Output) | Debug log server connection status. 调试日志服务器连接状态 |

---

## Panel-Only Controls (No Pins) / 面板专用控件（无引脚）

| Control / 控件 | Type / 类型 | Description / 说明 |
|---|---|---|
| `IP_Input` | Text Edit | Crestron processor IP address (overrides property default). Crestron处理器IP（覆盖属性默认值） |
| `IPID_Input` | Text Edit | CIP IPID in hex (e.g. "03", "0A", "FF"). Overrides property default. CIP设备ID（十六进制） |
| `Debug_Server_IP` | Text Edit | Remote debug log server IP. 远程调试日志服务器IP |
| `Debug_Server_Port` | Text Edit | Remote debug log server port (default 10001). 远程调试日志服务器端口 |

---

## Properties / 属性参数

| Property / 属性 | Type / 类型 | Default / 默认 | Range / 范围 | Description / 说明 |
|---|---|---|---|---|
| `cip_ip` | String | 192.168.1.100 | — | Default Crestron processor IP address (panel IP_Input overrides). 默认Crestron处理器IP |
| `cip_ipid` | String | "03" | "03"-"FF" | CIP IPID in hex string. CIP设备ID（十六进制字符串） |
| `digital_count` | Double | 32 | 0-500 | Number of digital joins. 数字量数量 |
| `analog_count` | Double | 32 | 0-500 | Number of analog joins. 模拟量数量 |
| `serial_count` | Double | 32 | 0-500 | Number of serial joins. 串量数量 |
| `debug_server_ip` | String | 192.168.1.100 | — | Default remote debug log server IP. 默认远程调试日志服务器IP |
| `debug_server_port` | Double | 10001 | 1-65535 | Default remote debug log server port. 默认远程调试日志服务器端口 |

---

## CIP Protocol Reference / CIP 协议参考

### Connection / 连接

| Item / 项目 | Value / 值 |
|---|---|
| TCP Port / 端口 | 41794 |
| Heartbeat interval / 心跳间隔 | 15 seconds |
| Poll interval / 轮询间隔 | 50ms |

### Packet Types / 报文类型

| Type / 类型 | Hex / 十六进制 | Description / 说明 |
|---|---|---|
| Registration / 注册 | `01` | IPID registration request (client → server) IPID注册请求 |
| Registration Response / 注册响应 | `02` | IPID registration response (server → client) IPID注册响应 |
| Data / 数据 | `05` | Digital/Analog data, sync completion 数字/模拟数据、同步完成 |
| Serial / 串量 | `12` | Serial/string data 串量数据 |
| Heartbeat / 心跳 | `0D` | Heartbeat request (client → server) 心跳请求 |
| Heartbeat Response / 心跳响应 | `0E` | Heartbeat response (server → client) 心跳响应 |

### Registration Packet / 注册包

```
01 00 0B 00 00 00 00 00 <IPID> 40 FF FF F1 01
```

### Registration Success Response / 注册成功响应

```
02 00 04 00 00 00 1F
```

### Digital Join / 数字量

```
05 00 06 00 00 03 00 <joinLow> <joinHigh+flag>
```
- flag `0x00` = press (ON), flag `0x80` = release (OFF)
- join number = (joinHigh & 0x7F) * 256 + joinLow + 1

### Analog Join / 模拟量

```
05 00 08 00 00 05 14 <joinHigh> <joinLow> <valueHigh> <valueLow>
```
- value = valueHigh * 256 + valueLow (0-65535)

### Serial Join / 串量

```
12 <lenHigh> <lenLow> 00 00 00 <payloadLen> 34 <joinHigh> <joinLow> 03 <string...>
```

### Sync Completion / 同步完成

```
Server → Client: 05 00 05 00 00 02 03 1C
Client → Server: 05 00 05 00 00 02 03 1D
```

---

## Usage / 使用方法

1. Set the Crestron processor IP and IPID in Properties (or override on Connection page).
   在属性中设置Crestron处理器IP和IPID（或在连接页面覆盖）。
2. Set `digital_count`, `analog_count`, `serial_count` to match your Crestron program join counts.
   设置数字量/模拟量/串量数量，与Crestron程序中的join数量匹配。
3. Toggle `CIP_Connect` ON to establish TCP connection (port 41794) and register IPID.
   打开 `CIP_Connect` 建立TCP连接（端口41794）并注册IPID。
4. `CIP_Status` turns ON when connected and registered successfully.
   连接并注册成功后 `CIP_Status` 变为ON。
5. Wire Q-SYS controls to `Digital_Out`, `Analog_Out`, `Serial_Out` pins to send data to Crestron.
   连接Q-SYS控件到输出引脚，发送数据到Crestron。
6. Wire `Digital_In`, `Analog_In`, `Serial_In` pins to Q-SYS controls to receive data from Crestron.
   连接输入引脚到Q-SYS控件，接收Crestron数据。
7. **Debugging / 调试**: Configure `debug_server_ip` and `debug_server_port`, then toggle `Debug_Connect` ON to stream all CIP packet logs (hex) to your TCP client for protocol analysis.
   调试：配置日志服务器IP和端口，打开 `Debug_Connect`，将所有CIP报文日志（十六进制）发送到TCP客户端进行协议分析。

---

## Signal Direction Reference / 信号方向参考

| Q-SYS Control / Q-SYS控件 | Pin Style / 引脚方向 | Direction / 方向 |
|---|---|---|
| `Digital_Out_{N}` | Input | Q-SYS → Crestron (Q-SYS sends) |
| `Digital_In_{N}` | Output | Crestron → Q-SYS (Q-SYS receives) |
| `Analog_Out_{N}` | Input | Q-SYS → Crestron |
| `Analog_In_{N}` | Output | Crestron → Q-SYS |
| `Serial_Out_{N}` | Input | Q-SYS → Crestron |
| `Serial_In_{N}` | Output | Crestron → Q-SYS |

**Note / 注意**: "In/Out" naming is from Crestron's perspective: `Digital_In` = signal coming IN to Q-SYS FROM Crestron.
命名以Crestron视角为准：`Digital_In` = 从Crestron进入Q-SYS的信号。

---

## Notes / 注意事项

- IPID must be in hex format (03-FF). Decimal 3 = hex "03", decimal 10 = hex "0A", decimal 255 = hex "FF".
  IPID必须为十六进制格式（03-FF）。
- Changing signal counts in Properties requires re-adding the component to schematic (pin count changes).
  修改信号数量后需要重新拖入组件（引脚数量变化）。
- Serial strings received from Crestron have control characters and null bytes stripped automatically.
  从Crestron接收的串量会自动去除控制字符和空字节。
- All non-heartbeat received packets are logged in hex via print() and debug server — useful for protocol reverse engineering.
  所有非心跳接收报文以十六进制记录，便于协议分析。
- Connection drops trigger auto-cleanup; toggle CIP_Connect OFF then ON to reconnect.
  连接断开会自动清理；切换 CIP_Connect  OFF→ON 重新连接。
