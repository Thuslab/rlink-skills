# RTT Logging Tool Schema

> Call tools via `rlink_invoke({ "tool": "...", "args": {...} })`.
> `?` suffix means optional.

## rtt_bind_channel

Bind RTT channel to TCP. Returns {bind_port}. Requires: rtt_start. [serialized:rtt]

**Parameters**

- `channel` (integer) — Channel index.
- `sn` (string?) — Device SN. Optional if single device.

---

## rtt_channels

List RTT channels. Returns {channels: [{index, name, up_size, down_size, bind_port}]}. [serialized:rtt]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## rtt_clear

Clear RTT channel read buffer. [serialized:rtt]

**Parameters**

- `channel` (integer) — Channel index.
- `sn` (string?) — Device SN. Optional if single device.

---

## rtt_read

Read RTT data. timeout_ms: default 5000. Returns {data, bytes_read}. Requires: rtt_bind_channel. [serialized:rtt]

**Parameters**

- `channel` (integer) — Channel index.
- `format` (string?) — "text" (default) | "hex" | "both".
- `max_bytes` (integer?) — Max bytes. Default 4096.
- `sn` (string?) — Device SN. Optional if single device.
- `timeout_ms` (integer?) — Timeout ms. Default 5000.

---

## rtt_start

Start RTT scan. scan_addr: "0x..." (default 0x20000000). Returns {channels}. Requires: openocd_start. [serialized:rtt]

**Parameters**

- `buffer_cap` (integer?) — Message buffer capacity. Default 4096.
- `interval_ms` (integer?) — Polling interval ms. Default 100.
- `scan_addr` (string?) — Scan start address hex. Default 0x20000000.
- `scan_size` (integer?) — Scan range bytes. Default 65536.
- `sn` (string?) — Device SN. Optional if single device.

---

## rtt_unbind_channel

Unbind RTT channel TCP bridge. [serialized:rtt]

**Parameters**

- `channel` (integer) — Channel index.
- `sn` (string?) — Device SN. Optional if single device.

---

## rtt_write

Write to RTT down-buffer. data: string or {"hex": "..."}. Requires: rtt_bind_channel. [serialized:rtt]

**Parameters**

- `channel` (integer) — Channel index.
- `data` (any) — Data payload: plain string for text, or {"hex": "..."} for binary.
- `sn` (string?) — Device SN. Optional if single device.

---

