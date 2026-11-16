---
title: C语言进阶 21：联合体、枚举与带标签的数据模型
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 进阶, 联合体, 枚举]
category: C语言
series: C语言学习笔记
seriesOrder: 21
draft: false
updated: 2026-11-16
---

> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- union 成员共享存储，读取非最近写入成员涉及表示重解释，不能对任意组合保证可移植值。用标签明确当前有效成员。
- enum 不自动校验输入属于列出的常量；C17 不保证枚举对象总与 int 同大小。作为数组下标前检查范围。

## 本章目标

1. 掌握联合体的特点和代码的编写

2. 掌握联合体的使用场景

3. 掌握枚举的优点和使用场景，能够编写代码

## 1. 联合体

### 1.1 联合体类型的声明

像结构体一样，联合体也是由一个或者多个成员构成，这些成员可以是不同的类型，联合体的关键字是 union 。

但是编译器只为最大的成员分配足够的内存空间，联合体的特点是：所有成员共用同一块内存空间。

所以联合体也叫：共用体，给联合体其中一个成员赋值，其他成员的值也跟着变化。

联合体类型声明的语法：

**课件代码 21-001 · PDF 第 1 页**

```c
union 名字
{
    成员列表;
};
```

联合体类型的声明方式和结构体非常相似，但是实际意义有很大的区别，下面慢慢体会。

我们首先看一个代码，来感受一下，联合体和结构体的差异：

**课件代码 21-002 · PDF 第 1,2 页**

```c
#include <stdio.h>

//联合类型的声明
union Un
{
    char c;
    int i;
};

int main()
{
    //联合变量的定义
    union Un un = {0};
    //计算连个变量的大小
    printf("%d\n", sizeof(un));

    return 0;
}
```

输出的结果：

**课件代码 21-003 · PDF 第 2 页**

```text
4
```

为什么是4呢？如果这里不是联合体，而是换成成员相同的结构体的话，结果应该是8个字节。

### 1.2 联合体的特点

接下来我们探究一下，联合体的特点，先看两段代码：

代码块 代码块

**课件代码 21-004 · PDF 第 2,3 页**

```c
//代码1
#include <stdio.h>

//联合类型的声明
union Un
{
    char c;
    int i;
};

int main()
{
    //联合变量的定义
    union Un un = {0};
    //看看输出的结果
    printf("%p\n", &(un.i));
    printf("%p\n", &(un.c));
    printf("%p\n", &un);
    return 0;
}
```

**课件代码 21-005 · PDF 第 2,3 页**

```c
//代码2
#include <stdio.h>

//联合类型的声明
union Un
{
    char c;
    int i;
};

int main()
{
    //联合变量的定义
    union Un un = {0};
    un.i = 0x11223344;
    un.c = 0x55;
    printf("%x\n", un.i);
    return 0;
}
```

输出的结果：

代码块 代码块

**课件代码 21-006 · PDF 第 3 页**

```text
001AF85C
001AF85C
001AF85C
```

**课件代码 21-007 · PDF 第 3 页**

```text
11223355
```

代码1输出的三个地址一模一样，代码2的输出，我们发现将 i 的第4个字节的内容修改为 0x55 了。

我们仔细分析就可以画出， un 的内存布局图。

![第21章第3页配图](/images/c-language-course/21-p03-fig01.png)

结论：

- 联合的成员是共用同一块内存空间的

- 这样一个联合变量的大小，至少是最大成员的大小，因为联合至少得有能力保存最大的那个成员。

- 同一时刻只能安全地使用一个成员。

### 1.3 相同成员的结构体和联合体对比

我们再对比一下相同成员的结构体和联合体的内存布局情况。

代码块 代码块

**课件代码 21-008 · PDF 第 3,4 页**

```c
struct S
{
    char c;
    int i;
};
struct S s = {0};
```

**课件代码 21-009 · PDF 第 3,4 页**

```c
union Un
{
    char c;
    int i;
};
union Un un = {0};
```

![第21章第4页配图](/images/c-language-course/21-p04-fig01.png)

结构体和联合体的内存对比

### 1.4 联合体大小的计算

- 联合的大小至少是最大成员的大小。

- 当最大成员大小不是最大对齐数的整数倍的时候，就要对齐到最大对齐数的整数倍。

**课件代码 21-010 · PDF 第 4 页**

```c
#include <stdio.h>
union Un1
{
    char c[5];
    int i;
};

union Un2
{
    short c[7];
    int i;
};

int main()
{
    //下面输出的结果是什么？
    printf("%d\n", sizeof(union Un1));
    printf("%d\n", sizeof(union Un2));
    return 0;
}
```

### 1.5 联合体使用举例

比如，我们要搞一个活动，要上线一个礼品兑换单，礼品兑换单中有三种商品：图书、杯子、衬衫。

每一种商品都有：库存量、价格、商品类型和商品类型相关的其他信息。

图书：书名、作者、页数杯子：设计衬衫：设计、可选颜色、可选尺寸我们梳理一下各种商品完整的属性：

![第21章第5页图表](/images/c-language-course/21-p05-table02.png)

那我们不耐心思考，直接写出一下结构：

**课件代码 21-011 · PDF 第 5,6 页**

```c
struct gift_list
{
    //公共属性
    int stock_number;//库存量
    double price; //定价
    int item_type;//商品类型

    //特殊属性
    char title[20];//书名
    char author[20];//作者
    int num_pages;//页数

    char design[30];//设计2
    int colors;//颜色
    int sizes;//尺寸
};
```

上述的结构其实设计的很简单，用起来也方便，但是结构的设计中包含了所有礼品的各种属性，这样使得结构体的大小就会偏大，比较浪费内存。因为对于礼品兑换单中的商品来说，只有部分属性信息是常用的。

比如：商品是图书，就不需要 design 、 colors 、 sizes 。

所以可以把公共属性单独写出来，剩余属于各种商品本身的属性使用联合体起来，这样就可以介绍所需的内存空间，一定程度上节省了内存。

**课件代码 21-012 · PDF 第 6 页**

```c
struct gift_list
{
    int stock_number;//库存量
    double price; //定价
    int item_type;//商品类型

    union
    {
        struct
        {
            char title[20];//书名
            char author[20];//作者
            int num_pages;//页数
        } book;
        struct
        {
            char design[30];//设计
        } mug;
        struct
        {
            char design[30];//设计
            int colors;//颜色
            int sizes;//尺寸
        } shirt;
    } item;
};
```

### 1.6 联合的一个练习

写一个程序，判断当前机器是大端？还是小端？

这个题目我们之前做过的

**课件代码 21-013 · PDF 第 7 页**

```c
int check_sys()
{
    union
    {
        int i;
        char c;
    }un;
    un.i = 1;
    return un.c;//返回1是小端，返回0是大端
}
```

## 2. 枚举类型

写代码时，状态码、类型码用0、1、2直接写，过两周你自己都忘了含义。 enum 就是给整型常量起个有意义的名字，提高可读性和可维护性。

### 2.1 枚举类型的声明

枚举顾名思义就是一一列举，把某种类型的可能的取值一一列举。

比如我们现实生活中：

一周的星期一到星期日是有限的7天，可以一一列举性别有：男、女、保密，也可以一一列举月份有12个月，也可以一一列举三原色（红、绿、蓝）也是可以一一列举红绿灯的颜色（红、绿、黄），也是可以一一列举这些数据的表示就可以使用枚举了。

星期 性别 红绿灯

**课件代码 21-014 · PDF 第 7 页**

```c
enum Day
{
    Mon,
    Tues,
    Wed,
    Thur,
    Fri,
    Sat,
    Sun
};
```

**课件代码 21-015 · PDF 第 7 页**

```c
enum Sex
{
    MALE,
    FEMALE,
    SECRET
}
```

**课件代码 21-016 · PDF 第 7 页**

```c
enum TrafficLight
{
    RED,
    GREEN,
    YELLOW
};
```

以上定义的 enum Day ， enum Sex ， enum TrafficLight 都是枚举类型。

{}中的内容是枚举类型的可能取值，也叫 枚举常量 。

这些可能取值都是有值的，默认从0开始，依次递增1，当然在声明枚举类型的时候也可以赋初值。

**课件代码 21-017 · PDF 第 8 页**

```c
enum Color//颜色
{
    RED=2,
    GREEN=4,
    BLUE=8
};
```

### 2.2 枚举类型的优点

为什么使用枚举？

我们可以使用 #define 定义常量，也可以让整型值有名字可用，为什么非要使用枚举？

枚举的优点：

1. 增加代码的可读性和可维护性

2. 和#define定义的标识符比较枚举有类型检查，更加严谨。

3. 便于调试，预处理阶段会删除 #define 定义的符号

4. 使用方便，一次可以定义多个常量

5. 枚举常量是遵循作用域规则的，枚举声明在函数内，只能在函数内使用

### 2.3 枚举类型的使用

**课件代码 21-018 · PDF 第 8,9 页**

```c
#include <stdio.h>
#include <windows.h>

// 状态枚举，末尾的 LIGHT_COUNT 自动得到状态总数
typedef enum
{
    RED,
    GREEN,
    YELLOW,
    LIGHT_COUNT
} TrafficLight;

int main()
{
    // 每种灯的名称和持续时间，用枚举值作为索引
    const char* message[] = { [RED] = "🔴 红灯",[GREEN] = "🟢 绿灯",[YELLOW] = "🟡 黄灯" };
    const int seconds[]   = { [RED] = 10,       [GREEN] = 10,       [YELLOW] = 3 };

    TrafficLight current = RED;

    printf("🚦 自动红绿灯 (Ctrl+C 退出)\n\n");

    while (1)
    {
        int sec = seconds[current];//计算出当前状态下的秒数
        printf("现在是 %s，持续 %d 秒\n", message[current], sec);

        for (int i = sec; i > 0; i--)
        {
            printf("  剩余 %2d 秒\n", i);
            Sleep(1000);          // 暂停1秒
        }

        printf("  >>> %s 结束 <<<\n\n", message[current]);

        // 循环切换到下一个灯
        current = (current + 1) % LIGHT_COUNT;
    }

    return 0;
}
```
