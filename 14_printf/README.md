# 14_printf — 串口printf移植实验

## 1. 目录功能概述

本目录存放I.MX6ULL裸机开发工程第十四个实验——**串口printf移植**的完整代码。该实验在串口驱动基础上，将标准C库中的`printf`、`scanf`等格式化输入输出函数移植到裸机环境，通过串口实现格式化打印功能。printf是嵌入式调试中不可或缺的工具，能够极大地提升程序调试效率。

## 2. 核心源码文件说明

| 文件名 | 类型 | 说明 |
|--------|------|------|
| `project/main.c` | 主程序 | 使用printf/scanf实现交互式输入输出演示 |
| `project/start.S` | 汇编启动文件 | 程序入口，初始化C运行环境 |
| `stdio/` | 标准库移植 | 标准C库移植目录，包含`printf`、`scanf`、`sprintf`等函数实现 |
| `bsp/uart/` | 外设驱动 | UART驱动模块 |
| `bsp/delay/` | 公共工具 | 高精度延时模块 |
| `bsp/int/` | 公共工具 | 中断系统管理模块 |
| `bsp/exit/` | 外设驱动 | 外部中断驱动模块 |
| `bsp/key/` | 外设驱动 | 按键驱动模块 |
| `bsp/gpio/` | 公共工具 | 通用GPIO操作封装 |
| `bsp/beep/` | 外设驱动 | 蜂鸣器驱动模块 |
| `bsp/led/` | 外设驱动 | LED驱动模块 |
| `bsp/clk/` | 外设驱动 | 时钟使能与系统时钟配置模块 |
| `imx6ul/` | 头文件集合 | 寄存器定义头文件 |
| `imx6ul.lds` | 链接脚本 | 代码加载地址及段布局 |
| `Makefile` | 编译脚本 | 多目录自动编译构建规则 |
| `imxdownload` | 烧录工具 | SD卡烧写工具 |

## 3. 关键原理简述

### 3.1 printf移植原理
标准C库的`printf`函数依赖底层`_write()`系统调用。在裸机环境中，需将`_write()`重定向到UART发送函数：
```c
int _write(int fd, char *buf, int count) {
    // 将buf中的数据通过UART发送
    for(int i = 0; i < count; i++) {
        uart_putchar(buf[i]);
    }
    return count;
}
```

### 3.2 scanf移植原理
同理，`scanf`依赖底层`_read()`系统调用，需将`_read()`重定向到UART接收函数：
```c
int _read(int fd, char *buf, int count) {
    // 从UART接收数据到buf
    for(int i = 0; i < count; i++) {
        buf[i] = uart_getchar();
    }
    return count;
}
```

### 3.3 标准库裁剪
由于裸机环境无操作系统支持，需裁剪标准C库，只保留必要的格式化函数。正点原子提供的`stdio`目录中包含精简的`printf`、`scanf`、`sprintf`等实现，不依赖完整的标准库。

## 4. 编译与构建方式

### 4.1 编译命令
```bash
make
```

### 4.2 Makefile说明
Makefile中新增了`stdio`目录的路径：
```makefile
INCDIRS := imx6ul stdio/include bsp/clk ...
SRCDIRS := project stdio/lib bsp/clk ...
```

### 4.3 清理构建
```bash
make clean
```

## 5. 下载与验证步骤

1. 编译生成`print.bin`
2. 使用`imxdownload`工具烧录至SD卡：
```bash
./imxdownload print.bin /dev/sdX
```
3. SD卡插入开发板，设置SD卡启动模式
4. 连接USB转TTL至PC串口助手（115200，8N1）
5. 上电启动

## 6. 预期运行现象

开发板上电后：
- 串口助手显示`请输入两个整数，使用空格隔开:`提示
- 输入两个整数后，串口助手显示两数之和
- LED0随每次输入状态翻转

验证标准：格式化输入输出功能正常，说明printf/scanf移植成功。

## 7. 常见问题排查

| 故障现象 | 可能原因 | 解决办法 |
|----------|----------|----------|
| printf无输出 | _write未正确重定向 | 检查`_write`函数是否调用uart_putchar |
| printf输出乱码 | 字符编码问题 | 确保发送的是ASCII码 |
| scanf无响应 | _read未正确重定向 | 检查`_read`函数是否调用uart_getchar |
| 编译报错`undefined reference to printf` | 标准库未链接 | 检查stdio目录是否已加入SRCDIRS |
| 格式化输出异常 | 浮点数格式不支持 | 确认移植的printf是否支持浮点格式化 |
