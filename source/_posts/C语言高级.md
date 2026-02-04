---
title: C语言高级
date: 2025-09-02 11:36:37
cover: "https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509171225629.png"
tags:
  - 嵌入式基础
  - C语言高级 
---

## 关于const的进一步思考

> 背景：在进行宿舍精灵项目的代码编写时， 我使用了映射表来表现楼栋，我使用了const来修饰 以稳定内容 
>
> 但是在我尝试将楼栋的内容分离出来时（例如中1C 在oled上需把中文提取出来 然后剩下用oled_print来显示在oled上） 但却出现了指针错误 

分析了一下：const是将对应的字符数组存放在只读存储区中 而我因为想要分离 从而修改了指针的内容 故造成指针错误

`解决措施`：通过malloc来申请内存，然后使用strcpy或者sprintf来存放在新申请的内存中

```C
char *get_building_map_remain(void)
{
    char *map = get_building_map();
    if (map == NULL)
    {
        return NULL;
    }

    size_t len = strlen(map);
    if (len <= 3)
    {
        // 返回空字符串
        char *result = malloc(1);
        if (result)
        {
            result[0] = '\0';
        }
        return result;
    }

    // 分配新内存并复制跳过前3字节的内容
    char *result = malloc(len - 3 + 1);
    if (result == NULL)
    {
        return NULL;
    }

    strcpy(result, map + 3);
    return result;
}
```



{% externalLinkCard "freertos" "https://mextra.netlify.app/2025/09/05/freertos/" "https://cdn.jsdelivr.net/gh/2250272571/tuchuang@main/img/202509071511987.png" %}

## 二维数组高级应用

> 背景：在进一步封装MP3_Test项目的时候，本身有个二维数组mp3Files用来存放MP3文件路径，在修饰为static保护变量时，遇到了不会返回的难题
>
> []优先级高于*

`解决措施`：通过询问`AI`后，得出可以将其封装为`一个返回值为数组指针的函数`，然后定义一个数组指针来接收`原型为指针，但指向的是数组`

### 函数封装

```C
static char mp3Files[100][64]; // 假设最多有100个MP3文件

char (*get_mp3_files(void)) [64]
{
    return mp3Files;
}

void task(void)
{
    char (*files)[64] = get_mp3_files();
    for(int i = 0; i < 100 ;i++)
        printf("file[%d]:%s\r\n",i,files[i]);
}
```

### 指针访问

```C
static char mp3Files[100][64]; // 假设最多有100个MP3文件

void task(void)
{
    /*（1）指向数组的指针（按行访问） 即指针数组*/
    char (*p)[64] = mp3Files;
    /*（2）指向数组的指针（逐个访问）*/
    char *p = &mp3Files[0][0];
 //或	char *p = mp3Files[0];
}
```

