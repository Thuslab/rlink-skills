# 设备发现与调试环境建立

本文件描述嵌入式设备调试的标准 setup 流程：从设备发现到 OpenOCD + GDB 全链路建立。

---

## 第一步：设备发现

- `list_devices()` — 列出所有在线设备序列号
- 单设备时后续调用可省略 `sn` 参数（自动解析）
- 多设备时用 `set_active_device(sn)` 切换，或在后续所有调用中传 `sn` 参数

### 多设备处理规则

当 `list_devices()` 返回多个设备时：
1. 列出所有设备序列号，询问用户选择
2. 可提供"自动探测芯片型号"选项辅助用户决策
3. 如果用户选择探测，对每个设备调用 `chip_detect(sn)` 并展示结果
4. **探测后仍需用户选择**，禁止根据探测结果自动选择设备
5. 最终选择必须由用户确认

> 禁止：自动选择设备、探测后自动匹配使用。

> 提示：大多数情况下只需一台设备，`list_devices()` 确认设备在线即可，不需要显式切换。

---

## 第二步：查看当前状态

- `get_full_status(sn?)` — 查看当前设备状态
- 返回的 `hints` 数组会告诉你下一步该做什么
- 如果 `gdb_connected: true` 且 OpenOCD running，环境已就绪，跳过后续步骤

```json
// 示例返回（环境未就绪）
{
  "sn": "1002474828",
  "gdb_connected": false,
  "device": {
    "deviceName": "MySTM32",
    "openocd": { "running": false },
    "serial": [],
    "rtt_channels": []
  },
  "hints": [
    "OpenOCD not running. Call openocd_start first.",
    "GDB not connected. Call gdb_connect after openocd_start."
  ]
}
// hints 非空 → 按提示执行下一步
```

---

## 第三步：芯片识别

两种方式，按实际情况选择：

- **知道芯片型号**：直接 `chip_search(query)` 查芯片数据库
  - 支持模糊搜索，如 `"STM32F4"` 会匹配整个 F4 系列
  - 返回结果包含精确的芯片名称，用于后续 OpenOCD 启动
- **不知道芯片型号**：`chip_detect(sn?)` 自动探测（通过 JTAG/SWD IDCODE）
  - 会临时启动 OpenOCD 进行探测，需要设备在线
  - 返回候选芯片列表（按置信度排序）
- `chip_get(name)` 获取芯片详细信息（需要chip_search返回的精确芯片型号，返回flash 地址、RAM 地址等）

---

## 第四步：启动 OpenOCD

- `openocd_start(sn?, chip, transport?, speed?)` — 启动调试服务器
  - `chip`：从 chip_search 结果中获取的精确名称
  - `transport`：`"swd"`（默认）或 `"jtag"`
  - `speed`：kHz，`0` = 自动（默认）
  - 成功即表示 OpenOCD 已运行；失败返回错误
- 注意：如果 OpenOCD 已被其他主机占用，`start` 会接管所有权

---

## 第五步：连接 GDB

- `gdb_connect(sn?, port?)` — 建立 GDB 连接
  - `port`：默认从 OpenOCD 状态获取，一般不需要指定
- 前提：OpenOCD 必须 running 且 `is_owner=true`，可通过openocd_status(sn?)获取状态信息

连接成功后 MCU 进入 halt 状态，可进行寄存器读写、flash 烧录等操作。

---

## 完整示例

```
用户：帮我连接 STM32F407VGT6 开发板

步骤：
1. list_devices()                          → { devices: ["12345678"], count: 1 }
2. get_full_status(sn="12345678")          → { gdb_connected: false, hints: ["OpenOCD not running..."] }
3. chip_search("STM32F407")                → { results: [{ name: "STM32F407VGTx", ... }] }
4. openocd_start(sn="12345678", chip="STM32F407VGTx") → { ok: true, ... }
5. gdb_connect(sn="12345678")              → { connected: true, ... }
6. get_full_status(sn="12345678")          → { gdb_connected: true, hints: [] }
```

---

## 关闭环境

当调试结束时，按此顺序关闭：

1. `rtt_unbind_channel(sn, channel)` — 解绑 RTT 通道（如有，需在 openocd_stop 前）
2. `gdb_del_all_breakpoints(sn)` — 清理所有断点（避免残留）
3. `gdb_disconnect(sn)` — 断开 GDB（MCU 状态不确定）
4. `ocd_resume(sn)` — 确保 MCU 运行
5. `openocd_stop(sn)` — 停止 OpenOCD

> **注意**：
> - gdb_disconnect 后 MCU 状态不确定，之后应调用 ocd_resume 确保运行
> - RTT unbind 若在 openocd_stop 后调用会报 channelNotFound，可忽略
> - Serial 与 OpenOCD 独立，serial_close 可在任意时机调用
