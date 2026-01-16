---
title: LVGL学习之旅（基于正点原子STM32F407探索者）
date: 2025-09-07 14:58:25
cover: "https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071514562.png"
tags:
 - LVGL v8.2
 - RTOS
 - 嵌入式基础
---

## LVGL 基础

> LVGL（Light and Versatile Graphics Library）轻量级通用型图形库，是一个免费的开源图形库，提供了创建嵌入式 GUI 所需的一切，具有易于使用的组件，美观的视觉效果和低内存占用等特点。支持触摸屏操作，移植简单方便，开发者一直在不断完善更新。
>
> 特点：
>
> - 丰富且强大的模块化图形组件：按钮 (buttons)、图表 (charts)、列表 (lists)、滑动条 (sliders)、图片 (images) 等
> - 高级的图形引擎：动画、抗锯齿、透明度、平滑滚动、图层混合等效果
> - 不依赖特定的硬件平台，可以在任何显示屏上运行
> - 配置可裁剪（最低资源占用：<span style="color:#FF0000;">64 kB Flash，16 kB RAM</span>）
> - 基于<span style="color:#FF0000;">UTF-8</span>的多语种支持，例如中文、日文、韩文、阿拉伯文等
> - 可以通过类CSS的方式来设计、布局图形界面（例如：Flexbox、Grid）
> - 支持操作系统、外置内存、以及硬件加速（LVGL已内建支持STM32 DMA2D、NXP PXP和VGLite）
> - 即便仅有单缓冲区(frame buffer)的情况下，也可保证渲染如丝般顺滑
> - 全部由C编写完成，并支持C++调用
> - 支持Micropython编程，参见：LVGL API in Micropython
> - 支持模拟器仿真，可以无硬件依托进行开发

## TFT-LCD配置——移植正点原子官方LCD例程

### 点亮LCD——FSMC配置

#### 先根据原理图配置`FSMC`

![image-20250907155616208](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071556284.png)

![image-20250907160758201](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071607256.png)

#### 详细FSMC参数配置

> 这里根据正点原子官方`lcd_init` 来设置 如下图

![image-20250907164335816](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071643877.png)

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071724936.png)

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071725408.png)



#### 背光引脚配置

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071725075.png)

#### 生成代码并将LCD、sys、delay文件夹添加到工程中

![image-20250907234315989](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509072343199.png)

##### 首先将*HAL_SRAM_MspInit*函数整个屏蔽

![image-20250907171842072](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071718133.png)

##### 将lcd_init中有关fsmc部分屏蔽

> 屏蔽部分如下

```C
void LCD_Init(void)
{ 	  
//	GPIO_InitTypeDef GPIO_Initure;
//	FSMC_NORSRAM_TimingTypeDef FSMC_ReadWriteTim;
//	FSMC_NORSRAM_TimingTypeDef FSMC_WriteTim;
//    
//	__HAL_RCC_GPIOB_CLK_ENABLE();			//开启GPIOB时钟
//	GPIO_Initure.Pin=GPIO_PIN_15;          	//PB15,背光控制
//	GPIO_Initure.Mode=GPIO_MODE_OUTPUT_PP;  //推挽输出
//	GPIO_Initure.Pull=GPIO_PULLUP;          //上拉
//	GPIO_Initure.Speed=GPIO_SPEED_HIGH;     //高速
//	HAL_GPIO_Init(GPIOB,&GPIO_Initure); 
//	
//	TFTSRAM_Handler.Instance=FSMC_NORSRAM_DEVICE;                
//	TFTSRAM_Handler.Extended=FSMC_NORSRAM_EXTENDED_DEVICE;    
//    
//	TFTSRAM_Handler.Init.NSBank=FSMC_NORSRAM_BANK4;     				//使用NE4
//	TFTSRAM_Handler.Init.DataAddressMux=FSMC_DATA_ADDRESS_MUX_DISABLE; 	//地址/数据线不复用
//	TFTSRAM_Handler.Init.MemoryType=FSMC_MEMORY_TYPE_SRAM;   			//SRAM
//	TFTSRAM_Handler.Init.MemoryDataWidth=FSMC_NORSRAM_MEM_BUS_WIDTH_16; //16位数据宽度
//	TFTSRAM_Handler.Init.BurstAccessMode=FSMC_BURST_ACCESS_MODE_DISABLE; //是否使能突发访问,仅对同步突发存储器有效,此处未用到
//	TFTSRAM_Handler.Init.WaitSignalPolarity=FSMC_WAIT_SIGNAL_POLARITY_LOW;//等待信号的极性,仅在突发模式访问下有用
//	TFTSRAM_Handler.Init.WaitSignalActive=FSMC_WAIT_TIMING_BEFORE_WS;   //存储器是在等待周期之前的一个时钟周期还是等待周期期间使能NWAIT
//	TFTSRAM_Handler.Init.WriteOperation=FSMC_WRITE_OPERATION_ENABLE;    //存储器写使能
//	TFTSRAM_Handler.Init.WaitSignal=FSMC_WAIT_SIGNAL_DISABLE;           //等待使能位,此处未用到
//	TFTSRAM_Handler.Init.ExtendedMode=FSMC_EXTENDED_MODE_ENABLE;        //读写使用不同的时序
//	TFTSRAM_Handler.Init.AsynchronousWait=FSMC_ASYNCHRONOUS_WAIT_DISABLE;//是否使能同步传输模式下的等待信号,此处未用到
//	TFTSRAM_Handler.Init.WriteBurst=FSMC_WRITE_BURST_DISABLE;           //禁止突发写
//	TFTSRAM_Handler.Init.ContinuousClock=FSMC_CONTINUOUS_CLOCK_SYNC_ASYNC;
//    
//	//FMC读时序控制寄存器
//	FSMC_ReadWriteTim.AddressSetupTime=0x0F;       	//地址建立时间（ADDSET）为16个HCLK 1/168M=6ns*16=96ns
//	FSMC_ReadWriteTim.AddressHoldTime=0;
//	FSMC_ReadWriteTim.DataSetupTime=60;				//数据保存时间为60个HCLK	=6*60=360ns
//	FSMC_ReadWriteTim.AccessMode=FSMC_ACCESS_MODE_A;//模式A
//	//FMC写时序控制寄存器
//	FSMC_WriteTim.BusTurnAroundDuration=0;			//总线周转阶段持续时间为0，此变量不赋值的话会莫名其妙的自动修改为4。导致程序运行正常
//	FSMC_WriteTim.AddressSetupTime=9;          		//地址建立时间（ADDSET）为9个HCLK =54ns 
//	FSMC_WriteTim.AddressHoldTime=0;
//	FSMC_WriteTim.DataSetupTime=8;              	//数据保存时间为6ns*9个HCLK=54n
//	FSMC_WriteTim.AccessMode=FSMC_ACCESS_MODE_A;    //模式A
//	HAL_SRAM_Init(&TFTSRAM_Handler,&FSMC_ReadWriteTim,&FSMC_WriteTim);	

//	HAL_Delay(50); // delay 50 ms 
	
	//尝试9341 ID的读取		
	LCD_WR_REG(0XD3);		
    .........
}
```

> 注意有个坑！！！！
>
> 正点原子官方的`lcd_init`函数中有个 `printf(" LCD ID:%x\r\n",lcddev.id);`，所以在此之前需先进行串口重定向  

##### 替换官方的delay函数

```C
void delay_us(u32 nus)
{
	uint32_t Delay = nus * 168 / 4;
	do
	{
		__NOP();
	} while (Delay--);
}

// 延时nms
// nms:要延时的ms数
void delay_ms(u16 nms)
{
	HAL_Delay(nms);
}
```



##### 修改点亮背光函数

> 在`lcd_init`函数最后 将`LCD_BL(1)`替换为`HAL_GPIO_WritePin(LCD_BL_GPIO_Port, LCD_BL_Pin, GPIO_PIN_SET);`

### 触摸屏TOUCH移植

#### 将BSP文件夹下的24CXX、IIC与TOUCH文件夹添加到项目中

![image-20250907180027576](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071800663.png)

## LVGL移植

### LVGL源码下载

- 国内码云仓库：https://gitee.com/mirrors/lvgl

  ![image-20250907180958855](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071809967.png)

![image-20250907181022271](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071810376.png)

> 注意不要直接克隆 直接克隆下载的是最新的 可以选择右上角下载zip 就可以选中当前分支

### 裁剪LVGL源码

#### 根目录

> 仅保留 demos、examples、src文件夹以及两个.h文件 如下图

![image-20250907181309155](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071813234.png)

#### 进入`examples`文件夹 删去除`porting`文件夹

![image-20250907181406209](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071814287.png)

#### 更改`lv_conf_template.h`

> 1. 将`lv_conf_template.h`更名为`lv_conf.h`
> 2. 进入该文件 将里面的#if 0 改为#if 1
>
> ![image-20250907182017202](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071820299.png)

### 按以下结构创建文件夹

```C
Project
----Middlewares
--------LVGL
------------GUI
----------------lvgl
------------GUI_APP
```

> 将裁剪后的LVGL库文件复制到`Middlewares/LVGL/GUI/lvgl`文件夹下

### Keil配置

#### 创建如下管理

![image-20250907194802955](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071948050.png)

#### 添加路径

![image-20250907194859063](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071948156.png)

> - 注意一定要开启C99模式
>
> ![image-20250907195152454](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071951544.png)
>
> - 可以屏蔽MDK警告
>
>   --diag_suppress=68 --diag_suppress=111 --diag_suppress=188 --diag_suppress=223 --diag_suppress=546  --diag_suppress=1295
>
> 

#### 添加文件到项目

![image-20250907195111490](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071951610.png)

### LVGL文件修改

#### 配置LCD屏幕显示

![image-20250907195420988](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071954102.png)

##### 包含输出设备驱动头文件

> 在`lv_port_disp_template.c`引入`lcd.h` 如`#include "../BSP/LCD/lcd.h"`

##### 在`disp_init`函数中初始化

```C
static void disp_init(void)
{
    /*You code here*/
    LCD_Init();
    LCD_Display_Dir(1); // 横屏
}
```

##### 配置图形数据缓冲方式

> 进入`lv_port_disp_init`函数
>
> 选择缓冲方式 另外两个将其屏蔽
>
> ![image-20250907195842896](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071958994.png)

##### 修改`disp_flush`函数

```C
static void disp_flush(lv_disp_drv_t *disp_drv, const lv_area_t *area, lv_color_t *color_p)
{
    /*The most simple case (but also the slowest) to put all pixels to the screen one-by-one*/
    /*官方打点函数 */
    // int32_t x;
    // int32_t y;
    // for (y = area->y1; y <= area->y2; y++)
    // {
    //     for (x = area->x1; x <= area->x2; x++)
    //     {
    //         /*Put a pixel to the display. For example:*/
    //         /*put_px(x, y, *color_p)*/
    //         color_p++;
    //     }
    // }

    LCD_Color_Fill(area->x1, area->y1, area->x2, area->y2, (uint16_t *)color_p);
    /*IMPORTANT!!!
     *Inform the graphics library that you are ready with the flushing*/
    lv_disp_flush_ready(disp_drv);
}
```

#### 配置输入（触摸）

![image-20250907200123083](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509072001216.png)

##### 按需裁剪输入设备

> 打开`lv_port_indev_template.c` 此处我们只需要触摸输入 所以把除了touchpad 其他都屏蔽或删去
>
> 注意：一定要把其他无关的屏蔽掉 否则会无法触摸

##### 在`touchpad_init`函数中初始化触摸屏

```C
static void touchpad_init(void)
{
    /*Your code comes here*/
    tp_dev.init();
}
```

##### 修改`touchpad_is_pressed`函数

```C
static bool touchpad_is_pressed(void)
{
    /*Your code comes here*/
    tp_dev.scan(0); // É¨ÃèÎïÀí×ø±ê
    if (tp_dev.sta & TP_PRES_DOWN)
    {
        return true;
    }
    return false;
}
```

##### 配置坐标获取函数

```C
static void touchpad_get_xy(lv_coord_t *x, lv_coord_t *y)
{
    /*Your code comes here*/

    (*x) = tp_dev.x[0];
    (*y) = tp_dev.y[0];
}
```

#### 提供时基

> - 使用CubeMX生成一个定时1ms的定时器
> - 在`tim.c`中引用`#include "lvgl.h"`
> - 在定时器中断回调函数中`lv_tick_inc(1);`

![image-20250907201034944](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509072010067.png)

![image-20250907200845233](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509072008331.png)

#### `main`函数部分

![image-20250907201115783](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509072011915.png)

```C
#include "lvgl.h"
#include "lv_port_disp_template.h"
#include "lv_port_indev_template.h"
....
int main(void)
{
    .....
  HAL_TIM_Base_Start_IT(&htim3); // 启动定时器3中断 为lvgl提供时基
  lv_init();                     // 初始化LVGL
  lv_port_disp_init();           // 初始化显示接口
  lv_port_indev_init();          // 初始化输入接口
  while (1)
  {
    lv_timer_handler(); // 处理LVGL任务
    delay_ms(5);
  }
}
```

