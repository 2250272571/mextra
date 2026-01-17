---
title: freertos
date: 2025-09-05 15:48:39
sticky: true
cover: "https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071511987.png"
tags:
  - 实时操作系统
  - RTOS
  - FreeRTOS
layout: page
categories: MCU
---

## FreeRTOS基础

### 获取系统时间

```C
TickType_t currentTicks;
while(1)
{
     currentTicks = xTaskGetTickCount();
     printf("[%lu]dev:%u,name:%s\r\n", currentTicks, data->dev, data->data);
}
```



### 堆和栈

堆：一块空闲的内存 可以使用`malloc`申请内存 使用完后用`free`释放内存

> 堆也可以理解为就是定义了一个堆大小的数组 然后定义一个长度 每次动态改变长度来申请空间（链表）
>
> 在`FreeRTOSConfig.h`中就有定义堆大小的宏`configTOTAL_HEAP_SIZE` 他被引用到 `heap.c`中的 `static uint8_t ucHeap[configTOTAL_HEAP_SIZE];`

栈：一块内存空间，CPU的SP寄存器指向它，它可以用于函数调用、局部变量、`多任务系统里保存现场`

> 栈是`FreeRTOS`中非常重要的概念，每个任务都有自己的栈

###  FreeRTOS源码概述

#### 目录结构

![img](https://photos.100ask.net/renesas-docs/DShanMCU_RA6M5/FreeRTOS/chapter-6/image1.png)

> 主要涉及3个目录：
>
> - `ra\aws\FreeRTOS\FreeRTOS\Source`：存放的是FreeRTOS的核心文件
> - `ra_gen`：从main.c可以看到创建任务的函数调用过程
> - `src`：使用RASC创建任务时，在src目录下生成任务的入口函数

#### 核心文件

> FreeRTOS的最核心文件只有2个：
>
> - `FreeRTOS/Source/tasks.c`
> - `FreeRTOS/Source/list.c`

其他文件的作用也一起列表如下：

| **`FreeRTOS/Source/`下的文件** | **作用**                                      |
| ------------------------------ | --------------------------------------------- |
| `tasks.c`                      | 必需，任务操作                                |
| `list.c`                       | 必须，列表                                    |
| `queue.c`                      | 基本必需，提供队列操作、信号量(semaphore)操作 |
| `timer.c`                      | 可选，software timer                          |
| `event_groups.c`               | 可选，提供event group功能                     |

#### 头文件相关

##### 头文件目录

> FreeRTOS需要3个头文件目录：
>
> - FreeRTOS本身的头文件：`ra\aws\FreeRTOS\FreeRTOS\Source\include`
> - 移植时用到的头文件：`ra\fsp\src\rm_freertos_port\portmacro.h`
> - 含有配置文件`FreeRTOSConfig.h`的目录：`ra_cfg\aws`

##### 头文件

列表如下：

| **头文件**         | **作用**                                                     |
| ------------------ | ------------------------------------------------------------ |
| `FreeRTOSConfig.h` | FreeRTOS的配置文件，比如选择调度算法：`configUSE_PREEMPTION` 每个工程都必定含有`FreeRTOSConfig.h` |
| `FreeRTOS.h`       | 使用FreeRTOS API函数时，<span style="font-weight:bold;">必须</span>包含此文件。 在`FreeRTOS.h`之后，再去包含其他头文件，比如： `task.h`、`queue.h`、`semphr.h`、`event_group.h` |

#### 独属于FreeRTOS的数据类型和编程规范

##### 数据类型

每个移植的版本都含有自己的 **`portmacro.h`** 头文件，里面定义了2个数据类型：

1. `TickType_t`：
   - 时钟计数单位 （可以是uint16_t 也可以是uint32_t）
     当`FreeRTOSConfig.h`中定义`configUSE_16_BIT_TICKS时 TickType_t`则为uint16_t

> 对于32位架构，建议把TickType_t配置为uint32_t

2. `BaseType_t`:这是该架构最高效的数据类型(取决于CPU是多少位的架构)
   - `BaseType_t`通常用作简单的返回值的类型，还有逻辑值，比如 **pdTRUE/pdFALSE**

#####  变量名

变量名的前缀 对应其类型 如：`pcName` (代表着他是一个指向char类型的指针p)

| **变量名前缀** | **含义**                                                     |
| -------------- | ------------------------------------------------------------ |
| c              | char                                                         |
| s              | int16_t，short                                               |
| l              | int32_t，long                                                |
| x              | BaseType_t， 其他非标准的类型：结构体、task handle、queue handle等 |
| u              | unsigned                                                     |
| p              | 指针                                                         |
| uc             | uint8_t，unsigned char                                       |
| pc             | char指针                                                     |

##### 函数名

函数名的前缀有2部分：返回值类型、在哪个文件定义。

| **函数名前缀**                                           | **含义**                                     |
| -------------------------------------------------------- | -------------------------------------------- |
| v<span style="font-weight:bold;">Task</span>PrioritySet  | 返回值类型：void 在`task.c`中定义            |
| x<span style="font-weight:bold;">Queue</span>Receive     | 返回值类型：BaseType_t 在`queue.c`中定义     |
| pv<span style="font-weight:bold;">Timer</span>GetTimerID | 返回值类型：pointer to void 在`tmer.c`中定义 |

##### 宏的名

宏的名字是大小，可以添加小写的前缀。前缀是用来表示：宏在<span style="font-weight:bold;">哪个文件</span>中定义。

| **宏的前缀**                                                 | **含义：在哪个文件里定义**  |
| ------------------------------------------------------------ | --------------------------- |
| port (比如<span style="font-weight:bold;">port</span>MAX_DELAY) | `portable.h`或`portmacro.h` |
| task (比如<span style="font-weight:bold;">task</span>ENTER_CRITICAL()) | `task.h`                    |
| pd (比如<span style="font-weight:bold;">pd</span>TRUE)       | `projdefs.h`                |
| config (比如<span style="font-weight:bold;">config</span>USE_PREEMPTION) | `FreeRTOSConfig.h`          |
| err (比如<span style="font-weight:bold;">err</span>QUEUE_FULL) | `projdefs.h`                |

通用的宏定义如下：

| **宏**    | **值** |
| --------- | ------ |
| `pdTRUE`  | 1      |
| `pdFALSE` | 0      |
| `pdPASS`  | 1      |
| `pdFAIL`  | 0      |

### 内存管理

>  FreeRTOS中内存管理的接口函数为：`pvPortMalloc` 、`vPortFree`，对应于C库的`malloc`、`free`。 文件在`FreeRTOS/Source/portable/MemMang`下，它也是放在`portable目录`下，表示你可以提供自己的函数。
>
> 源码中默认提供了5个文件，对应内存管理的5种方法。

| **文件**         | **优点**                                           | **缺点**                                             |
| ---------------- | -------------------------------------------------- | ---------------------------------------------------- |
| `heap_1.c`       | 分配简单，时间确定                                 | 只分配、<span style="color:#FF0000;">不回收</span>   |
| `heap_2.c`       | 动态分配、最佳匹配                                 | <span style="color:#FF0000;">碎片</span>、时间不定   |
| `heap_3.c`       | 调用<span style="color:#FF3333;">标准库</span>函数 | <span style="color:#FF0000;">速度慢</span>、时间不定 |
| `heap_4.c`(常用) | 相邻空闲内存可合并                                 | 可解决碎片问题、时间不定                             |
| `heap_5.c`       | 在heap_4基础上支持分隔的内存块                     | 可解决碎片问题、时间不定                             |

#### Heap相关的函数

#####  分配、释放内存

函数原型：

```c
void * pvPortMalloc( size_t xWantedSize );
void vPortFree( void * pv );
```

作用：分配内存、释放内存。

如果分配内存不成功，则返回值为NULL。

##### 检测剩余多少空闲内存

函数原型：

```c
size_t xPortGetFreeHeapSize( void );
```

> 当前还有多少空闲内存，这函数可以用来优化内存的使用情况。比如当所有内核对象都分配好后，执行此函数返回2000，那么`configTOTAL_HEAP_SIZE`就可减小2000。

注意：在heap_3中无法使用。

#####  获取空闲内存容量的最小值

函数原型：

```c
size_t xPortGetMinimumEverFreeHeapSize( void );
```

返回：程序运行过程中，空闲内存容量的最小值。

注意：只有heap_4、heap_5支持此函数。

##### `malloc失败的钩子函数`

在pvPortMalloc函数内部：

```c
void * pvPortMalloc( size_t xWantedSize )vPortDefineHeapRegions
{
    ......
    #if ( configUSE_MALLOC_FAILED_HOOK == 1 )
        {
            if( pvReturn == NULL )
            {
                extern void vApplicationMallocFailedHook( void );
                vApplicationMallocFailedHook();
            }
        }
    #endif
    
    return pvReturn;        
}
```

所以，如果想使用这个钩子函数：

- 在`FreeRTOSConfig.h`中，把`configUSE_MALLOC_FAILED_HOOK`定义为1
- 提供`vApplicationMallocFailedHook`函数
- `pvPortMalloc`失败时，才会调用此函数

### 任务管理

#### 任务创建与删除

##### 什么是任务

在FreeRTOS中，任务就是一个函数，原型如下：

```c
void ATaskFunction( void *pvParameters );
```

要注意的是：

- 这个函数不能返回
- 同一个函数，可以用来创建多个任务；换句话说，多个任务可以运行同一个函数
- 函数内部，尽量使用局部变量：
  - 每个任务都有自己的栈
  - 每个任务运行这个函数时
    - 任务A的局部变量放在任务A的栈里、任务B的局部变量放在任务B的栈里
    - 不同任务的局部变量，有自己的副本
  - 函数使用全局变量、静态变量的话
    - 只有一个副本：多个任务使用的是同一个副本
    - 要防止冲突(后续会讲) 下面是一个示例：

```c
void ATaskFunction( void *pvParameters )
{
	/* 对于不同的任务，局部变量放在任务的栈里，有各自的副本 */
	int32_t lVariableExample = 0;
	
    /* 任务函数通常实现为一个无限循环 */
	for( ;; )
	{
		/* 任务的代码 */
	}

    /* 如果程序从循环中退出，一定要使用vTaskDelete删除自己
     * NULL表示删除的是自己
     */
	vTaskDelete( NULL );
    
    /* 程序不会执行到这里, 如果执行到这里就出错了 */
}
```

##### 创建任务

创建任务时使用的函数如下：

```c
BaseType_t xTaskCreate( TaskFunction_t pxTaskCode, // 函数指针, 任务函数
                        const char * const pcName, // 任务的名字
                        const configSTACK_DEPTH_TYPE usStackDepth, // 栈大小,单位为word,10表示40字节
                        void * const pvParameters, // 调用任务函数时传入的参数
                        UBaseType_t uxPriority,    // 优先级
                        TaskHandle_t * const pxCreatedTask ); // 任务句柄, 以后使用它来操作这个任务
```

参数说明：

| **参数**      | **描述**                                                     |
| ------------- | ------------------------------------------------------------ |
| pvTaskCode    | 函数指针，任务对应的 C 函数。任务应该永远不退出，或者在退出时调用 "`vTaskDelete(NULL)`"。 |
| pcName        | 任务的名称，仅用于调试目的，FreeRTOS 内部不使用。pcName 的长度为 `configMAX_TASK_NAME_LEN`。 |
| usStackDepth  | 每个任务都有自己的栈，usStackDepth 指定了栈的大小，单位为 word。例如，如果传入 100，表示栈的大小为 100 word，即 400 字节。最大值为 uint16_t 的最大值。确定栈的大小并不容易，通常是根据估计来设定。精确的办法是查看反汇编代码。 |
| pvParameters  | 调用 pvTaskCode 函数指针时使用的参数：`pvTaskCode(pvParameters)`。 |
| uxPriority    | 任务的优先级范围为 0~(`configMAX_PRIORITIES – 1`)。数值越小，优先级越低。如果传入的值过大，`xTaskCreate `会将其调整为 (`configMAX_PRIORITIES – 1`)。 |
| pxCreatedTask | 用于保存 xTaskCreate 的输出结果，即任务的句柄（task handle）。如果以后需要对该任务进行操作，如修改优先级，则需要使用此句柄。如果不需要使用该句柄，可以传入 NULL。 |
| 返回值        | 成功时返回 pdPASS，失败时返回 `errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY`（失败原因是内存不足）。请注意，文档中提到的失败返回值是 pdFAIL 是不正确的。pdFAIL 的值为 0，而 `errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY` 的值为 -1。 |

使用静态分配内存的函数如下：

```c
TaskHandle_t xTaskCreateStatic ( 
    TaskFunction_t pxTaskCode,   // 函数指针, 任务函数
    const char * const pcName,   // 任务的名字
    const uint32_t ulStackDepth, // 栈大小,单位为word,10表示40字节
    void * const pvParameters,   // 调用任务函数时传入的参数
    UBaseType_t uxPriority,      // 优先级
    StackType_t * const puxStackBuffer, // 静态分配的栈，就是一个buffer
    StaticTask_t * const pxTaskBuffer // 静态分配的任务结构体的指针，用它来操作这个任务
);
```

相比于使用动态分配内存创建任务的函数，最后2个参数不一样：

| **参数**       | **描述**                                                     |
| -------------- | ------------------------------------------------------------ |
| pvTaskCode     | 函数指针，可以简单地认为任务就是一个C函数。 它稍微特殊一点：永远不退出，或者退出时要调用"`vTaskDelete(NULL)`" |
| pcName         | 任务的名字，FreeRTOS内部不使用它，仅仅起调试作用。 长度为：`configMAX_TASK_NAME_LEN` |
| usStackDepth   | 每个任务都有自己的栈，这里指定栈大小。 单位是word，比如传入100，表示栈大小为100 word，也就是400字节。 最大值为uint16_t的最大值。 怎么确定栈的大小，并不容易，很多时候是估计。 精确的办法是看反汇编码。 |
| pvParameters   | 调用pvTaskCode函数指针时用到：pvTaskCode(pvParameters)       |
| uxPriority     | 优先级范围：0~(`configMAX_PRIORITIES – 1`) 数值越小优先级越低， 如果传入过大的值，`xTaskCreate`会把它调整为(`configMAX_PRIORITIES – 1`) |
| puxStackBuffer | 静态分配的栈内存，比如可以传入一个数组， 它的大小是`usStackDepth*4`。 |
| pxTaskBuffer   | 静态分配的`StaticTask_t`结构体的指针                         |
| 返回值         | 成功：返回任务句柄； 失败：NULL                              |

#### 任务栈大小估算

任务栈空间包含

- 返回地址`LR`与其他寄存器：可以理解为<span style="color:#FF0000;">函数调用深度</span>
- 局部变量：如`char buff[1000]` 直接看类型 此处则可以算1个
- 保留现场：可以理解为固定16个寄存器 即<span style="color:#FF3333;">16 * 4 = 64b</span>

> 公式如下：
>
> - （n层调用 * 36） + 局部变量 + 64b
>
> - 例如：
>
>   ![image-20250914221448446](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509142214793.png)

#### 使用任务参数

我们说过，多个任务可以使用同一个函数，怎么体现它们的差别？

- 栈不同
- 创建任务时可以传入不同的参数

我们创建2个任务，使用同一个函数，但是在LCD上打印不一样的信息。

```c
typedef struct{
    int x;
    int y;
    const char *str;
}DisplayInfo;
void vTaskFunction( void *pvParameters )
{
	DisplayInfo *info = pvParameters;
	uint32_t cnt = 0;
uint32_t len;
	
	/* 任务函数的主体一般都是无限循环 */
	for( ;; )
	{
		/* 打印任务的信息 */
		len = LCD_PrintString(info->x, info->y, info->str);
		LCD_PrintSignedVal(len+1, info->y, cnt++);

		mdelay(500);
	}
}
```

上述代码中的info来自参数pvParameters，pvParameters来自哪里？创建任务时传入的。

代码如下：

- 使用xTaskCreate创建任务时，第4个参数就是pvParameters
- 不同的任务，pvParameters不一样

```c
DisplayInfo g_Task1Info ={0,0,"task1"};
DisplayInfo g_Task2Info ={3,0,"task2"};
DisplayInfo g_Task3Info ={6,0,"task3"};

/* 使用同一个函数创建不同的任务 */
  xTaskCreate(LcdPrintTask, "task1", 128, &g_Task1Info, osPriorityNormal, NULL);
  xTaskCreate(LcdPrintTask, "task2", 128, &g_Task2Info, osPriorityNormal, NULL);
  xTaskCreate(LcdPrintTask, "task3", 128, &g_Task3Info, osPriorityNormal, NULL);
```

#### 任务的删除

删除任务时使用的函数如下：

```c
void vTaskDelete( TaskHandle_t xTaskToDelete );
```

参数说明：

| **参数**   | **描述**                                                     |
| ---------- | ------------------------------------------------------------ |
| pvTaskCode | 任务句柄，使用xTaskCreate创建任务时可以得到一个句柄。 也可传入NULL，这表示删除自己。 |

怎么删除任务？举个不好的例子：

- 自杀：`vTaskDelete(NULL)`
- 被杀：别的任务执行`vTaskDelete(pvTaskCode)`，`pvTaskCode`是自己的句柄
- 杀人：执行`vTaskDelete(pvTaskCode)`，`pvTaskCode`是别的任务的句柄

#### 任务管理与调度机制

##### 调度机制

> - 同等优先级的任务 轮流运行
> - 最高优先级的任务先运行
>   - 高优先级的任务未运行完，低优先级任务无法运行
>   - 一旦高优先级任务就绪 马上运行
>   - 最高优先级的任务有多个 他们轮流运行

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509151548547.png)

##### 任务管理

任务的切换通过链表操作进行切换 RTOS会创建3个链表 分别是就绪链表ReadyList、阻塞链表DelayTaskList、挂起链表

- 先在优先级数组中按下标从大到小遍历，使用任务创建函数<span style="color:#33FFFF;">本质其实是在对应优先级数组中创建TCB结构体，并且会有一个全局指针——当前执行TCB结构体pxCurrentTCB</span>，这就说明了为什么同等优先级下，最后创建的任务先运行（因为指针最终指向了最后一个任务链表，启动调度器后，开始运行）

- 当调用`vTaskDelay`后会将当前任务链表丢到阻塞链表上，并根据`vTaskDelay`的参数等待`Tick中断`判断到达时间后，该任务链表才能再次回到原先链表上

- 当任务调用`vTaskSuspend`后 当前任务会被丢到挂起链表，此任务只有调用`vTaskResume`才能回到原先链表，不能通过`Tick中断`回归

> Tick中断是什么？他又做些什么？
>
> - Tick中断来源于
>
>   ![image-20250915160032344](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509151600535.png)
>
> - Tick中断做什么？
>
>   1. 有个计数器count，count一直执行++
>   2. 判断DelayTaskList里任务是否可以恢复（可恢复则将其丢到ReadyList）
>   3. 发起调度（遍历优先级数组，改变pxCurrentTCB指针指向）

##### 空闲任务

空闲任务是Freertos启动调度器时 会自动创建的一个最低优先级的任务`prvIdleTask`

- 负责清理用户的自杀任务 如`vTaskDelete(NULL)`的内存释放
- 由于优先级最低 所以当用户任务不释放CPU时，就会导致空闲任务不会运行，因而`vTaskDelete(NULL)`得不到释放
- 因此尽量不要使用死延时，改用`vTaskDelay`释放CPU 或者不使用自杀函数`vTaskDelete(NULL)`

> 此外 空闲任务中还有提供钩子函数 可以方便我们在里面打印调试信息
>
> 注意每个任务都要使用while(1)进行死循环
>
> - 如果有一个任务没有则会触发任务退出错误中断`prvTaskExitError`
> - ![image-20250915163446773](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509151634028.png)

### 同步与互斥

#### 同步与互斥的概念

一句话理解同步与互斥：我等你用完厕所，我再用厕所。

- 什么叫同步？就是：哎哎哎，我正在用厕所，你等会。 
- 什么叫互斥？就是：哎哎哎，我正在用厕所，你不能进来。

同步与互斥经常放在一起讲，是因为它们之的关系很大，“互斥”操作可以使用“同步”来实现。我“等”你用完厕所，我再用厕所。这不就是用“同步”来实现“互斥”吗？

再举一个例子。在团队活动里，同事A先写完报表，经理B才能拿去向领导汇报。经理B必须等同事A完成报表，AB之间有依赖，B必须放慢脚步，被称为同步。在团队活动中，同事A已经使用会议室了，经理B也想使用，即使经理B是领导，他也得等着，这就叫互斥。经理B跟同事A说：你用完会议室就提醒我。这就是使用"同步"来实现"互斥"。

有时候看代码更容易理解，伪代码如下：

```c
 void  抢厕所(void)
 {
   if (有人在用) 我眯一会;
   用厕所;
   喂，醒醒，有人要用厕所吗;
 }
```

假设有A、B两人早起抢厕所，A先行一步占用了；B慢了一步，于是就眯一会；当A用完后叫醒B，B也就愉快地上厕所了。

在这个过程中，A、B是互斥地访问“厕所”，“厕所”被称之为临界资源。我们使用了“休眠-唤醒”的同步机制实现了“临界资源”的“互斥访问”。

同一时间只能有一个人使用的资源，被称为临界资源。比如任务A、B都要使用串口来打印，串口就是临界资源。如果A、B同时使用串口，那么打印出来的信息就是A、B混杂，无法分辨。所以使用串口时，应该是这样：A用完，B再用；B用完，A再用

#### 各类方法的对比

能实现同步、互斥的内核方法有：任务通知(task notification)、队列(queue)、事件组(event group)、信号量(semaphoe)、互斥量(mutex)。

它们都有类似的操作方法：获取/释放、阻塞/唤醒、超时。比如：

- 任务A获取资源，用完后任务A释放资源
- 任务A获取不到资源则阻塞，任务B释放资源并把任务A唤醒
- 任务A获取不到资源则阻塞，并定个闹钟；A要么超时返回，要么在这段时间内因为任务B释放资源而被唤醒。

这些内核对象五花八门，记不住怎么办？我也记不住，通过对比的方法来区分它们。

- 能否传信息？还是只能传递状态？
- 为众生（所有任务都可以使用）？只为你（只能指定任务使用）？
- 我生产，你们消费？
- 我上锁，只能由我开锁

| **内核对象** | **生产者** | **消费者** | **数据/状态**                                                | **说明**                                                     |
| ------------ | ---------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 队列         | ALL        | ALL        | 数据：若干个数据 谁都可以往队列里扔数据， 谁都可以从队列里读数据 | 用来传递数据， 发送者、接收者无限制， 一个数据只能唤醒一个接收者 |
| 事件组       | ALL        | ALL        | 多个位：或、与 谁都可以设置(生产)多个位， 谁都可以等待某个位、若干个位 | 用来传递事件， 可以是N个事件， 发送者、接受者无限制， 可以唤醒多个接收者：像广播 |
| 信号量       | ALL        | ALL        | 数量：0~n 谁都可以增加一个数量， 谁都可消耗一个数量          | 用来维持资源的个数， 生产者、消费者无限制， 1个资源只能唤醒1个接收者 |
| 任务通知     | ALL        | 只有我     | 数据、状态都可以传输， 使用任务通知时， 必须指定接受者       | N对1的关系： 发送者无限制， 接收者只能是这个任务             |
| 互斥量       | 只能A开锁  | A上锁      | 位：0、1 我上锁：1变为0， 只能由我开锁：0变为1               | 就像一个空厕所， 谁使用谁上锁， 也只能由他开锁               |

使用图形对比如下：

- 队列：
  - 里面可以放任意数据，可以放多个数据
  - 任务、ISR都可以放入数据；任务、ISR都可以从中读出数据
- 事件组：
  - 一个事件用一bit表示，1表示事件发生了，0表示事件没发生
  - 可以用来表示事件、事件的组合发生了，不能传递数据
  - 有广播效果：事件或事件的组合发生了，等待它的多个任务都会被唤醒
- 信号量：
  - 核心是"计数值"
  - 任务、ISR释放信号量时让计数值加1
  - 任务、ISR获得信号量时，让计数值减1
- 任务通知：
  - 核心是任务的TCB里的数值
  - 会被覆盖
  - 发通知给谁？必须指定接收任务
  - 只能由接收任务本身获取该通知
- 互斥量：
  - 数值只有0或1
  - 谁获得互斥量，就必须由谁释放同一个互斥量

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509151740912.png)

> 为何要引用这些机制呢？
>
> - 原本逻辑可以采用全局变量来进行传递信息，但在RTOS下，可能还未来得及改变全局变量的值就被切换到其他任务，这样就会发生全局变量未及时更新的问题
> - 引入事件阻塞与唤醒大大提高了CPU的利用率
>   - 当任务A计数1s，任务B负责打印任务A的最终计数值，就不会出现A运行，B也运行，而是等待A运行完后通知任务B，或大概估算任务A结束的时间，让B阻塞这么多的时间，这样就实现了整个时间轴都是任务A在运行

### 队列

#### 队列的本质

队列中，数据的读写就是环形缓冲区，在这个基础上增加了互斥措施、阻塞-唤醒机制

- 如果这个队列不传输数据，只调整“数据个数”，他就是信号量（semaphore）
- 如果信号量中，限定“数据个数”最大值为1，他就是互斥量（mutex）

#### 传输数据的两种方法

使用队列传输数据时有两种方法：

- 拷贝：把数据、把变量的值复制进队列里
- 引用：把数据、把变量的地址复制进队列里

FreeRTOS使用拷贝值的方法，这更简单：

- 局部变量的值可以发送到队列中，后续即使函数退出、局部变量被回收，也不会影响队列中的数据
- 无需分配buffer来保存数据，队列中有buffer
- 局部变量可以马上再次使用
- 发送任务、接收任务解耦：接收任务不需要知道这数据是谁的、也不需要发送任务来释放数据
- 如果数据实在太大，你还是可以使用队列传输它的地址
- 队列的空间有FreeRTOS内核分配，无需任务操心
- 对于有内存保护功能的系统，如果队列使用引用方法，也就是使用地址，必须确保双方任务对这个地址都有访问权限。使用拷贝方法时，则无此限制：内核有足够的权限，把数据复制进队列、再把数据复制出队列。

#### 队列的阻塞访问

只要知道队列的句柄，谁都可以读、写该队列。任务、ISR都可读、写队列。可以多个任务读写队列。

任务读写队列时，简单地说：如果读写不成功，则阻塞；可以指定超时时间。口语化地说，就是可以定个闹钟：如果能读写了就马上进入就绪态，否则就阻塞直到超时。

某个任务读队列时，如果队列没有数据，则该任务可以进入阻塞状态：还可以指定阻塞的时间。如果队列有数据了，则该阻塞的任务会变为就绪态。如果一直都没有数据，则时间到之后它也会进入就绪态。

既然读取队列的任务个数没有限制，那么当多个任务读取空队列时，这些任务都会进入阻塞状态：有多个任务在等待同一个队列的数据。当队列中有数据时，哪个任务会进入就绪态？

- 优先级最高的任务
- 如果大家的优先级相同，那等待时间最久的任务会进入就绪态

跟读队列类似，一个任务要写队列时，如果队列满了，该任务也可以进入阻塞状态：还可以指定阻塞的时间。如果队列有空间了，则该阻塞的任务会变为就绪态。如果一直都没有空间，则时间到之后它也会进入就绪态。

既然写队列的任务个数没有限制，那么当多个任务写"满队列"时，这些任务都会进入阻塞状态：有多个任务在等待同一个队列的空间。当队列中有空间时，哪个任务会进入就绪态？

- 优先级最高的任务
- 如果大家的优先级相同，那等待时间最久的任务会进入就绪态

#### 队列函数

##### 创建

队列的创建有两种方法：动态分配内存、静态分配内存，

- 动态分配内存：xQueueCreate，队列的内存在函数内部动态分配

函数原型如下：

```c
QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength, UBaseType_t uxItemSize );
```

| **参数**        | **说明**                                                     |
| --------------- | ------------------------------------------------------------ |
| `uxQueueLength` | 队列长度，最多能存放多少个数据(item)                         |
| `uxItemSize`    | 每个数据(item)的大小：以字节为单位（可以定义一个结构体存放，然后sizeof(结构体)） |
| 返回值          | 非0：成功，返回句柄，以后使用句柄来操作队列 NULL：失败，因为内存不足 |

- 静态分配内存：`xQueueCreateStatic`，队列的内存要事先分配好

函数原型如下：

```c
QueueHandle_t xQueueCreateStatic(*
              		UBaseType_t uxQueueLength,*
              		UBaseType_t uxItemSize,*
              		uint8_t *pucQueueStorageBuffer,*
              		StaticQueue_t *pxQueueBuffer*
           		 );
```

| **参数**                | **说明**                                                     |
| ----------------------- | ------------------------------------------------------------ |
| `uxQueueLength`         | 队列长度，最多能存放多少个数据(item)                         |
| `uxItemSize`            | 每个数据(item)的大小：以字节为单位                           |
| `pucQueueStorageBuffer` | 如果uxItemSize非0，`pucQueueStorageBuffer`必须指向一个uint8_t数组， 此数组大小至少为"uxQueueLength * uxItemSize" |
| `pxQueueBuffer`         | 必须执行一个`StaticQueue_t`结构体，用来保存队列的数据结构    |
| 返回值                  | 非0：成功，返回句柄，以后使用句柄来操作队列 NULL：失败，因为`pxQueueBuffer`为NULL |

##### 复位

队列刚被创建时，里面没有数据；使用过程中可以调用 **xQueueReset()** 把队列恢复为初始状态，此函数原型为：

```c
/*  pxQueue : 复位哪个队列;
 * 返回值: pdPASS(必定成功)
*/
BaseType_t xQueueReset( QueueHandle_t pxQueue);
```

##### 删除

删除队列的函数为 **vQueueDelete()** ，只能删除使用动态方法创建的队列，它会释放内存。原型如下：

```c
void vQueueDelete( QueueHandle_t xQueue );
```

##### 写队列

可以把数据写到队列头部，也可以写到尾部，这些函数有两个版本：在任务中使用、在ISR中使用。函数原型如下：

```c
/* 等同于xQueueSendToBack
 * 往队列尾部写入数据，如果没有空间，阻塞时间为xTicksToWait
 */
BaseType_t xQueueSend(
                                QueueHandle_t    xQueue,
                                const void       *pvItemToQueue,
                                TickType_t       xTicksToWait
                            );

/* 
 * 往队列尾部写入数据，如果没有空间，阻塞时间为xTicksToWait
 */
BaseType_t xQueueSendToBack(
                                QueueHandle_t    xQueue,
                                const void       *pvItemToQueue,
                                TickType_t       xTicksToWait
                            );


/* 
 * 往队列尾部写入数据，此函数可以在中断函数中使用，不可阻塞
 */
BaseType_t xQueueSendToBackFromISR(
                                      QueueHandle_t xQueue,
                                      const void *pvItemToQueue,
                                      BaseType_t *pxHigherPriorityTaskWoken
                                   );

/* 
 * 往队列头部写入数据，如果没有空间，阻塞时间为xTicksToWait
 */
BaseType_t xQueueSendToFront(
                                QueueHandle_t    xQueue,
                                const void       *pvItemToQueue,
                                TickType_t       xTicksToWait
                            );

/* 
 * 往队列头部写入数据，此函数可以在中断函数中使用，不可阻塞
 */
BaseType_t xQueueSendFromISR(
                                      QueueHandle_t xQueue,
                                      const void *pvItemToQueue,
                                      BaseType_t *pxHigherPriorityTaskWoken
                                   );
```

这些函数用到的参数是类似的，统一说明如下：

| 参数            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `xQueue`        | 队列句柄，要写哪个队列                                       |
| `pvItemToQueue` | 数据指针，这个数据的值会被复制进队列， 复制多大的数据？在创建队列时已经指定了数据大小 |
| `xTicksToWait`  | 如果队列满则无法写入新数据，可以让任务进入阻塞状态， `xTicksToWait`表示阻塞的最大时间(Tick Count)。 如果被设为0，无法写入数据时函数会立刻返回； 如果被设为`portMAX_DELAY`，则会一直阻塞直到有空间可写 |
| 返回值          | pdPASS：数据成功写入了队列 `errQUEUE_FULL`：写入失败，因为队列满了。 |

##### 读队列

使用 **xQueueReceive()** 函数读队列，读到一个数据后，队列中该数据会被移除。这个函数有两个版本：在任务中使用、在ISR中使用。函数原型如下：

```c
BaseType_t xQueueReceive( QueueHandle_t xQueue,
                          void * const pvBuffer,
                          TickType_t xTicksToWait );

BaseType_t xQueueReceiveFromISR(
                                    QueueHandle_t    xQueue,
                                    void             *pvBuffer,
                                    BaseType_t       *pxTaskWoken
                                );
```

参数说明如下：

| **参数**       | **说明**                                                     |
| -------------- | ------------------------------------------------------------ |
| `xQueue`       | 队列句柄，要读哪个队列                                       |
| `pvBuffer`     | buffer指针，队列的数据会被复制到这个buffer 复制多大的数据？在创建队列时已经指定了数据大小 |
| `xTicksToWait` | 果队列空则无法读出数据，可以让任务进入阻塞状态， xTicksToWait表示阻塞的最大时间(Tick Count)。 如果被设为0，无法读出数据时函数会立刻返回； 如果被设为portMAX_DELAY，则会一直阻塞直到有数据可写 |
| 返回值         | pdPASS：从队列读出数据入 `errQUEUE_EMPTY`：读取失败，因为队列空了。 |

##### 查询

可以查询队列中有多少个数据、有多少空余空间。函数原型如下：

```c
/*
 * 返回队列中可用数据的个数
 */
UBaseType_t uxQueueMessagesWaiting( const QueueHandle_t xQueue );

/*
 * 返回队列中可用空间的个数
 */
UBaseType_t uxQueueSpacesAvailable( const QueueHandle_t xQueue );
```

##### 覆盖/偷看

当队列长度为1时，可以使用 **xQueueOverwrite()** 或 **xQueueOverwriteFromISR()** 来覆盖数据。

注意，队列长度必须为1。当队列满时，这些函数会覆盖里面的数据，这也以为着这些函数不会被阻塞。

函数原型如下：

```c
/* 覆盖队列
 * xQueue: 写哪个队列
 * pvItemToQueue: 数据地址
 * 返回值: pdTRUE表示成功, pdFALSE表示失败
 */
BaseType_t xQueueOverwrite(
                           QueueHandle_t xQueue,
                           const void * pvItemToQueue
                      );

BaseType_t xQueueOverwriteFromISR(
                           QueueHandle_t xQueue,
                           const void * pvItemToQueue,
                           BaseType_t *pxHigherPriorityTaskWoken
                      );
```

如果想让队列中的数据供多方读取，也就是说读取时不要移除数据，要留给后来人。那么可以使用"窥视"，也就是**xQueuePeek()\**或\**xQueuePeekFromISR()**。这些函数会从队列中复制出数据，但是不移除数据。这也意味着，如果队列中没有数据，那么"偷看"时会导致阻塞；一旦队列中有数据，以后每次"偷看"都会成功。

函数原型如下：

```c
/* 偷看队列
 * xQueue: 偷看哪个队列
 * pvItemToQueue: 数据地址, 用来保存复制出来的数据
 * xTicksToWait: 没有数据的话阻塞一会
 * 返回值: pdTRUE表示成功, pdFALSE表示失败
 */
BaseType_t xQueuePeek(
                          QueueHandle_t xQueue,
                          void * const pvBuffer,
                          TickType_t xTicksToWait
                      );

BaseType_t xQueuePeekFromISR(
                                 QueueHandle_t xQueue,
                                 void *pvBuffer,
                             );
```

#### 使用队列时注意

> 写队列与读队列 所传入的`buffer`一定要严格按照初始创建时的类型，否则就会出现死机
>
> - 情况一：创建时类型为指针（常用，可以接收多种类型）
>
> ```C
> typedef struct
> {
>     uint8_t data[128];
>     uint8_t index;
> }Uart_RxTypeDef;
> 
> Uart_RxTypeDef UartRx = {{0},0};
> 
> void task(void *arg)
> {
>     Uart_RxTypeDef *data = &UartRx;
>     debug_TX_QueueHandle = osMessageQueueNew(5, sizeof(void *), &debug_TX_Queue_attributes);
>     xQueueSend(debug_TX_QueueHandle,&data,NULL);//此处一定要&data 因为创建队列的时候是指针 直接写data 相当于&UartRx 他并不是一个指针
> }
> 
> ```
>
> - 情况二：创建时类型为数据
>
> ```C
> typedef struct
> {
>     uint8_t data[128];
>     uint8_t index;
> }Uart_RxTypeDef;
> 
> Uart_RxTypeDef UartRx = {{0},0};
> 
> void task(void *arg)
> {
>     Uart_RxTypeDef data = {{0},0};
>     debug_TX_QueueHandle = osMessageQueueNew(5, sizeof(Uart_RxTypeDef), &debug_TX_Queue_attributes);
>     xQueueSend(debug_TX_QueueHandle,&data,NULL);//此处的&data与情况一不一样 这个是参数要求要地址 所以需要将结构体的地址赋值过去
> }
> ```
>
> 

#### 队列集

> 当多个队列需要上报到同一个处理任务时，队列集可以实现统合多个队列，再往上传递到同一个任务

![image-20250915213146479](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509152131801.png)

##### 创建队列集

> 注意要使用队列集 需要先在`Core/inc/FreeRTOSConfig.h` 定义
>
> ```bash
> #define configUSE_QUEUE_SETS 1
> ```

函数原型如下：

```c
QueueSetHandle_t xQueueCreateSet( const UBaseType_t uxEventQueueLength )
```

| **参数**      | **说明**                                                     |
| ------------- | ------------------------------------------------------------ |
| uxQueueLength | 队列集长度，最多能存放多少个数据(队列句柄)                   |
| 返回值        | 非0：成功，返回句柄，以后使用句柄来操作队列NULL：失败，因为内存不足 |

##### 把队列加入队列集

函数原型如下：

```c
BaseType_t xQueueAddToSet( QueueSetMemberHandle_t xQueueOrSemaphore,

                QueueSetHandle_t xQueueSet );

 
```

| **参数**          | **说明**                       |
| ----------------- | ------------------------------ |
| xQueueOrSemaphore | 队列句柄，这个队列要加入队列集 |
| xQueueSet         | 队列集句柄                     |
| 返回值            | pdTRUE：成功pdFALSE：失败      |

##### 读取队列集

函数原型如下：

```c
QueueSetMemberHandle_t xQueueSelectFromSet( QueueSetHandle_t xQueueSet,

                        TickType_t const xTicksToWait );
```

| **参数**     | **说明**                                                     |
| ------------ | ------------------------------------------------------------ |
| xQueueSet    | 队列集句柄                                                   |
| xTicksToWait | 如果队列集空则无法读出数据，可以让任务进入阻塞状态，xTicksToWait表示阻塞的最大时间(Tick Count)。如果被设为0，无法读出数据时函数会立刻返回；如果被设为portMAX_DELAY，则会一直阻塞直到有数据可写 |
| 返回值       | NULL：失败，队列句柄：成功                                   |

### 信号量

前面介绍的队列(queue)可以用于传输数据：在任务之间、任务和中断之间。

消息队列用于传输多个数据，但是有时候我们只需要传递状态，这个状态值需要用一个数值表示，比如：

- 卖家：做好了1个包子！做好了2个包子！做好了3个包子！
- 买家：买了1个包子，包子数量减1
- 这个停车位我占了，停车位减1
- 我开车走了，停车位加1

在这种情况下我们只需要维护一个数值，使用信号量效率更高、更节省内存 本章涉及如下内容：

- 怎么创建、删除信号量
- 怎么发送、获得信号量
- 什么是计数型信号量？什么是二进制信号量？

####  信号量的特性

##### 信号量的常规操作

信号量这个名字很恰当：

- 信号：起通知作用
- 量：还可以用来表示资源的数量
  - 当"量"没有限制时，它就是"计数型信号量"(Counting Semaphores)
  - 当"量"只有0、1两个取值时，它就是"二进制信号量"(Binary Semaphores)
- 支持的动作："give"给出资源，计数值加1；"take"获得资源，计数值减1

计数型信号量的典型场景是：

- 计数：事件产生时"give"信号量，让计数值加1；处理事件时要先"take"信号量，就是获得信号量，让计数值减1。
- 资源管理：要想访问资源需要先"take"信号量，让计数值减1；用完资源后"give"信号量，让计数值加1。 信号量的"give"、"take"双方并不需要相同，可以用于生产者-消费者场合：
- 生产者为任务A、B，消费者为任务C、D
- 一开始信号量的计数值为0，如果任务C、D想获得信号量，会有两种结果：
  - 阻塞：买不到东西咱就等等吧，可以定个闹钟(超时时间)
  - 即刻返回失败：不等
- 任务A、B可以生产资源，就是让信号量的计数值增加1，并且把等待这个资源的顾客唤醒
- 唤醒谁？谁优先级高就唤醒谁，如果大家优先级一样就唤醒等待时间最长的人

二进制信号量跟计数型的唯一差别，就是计数值的最大值被限定为1。

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509161022312.png)

##### 信号量跟队列的对比

差异列表如下：

| 队列                                                         | 信号量                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 可以容纳多个数据， 创建队列时有2部分内存: 队列结构体、存储数据的空间 | 只有计数值，无法容纳其他数据。 创建信号量时，只需要分配信号量结构体 |
| 生产者：没有空间存入数据时可以阻塞                           | 生产者：用于不阻塞，计数值已经达到最大时返回失败             |
| 消费者：没有数据时可以阻塞                                   | 消费者：没有资源时可以阻塞                                   |

##### 两种信号量的对比

信号量的计数值都有限制：限定了最大值。如果最大值被限定为1，那么它就是二进制信号量；如果最大值不是1，它就是计数型信号量。

差别列表如下：

| 二进制信号量      | 计数型信号量           |
| ----------------- | ---------------------- |
| 被创建时初始值为0 | 被创建时初始值可以设定 |
| 其他操作是一样的  | 其他操作是一样的       |

#### 信号量函数

使用信号量时，先创建、然后去添加资源、获得资源。使用句柄来表示一个信号量。

##### 创建

使用信号量之前，要先创建，得到一个句柄；使用信号量时，要使用句柄来表明使用哪个信号量。 对于二进制信号量、计数型信号量，它们的创建函数不一样：

|          | 二进制信号量                                     | 计数型信号量                     |
| -------- | ------------------------------------------------ | -------------------------------- |
| 动态创建 | `xSemaphoreCreateBinary `计数值初始值为0         | `xSemaphoreCreateCounting`       |
|          | `vSemaphoreCreateBinary`(过时了) 计数值初始值为1 |                                  |
| 静态创建 | `xSemaphoreCreateBinaryStatic`                   | `xSemaphoreCreateCountingStatic` |

创建二进制信号量的函数原型如下：

```c
/* 创建一个二进制信号量，返回它的句柄。
 * 此函数内部会分配信号量结构体 
 * 返回值: 返回句柄，非NULL表示成功
 */
SemaphoreHandle_t xSemaphoreCreateBinary( void );

/* 创建一个二进制信号量，返回它的句柄。
 * 此函数无需动态分配内存，所以需要先有一个StaticSemaphore_t结构体，并传入它的指针
 * 返回值: 返回句柄，非NULL表示成功
 */
SemaphoreHandle_t xSemaphoreCreateBinaryStatic( StaticSemaphore_t *pxSemaphoreBuffer );
```

创建计数型信号量的函数原型如下：

```c
/* 创建一个计数型信号量，返回它的句柄。
 * 此函数内部会分配信号量结构体 
 * uxMaxCount: 最大计数值
 * uxInitialCount: 初始计数值
 * 返回值: 返回句柄，非NULL表示成功
 */
SemaphoreHandle_t xSemaphoreCreateCounting(UBaseType_t uxMaxCount, UBaseType_t uxInitialCount);

/* 创建一个计数型信号量，返回它的句柄。
 * 此函数无需动态分配内存，所以需要先有一个StaticSemaphore_t结构体，并传入它的指针
 * uxMaxCount: 最大计数值
 * uxInitialCount: 初始计数值
 * pxSemaphoreBuffer: StaticSemaphore_t结构体指针
 * 返回值: 返回句柄，非NULL表示成功
 */
SemaphoreHandle_t xSemaphoreCreateCountingStatic( UBaseType_t uxMaxCount, 
                                                 UBaseType_t uxInitialCount, 
                                                 StaticSemaphore_t *pxSemaphoreBuffer );
```

##### 删除

对于动态创建的信号量，不再需要它们时，可以删除它们以回收内存。

vSemaphoreDelete可以用来删除二进制信号量、计数型信号量，函数原型如下：

```c
/*
 * xSemaphore: 信号量句柄，你要删除哪个信号量
 */
void vSemaphoreDelete( SemaphoreHandle_t xSemaphore );
```

##### 释放/获取信号量

二进制信号量、计数型信号量的give、take操作函数是一样的。这些函数也分为2个版本：给任务使用，给ISR使用。列表如下：

|      | 在任务中使用     | 在ISR中使用             |
| ---- | ---------------- | ----------------------- |
| give | `xSemaphoreGive` | `xSemaphoreGiveFromISR` |
| take | `xSemaphoreTake` | `xSemaphoreTakeFromISR` |

xSemaphoreGive的函数原型如下：

```c
BaseType_t xSemaphoreGive( SemaphoreHandle_t xSemaphore );
```

xSemaphoreGive函数的参数与返回值列表如下：

| 参数       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| xSemaphore | 信号量句柄，释放哪个信号量                                   |
| 返回值     | pdTRUE表示成功, 如果二进制信号量的计数值已经是1，再次调用此函数则返回失败； 如果计数型信号量的计数值已经是最大值，再次调用此函数则返回失败 |

pxHigherPriorityTaskWoken的函数原型如下：

```c
BaseType_t xSemaphoreGiveFromISR(
                        SemaphoreHandle_t xSemaphore,
                        BaseType_t *pxHigherPriorityTaskWoken
                    );
```

xSemaphoreGiveFromISR函数的参数与返回值列表如下：

| 参数                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| xSemaphore                | 信号量句柄，释放哪个信号量                                   |
| pxHigherPriorityTaskWoken | 如果释放信号量导致更高优先级的任务变为了就绪态， 则*pxHigherPriorityTaskWoken = pdTRUE |
| 返回值                    | pdTRUE表示成功, 如果二进制信号量的计数值已经是1，再次调用此函数则返回失败； 如果计数型信号量的计数值已经是最大值，再次调用此函数则返回失败 |

xSemaphoreTake的函数原型如下：

```c
BaseType_t xSemaphoreTake(
                   SemaphoreHandle_t xSemaphore,
                   TickType_t xTicksToWait
               );
```

xSemaphoreTake函数的参数与返回值列表如下：

| 参数           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| `xSemaphore`   | 信号量句柄，获取哪个信号量                                   |
| `xTicksToWait` | 如果无法马上获得信号量，阻塞一会： 0：不阻塞，马上返回 `portMAX_DELAY`: 一直阻塞直到成功 其他值: 阻塞的Tick个数，可以使用*`pdMS_TO_TICKS`()*来指定阻塞时间为若干`ms` |
| 返回值         | pdTRUE表示成功                                               |

xSemaphoreTakeFromISR的函数原型如下：

```c
BaseType_t xSemaphoreTakeFromISR(
                        SemaphoreHandle_t xSemaphore,
                        BaseType_t *pxHigherPriorityTaskWoken
                    );
```

xSemaphoreTakeFromISR函数的参数与返回值列表如下：

| 参数                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `xSemaphore`                | 信号量句柄，获取哪个信号量                                   |
| `pxHigherPriorityTaskWoken` | 如果获取信号量导致更高优先级的任务变为了就绪态， 则`*pxHigherPriorityTaskWoken = pdTRUE` |
| 返回值                      | pdTRUE表示成功                                               |

#### 优先级反转

信号量使用不当 就会出现优先级反转的问题——任务执行顺序：任务1、任务2、任务1、任务3

即最高优先级的任务最后运行

> 优先级反转：假设信号量值为1，同时创建了低、中等、高优先级任务
>
> - 低优先级任务获取信号量，中等优先级先阻塞一段时间，确保低优先级任务先成功获取到信号量
> - 高优先级任务先阻塞一段时间，确保低优先级、中等优先级先完成任务，然后尝试获取信号量
> - 此时由于低优先级获取了信号量，高优先级任务就会被阻塞
> - 中等优先级不释放信号量，那么就会发生高优先级任务无法运行

##### 示例

```C
#include "uart.h"
typedef struct __FILE FILE;

static SemaphoreHandle_t task_mutex = NULL;

typedef struct
{
    uint8_t dev;
    uint8_t *data;
} uart_info_t;

uart_info_t task1_data = {1, "task1"};
uart_info_t task2_data = {2, "task2"};
uart_info_t task3_data = {3, "task3"};
int fputc(int ch, FILE *f)
{
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xFFFF);
    return ch;
}

void task1(void *param)
{
    uint8_t cnt = 0;
    xSemaphoreTake(task_mutex, portMAX_DELAY);
    uart_info_t *data = param;
    while (1)
    {
        printf("dev:%u,name:%s\r\n", data->dev, data->data);
        if (++cnt == 5)
        {
            xSemaphoreGive(task_mutex);
        }
        vTaskDelay(50);
    }
}

void task2(void *param)
{
    uint8_t cnt = 0;
    vTaskDelay(300);//先阻塞一段时间 确保任务1成功获取信号量且能够被切换为任务2
    uart_info_t *data = param;
    while (1)
    {
        printf("dev:%u,name:%s\r\n", data->dev, data->data);
        if (++cnt == 5)
        {
            vTaskDelete(NULL);
        }
        HAL_Delay(50);
    }
}

void task3(void *param)
{
    uint8_t cnt = 0;
    vTaskDelay(500);	//先阻塞一段时间 确保任务1先成功获取信号量且能够被切换为任务2
    xSemaphoreTake(task_mutex, portMAX_DELAY);
    uart_info_t *data = param;
    while (1)
    {
        printf("dev:%u,name:%s\r\n", data->dev, data->data);
        if (++cnt == 5)
        {
            vTaskDelete(NULL);
        }
    }
}

void uart_init(void)
{
    task_mutex = xSemaphoreCreateBinary();
    xSemaphoreGive(task_mutex);
    xTaskCreate(task1, "task1", 128, &task1_data, osPriorityNormal, NULL);	//低优先级任务
    xTaskCreate(task2, "task2", 128, &task2_data, osPriorityNormal1, NULL);	//中等优先级任务
    xTaskCreate(task3, "task3", 128, &task3_data, osPriorityNormal2, NULL);	//高优先级任务
}
```

![image-20250916160014655](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509161600972.png)

#### 解决优先级反转

> 使用互斥量 （实现优先级继承），当最高优先级任务等待互斥量时，可以将低优先级任务继承最高优先级，使得最高优先级任务也可以先运行
>
> 现象：任务1执行一小段时间被任务2抢占，当任务3时间到来，任务1继承任务3优先级，所以任务1运行释放后，任务1恢复原先优先级，任务3启动

### 事件组

> 上述任务同步与互斥的方法都是一对一通知，难以实现一对多通知（广播），因此引入了事件组

#### 事件组的概念

事件组可以简单地认为就是一个整数：

- 的每一位表示一个事件
- 每一位事件的含义由程序员决定，比如：Bit0表示用来串口是否就绪，Bit1表示按键是否被按下
- 这些位，值为1表示事件发生了，值为0表示事件没发生
- 一个或多个任务、ISR都可以去写这些位；一个或多个任务、ISR都可以去读这些位
- 可以等待某一位、某些位中的任意一个，也可以等待多位

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509161809736.png)

事件组用一个整数来表示，其中的高8位留给内核使用，只能用其他的位来表示事件。那么这个整数是多少位的？

- 如果configUSE_16_BIT_TICKS是1，那么这个整数就是16位的，低8位用来表示事件
- 如果configUSE_16_BIT_TICKS是0，那么这个整数就是32位的，低24位用来表示事件
- configUSE_16_BIT_TICKS是用来表示Tick Count的，怎么会影响事件组？这只是基于效率来考虑
  - 如果configUSE_16_BIT_TICKS是1，就表示该处理器使用16位更高效，所以事件组也使用16位
  - 如果configUSE_16_BIT_TICKS是0，就表示该处理器使用32位更高效，所以事件组也使用32位

#### 事件组的操作

事件组和队列、信号量等不太一样，主要集中在2个地方：

- 唤醒谁？
  - 队列、信号量：事件发生时，只会唤醒一个任务
  - 事件组：事件发生时，会唤醒所有符号条件的任务，简单地说它有"广播"的作用
- 是否清除事件？
  - 队列、信号量：是消耗型的资源，队列的数据被读走就没了；信号量被获取后就减少了
  - 事件组：被唤醒的任务有两个选择，可以让事件保留不动，也可以清除事件

以上图为列，事件组的常规操作如下：

- 先创建事件组
- 任务C、D等待事件：
  - 等待什么事件？可以等待某一位、某些位中的任意一个，也可以等待多位。简单地说就是"或"、"与"的关系。
  - 得到事件时，要不要清除？可选择清除、不清除。
- 任务A、B产生事件：设置事件组里的某一位、某些位

#### 事件组函数

##### 创建

使用事件组之前，要先创建，得到一个句柄；使用事件组时，要使用句柄来表明使用哪个事件组。

有两种创建方法：动态分配内存、静态分配内存。函数原型如下：

```c
/* 创建一个事件组，返回它的句柄。
 * 此函数内部会分配事件组结构体 
 * 返回值: 返回句柄，非NULL表示成功
 */
EventGroupHandle_t xEventGroupCreate( void );

/* 创建一个事件组，返回它的句柄。
 * 此函数无需动态分配内存，所以需要先有一个StaticEventGroup_t结构体，并传入它的指针
 * 返回值: 返回句柄，非NULL表示成功
 */
EventGroupHandle_t xEventGroupCreateStatic( StaticEventGroup_t * pxEventGroupBuffer );
```

##### 删除

对于动态创建的事件组，不再需要它们时，可以删除它们以回收内存。

**vEventGroupDelete**可以用来删除事件组，函数原型如下：

```c
/*
 * xEventGroup: 事件组句柄，你要删除哪个事件组
 */
void vEventGroupDelete( EventGroupHandle_t xEventGroup )
```

##### 设置事件

可以设置事件组的某个位、某些位，使用的函数有2个：

- 在任务中使用**xEventGroupSetBits()**
- 在ISR中使用**xEventGroupSetBitsFromISR()**

有一个或多个任务在等待事件，如果这些事件符合这些任务的期望，那么任务还会被唤醒。

函数原型如下：

```c
/* 设置事件组中的位
 * xEventGroup: 哪个事件组
 * uxBitsToSet: 设置哪些位? 
 *              如果uxBitsToSet的bitX, bitY为1, 那么事件组中的bitX, bitY被设置为1
 *              可以用来设置多个位，比如 0x15 就表示设置bit4, bit2, bit0
 * 返回值: 返回原来的事件值(没什么意义, 因为很可能已经被其他任务修改了)
 */
EventBits_t xEventGroupSetBits( EventGroupHandle_t xEventGroup,
                                    const EventBits_t uxBitsToSet );

/* 设置事件组中的位
 * xEventGroup: 哪个事件组
 * uxBitsToSet: 设置哪些位? 
 *              如果uxBitsToSet的bitX, bitY为1, 那么事件组中的bitX, bitY被设置为1
 *              可以用来设置多个位，比如 0x15 就表示设置bit4, bit2, bit0
 * pxHigherPriorityTaskWoken: 有没有导致更高优先级的任务进入就绪态? pdTRUE-有, pdFALSE-没有
 * 返回值: pdPASS-成功, pdFALSE-失败
 */
BaseType_t xEventGroupSetBitsFromISR( EventGroupHandle_t xEventGroup,
									  const EventBits_t uxBitsToSet,
									  BaseType_t * pxHigherPriorityTaskWoken );
```

值得注意的是，ISR中的函数，比如队列函数**xQueueSendToBackFromISR**、信号量函数**xSemaphoreGiveFromISR**，它们会唤醒某个任务，最多只会唤醒1个任务。

但是设置事件组时，有可能导致多个任务被唤醒，这会带来很大的不确定性。所以**xEventGroupSetBitsFromISR**函数不是直接去设置事件组，而是给一个FreeRTOS后台任务(daemon task)发送队列数据，由这个任务来设置事件组。

如果后台任务的优先级比当前被中断的任务优先级高，**xEventGroupSetBitsFromISR**会设置**pxHigherPriorityTaskWoken**为pdTRUE。

如果daemon task成功地把队列数据发送给了后台任务，那么**xEventGroupSetBitsFromISR**的返回值就是pdPASS。

##### 等待事件

使用**xEventGroupWaitBits**来等待事件，可以等待某一位、某些位中的任意一个，也可以等待多位；等到期望的事件后，还可以清除某些位。

函数原型如下：

```c
EventBits_t xEventGroupWaitBits( EventGroupHandle_t xEventGroup,
                                 const EventBits_t uxBitsToWaitFor,
                                 const BaseType_t xClearOnExit,
                                 const BaseType_t xWaitForAllBits,
                                 TickType_t xTicksToWait );
```

先引入一个概念：unblock condition。一个任务在等待事件发生时，它处于阻塞状态；当期望的时间发生时，这个状态就叫"unblock condition"，非阻塞条件，或称为"非阻塞条件成立"；当"非阻塞条件成立"后，该任务就可以变为就绪态。

函数参数说明列表如下：

|    **参数**     | **说明**                                                     |
| :-------------: | :----------------------------------------------------------- |
|   xEventGroup   | 等待哪个事件组？                                             |
| uxBitsToWaitFor | 等待哪些位？哪些位要被测试？                                 |
| xWaitForAllBits | 怎么测试？是"AND"还是"OR"？ pdTRUE: 等待的位，全部为1; pdFALSE: 等待的位，某一个为1即可 |
|  xClearOnExit   | 函数提出前是否要清除事件？ pdTRUE: 清除uxBitsToWaitFor指定的位 pdFALSE: 不清除 |
|  xTicksToWait   | 如果期待的事件未发生，阻塞多久。 可以设置为0：判断后即刻返回； 可设置为portMAX_DELAY：一定等到成功才返回； 可以设置为期望的Tick Count，一般用*pdMS_TO_TICKS()*把ms转换为Tick Count |
|     返回值      | 返回的是事件值， 如果期待的事件发生了，返回的是"非阻塞条件成立"时的事件值； 如果是超时退出，返回的是超时时刻的事件值。 |

举例如下：

| 事件组的值 | uxBitsToWaitFor | xWaitForAllBits | 说明                                                         |
| :--------: | :-------------: | :-------------: | :----------------------------------------------------------- |
|    0100    |      0101       |     pdTRUE      | 任务期望bit0,bit2都为1， 当前值只有bit2满足，任务进入阻塞态； 当事件组中bit0,bit2都为1时退出阻塞态 |
|    0100    |      0110       |     pdFALSE     | 任务期望bit0,bit2某一个为1， 当前值满足，所以任务成功退出    |
|    0100    |      0110       |     pdTRUE      | 任务期望bit1,bit2都为1， 当前值不满足，任务进入阻塞态； 当事件组中bit1,bit2都为1时退出阻塞态 |

你可以使用*xEventGroupWaitBits()*等待期望的事件，它发生之后再使用*xEventGroupClearBits()*来清除。但是这两个函数之间，有可能被其他任务或中断抢占，它们可能会修改事件组。

可以使用设置*xClearOnExit*为pdTRUE，使得对事件组的测试、清零都在*xEventGroupWaitBits()*函数内部完成，这是一个原子操作。

##### 同步点

有一个事情需要多个任务协同，比如：

- 任务A：炒菜
- 任务B：买酒
- 任务C：摆台
- A、B、C做好自己的事后，还要等别人做完；大家一起做完，才可开饭

使用 **xEventGroupSync()** 函数可以同步多个任务：

- 可以设置某位、某些位，表示自己做了什么事
- 可以等待某位、某些位，表示要等等其他任务
- 期望的时间发生后， **xEventGroupSync()** 才会成功返回。
- **xEventGroupSync**成功返回后，会清除事件

**xEventGroupSync** 函数原型如下：

```text
EventBits_t xEventGroupSync(    EventGroupHandle_t xEventGroup,
                                const EventBits_t uxBitsToSet,
                                const EventBits_t uxBitsToWaitFor,
                                TickType_t xTicksToWait );
```

参数列表如下：

|    **参数**     | **说明**                                                     |
| :-------------: | ------------------------------------------------------------ |
|   xEventGroup   | 哪个事件组？                                                 |
|   uxBitsToSet   | 要设置哪些事件？我完成了哪些事件？ 比如0x05(二进制为0101)会导致事件组的bit0,bit2被设置为1 |
| uxBitsToWaitFor | 等待那个位、哪些位？ 比如0x15(二级制10101)，表示要等待bit0,bit2,bit4都为1 |
|  xTicksToWait   | 如果期待的事件未发生，阻塞多久。 可以设置为0：判断后即刻返回； 可设置为portMAX_DELAY：一定等到成功才返回； 可以设置为期望的Tick Count，一般用*pdMS_TO_TICKS()*把ms转换为Tick Count |
|     返回值      | 返回的是事件值， 如果期待的事件发生了，返回的是"非阻塞条件成立"时的事件值； 如果是超时退出，返回的是超时时刻的事件值。 |

参数列表如下：

|    **参数**     | **说明**                                                     |
| :-------------: | :----------------------------------------------------------- |
|   xEventGroup   | 哪个事件组？                                                 |
|   uxBitsToSet   | 要设置哪些事件？我完成了哪些事件？ 比如0x05(二进制为0101)会导致事件组的bit0,bit2被设置为1 |
| uxBitsToWaitFor | 等待那个位、哪些位？ 比如0x15(二级制10101)，表示要等待bit0,bit2,bit4都为1 |
|  xTicksToWait   | 如果期待的事件未发生，阻塞多久。 可以设置为0：判断后即刻返回； 可设置为portMAX_DELAY：一定等到成功才返回； 可以设置为期望的Tick Count，一般用*pdMS_TO_TICKS()*把ms转换为Tick Count |
|     返回值      | 返回的是事件值， 如果期待的事件发生了，返回的是"非阻塞条件成立"时的事件值； 如果是超时退出，返回的是超时时刻的事件值。 |

### 任务通知

所谓"任务通知"，你可以反过来读"通知任务"。

我们使用队列、信号量、事件组等等方法时，并不知道对方是谁。使用任务通知时，可以明确指定：通知哪个任务。

使用队列、信号量、事件组时，我们都要事先创建对应的结构体，双方通过中间的结构体通信：

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509161952531.png)

使用任务通知时，任务结构体TCB中就包含了内部对象，可以直接接收别人发过来的"通知"：

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509161952379.png)

#### 任务通知的特性

##### 优势及限制

任务通知的优势：

- 效率更高：使用任务通知来发送事件、数据给某个任务时，效率更高。比队列、信号量、事件组都有大的优势。
- 更节省内存：使用其他方法时都要先创建对应的结构体，使用任务通知时无需额外创建结构体。

任务通知的限制：

- 不能发送数据给ISR：
- ISR并没有任务结构体，所以无法使用任务通知的功能给ISR发送数据。但是ISR可以使用任务通知的功能，发数据给任务。
- 数据只能给该任务独享
- 使用队列、信号量、事件组时，数据保存在这些结构体中，其他任务、ISR都可以访问这些数据。使用任务通知时，数据存放入目标任务中，只有它可以访问这些数据。
- 在日常工作中，这个限制影响不大。因为很多场合是从多个数据源把数据发给某个任务，而不是把一个数据源的数据发给多个任务。
- 无法缓冲数据
- 使用队列时，假设队列深度为N，那么它可以保持N个数据。
- 使用任务通知时，任务结构体中只有一个任务通知值，只能保持一个数据。
- <span style="color:#00FFFF;">无法广播给多个任务</span>
- 使用事件组可以同时给多个任务发送事件。
- 使用任务通知，只能发个一个任务。
- 如果发送受阻，发送方无法进入阻塞状态等待
- 假设队列已经满了，使用 **xQueueSendToBack()** 给队列发送数据时，任务可以进入阻塞状态等待发送完成。
- 使用任务通知时，即使对方无法接收数据，发送方也无法阻塞等待，只能即刻返回错误。

#####  通知状态和通知值

每个任务都有一个结构体：TCB(Task Control Block)，里面有2个成员：

- 一个是uint8_t类型，用来表示通知状态
- 一个是uint32_t类型，用来表示通知值

```c
typedef struct tskTaskControlBlock
{
    ......
    /* configTASK_NOTIFICATION_ARRAY_ENTRIES = 1 */
    volatile uint32_t ulNotifiedValue[ configTASK_NOTIFICATION_ARRAY_ENTRIES ];
    volatile uint8_t ucNotifyState[ configTASK_NOTIFICATION_ARRAY_ENTRIES ];
    ......
} tskTCB;
```

通知状态有3种取值：

- taskNOT_WAITING_NOTIFICATION：任务没有在等待通知
- taskWAITING_NOTIFICATION：任务在等待通知
- taskNOTIFICATION_RECEIVED：任务接收到了通知，也被称为pending(有数据了，待处理)

```c
##define taskNOT_WAITING_NOTIFICATION              ( ( uint8_t ) 0 )  /* 也是初始状态 */
##define taskWAITING_NOTIFICATION                  ( ( uint8_t ) 1 )
##define taskNOTIFICATION_RECEIVED                 ( ( uint8_t ) 2 )
```

通知值可以有很多种类型：

- 计数值
- 位(类似事件组)
- 任意数值

#### 任务通知的使用

使用任务通知，可以实现轻量级的队列(长度为1)、邮箱(覆盖的队列)、计数型信号量、二进制信号量、事件组。

##### 两类函数

任务通知有2套函数，简化版、专业版，列表如下：

- 简化版函数的使用比较简单，它实际上也是使用专业版函数实现的
- 专业版函数支持很多参数，可以实现很多功能

|          | 简化版                                      | 专业版                              |
| -------- | ------------------------------------------- | ----------------------------------- |
| 发出通知 | `xTaskNotifyGive ` `vTaskNotifyGiveFromISR` | `xTaskNotify ` `xTaskNotifyFromISR` |
| 取出通知 | `ulTaskNotifyTake`                          | `xTaskNotifyWait`                   |

##### 简化版任务通知

在任务中使用`xTaskNotifyGive`函数，在ISR中使用`vTaskNotifyGiveFromISR`函数，都是直接给其他任务发送通知：

- 使得通知值加一
- 并使得通知状态变为"pending"，也就是**taskNOTIFICATION_RECEIVED**，表示有数据了、待处理

可以使用`ulTaskNotifyTake`函数来取出通知值：

- 如果通知值等于0，则阻塞(可以指定超时时间)
- 当通知值大于0时，任务从阻塞态进入就绪态
- 在ulTaskNotifyTake返回之前，还可以做些清理工作：把通知值减一，或者把通知值清零

使用ulTaskNotifyTake函数可以实现轻量级的、高效的二进制信号量、计数型信号量。

这几个函数的原型如下：

```c
BaseType_t xTaskNotifyGive( TaskHandle_t xTaskToNotify );

void vTaskNotifyGiveFromISR( TaskHandle_t xTaskHandle, BaseType_t *pxHigherPriorityTaskWoken );

uint32_t ulTaskNotifyTake( BaseType_t xClearCountOnExit, TickType_t xTicksToWait );
```

`xTaskNotifyGive`函数的参数说明如下：

|    **参数**     | **说明**                                   |
| :-------------: | ------------------------------------------ |
| `xTaskToNotify` | 任务句柄(创建任务时得到)，给哪个任务发通知 |
|     返回值      | 必定返回pdPASS                             |

`vTaskNotifyGiveFromISR`函数的参数说明如下：

|          **参数**           | **说明**                                                     |
| :-------------------------: | ------------------------------------------------------------ |
|        `xTaskHandle`        | 任务句柄(创建任务时得到)，给哪个任务发通知                   |
| `pxHigherPriorityTaskWoken` | 被通知的任务，可能正处于阻塞状态。 此函数发出通知后，会把它从阻塞状态切换为就绪态。 如果被唤醒的任务的优先级，高于当前任务的优先级， 则"*pxHigherPriorityTaskWoken"被设置为pdTRUE， 这表示在中断返回之前要进行任务切换。 |

`ulTaskNotifyTake`函数的参数说明如下：

|      **参数**       | **说明**                                                     |
| :-----------------: | ------------------------------------------------------------ |
| `xClearCountOnExit` | 函数返回前是否清零： pdTRUE：把通知值清零 pdFALSE：如果通知值大于0，则把通知值减一 |
|   `xTicksToWait`    | 任务进入阻塞态的超时时间，它在等待通知值大于0。 0：不等待，即刻返回； portMAX_DELAY：一直等待，直到通知值大于0； 其他值：Tick Count，可以用*pdMS_TO_TICKS()*把ms转换为Tick Count |
|       返回值        | 函数返回之前，在清零或减一之前的通知值。 如果xTicksToWait非0，则返回值有2种情况： 1. 大于0：在超时前，通知值被增加了 2. 等于0：一直没有其他任务增加通知值，最后超时返回0 |

##### 专业版任务通知

**xTaskNotify** 函数功能更强大，可以使用不同参数实现各类功能，比如：

- 让接收任务的通知值加一：这时 **xTaskNotify()** 等同于 **xTaskNotifyGive()**
- 设置接收任务的通知值的某一位、某些位，这就是一个轻量级的、更高效的事件组
- 把一个新值写入接收任务的通知值：上一次的通知值被读走后，写入才成功。这就是轻量级的、长度为1的队列
- 用一个新值覆盖接收任务的通知值：无论上一次的通知值是否被读走，覆盖都成功。类似 **xQueueOverwrite()** 函数，这就是轻量级的邮箱。

**xTaskNotify()** 比 **xTaskNotifyGive()** 更灵活、强大，使用上也就更复杂。**xTaskNotifyFromISR()** 是它对应的ISR版本。

这两个函数用来发出任务通知，使用哪个函数来取出任务通知呢？

使用 **xTaskNotifyWait()** 函数！它比 **ulTaskNotifyTake()** 更复杂：

- 可以让任务等待(可以加上超时时间)，等到任务状态为"pending"(也就是有数据)
- 还可以在函数进入、退出时，清除通知值的指定位

这几个函数的原型如下：

```c
BaseType_t xTaskNotify( TaskHandle_t xTaskToNotify, uint32_t ulValue, eNotifyAction eAction );

BaseType_t xTaskNotifyFromISR( TaskHandle_t xTaskToNotify,
                               uint32_t ulValue, 
                               eNotifyAction eAction, 
                               BaseType_t *pxHigherPriorityTaskWoken );

BaseType_t xTaskNotifyWait( uint32_t ulBitsToClearOnEntry, 
                            uint32_t ulBitsToClearOnExit, 
                            uint32_t *pulNotificationValue, 
                            TickType_t xTicksToWait );
```

`xTaskNotify`函数的参数说明如下：

|    **参数**     | **说明**                                                     |
| :-------------: | ------------------------------------------------------------ |
| `xTaskToNotify` | 任务句柄(创建任务时得到)，给哪个任务发通知                   |
|    `ulValue`    | 怎么使用ulValue，由eAction参数决定                           |
|    `eAction`    | 见下表                                                       |
|     返回值      | pdPASS：成功，大部分调用都会成功 pdFAIL：只有一种情况会失败，当eAction为eSetValueWithoutOverwrite， 并且通知状态为"pending"(表示有新数据未读)，这时就会失败。 |

`eNotifyAction`参数说明：

|   **eNotifyAction取值**   | **说明**                                                     |
| :-----------------------: | ------------------------------------------------------------ |
|        `eNoAction`        | 仅仅是更新通知状态为"pending"，未使用ulValue。 这个选项相当于轻量级的、更高效的二进制信号量。 |
|        `eSetBits`         | 通知值 = 原来的通知值 \| ulValue，按位或。 相当于轻量级的、更高效的事件组。 |
|       `eIncrement`        | 通知值 = 原来的通知值 + 1，未使用ulValue。 相当于轻量级的、更高效的二进制信号量、计数型信号量。 相当于**xTaskNotifyGive()**函数。 |
| eSetValueWithoutOverwrite | 不覆盖。 如果通知状态为"pending"(表示有数据未读)， 则此次调用xTaskNotify不做任何事，返回pdFAIL。 如果通知状态不是"pending"(表示没有新数据)， 则：通知值 = ulValue。 |
|  eSetValueWithOverwrite   | 覆盖。 无论如何，不管通知状态是否为"pendng"， 通知值 = ulValue。 |

xTaskNotifyFromISR函数跟xTaskNotify很类似，就多了最后一个参数**pxHigherPriorityTaskWoken**。在很多ISR函数中，这个参数的作用都是类似的，使用场景如下：

- 被通知的任务，可能正处于阻塞状态
- **xTaskNotifyFromISR**函数发出通知后，会把接收任务从阻塞状态切换为就绪态
- 如果被唤醒的任务的优先级，高于当前任务的优先级，则"*pxHigherPriorityTaskWoken"被设置为pdTRUE，这表示在中断返回之前要进行任务切换。

xTaskNotifyWait函数列表如下：

|        **参数**        | **说明**                                                     |
| :--------------------: | ------------------------------------------------------------ |
| `ulBitsToClearOnEntry` | 在xTaskNotifyWait入口处，要清除通知值的哪些位？ 通知状态不是"pending"的情况下，才会清除。 它的本意是：我想等待某些事件发生，所以先把"旧数据"的某些位清零。 能清零的话：通知值 = 通知值 & ~(ulBitsToClearOnEntry)。 比如传入0x01，表示清除通知值的bit0； 传入0xffffffff即ULONG_MAX，表示清除所有位，即把值设置为0 |
| `ulBitsToClearOnExit`  | 在xTaskNotifyWait出口处，如果不是因为超时推出，而是因为得到了数据而退出时： 通知值 = 通知值 & ~(ulBitsToClearOnExit)。 在清除某些位之前，通知值先被赋给"*pulNotificationValue"。 比如入0x03，表示清除通知值的bit0、bit1； 传入0xffffffff即ULONG_MAX，表示清除所有位，即把值设置为0 |
| `pulNotificationValue` | 用来取出通知值。 在函数退出时，使用ulBitsToClearOnExit清除之前，把通知值赋给"*pulNotificationValue"。 如果不需要取出通知值，可以设为NULL。 |
|     `xTicksToWait`     | 任务进入阻塞态的超时时间，它在等待通知状态变为"pending"。 0：不等待，即刻返回； portMAX_DELAY：一直等待，直到通知状态变为"pending"； 其他值：Tick Count，可以用*pdMS_TO_TICKS()*把ms转换为Tick Count |
|         返回值         | 1. pdPASS：成功 这表示xTaskNotifyWait成功获得了通知： 可能是调用函数之前，通知状态就是"pending"； 也可能是在阻塞期间，通知状态变为了"pending"。 2. pdFAIL：没有得到通知。 |
