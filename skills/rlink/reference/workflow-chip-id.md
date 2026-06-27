# 芯片识别工作流

## 场景：用户不知道连接的是什么芯片

### chip_detect
- `chip_detect(sn, transport?, speed?)` — 通过调试接口自动探测芯片型号
  - 临时启动 OpenOCD，读取芯片 ID 寄存器后自动关闭
  - 返回候选芯片列表（按置信度排序）
  - **即使识别失败也返回原始硬件信息**：cpuid + idcode
  - 前置：设备在线

```
用户：我有个开发板不知道什么型号

步骤：
1. list_devices()              → 确认设备在线
2. chip_detect()           → {
     detected: true,
     top_candidates: ["STM32::STM32F407VGTx", ...],
     matches: [...],
     cpuid: {
       raw: "0x410FC241",
       implementer: "ARM",
       part_no: "Cortex-M4",
       variant: 0,
       revision: 1
     },
     idcode: {
       dbgmcu_idcode: "0x10016413",
       dev_id: "0x413",
       rev_id: "0x1001",
       family: "STM32/GD32/AT32"
     }
   }
3. chip_get(name)              → 获取详细信息（flash/RAM 地址等）
```

### 芯片数据库查询
- `chip_search(query)` — 按关键词模糊搜索
- `chip_get(name)` — 获取芯片详细信息

## 识别流程

```
用户说不知道芯片型号
    │
    ├─ 设备在线？
    │   ├─ 是 → chip_detect() 一步到位
    │   │       ├─ detected=true → 从 matches 选择芯片
    │   │       └─ detected=false → 检查 cpuid/idcode 原始值
    │   │           → 手动匹配或让用户确认
    │   └─ 否 → 让用户连接调试器
    │
    └─ 用户说了大概型号？
        └─ chip_search(query) 搜索 → 从结果中选择
```

## 返回字段说明

### detected=true 时
- `top_candidates`: 置信度最高的候选芯片（字符串数组）
- `matches`: 所有匹配结果详细信息（包含 vendor, name, confidence, flash_kb 等）
- `cpuid`: CPU 核心信息
- `idcode`: 芯片 ID 寄存器信息

### detected=false 时
- `cpuid`: CPU 核心信息（始终返回）
- `idcode`: 芯片 ID 寄存器信息（如果读取成功）
- `hint`: 识别失败提示
