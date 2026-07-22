# 02_ledc — C语言LED点灯实验

## 1. 目录功能概述

本目录存放I.MX6ULL裸机开发工程第二个实验——**C语言实现LED闪烁**的完整代码。

该实验在纯汇编点灯的基础上，引入C语言作为开发主体语言，通过汇编启动文件完成C运行环境初始化（设置栈指针、进入SVC模式），然后跳转至C代码执行。

此实验标志着从纯汇编开发向C语言裸机开发过渡，是学习嵌入式C语言开发的基石。

## 2. 核心源码文件说明

| 文件名 | 类型 | 说明 |
|--------|------|------|
| `start.S` | 汇编启动文件 | 程序入口`_start`，完成SVC模式切换、栈指针设置（`0x80200000`），跳转至`main()`函数 |
| `main.c` | 主程序 | C语言实现时钟使能、GPIO初始化、LED闪烁逻辑 |
| `main.h` | 头文件 | 寄存器地址宏定义及函数声明 |
| `imx6ul.lds` | 链接脚本 | 定义代码段起始地址`0x87800000`及`.text`/`.rodata`/`.data`/`.bss`段布局 |
| `Makefile` | 编译脚本 | 定义多文件编译、链接、二进制转换规则 |
| `imxdownload` | 烧录工具 | SD卡烧写工具 |

## 3. 关键原理简述

### 3.1 C语言运行环境搭建
在`start.S`中，汇编代码完成两项关键初始化：
1. **模式切换**：通过修改`CPSR`寄存器，进入ARM的SVC（Supervisor）模式
2. **栈指针设置**：`ldr sp, =0x80200000`，为C函数的局部变量和函数调用提供栈空间

### 3.2 寄存器宏定义访问
在`main.h`中通过宏定义将寄存器地址映射为可直接访问的C变量：
```c
#define CCM_CCGR0        *((volatile unsigned int *)0x020C4068)
#define GPIO1_GDIR       *((volatile unsigned int *)0x0209C004)
#define GPIO1_DR         *((volatile unsigned int *)0x0209C000)
```
使用`volatile`关键字防止编译器优化，确保每次访问都直接读写寄存器。

### 3.3 GPIO控制逻辑
与汇编实验相同，通过配置IOMUXC复用寄存器、PAD属性寄存器、GPIO方向寄存器和数据寄存器，实现对LED的开关控制。

`main.c`中通过位操作实现LED的周期性闪烁。

### 3.4 延时函数
采用简单的空循环延时函数，在396MHz主频下通过调整循环次数实现约1ms的延时精度。

注意：此延时为粗略延时，实际精度受编译优化和CPU主频影响。

## 4. 编译与构建方式

### 4.1 工具链

- `arm-linux-gnueabihf-gcc`
- `arm-linux-gnueabihf-ld`
- `arm-linux-gnueabihf-objcopy`
- `arm-linux-gnueabihf-objdump`

### 4.2 编译命令
```bash
# 一键编译
make

# 手动编译流程
arm-linux-gnueabihf-gcc -Wall -nostdlib -c -O2 -o start.o start.S
arm-linux-gnueabihf-gcc -Wall -nostdlib -c -O2 -o main.o main.c
arm-linux-gnueabihf-ld -Timx6ul.lds -o ledc.elf start.o main.o
arm-linux-gnueabihf-objcopy -O binary -S ledc.elf ledc.bin
arm-linux-gnueabihf-objdump -D -m arm ledc.elf > ledc.dis
```

### 4.3 Makefile规则说明
- `%.o:%.s` 和 `%.o:%.S`：汇编文件编译规则
- `%.o:%.c`：C文件编译规则，使用`-nostdlib`避免链接标准库
- `-T imx6ul.lds`：使用链接脚本指定代码加载地址

### 4.4 清理构建
```bash
make clean
```

## 5. 下载与验证步骤

1. 编译生成`ledc.bin`
2. 使用`imxdownload`工具将bin文件烧录至SD卡：
```bash
./imxdownload ledc.bin /dev/sdX
```
3. 将SD卡插入开发板，设置拨码开关为SD卡启动模式
4. 上电启动

## 6. 预期运行现象

开发板上电后，**LED0以约1秒为周期进行闪烁**（亮500ms，灭500ms）。

验证标准：LED灯规律性闪烁，说明C语言运行环境搭建成功，GPIO控制逻辑正常。

## 7. 常见问题排查

| 故障现象 | 可能原因 | 解决办法 |
|----------|----------|----------|
| LED不闪烁/常亮 | 栈指针设置错误 | 检查`start.S`中`sp`是否设置为`0x80200000` |
| LED不闪烁/常亮 | 链接脚本地址错误 | 确认`imx6ul.lds`中起始地址为`0x87800000` |
| 编译报错`undefined reference to main` | `main.c`未编译或文件名拼写错误 | 检查`main.c`是否存在，Makefile中obj列表是否正确 |
| 延时时间不准确 | 编译优化导致空循环被优化 | 确保延时函数参数使用`volatile`修饰，或关闭优化编译 |
| 程序卡死 | 栈空间溢出 | 检查栈指针设置地址是否合理，是否有递归调用或大量局部变量 |
