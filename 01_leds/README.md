# 01_leds — 汇编语言LED点灯实验

## 1. 目录功能概述

本目录存放I.MX6ULL裸机开发工程第一个实验——**纯汇编语言点亮LED灯**的完整代码与构建产物。
该实验属于裸机入门级别，核心目标是让读者在没有操作系统、没有C语言运行时环境的前提下，通过ARM汇编直接操作I.MX6ULL处理器的时钟控制与GPIO寄存器，完成LED灯的点亮与熄灭控制。
此实验为后续C语言开发、BSP工程化开发奠定最底层的硬件操作认知基础。

## 2. 核心源码文件说明

| 文件名 | 类型 | 说明 |
|--------|------|------|
| `led.s` | 汇编启动文件 | 核心源码，包含全局入口标号`_start`，完成时钟使能、GPIO复用配置、方向设置及点灯逻辑 |
| `Makefile` | 编译脚本 | 定义汇编编译、链接、二进制转换及反汇编规则 |
| `imxdownload` | 烧录工具 | 正点原子提供的SD卡烧写工具，将bin文件转换成.imx镜像并写入SD卡 |
| `imx6ul.lds` | 链接脚本 | 无（本实验通过`-Ttext`直接指定地址） |
| `led.bin` / `led.elf` / `led.dis` | 构建产物 | 编译生成的二进制文件、ELF文件及反汇编文件 |
| `load.imx` | 烧录镜像 | 通过`imxdownload`生成的SD卡启动镜像 |

## 3. 关键原理简述

### 3.1 时钟使能
I.MX6ULL上电后内部Boot ROM将主频初始化为396MHz。为简化操作，本实验通过向CCM（Clock Controller Module）的`CCGR0`~`CCGR6`寄存器写入`0xFFFFFFFF`，一次性使能所有外设时钟。

### 3.2 GPIO配置流程
开发板LED0连接至GPIO1_IO03，需完成以下寄存器配置：
1. **IO复用**：向`IOMUXC_SW_MUX_CTL_PAD_GPIO1_IO03`（地址`0x020E0068`）写入`0x5`，将引脚复用为GPIO功能
2. **PAD属性**：向`IOMUXC_SW_PAD_CTL_PAD_GPIO1_IO03`（地址`0x020E02F4`）写入`0x10B0`，配置驱动能力、上下拉、速度等电气属性
3. **方向设置**：向`GPIO1_GDIR`（地址`0x0209C004`）对应bit置1，配置为输出模式
4. **输出控制**：向`GPIO1_DR`（地址`0x0209C000`）对应bit写0，输出低电平点亮LED（共阴极接法）

### 3.3 汇编级实现
代码通过ARM汇编的`ldr`/`str`指令直接对寄存器地址进行读写，无任何库函数封装，最直观地体现了裸机寄存器操作的本质。

## 4. 编译与构建方式

### 4.1 工具链
- **编译器**：`arm-linux-gnueabihf-gcc`
- **链接器**：`arm-linux-gnueabihf-ld`
- **格式转换**：`arm-linux-gnueabihf-objcopy`
- **反汇编**：`arm-linux-gnueabihf-objdump`

### 4.2 编译命令
```bash
# 使用Makefile一键编译
make

# 等价于手动执行：
arm-linux-gnueabihf-gcc -g -c led.s -o led.o
arm-linux-gnueabihf-ld -Ttext 0X87800000 led.o -o led.elf
arm-linux-gnueabihf-objcopy -O binary -S -g led.elf led.bin
arm-linux-gnueabihf-objdump -D led.elf > led.dis
```

### 4.3 清理构建
```bash
make clean
```

## 5. 下载与验证步骤

### 5.1 生成SD卡启动镜像
```bash
./imxdownload led.bin /dev/sdX
```
> 注意：`/dev/sdX`替换为实际SD卡设备节点（如`/dev/sdb`），请务必确认设备正确，避免误写硬盘。

### 5.2 开发板操作流程
1. 将SD卡插入电脑，使用`imxdownload`工具将`led.bin`烧录至SD卡
2. 将SD卡插入正点原子I.MX6ULL ALPHA开发板SD卡槽
3. 拨码开关设置为SD卡启动模式（通常设置为：`1-OFF, 2-ON, 3-OFF, 4-OFF`等，请参考开发板手册）
4. 上电启动

## 6. 预期运行现象

开发板上电后，**LED0（红色）常亮**。

由于本实验仅实现点灯，无延时闪烁逻辑，LED灯上电即保持点亮状态，无其他动态效果。

验证标准：上电瞬间LED0立即点亮，说明汇编代码已正确执行，GPIO配置成功。

## 7. 常见问题排查

| 故障现象 | 可能原因 | 解决办法 |
|----------|----------|----------|
| LED不亮 | 拨码开关未设置为SD卡启动 | 检查拨码开关设置，确保SD卡启动模式正确 |
| LED不亮 | `imxdownload`烧录失败 | 确认SD卡设备节点正确，检查烧录权限（需root或sudo） |
| LED不亮 | 寄存器地址写错 | 核对`led.s`中所有寄存器地址是否与IMX6ULL参考手册一致 |
| 编译报错`arm-linux-gnueabihf-gcc: command not found` | 交叉编译器未安装或环境变量未配置 | 安装交叉编译工具链，确保`bin`目录在PATH中 |
| 反汇编文件为空 | 编译顺序错误 | 确保先编译生成`.o`文件，再链接生成`.elf` |
