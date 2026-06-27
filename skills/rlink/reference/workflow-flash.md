# 固件烧写工作流

## 前置条件
- GDB 已连接（gdb_connect 已调用）

## 推荐方式：gdb_load（一步完成）

`gdb_load(sn, path, addr?, erase_timeout_ms?, write_timeout_ms?)` — 自动烧写：
- 格式：.bin/.hex/.elf/.axf
- ELF/AXF 自动加载符号（无需再调 elf_load）
- 流程：halt → erase → write → done（**完成后 CPU halt**，需手动 gdb_reset）
- addr: 仅 .bin 需要（如 "0x08000000"）
- 发送 4 阶段进度通知：halt→erase→write→done

### 超时参数

**默认超时**：
- `erase_timeout_ms`: 10 秒
- `write_timeout_ms`: 60 秒

**大文件烧写建议超时**：

| 文件大小 | 典型耗时 | 建议 write_timeout_ms |
|----------|----------|----------------------|
| 16KB     | 1-3s     | 默认即可 |
| 128KB    | 5-10s    | 默认即可 |
| 512KB    | 15-30s   | 60000 |
| 1MB      | 30-60s   | 90000 |
| 2MB      | 60-120s  | 120000 |

**注意**：实际时间取决于目标芯片 flash 速度和调试器速度。

### 超时恢复

如果 gdb_load 超时，用 `gdb_disconnect` 恢复：
1. `gdb_disconnect(sn)` — 断开旧连接
2. `gdb_connect(sn)` — 重连（幂等）
3. `gdb_load(sn, path, addr, erase_timeout_ms=30000, write_timeout_ms=60000)` — 加长超时重试

### 示例

```
用户：把 firmware.bin 烧到 0x08000000

步骤：
1. gdb_connect()
2. gdb_load(sn, path="build/firmware.bin", addr="0x08000000")
   → 自动 halt/擦除/写入/完成
3. gdb_reset(mode="run")  → 手动复位运行
```

```
用户：烧写 firmware.elf

步骤：
1. gdb_connect()
2. gdb_load(sn, path="build/firmware.elf")
   → 烧写+加载符号，完成后 halt
3. gdb_reset(mode="run")  → 复位运行
```

```
用户：烧写 512KB 的固件

步骤：
1. gdb_connect()
2. gdb_load(sn, path="build/app.bin", addr="0x08000000",
            erase_timeout_ms=30000, write_timeout_ms=60000)
   → 大文件，加长超时
3. gdb_reset(mode="run")  → 手动复位运行
```

## 手动方式（仅当 gdb_load 不满足时使用）

当需要分段写入、部分擦除、或特殊 flash 操作时：

1. `gdb_flash_erase(sn, addr, length)` — 擦除 flash 区域
   - addr: 起始地址（自动 4KB 对齐）
   - length: 字节数
2. `gdb_flash_write(sn, addr, data_hex)` — 写入数据
   - data_hex: 连续十六进制字符串，如 "deadbeef"
   - 必须在 erase 之后调用
3. `gdb_flash_done(sn)` — 完成烧写序列
   - 必须在所有 write 之后调用

**注意**：
- gdb_flash_write 不会自动擦除，必须先 erase
- gdb_load 是首选方式，手动流程仅在需要精细控制时使用

## 烧写后调试

```
用户：烧写并设断点调试

步骤：
1. gdb_load(path="build/app.elf")       → 烧写（完成后 halt）
2. gdb_set_breakpoint(location="main")  → 设断点
3. gdb_reset(mode="halt")               → 复位
4. gdb_resume()                          → 运行
5. gdb_wait_stop()                       → 等待断点命中
```