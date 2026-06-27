# 并发规则与依赖约束

## 并发分类定义

各工具的并发标注见 [tool-reference.md](tool-reference.md)。

**核心模型：通道（lane）**。同一设备内部有 4 条互相独立的物理/逻辑通道，
每条通道内部必须串行，**不同通道之间可以并行**。标签即通道名：

| 标签 | 通道 / 共享资源 | 包含工具 | AI 行为指导 |
|------|----------------|----------|------------|
| **serialized:gdb** | GDB 调试会话 | gdb_connect/disconnect 及所有 gdb_* 操作（执行控制: gdb_halt/resume/reset；查询: gdb_read_reg/mem/cpuid；断点: gdb_set/del；烧写: gdb_load/flash_*；命令: gdb_command/monitor） | 同一设备的 gdb_* 之间串行 |
| **serialized:ocd** | OpenOCD 实例 | ocd_*、openocd_*、chip_detect | 同一设备的本组工具之间串行 |
| **serialized:serial** | UART 串口 TCP 桥（按 index 独立） | serial_open/close/bind/unbind/write/read | 同一 index 的 serial_* 之间串行 |
| **serialized:rtt** | RTT 通道 TCP 桥（按 channel 独立） | rtt_start/list/bind/unbind/write/read | 同一 channel 的 rtt_* 之间串行 |
| **parallel-safe** | 无共享资源 | list_devices、get_full_status、chip_search/get、elf 查询、svd 元数据查询、serial_clear/rtt_clear 等 | 可同时发起多个调用 |
| **serialized-global** | 进程级全局状态 | elf_load、svd_set_chip | 同一时间只能有一个在执行 |

### 关键区分

- **不同通道可并行**：例如 serial_*（serialized:serial）与任意调试通道完全物理隔离，可放心并行；
  gdb_*（serialized:gdb）与 ocd_*（serialized:ocd）走不同通道，**可以并发发起**。
- **但三条调试通道共享目标硬件的 halt 状态**：serialized:gdb / serialized:ocd / serialized:rtt
  虽可并发发起，却操作同一颗芯片的同一个 halt 点，**无一致性保证**——
  并行读写寄存器/内存时结果可能交错，需要一致快照时仍应自行串行。
- `serial_write` / `serial_read` 读写分离，同一 index 内可并行；

> **执行控制规则**：gdb_connect 之后必须用 gdb_halt/gdb_resume/gdb_reset，禁止混用 ocd_halt/ocd_resume/ocd_reset。仅在未连接 GDB 或 gdb_disconnect 之后使用 ocd_* 执行控制。

## 常见陷阱详解

### 陷阱 1：gdb_disconnect 后 MCU 状态不确定

断开 GDB 连接后，MCU 可能 halt 也可能继续运行，状态不确定。
**恢复方法**：disconnect 后调用 `ocd_resume` 确保 MCU 运行。

### 陷阱 2：read 轮询前应 clear 历史数据

使用 `serial_read`/`rtt_read` 轮询时，缓冲区中可能残留历史数据。
**正确做法**：
- 命令-响应（write → read）：`clear → write → read`，先清空历史数据再发送命令
- 被动监听（仅 read）：`clear → 循环 read`
持续监听场景应使用 `bind_port` TCP 直连。

### 陷阱 3：gdb_connect 后使用 ocd_halt/ocd_resume/ocd_reset

GDB 连接后，执行控制必须走 gdb_halt/gdb_resume/gdb_reset。ocd_* 执行控制走不同的路径，混用会导致 GDB 的状态跟踪与实际 CPU 状态不同步。
**正确做法**：gdb_connect 后用 gdb_halt；gdb_disconnect 后才用 ocd_halt/ocd_resume/ocd_reset。

### 陷阱 4：gdb_flash_write 必须在 erase 之后

`gdb_flash_write` 不会自动擦除，如果 flash 区域有旧数据会写入失败。
**正确做法**：先 `gdb_flash_erase`，或直接用 `gdb_load`（自动处理）。

### 陷阱 5：RTT unbind 在 openocd_stop 后会失败

RTT 依赖 OpenOCD 会话。`openocd_stop` 后 RTT 资源已释放，`rtt_unbind_channel` 会报 `channelNotFound`。
**正确做法**：在 `openocd_stop` 前 unbind RTT；或忽略 unbind 错误（资源已释放）。

### 陷阱 6：多设备时必须指定 sn

当多个设备在线时，不指定 `sn` 参数会返回 `MultipleDevices` 错误。
**正确做法**：先 `list_devices`，再 `set_active_device` 或在每个调用中指定 `sn`。

### 陷阱 7：gdb_load 超时导致连接锁死

`gdb_load` 的 erase/write 操作如果超时，连接会进入不一致状态。
后续所有 `gdb_*` 调用都会卡死，因为 OpenOCD 侧还在等待上一个命令完成。
**正确做法**：
1. `gdb_disconnect(sn)` — 断开旧连接
2. `gdb_connect(sn)` — 重连（幂等，会自动清除残留后端）
3. 用更长的超时重试：`gdb_load(sn, path, addr, erase_timeout_ms=30000, write_timeout_ms=60000)`

| 文件大小 | 建议 write_timeout_ms |
|----------|----------------------|
| ≤512KB   | 默认 60000 |
| 1MB      | 90000 |
| 2MB      | 120000 |

### 陷阱 8：读寄存器需要 CPU halt

`gdb_read_reg` 在 CPU 运行时返回错误。如果需要读寄存器，先 `gdb_halt`。
`gdb_read_mem` 不需要 halt。**但读-改-写外设寄存器序列（read 后 write）应在 halt 下完成**，否则两次内存访问间固件可能并发修改同一寄存器。

### 陷阱 9：rtt_start 需要 RTT 控制块已初始化

固件未运行到 RTT 初始化时扫描会失败。
**正确做法**：gdb_load → gdb_reset(run) → rtt_start
**禁止**：gdb_load 与 rtt_start 并行

### 陷阱 10：gdb_write_reg 需要 CPU halt

`gdb_write_reg` 与 `gdb_read_reg` 一样需要 CPU halt。
**正确做法**：先 `gdb_halt`，再 `gdb_write_reg`。

### 陷阱 11：异常 handler 中写寄存器可能无效

在 HardFault 等异常 handler 中，通用寄存器（r0-r12）的原始值保存在异常栈帧上，直接写寄存器不影响返回后的状态。
**注意**：如需修改 fault 前的寄存器值，需修改栈上的保存值。