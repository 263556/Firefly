---
title: C语言进阶 16：函数指针、回调与通用排序
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 进阶, 函数指针, 回调]
category: C语言
series: C语言学习笔记
seriesOrder: 16
draft: false
updated: 2026-11-16
---

> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- qsort 比较函数直接相减可能导致有符号溢出。整数比较可写为 `(a > b) - (a < b)`，结构体也按字段返回符号。
- 课件零地址函数调用只是声明拆解题，调用空函数指针是未定义行为；函数指针不能普遍按对象指针传给 `%p`。qsort 不保证稳定性，也不保证具体使用快速排序。

## 本章目标

1. 掌握函数指针变量

2. 掌握回调函数的基本原理

3. 能够使用函数指针实现回调函数代码的编写

4. 学会拆解抽象的函数指针相关的代码

5. 能够使用typedef简化抽象代码

6. 熟悉qsort的使用

7. 理解qsort函数函数的设计原理，自己能模拟实现

## 1. 函数的地址

我们之前学习到变量有地址，数组有地址，那函数有地址吗？

我们做个测试：

**课件代码 16-001 · PDF 第 1 页**

```c
#include <stdio.h>
void test()
{
    printf("hehe\n");
}
int main()
{
    printf("test:  %p\n", test);
    printf("&test: %p\n", &test);
    return 0;
}
```

输出结果如下：

**课件代码 16-002 · PDF 第 1,2 页**

```text
test:  00007FF6D08013AC
&test: 00007FF6D08013AC
```

- 确实打印出来了地址，所以函数是有地址的；

- 函数名就是函数的地址，当然也可以通过 &amp;函数名 的方式获得函数的地址。

- 函数名 和 &amp;函数名 获得的都是函数的地址，没有区别。

如果我们要将函数的地址存放起来，就得创建函数指针变量，函数指针变量的写法其实和数组指针非常类似。

## 2. 函数指针变量

### 2.1 函数指针变量的创建

什么是函数指针变量呢？

根据前面学习数组指针变量的时候，我们的类比方法

- 整型指针变量，存放整型变量的地址，是指向整型变量的指针

- 数组指针变量，存放了数组的地址，是指向数组的指针变量

那么函数指针变量应该是用来存放函数地址的，是指向函数的指针变量。

如下：

**课件代码 16-003 · PDF 第 2 页**

```c
void test()
{
    printf("hehe\n");
}

void (*pf1)() = &test;
void (*pf2)()= test;

int Add(int x, int y)
{
    return x+y;
}

int(*pf3)(int, int) = Add;
int(*pf3)(int x, int y) = &Add;//x和y写上或者省略都是可以的
```

函数指针类型解析：

**课件代码 16-004 · PDF 第 3 页**

```c
int      (* pf3)   (int x, int y)
 |          |      ------------
 |          |            |
 |          |            pf3指向函数的参数类型和个数的交代
 |          函数指针变量名
 pf3指向函数的返回类型

 int (*) (int x, int y)  //pf3函数指针变量的类型
```

### 2.2 函数指针变量的使用

通过函数指针调用其指向的函数。

代码块 代码块

**课件代码 16-005 · PDF 第 3 页**

```c
#include <stdio.h>

int Add(int x, int y)
{
    return x+y;
}

int main()
{
    int(*pf3)(int, int) = Add;
    printf("%d\n", (*pf3)(2, 3));

    return 0;
}
```

**课件代码 16-006 · PDF 第 3 页**

```c
#include <stdio.h>

int Add(int x, int y)
{
    return x+y;
}

int main()
{
    int(*pf3)(int, int) = Add;
    printf("%d\n", pf3(3, 5));

    return 0;
}
```

输出结果：

**课件代码 16-007 · PDF 第 3 页**

```text
5
8
```

拿到函数指针后，通过函数指针调用函数时，上面两种方式都是支持的。

## 3. 两段有趣的代码

### 3.1 练习

代码1

**课件代码 16-008 · PDF 第 4 页**

```c
(*(void (*)())0)();
```

代码解析：

1. 上述代码其实是一次函数调用，调用的是0地址处的一个函数，这个函数没有参数，没有返回值。

2. 代码中的 void (\*)() 是函数指针类型， (void (\*)())0 类型放在括号中意思是强制类型转

化，是将0这个整型值，强制类型转化成这种函数指针类型，也就是说0被当做函数的地址了。

3. \*(void (\*)())0 ，前面加一个 \* ，就是调用0地址处的这个函数，根据函数指针的类型能知

道，这个函数没有参数，也没有返回值。

代码2

**课件代码 16-009 · PDF 第 4 页**

```c
void ( *signal(int , void(*)(int)) )(int);
```

![第16章第5页配图](/images/c-language-course/16-p05-fig01.png)

代码解析：

1. 上述代码是一次函数的声明。

2. 声明的函数名字叫 signal ，函数的参数有2个，第一个是int类型，第二个是函数指针类

型: void(\*)(int) 。 signal 函数的返回值类型也是函数指针类型 void(\*)(int) 。

上述两段代码均出自：《C陷阱和缺陷》这本书上面两段代码中都是出现了函数指针类型，代码的可读性不是很好，有没有什么办法能简化一下代码呢？

### 3.2 typedef 关键字

typedef 是用来类型重命名的，可以将复杂的类型，简单化。

1. typedef对普通数据类型重命名

比如，你觉得 unsigned int 写起来不方便，如果能写成 uint 就方便多了，那么我们可以使用：

**课件代码 16-010 · PDF 第 5 页**

```c
typedef unsigned int uint;
//将unsigned int 重命名为uint
```

2. typedef 对指针类型重命名

如果是指针类型，能否重命名呢？其实也是可以的，比如，将 int\* 重命名为 ptr\_t ,这样写：

**课件代码 16-011 · PDF 第 5 页**

```c
typedef int* ptr_t;
```

连续定义多个指针不同写法和效果差异对比：

代码块 代码块 代码块

**课件代码 16-012 · PDF 第 6 页**

```c
int *p1, p2;
int *p3, *p4;
```

**课件代码 16-013 · PDF 第 6 页**

```c
typedef int* pint_t;
pint_t p5, p6;
```

**课件代码 16-014 · PDF 第 6 页**

```c
#define PINT_T int*;
PINT_T p7, p8;
```

3. typedef 对数组指针类型和函数指针类型重命名

但是对于数组指针和函数指针稍微有点区别：

比如我们有数组指针类型 int(\*)[5] ,需要重命名为 parr\_t ，那可以这样写：

**课件代码 16-015 · PDF 第 6 页**

```c
typedef int(*parr_t)[5]; //新的类型名必须在*的右边
```

函数指针类型的重命名也是一样的，比如，将 void(\*)(int) 类型重命名为 pf\_t ,就可以这样写：

**课件代码 16-016 · PDF 第 6 页**

```c
typedef void(*pfun_t)(int);//新的类型名必须在*的右边
```

那么要简化代码2，可以这样写：

**课件代码 16-017 · PDF 第 6 页**

```c
typedef void(*pfun_t)(int);
pfun_t signal(int, pfun_t);
```

## 4. 函数指针数组

数组是一个存放相同类型数据的存储空间，我们已经学习了指针数组，比如：

**课件代码 16-018 · PDF 第 6 页**

```c
int * arr[10];
//数组的每个元素是int*
```

那要把函数的地址存到一个数组中，那这个数组就叫函数指针数组，那函数指针的数组如何定义呢？

**课件代码 16-019 · PDF 第 7 页**

```c
int (*parr1[3])();
int *parr2[3]();
int (*)() parr3[3];
```

答案是：parr1 parr1 先和 [] 结合，说明 parr1是数组，数组的内容是什么呢？

是 int (\*)() 类型的函数指针。

## 5. 函数指针数组应用：转移表

函数指针数组的用途：转移表举例：实现一个能够实现整数加法、减法、乘法、除法的计算器

**课件代码 16-020 · PDF 第 7,8 页**

```c
#include <stdio.h>
int add(int a, int b)
{
    return a + b;
}
int sub(int a, int b)
{
    return a - b;
}
int mul(int a, int b)
{
    return a * b;
}
int div(int a, int b)
{
    return a / b;
}
int main()
{
    int x, y;
    int input = 1;
    int ret = 0;
    do
    {
        printf("*************************\n");
        printf("  1:add           2:sub  \n");
        printf("  3:mul           4:div  \n");
        printf("  0:exit                 \n");
        printf("*************************\n");
        printf("请选择：");
        scanf("%d", &input);
        switch (input)
        {
        case 1:
            printf("输入操作数：");
            scanf("%d %d", &x, &y);
            ret = add(x, y);
            printf("ret = %d\n", ret);
            break;
        case 2:
            printf("输入操作数：");
            scanf("%d %d", &x, &y);
            ret = sub(x, y);
            printf("ret = %d\n", ret);
            break;
        case 3:
            printf("输入操作数：");
            scanf("%d %d", &x, &y);
            ret = mul(x, y);
            printf("ret = %d\n", ret);
            break;
        case 4:
            printf("输入操作数：");
            scanf("%d %d", &x, &y);
            ret = div(x, y);
            printf("ret = %d\n", ret);
            break;
        case 0:
            printf("退出程序\n");
            break;
        default:
            printf("选择错误\n");
            break;
        }
    } while (input);

    return 0;
}
```

使用函数指针数组的实现：

**课件代码 16-021 · PDF 第 9,10 页**

```c
#include <stdio.h>
int add(int a, int b)
{
    return a + b;
}
int sub(int a, int b)
{
    return a - b;
}
int mul(int a, int b)
{
    return a*b;
}
int div(int a, int b)
{
    return a / b;
}
int main()
{
    int x, y;
    int input = 1;
    int ret = 0;
    int(*p[5])(int x, int y) = { 0, add, sub, mul, div }; //转移表
    do
    {
        printf("*************************\n");
        printf("  1:add           2:sub  \n");
        printf("  3:mul           4:div  \n");
        printf("  0:exit                 \n");
        printf("*************************\n");
        printf( "请选择：" );
        scanf("%d", &input);
        if ((input <= 4 && input >= 1))
        {
            printf( "输入操作数：" );
            scanf( "%d %d", &x, &y);
            ret = (*p[input])(x, y);
            printf( "ret = %d\n", ret);
        }
        else if(input == 0)
        {
            printf("退出计算器\n");
        }
        else
        {
            printf( "输入有误\n" );
        }
    }while (input);
    return 0;
}
```

## 6. 回调函数

回调函数就是一个通过函数指针调用的函数。

如果你把函数的指针（地址）作为参数传递给另一个函数，当这个指针被用来调用其所指向的函数时，被调用的函数就是回调函数。回调函数不是由该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的，用于对该事件或条件进行响应。

第13讲中我们写的计算机的实现的代码中，红色框中的代码是重复出现的，其中虽然执行计算的逻辑是区别的，但是输入输出操作是冗余的，有没有办法，简化一些呢？

因为红色框中的代码，只有调用函数的逻辑是有差异的，我们可以把调用的函数的地址以参数的形式传递过去，使用函数指针接收，函数指针指向什么函数就调用什么函数，这里其实使用的就是回调函数的功能。

代码块 代码块

**课件代码 16-022 · PDF 第 10,11,12 页**

```c
//使用回调函数改造前
#include <stdio.h>
int add(int a, int b)
{
    return a + b;
}
int sub(int a, int b)
{
    return a - b;
}
int mul(int a, int b)
{
    return a * b;
}
int div(int a, int b)
{
    return a / b;
}
int main()
{
    int x, y;
    int input = 1;
    int ret = 0;
    do
    {
printf("*************************\n");
        printf("  1:add 2:sub  \n");
        printf("  3:mul 4:div  \n");
printf("*************************\n");
        printf("请选择：");
        scanf("%d", &input);
        switch (input)
        {
        case 1:
            printf("输入操作数：");
            scanf("%d %d", &x, &y);
            ret = add(x, y);
            printf("ret = %d\n", ret);
            break;
        case 2:
            printf("输入操作数：");
            scanf("%d %d", &x, &y);
            ret = sub(x, y);
            printf("ret = %d\n", ret);
            break;
        case 3:
            printf("输入操作数：");
            scanf("%d %d", &x, &y);
            ret = mul(x, y);
            printf("ret = %d\n", ret);
            break;
        case 4:
            printf("输入操作数：");
            scanf("%d %d", &x, &y);
            ret = div(x, y);
            printf("ret = %d\n", ret);
            break;
        case 0:
            printf("退出程序\n");
            break;
        default:
            printf("选择错误\n");
            break;
        }
    } while (input);

    return 0;
}
```

**课件代码 16-023 · PDF 第 10,11 页**

```c
//使用回到函数改造后
#include <stdio.h>
int add(int a, int b)
{
    return a + b;
}
int sub(int a, int b)
{
    return a - b;
}
int mul(int a, int b)
{
    return a * b;
}
int div(int a, int b)
{
    return a / b;
}
void calc(int(*pf)(int, int))
{
    int ret = 0;
    int x, y;
    printf("输入操作数：");
    scanf("%d %d", &x, &y);
    ret = pf(x, y);
    printf("ret = %d\n", ret);
}
int main()
{
    int input = 1;
    do
    {
printf("*************************\n");
        printf("  1:add 2:sub  \n");
        printf("  3:mul 4:div  \n");
printf("*************************\n");
        printf("请选择：");
        scanf("%d", &input);
        switch (input)
        {
        case 1:
            calc(add);
            break;
        case 2:
            calc(sub);
            break;
        case 3:
            calc(mul);
            break;
        case 4:
            calc(div);
            break;
        case 0:
            printf("退出程序\n");
            break;
        default:
            printf("选择错误\n");
            break;
        }
    } while (input);

    return 0;
}
```

## 7. qsort

我们在《指针和数组》这一章学习过冒泡排序，当时写的冒泡排序的函数，只能排序整型数据。

**课件代码 16-024 · PDF 第 12 页**

```c
void bubble_sort(int arr[], int sz)
{
    int i = 0;
    for(i = 0; i < sz-1; i++) //确定趟数
    {
        int j = 0;
        for(j = 0; j < sz-i-1; j++)  //一趟排序中的两两相邻元素的比较
        {
            if(arr[j] > arr[j+1])
            {
                int tmp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = tmp;
            }
        }
    }
}
```

如果我们想排序浮点型数组，结构体数组的内容怎么办呢？这个函数就不能胜任了，其实在C语言有一个现成的函数 qsort ，可以排序任意类型的数据，在这个函数就使用函数指针，实现了回调函数。

我们先看一下 qsort 函数的原型：

**课件代码 16-025 · PDF 第 13 页**

```c
void qsort (void* base, size_t num, size_t size, int (*cmp)(const void*, const void*));
//base 是指向待排序数据的起始位置
//num 是待排序数据的元素个数
//size 是待排序数据中一个元素的宽度
//cmp - 是函数指针，指针指向的函数，在排序时，用来比较2个数据的大小关系的比较函数
```

1. 为什么qsort函数的第一个参数是void\*类型的？

因为qsort函数需要能排序任意类型的数据，那么为了让qsort能接受任意数据的地址，就得设计成void\*的指针；void\*的指针，能接受任意类型数据的地址。

2. 为什么需要第2、3个参数？

在qsort函数设计的时候，排序什么类型的数据是未知的，所以base的类型是void\*，我们只知道数据从哪里开始，但是并不知道有多少个数据？每个数据的宽度和类型，所以才有了第2、3个参数

3. 为什么需要cmp这个函数指针？

在排序数据的时候，一定会涉及到比较两个数据的大小，不同类型的数据在比较大小的时候，方法可能是不一样的，比如两个整型可以使用关系运算符直接比较大小，但是两个结构体数据就不能直接比较大小。所以我们得根据不同的数据提供不同的比较方法，这个时候cmp函数指针就出现了。

### 7.1 使用qsort函数排序整型数据

**课件代码 16-026 · PDF 第 13,14 页**

```c
#include <stdio.h>
#include <stdlib.h>

//qosrt函数的使用者得实现一个比较函数
int int_cmp(const void * p1, const void * p2)
{
     return (*( int *)p1 - *(int *) p2);
}

int main()
{
    int arr[] = { 1, 3, 5, 7, 9, 2, 4, 6, 8, 0 };
    int i = 0;

    qsort(arr, sizeof(arr) / sizeof(arr[0]), sizeof (int), int_cmp);
    for (i = 0; i< sizeof(arr) / sizeof(arr[0]); i++)
    {
       printf( "%d ", arr[i]);
    }
    printf("\n");
    return 0;
}
```

### 7.2 使用qsort排序结构数据

**课件代码 16-027 · PDF 第 14,15 页**

```c
#include <stdio.h>
#include <stdlib.h>

struct Stu //学生
{
    char name[20];//名字
    int age;//年龄
};

//假设按照年龄来比较
int cmp_stu_by_age(const void* e1, const void* e2)
{
    return ((struct Stu*)e1)->age - ((struct Stu*)e2)->age;
}

//strcmp - 是库函数，是专门用来比较两个字符串的大小的
//假设按照名字来比较
int cmp_stu_by_name(const void* e1, const void* e2)
{
    return strcmp(((struct Stu*)e1)->name, ((struct Stu*)e2)->name);
}

//按照年龄来排序
void test2()
{
    struct Stu s[] = { {"zhangsan", 20}, {"lisi", 30}, {"wangwu", 15} };
    int sz = sizeof(s) / sizeof(s[0]);
    qsort(s, sz, sizeof(s[0]), cmp_stu_by_age);
}

//按照名字来排序
void test3()
{
    struct Stu s[] = { {"zhangsan", 20}, {"lisi", 30}, {"wangwu", 15} };
    int sz = sizeof(s) / sizeof(s[0]);
    qsort(s, sz, sizeof(s[0]), cmp_stu_by_name);
}
int main()
{
    test2();
    test3();
    return 0;
}
```

📌 AI提示词：使用冒泡排序的思想，实现一个类似于 qsort 函数的通用排序函数

bubble\_sort 。

## 8. qsort函数的模拟实现【加餐】

使用回调函数，模拟实现 qsort ，因为我们还没有学习快速排序的算法思想，这里采用冒泡排序的算法来实现。但是函数原型保持和 qsort 保持一致。

函数原型

**课件代码 16-028 · PDF 第 15 页**

```c
//qsort函数的原型
void qsort (void* base, size_t num, size_t size, int (*cmp)(const void* e1, const void* e2));
//base 是指向待排序数据的起始位置
//num 是待排序数据的元素个数
//size 是待排序数据中一个元素的宽度
//cmp - 是函数指针，指针指向的函数，在排序时，用来比较2个数据的大小关系的比较函数
//cmp函数的指针两个参数e1,e2分别指向了要比较的2个元素的

//既然是模拟实现，bubble_sort也应该如此设计
void bubble_sort(void* base, size_t num, size_t size, int(*cmp)(const void*e1, const void *e2));
```

完整代码：

**课件代码 16-029 · PDF 第 15,16 页**

```c
#include <stdio.h>

int int_cmp(const void * p1, const void * p2)
{
    return (*( int *)p1 - *(int *) p2);
}

void _swap(void *p1, void * p2, int size)
{
    int i = 0;
    for (i = 0; i< size; i++)
    {
        char tmp = *((char *)p1 + i);
       *(( char *)p1 + i) = *((char *) p2 + i);
       *(( char *)p2 + i) = tmp;
    }
}

void bubble_sort(void *base, size_t count , size_t size, int(*cmp )(const void *, const void *))
{
    int i = 0;
    int j = 0;
    for (i = 0; i < count - 1; i++)
    {
       for (j = 0; j < count-i-1; j++)
       {
            if (cmp ((char *) base + j*size , (char *)base + (j + 1)*size) > 0)
           {
               _swap(( char *)base + j*size, (char *)base + (j + 1)*size, size);
           }
       }
    }
}

void test1()
{
    int arr[] = { 4, 3, 5, 7, 9, 2, 1, 6, 8, 0 };
    bubble_sort(arr, sizeof(arr) / sizeof(arr[0]), sizeof (int), int_cmp);

    for (int i = 0; i< sizeof(arr) / sizeof(arr[0]); i++)
    {
       printf( "%d ", arr[i]);
    }
    printf("\n");
}

int main()
{
    test1();
    return 0;
}
```

![第16章第17页配图](/images/c-language-course/16-p17-fig01.png)

## 课件中的参考链接

- [参考资料 1](https://legacy.cplusplus.com/reference/cstdlib/qsort/?kw=qsort)
- [参考资料 2](https://bitejiuyeke.feishu.cn/wiki/LoeowVA3RiUxTokGUegcaIKHnWb)
