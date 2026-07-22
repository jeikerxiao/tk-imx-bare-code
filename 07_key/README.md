# 07_key — 按键输入实验

## 1. 目录功能概述

本目录存放I.MX6ULL裸机开发工程第七个实验——**按键输入检测**的完整代码。
该实验在蜂鸣器驱动基础上新增按键（KEY）和通用GPIO（gpio）驱动模块，学习如何将GPIO配置为输入模式，实现按键检测功能。
实验通过按键控制蜂鸣器开关和LED状态，演示GPIO输入在实际项目中的典型应用场景。

## 2. 核心源码文件说明

| 文件名 | 类型 | 说明 |
|--------|------|------|
| `project/main.c` | 主程序 | 主循环中轮询检测按键状态，控制蜂鸣器和LED |
| `project/start.S` | 汇编启动文件 | 程序入口，初始化C运行环境 |
| `bsp/key/` | 外设驱动 | 按键驱动模块：`bsp_key.c`、`bsp_key.h` |
| `bsp/gpio/` | 公共工具 | 通用GPIO操作封装：`bsp_gpio.c`、`bsp_gpio.h` |
| `bsp/beep/` | 外设驱动 | 蜂鸣器驱动模块 |
| `bsp/led/` | 外设驱动 | LED驱动模块 |
| `bsp/clk/` | 外设驱动 | 时钟使能模块 |
| `bsp/delay/` | 公共工具 | 延时函数模块 |
| `imx6ul/` | 头文件集合 | 寄存器定义头文件 |
| `imx6ul.lds` | 链接脚本 | 代码加载地址及段布局 |
| `Makefile` | 编译脚本 | 多目录自动编译构建规则 |
| `imxdownload` | 烧录工具 | SD卡烧写工具 |

## 3. 关键原理简述

### 3.1 按键硬件连接
开发板KEY0按键连接至GPIO1_IO18，采用下拉电阻设计，默认低电平。按键按下时引脚被拉高至高电平，软件通过读取GPIO输入寄存器（`GPIO1_DR`或`GPIO1_PSR`）检测按键状态。

### 3.2 GPIO输入配置
按键GPIO配置流程：
1. **时钟使能**：使能GPIO1时钟
2. **IO复用配置**：将GPIO1_IO18复用为GPIO功能
3. **PAD属性配置**：配置上拉/下拉电阻，使能输入缓冲
4. **方向设置**：将GPIO1_IO18设置为输入模式（`GPIO1_GDIR`对应bit清零）
5. **状态读取**：读取`GPIO1_DR`或`GPIO1_PSR`寄存器对应bit判断按键是否按下

### 3.3 软件消抖
按键机械触点存在抖动，按下和释放时会产生多次高低电平跳变。本实验采用软件延时消抖：检测到按键按下后延时约10ms再次确认，若仍为按下状态则判定为有效按键。

### 3.4 GPIO通用封装
`bsp_gpio.h`中定义了通用的GPIO读写函数，如`gpio_direction_output()`、`gpio_direction_input()`、`gpio_set_value()`、`gpio_get_value()`等，方便不同外设复用。

## 4. 编译与构建方式

### 4.1 编译命令
```bash
make
```

### 4.2 Makefile路径更新
```makefile
INCDIRS := imx6ul bsp/clk bsp/led bsp/delay bsp/beep bsp/gpio bsp/key
SRCDIRS := project bsp/clk bsp/led bsp/delay bsp/beep bsp/gpio bsp/key
```

### 4.3 清理构建
```bash
make clean
```

## 5. 下载与验证步骤

1. 编译生成`key.bin`
2. 使用`imxdownload`工具烧录至SD卡：
```bash
./imxdownload key.bin /dev/sdX
```
3. SD卡插入开发板，设置SD卡启动模式
4. 上电启动

## 6. 预期运行现象

开发板上电后：
- LED0以约500ms为周期自动闪烁
- 按下KEY0按键时，蜂鸣器状态翻转（响/停切换）

验证标准：按键按下蜂鸣器响应，说明GPIO输入检测功能正常，GPIO输入输出联动工作正常。

## 7. 常见问题排查

| 故障现象 | 可能原因 | 解决办法 |
|----------|----------|----------|
| 按键无响应 | GPIO方向设置错误 | 检查是否将按键引脚配置为输入模式（GDIR对应bit=0） |
| 按键误触发 | 未消抖或消抖时间不足 | 增加软件消抖延时，通常10ms~20ms |
| 按键持续触发 | 上下拉配置错误 | 检查PAD属性中上下拉配置，确保默认状态为低电平 |
| LED不闪烁 | main循环逻辑错误 | 检查主循环中LED控制代码是否正确执行 |
