# 调试工作流

## 前置条件
- OpenOCD 已启动（openocd_status.running = true）
- GDB 已连接（get_full_status.gdb_connected = true）

## 停住 CPU
两种方式：
- `gdb_halt(sn)` — 通过 GDB halt（gdb_connect 之后必须用此工具，返回 StopInfo）
- `gdb_wait_stop(sn, timeout_ms?)` — 等待 CPU 自然停止（用于断点命中等待）

> **注意**：gdb_connect 之后禁止使用 ocd_halt/ocd_resume/ocd_reset。GDB 和 OCD 走不同的控制路径，混用会导致 GDB 状态不同步。仅在未连接 GDB 或 gdb_disconnect 之后使用 ocd_* 执行控制。

## 读取状态

### 寄存器
- `gdb_read_reg(sn, decode?)` — 读所有 CPU 寄存器
  - **CPU 必须 halt**，运行时调用会返回错误
  - decode=true 时额外返回 xPSR 标志解码（N/Z/C/V/Q/ISR）
  - 返回 [{name, value, decoded}]

### 内存
- `gdb_read_mem(sn, addr, len, fmt?, decode_as?)` — 读内存
  - addr: "0x" 格式十六进制地址
  - len: 字节数
  - fmt: "u8"/"u16"/"u32"/"u64"（默认 u32）
  - decode_as: "float"/"int16"/"uint16"/"int32" 重解释

### 全局变量（不需要 halt，不需要 GDB 连接）
- `gdb_read_var(sn, name)` — 通过 ELF 符号表读取全局变量，不需 halt
  - 前提：elf_load 已调用、OpenOCD 已启动

## 写入操作

### 写寄存器
- `gdb_write_reg(sn, name, value)` — 写单个寄存器
  - name: "pc", "r0", "sp" 等

### 写内存
- `gdb_write_mem(sn, addr, data)` — 写字节数组
  - data: [0,1,2,3] 格式

## 断点调试

### 软件断点
1. `gdb_set_breakpoint(sn, location)` — 设置断点
   - location: "0xADDR"、符号名 "main"、或 "file:line"
   - 符号名需要 ELF 已加载
   - 返回 breakpoint ID
2. `gdb_resume(sn)` — 恢复运行
3. `gdb_wait_stop(sn)` — 等待断点命中（返回 StopInfo）
4. 检查状态：`gdb_read_reg` / `gdb_read_mem`
5. `gdb_del_breakpoint(sn, id)` — 清理断点

### 数据观察点
1. `gdb_set_watchpoint(sn, addr, kind?)` — 设置观察点
   - addr: "0x" 格式
   - kind: "read"/"write"/"access"（默认 write）
   - 返回 watchpoint ID
2. `gdb_resume(sn)` — 恢复运行
3. `gdb_wait_stop(sn)` — 等待触发

### 查看所有断点
- `gdb_list_breakpoints(sn)` — 列出所有活跃断点和观察点

### 删除所有断点
- `gdb_del_all_breakpoints(sn)` — 一次删除所有断点和观察点，返回 `{deleted: N}`

## 单步调试
- `gdb_step(sn)` — 单步一条指令（CPU 必须 halt）
- `gdb_step_out(sn)` — 跳出当前函数
  - CPU 必须 halt
  - 等待 CPU 再次 halt（最多 10s），超时返回 `{stopped: false, reason: timeout}`

## 恢复执行
- `gdb_resume(sn)` — 通过 GDB 恢复运行（gdb_connect 后使用）

## 故障分析
- `ocd_fault_analysis(sn)` — Cortex-M 故障分析
  - 读取 CFSR/HFSR/MMFAR/BFAR
  - 解析 fault PC 和 SP
  - 如果有 ELF，解析 fault_pc 到源码位置
  - 返回结构化故障报告

## 芯片识别（需要 OpenOCD 运行，不需要 GDB 连接）
- `ocd_read_cpuid(sn)` — 读 CPU 核心类型，解码 implementer/part_no/variant/revision
- `ocd_read_idcode(sn)` — 识别芯片型号（含 CPUID），返回 family/dev_id/rev_id

## 通用命令
- `gdb_command(sn, cmd)` — 执行类 GDB 命令
  - "break main" → 符号断点
  - "print var" → 读变量
  - "info reg" → 寄存器
  - "x/10xw 0x20000000" → 检查内存
  - "bt" → 调用栈
  - "monitor cmd" → OpenOCD 直通（等同 gdb_monitor）
- `gdb_monitor(sn, cmd)` — 发送原始命令到 OpenOCD 服务器。需要 GDB 连接。
  - 例如 "reset halt", "reg", "flash banks"

## 标准调试流程示例

```
用户：程序卡住了，帮我看看

步骤：
1. get_full_status()           → 确认 gdb_connected=true
2. ocd_get_target_arch()       → 确认架构
3. 如果是 Cortex-M：ocd_fault_analysis() → 自动判断是否需要 halt，分析 HardFault（含 PC/SP/CFSR/HFSR）
4. [分析结果，告诉用户问题所在]
5. gdb_resume()                → 恢复运行（如果需要）
```

## 断点调试示例

```
用户：在 main 函数设断点，单步调试

步骤：
1. elf_load(path="build/app.elf")  → 加载符号
2. gdb_set_breakpoint(location="main")  → 设断点，返回 id=1
3. gdb_reset(mode="halt")  → 复位并 halt
4. gdb_resume()  → 运行
5. gdb_wait_stop(timeout_ms=5000)  → 等待命中 main 断点
6. gdb_read_reg(decode:true)  → 查看寄存器状态
7. gdb_step()  → 单步
8. gdb_read_reg()  → 查看变化
9. gdb_step_out()  → 跳出当前函数（等待再次 halt）
10. gdb_del_all_breakpoints()   → 清理所有断点
```