# ELF 符号解析与 SVD 寄存器解码

## ELF 符号解析 (elf_*)

### 加载 ELF
- `elf_load(path)` — 加载 ELF/AXF 文件
  - 设为默认 ELF（后续 elf_* 自动使用）
  - 返回：架构、入口地址、符号数量
  - **serialized-global**：同一时间只能加载一个

### 查询符号
- `elf_symbols(elf_path?, filter?, limit?)` — 查询符号表
  - filter: 名称子串过滤
  - limit: 最大结果数（默认 100，最大 1000）
  - 返回 [{name, address, size, type}]
  - **parallel-safe**：可并行调用

### 地址解析
- `elf_addr2line(elf_path?, addresses[])` — 地址 → 源码位置
  - addresses: 虚拟地址数组
  - 返回 [{address, file, line, function}]
  - 需要 ELF 有 debug sections
  - **parallel-safe**：可并行调用

### 反汇编
- `elf_disasm(elf_path?, address, count?)` — 从地址开始反汇编
  - count: 指令数（默认 20，最大 200）
  - 返回 [{address, mnemonic, operands, bytes, symbol?}]
  - **parallel-safe**：可并行调用

### 典型用法

```
场景：crash 后分析 fault PC

1. elf_load(path="build/app.elf")       → 加载符号
2. openocd_start(chip="STM32F407VGTx")  → 启动 OpenOCD
3. ocd_fault_analysis()                 → 自动 halt，返回故障报告（含 fault_pc 及其源码位置）
4. elf_disasm(address=0x08001234, count=10)  → 反汇编上下文
```

```
场景：查找某个函数的地址并设断点

1. elf_load(path="build/app.elf")
2. elf_symbols(filter="main")   → [{name: "main", address: 0x08001000, ...}]
3. openocd_start(chip="STM32F407VGTx")  → 启动 OpenOCD
4. gdb_connect()  → 连接 GDB
5. gdb_set_breakpoint(location="main")  → 符号名设断点（不需要 halt）

gdb_set_breakpoint 支持三种位置格式：
- 地址：gdb_set_breakpoint(location="0x08001000")
- 符号：gdb_set_breakpoint(location="main")  → 需要 elf_load
- 文件行号：gdb_set_breakpoint(location="main.c:42")  → 需要 debug sections
```

```
场景：读取全局变量

1. elf_load(path="build/app.elf")
2. openocd_start(chip="STM32F407VGTx")  → 启动 OpenOCD
3. gdb_connect()                → 连接 GDB
4. gdb_read_var(name="g_counter")  → {name: "g_counter", address: "0x20000100", raw_hex: "2A000000"}（不需要 halt）
```

---

## SVD 寄存器解码 (svd_*)

### 设置芯片
- `svd_set_chip(chip)` — 设置 SVD 目标芯片
  - chip: 精确芯片名称（来自 chip_search）
  - **serialized-global**：同一时间只能设置一个芯片
  - 必须在其他 svd_* 调用之前完成

### 浏览外设
- `svd_list_peripherals(query?, offset?, limit?)` — 列出外设
  - query: 关键词过滤
  - offset/limit: 分页（默认 0/50）
  - **parallel-safe**：可并行调用

### 查看寄存器定义
- `svd_get_register(peripheral, register)` — 获取寄存器描述
  - 返回：SVD 元数据（位域定义、复位值、绝对地址）
  - **parallel-safe**

### 典型用法

```
场景：查看并配置 GPIOA 引脚

1. svd_set_chip(chip="STM32F407VGT6")
2. svd_list_peripherals(query="GPIO")  → [{name: "GPIOA", ...}, {name: "GPIOB", ...}]
3. svd_get_register(peripheral="GPIOA", register="MODER")  → {abs_addr: "0x40020000", fields: [...]}
4. openocd_start(chip="STM32F407VGTx")  → 启动 OpenOCD
5. gdb_connect()  → 连接 GDB
6. gdb_halt()  → 停住 CPU（避免读-改-写期间固件并发修改寄存器）
7. gdb_read_mem(sn, addr="0x40020000", len=4, fmt="u32")  → {words: ["0xA8000000"]}
8. [解码：bit0=0 → 输入模式，修改为 01 输出]
9. gdb_write_mem(sn, addr="0x40020000", data=[0,0,0,1])  → 设置 MODER bit0=01
10. gdb_resume()  → 恢复运行（如果需要）
```
