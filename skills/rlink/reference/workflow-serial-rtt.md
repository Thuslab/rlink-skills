# 串口与 RTT 数据流

## 串口 (serial_*)

### 前置条件
- 设备已连接（list_devices 可见）

### 建立连接
1. `serial_list(sn)` — 列出所有串口，返回 name, is_open, bind_port, config
2. `serial_open(sn, index, baud)` — 打开串口
   - index: 0=UART1, 1=UART2, 最后一个=调试器串口
   - baud: 波特率（必填），如 115200
3. `serial_bind(sn, index)` — 绑定到本地 TCP 端口
   - 返回 `bind_port`，可用于直接 TCP socket 读写（持续监听）

### 数据收发（命令-响应 / 轮询）

- `serial_write(sn, index, data)` — 发送数据（字符串或 {hex: "..."}）
- `serial_read(sn, index, timeout_ms?)` — 同步读取（默认超时 5000ms）
- 适合：发送命令等回复、定时轮询
- 持续监听：循环调用 `serial_read` 即可，每次返回这段时间收到的数据，不会卡住

### 持续监听（直接 TCP 连接 bind_port）

当需要长时间不间断接收数据流时，**不要轮询，直接 TCP 连接 `serial_bind` 返回的 `bind_port`**：

```
1. serial_open(index=0, baud=115200)
2. serial_bind(index=0)            → 返回 bind_port，如 38080
3. 用任意 TCP 客户端连接 127.0.0.1:38080
   - 该 socket 收到的字节流即 UART 接收数据
   - 向该 socket 写入的字节会发往 UART 发送
   - 双向、无需轮询、不会丢数据
```

### 关闭连接
1. `serial_unbind(sn, index)` — 解绑 TCP（断开 bind_port）
2. `serial_close(sn, index)` — 关闭串口（自动 unbind）

### 清空缓冲区
- `serial_clear(sn, index)` — 清空读缓冲区

### 完整示例

```
用户：打开串口 115200 波特率，发送 AT 命令

步骤：
1. serial_list()                    → 查看可用串口
2. serial_open(index=0, baud=115200) → 打开 UART1
3. serial_bind(index=0)             → 绑定 TCP
4. serial_clear(index=0)            → 清空历史数据
5. serial_write(index=0, data="AT\r\n")  → 发送
6. serial_read(index=0, timeout_ms=2000) → 等待回复
7. serial_close(index=0)            → 关闭
```

```
用户：持续监听串口日志

步骤：
方案 A（轮询）：
1. serial_open(index=0, baud=115200)
2. serial_bind(index=0)
3. serial_clear(index=0)                   → 清空历史数据
4. 循环 serial_read(index=0, timeout_ms=2000)  → 每次取一段数据
5. serial_close(index=0)

方案 B（外部程序长连接）：
1. serial_open + serial_bind          → 得到 bind_port
2. 外部程序 TCP 连接 127.0.0.1:bind_port，持续读写
3. serial_close 收尾
```

---

## RTT (rtt_*)

### 前置条件
- OpenOCD 已启动（openocd_start 已调用）
- MCU 固件中包含 RTT 控制块（通常使用 SEGGER RTT 库）
- 确保固件启动后运行过RTT 控制块初始化代码

### 运行时要求
RTT 数据收发依赖 MCU 固件正在运行。CPU halt 状态下，RTT 通道无法产生新数据。

### 建立连接
1. `rtt_start(sn, scan_addr?)` — 扫描 RTT 控制块
   - scan_addr: 扫描起始地址（默认 0x20000000，即 SRAM 起始）
   - 返回扫描到的通道列表（不含 bind_port）
2. `rtt_channels(sn)` — 列出 RTT 通道详情
   - 返回每个通道的 index、up_name、up_size、down_name、down_size
   - up_name/down_name 为 null 表示该方向不存在（通道是单向的）
3. `rtt_bind_channel(sn, channel)` — 绑定通道到本地 TCP 端口，返回 `bind_port`
   - channel: 通道索引（通常 0=Terminal）
   - 返回 `bind_port`，可用于直接 TCP socket 读写

### 数据收发（命令-响应 / 轮询）

- `rtt_write(sn, channel, data)` — 写入 down-buffer（PC→MCU），**需要通道有 down buffer**
- `rtt_read(sn, channel, timeout_ms?)` — 同步读取 up-buffer（MCU→PC），**需要通道有 up buffer**
- 持续监听：循环调用 `rtt_read` 即可
- **先通过 `rtt_channels` 确认通道方向**，向只有 up buffer 的通道写数据会失败

### 持续监听（直接 TCP 连接 bind_port）

与串口完全相同——长时间监听某通道时，直接 TCP 连接 `rtt_bind_channel` 返回的 `bind_port`：

```
1. rtt_start()
2. rtt_bind_channel(channel=2)     → 返回 bind_port，如 39090
3. TCP 连接 127.0.0.1:39090，持续读取该通道的 up 数据流
```

### 关闭连接
1. `rtt_unbind_channel(sn, channel)` — 解绑 TCP

### 清空缓冲区
- `rtt_clear(sn, channel)` — 清空指定通道缓冲区

### 完整示例

```
用户：连接 RTT 查看日志（用轮询）

步骤：
1. openocd_start(chip="STM32F407VGTx")  → 如果还没启动
2. rtt_start()                           → 扫描控制块，得到通道列表
3. rtt_channels()                        → 查看通道，确认目标通道有 up buffer
4. rtt_bind_channel(channel=0)           → 绑定通道 0
5. rtt_clear(channel=0)                  → 清空历史数据
6. 循环 rtt_read(channel=0, timeout_ms=2000)  → 持续取数据
7. rtt_unbind_channel(channel=0)
```

```
用户：通过 RTT 发送命令到 MCU

步骤：
1. rtt_start()
2. rtt_channels()                                → 确认通道有 down buffer（发送）和 up buffer（接收）
3. rtt_bind_channel(channel=0)
4. rtt_clear(channel=0)                          → 清空历史数据
5. rtt_write(channel=0, data="hello")            → 发送
6. rtt_read(channel=0, timeout_ms=2000)          → 等待回复
```
