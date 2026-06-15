---
title: ESP32学习
date: 2025-09-09 11:04:10
cover: "https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091106486.jpg"
tags:
  - ESP32
  - RTOS
categories: MCU
---

## ESP32环境搭建

### WSL2安装与配置

推荐查阅官方文档说明：[安装 WSL | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/install)

#### 手动下载安装包

1. 访问 WSL 的 GitHub Releases 页面 https://github.com/microsoft/WSL/releases

2. 查找并下载最新版本的 .msi 安装包（例如 wsl.2.1.5.0.x64.msi）

![image-20250909111242531](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091112594.png)

#### 配置控制面板

![image-20250909111552848](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091115919.png)

![image-20250909111621317](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091116357.png)

> 有些Windows 11新系统会没有`虚拟机平台`这个选项 没有关系！！！！
>
> 配置完后重新启动电脑

#### 管理员身份运行Windows PowerShell

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091108898.png)

#### 安装Ubuntu

```bash
wsl --install
```

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091108464.png)

> 如有可选 则选择Uuntu 如 `wsl --install -d Ubuntu`

#### 等待安装完成后重启电脑打开Ubuntu

![image-20250909112101810](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091121963.png)

#### 输入账号密码

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091121306.png)

#### 更新软件源

```bash
sudo apt update
```

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091124279.png)

#### 更新完后输入下方指令，下载espidf依赖库和工具

> ```bash
> sudo apt-get install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
> ```

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091126525.png)

#### 输入y

![image-20250909112720315](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091127364.png)

### VScode配置

#### 选择连接到WSL

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091129218.png)

#### 插件安装

> - 安装`Chinese`插件
> - 安装`C/C++`插件
> - 安装`ESP-IDF`插件

#### 在linux账户新建文件用于存放espidf内容，方便后续管理

![image-20250909113527188](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091135250.png)

#### ESP-IDF安装

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091131190.png)

#### 在安装路径中选择自己新建的文件夹

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091135132.png)

> 如果出现下图这个情况
>
> ![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091137839.png)
>
> 那就运行一下下面的指令 
>
> ```bash
> sudo chown -R $USER:$USER /home/你的用户名/你的IDF文件名/
> chmod -R 755 /home/你的用户名/你的IDF文件名/
> ```

#### 安装完成之后先搞个工程测试一下编译能不能正常

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091140479.png)

#### 选择文件夹来存放代码

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091143918.png)

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091143292.png)

#### 设置环境

##### 设置IDF_tools环境（豆包四条命令搞定）

> 注意下方命令中：
>
> `~/ESP_IDF/v5.5.1/esp-idf`：你的IDF文件夹路径 要一直引入到`esp-idf`
>
> `~/ESP_IDF/Tools`：你的IDF_Tools文件夹路径

```bash
echo 'export IDF_PATH=~/ESP_IDF/v5.5.1/esp-idf' >> ~/.bashrc
echo 'export IDF_TOOLS_PATH=~/ESP_IDF/Tools' >> ~/.bashrc
echo 'source ~/ESP_IDF/v5.5.1/esp-idf/export.sh' >> ~/.bashrc
source ~/.bashrc  # 立即生效
```

> 成功的话 重启`vscode`就会自动出现`idf`的命令
>
> ![image-20250909221836025](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509092218229.png)

##### 若上面还不行 则根据报错情况执行下面

###### 场景 1：运行 `./export.sh` 时报 “Python 虚拟环境不存在”

`报错信息`

```bash
ERROR: ESP-IDF Python virtual environment "$HOME/ESP_IDF/Tools/python_env/idf5.5_py3.12_env/bin/python" not found. Please run the install script to set it up before proceeding.
```

`原因`

未执行 ESP-IDF 安装脚本，Python 虚拟环境未创建。

`解决方案`

必须先运行安装脚本，自动创建虚拟环境和安装依赖：

```bash
# 进入 ESP-IDF 根目录（替换为你的实际路径）
cd ~/ESP_IDF/v5.5.1/esp-idf
# 执行安装脚本（Linux/macOS 用 install.sh，Windows 用 install.bat）
./install.sh
# 安装完成后再执行 export.sh 导出环境
. ./export.sh
```

###### 场景 2：运行 `./install.sh` 时 “下载工具包卡住”

`报错表现`

安装过程中卡在某一工具包下载（如 `riscv32-esp-elf-gdb`），进度条不动或反复重试。

`原因`

ESP-IDF 工具包托管在 GitHub，国内网络访问速度慢或不稳定，导致下载中断。

1. **用国内镜像加速（推荐）**
   配置 `IDF_GITHUB_ASSETS` 环境变量，将下载源切换为 Espressif 国内镜像，无需手动下载：

   ```bash
   # 临时生效，每次打开终端需重新执行（或写入 shell 配置文件）
   export IDF_GITHUB_ASSETS="dl.espressif.com/github_assets"
   # 重新运行安装脚本
   ./install.sh
   ```

#### 烧录文件

##### 下载wsl-usb-manager.exe

> <a href="https://mextra.netlify.app/download/ESP32/wsl-usb-manager.exe">wsl-usb-manager.exe </a>

##### 插上ESP32S3

![059f326e4709ef71f539677c5da784c2](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091454477.jpg)

##### 配置WSL USB

###### 管理员身份打开Windows PowerShell

> 输入`winget install usbipd`，然后输入`y`（`看情况选择开不开魔法`，不开也能20s左右下载安装完）

这个指令是安装usbipd-win工具，用于将windows系统的usb设备共享给wsl环境。

![img](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509092343890.png)

安装好之后 更改权限

> - 输入`sudo usermod -aG dialout 你的用户名`
>
> - 然后`重启电脑`
> - 输入`groups`验证一下
>   - 如果输出中包含 `dialout` 或 `uucp`，则表示设置成功，之后就可以正常访问串口了。

##### 进入WSL USB选择端口并烧录

![image-20250909145724459](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091457549.png)

![image-20250909150034639](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091500732.png)

以上环境参考：[（二）vscode搭建espidf环境，配置wsl2_wsl 编译idf项目-CSDN博客](https://blog.csdn.net/QL_SD/article/details/150216599?spm=1001.2014.3001.5502)

感谢[QL.ql](https://blog.csdn.net/QL_SD)大博主

## ESP32基础知识

![image-20250909173749995](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091737219.png)

![image-20250909173836126](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509091738212.png)

### 使用命令行创建工程

```bash
idf.py create-project 工程名
```

#### 打开vscode 切换到新建的工程

```bash
cd ~/ESP_Code/刚刚的工程名
code .

按`ctrl+shift+p` 寻找add 添加idf路径
```

> 在写代码时候 如果出现宏没定义 则重新设置一下芯片型号 稍等一会就会出现
>
> ![image-20250910100948617](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509101009919.png)
>
> ![image-20250910102600612](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509101026730.png)
>
> 如果想要删除文件夹 则运行以下命令
>
> ```bash
> cd ..
> rm -rf 文件夹
> ```
>
> 必备头文件
>
> ```C
> #include "freertos/FreeRTOS.h"
> #include "freertos/task.h"
> #include "esp_log.h"
> ```
>

### CMakeList.txt使用说明

一般CmakeList.txt呈现下面的结构

```C
main
    main.c
    CMakeList.txt
CMakeList.txt
```

#### 创建自定义文件夹

> 如
>
> ```C
> components
>     led
>     	led.c
>     	led.h
>     	CMakeList.txt
> main
>     main.c
>     CMakeList.txt
> ```
>
> 在这个目录下 则`components/led`文件夹下的`CMakeList.txt`，可以这么写
>
> ```cmake
> idf_component_register(SRCS "led.c"
>                     INCLUDE_DIRS "."
>                     REQUIRES esp_driver_gpio
>                     )
> ```
>
> - `SRCS`：`.c`文件 多个用空格隔开
>
> - `INCLUDE_DIRS`：文件名 多个同样用空格隔开
>
> - `REQUIRES`：引入ESP官方的组件库`components` ，名字为官方`idf`下的`components`文件夹下的文件夹名 如此处的`esp_driver_gpio`
>
>   ![image-20250912184005520](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509121840992.png)
>
> 而若要引用 则`main`文件夹下的`CMakeList.txt` 可以这么写
>
> ```cmake
> idf_component_register(SRCS "main.c" "../components/led/led.c"
>                     INCLUDE_DIRS "." "../components")
> ```

### FreeRTOS-ESP32 基础API

#### 任务创建

```C
BaseType_t xTaskCreatePinnedToCore(
	TaskFunction_t pvTaskCode,				//任务函数指针，原型是voidfun(void*param)
	const char *constpcName,				//任务的名称，打印调试可能会有用
	const uint32_t usStackDepth,			//指定的任务堆栈空间大小（字节）
	void *constpvParameters,				//任务参数
	UBaseType_t uxPriority,					//优先级，数字越大，优先级越大，0到(configMAX_PRIORITIES-1) 即0-24
	TaskHandle_t *constpvCreatedTask,		 //传回来的任务句柄
	const BaseType_t xCoreID)				//分配在哪个内核上运行 ESP32是双核 所以可以选0或1
```

> ESP32的任务堆栈空间大小`usStackDepth`与原生FreeRTOS不一样
>
> - 原生FreeRTOS的任务堆栈空间大小单位是`字`，即目标平台若是32位 则为4个字节
> - ESP32的任务堆栈空间大小单位是`字节`，最小是`2048`

#### 阻塞函数（少用 可替代）

```C
//粗略延时xTicksToDelay个周期 
//使用该函数后 任务进入阻塞 等延时结束且调度到来才会再次运行
void vTaskDelay(const TickType_t xTicksToDelay)	

//用于表示精确的解除阻塞时间    
//即使任务的执行时间超过了周期时间，它也能自动调整延迟时间，保证任务的周期性运行
void vTaskDelayUntil(TickType_t *pxPreviousWakeTime,const TickType_t xTimeIncrement)

```

#### 打印调试

```C
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
void taskA(void *param)
{
    while (1)
    {
        ESP_LOGI("TaskA", "Hello from Task A%d", 1);//前面是一个TAG标签，后面是打印内容 他会自动回车换行
        vTaskDelay(pdMS_TO_TICKS(500));
    }
}

void app_main(void)
{
    xTaskCreatePinnedToCore(taskA, "TaskA", 2048, NULL, 5, NULL, 1);
}
	
```

效果如图

![image-20250910101607354](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509101016442.png)

#### 队列

> 队列特性：先进先出

```C
QueueHandle_t queue_handle = NULL; // 创建队列句柄

/**
 * @brief 创建一个队列
 * @param uxQueueLength 队列长度
 * @param uxItemSize 每个元素的大小(单位是字节)
 */
QueueHandle_t xQueueCreate(UBaseType_t uxQueueLength, UBaseType_t uxItemSize);

/**
 * @brief 向队列头部发送数据
 * @param xQueue 队列句柄
 * @param pvItemToQueue 要发送的数据的指针
 * @param xTicksToWait 等待的时间(单位是tick)
 * @return 如果在等待时间内发送成功返回pdTRUE，否则返回errQUEUE_FULL
 */
BaseType_t xQueueSend(QueueHandle_t xQueue, const void *pvItemToQueue, TickType_t xTicksToWait);

/**
 * @brief 向队列尾部发送数据（新数据放在队尾）
 * @param xQueue 队列句柄
 * @param pvItemToQueue 要发送的数据的指针
 * @param xTicksToWait 等待的时间
 * @return pdTRUE 如果发送成功
 *         errQUEUE_FULL 如果队列已满，发送失败
 */
BaseType_t xQueueSendToBack(QueueHandle_t xQueue, const void *pvItemToQueue, TickType_t xTicksToWait);

/**
 * @brief 从队列中接收数据
 * @param xQueue 队列句柄
 * @param pvBuffer 用于存储接收数据的缓冲区
 * @param xTicksToWait 等待数据的最大时间，以系统节拍为单位
 * @return pdPASS 表示接收成功，errQUEUE_EMPTY 表示队列为空且等待超时，接收失败。
 */
BaseType_t xQueueReceive(QueueHandle_t xQueue, void *pvBuffer, TickType_t xTicksToWait);

/**
 * @brief 该函数在中断中使用，从队列发送数据 这个函数的行为和普通的 xQueueSend 有差异，且不允许阻塞（等待）
 * @param xQueue 队列句柄
 * @param pvItemToQueue 要发送的数据
 * @param pxHigherPriorityTaskWoken 如果发送数据后，有更高优先级的任务被唤醒，则该参数会被置为pdTRUE
 * @return 如果发送成功，返回pdTRUE，否则返回errQUEUE_FULL
 */
BaseType_t xQueueSendFromISR(QueueHandle_t xQueue, const void *pvItemToQueue, BaseType_t *pxHigherPriorityTaskWoken);
```

##### 调用示例

```C
#include <stdio.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "esp_log.h"

QueueHandle_t queue_handle = NULL; // 定义一个队列句柄

typedef struct
{
    int value;
} queue_date_t;

void taskA(void *param)
{
    // 从队列里面接收数据，并打印
    queue_date_t data;
    while (1)
    {
        if (xQueueReceive(queue_handle, &data, 100) == pdTRUE) // 从队列中接收数据，阻塞等待
        {
            ESP_LOGI("re_queue", "receive queue value:%d", data.value); // 前面是一个TAG标签，后面是打印内容 他会自动回车换行
        }
    }
}

void taskB(void *param)
{
    queue_date_t data;
    memset(&data, 0, sizeof(data));
    // 每隔1s向队列里面发送数据
    while (1)
    {
        xQueueSendToBack(queue_handle, &data, 100); // 向队列中发送数据，阻塞等待
        vTaskDelay(pdMS_TO_TICKS(1000));            // 延时1s
        data.value++;
    }
}

void app_main(void)
{
    queue_handle = xQueueCreate(10, sizeof(queue_date_t)); // 创建一个队列，队列长度为10，每个元素的大小为queue_date_t结构体的大小
    xTaskCreatePinnedToCore(taskA, "re_queue", 2048, NULL, 3, NULL, 1);
    xTaskCreatePinnedToCore(taskB, "se_queue", 2048, NULL, 3, NULL, 1);
}
```

> ![image-20250910110704875](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509101107988.png)

#### 信号量

```C
/**
 * @brief 创建一个二值信号量
 * @return SemaphoreHandle_t 信号量句柄
 */
SemaphoreHandle_t xSemaphoreCreateBinary(void);

/**
 * @brief 创建一个计数信号量
 * @param uxMaxCount 计数信号量的最大值
 * @param uxInitialCount 计数信号量的初始值
 * 
 */
SemaphoreHandle_t xSemaphoreCreateCounting(UBaseType_t uxMaxCount, UBaseType_t uxInitialCount);

/**
 * @brief 创建一个互斥锁
 * @return SemaphoreHandle_t 互斥锁句柄
 */
SemaphoreHandle_t xSemaphoreCreateMutex(void);

/**
 * @brief 取得一个信号量
 * @param xSemaphore 信号量句柄
 * @param xTicksToWait 等待信号量的时间，单位为ticks
 * @return 如果成功取得信号量则返回pdTRUE，否则返回pdFALSE
 */
BaseType_t xSemaphoreTake(SemaphoreHandle_t xSemaphore, TickType_t xTicksToWait);

/**
 * @brief 释放一个信号量
 * @param xSemaphore 信号量句柄
 * @return 如果成功释放信号量则返回pdTRUE，否则返回pdFALSE
 */
BaseType_t xSemaphoreGive(SemaphoreHandle_t xSemaphore);

/**
 * @brief 删除一个信号量
 * @param xSemaphore 要删除的信号量句柄
 */
void vSemaphoreDelete(SemaphoreHandle_t xSemaphore);
```

##### 调用示例

```C
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "freertos/semphr.h"
#include "esp_log.h"

SemaphoreHandle_t mutex_handle;

void taskA(void *param)
{
    while (1)
    {
        vTaskDelay(pdMS_TO_TICKS(1000));
        xSemaphoreTake(mutex_handle, portMAX_DELAY);
        ESP_LOGI("TASK_A", "Task A is running");

        xSemaphoreGive(mutex_handle);
        vTaskDelay(pdMS_TO_TICKS(10)); // 给其他任务机会
    }
}

void taskB(void *param)
{
    while (1)
    {
        vTaskDelay(pdMS_TO_TICKS(500));
        xSemaphoreTake(mutex_handle, portMAX_DELAY);
        ESP_LOGI("TASK_B", "Task B is running");

        xSemaphoreGive(mutex_handle);
        vTaskDelay(pdMS_TO_TICKS(10)); // 给其他任务机会
    }
}

void app_main(void)
{
    mutex_handle = xSemaphoreCreateMutex();
    xTaskCreatePinnedToCore(taskA, "taskA", 2048, NULL, 6, NULL, 1);
    xTaskCreatePinnedToCore(taskB, "taskB", 2048, NULL, 4, NULL, 1);
}
```

> 注意释放信号量的时候 一般会给个延时 不加的话任务会马上回到`while(1)`初始的时候 反应出来的现象就是一直是A任务运行
>
> 原因：RTOS任务调度需要时间 并且最少只能是10ms 已测试

#### 事件组

```C
/**
 * @brief 创建事件组
 * @return EventGroupHandle_t 事件组句柄
 */
EventGroupHandle_t xEventGroupCreate(void);

/**
 * @brief 设置事件位
 * @param xEventGroup 事件组句柄
 * @param uxBitsToSet 要设置的事件位
 * @return 设置后事件组的值
 */
EventBits_t xEventGroupSetBits(EventGroupHandle_t xEventGroup, const EventBits_t uxBitsToSet);

/**
 * @brief 事件组等待位函数
 * @param xEventGroup 事件组句柄
 * @param uxBitsToWaitFor 等待的事件位
 * @param xClearOnExit 退出时是否清除事件位 pdTRUE or pdFALSE
 * @param xWaitForAllBits 是否等待所有位 pdTRUE or pdFALSE
 * @param xTicksToWait 等待的最大时间
 * @return 返回实际获得的事件位
 */
EventBits_t xEventGroupWaitBits(const EventGroupHandle_t xEventGroup,
                                const EventBits_t uxBitsToWaitFor,
                                const BaseType_t xClearOnExit,
                                const BaseType_t xWaitForAllBits,
                                TickType_t xTicksToWait);


/**
 * @brief 清除事件组指定事件位
 * @param xEventGroup 事件组句柄
 * @param uxBitsToClear 要清除的事件位
 * @return 清除前的事件组位值
 */
EventBits_t xEventGroupClearBits(EventGroupHandle_t xEventGroup, const EventBits_t uxBitsToClear);

/* 以下使用消息直达机制*/

/**
 * @brief 任务通知函数
 * @param xTaskToNotify 任务句柄
 * @param ulValue 发送的值
 * @param eAction 发送的动作 
 *                eNoAction 不改变任务的通知值(即不更新 会一直发送初始值)
 *                eSetBits 将ulValue的值按位或 到任务的通知值中 
 *                eIncrement 将任务的通知值加1
 *                eSetValueWithOverwrite 将任务的通知值设置为ulValue
 *                eSetValueWithoutOverwrite 如果任务的通知值为0，则将其设置为ulValue，否则不改变任务的通知值
 * @return 如果通知成功返回pdPASS，否则返回errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY
 */
BaseType_t xTaskNotify(TaskHandle_t xTaskToNotify,
                       uint32_t ulValue,
                       eNotifyAction eAction);

/**
 * @brief 将任务设置为等待接收通知
 * @param ulBitsToClearOnEntry 进入等待前要清除的位
 * @param ulBitsToClearOnExit 退出等待后要清除的位
 * @param pulNotificationValue 指向接收通知值的指针
 * @param xTicksToWait 等待通知的最大时间（以滴答为单位）
 * @return 如果在指定的等待时间内收到通知，则返回pdTRUE；否则返回
 */
BaseType_t xTaskNotifyWait(uint32_t ulBitsToClearOnEntry,
                           uint32_t ulBitsToClearOnExit,
                           uint32_t *pulNotificationValue,
                           TickType_t xTicksToWait);
```

##### 调用示例

###### 事件组设置

```C
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "freertos/semphr.h"
#include "freertos/event_groups.h"
#include "esp_log.h"

#define EVENT_BIT_0 BIT0
#define EVENT_BIT_1 BIT1

EventGroupHandle_t event_handle;

void taskA(void *param)
{
    // 定时1s向任务B发送事件消息
    while (1)
    {
        xEventGroupSetBits(event_handle, EVENT_BIT_0);
        vTaskDelay(pdMS_TO_TICKS(1000));
        xEventGroupSetBits(event_handle, EVENT_BIT_1);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void taskB(void *param)
{
    EventBits_t ev;
    while (1)
    {
        ev = xEventGroupWaitBits(event_handle, EVENT_BIT_0 | EVENT_BIT_1, pdTRUE, pdFALSE, pdMS_TO_TICKS(5000));
        if (ev & EVENT_BIT_0)
        {
            ESP_LOGI("ev", "ev get event0");
        }
        if (ev & EVENT_BIT_1)
        {
            ESP_LOGI("ev", "ev get event1");
        }
    }
}

void app_main(void)
{
    event_handle = xEventGroupCreate();
    xTaskCreatePinnedToCore(taskA, "taskA", 2048, NULL, 3, NULL, 1);
    xTaskCreatePinnedToCore(taskB, "taskB", 2048, NULL, 3, NULL, 1);
}
```

###### 消息直达

```C
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "freertos/semphr.h"
#include "freertos/event_groups.h"
#include "esp_log.h"

static TaskHandle_t taskA_handle;
static TaskHandle_t taskB_handle;

void taskA(void *param)
{
    // 定时1s向任务B发送事件消息
    uint32_t count = 0;
    vTaskDelay(pdMS_TO_TICKS(100));
    while (1)
    {
        xTaskNotify(taskB_handle, count, eSetValueWithoutOverwrite);
        count++;
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void taskB(void *param)
{
    uint32_t count = 0;
    while (1)
    {
        xTaskNotifyWait(0, 0, &count, portMAX_DELAY);
        ESP_LOGI("taskB", "Notify count: %lu", count);
    }
}

void app_main(void)
{
    xTaskCreatePinnedToCore(taskA, "taskA", 2048, NULL, 3, &taskA_handle, 1);
    xTaskCreatePinnedToCore(taskB, "taskB", 2048, NULL, 3, &taskB_handle, 1);
}
```

#### ESP-IDF中的FreeRTOS与原生区别

> 1. ESP32是多核的 所以可以完全实现程序并发执行 即不是单核那种 低优先级的任务因为高优先级任务存在而执行不了
> 2. esp-idf自动创建五个任务
>    - 空闲任务（每个核都会创建一个，`优先级为0`）
>    - FreeRTOS定时器任务（`优先级为1`）
>    - app_main任务（我们的应用入口，`优先级为1`）
>    - IPC任务（每个核创建一个，用于多核协调，`优先级为24`）
>    - ESP定时器任务（负责ESP32定时器回调，`优先级为22`）
> 3. esp-idf不使用原生FreeRTOS的内存堆管理，实现了自己的堆
> 4. 创建任务使用`xTaskCreatePinnedToCore()` 可以指定用哪个核创建任务
>    - 负责处理`无线网络的任务（如WiFi蓝牙）`将被固定到`CPU0`，即`PRO_CPU`
>    - 处理`应用程序`其余部分任务将被固定到`CPU1`，即`APP_CPU`
> 5. 如果任务中用到浮点运算，则创建任务的时候必须指定具体运行在哪个核上，不能由系统自动安排

## GPIO配置

### 引脚基础配置

#### `gpio_config()`：批量配置 GPIO（核心入口）

- **函数原型**

  ```c
  esp_err_t gpio_config(const gpio_config_t *pGPIOConfig);
  ```

- **作用**：一次性配置一组 GPIO 的模式、上下拉、中断类型（覆盖该引脚当前所有配置）。

- **参数**：`pGPIOConfig`：指向 `gpio_config_t` 结构体的指针（需提前初始化）。

- **返回值**：`ESP_OK`（成功）、`ESP_ERR_INVALID_ARG`（参数错误，如引脚无效）。

- **示例**：配置 GPIO2、GPIO3 为输出模式，禁用上下拉和中断：

  ```c
  gpio_config_t io_conf = {
      .pin_bit_mask = (1ULL << GPIO_NUM_2) | (1ULL << GPIO_NUM_3),  // 仅配置 GPIO2
      .mode = GPIO_MODE_OUTPUT,
      .pull_up_en = GPIO_PULLUP_DISABLE,
      .pull_down_en = GPIO_PULLDOWN_DISABLE,
      .intr_type = GPIO_INTR_DISABLE
  };
  gpio_config(&io_conf);  // 执行配置
  ```

> `intr_type` :
>
> - `GPIO_INTR_DISABLE`：禁用中断
> -  `GPIO_INTR_POSEDGE`：上升沿触发
> - `GPIO_INTR_NEGEDGE`：下降沿触发
> - `GPIO_INTR_ANYEDGE`：双边沿触发
> - `GPIO_INTR_LOW_LEVEL`：低电平触发
> - `GPIO_INTR_HIGH_LEVEL`：高电平触发

#### `gpio_reset_pin()`：重置引脚为默认状态

- **函数原型**

  ```c
  esp_err_t gpio_reset_pin(gpio_num_t gpio_num);
  ```

- **作用**：将指定 GPIO 重置为默认状态（输入模式、禁用上下拉、禁用中断、选择 GPIO 功能）。

- **参数**：`gpio_num`：要重置的 GPIO 编号（如 `GPIO_NUM_4`）。

- **示例**：重置 GPIO4，恢复初始状态：

  ```c
  gpio_reset_pin(GPIO_NUM_4);
  ```

#### `gpio_set_direction()`：单独设置引脚方向

- **函数原型**

  ```c
  esp_err_t gpio_set_direction(gpio_num_t gpio_num, gpio_mode_t mode);
  ```

- **作用**：单独修改某个 GPIO 的模式（输入 / 输出 / 输入输出），覆盖当前模式。

- **参数**

  - `gpio_num`：目标 GPIO 编号。
  - `mode`：目标模式（如 `GPIO_MODE_INPUT`）。

- **示例**：将已配置的 GPIO2 改为输入模式

  ```c
  gpio_set_direction(GPIO_NUM_2, GPIO_MODE_INPUT);
  ```

#### 配置上下拉函数（`gpio_pullup_en`/`gpio_pulldown_en`）

- **函数原型**

  ```c
  esp_err_t gpio_pullup_en(gpio_num_t gpio_num);    // 启用上拉
  esp_err_t gpio_pullup_dis(gpio_num_t gpio_num);   // 禁用上拉
  esp_err_t gpio_pulldown_en(gpio_num_t gpio_num);  // 启用下拉
  esp_err_t gpio_pulldown_dis(gpio_num_t gpio_num); // 禁用下拉
  ```

- **作用**：比 `gpio_set_pull_mode` 更直观，单独控制上拉 / 下拉的启用 / 禁用。

- **示例**：启用 GPIO0 下拉，禁用上拉：

  ```c
  gpio_pulldown_en(GPIO_NUM_0);
  gpio_pullup_dis(GPIO_NUM_0);
  ```

### 引脚电平控制函数

#### `gpio_set_level()`：设置输出电平

- **函数原型**

  ```c
  esp_err_t gpio_set_level(gpio_num_t gpio_num, uint32_t level);
  ```

- **作用**：给输出模式的 GPIO 设置高 / 低电平（仅输出模式有效）。

- **参数**：

  - `level`：`0`（低电平）、`1`（高电平）。

#### `gpio_get_level()`：读取输入电平

- **函数原型**

  ```c
  int gpio_get_level(gpio_num_t gpio_num);
  ```

- **作用**：读取 GPIO 的当前输入电平（需引脚为输入 / 输入输出模式，否则返回 0）。

- **返回值**：`0`（低电平）、`1`（高电平）。

## LEDC库学习笔记

> **定义**：LEDC（Light Emitting Diode Controller）是 ESP32 内置的 LED 控制器，核心功能是生成 PWM（脉冲宽度调制）信号，不仅用于 LED 亮度调节，还可驱动电机、舵机等需 PWM 信号的设备。
>
> **核心优势**：无需手动编写定时器中断逻辑，ESP-IDF 提供完整库接口，配置简单、信号稳定
>
> **注意**：LEDC 控制器有自己`专属的 4 个定时器`（`LEDC_TIMER_0` ~ `LEDC_TIMER_3`），这些定时器是 LEDC 模块内部集成的，专门用于生成 PWM 信号。
>
> 与`通用定时器（TIMER_0 ~ TIMER_3）`完全分离

### LEDC关键概念

| 概念                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| 通道（Channel）           | ESP32 有 16 个独立通道，每个通道可独立配置 PWM 参数（频率、占空比），并映射到 1 个 GPIO 引脚输出信号。 |
| 定时器（Timer）           | 每个通道需绑定定时器，定时器决定 PWM 频率（由预分频和计数范围计算），多个通道可共享同一定时器（频率相同）。 |
| 频率（Frequency）         | PWM 信号周期的倒数（单位：Hz），如 50Hz（舵机常用，对应 20ms 周期）、1000Hz（LED 常用，避免人眼闪烁）。 |
| 占空比（Duty Cycle）      | PWM 高电平持续时间与周期的比值（0~100%），LEDC 中用数值表示（如 12 位分辨率对应 0~4095）。 |
| 分辨率（Duty Resolution） | 占空比的可调精度，范围 `LEDC_TIMER_1_BIT`~`LEDC_TIMER_20_BIT`，如 12 位表示占空比可设为 0~4095（共 4096 级）。 |
| 速度模式                  | 分 `LEDC_LOW_SPEED_MODE`（低速模式）和 `LEDC_HIGH_SPEED_MODE`（高速模式），高速模式支持更高 PWM 频率。 |

### 常用API（需包含头文件 `driver/ledc.h`）

#### 定时器初始化

- **作用**：初始化 LEDC 定时器，设置频率、分辨率等核心参数。

- 函数原型

  ```c
  esp_err_t ledc_timer_config(const ledc_timer_config_t *timer_conf);
  ```

- **参数结构体 `ledc_timer_config_t` 说明**：

  | 参数              | 取值 / 说明                                                  |
  | ----------------- | ------------------------------------------------------------ |
  | `speed_mode`      | 速度模式：`LEDC_LOW_SPEED_MODE` 或 `LEDC_HIGH_SPEED_MODE`    |
  | `duty_resolution` | 占空比分辨率：`LEDC_TIMER_1_BIT` ~ `LEDC_TIMER_20_BIT`（如 12 位选 `LEDC_TIMER_12_BIT`） |
  | `timer_num`       | 定时器编号：`LEDC_TIMER_0` ~ `LEDC_TIMER_3`（共 4 个定时器） |
  | `freq_hz`         | PWM 频率（如 50、1000，单位：Hz）                            |
  | `clk_cfg`         | 时钟源：通常用 `LEDC_AUTO_CLK`（自动选择最优时钟）           |

- **示例**：配置 1000Hz、12 位分辨率的定时器 0（低速模式）

  ```c
  ledc_timer_config_t timer_conf = {
      .speed_mode = LEDC_LOW_SPEED_MODE,
      .duty_resolution = LEDC_TIMER_12_BIT,  // 0~4095 级占空比
      .timer_num = LEDC_TIMER_0,
      .freq_hz = 1000,                       // 1000Hz 频率
      .clk_cfg = LEDC_AUTO_CLK
  };
  ledc_timer_config(&timer_conf);  // 执行配置
  ```

####  通道配置：`ledc_channel_config()`

#### 通道配置：`ledc_channel_config()`

- **作用**：将 LEDC 通道绑定到定时器和 GPIO 引脚，设置初始占空比。

- **函数原型**

  ```c
  esp_err_t ledc_channel_config(const ledc_channel_config_t *channel_conf);
  ```

- **参数结构体** `ledc_channel_config_t` **说明**

  | 参数         | 取值 / 说明                                                  |
  | ------------ | ------------------------------------------------------------ |
  | `speed_mode` | 需与绑定的定时器速度模式一致（如 `LEDC_LOW_SPEED_MODE`）     |
  | `channel`    | 通道编号：`LEDC_CHANNEL_0` ~ `LEDC_CHANNEL_15`（共 16 个通道） |
  | `timer_sel`  | 绑定的定时器编号（如 `LEDC_TIMER_0`）                        |
  | `intr_type`  | 中断类型：通常设 `LEDC_INTR_DISABLE`（禁用中断）             |
  | `gpio_num`   | 输出 PWM 的 GPIO 引脚（如 `GPIO_NUM_2`、`GPIO_NUM_4`）       |
  | `duty`       | 初始占空比（0 ~ 分辨率最大值，如 12 位初始为 0 表示 LED 灭） |
  | `hpoint`     | 脉冲起点偏移：通常设 0（无偏移）                             |

- 示例

  ```c
  ledc_channel_config_t channel_conf = {
      .speed_mode = LEDC_LOW_SPEED_MODE,
      .channel = LEDC_CHANNEL_0,
      .timer_sel = LEDC_TIMER_0,  // 绑定定时器 0
      .intr_type = LEDC_INTR_DISABLE,
      .gpio_num = GPIO_NUM_2,     // GPIO2 输出 PWM
      .duty = 0,                  // 初始占空比 0（LED 不亮）
      .hpoint = 0
  };
  ledc_channel_config(&channel_conf);  // 执行配置
  ```

#### 3. 占空比修改：`ledc_set_duty()` + `ledc_update_duty()`

- **作用**：动态调整通道的 PWM 占空比（`ledc_set_duty` 仅修改寄存器，`ledc_update_duty` 使修改生效）。

- **函数原型**

  ```c
  // 1. 设置占空比（不立即生效）
  esp_err_t ledc_set_duty(ledc_mode_t speed_mode, ledc_channel_t channel, uint32_t duty);
  // 2. 使占空比生效
  esp_err_t ledc_update_duty(ledc_mode_t speed_mode, ledc_channel_t channel);
  ```

- **参数说明**

  - `speed_mode`：速度模式（与通道一致）
  - `channel`：目标通道编号
  - `duty`：目标占空比（0 ~ 分辨率最大值）

- **示例**：将通道 0 占空比设为 50%（12 位分辨率对应 2048）

  ```c
  // 12 位分辨率下，50% 占空比 = 4096 / 2 = 2048
  ledc_set_duty(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, 2048);
  ledc_update_duty(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0);  // 占空比生效
  ```

#### 4. 硬件渐变：`ledc_fade_*` 系列函数

- **作用**：实现硬件自动渐变（无需软件循环），适合 LED 呼吸灯、平滑调光等场景。

- **核心函数**：

  | 函数                        | 作用                                                    |
  | --------------------------- | ------------------------------------------------------- |
  | `ledc_fade_func_install()`  | 初始化渐变功能（需开启中断，参数 0 表示不使用中断回调） |
  | `ledc_set_fade_with_time()` | 配置渐变参数（目标占空比、渐变时间）                    |
  | `ledc_fade_start()`         | 启动渐变（可选择阻塞 / 非阻塞模式）                     |
  | `ledc_cb_register()`        | 注册 LEDC 回调函数                                      |

- **函数原型**

  ```c
  // 1. 初始化渐变功能
  esp_err_t ledc_fade_func_install(int intr_alloc_flags);
  // 2. 配置渐变：在指定时间内从当前占空比渐变到目标占空比
  esp_err_t ledc_set_fade_with_time(ledc_mode_t speed_mode, ledc_channel_t channel, uint32_t target_duty, uint32_t fade_time_ms);
  // 3. 启动渐变
  esp_err_t ledc_fade_start(ledc_mode_t speed_mode, ledc_channel_t channel, ledc_fade_mode_t fade_mode);
  ```

- **参数说明**：

  - `intr_alloc_flags`：中断分配标志（设 0 即可）
  - `target_duty`：渐变目标占空比
  - `fade_time_ms`：渐变总时间（单位：ms）
  - `fade_mode`：渐变模式：`LEDC_FADE_WAIT_DONE`（阻塞，等待渐变完成返回）/ `LEDC_FADE_NO_WAIT`（非阻塞）

- **示例**：2 秒内从当前占空比渐变到最大亮度（12 位对应 4095）

  ```c
  // 1. 初始化渐变功能
  ledc_fade_func_install(0);  // 不使用中断回调
  
  // 2. 配置渐变：2000ms（2秒）内渐变到 4095（最大亮度）
  ledc_set_fade_with_time(LEDC_LOW_SPEED_MODE, 
                          LEDC_CHANNEL_0, 
                          4095,        // 目标占空比（最大）
                          2000);       // 渐变时间（ms）
  
  // 3. 启动渐变（阻塞模式，等待渐变完成后再执行后续代码）
  ledc_fade_start(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, LEDC_FADE_WAIT_DONE);
  ```

##### 启用中断模式

- **函数原型**

  ```C
  esp_err_t ledc_cb_register(ledc_mode_t speed_mode, ledc_channel_t channel, ledc_cbs_t *cbs, void *user_arg)
  ```

- **参数结构体** `ledc_cbs_t` **说明**

  - `fade_cb`：需要注册的中断回调函数

- **示例**

  ```C
  #include <stdio.h>
  #include "freertos/FreeRTOS.h"
  #include "freertos/task.h"
  #include "driver/gpio.h"
  #include "driver/ledc.h"
  
  #define LED_GPIO GPIO_NUM_18
  
  #define FULL_EV_BIT BIT0
  #define EMPTY_EV_BIT BIT1
  
  static EventGroupHandle_t led_fade_handle;
  
  /**
   * @brief 重写LEDC完成回调函数
   * @note IRAM_ATTR 修饰符表示该函数在IRAM中执行，中断服务程序中不能调用RAM中的函数
   */
  bool IRAM_ATTR ledc_finish_cb(const ledc_cb_param_t *param, void *user_arg)
  {
      BaseType_t taskWoken;
      if (param->duty)
      {
          xEventGroupSetBitsFromISR(led_fade_handle, FULL_EV_BIT, &taskWoken);
      }
      else
      {
          xEventGroupSetBitsFromISR(led_fade_handle, EMPTY_EV_BIT, &taskWoken);
      }
      return taskWoken;
  }
  
  void led_run_task(void *param)
  {
      EventBits_t ev;
      while (1)
      {
          ev = xEventGroupWaitBits(led_fade_handle, FULL_EV_BIT | EMPTY_EV_BIT, pdTRUE, pdFALSE, pdMS_TO_TICKS(5000));
          if (ev & FULL_EV_BIT)
          {
              ledc_set_fade_with_time(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, 0, 2000);
              ledc_fade_start(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, LEDC_FADE_NO_WAIT);
          }
          if (ev & EMPTY_EV_BIT)
          {
              ledc_set_fade_with_time(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, 8191, 2000);
              ledc_fade_start(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, LEDC_FADE_NO_WAIT);
          }
  
          ledc_cbs_t ledc_cbs = {
              .fade_cb = ledc_finish_cb,
          };
  
          ledc_cb_register(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, &ledc_cbs, NULL);
      }
  }
  
  void app_main(void)
  {
      // gpio_config_t led_cfg = {
      //     .pin_bit_mask = (1 << LED_GPIO),
      //     .pull_down_en = GPIO_PULLDOWN_DISABLE,
      //     .pull_up_en = GPIO_PULLUP_DISABLE,
      //     .intr_type = GPIO_INTR_DISABLE,
      //     .mode = GPIO_MODE_OUTPUT,
      // };
      // gpio_config(&led_cfg);
  
      ledc_timer_config_t ledc_timer = {
          .speed_mode = LEDC_LOW_SPEED_MODE,
          .timer_num = LEDC_TIMER_0,
          .clk_cfg = LEDC_AUTO_CLK,
          .freq_hz = 5000,
          .duty_resolution = LEDC_TIMER_13_BIT, // 占空比分辨率 0-8191
      };
      ledc_timer_config(&ledc_timer);
  
      ledc_channel_config_t ledc_channel = {
          .gpio_num = LED_GPIO,
          .channel = LEDC_CHANNEL_0,
          .duty = 0,
          .speed_mode = LEDC_LOW_SPEED_MODE,
          .timer_sel = LEDC_TIMER_0,
          .intr_type = LEDC_INTR_DISABLE,
      };
      ledc_channel_config(&ledc_channel);
  
      ledc_fade_func_install(0); // 使能渐变 不需要自己手动调节占空比
      ledc_set_fade_with_time(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, 8191, 2000);
      ledc_fade_start(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, LEDC_FADE_NO_WAIT);
  
      led_fade_handle = xEventGroupCreate();
  
      ledc_cbs_t ledc_cbs = {
          .fade_cb = ledc_finish_cb,
      };
  
      ledc_cb_register(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_0, &ledc_cbs, NULL);
  
      xTaskCreatePinnedToCore(led_run_task, "led_run_task", 2048, NULL, 3, NULL, 1);
  }
  ```


## UART配置

### ESP32 UART 硬件特性

- **硬件资源**：ESP32 内置 3 个 UART 接口（UART0、UART1、UART2），支持不同引脚映射
- **通信参数**：可配置波特率（1200 ~ 5000000bps）、数据位（5 ~ 8 位）、校验位（无 / 奇 / 偶）、停止位（1/1.5/2 位）
- **缓冲区**：每个 UART 支持硬件 FIFO 缓冲区，可通过软件配置环形缓冲区大小
- **事件驱动**：支持通过 FreeRTOS 队列触发 UART 事件（数据接收、缓冲区满、溢出等）

### 关键API

| API 函数                | 功能描述                               | 核心参数说明                                                 |
| ----------------------- | -------------------------------------- | ------------------------------------------------------------ |
| `uart_param_config()`   | 配置 UART 通信参数（波特率、数据位等） | 端口号、`uart_config_t` 结构体                               |
| `uart_set_pin()`        | 绑定 UART 信号到物理引脚               | 端口号、TX 引脚、RX 引脚、RTS 引脚、CTS 引脚                 |
| `uart_driver_install()` | 安装 UART 驱动并创建事件队列           | 端口号、接收缓冲区大小、发送缓冲区大小、队列容量、队列句柄、标志 |
| `uart_read_bytes()`     | 读取 UART 接收数据                     | 端口号、接收缓冲区、读取长度、超时时间                       |
| `uart_write_bytes()`    | 发送 UART 数据                         | 端口号、发送缓冲区、发送长度                                 |
| `uart_flush_input()`    | 清空 UART 接收缓冲区                   | 端口号                                                       |
| `xQueueReceive()`       | 从事件队列接收 UART 事件               | 队列句柄、事件存储变量地址、超时时间                         |
| `xTaskCreate()`         | 创建独立任务处理 UART 事件             | 任务函数、任务名称、栈大小、优先级等                         |

### 基础配置

#### 步骤 1：定义核心参数（宏定义）

```c
// 1. 定义 UART 端口号
#define USER_UART_NUM UART_NUM_1  // 避免 UART0 冲突（默认用于串口打印）
#define USER_UART_BAUD 115200  // 串口波特率设置

// 2. 定义引脚（需根据硬件接线调整）
#define UART_TX_PIN 17  // TX 发送引脚
#define UART_RX_PIN 16  // RX 接收引脚

// 3. 定义缓冲区和队列参数
#define UART_BUFFER_SIZE 1024    // 数据缓冲区大小
#define UART_QUEUE_SIZE 10       // 事件队列容量（最多缓存 10 个事件）
#define UART_TASK_STACK_SIZE 4096// UART 处理任务栈大小
#define UART_TASK_PRIORITY 5     // UART 任务优先级（1~24，数值越大优先级越高）

// 4. 全局变量声明
static QueueHandle_t uart_event_queue;  // UART 事件队列句柄
static uint8_t uart_rx_buffer[UART_BUFFER_SIZE];  // 接收数据缓冲区
static const char *TAG = "UART_DEV";    // 日志标签
```

#### 步骤 2：配置 UART 硬件参数

通过 `uart_config_t` 结构体配置通信参数，再调用 `uart_param_config()` 生效：

```c
// 配置 UART 通信参数
uart_config_t uart_config = {
    .baud_rate = USER_UART_BAUD,          // 波特率（常用：9600、115200、1000000）
    .data_bits = UART_DATA_8_BITS,// 数据位：8 位
    .parity = UART_PARITY_DISABLE,// 校验位：禁用
    .stop_bits = UART_STOP_BITS_1,// 停止位：1 位
    .flow_ctrl = UART_HW_FLOWCTRL_DISABLE,  // 硬件流控：禁用（无需 RTS/CTS 引脚）
    .source_clk = UART_SCLK_DEFAULT,         // 时钟源：默认
};
// 应用配置到指定 UART 端口
ESP_ERROR_CHECK(uart_param_config(USER_UART_NUM, &uart_config));
```

#### 步骤 3：绑定引脚并安装驱动 （如果用默认串口的话 则不需要绑定引脚）

- **引脚绑定**：通过 `uart_set_pin()` 将 UART 信号映射到 ESP32 物理引脚
- **驱动安装**：调用 `uart_driver_install()` 初始化 UART 驱动，创建事件队列（可选）

```c
// 1. 绑定 UART 引脚（TX/RX，流控引脚填 UART_PIN_NO_CHANGE 表示禁用）
ESP_ERROR_CHECK(uart_set_pin(USER_UART_NUM, 
                            UART_TX_PIN,  // TX 引脚
                            UART_RX_PIN,  // RX 引脚
                            UART_PIN_NO_CHANGE,  // RTS 引脚（禁用）
                            UART_PIN_NO_CHANGE)); // CTS 引脚（禁用）

// 2. 安装 UART 驱动（参数依次：端口号、接收缓冲区大小、发送缓冲区大小、事件队列容量、队列句柄、标志）
ESP_ERROR_CHECK(uart_driver_install(USER_UART_NUM, 
                                    UART_BUFFER_SIZE * 2,  // 接收缓冲区（建议比单次接收大）
                                    UART_BUFFER_SIZE * 2,  // 发送缓冲区
                                    UART_QUEUE_SIZE,       // 事件队列容量
                                    &uart_event_queue,     // 事件队列句柄（传出）
                                    0));                   // 标志：0（默认）
```

### ESP32 UART 数据处理方式

| 特性       | 轮询方式（简单场景）                      | 事件驱动方式（高效场景）          |
| ---------- | ----------------------------------------- | --------------------------------- |
| 核心函数   | `uart_read_bytes()`、`uart_write_bytes()` | `xQueueReceive()`（监听事件队列） |
| 适用场景   | 简单回显、低频率数据交互                  | 高频率数据、多事件处理（如溢出）  |
| CPU 占用率 | 较高（循环查询）                          | 较低（事件触发，无轮询）          |
| 代码复杂度 | 低                                        | 中（需处理多事件类型）            |
| 看门狗风险 | 高（需避免长阻塞）                        | 低（事件队列超时释放 CPU）        |

### 实战代码示例

#### 事件驱动方式

- 通过 FreeRTOS 队列接收 UART 事件，按需处理数据接收、缓冲区满、溢出等场景：

  ```C
  #include <stdio.h>
  #include "freertos/FreeRTOS.h"
  #include "freertos/task.h"
  #include "driver/uart.h"
  #include "esp_log.h"
  #include "driver/gpio.h"
  
  // -------------- 1. 宏定义参数 --------------
  #define USER_UART_NUM UART_NUM_1
  #define USER_UART_BAUD 115200
  #define UART_TX_PIN 17
  #define UART_RX_PIN 16
  #define UART_BUFFER_SIZE 1024
  #define UART_QUEUE_SIZE 10
  #define UART_TASK_STACK_SIZE 4096
  #define UART_TASK_PRIORITY 5
  
  // -------------- 2. 全局变量 --------------
  static QueueHandle_t uart_event_queue;
  static uint8_t uart_rx_buffer[UART_BUFFER_SIZE];
  static const char *TAG = "UART_DEV";
  
  // -------------- 3. UART 事件处理任务 --------------
  void uart_event_task(void *arg) {
      uart_event_t event;  // 存储 UART 事件的结构体
  
      while (1) {
          // 从事件队列接收事件（超时 100ms，避免阻塞触发看门狗）
          if (xQueueReceive(uart_event_queue, &event, pdMS_TO_TICKS(100)) == pdTRUE) {
              switch (event.type) {
                  // 1. 接收数据事件
                  case UART_DATA:
                      ESP_LOGI(TAG, "收到数据，长度：%d 字节", event.size);
                      // 读取数据到缓冲区（超时 50ms，确保数据完整性）
                      int read_len = uart_read_bytes(USER_UART_NUM, 
                                                    uart_rx_buffer, 
                                                    event.size, 
                                                    pdMS_TO_TICKS(50));
                      if (read_len > 0) {
                          // 示例1：回显数据（发送收到的数据）
                          uart_write_bytes(USER_UART_NUM, uart_rx_buffer, read_len);
                          // 示例2：打印接收的数据（十六进制）
                          ESP_LOGI(TAG, "接收数据（hex）：");
                          for (int i = 0; i < read_len; i++) {
                              printf("%02X ", uart_rx_buffer[i]);
                          }
                          printf("\n");
                      }
                      break;
  
                  // 2. 接收缓冲区满事件
                  case UART_BUFFER_FULL:
                      ESP_LOGI(TAG, "接收缓冲区满，清空缓冲区");
                      uart_flush_input(USER_UART_NUM);  // 清空接收缓冲区
                      xQueueReset(uart_event_queue);    // 重置事件队列
                      break;
  
                  // 3. FIFO 溢出事件（硬件缓冲区溢出）
                  case UART_FIFO_OVF:
                      ESP_LOGI(TAG, "UART FIFO 溢出，清空缓冲区");
                      uart_flush_input(USER_UART_NUM);
                      xQueueReset(uart_event_queue);
                      break;
  
                  // 4. 帧错误事件（数据格式错误）
                  case UART_FRAME_ERR:
                      ESP_LOGI(TAG, "帧错误（数据格式异常）");
                      break;
  
                  // 5. 校验错误事件（启用校验时生效）
                  case UART_PARITY_ERR:
                      ESP_LOGI(TAG, "校验错误（数据校验不匹配）");
                      break;
  
                  // 其他未处理事件
                  default:
                      ESP_LOGI(TAG, "未处理事件类型：%d", event.type);
                      break;
              }
          }
      }
  }
  
  // -------------- 4. 主函数初始化 --------------
  void app_main(void) {
      // 1. 配置 UART 硬件参数
      uart_config_t uart_config = {
          .baud_rate = USER_UART_BAUD,
          .data_bits = UART_DATA_8_BITS,
          .parity = UART_PARITY_DISABLE,
          .stop_bits = UART_STOP_BITS_1,
          .flow_ctrl = UART_HW_FLOWCTRL_DISABLE,
          .source_clk = UART_SCLK_DEFAULT,
      };
      ESP_ERROR_CHECK(uart_param_config(USER_UART_NUM, &uart_config));
  
      // 2. 绑定引脚
      ESP_ERROR_CHECK(uart_set_pin(USER_UART_NUM, 
                                  UART_TX_PIN, 
                                  UART_RX_PIN, 
                                  UART_PIN_NO_CHANGE, 
                                  UART_PIN_NO_CHANGE));
  
      // 3. 安装 UART 驱动并创建事件队列
      ESP_ERROR_CHECK(uart_driver_install(USER_UART_NUM, 
                                          UART_BUFFER_SIZE * 2, 
                                          UART_BUFFER_SIZE * 2, 
                                          UART_QUEUE_SIZE, 
                                          &uart_event_queue, 
                                          0));
  
      // 4. 创建 UART 事件处理任务（独立任务，避免阻塞 main 任务）
      xTaskCreate(uart_event_task,          // 任务函数
                  "uart_event_task",        // 任务名称
                  UART_TASK_STACK_SIZE,     // 栈大小
                  NULL,                     // 任务参数
                  UART_TASK_PRIORITY,       // 优先级
                  NULL);                    // 任务句柄（无需则填 NULL）
  }
  ```

#### 轮询方式（简单场景，如回显）

- 适用于数据量小、交互频率低的场景，代码更简洁：

  ```C
  void app_main(void) {
      // 1. 配置 UART 参数（同事件驱动方式）
      uart_config_t uart_config = {
          .baud_rate = 115200,
          .data_bits = UART_DATA_8_BITS,
          .parity = UART_PARITY_DISABLE,
          .stop_bits = UART_STOP_BITS_1,
          .flow_ctrl = UART_HW_FLOWCTRL_DISABLE,
      };
      ESP_ERROR_CHECK(uart_param_config(USER_UART_NUM, &uart_config));
      ESP_ERROR_CHECK(uart_set_pin(USER_UART_NUM, 17, 16, UART_PIN_NO_CHANGE, UART_PIN_NO_CHANGE));
      ESP_ERROR_CHECK(uart_driver_install(USER_UART_NUM, 1024, 1024, 0, NULL, 0));
  
      // 2. 轮询读取数据并回显
      while (1) {
          // 读取数据（超时 50ms，无数据则返回 0）
          int read_len = uart_read_bytes(USER_UART_NUM, uart_rx_buffer, 1024, pdMS_TO_TICKS(50));
          if (read_len > 0) {
              // 回显数据
              uart_write_bytes(USER_UART_NUM, uart_rx_buffer, read_len);
              ESP_LOGI(TAG, "轮询接收数据长度：%d", read_len);
          }
          // 短暂延迟，释放 CPU（避免触发看门狗）
          vTaskDelay(pdMS_TO_TICKS(10));
      }
  }
  ```

### 常见问题与解决方案

#### 触发 Task Watchdog（任务看门狗）

- **原因**：
  1. 轮询方式中 `uart_read_bytes()` 超时过长，或无 `vTaskDelay()` 释放 CPU；
  2. 事件驱动方式中 `xQueueReceive()` 使用 `portMAX_DELAY` 永久阻塞。
- **解决方案**：
  - 轮询方式：在循环中添加 `vTaskDelay(pdMS_TO_TICKS(10))`；
  - 事件驱动方式：`xQueueReceive()` 超时设为有限值（如 `pdMS_TO_TICKS(100)`）。

#### 数据接收不完整或丢失

- **原因**：
  1. 接收缓冲区过小，无法容纳单次接收的数据；
  2. `uart_read_bytes()` 超时过短，数据未完全接收；
  3. 波特率不匹配（发送端与接收端波特率必须一致）。
- **解决方案**：
  - 增大接收缓冲区（如 `UART_BUFFER_SIZE * 2`）；
  - 延长 `uart_read_bytes()` 超时时间（如 50~100ms）；
  - 确保发送端与 ESP32 UART 波特率、数据位、校验位完全一致。

#### UART0 与串口打印冲突

- **原因**：ESP32 默认将 `printf()` 映射到 UART0（引脚 TX=1，RX=3），若使用 UART0 通信会导致打印乱码。
- **解决方案**：
  - 优先使用 UART1 或 UART2 作为通信端口；
  - 若必须使用 UART0，需重新映射 `printf()` 到其他 UART（需修改 ESP-IDF 配置）。

## WIFI

### ESP32下WIFI组件三种模式

- **AP 模式**：ESP32 自身作为接入点，其他设备（如手机、电脑）可连接 ESP32 创建的 WiFi 热点，适用于 “本地设备直连 ESP32” 场景（如配置 ESP32 参数）；

- **AP+STA 混合模式**：ESP32 同时作为接入点和客户端，既允许其他设备连接自身，又能接入外部路由器，适用于 “本地控制 + 远程联网” 复合场景（如智能家居设备）；

- **STA 模式**：仅作为客户端，专注于接入外部网络，是 ESP32 实现联网功能的最常用模式（如传感器数据上传云端、远程控制指令接收）。

### STA模式核心工作原理

ESP32 的 STA 模式连接 WiFi 的过程遵循 IEEE 802.11 协议，核心流程可分为 4 个阶段，各阶段依赖 ESP-IDF 提供的 WiFi 驱动接口实现：

1. **初始化阶段**：

   - 初始化 WiFi 组件（调用esp_wifi_init()），配置 WiFi 工作模式为 STA（通过wifi_init_config_t结构体设置mode = WIFI_MODE_STA）；

   - 配置 TCP/IP 协议栈（ESP-IDF 默认自动初始化，无需手动操作，支持 DHCP 自动获取 IP 或静态 IP 配置）。

2. **配置连接参数阶段**：

   - 定义wifi_config_t结构体，填入目标 AP 的**SSID（WiFi 名称）** 和**Password（WiFi 密码）**，确保参数与 AP 一致（区分 SSID 大小写，密码长度需符合 AP 要求，如 WPA2-PSK 密码通常为 8-63 位）；

   - 通过esp_wifi_set_config(WIFI_IF_STA, &wifi_config)将配置参数写入 ESP32 的 WiFi 接口。

3. **启动与连接阶段**：

   - 启动 STA 接口（esp_wifi_start()），ESP32 开始扫描周围 WiFi 信号；

   - 自动匹配目标 AP 的 SSID，发起连接请求，与 AP 进行**认证（如 WPA2-PSK 认证）** 和**关联**；

   - 若认证通过，ESP32 通过 DHCP 向 AP 请求 IP 地址（默认开启 DHCP 客户端），获取到 IP、子网掩码、网关后，完成 “物理连接→网络层连接” 的全流程。

4. **状态监测阶段**：

   - ESP32 通过**事件回调机制**（esp_event_loop_create_default()创建事件循环，esp_event_handler_instance_register()注册事件处理函数）反馈连接状态；

   - 关键事件包括：WIFI_EVENT_STA_START（STA 接口启动成功）、WIFI_EVENT_STA_CONNECTED（与 AP 物理连接成功）、IP_EVENT_STA_GOT_IP（获取 IP 地址，联网完成）、WIFI_EVENT_STA_DISCONNECTED（与 AP 断开连接）。

#### 实现步骤

##### 重写事件处理函数（核心）

```C
/**
 * @brief WiFi事件与IP事件处理回调函数（核心逻辑）
 * @param arg：用户自定义参数，本代码未使用，传入NULL即可
 * @param event_base：事件基础类型，用于区分是WiFi相关事件（WIFI_EVENT）还是IP相关事件（IP_EVENT）
 * @param event_id：具体事件ID，标识当前触发的是该事件类型下的哪一种具体事件
 * @param event_data：事件关联的数据指针，如IP事件中可获取分配的IP地址信息
 * 功能：响应ESP32 WiFi模块的关键事件，更新连接状态、触发重连逻辑、通知外部状态变化
 */
static void event_handler(void *arg, esp_event_base_t event_base,
                          int32_t event_id, void *event_data)
```

##### 实现 STA 模式初始化函数

```C
// 引入所需头文件：自定义WiFi管理模块头文件、ESP32 WiFi驱动库、事件处理库
// 日志打印库、FreeRTOS实时操作系统任务调度库、标准输入输出库
#include "wifi_manager.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include <stdio.h>

#define WIFI_SSID "2078"
#define WIFI_PASSWORD "12345678"

// 静态全局变量：WiFi状态回调函数指针，用于将WiFi连接/断开状态通知给外部调用者
// 外部需通过wifi_manager_init函数传入自定义回调，接收WIFI_STATE_CONNECTED/WIFI_STATE_DISCONNECTED状态
static p_wifi_state_cb wifi_state_cb = NULL;

// 宏定义：STA模式下最大重连次数，限制重连次数避免无限循环占用CPU资源
#define MAX_CONNECT_RETRY 10
// 宏定义：日志标签，串口打印时用于区分该模块的日志，方便调试定位
#define TAG "wifi_manager"

// 静态全局变量：记录STA模式当前的重连尝试次数，初始值为0，每次重连失败自增1
static int sta_connect_cnt = 0; // 重连次数

// 静态全局变量：标记STA模式是否已成功连接到目标AP，true表示已连接，false表示未连接/已断开
static bool is_sta_connected = false;

/**
 * @brief WiFi事件与IP事件处理回调函数（核心逻辑）
 * @param arg：用户自定义参数，本代码未使用，传入NULL即可
 * @param event_base：事件基础类型，用于区分是WiFi相关事件（WIFI_EVENT）还是IP相关事件（IP_EVENT）
 * @param event_id：具体事件ID，标识当前触发的是该事件类型下的哪一种具体事件
 * @param event_data：事件关联的数据指针，如IP事件中可获取分配的IP地址信息
 * 功能：响应ESP32 WiFi模块的关键事件，更新连接状态、触发重连逻辑、通知外部状态变化
 */
static void event_handler(void *arg, esp_event_base_t event_base,
                          int32_t event_id, void *event_data)
{
    // 判断事件类型为WiFi相关事件（WIFI_EVENT）
    if (event_base == WIFI_EVENT)
    {
        // 根据具体事件ID处理不同场景
        switch (event_id)
        {
        // 事件：STA接口启动成功（调用esp_wifi_start()后触发此事件）
        case WIFI_EVENT_STA_START:
            // 启动成功后，主动调用esp_wifi_connect()发起WiFi连接请求
            esp_wifi_connect();
            break;

        // 事件：STA与已连接的AP断开连接（如信号弱、密码错误、路由器重启等）
        case WIFI_EVENT_STA_DISCONNECTED:
            // 若之前处于已连接状态，先更新连接状态并通知外部
            if (is_sta_connected)
            {
                is_sta_connected = false; // 将连接状态标记为未连接
                if (wifi_state_cb)        // 若外部注册了回调函数，触发"断开连接"状态通知
                    wifi_state_cb(WIFI_STATE_DISCONNECTED);
            }

            // 检查当前重连次数是否小于最大重连次数，若未达到则继续重试
            if (sta_connect_cnt < MAX_CONNECT_RETRY)
            {
                sta_connect_cnt++;  // 重连次数自增1
                esp_wifi_connect(); // 发起新一轮连接请求
            }
            // 若已达到最大重连次数，不再重试（避免无限重连）
            wifi_state_cb(WIFI_STATE_DISCONNECTED);
            break;

        // 其他未定义的WiFi事件，暂不处理，直接break
        default:
            break;
        }
    }
    // 判断事件类型为IP相关事件（IP_EVENT）
    else if (event_base == IP_EVENT)
    {
        // 事件：STA成功从DHCP服务器获取到IP地址（标志网络层连接完成）
        if (event_id == IP_EVENT_STA_GOT_IP)
        {
            // 打印日志提示已获取IP地址，便于调试确认
            ESP_LOGI(TAG, "Got IP address");
            sta_connect_cnt = 0;     // 重置重连次数
            is_sta_connected = true; // 将连接状态标记为已连接
            if (wifi_state_cb)       // 若外部注册了回调函数，触发"连接成功"状态通知
                wifi_state_cb(WIFI_STATE_CONNECTED);
        }
    }
}

/**
 * @brief WiFi管理器初始化函数（STA模式初始化入口）
 * @param f：外部传入的WiFi状态回调函数，用于接收连接/断开状态通知
 * 功能：初始化ESP32网络接口、事件循环、WiFi驱动，配置STA模式并启动WiFi模块
 * 注意：使用WiFi功能前必须先调用此函数完成初始化，建议在app_main()中优先执行
 */
void wifi_manager_init(p_wifi_state_cb f)
{
    // 1. 初始化NVS（ESP32存储WiFi配置的依赖组件，必须先初始化）
    esp_err_t ret = nvs_flash_init();
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND)
    {
        ESP_ERROR_CHECK(nvs_flash_erase()); // 若NVS有问题，先擦除再初始化
        ret = nvs_flash_init();
    }

    // 2. 初始化ESP32网络接口（TCP/IP协议栈初始化，必须在WiFi初始化前执行）
    ESP_ERROR_CHECK(esp_netif_init());

    // 3. 创建默认的事件循环（用于处理WiFi和IP事件）
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    esp_netif_create_default_wifi_sta(); // 使用默认的STA模式网络接口

    // 4. 初始化WiFi驱动：使用WIFI_INIT_CONFIG_DEFAULT()获取默认配置（包含时钟、中断等参数）
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg)); // 执行WiFi驱动初始化，若失败则触发错误检查

    // 5. 注册事件处理函数：监听WiFi事件和IP事件
    // 5.1注册WiFi事件监听器：监听所有WiFi事件（ESP_EVENT_ANY_ID），触发时调用event_handler处理
    ESP_ERROR_CHECK(esp_event_handler_instance_register(WIFI_EVENT,
                                                        ESP_EVENT_ANY_ID,
                                                        &event_handler,
                                                        NULL,
                                                        NULL));
    // 5.2注册IP事件监听器：仅监听"STA获取IP"事件（IP_EVENT_STA_GOT_IP）
    ESP_ERROR_CHECK(esp_event_handler_instance_register(IP_EVENT,
                                                        IP_EVENT_STA_GOT_IP,
                                                        &event_handler,
                                                        NULL,
                                                        NULL));

    // 6. 保存外部传入的状态回调函数，后续状态变化时通过此函数通知外部
    wifi_state_cb = f;

    wifi_config_t wifi_config = {
        .sta = {
            // 配置STA的认证模式为WPA2-PSK（主流家用/工业路由器默认加密方式）
            .threshold.authmode = WIFI_AUTH_WPA2_PSK,
            .ssid = WIFI_SSID,
            .password = WIFI_PASSWORD,
        },
    };

    // 7. 配置WiFi工作模式为纯STA模式（仅作为客户端连接外部AP，不开启AP模式）
    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
    ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_STA, &wifi_config)); // 写入配置
    // 8. 启动WiFi模块（启动后会触发WIFI_EVENT_STA_START事件，进而发起连接）
    ESP_ERROR_CHECK(esp_wifi_start());
    ESP_LOGI(TAG, "STA initialization done");
}

```







