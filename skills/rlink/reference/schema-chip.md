# Chip Database Tool Schema

> Call tools via `rlink_invoke({ "tool": "...", "args": {...} })`.
> `?` suffix means optional.

## chip_detect

Auto-detect connected chip. Returns {chip, cpuid, idcode}. Requires: device online. [serialized:ocd]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.
- `speed` (integer?) — Debug speed kHz, default 4000
- `transport` (string?) — Debug transport: swd (default) or jtag

---

## chip_get

Get chip details by name. Returns chip config for openocd_start. [parallel-safe]

**Parameters**

- `name` (string) — Exact chip name, from chip_search results

---

## chip_search

Search chip database. Returns {results, count, total, has_more}. limit: default 20. [parallel-safe]

**Parameters**

- `limit` (integer?) — Max results, default 20, max 100
- `query` (string) — Search keyword, e.g. STM32F407, GD32F103

---

