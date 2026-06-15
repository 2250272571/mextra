---
title: RT_Thread
date: 2025-05-21 20:26:03
cover: "https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505212030199.png"
tags:
  - 实时操作系统
  - RTOS
  - RT_Thread
layout: page
categories: MCU
---

## 时钟管理

### 时钟节拍产生与获取

时钟节拍依赖于硬件中的定时器。你可以把定时器看作是⼀个不停计时的装置，它每隔⼀段时间就会提醒系统“滴答”⼀次。<span style="text-decoration:underline;">每当定时器中断发⽣时，RT-Thread 会调⽤ rt_tick_increase()函数，系统时间增加 1 个节拍</span>。`就像家中的闹钟响⼀次，时间就过去了⼀分钟⼀样。`

> 其中rt_tick_increase()函数中有rt_tick类似于HAL库中的uwTick，可以直接调用 获取系统时间

## 定时器管理

### 硬件定时器 vs 软件定时器

1. <span style="color:#FF3399;">硬件定时器</span>：就像厨房⾥的倒计时器，它是由硬件直接提供的。硬件定时器精度⾮常⾼，可以达到纳秒级，适合⼀些对时间要求⾮常严格的任务，⽐如控制电机的旋转时间。
2. <span style="color:#FF3399;">软件定时器</span>：可以理解为⼿机上的计时 App。它依赖于操作系统的时钟节拍来⼯作，它的时间间隔精度由 OS Tick 决定。例如，如果 OS Tick 是 10ms，那么软件定时器只能精确到 10ms。虽然精度不如硬件定时器⾼，但它更灵活、容易使⽤。

### RTT定时器分类

- <span style="color:#FF3399;">单次触发定时器</span>：就像倒计时⼀次的闹钟，响⼀次后就停⽌。例如，设定⼀个 5 秒的倒计时，时间到了闹钟响⼀次就结束了。
- <span style="color:#FF3399;">周期触发定时器</span>：就像那些每天早上 7 点重复响起的闹钟。每隔⼀段时间，它会周期性地触发事件，直到你⼿动关闭它。

根据定时器回调函数执⾏的上下⽂环境，定时器还可以分为两种模式：

- <span style="color:#FF3399;">硬件模式 </span>(HARD_TIMER)：定时器的回调函数在<span style="text-decoration:underline;">中断上下⽂</span>中执⾏`就像闹钟响起时你⽴刻跳起来⼀样`它反应⾮常快，但要求处理时间短。
- <span style="color:#FF3399;">软件模式</span> (SOFT_TIMER)：回调函数在<span style="text-decoration:underline;">系统的 timer 线程中</span>执⾏，相当于`有⼈提醒你“闹钟响了”`，然后你再去执⾏⼀些动作。这个模式可以做更复杂的操作，因为它不是在中断中执⾏。

### 控制定时器

`通过该函数你可以随时改变定时器触发方式以及定时器时间`

![image-20250521221429133](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505212214569.png)

### 高精度延时

`使用该函数可以实现微妙级的精确延时，适合高精度时间控制场合`

![image-20250521221525259](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505212215044.png)

## 线程创建

![image-20250521203425658](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505212034310.png)

- `name`：线程名字
- `entry`：线程入口函数
- `parameter`：传递给线程的参数
- `stack_size`：线程栈大小
- `priority`：线程优先级
- `tick`：时间片

### 详细步骤

1. 声明一个pcb结构体 `static rt_thread_t tid1 = RT_NULL;`
2. 编写线程入口函数 `static void thread1_entry(void *parameter)`，参数为必需，其为传递给线程的参数
3. 利用`rt_thread_create`函数赋值给所声明的pcb结构体tid1
4. 利用``rt_thread_startup(tid1)``函数启动线程

### 实现示例

```C
static void thread1_proc(void *parameter)
{
    while(1)
    {
        /*实际功能*/
        rt_thread_mdelay(1000);//休息1秒
    }
}
int thread1_init(void)
{
    rt_thread_t tid1 =  rt_thread_create("thread1_proc",thread1_proc,RT_NULL,1024,10,20);
    if(tid1 != RT_NULL)
    {
        rt_thread_startup(tid1);
    }
    return RT_EOK;
}
INIT_COMPONENT_EXPORT(thread1_init);   //用这个宏就不需要另外再去注册
```

> <span style="font-style:italic;">rt_thread_mdelay(1000)的工作机制：</span>
>
> <span style="font-style:italic;">1、当 rt_thread_mdelay 被调⽤时，当前线程会进⼊ RT_THREAD_SUSPEND 状态（<span style="color:#CC00CC;">挂起</span>状态）。</span>
>
> <span style="font-style:italic;">2、调度器开始运⾏其他优先级相同或更⾼的线程。</span>
>
> <span style="font-style:italic;">3、 <span style="text-decoration:underline;">延时时间结束后，线程从 挂起状态 转变为 就绪状态</span> ，等待系统再次调度该线程。</span>
>
> <span style="font-style:italic;">PS：若此时无其他线程，则CPU就处于释放状态</span>

![image-20250521205640868](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505212056766.png)

## 线程间同步

### 信号量

信号量可以理解为电影院的座位数（<span style="color:#FF0000;">资源数量</span>）。当所有座位都满了，新的观众（线程）就需要等前⾯的观众离开（释放资源）才能进⼊。当有空位时，观众可以进⼊。`信号量通过控制资源的数量来管理多个线程对资源的访问`

#### 工作机制

- `获取信号量`：当资源可⽤时，线程可以获取信号量，<span style="color:#FF0000;">信号量的值减1</span>。<span style="background:#66FFFF;">如果信号量的值为0，线程就会等待，直到资源释放</span>

- `释放信号量`：使⽤完资源后，线程释放信号量，<span style="color:#FF0000;">信号量的值加1</span>，允许其他线程访问资源

##### 创建信号量

```c
/*动态分配内存并创建信号量*/
rt_sem_t rt_sem_create(const char *name, rt_uint32_t value, rt_uint8_t flag);
/*静态分配内存并创建信号量*/
rt_err_t rt_sem_init(rt_sem_t sem, const char *name, rt_uint32_t value,rt_uint8_t flag);
```

- <span style="color:#3399FF;">参数说明</span>：

- `name`：信号量的名字（可以为 `RT_NULL` 表⽰匿名信号量）。

- `value`：信号量的初始值，表⽰信号量当前持有的资源数量（`例如初始值为 0 表⽰等待事件，为 1 表⽰互斥量，或者更⼤值表⽰资源计数`）。

- `flag`：PC 对象的属性标志，常⽤值：
- `RT_IPC_FLAG_PRIO` ：<span style="color:#FF0000;">优先级</span>等待⽅式，等待线程按照优先级顺序排列。
  
- `RT_IPC_FLAG_FIFO` ：<span style="color:#FF0000;">先⼊先出</span>等待⽅式，等待线程按照进⼊顺序排列。

##### 获取信号量

⽤于获取信号量（P 操作），线程会尝试获取信号量，`如果信号量的计数值为 0，线程会进⼊等待状态`，直到信号量被释放或超时。

```c
rt_err_t rt_sem_take(rt_sem_t sem, rt_int32_t time);
```

- <span style="color:#3399FF;">参数说明</span>：

- `sem`：信号量的句柄。

- `time`：超时时间（单位为系统 tick），表⽰最⼤等待时间。如果设置为 `RT_WAITING_FOREVER` ，线程会⼀直等待

##### 释放信号量

⽤于释放信号量（V 操作），**增加**信号量的计数值，`并唤醒等待该信号量的线程`。

```c
rt_err_t rt_sem_release(rt_sem_t sem);
```

#### 应用示例

```c
#include <rtthread.h>

rt_sem_t sem = RT_NULL;

void thread1(void *parameter)
{
   rt_kprintf("线程1：等待线程2发送信号\n");

   /*等待信号量，超时时间为永久等待*/
   rt_sem_take(sem,RT_WAITING_FOREVER);

   rt_kprintf("线程1：收到线程2的信号，继续执行....\n");
}

void thread2(void *parameter)
{
   rt_kprintf("线程2：执行一些操作\n");
   /*模拟执行一些动作*/
   rt_thread_mdelay(2000);
   rt_kprintf("线程2：操作完成，发送信号给线程1\n");
   /*释放信号量 通知线程1*/
   rt_sem_release(sem);
}

int main(void)
{
    rt_thread_t tid1 = RT_NULL;
    rt_thread_t tid2 = RT_NULL;

    sem = rt_sem_create("sem", 0, RT_IPC_FLAG_PRIO);
    tid1 = rt_thread_create("tid1", thread1, RT_NULL, 1024, 25, 100);
    tid2 = rt_thread_create("tid2", thread2, RT_NULL, 1024, 25, 100);

    if(tid1 != RT_NULL) rt_thread_startup(tid1);
    if(tid2 != RT_NULL) rt_thread_startup(tid2);
    return 0;
}
```

#### 使用场景

1. 任务同步（多任务协调执行顺序）
2. 任务互斥(使用`互斥量` 保证同一时间只有一个线程 访问公共资源)
3. 事件触发
4. 限量资源管理

### **互斥量（Mutex）**

#### **工作机制**

互斥量确保同⼀时刻**只有⼀个线程能够访问共享资源**：

- `获取互斥量`：当⼀个线程获取到互斥量时，其他线程不能访问该资源，直到互斥量被释放。
- `释放互斥量`：当线程释放互斥量后，其他等待的线程才能访问资源

##### **创建互斥量**

```c
/*动态创建一个互斥量*/
rt_mutex_t rt_mutex_create(const char *name, rt_uint8_t flag);
/*静态创建一个互斥量*/
rt_err_t rt_mutex_init(rt_mutex_t mutex, const char *name, rt_uint8_t flag);
```

- <span style="color:#3399FF;">参数说明</span>：
- `name`：互斥量的名字（可以为 `RT_NULL` 表⽰匿名互斥量）。
- `flag`：IPC 对象的属性标志，通常使⽤ `RT_IPC_FLAG_PRIO` 表⽰优先级等待。

##### **获取互斥量**

⽤于获取互斥量，线程通过调⽤此函数进⼊临界区。`如果互斥量已经被其他线程获取，当前线程会进⼊等待状态，直到获取成功或超时。`

```c
rt_err_t rt_mutex_take(rt_mutex_t mutex, rt_int32_t time);
```

- <span style="color:#3399FF;">参数说明</span>：
- `mutex`：互斥量的句柄。
- `time`：超时时间（单位为系统 tick），可以指定等待时⻓或 `RT_WAITING_FOREVER`表⽰永远等待。

##### 释放互斥量

⽤于释放互斥量（V 操作），**增加**互斥量的计数值，并唤醒等待该互斥量的线程。

```c
rt_err_t rt_mutex_release(rt_mutex_t mutex);
```

#### **应用示例**

```c
#include <rtthread.h>

rt_mutex_t mutex = RT_NULL;
rt_uint32_t cnt = 0;

void thread1(void *parameter)
{
   //rt_kprintf("线程1：等待线程2发送信号\n");
   while(1)
   {
       rt_mutex_take(mutex,RT_WAITING_FOREVER);
       cnt++;
       rt_thread_mdelay(1000);
       rt_kprintf("线程1 cnt = %d \n",cnt);
       rt_mutex_release(mutex);
   }
}

void thread2(void *parameter)
{
   //rt_kprintf("线程2：执行一些操作\n");
    while(1)
    {
        rt_mutex_take(mutex,RT_WAITING_FOREVER);
        cnt++;
        rt_thread_mdelay(1000);
        rt_kprintf("线程2 cnt = %d \n",cnt);
        rt_mutex_release(mutex);
    }
}

int main(void)
{
    rt_thread_t tid1 = RT_NULL;
    rt_thread_t tid2 = RT_NULL;

    mutex = rt_mutex_create("mutex", RT_IPC_FLAG_PRIO);
    tid1 = rt_thread_create("tid1", thread1, RT_NULL, 1024, 25, 100);
    tid2 = rt_thread_create("tid2", thread2, RT_NULL, 1024, 25, 100);

    if(tid1 != RT_NULL) rt_thread_startup(tid1);
    if(tid2 != RT_NULL) rt_thread_startup(tid2);
    return 0;
}

```

### **任务同步与任务互斥差异**

![image-20250522203011255](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222030206.png)

### **事件集**

事件集可以看作公交⻋站的场景。线程可以等待多个事件发⽣，`⽐如等待公交⻋或等待同伴到达。如果满⾜某个或某⼏个条件，线程将被唤醒继续执⾏。`

#### **工作机制**

事件集⽤于线程间的同步，可以让线程等待⼀个或多个事件的触发：

- `逻辑与（AND)`：线程等待多个事件同时发⽣才被唤醒。
- `逻辑或（OR）`：线程只需等待其中⼀个事件发⽣即可被唤醒。 

##### **创建事件对象**

```c
/*动态创建一个对象*/
rt_event_t rt_event_create(const char *name, rt_uint8_t flag);
/*静态创建一个对象*/
rt_err_t rt_event_init(rt_event_t event, const char *name, rt_uint8_t flag); 
```

- <span style="color:#3399FF;">参数说明</span>：
- `name`：事件对象的名字（可以为 `RT_NULL` 表⽰匿名事件对象）。
- `flag`：IPC 对象的属性标志，⼀般使⽤ `RT_IPC_FLAG_PRIO` 表⽰优先级等待，或`RT_IPC_FLAG_FIFO` 表⽰先进先出等待。

##### **发送(触发)事件**

```c
rt_err_t rt_event_send(rt_event_t event, rt_uint32_t set);
```

- <span style="color:#3399FF;">参数说明</span>：
- `event`：事件对象的句柄。
- `set`：事件标志，设置哪⼏位事件标志被触发 `可以⽤ 1 << n 来表⽰第 n 位的事件。`

##### **接收(等待)事件**

```c
rt_err_t rt_event_recv(rt_event_t event, rt_uint32_t set, rt_uint8_t option, rt_int32_t timeout, rt_uint32_t *recved);
```

- <span style="color:#3399FF;">参数说明</span>：
- `event`：事件对象的句柄。
- `set`：需要等待的事件标志 `例如 1 << 3 | 1 << 5 表⽰同时等待第3位和第5位的事件`
- `option` ：等待选项，常⻅选项包括：
  - `RT_EVENT_FLAG_AND` ：<span style="color:#3399FF;">所有</span>指定的事件标志都满⾜时才唤醒线程。
  - `RT_EVENT_FLAG_OR` ：<span style="color:#3399FF;">只要有⼀个</span>指定的事件标志满⾜就唤醒线程。
  - `RT_EVENT_FLAG_CLEAR` ：<span style="color:#3399FF;">收到</span>事件标志后<span style="color:#3399FF;">清除</span>这些事件标志。
- `timeout` ：超时时间（单位为系统 tick），可以为 `RT_WAITING_FOREVER` 表⽰永远等待。
- `recved` ：输出参数，返回实际接收到的事件标志

#### **应用示例**

```c
#include <rtthread.h>

rt_event_t event = RT_NULL;
/*线程1：等待事件*/
void thread1(void *parameter)
{
   rt_uint32_t recei;
   rt_kprintf("线程1：等待公交车3或5到来...\n");

   rt_event_recv(event, (1 << 3 | 1 << 5), RT_EVENT_FLAG_OR | RT_EVENT_FLAG_CLEAR, RT_WAITING_FOREVER, &recei);

   rt_kprintf("线程1：公交车 %d 到站，出发！....\n",recei);
}
/*线程2：发送事件（公交车到站）*/
void thread2(void *parameter)
{
   rt_thread_mdelay(1000);//模拟公交车到站
   rt_kprintf("线程2：公交车3到站\n");
   rt_event_send(event, (1 << 3));
}

int main(void)
{
    rt_thread_t tid1 = RT_NULL;
    rt_thread_t tid2 = RT_NULL;

    event = rt_event_create("event", RT_IPC_FLAG_PRIO);
    tid1 = rt_thread_create("tid1", thread1, RT_NULL, 1024, 25, 100);
    tid2 = rt_thread_create("tid2", thread2, RT_NULL, 1024, 25, 100);

    if(tid1 != RT_NULL) rt_thread_startup(tid1);
    if(tid2 != RT_NULL) rt_thread_startup(tid2);
    return 0;
}
```

## **线程间通信**

裸机编程中经常使用全局变量进行功能间的通信（标志）：某些功能由于特定的操作改变全局变量的值，另一个功能对此全局变量进行读取，根据读取到的全局变量执行相应的操作来达到通信协作的目的。RTT提供了邮箱、消息队列和信号等工具来完成这样的类似操作

### **邮箱**

#### 工作机制

邮箱的特点：开销低、效率高、支持一对多

邮箱中一封邮件**只能容纳固定的4字节信息**（针对32位操作系统，指针大小为4字节，一封邮件恰好能容纳一个指针），所以典型的邮箱也被称为交换消息

> 一般如果邮箱中<span style="color:#3399FF;">存在邮件</span>且收取邮件时的<span style="color:#3399FF;">超时时间为0</span>，邮件的收取过程是<span style="font-weight:bold;">非阻塞</span>的
>
> 但邮箱中<span style="color:#3399FF;">不存在邮件</span>且<span style="color:#3399FF;">超时时间不为0</span>时，邮件收取过程就是<span style="font-weight:bold;">阻塞</span>的，邮件收取阻塞的情况下只能由线程进行邮件的收取。

- `邮件发送阻塞`：一个线程向邮箱发送邮件时：
  - 如果邮箱未满，则把邮件复制到邮箱中
  - 如果邮箱已满，则<span style="color:#3399FF;">发送线程挂起</span>并等待邮箱有空间时将其唤醒再发送邮件 或 直接返回`-RT_EFULL`

- `邮件接收阻塞`：一个线程从邮箱中接收邮件时：
  - 如果邮箱已空，接收线程可以挂起直到收到新的邮件被唤醒 `也可以设置超时时间并进行等待`
  - 如果达到设置的超时时间<span style="color:#00FFFF;">但邮箱仍未收到邮件</span>时，超时线程将被唤醒并返回`-RT_ETIMEOUT`；
  - 如果邮箱中存在邮件，则接收线程赋值邮箱中的邮件到接收缓存

##### **创建邮箱**

```C
/*创建动态邮箱*/
rt_mailbox_t rt_mb_create (const char* name, rt_size_t size, rt_uint8_t flag);
/*创建静态邮箱*/
rt_err_t rt_mb_init(rt_mailbox_t mb,const char* name,void* msgpool,rt_size_t size,rt_uint8_t flag)
```

- <span style="color:#3399FF;">参数说明</span>：
- `mb`：邮箱对象的句柄。
- `name`：邮箱容量(几封邮件)
- `size` ：邮箱容量(几封邮件)。
- `flag` ：邮箱标志，它可以取如下数值：`一般除非应用程序非常在意先来后到，不然都采用PRIO确保线程实时性`
  - `RT_IPC_FLAG_FIFO` ：非实时调度方式 先来后到
  - `RT_IPC_FLAG_PRIO` ：实时调度方式 优先级
- `msgpool` ：缓冲区指针。
- `recved` ：输出参数，返回实际接收到的事件标志

##### **删除或脱离邮箱**

```C
/*删除动态邮箱create*/
rt_err_t rt_mb_delete (rt_mailbox_t mb);
/*删除静态邮箱init*/
rt_err_t rt_mb_detach(rt_mailbox_t mb);

//返回值：RT_EOK 成功
```

- <span style="color:#3399FF;">参数说明</span>：
- `mb`：邮箱对象的句柄。

> 调用删除函数时，内核会<span style="color:#00FFFF;">先唤醒所有</span>挂在该邮箱上的线程（线程返回值是`RT_ERROR`），然后<span style="color:#00FFFF;">再释放</span>邮箱使用的<span style="color:#00FFFF;">内存</span>，最后<span style="color:#00FFFF;">删除邮箱对象</span>

##### **发送邮件**

```c
/*1.直接发送邮件*/
/*！！！！！记得value需要强转rt_uint32_t！！！！！*/
rt_err_t rt_mb_send (rt_mailbox_t mb, rt_uint32_t value);
/**************返回值*****************/
RT_EOK        发送成功
-RT_EFULL      邮箱已经满了

/*2.等待方式发送邮件*/
/*@note 若邮箱已满，则根据设定的timeout等待邮箱中因为收取邮件而空出空间
/*@note 若超时时间到达依然没有空出时间，此时发送线程被唤醒并返回错误码
*/
rt_err_t rt_mb_send_wait (rt_mailbox_t mb,rt_uint32_t value,rt_int32_t timeout);
/**************返回值*****************/
RT_EOK        发送成功
-RT_ETIMEOUT     超时
-RT_ERROR    失败，返回错误

/*3.发送紧急邮件*/
/*@note 发送紧急邮件时，邮件被直接插队放入了邮件队首,这样就能最优先接收到并做出处理
*/
rt_err_t rt_mb_urgent (rt_mailbox_t mb, rt_ubase_t value);
/**************返回值*****************/
RT_EOK       发送成功
-RT_EFULL    邮箱已满
```

- <span style="color:#3399FF;">参数说明</span>：
- `mb`：邮箱对象的句柄。
- `value` ：发送的邮件内容。
- `timeout` ：超时时间。

##### **接收邮件**

```C
/*@note 只有当接收者接收的邮箱中有邮件时，接收者才能立即取到邮件并返回 RT_EOK 的返回值
否则接收线程会根据超时时间设置，或挂起在邮箱的等待线程队列上，或直接返回。
！！！！！记得value需要强转rt_uint32_t！！！！！
*/
rt_err_t rt_mb_recv (rt_mailbox_t mb, rt_uint32_t* value, rt_int32_t timeout);
/**************返回值*****************/
RT_EOK        接收成功
-RT_ETIMEOUT     超时
-RT_ERROR    失败，返回错误
```

- <span style="color:#3399FF;">参数说明</span>：
- `mb`：邮箱对象的句柄。
- `value` ：邮件内容`(定义一个变量来接收 记得强转)`
- `timeout` ：超时时间。

#### **应用示例**

```C
#include <rtthread.h>

/*创建邮箱*/
rt_mailbox_t mb;

/*线程1：发送邮箱*/
void thread_entry1(void *parameter)
{
    char msg = 'A';//发送'A'作为邮件
    rt_kprintf("线程1：发送邮件...\n");
    //发送邮件  因为邮箱固定4字节 所以需要强转
    rt_mb_send(mb, (rt_uint32_t)msg);
}
/*线程2：接收邮件*/
void thread_entry2(void *parameter)
{
   char msg;
   rt_kprintf("线程2：等待接收邮件...\n");
   //若邮箱内为空  则执行的是阻塞式的等待接收邮件
   rt_mb_recv(mb, (rt_uint32_t)&msg, RT_WAITING_FOREVER);//接收邮件
   rt_kprintf("线程2：收到邮件：%c\n",msg);
}

int main(void)
{
    /*创建一个容量为4的4封邮箱(大小为4*4)) */
    mb = rt_mb_create("mb", 4, RT_IPC_FLAG_PRIO);
    /*创建两个线程 */
    rt_thread_t tid1 = rt_thread_create("t1", thread_entry1, RT_NULL, 1024, 10,
    10);
    rt_thread_t tid2 = rt_thread_create("t2", thread_entry2, RT_NULL, 1024, 10,
    10);
    /* 启动线程 */
    rt_thread_startup(tid1);
    rt_thread_startup(tid2);
    return 0;
}
```

### **消息队列**

消息队列是另一种常用的线程间通讯方式，是邮箱的扩展。他可以动态分配内存，对比邮箱局限的4字节存储特性，更适合用来传输复杂大量数据或进行任务队列管理

#### **工作机制**

##### **创建消息队列**

```C
/*创建动态消息队列*/
rt_mq_t rt_mq_create(const char* name, rt_size_t msg_size,rt_size_t max_msgs, rt_uint8_t flag);

/*创建静态消息队列*/
/*@note 一般放于读数据段或未初始化数据段中。在使用这类静态消息队列对象前，需要进行初始化
*/
rt_err_t rt_mq_init(rt_mq_t mq, const char* name,void *msgpool,rt_size_t msg_size,rt_size_t pool_size, rt_uint8_t flag);
```

- <span style="color:#3399FF;">参数说明</span>：

- `name `：消息队列的名称。

- `msg_size ` ：消息队列中一条消息的最大长度，<span style="color:#00FFFF;">单位字节</span>。

- `msg_msgs`：消息队列的最大个数(消息个数)。

- `flag` ：消息队列采用的等待方式，它可以取如下数值：`一般除非应用程序非常在意先来后到，不然都采用PRIO确保线程实时性`

  - `RT_IPC_FLAG_FIFO`：非实时调度方式 先来后到

  - `RT_IPC_FLAG_PRIO`：实时调度方式 优先级

- `mq` ：消息队列对象的句柄。

- `msgpool`：指向存放消息的缓冲区的指针。

- `pool_size` ：存放消息的缓冲区大小

##### **删除或脱离消息队列**

```C
/*删除动态消息队列create*/
rt_err_t rt_mq_delete(rt_mq_t mq);
/*删除静态消息队列init*/
rt_err_t rt_mq_detach(rt_mq_t mq);
//返回值：RT_EOK 成功
```

- <span style="color:#3399FF;">参数说明</span>：
- `mq`：消息队列对象的句柄

> 删除消息队列时，如果<span style="color:#00FFFF;">有线程</span>被挂起在该消息队列等待队列上，则内核<span style="color:#00FFFF;">先唤醒挂起</span>在该消息等待队列上的所有线程（线程返回值是 `- RT_ERROR`），然后再<span style="color:#00FFFF;">释放</span>消息队列使用的内存，最后<span style="color:#00FFFF;">删除</span>消息队列对象

##### **发送消息**

```C
/*1.直接发送消息*/
rt_err_t rt_mq_send (rt_mq_t mq, void* buffer, rt_size_t size);
/**************返回值*****************/
RT_EOK        发送成功
-RT_EFULL      消息队列已经满了
-RT_ERROR       失败(发送消息长度>队列的最大长度)

/*2.等待方式发送消息*/
/*@note 若消息队列已满，则根据设定的timeout等待
/*@note 若超时时间到达依然没有空出时间，此时发送线程被唤醒并返回错误码
*/
rt_err_t rt_mq_send_wait(rt_mq_t mq,const void *buffer,rt_size_t size,rt_int32_t  timeout);
/**************返回值*****************/
RT_EOK        发送成功
-RT_ETIMEOUT     超时
-RT_ERROR    失败(发送消息长度>队列的最大长度)

/*3.发送紧急消息*/
/*@note 发送紧急消息时，邮件被直接插队放入了消息队列队首,这样就能最优先接收到并做出处理
*/
rt_err_t rt_mq_urgent(rt_mq_t mq, void* buffer, rt_size_t size);
/**************返回值*****************/
RT_EOK       发送成功
-RT_EFULL    消息队列已经满了
-RT_ERROR    失败(发送消息长度>队列的最大长度)
```

- <span style="color:#3399FF;">参数说明</span>：
- `mq`：消息队列对象的句柄
- `buffer` ：发送的消息内容。
- `size` ：消息大小。
- `timeout` ：超时时间。

##### **接收消息**

```C
/*@note 只有当消息队列中有消息时，接收者才能立即取到邮件并返回 RT_EOK 的返回值
否则接收线程会根据超时时间设置，或挂起在邮箱的等待线程队列上，或直接返回。
*/
rt_ssize_t rt_mq_recv (rt_mq_t mq, void* buffer,rt_size_t size,rt_int32_t timeout);
/**************返回值*****************/
RT_EOK             成功收到
-RT_ETIMEOUT             超时
-RT_ERROR            失败，返回错误
```

- <span style="color:#3399FF;">参数说明</span>：
- `mq`：消息队列对象的句柄
- `buffer` ：发送的消息内容。
- `size` ：消息大小。
- `timeout` ：超时时间

#### **应用示例**

```C
#include <rtthread.h>

/*创建消息队列*/
rt_mq_t mq;

/*线程1：发送消息*/
void thread_entry1(void *parameter)
{
    char msg[] = "Hello, RT-Thread!";//发送'A'作为邮件
    rt_kprintf("线程1：发送消息...\n");
    //发送消息到消息队列
    rt_mq_send(mq, msg,sizeof(msg));
}
/*线程2：接收邮件*/
void thread_entry2(void *parameter)
{
   char buffer[32];
   rt_kprintf("线程2：等待接收消息...\n");
   //若消息队列内无消息  则执行的是阻塞式的等待接收消息
   rt_mq_recv(mq, buffer,sizeof(buffer), RT_WAITING_FOREVER);//接收消息
   rt_kprintf("线程2：收到消息：%s\n",buffer);
}

int main(void)
{
    /*创建一个消息队列，容量为4，消息大小为32字节 */
    mq = rt_mq_create("mq", 32, 4, RT_IPC_FLAG_PRIO);
    /*创建两个线程 */
    rt_thread_t tid1 = rt_thread_create("t1", thread_entry1, RT_NULL, 1024, 10,
    10);
    rt_thread_t tid2 = rt_thread_create("t2", thread_entry2, RT_NULL, 1024, 10,
    10);
    /* 启动线程 */
    rt_thread_startup(tid1);
    rt_thread_startup(tid2);
    return 0;
}

```

### **信号**

`信号（又称为软中断信号）`，在软件层次上是对中断机制的一种模拟。常用于通知线程发生了某个事件。线程收到信号时会根据信号类型执行相应的操作

- 与消息队列区别：

1. <span style="color:#00FFFF;">信号仅携带控制信息，不包含数据</span>

2. 信号不保证顺序性，<span style="color:#00FFFF;">信号发生时立即通知线程</span>

3. 消息队列则有顺序性（通常是PRIO）

#### **工作机制**

##### **线程中安装信号**

```C
/*
t_sighandler_t[] 表示一个函数指针数组
t_sighandler_t handler[] 是一个 变量声明
二者是不一样的！
*/
rt_sighandler_t rt_signal_install(int signo, rt_sighandler_t[] handler);
/**************返回值*****************/
SIG_ERR                错误的信号
安装信号前的handler值    成功
```

- <span style="color:#3399FF;">参数说明</span>：
- `signo `：信号值（只有 `SIGUSR1` 和 `SIGUSR2` 是开放给用户使用的）。
- `buffer` ：发送的消息内容。
- `handler` ：设置对信号值的处理方式。

>  在信号安装时设定 handler 参数，决定了该信号的不同的处理方法。处理方法可以分为三种：
>
> 1. 类似`中断`的处理方式，参数指向当信号发生时用户`自定义的处理函数`， 在内部写入处理方式，由该函数来处理。
>
> 2. 参数设为 `SIG_IGN`，忽略某个信号，`对该信号不做任何处理`，就像未发生过一样。
>
> 3. 参数设为 `SIG_DFL`，系统会调用默认的处理函数`_signal_default_handler()`。

##### **阻塞信号与解除阻塞**

```C
/*信号阻塞(屏蔽信号) 该信号不会递答安装此信号的线程*/
void rt_signal_mask(int signo);
/*解除信号阻塞
使用此函数可以对其中一些信号给予 “关注”，那么发送这些信号都会引发该线程的软中断
*/
void rt_signal_unmask(int signo);
```

- <span style="color:#3399FF;">参数说明</span>：
- `signo`：信号值

##### **发送信号**

```C
/*需要异常处理时，可以给安装了信号的线程发送信号*/
int rt_thread_kill(rt_thread_t tid, int sig);
/**************返回值*****************/
RT_EOK        发送成功
-RT_EINVAL    参数错误
```

- <span style="color:#3399FF;">参数说明</span>：
- `tid`：接收信号的线程。
- `sig `：信号值。

#### **应用示例**

```C
#include <rtthread.h>

#define THREAD_PRIORITY         25
#define THREAD_STACK_SIZE       512
#define THREAD_TIMESLICE        5

static rt_thread_t tid1 = RT_NULL;

/* 线程 1 的信号处理函数 */
void thread1_signal_handler(int sig)
{
    rt_kprintf("thread1 received signal %d\n", sig);
}

/* 线程 1 的入口函数 */
static void thread1_entry(void *parameter)
{
    int cnt = 0;

    /* 安装信号 */
    rt_signal_install(SIGUSR1, thread1_signal_handler);
    rt_signal_unmask(SIGUSR1);

    /* 运行 10 次 */
    while (cnt < 10)
    {
        /* 线程 1 采用低优先级运行，一直打印计数值 */
        rt_kprintf("thread1 count : %d\n", cnt);

        cnt++;
        rt_thread_mdelay(100);
    }
}

/* 信号示例的初始化 */
int signal_sample(void)
{
    /* 创建线程 1 */
    tid1 = rt_thread_create("thread1",thread1_entry, RT_NULL,THREAD_STACK_SIZE, THREAD_PRIORITY, THREAD_TIMESLICE);

    if (tid1 != RT_NULL)
        rt_thread_startup(tid1);

    rt_thread_mdelay(300);

    /* 发送信号 SIGUSR1 给线程 1 */
    rt_thread_kill(tid1, SIGUSR1);

    return 0;
}

```

## **框架移植**

```C
//bsp_system.h中引入以下四个头文件
#include <rtthread.h>
#include <rtdevice.h>
#include <drv_common.h>
#include <board.h>
```

### **系统初始化宏定义**

​    ![image-20250522214349807](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222143758.png)

```C
/*1、板级初始化 fn为函数指针*/
INIT_BOARD_EXPORT(fn)
/*2、纯软件函数初始化 fn为函数指针*/
INIT_PREV_EXPORT(fn)
/*3、设备自动初始化，fn为函数指针*/
INIT_DEVICE_EXPORT(fn)
/*4、组件自动初始化，fn为函数指针*/
INIT_COMPONENT_EXPORT(fn)
/*5、系统环境自动初始化，fn为函数指针*/
INIT_ENV_EXPORT(fn) 
/*6、应用自动初始化，fn为函数指针*/
INIT_APP_EXPORT(fn)
```

### **ESP8266移植**

> 将`uart_app.c`与`uart_app.h`移去 不使用！！！然后RTT打开控制台输出rt_kprintf，选择串口1

   ![image-20250522214536841](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222145092.png)

![image-20250522220417747](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222204947.png) ![image-20250522220447380](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222204548.png)

![image-20250522214750185](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222147073.png)

![image-20250522214705123](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222147664.png)

![image-20250522214840651](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222148942.png)

![image-20250522214859546](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222149729.png)

![image-20250522214917115](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222149203.png)

![image-20250522214947877](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222149796.png)

直接编译开启终端 下载观察现象，出现以下信息则连接成功

​    ![image-20250522215015453](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222150747.png)

### **DHT11移植**

#### **组件添加**

​    ![image-20250522215146162](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222151707.png)

![image-20250522215207600](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222152193.png)

![image-20250522220342327](https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202505222203028.png)



#### **proc代码构建**

##### **创建sensor_app.c与.h**

```C
//引入必要库
#include <sensor_app.h>
#include <sensor.h>
#include "sensor_dallas_dht11.h"
```

##### **注册设备**

    ```C
        rt_device_t dev = RT_NULL;
        //创建传感器结构体对象
        struct rt_sensor_data sensor_data;
        rt_uint8_t get_data_freq = 1;///默认1hz
        rt_size_t res;//返回值 如RT_EOK
    
        /*注册传感器*/
        dev =  rt_device_find("temp_dht11");//注意此处注册需要有前缀 如temp_(在sensor.c中)
        if(dev == RT_NULL)
        {
            rt_kprintf("Not Found\r\n");
            return;
        }
        if(rt_device_open(dev, RT_DEVICE_FLAG_RDWR) != RT_EOK)
        {
            rt_kprintf("Open Error\r\n");
            return;
        }
    
        rt_device_control(dev, RT_SENSOR_CTRL_SET_ODR, (void *)&get_data_freq);
    ```

##### **读取传感器数据**

```C
 /*读取传感器*/
    while(1)
    {
        res = rt_device_read(dev, 0, &sensor_data, 1);//第0位开始读一个
        //读错了返回0
        if(res == 0)
        {
            rt_kprintf("Read Error Res:%d",res);//返回读取错误编号0
            return;
        }
        else //读取成功
        {
            if(sensor_data.data.temp >= 0) //数据大于等于0 包括了温度与湿度
            {
                uint8_t temp = (sensor_data.data.temp & 0xffff);
                uint8_t humi = (sensor_data.data.temp & 0xffff0000) >> 16;
                rt_kprintf("temp:%d humi:%d\r\n",temp,humi);
            }
        }
        rt_thread_delay(1000);
    }
```

##### **init代码构建**

```C
int sensor_init(void)
{
    struct rt_sensor_config cfg;//操作集结构体
    /*注册设备 配置接口*/
    cfg.intf.user_data = (void *)DHT11_PIN;//配置接口为IO口
    rt_hw_dht11_init("dht11", &cfg);

    rt_thread_t sensor_thread = rt_thread_create("sensor_proc", sensor_proc, RT_NULL, 1024, 10, 20);
    if(sensor_thread != RT_NULL)
    {
        rt_thread_startup(sensor_thread);
    }
    return RT_EOK;
}
INIT_COMPONENT_EXPORT(sensor_init);
```

