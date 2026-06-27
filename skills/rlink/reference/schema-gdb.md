# GDB Debug Tool Schema

> Call tools via `rlink_invoke({ "tool": "...", "args": {...} })`.
> `?` suffix means optional.

## gdb_backtrace

Get call stack. limit: frame count. Requires: CPU halted. Returns {frames}. [serialized:gdb]

**Parameters**

- `limit` (integer?) — Max frames.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_command

Execute GDB command. Supports: "break main", "info reg", "x/10xw 0x...", "bt", "monitor cmd". [serialized:gdb]

**Parameters**

- `command` (string) — GDB command string.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_connect

Connect to GDB server. Requires: openocd_start, is_owner=true. Returns target info. [serialized:gdb]

**Parameters**

- `port` (integer?) — GDB port. Default from OpenOCD.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_del_all_breakpoints

Delete all breakpoints and watchpoints. Returns {deleted: count}. [serialized:gdb]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_del_breakpoint

Delete breakpoint by id. [serialized:gdb]

**Parameters**

- `id` (integer) — Breakpoint ID.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_del_watchpoint

Delete watchpoint by id. [serialized:gdb]

**Parameters**

- `id` (integer) — Breakpoint ID.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_disconnect

Disconnect GDB. MCU may halt; call ocd_resume to recover. [serialized:gdb]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_flash_done

Finalize flash write. Call after all gdb_flash_write. [serialized:gdb]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.
- `timeout_ms` (integer?) — Timeout ms.

---

## gdb_flash_erase

Erase flash region. addr: "0x...", length: bytes. Requires: gdb_connect. [serialized:gdb]

**Parameters**

- `addr` (string) — Start addr "0x..." (4KB aligned).
- `length` (integer) — Bytes to erase.
- `sn` (string?) — Device SN. Optional if single device.
- `timeout_ms` (integer?) — Timeout ms.

---

## gdb_flash_write

Write flash. MUST call gdb_flash_erase first. addr: "0x...", data_hex: "AABB". Call gdb_flash_done after all writes. [serialized:gdb]

**Parameters**

- `addr` (string) — Addr "0x...".
- `data_hex` (string) — Hex data "AABB...".
- `sn` (string?) — Device SN. Optional if single device.
- `timeout_ms` (integer?) — Timeout ms.

---

## gdb_get_hw_breakpoint_count

Get hardware breakpoint slot count. Returns {hw_breakpoints: count}. [parallel-safe]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_halt

Halt CPU. Returns {pc, reason, source}. Requires: gdb_connect. [serialized:gdb]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_is_halted

Check if CPU halted. Returns {halted: bool}. Non-blocking. [parallel-safe]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_list_breakpoints

List active breakpoints. Returns {breakpoints: [{id, addr}]}. [serialized:gdb]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_load

Load firmware (.bin/.hex/.elf). Auto halt/erase/write. ELF auto-loads symbols. Returns {written, addr, symbols_loaded}. [serialized:gdb]

**Parameters**

- `addr` (string?) — Flash start address in hex. Optional for .elf/.axf (auto-detected from ELF headers).
- `erase_timeout_ms` (integer?) — Erase timeout ms.
- `path` (string) — Firmware path (.bin/.hex/.elf/.axf).
- `sn` (string?) — Device SN. Optional if single device.
- `write_timeout_ms` (integer?) — Write timeout ms.

---

## gdb_monitor

Send raw command to OpenOCD. e.g. "reset halt", "flash banks". [serialized:gdb]

**Parameters**

- `cmd` (string) — OpenOCD command.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_read_mem

Read memory. addr: "0x...", len: bytes, fmt: u8/u16/u32. Returns {words, dump}. [serialized:gdb]

**Parameters**

- `addr` (string) — Address in "0x..." format.
- `decode_as` (string?) — Re-interpret raw bytes as: "float", "int16", "uint16", "int32".
- `fmt` (string?) — "u8"/"u16"/"u32"/"u64". Default "u32".
- `len` (integer) — Byte count.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_read_reg

Read all registers. decode: true adds xPSR flags. Returns [{name, value, decoded}]. Requires: CPU halted. [serialized:gdb]

**Parameters**

- `decode` (boolean?) — When true, adds decoded xPSR flags (N/Z/C/V/Q/ISR) to the output.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_read_single_reg

Read one register by name. Returns {name, value, decoded}. [serialized:gdb]

**Parameters**

- `name` (string) — Register name.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_read_var

Read global variable by name. Requires: elf_load. Returns {name, address, raw_hex}. [serialized:gdb]

**Parameters**

- `max_size` (integer?) — Max bytes to read (default 64, max 1024)
- `name` (string) — Global variable name (from ELF symbol table)
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_reset

Reset target. mode: "halt"/"run". Returns {pc, reason} or {reset, mode}. [serialized:gdb]

**Parameters**

- `mode` (string?) — "halt" (default) or "run".
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_resume

Resume CPU. Non-blocking. Use gdb_wait_stop to wait for breakpoint. [serialized:gdb]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_set_breakpoint

Set breakpoint. location: "0xADDR" or symbol name. Returns {id}. [serialized:gdb]

**Parameters**

- `condition` (string?) — Optional condition expression. Stored for future use (RSP condition support is limited).
- `location` (string) — "0xADDR", symbol, or "file:line".
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_set_watchpoint

Set data watchpoint. addr: "0x...", kind: read/write/access. Returns {id}. [serialized:gdb]

**Parameters**

- `addr` (string) — Address in "0x..." format.
- `kind` (string?) — "read"/"write"/"access". Default "write".
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_step

Single-step one instruction. Requires: CPU halted. Returns {pc, reason, source}. [serialized:gdb]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_step_out

Step out of current function. Waits up to 10s. Returns {stopped_at, source}. [serialized:gdb]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_wait_stop

Wait for CPU stop. timeout_ms: default 2000. Returns {stopped, pc, reason} or {running: true}. [serialized:gdb]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.
- `timeout_ms` (integer?) — Timeout ms. Default 2000, max 60000.

---

## gdb_write_mem

Write memory. addr: "0x...", data: byte array. [serialized:gdb]

**Parameters**

- `addr` (string) — Address in "0x..." format.
- `data` (array) — Bytes to write.
- `sn` (string?) — Device SN. Optional if single device.

---

## gdb_write_reg

Write register. Requires: CPU halted. name: "pc"/"r0"/"sp"/etc. value: integer (not hex string). May not work in exception handler. [serialized:gdb]

**Parameters**

- `name` (string) — Register: "pc", "sp", "r0", etc.
- `sn` (string?) — Device SN. Optional if single device.
- `value` (integer) — Integer value.

---

## get_toolchain_info

Get toolchain paths. arch: "arm"/"riscv". Returns {gdb, gcc, objdump paths}. [parallel-safe]

**Parameters**

- `arch` (string?) — "arm" or "riscv".

---

