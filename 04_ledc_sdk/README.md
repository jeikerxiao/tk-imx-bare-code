# 04_ledc_sdk — NXP官方SDK风格点灯实验

## 1. 目录功能概述

本目录存放I.MX6ULL裸机开发工程第四个实验——**采用NXP官方SDK（Software Development Kit）寄存器定义方式点亮LED**的完整代码。该实验移植了NXP为I.MX6ULL提供的官方IAR SDK中的寄存器头文件，包括`MCIMX6Y2.h`、`fsl_common.h`、`fsl_iomuxc.h`等，利用官方定义的结构体和宏来操作寄存器。此实验展示了如何使用芯片原厂提供的SDK进行裸机开发，避免了手工定义大量寄存器的繁琐工作，是工程化开发的重要参考。

## 2. 核心源码文件说明

| 文件名 | 类型 | 说明 |
|--------|------|------|
| `start.S` | 汇编启动文件 | 程序入口，完成SVC模式与栈指针初始化 |
| `main.c` | 主程序 | 使用NXP SDK提供的API和宏操作寄存器，实现LED闪烁 |
| `MCIMX6Y2.h` | 头文件（官方SDK） | NXP官方寄存器定义头文件，包含I.MX6Y2所有外设寄存器结构体 |
| `fsl_common.h` | 头文件（官方SDK） | NXP官方通用宏定义和类型定义 |
| `fsl_iomuxc.h` | 头文件（官方SDK） | NXP官方IOMUXC（IO复用控制器）宏定义 |
| `cc.h` | 头文件 | 兼容性头文件，适配GCC编译器环境 |
| `imx6ul.lds` | 链接脚本 | 代码加载地址及段布局定义 |
| `Makefile` | 编译脚本 | 多文件编译构建规则 |
| `imxdownload` | 烧录工具 | SD卡烧写工具 |

## 3. 关键原理简述

### 3.1 NXP SDK寄存器定义体系
NXP SDK采用分层结构定义寄存器：
- **`MCIMX6Y2.h`**：顶层头文件，定义所有外设的结构体类型和基地址指针
- **`fsl_common.h`**：通用类型定义，如`uint32_t`、`status_t`等
- **`fsl_iomuxc.h`**：IOMUXC引脚复用宏定义，提供类似`IOMUXC_GPIO1_IO03_GPIO1_IO03`的语义化宏

### 3.2 SDK风格寄存器访问
```c
// 使用NXP SDK提供的宏和函数
IOMUXC_SetPinMux(IOMUXC_GPIO1_IO03_GPIO1_IO03, 0);      // 设置引脚复用
IOMUXC_SetPinConfig(IOMUXC_GPIO1_IO03_GPIO1_IO03, 0x10B0); // 设置PAD属性
GPIO1->GDIR |= (1 << 3);  // 设置输出方向
GPIO1->DR &= ~(1 << 3);   // 输出低电平点亮LED
```

### 3.3 与手工定义的对比
NXP SDK通过脚本自动生成寄存器定义，覆盖I.MX6ULL全部外设。相比手工定义，具有以下优势：
- **完整性**：涵盖所有寄存器及其位域定义
- **准确性**：直接来自芯片设计文档，避免人工抄录错误
- **可维护性**：芯片更新时只需替换SDK文件

## 4. 编译与构建方式

### 4.1 编译命令
```bash
make
```

### 4.2 Makefile关键说明
- 需确保SDK头文件路径正确包含在编译命令中
- 使用`-nostdlib`避免标准库链接，保持纯裸机环境

### 4.3 清理构建
```bash
make clean
```

## 5. 下载与验证步骤

1. 编译生成`ledc_sdk.bin`
2. 使用`imxdownload`工具烧录至SD卡：
```bash
./imxdownload ledc_sdk.bin /dev/sdX
```
3. SD卡插入开发板，设置SD卡启动模式
4. 上电启动

## 6. 预期运行现象

开发板上电后，**LED0以约1秒为周期进行闪烁**（亮500ms，灭500ms）。

验证标准：LED规律性闪烁，说明NXP SDK寄存器定义正确，官方头文件移植成功。

## 7. 常见问题排查

| 故障现象 | 可能原因 | 解决办法 |
|----------|----------|----------|
| 编译报错`stdint.h: No such file` | `cc.h`中未正确配置GCC兼容层 | 检查`cc.h`中对`stdint.h`的处理，裸机环境需自定义基础类型 |
| 编译报错宏未定义 | 头文件包含顺序错误 | 确保按`fsl_common.h` → `fsl_iomuxc.h` → `MCIMX6Y2.h`的顺序包含 |
| LED不闪烁 | SDK引脚复用宏与实际硬件不符 | 核对开发板原理图与SDK宏定义的对应关系 |
| 警告`implicit declaration` | 缺少函数原型声明 | 检查`main.c`中是否包含所有必要的头文件 |
