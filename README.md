# RLink Skills

RLink 嵌入式调试工具的 Agent Skill，通过 RLink MCP 协议提供芯片调试、固件烧写、寄存器访问、串口通信和 RTT 日志能力。

## 功能

- 芯片识别与连接管理
- GDB 调试（断点、单步、寄存器/内存读写）
- 固件烧写（.bin/.hex/.elf）
- UART 串口通信
- RTT 实时日志
- ELF 符号解析与 SVD 寄存器解码

## 结构

```
skills/rlink/
├── SKILL.md              # 入口：工作流路由与快速参考
└── reference/
    ├── workflow-*.md     # 场景化工作流（setup/debug/flash/serial-rtt/elf-svd/chip-id）
    ├── schema-*.md       # 参数参考（device/chip/openocd/gdb/elf/svd/serial/rtt）
    └── constraints.md    # 并发规则与常见陷阱
```

## 使用

1. `list_devices` → 确认设备
2. `get_full_status` → 查看状态
3. 按需进入具体工作流（调试 / 烧写 / 串口 / RTT）

## 许可证

MIT