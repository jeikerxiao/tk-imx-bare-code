# 05_ledc_bsp — BSP模块化LED驱动实验

## 1. 目录功能概述

本目录存放I.MX6ULL裸机开发工程第五个实验——**BSP（Board Support Package）模块化驱动方式的LED控制**的完整代码。
该实验将LED驱动、时钟使能、延时函数等功能封装为独立的BSP模块，每个模块包含`.c`实现和`.h`接口头文件，通过Makefile统一管理编译和链接。
此实验标志着从单一文件开发向模块化、工程化开发模式的转变，是构建大型嵌入式项目的基础。

## 2. 核心源码文件说明

| 文件名 | 类型 | 说明 |
|--------|------|------|
| `project/main.c` | 主程序 | 调用各BSP模块接口实现LED闪烁功能 |
| `project/start.S` | 汇编启动文件 | 程序入口，初始化C运行环境 |
| `bsp/clk/` | 外设驱动 | 时钟使能模块：`bsp_clk.c`、`bsp_clk.h` |
| `bsp/led/` | 外设驱动 | LED驱动模块：`bsp_led.c`、`bsp_led.h` |
| `bsp/delay/` | 公共工具 | 延时函数模块：`bsp_delay.c`、`bsp_delay.h` |
| `imx6ul/` | 头文件集合 | 包含`MCIMX6Y2.h`、`imx6ul.h`、`cc.h`等寄存器定义头文件 |
| `imx6ul.lds` | 链接脚本 | 代码加载地址及段布局 |
| `Makefile` | 编译脚本 | 高级通用Makefile，自动搜索多目录源文件并编译 |
| `imxdownload` | 烧录工具 | SD卡烧写工具 |

## 3. 关键原理简述

### 3.1 BSP模块化设计思想
将功能拆分为独立模块，每个模块职责单一：
- **bsp_clk**：负责时钟使能，提供`clk_enable()`接口
- **bsp_led**：负责LED初始化与控制，提供`led_init()`、`led_on()`、`led_off()`等接口
- **bsp_delay**：负责延时，提供`delay()`接口

### 3.2 通用Makefile设计
本实验Makefile采用高级技巧实现自动化的多目录编译：
- 使用`INCDIRS`和`SRCDIRS`变量定义头文件和源文件搜索路径
- 使用`VPATH`实现自动依赖查找
- 使用`obj/`目录存放编译生成的`.o`文件，保持源码目录整洁
- 支持任意数量的BSP模块扩展

### 3.3 抽象层设计
每个BSP模块提供统一的函数接口，上层应用（`main.c`）只需调用接口函数，无需关心底层寄存器操作细节。这种设计符合软件工程中的"高内聚、低耦合"原则。

## 4. 编译与构建方式

### 4.1 编译命令
```bash
make
```

### 4.2 Makefile特点
```makefile
INCDIRS := imx6ul bsp/clk bsp/led bsp/delay
SRCDIRS := project bsp/clk bsp/led bsp/delay
```
- 新增BSP模块时，只需在`INCDIRS`和`SRCDIRS`中追加路径即可
- 自动搜索所有目录下的`.c`和`.S`文件并编译

### 4.3 清理构建
```bash
make clean
```

## 5. 下载与验证步骤

1. 编译生成`bsp.bin`
2. 使用`imxdownload`工具烧录至SD卡：
```bash
./imxdownload bsp.bin /dev/sdX
```
3. SD卡插入开发板，设置SD卡启动模式
4. 上电启动

## 6. 预期运行现象

开发板上电后，**LED0以约1秒为周期进行闪烁**（亮500ms，灭500ms）。

验证标准：LED规律性闪烁，说明BSP模块化驱动架构工作正常，各模块接口调用无误。

## 7. 常见问题排查

| 故障现象 | 可能原因 | 解决办法 |
|----------|----------|----------|
| 编译报错`bsp_led.h: No such file` | 头文件搜索路径未配置 | 检查Makefile中`INCDIRS`是否包含`bsp/led`路径 |
| 链接报错`undefined reference to led_init` | `bsp_led.c`未编译或obj文件缺失 | 检查`SRCDIRS`是否包含`bsp/led`路径 |
| LED控制无响应 | BSP模块中寄存器地址错误 | 核对各BSP模块中的寄存器地址定义 |
| obj目录不存在导致编译失败 | Makefile未创建obj目录 | 检查Makefile中是否有创建`obj`目录的规则，或手动创建 |
