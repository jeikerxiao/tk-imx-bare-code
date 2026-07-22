# 正点原子 I.MX6ULL ALPHA 裸机开发工程

> 基于 ARM Cortex-A7 的 I.MX6ULL 处理器，纯裸机（Bare-metal）程序开发，无操作系统，涵盖 GPIO、时钟、中断、定时器、UART、LCD、I2C、SPI、PWM、ADC 等完整外设驱动实验。

---

## 1. 项目概述

本工程是正点原子 I.MX6ULL ALPHA 开发板的裸机开发例程集合，采用 ARM 汇编 + C 语言开发，从零开始逐步构建完整的嵌入式裸机开发体系。所有代码直接操作 I.MX6ULL 寄存器，不依赖 Linux 内核和任何操作系统，是学习 ARM 嵌入式底层开发的理想教材。

### 核心特点
- **纯裸机开发**：直接操作寄存器，深入理解硬件工作原理
- **循序渐进**：从汇编点灯到复杂外设驱动，难度逐步递进
- **模块化设计**：采用 BSP（Board Support Package）架构，代码可复用性强
- **工程化实践**：包含完整的 Makefile 构建系统、链接脚本和烧录工具

### 目标硬件
- **开发板**：正点原子 I.MX6ULL ALPHA 开发板
- **处理器**：NXP I.MX6ULL（ARM Cortex-A7，主频 528MHz）
- **内存**：512MB DDR3
- **存储**：SD 卡启动

### 开发环境
- **交叉编译器**：`arm-linux-gnueabihf-gcc`
- **链接器**：`arm-linux-gnueabihf-ld`
- **烧录工具**：`imxdownload`（正点原子提供）
- **调试接口**：UART1（波特率 115200，8N1）

---

## 2. 工程目录结构

```
tk-imx-bare-code/
├── 01_leds/               # 实验1：汇编语言点亮 LED
├── 02_ledc/               # 实验2：C 语言 LED 闪烁
├── 03_ledc_stm32/         # 实验3：STM32 风格寄存器操作
├── 04_ledc_sdk/           # 实验4：NXP 官方 SDK 风格
├── 05_ledc_bsp/           # 实验5：BSP 模块化驱动
├── 06_beep/               # 实验6：蜂鸣器驱动
├── 07_key/                # 实验7：按键输入检测
├── 08_clk/                # 实验8：系统时钟配置
├── 09_int/                # 实验9：中断系统
├── 10_epit_timer/         # 实验10：EPIT 定时器
├── 11_key_filter/         # 实验11：定时器按键消抖
├── 12_highpreci_delay/    # 实验12：高精度延时
├── 13_uart/               # 实验13：串口通信
├── 14_printf/             # 实验14：printf 移植
├── 15_lcd/                # 实验15：LCD 液晶屏驱动
├── 16_rtc/                # 实验16：实时时钟
├── 17_i2c/                # 实验17：I2C 总线与传感器
├── 18_spi/                # 实验18：SPI 总线与传感器
├── 19_touchscreen/        # 实验19：触摸屏驱动
├── 20_pwm_lcdbacklight/   # 实验20：PWM 背光控制
├── 21_adc/                # 实验21：ADC 模数转换
└── README.md              # 本文件
```

---

## 3. 例程一览表

| 序号 | 目录名 | 实验名称 | 核心技术点 | 难度 |
|:--:|:--|:--|:--|:--:|
| 01 | [`01_leds`](./01_leds/README.md) | 汇编语言 LED 点灯 | ARM 汇编、GPIO 寄存器直接操作 | ⭐ |
| 02 | [`02_ledc`](./02_ledc/README.md) | C 语言 LED 闪烁 | C 运行环境初始化、`volatile` 关键字 | ⭐ |
| 03 | [`03_ledc_stm32`](./03_ledc_stm32/README.md) | STM32 风格寄存器封装 | 结构体指针访问寄存器 |  |
| 04 | [`04_ledc_sdk`](./04_ledc_sdk/README.md) | NXP 官方 SDK 风格 | SDK 头文件移植、语义化宏 | ⭐ |
| 05 | [`05_ledc_bsp`](./05_ledc_bsp/README.md) | BSP 模块化 LED 驱动 | BSP 架构、通用 Makefile | ⭐⭐ |
| 06 | [`06_beep`](./06_beep/README.md) | 蜂鸣器驱动 | GPIO 输出、有源蜂鸣器控制 | ⭐⭐ |
| 07 | [`07_key`](./07_key/README.md) | 按键输入检测 | GPIO 输入、软件消抖 | ⭐⭐ |
| 08 | [`08_clk`](./08_clk/README.md) | 系统时钟配置 | PLL、PFD、CCM 时钟树 | ⭐⭐⭐ |
| 09 | [`09_int`](./09_int/README.md) | 系统中断 | GIC 中断控制器、外部中断 | ⭐⭐⭐ |
| 10 | [`10_epit_timer`](./10_epit_timer/README.md) | EPIT 定时器 | 周期中断定时、精确延时 | ⭐⭐⭐ |
| 11 | [`11_key_filter`](./11_key_filter/README.md) | 定时器按键消抖 | 中断 + 定时器消抖机制 | ⭐⭐⭐ |
| 12 | [`12_highpreci_delay`](./12_highpreci_delay/README.md) | 高精度延时 | GPT 定时器、微秒级精度 | ⭐⭐⭐ |
| 13 | [`13_uart`](./13_uart/README.md) | 串口通信 | UART1 初始化、波特率计算 | ⭐⭐⭐ |
| 14 | [`14_printf`](./14_printf/README.md) | printf 移植 | 标准库裁剪、`_write`/`_read` 重定向 | ⭐⭐⭐ |
| 15 | [`15_lcd`](./15_lcd/README.md) | LCD 液晶屏驱动 | ELCDIF 控制器、帧缓冲、时序参数 | ⭐⭐⭐⭐ |
| 16 | [`16_rtc`](./16_rtc/README.md) | 实时时钟 | SNVS RTC、时间格式转换 | ⭐⭐⭐ |
| 17 | [`17_i2c`](./17_i2c/README.md) | I2C 总线 / AP3216C 传感器 | I2C 通信协议、传感器驱动 | ⭐⭐⭐⭐ |
| 18 | [`18_spi`](./18_spi/README.md) | SPI 总线 / ICM20608 传感器 | ECSPI 控制器、六轴传感器 | ⭐⭐⭐ |
| 19 | [`19_touchscreen`](./19_touchscreen/README.md) | 触摸屏驱动 | 电容式触摸、I2C 通信 | ⭐⭐⭐⭐ |
| 20 | [`20_pwm_lcdbacklight`](./20_pwm_lcdbacklight/README.md) | PWM 背光控制 | PWM 波形生成、占空比调节 | ⭐⭐⭐ |
| 21 | [`21_adc`](./21_adc/README.md) | ADC 模数转换 | 12 位 ADC、电压采集 | ⭐⭐⭐⭐ |

---

## 4. 学习路线图

### 第一阶段：入门基础（实验 01~05）
从汇编点灯开始，逐步过渡到 C 语言开发，掌握寄存器操作的基本方法。理解不同编程风格（直接寄存器访问、STM32 结构体封装、NXP SDK 封装）的差异，最终建立 BSP 模块化工程化的开发思维。

> **关键能力**：寄存器读写、GPIO 配置、链接脚本、Makefile 基础、BSP 模块设计

### 第二阶段：核心外设（实验 06~14）
在 BSP 架构基础上，依次学习蜂鸣器、按键、时钟、中断、定时器、串口等核心外设。重点理解中断机制、定时器原理和串口通信，这些是嵌入式系统开发的基石。

> **关键能力**：中断处理、定时器配置、串口通信、printf 调试

### 第三阶段：高级应用（实验 15~21）
学习 LCD 显示、传感器通信（I2C/SPI）、触摸屏、PWM 和 ADC 等高级外设。这些实验涉及更复杂的硬件接口和通信协议，是实际产品开发中常用的技术。

> **关键能力**：LCD 驱动、I2C/SPI 通信协议、触摸屏校准、PWM 控制、ADC 采样

---

## 5. 快速开始

### 5.1 环境准备
1. 安装 ARM 交叉编译工具链（`arm-linux-gnueabihf-gcc`）
2. 准备一张 SD 卡（建议容量 4GB 以上）
3. 安装串口调试工具（如 SecureCRT、Putty、MobaXterm 等）
4. 准备 USB 转 TTL 模块（用于串口调试）

### 5.2 编译例程

以 `02_ledc` 为例：

```bash
# 进入目标例程目录
cd 02_ledc/

# 编译生成二进制文件
make

# 清理构建产物
make clean
```

### 5.3 烧录与运行

```bash
# 将编译生成的 .bin 文件烧录至 SD 卡
# 注意：/dev/sdX 请替换为实际的 SD 卡设备节点，务必确认正确！
sudo ./imxdownload ledc.bin /dev/sdX
```

1. 将 SD 卡插入开发板 SD 卡槽
2. 设置拨码开关为 SD 卡启动模式
3. 连接 USB 转 TTL 至 PC（如需串口调试）
4. 上电启动

---

## 6. 技术要点速查

### 6.1 通用编译命令
```bash
# 编译当前例程
make

# 清理构建产物
make clean

# 手动编译流程（以 02_ledc 为例）
arm-linux-gnueabihf-gcc -Wall -nostdlib -c -O2 -o start.o start.S
arm-linux-gnueabihf-gcc -Wall -nostdlib -c -O2 -o main.o main.c
arm-linux-gnueabihf-ld -Timx6ul.lds -o ledc.elf start.o main.o
arm-linux-gnueabihf-objcopy -O binary -S ledc.elf ledc.bin
```

### 6.2 代码加载地址
- **起始地址**：`0x87800000`
- **栈指针**：`0x80200000`

### 6.3 常用寄存器基地址
| 外设 | 基地址 |
|:--|:--|
| CCM（时钟控制） | `0x020C4000` |
| GPIO1 | `0x0209C000` |
| IOMUXC | `0x020E0000` |
| UART1 | `0x02020000` |
| GIC | `0x00A00000` |
| EPIT1 | `0x020D0000` |

---

## 7. 常见问题

| 问题 | 原因 | 解决 |
|:--|:--|:--|
| `arm-linux-gnueabihf-gcc: command not found` | 交叉编译器未安装或 PATH 未配置 | 安装工具链并添加到环境变量 |
| 程序无法启动 | 拨码开关设置错误 | 设置为 SD 卡启动模式 |
| LED 不亮 | 寄存器地址错误 | 核对参考手册寄存器地址 |
| 串口无输出 | 波特率不匹配 | 确认波特率为 115200，8N1 |
| LCD 无显示 | 时序参数错误 | 核对 LCD 屏幕规格书参数 |

更多问题详见各子目录下的 `README.md` 文件 **第7章：常见问题排查**。

---

## 8. 相关资源

- **I.MX6ULL 参考手册**：NXP 官方 Reference Manual（IMX6ULLRM）
- **I.MX6ULL 数据手册**：NXP 官方 Datasheet（IMX6ULLCEC）
- **正点原子开发板资料**：www.alientek.com
- **交叉编译工具链**：ARM GCC Toolchain

---

## 9. 版权与许可

Copyright © ALIENTEK Co., Ltd. All rights reserved.

本工程仅供学习研究使用，不得用于商业用途。

---

> **提示**：点击上表中的目录名可跳转到对应实验的详细 README 文档，获取该实验的完整技术说明。
