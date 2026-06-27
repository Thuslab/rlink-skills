# SVD Registers Tool Schema

> Call tools via `rlink_invoke({ "tool": "...", "args": {...} })`.
> `?` suffix means optional.

## svd_get_register

Get register description. Returns {peripheral, register, abs_addr, reset, fields}. [parallel-safe]

**Parameters**

- `peripheral` (string) — Peripheral name.
- `register` (string) — Register name.

---

## svd_list_peripherals

List peripherals. query: filter. Returns {peripherals, total, has_more}. [parallel-safe]

**Parameters**

- `limit` (integer?) — Max results.
- `offset` (integer?) — Pagination offset.
- `query` (string?) — Peripheral name filter.

---

## svd_set_chip

Set SVD target chip. chip: from chip_search. Required before svd_list_peripherals. [serialized-global]

**Parameters**

- `chip` (string) — Chip name from chip_search.

---

