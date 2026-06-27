# Serial Communication Tool Schema

> Call tools via `rlink_invoke({ "tool": "...", "args": {...} })`.
> `?` suffix means optional.

## serial_bind

Bind serial to TCP. Returns {bind_port}. Requires: serial_open. [serialized:serial]

**Parameters**

- `index` (integer) — Port index: 0=UART1, last=debugger-serial.
- `sn` (string?) — Device SN. Optional if single device.

---

## serial_clear

Clear serial read buffer. [serialized:serial]

**Parameters**

- `index` (integer) — Port index: 0=UART1, last=debugger-serial.
- `sn` (string?) — Device SN. Optional if single device.

---

## serial_close

Close serial port. Auto-unbinds if bound. [serialized:serial]

**Parameters**

- `index` (integer) — Port index: 0=UART1, last=debugger-serial.
- `sn` (string?) — Device SN. Optional if single device.

---

## serial_list

List serial ports. Returns {ports: [{index, name, is_open, bind_port}]}. [serialized:serial]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## serial_open

Open serial port. index: 0=UART1, 1=UART2. baud: e.g. 115200. [serialized:serial]

**Parameters**

- `baud` (integer) — Baud rate.
- `data_bits` (integer?) — Data bits. Default 8.
- `index` (integer) — Port index: 0=UART1, 1=UART2, last=debugger serial.
- `parity` (string?) — "none" | "odd" | "even". Default "none".
- `sn` (string?) — Device SN. Optional if single device.
- `stop_bits` (integer?) — Stop bits. Default 1.

---

## serial_read

Read serial data. timeout_ms: default 5000. Returns {data, bytes_read}. Requires: serial_bind. [serialized:serial]

**Parameters**

- `format` (string?) — "text" (default) | "hex" | "both".
- `index` (integer) — Port index: 0=UART1, last=debugger-serial.
- `max_bytes` (integer?) — Max bytes. Default 4096.
- `sn` (string?) — Device SN. Optional if single device.
- `timeout_ms` (integer?) — Timeout ms. Default 5000.

---

## serial_unbind

Unbind serial TCP bridge. [serialized:serial]

**Parameters**

- `index` (integer) — Port index: 0=UART1, last=debugger-serial.
- `sn` (string?) — Device SN. Optional if single device.

---

## serial_write

Write to serial. data: string or {"hex": "..."}. Requires: serial_bind. [serialized:serial]

**Parameters**

- `data` (any) — Data payload: plain string for text, or {"hex": "..."} for binary.
- `index` (integer) — Port index: 0=UART1, last=debugger-serial.
- `sn` (string?) — Device SN. Optional if single device.

---

