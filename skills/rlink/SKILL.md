---
name: rlink
description: 嵌入式设备调试工具。调试芯片/单片机、烧写固件、读写寄存器/内存、串口通信、RTT 日志、断点调试时使用。
---

# RLink 调试工具指南

## 工具调用方式

所有工具通过 `rlink_invoke` 调用：

```json
{ "tool": "<工具名>", "args": { <参数对象> } }
```

示例：
```json
{ "tool": "list_devices", "args": {} }
{ "tool": "get_full_status", "args": {} }
{ "tool": "gdb_load", "args": { "path": "firmware.elf" } }
```

## 工作流路由

根据用户意图选择对应工作流文档：

- **首次连接设备、启动调试环境** → [workflow-setup.md](reference/workflow-setup.md)
- **halt/step/读寄存器/读内存/断点/单步调试** → [workflow-debug.md](reference/workflow-debug.md)
- **烧写固件 (.bin/.hex/.elf)** → [workflow-flash.md](reference/workflow-flash.md)
- **串口通信或 RTT 日志** → [workflow-serial-rtt.md](reference/workflow-serial-rtt.md)
- **ELF 符号解析、SVD 寄存器解码** → [workflow-elf-svd.md](reference/workflow-elf-svd.md)
- **识别未知芯片（CPUID/IDCODE）** → [workflow-chip-id.md](reference/workflow-chip-id.md)
- **并发规则、依赖链、常见陷阱** → [constraints.md](reference/constraints.md)

## 详细参数参考

仅当参数不确定时，查阅对应 category 的 schema 文件：

- **device** → [schema-device.md](reference/schema-device.md)
- **chip** → [schema-chip.md](reference/schema-chip.md)
- **openocd** → [schema-openocd.md](reference/schema-openocd.md)
- **ocd** → [schema-ocd.md](reference/schema-ocd.md)
- **gdb** → [schema-gdb.md](reference/schema-gdb.md)
- **elf** → [schema-elf.md](reference/schema-elf.md)
- **svd** → [schema-svd.md](reference/schema-svd.md)
- **serial** → [schema-serial.md](reference/schema-serial.md)
- **rtt** → [schema-rtt.md](reference/schema-rtt.md)

## 首次使用流程

1. `list_devices` — 确认设备在线
2. `get_full_status` — 查看状态，按 hints 执行。设备标识优先用 `deviceName`，无则用 `sn`

## 常见陷阱

详见 [constraints.md](reference/constraints.md)。关键规则：
- **gdb_connect 后必须用 gdb_halt/gdb_resume/gdb_reset，禁止混用 ocd_***
- **gdb_disconnect 后调用 ocd_resume 恢复 MCU 运行**