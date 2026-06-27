# OpenOCD Lifecycle Tool Schema

> Call tools via `rlink_invoke({ "tool": "...", "args": {...} })`.
> `?` suffix means optional.

## openocd_start

Start OpenOCD. chip: from chip_search or omit for auto. Returns {gdb_port, is_owner}. [serialized:ocd]

**Parameters**

- `chip` (string?) — Target chip name, from chip_search. Omit to use generic Cortex-M config for chip detection.
- `sn` (string?) — Device SN. Optional if single device.
- `speed` (integer?) — Debug speed kHz, 0=auto (default)
- `transport` (string?) — Debug transport: swd (default) or jtag

---

## openocd_status

Get OpenOCD status. Returns {running, gdb_port, is_owner, chip}. [serialized:ocd]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## openocd_stop

Stop OpenOCD and release debug session. [serialized:ocd]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

