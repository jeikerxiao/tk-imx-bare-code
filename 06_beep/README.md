# 06_beep — 蜂鸣器驱动实验

## 1. 目录功能概述

本目录存放I.MX6ULL裸机开发工程第六个实验——**蜂鸣器驱动**的完整代码。该实验在BSP模块化架构基础上，新增蜂鸣器（BEEP）驱动模块，通过GPIO控制有源蜂鸣器的开关。实验目标是让读者掌握BSP模块的扩展方法，学会新增一个外设驱动的完整流程，包括头文件定义、源文件实现、Makefile路径配置及主程序调用。

## 2. 核心源码文件说明

| 文件名 | 类型 | 说明 |
|--------|------|------|
| `project/main.c` | 主程序 | 调用bsp_beep和bsp_led接口，实现LED与蜂鸣器同步闪烁 |
| `project/start.S` | 汇编启动文件 | 程序入口，初始化C运行环境 |
| `bsp/beep/` | 外设驱动 | 蜂鸣器驱动模块：`bsp_beep.c`、`bsp_beep.h` |
| `bsp/clk/` | 外设驱动 | 时钟使能模块 |
| `bsp/led/` | 外设驱动 | LED驱动模块 |
| `bsp/delay/` | 公共工具 | 延时函数模块 |
| `imx6ul/` | 头文件集合 | 寄存器定义头文件 |
| `imx6ul.lds` | 链接脚本 | 代码加载地址及段布局 |
| `Makefile` | 编译脚本 | 多目录自动编译构建规则 |
| `imxdownload` | 烧录工具 | SD卡烧写工具 |

## 3. 关键原理简述

### 3.1 蜂鸣器硬件连接
开发板上蜂鸣器为有源蜂鸣器，控制引脚连接至GPIO1_IO19。有源蜂鸣器只需提供高低电平即可控制发声与静音：
- **高电平**：蜂鸣器发声
- **低电平**：蜂鸣器静音

### 3.2 GPIO配置流程
蜂鸣器引脚配置与LED引脚配置类似，需完成以下步骤：
1. **时钟使能**：通过CCM使能GPIO1时钟
2. **IO复用配置**：将GPIO1_IO19复用为GPIO功能（MUX_MODE = 5）
3. **PAD属性配置**：配置驱动能力、上下拉等电气属性
4. **方向设置**：将GPIO1_IO19设置为输出模式
5. **输出控制**：输出高电平驱动蜂鸣器发声，输出低电平停止发声

### 3.3 BSP模块扩展方法
新增蜂鸣器驱动模块的流程：
1. 在`bsp/`目录下创建`beep`文件夹
2. 编写`bsp_beep.h`头文件，声明初始化与控制函数
3. 编写`bsp_beep.c`源文件，实现寄存器操作
4. 在`Makefile`的`INCDIRS`和`SRCDIRS`中追加`bsp/beep`路径
5. 在`main.c`中调用`bsp_beep.h`接口

## 4. 编译与构建方式

### 4.1 编译命令
```bash
make
```

### 4.2 Makefile更新说明
相比实验5，Makefile中新增了蜂鸣器模块的路径：
```makefile
INCDIRS := imx6ul bsp/clk bsp/led bsp/delay bsp/beep
SRCDIRS := project bsp/clk bsp/led bsp/delay bsp/beep
```

### 4.3 清理构建
```bash
make clean
```

## 5. 下载与验证步骤

1. 编译生成`beep.bin`
2. 使用`imxdownload`工具烧录至SD卡：
```bash
./imxdownload beep.bin /dev/sdX
```
3. SD卡插入开发板，设置SD卡启动模式
4. 上电启动

## 6. 预期运行现象

开发板上电后，**LED0和蜂鸣器同步以1秒为周期交替开关**（LED亮/蜂鸣器响500ms，LED灭/蜂鸣器停500ms）。

验证标准：LED与蜂鸣器同步周期性变化，说明蜂鸣器BSP模块扩展成功，GPIO输出控制正常。

## 7. 常见问题排查

| 故障现象 | 可能原因 | 解决办法 |
|----------|----------|----------|
| 蜂鸣器不响 | GPIO引脚复用错误 | 确认蜂鸣器控制引脚为GPIO1_IO19，复用模式为GPIO（MUX_MODE=5） |
| 蜂鸣器长鸣 | 默认输出电平为高电平 | 检查初始化时是否将GPIO输出设置为低电平 |
| 编译报错找不到bsp_beep.h | Makefile路径未更新 | 检查INCDIRS和SRCDIRS是否包含bsp/beep路径 |
| 蜂鸣器声音异常 | 驱动能力不足 | 检查PAD属性配置，确保驱动能力设置正确 |
