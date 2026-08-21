# NUCLEO-N657X0-Q Board Adaptation (contest2026_224_board)

## 概述

本目录包含 NUCLEO-N657X0-Q 开发板的 NuttX BSP 适配，用于 2026 openvela AI 硬件开发者大赛「新硬件适配」赛道。

## 硬件信息

| 项目 | 值 |
|------|-----|
| MCU | STM32N657X0 (Cortex-M55, ARMv8.1-M) |
| CPU 频率 | 200 MHz (HSI 64MHz PLL) |
| 内部 Flash | **无** (STM32N6 特殊) |
| 外部 Flash | MX25UM51245G (512Mbit/64MB) via XSPI2 |
| RAM | 4MB AXI SRAM |
| 调试串口 | USART1 (PE5=TX, PE6=RX) via ST-Link VCP |
| LEDs | LD5(PG10, Red), LD6(PG0, Green), LD7(PG8, Blue) |

## 当前状态

### 已完成 (Step 1: 最小 NSH 基线)
- [x] 板级 Kconfig / defconfig
- [x] board.h (时钟、引脚、LED 定义)
- [x] 链接脚本 (SRAM-only, DEV boot mode)
- [x] Make.defs (ARMv8-M toolchain)
- [x] CMakeLists.txt
- [x] 板级源文件 (boot, bringup, LEDs)

### 待完成 (Step 2: XSPI Flash 支持)
- [ ] XSPI 驱动 (`stm32n6_xspi.c`)
- [ ] VDDIO3 电源域配置 (1.8V for XSPI pins)
- [ ] BSEC OTP 熔丝配置 (高速 IO 优化)
- [ ] FSBL (第一阶段引导加载程序)
- [ ] XIP 链接脚本 (`0x70080000`)
- [ ] Flash 分区和 MTD 驱动
- [ ] LittleFS 掉电保存

## 构建方法

```bash
# 进入 openvela 工作区根目录
cd /home/warner/openvela

# 编译
./build.sh vendor/openvela/boards/contest2026_224_board/configs/nsh [-j8]

# 或使用 menuconfig 调整配置
./build.sh vendor/openvela/boards/contest2026_224_board/configs/nsh menuconfig
```

## 烧录运行 (DEV 模式)

```bash
# 通过 ST-Link 加载到 SRAM
STM32_Programmer_CLI -c port=SWD -w nuttx.bin 0x34000400 -rst
```

## 目录结构

```
contest_board/
├── CMakeLists.txt          # 顶层 CMake
├── Kconfig                 # 板级 Kconfig
├── README.md               # 本文件
├── configs/
│   └── nsh/
│       └── defconfig       # NSH 最小配置
├── include/
│   └── board.h             # 板级定义 (时钟、LED、引脚)
├── scripts/
│   ├── flash.ld            # 链接脚本 (SRAM-only)
│   └── Make.defs           # Toolchain 配置
└── src/
    ├── CMakeLists.txt      # 源文件 CMake
    ├── Make.defs           # 源文件 Make
    ├── contest_board.h     # 板级私有头文件
    ├── stm32_autoleds.c    # 自动 LED 控制
    ├── stm32_boot.c        # 启动初始化
    ├── stm32_bringup.c     # 板级外设初始化
    └── stm32_userleds.c    # 用户 LED 控制
```

## 参考项目

- 原始 NuttX 板级: `nuttx/boards/arm/stm32n6/nucleo-n657x0-q/`
- MicroPython FSBL 参考: `atk-dnn647-micropython/ports/stm32/boards/NUCLEO_N657X0/`
- XSPI 驱动参考: `atk-dnn647-micropython/ports/stm32/xspi.c`
