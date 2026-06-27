# Device Management Tool Schema

> Call tools via `rlink_invoke({ "tool": "...", "args": {...} })`.
> `?` suffix means optional.

## get_active_device

Get active device. Returns {sn} or {reason} if none. [parallel-safe]

**Parameters**: None

---

## get_full_status

Device status snapshot with next-step hints. Returns {sn, gdb_connected, device: {deviceName?, ide_type, openocd, serial_count, serial, rtt_channels}, hints}. Call first to check state. [parallel-safe]

**Parameters**

- `sn` (string?) — Device SN. Optional if single device.

---

## list_devices

List online devices. Returns {devices: [sn], count}. [parallel-safe]

**Parameters**: None

---

## set_active_device

Set active device by SN from list_devices. [parallel-safe]

**Parameters**

- `sn` (string) — Device SN.

---

