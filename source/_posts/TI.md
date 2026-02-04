---
title: TI
date: 2025-05-28 20:40:37
cover: "https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505282044781.jpg"
tags:
  - TI
categories: MCU
---

## 初始化代码

### 浮点单元（FPU）相关配置

```C
ROM_FPUEnable(); / 使能浮点单元
ROM_FPULazyStackingEnable(); / 启用浮点延迟堆栈模式，减少中断响应延迟    
```

**浮点单元（FPU）**

- 浮点单元是 Cortex-M4 芯片（如 Tiva-C）内的硬件模块，用于加速浮点运算（如浮点加法、乘法）。

- 如果你的项目中使用了`带小数`的计算（如定点PID控制、FFT计算、浮点除法），硬件 FPU 的加速能力远远优于软件模拟。

**Lazy Stacking 模式**

- 启用延迟堆栈推入操作后，在中断期间，只有确实需要使用浮点硬件时，才会将浮点寄存器保存到堆栈，从而减少中断响应时间和堆栈占用空间。

### **实际开发中怎么使用和修改**

#### ?  启用还是禁用  FPU：

- 如果项目中`不涉及浮点运算`，可以省略这两个函数的调用以节省资源

#### 启用延迟堆栈的条件：

- 如果中断中涉及浮点运算，建议`启用 Lazy Stacking` 优化性能。

- 但需要注意堆栈深度：<span style="text-decoration:line-through;">中断嵌套可能导致浮点堆栈溢出</span>，调试时<span style="color:#66FFFF;">增加栈尺寸</span>。

## 时钟系统配置

```C
ROM_SysCtlClockSet(SYSCTL_SYSDIV_2_5 | SYSCTL_USE_PLL | SYSCTL_XTAL_16MHZ | SYSCTL_OSC_MAIN); / 配置系统时钟
```

- <span style="color:#3399FF;">参数说明</span>：

1. `SYSCTL_SYSDIV_2_5`：
   - 设置系统分频比， 2.5 是分频因子。
   - 假设外部晶振频率为 16MHz，PLL 经过倍频后输出为 400MHz， 2.5 分频下得到时钟频率：400 ÷ 2.5 = 80MHz 。

2. `SYSCTL_USE_PLL`：
   - 指定使用 PLL（锁相环）作为系统时钟的倍频源。

3. `SYSCTL_XTAL_16MHZ`：
   - 配置外部晶振的规格为 16MHz，这需要与硬件实际晶振频率相匹配。

4. `SYSCTL_OSC_MAIN`：
   - 指定使用外部主振荡器作为系统的时钟输入源。

### 实际开发中怎么使用和修改

#### **如何修改分频比和主频**：

- 如果需要更改 MCU 的主频（系统时钟频率），只需调整分频因子：

  ```C
  ROM_SysCtlClockSet(SYSCTL_SYSDIV_4 | SYSCTL_USE_PLL | SYSCTL_XTAL_16MHZ | SYSCTL_OSC_MAIN);
  ```

  示例中，分频比改为 4 ，主频变为 400 ÷ 4 = 100MHz 。

#### 如果硬件使用其他晶振

- 如果使用 8MHz 的晶振，应修改 `SYSCTL_XTAL_16MHZ` 为 `SYSCTL_XTAL_8MHZ` ：

  ```C
  ROM_SysCtlClockSet(SYSCTL_SYSDIV_2_5 | SYSCTL_USE_PLL | SYSCTL_XTAL_8MHZ | SYSCTL_OSC_MAIN);
  ```

#### 切换时钟源

- 如果不使用外部晶振，可以切换为 **内部振荡器** (Precision Internal Oscillator, PIOSC)：

  ```C
  ROM_SysCtlClockSet(SYSCTL_SYSDIV_2_5 | SYSCTL_USE_PLL | SYSCTL_OSC_INT);
  ```

## 外设（GPIO）启用和配置

```C
ROM_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF); / 启用 GPIOF 外设
ROM_GPIOPinTypeGPIOOutput(GPIO_PORTF_BASE, GPIO_PIN_5); / 配置 GPIOF 5 为推挽输出
ROM_GPIOPinTypeGPIOOutput(GPIO_PORTF_BASE, GPIO_PIN_6); / 配置 GPIOF 6 为推挽输出
ROM_GPIOPinTypeGPIOOutput(GPIO_PORTF_BASE, GPIO_PIN_4); / 配置 GPIOF 4 为推挽输出
```

#### **GPIO 配置解析**

1. **使能时钟**：

   ```C
   ROM_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF);
   ```

- 使能`GPIOF`外设的时钟
- 每个外设模块在使用前都必须先使能其时钟，否则无法访问寄存器

2. **配置推挽输出**：

   ```C
   ROM_GPIOPinTypeGPIOOutput(GPIO_PORTF_BASE, GPIO_PIN_X);
   ```

- 将`GPIOF`的某个引脚配置为`推挽输出`

3. **配置输入模式**

   ```C
   ROM_GPIOPinTypeGPIOInput(GPIO_PORTF_BASE, GPIO_PIN_5);
   ```

- 将`PF5`引脚配置为输入

### 串口代码

```C
ROM_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOA); / 启用 GPIOA 外设
ROM_SysCtlPeripheralEnable(SYSCTL_PERIPH_UART0); / 启用 UART0 外设
```

#### 配置UART引脚模式

```C
ROM_GPIOPinConfigure(GPIO_PA0_U0RX); / 配置 GPIOA 引脚 0 为 UART0 RX
ROM_GPIOPinConfigure(GPIO_PA1_U0TX); / 配置 GPIOA 引脚 1 为 UART0 TX
ROM_GPIOPinTypeUART(GPIO_PORTA_BASE, GPIO_PIN_0 | GPIO_PIN_1); / 配置 GPIO PA0 和 PA1 为UART 模式
```

- <span style="color:#3399FF;">参数说明</span>：

1. `ROM_GPIOPinConfigure(pinConfig)`：
   - 将某个 GPIO 引脚映射为对应外设的功能模式，例如 UART RX（接收）或 UART TX（发送）模式
   - `GPIO_PA0_U0RX`和 `GPIO_PA1_U0TX` 分别表示将 GPIOA 的 0 和 1 引脚映射到 UART0 的 RX 和 TX信号。

2. `ROM_GPIOPinTypeUART()`：
   - 配置对应引脚为 UART 模式（主功能），指示这些引脚负责 UART 的输入和输出。

3. **引脚复用说明**：

   - 某些 GPIO 引脚具备多种功能（如 UART、SPI、I2C 等），可以通过 `ROM_GPIOPinConfigure() `设置引脚的复用功能。

   - 数据手册中会列出默认出厂配置（如 GPIOA0、GPIOA1 默认是普通 I/O）以及复用编号

#### 配置UART0模块

```C
UARTStdioConfig(0, 115200, 16000000); / 配置 UART0，设置波特率为 115200，时钟频率为 16MHz
UARTClockSourceSet(UART0_BASE, UART_CLOCK_PIOSC); / 将 UART0 的时钟源设为内部精确振荡器PIOSC
```

`UARTStdioConfig() 详解：`

- 用于配置 UART 模块的一些核心参数，例如 UART 通信模块的编号、默认的通信波特率、UART 时钟频率。

1. <span style="color:#3399FF;">参数说明</span>：

- `0` ：指定 UART 模块编号， 0 对应 UART0。
- `115200` ：波特率，表示每秒传输 115200 个比特。
- `16000000` ：UART 输入时钟频率（以 Hz 为单位），这里指定为 16MHz。

2. 作用：
   - 根据设置的波特率和时钟频率， `UARTStdioConfig` 会自动计算并配置 UART 波特率控制寄存器，从而实现正确的通信速率。
3. **默认时钟源（**PIOSC **内部振荡器）**

```C
UARTClockSourceSet(UART0_BASE, UART_CLOCK_PIOSC); / 使用 PIOSC（16 MHz）作为时钟源
```

- PIOSC 是 Tiva-C 系统内置的精确振荡器，频率为 16 MHz。
- 替代方案：你可以改用系统主时钟（如 PLL）作为 UART 的时钟源。

