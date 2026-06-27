# OCD Direct Operations Tool Schema

> Call tools via `rlink_invoke({ "tool": "...", "args": {...} })`.
> `?` suffix means optional.

## ocd_fault_analysis

Cortex-M fault analysis. Returns fault report: {cfsr, hfsr, pc, sp, fault_pc_location}. Requires: CPU halted. [serialized:ocd]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## ocd_get_target_arch

Get target architecture. Returns {arch}. [serialized:ocd]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## ocd_halt

Halt CPU via OpenOCD. Returns {halted: true}. [serialized:ocd]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## ocd_is_halted

Check if CPU halted. Returns {halted: bool}. [serialized:ocd]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## ocd_read_memory

Read memory. addr: "0x...", len: bytes. Returns {data: hex}. Requires: openocd_start. [serialized:ocd]

**Parameters**

- `addr` (string) — Address in "0x..." format.
- `len` (integer) — Byte count.
- `sn` (string?) — Device SN. Optional if single device.

---

## ocd_reset

Reset target. mode: "halt" (default) or "run". Returns {reset, mode}. [serialized:ocd]

**Parameters**

- `mode` (string?) — "halt" (default) or "run".
- `sn` (string?) — Device SN. Optional if single device.

---

## ocd_resume

Resume CPU via OpenOCD. Returns {running: true}. [serialized:ocd]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## ocd_write_memory

Write memory. addr: "0x...", data_hex: "AABB". Requires: openocd_start. [serialized:ocd]

**Parameters**

- `addr` (string) — Address in "0x..." format.
- `data_hex` (string) — Hex-encoded binary (e.g. "deadbeef").
- `sn` (string?) — Device SN. Optional if single device.

---

