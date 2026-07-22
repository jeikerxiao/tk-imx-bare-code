# 15_lcd — LCD液晶屏驱动实验

## 1. 目录功能概述

本目录存放I.MX6ULL裸机开发工程第十五个实验——**LCD液晶屏驱动**的完整代码。
该实验实现I.MX6ULL ELCDIF（Enhanced LCD Interface）控制器的初始化与RGB LCD屏的驱动，支持1024×600分辨率TFT液晶屏的显示。
LCD驱动是嵌入式图形界面的基础，后续触摸屏、GUI等实验均依赖此模块。

## 2. 核心源码文件说明

| 文件名 | 类型 | 说明 |
|--------|------|------|
| `project/main.c` | 主程序 | 初始化LCD并显示测试图案 |
| `project/start.S` | 汇编启动文件 | 程序入口，初始化C运行环境 |
| `bsp/lcd/` | 外设驱动 | LCD驱动模块：`bsp_lcd.c`、`bsp_lcd.h`，含`lcd_init()`、`lcd_show_string()`等接口 |
| `bsp/lcdapi/` | 公共工具 | LCD高级API封装：`bsp_lcdapi.c`、`bsp_lcdapi.h`，提供字符、图形绘制接口 |
| `bsp/uart/` | 外设驱动 | UART驱动模块 |
| `bsp/delay/` | 公共工具 | 高精度延时模块 |
| `bsp/int/` | 公共工具 | 中断系统管理模块 |
| `bsp/exit/` | 外设驱动 | 外部中断驱动模块 |
| `bsp/key/` | 外设驱动 | 按键驱动模块 |
| `bsp/gpio/` | 公共工具 | 通用GPIO操作封装 |
| `bsp/beep/` | 外设驱动 | 蜂鸣器驱动模块 |
| `bsp/led/` | 外设驱动 | LED驱动模块 |
| `bsp/clk/` | 外设驱动 | 时钟使能与系统时钟配置模块 |
| `stdio/` | 标准库移植 | printf等格式化输出函数 |
| `imx6ul/` | 头文件集合 | 寄存器定义头文件 |
| `imx6ul.lds` | 链接脚本 | 代码加载地址及段布局 |
| `Makefile` | 编译脚本 | 多目录自动编译构建规则 |
| `imxdownload` | 烧录工具 | SD卡烧写工具 |

## 3. 关键原理简述

### 3.1 ELCDIF控制器
I.MX6ULL内置ELCDIF（Enhanced LCD Interface），支持：
- 分辨率：最高可达WXGA（1366×768）
- 颜色深度：24bpp RGB888 / 18bpp RGB666 / 16bpp RGB565
- 接口类型：支持DOTCLK和VSYNC模式
- 显存：使用DDR中的帧缓冲区（Frame Buffer）

### 3.2 LCD显示流程
1. **时钟配置**：配置LCDIF时钟，通常使用PLL5（Video PLL）作为时钟源
2. **引脚复用**：将相关GPIO复用为LCDIF功能（数据线、控制线）
3. **ELCDIF配置**：设置分辨率、颜色格式、时序参数（HSYNC/VSYNC/PCLK等）
4. **显存分配**：在DDR中分配帧缓冲区，LCD控制器自动从帧缓冲区读取像素数据
5. **使能显示**：启动ELCDIF控制器，LCD屏幕显示帧缓冲区内容

### 3.3 时序参数
LCD显示需要正确的时序参数：
- **HSYNC**：水平同步信号，标志一行的开始
- **VSYNC**：垂直同步信号，标志一帧的开始
- **PCLK**：像素时钟，决定显示刷新率
- **HBP/HFP**：水平前后肩，行消隐期
- **VBP/VFP**：垂直前后肩，帧消隐期

## 4. 编译与构建方式

### 4.1 编译命令
```bash
make
```

### 4.2 清理构建
```bash
make clean
```

## 5. 下载与验证步骤

1. 编译生成`lcd.bin`
2. 使用`imxdownload`工具烧录至SD卡：
```bash
./imxdownload lcd.bin /dev/sdX
```
3. SD卡插入开发板，设置SD卡启动模式
4. 确保RGB LCD屏幕已正确连接至开发板
5. 上电启动

## 6. 预期运行现象

开发板上电后：
- LCD屏幕显示测试画面（如彩条、文字等）
- 屏幕刷新正常，无闪烁

验证标准：LCD正常显示，画面清晰稳定。

## 7. 常见问题排查

| 故障现象 | 可能原因 | 解决办法 |
|----------|----------|----------|
| 屏幕无显示 | 时钟配置错误 | 检查LCDIF时钟源和分频系数，确保像素时钟正确 |
| 屏幕显示花屏 | 时序参数错误 | 核对HSYNC/VSYNC/HBP/VBP等参数与LCD屏幕规格书是否匹配 |
| 屏幕颜色异常 | 颜色格式不匹配 | 确认ELCDIF颜色格式（RGB888/RGB565）与屏幕一致 |
| 屏幕闪烁 | 刷新率过低 | 提高像素时钟频率，或检查VSYNC信号 |
| 屏幕分辨率不对 | 分辨率参数错误 | 检查ELCDIF分辨率设置是否正确 |
