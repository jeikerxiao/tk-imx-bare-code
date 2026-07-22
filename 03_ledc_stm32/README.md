# 03_ledc_stm32 — STM32风格寄存器操作点灯实验

## 1. 目录功能概述

本目录存放I.MX6ULL裸机开发工程第三个实验——**采用类STM32库风格的寄存器操作方式点亮LED**的完整代码。该实验借鉴STM32微控制器中通过结构体指针访问寄存器的编程习惯，将I.MX6ULL的各组寄存器封装为C语言结构体，使代码更加模块化和可读。此实验帮助有STM32开发经验的工程师快速迁移到I.MX6ULL平台，理解不同ARM平台间寄存器访问的共性与差异。

## 2. 核心源码文件说明

| 文件名 | 类型 | 说明 |
|--------|------|------|
| `start.S` | 汇编启动文件 | 程序入口，初始化SVC模式与栈指针，跳转至C主函数 |
| `main.c` | 主程序 | 采用STM32风格的结构体指针方式操作寄存器，实现LED闪烁 |
| `imx6ul.h` | 头文件 | 核心寄存器结构体定义，包括CCM、GPIO、IOMUXC等外设的结构体封装 |
| `imx6ul.lds` | 链接脚本 | 定义代码加载地址及段布局 |
| `Makefile` | 编译脚本 | 多文件编译构建规则 |
| `imxdownload` | 烧录工具 | SD卡烧写工具 |

## 3. 关键原理简述

### 3.1 STM32风格寄存器封装
`imx6ul.h`中将I.MX6ULL的外设寄存器组封装为C结构体，通过基地址+偏移量的方式访问：
```c
typedef struct {
    volatile unsigned int DR;
    volatile unsigned int GDIR;
    volatile unsigned int PSR;
    // ... 其他寄存器
} GPIO_Type;

#define GPIO1_BASE  0x0209C000
#define GPIO1       ((GPIO_Type *)GPIO1_BASE)
```
访问方式与STM32标准库一致：`GPIO1->DR |= (1<<3);`

### 3.2 结构体与寄存器映射的对应关系
I.MX6ULL的GPIO寄存器在内存中连续排布，通过结构体可以自然地按顺序访问各个寄存器。这种封装方式隐藏了具体的寄存器地址，提高了代码的可移植性和可维护性。

### 3.3 LED控制逻辑
与实验2相同，通过配置IOMUXC复用、PAD属性、GPIO方向和数据寄存器实现LED控制。不同之处在于使用结构体指针语法访问寄存器，代码风格更接近STM32开发习惯。

## 4. 编译与构建方式

### 4.1 编译命令
```bash
make
```

### 4.2 Makefile关键参数
- `arm-linux-gnueabihf-gcc -Wall -nostdlib -c -O2`：编译选项与实验2一致
- 链接脚本`imx6ul.lds`定义代码从`0x87800000`开始加载

### 4.3 清理构建
```bash
make clean
```

## 5. 下载与验证步骤

1. 编译生成`ledc_stm32.bin`
2. 使用`imxdownload`工具烧录至SD卡：
```bash
./imxdownload ledc_stm32.bin /dev/sdX
```
3. SD卡插入开发板，设置SD卡启动模式
4. 上电启动

## 6. 预期运行现象

开发板上电后，**LED0以约1秒为周期进行闪烁**（亮500ms，灭500ms）。

验证标准：LED规律性闪烁，说明通过结构体指针方式访问寄存器成功，程序逻辑正常执行。

## 7. 常见问题排查

| 故障现象 | 可能原因 | 解决办法 |
|----------|----------|----------|
| LED不闪烁 | 结构体寄存器顺序错误 | 检查`imx6ul.h`中结构体成员顺序是否与参考手册寄存器偏移一致 |
| LED不闪烁 | 基地址定义错误 | 核对各外设基地址宏定义是否正确 |
| 编译警告`type defaults to int` | 函数声明缺失 | 确保所有函数在使用前都有声明，或统一在头文件中声明 |
| 从STM32迁移后LED不亮 | I.MX6ULL GPIO编号与STM32不同 | 注意I.MX6ULL的GPIO引脚命名方式为`GPIO1_IOxx`，而非STM32的`PAxx` |
