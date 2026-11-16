---
title: C语言进阶 15：动态内存、realloc 与柔性数组
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 进阶, 动态内存]
category: C语言
series: C语言学习笔记
seriesOrder: 15
draft: false
updated: 2026-11-16
---

> 本篇按《（新 AI 版）鹏哥 C 语言课件》第 15 章逐节整理。保留原课件代码、对照写法、练习及图表，仅压缩重复说明。
> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- C17 的 malloc(0) 存在实现定义结果，不应一概称为未定义行为。不要访问零大小分配，也不要跨标准依赖 realloc(p, 0) 释放。
- 分配前检查元素数乘 sizeof 是否溢出；realloc 用临时指针接收，成功后旧别名失效。柔性数组标准写法是末尾 `a[]`，`a[0]` 属编译器扩展。

## 本章目标

1. 掌握C语言动态内存管理的知识

2. 能够进行动态内存管理代码的编写

3. 熟悉动态内存管理的常见错误

4. 掌握柔性数组的特点

## 1. 动态内存管理

到目前为止，在我们课程中介绍的，向内存申请空间的两种方式，创建变量和数组。

**课件代码 15-001 · PDF 第 1 页**

```c
int g_val = 100;
int main()
{
    int val = 20;//在栈空间上开辟四个字节
    char arr[10] = {0};//在栈空间上开辟10个字节的连续空间
    return 0;
}
```

这样的方式申请到内存空间，有下面的局限性：

1. 大小是固定的，不能进行灵活调整。

2. 如果是在栈区上申请的空间，出了作用域，内存空间就回收了，不能继续使用。

3. 如果是全局变量只能等程序结束才释放空间。

但是程序对于空间的需求，不仅仅是上述的情况。有时候我们希望空间的大小可以灵活的调整，内存的申请和释放更加自由。

C语言引入了动态内存开辟，让程序员自己可以申请和释放空间，自己来管理内存，就比较灵活了。

C/C++的动态内存管理是一把双刃剑，有好的一面，也有坏的一面，后期慢慢体会。

动态内存管理是在内存的堆区上申请空间的，涉及到的函数有malloc , free , calloc , realloc ,下面我们一一介绍。

## 2. malloc

**课件代码 15-002 · PDF 第 2 页**

```c
#include <stdlib.h> //需要这个头文件

void* malloc (size_t size);
```

功能：向内存的堆区申请一块连续可用的空间，并返回指向这块空间的起始地址。

参数： size ：要分配的内存块的字节数。

返回值：

- 如果开辟成功，则返回这块空间的起始地址。

- 如果开辟失败（如系统内存不足），则返回一个 NULL 指针，因此 malloc 的返回值一定要做检

查。

- 返回值的类型是 void\* ，所以 malloc 函数并不知道开辟空间的类型，具体在使用的时候使用

者自己来决定。

注意事项：如果参数 size 为0， malloc 的行为是标准是未定义的，取决于编译器。

## 3. free

C语言提供了另外一个函数free，函数原型如下：

**课件代码 15-003 · PDF 第 2 页**

```c
void free (void* ptr);
```

功能：释放之前通过动态内存分配函数（如 malloc , calloc , realloc ）申请的内存空间，malloc 和 free 都声明在 stdlib.h 头文件中。

参数：

ptr ：指向要释放的内存块的指针。

- 如果参数 ptr 指向的空间不是动态开辟的，那free函数的行为是未定义的。

- 如果参数 ptr 是NULL指针，则函数什么事都不做。

例子：

数组形式 动态内存管理形式

**课件代码 15-004 · PDF 第 2,3 页**

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int num = 0;
    //数组形式
    scanf("%d", &num);
    int arr[num];

    int i = 0;
    for(i = 0; i < num; i++)
    {
        *(ptr+i) = 0；
    }
    return 0;
}
```

> **原例说明：** 课件数组版误写了未声明的 ptr，并出现全角分号；对应赋值应为 `arr[i] = 0;`。变长数组还要求 num 为合法正值。

**课件代码 15-005 · PDF 第 2,3 页**

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int num = 0;
    scanf("%d", &num);
    int* ptr = (int*)malloc(num*sizeof(int));
    if(NULL != ptr)//判断ptr指针是否为空
    {
        int i = 0;
        for(i = 0; i < num; i++)
        {
            *(ptr+i) = 0；
        }
    }
    else
    {
        printf("malloc 失败\n");
        return 1;
    }
    free(ptr);//释放ptr所指向的动态内存
    ptr = NULL;

    return 0;
}
```

> **原例说明：** 赋值处全角分号 `；` 应改为半角 `;`；真实输入还需验证 num 与分配大小。

上面这个代码中，使用 free 函数释放动态内存，这里只是将 ptr 指向的堆内存还给操作系统，不能再使用。但是并不会把指针 ptr 置为 NULL ，这时一定要手动将 ptr 置为 NULL ，否则 ptr 就变成了悬空指针了。

📌 悬空指针：指针曾经有效，但它指向的内存已经被释放或失效了，指针里还保留着原来的地

址

## 4. calloc

C语言还提供了一个函数叫 calloc ， calloc 函数也用来动态内存分配。原型如下：

**课件代码 15-006 · PDF 第 3 页**

```c
void* calloc (size_t num, size_t size);
```

- 函数的功能是为 num 个大小为 size 的元素开辟一块空间，并且把空间的每个字节初始化为 0。

- 与函数 malloc 的区别只在于 calloc 会在返回地址之前把申请的空间的每个字节初始化为全

0。

例子：

**课件代码 15-007 · PDF 第 4 页**

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *p = (int*)calloc(10, sizeof(int));
    if(NULL != p)
    {
         int i = 0;
         for(i = 0; i < 10; i++)
         {
             printf("%d ", *(p+i));
         }
    }
    free(p);
    p = NULL;
    return 0;
}
```

输出结果：

**课件代码 15-008 · PDF 第 4 页**

```text
0 0 0 0 0 0 0 0 0 0
```

所以如果我们对申请的内存空间的内容要求初始化为 0，那么可以很方便的使用 calloc 函数来完成任务。

## 5. realloc

- realloc 函数的出现让动态内存管理更加灵活。

- 有时候我们发现过去申请的空间太小了，有时候我们又会觉得申请的空间过大了，那为了合理的使

用内存，我们一定会对内存的大小做灵活的调整。那 realloc 函数就可以做到对动态开辟内存大小的调整。

函数原型如下：

**课件代码 15-009 · PDF 第 5 页**

```c
void* realloc (void* ptr, size_t size);
```

功能：重新调整之前分配的内存块的大小，它可以在不丢失原有数据的情况下，扩大或缩小动态分配的内存块。

参数：

- ptr 是要调整的内存空间的起始地址，如果 ptr 是 NULL 指针， realloc 函数的功能类似于

malloc 函数。

- size 调整之后新大小，单位是字节。

返回值：

- 成功：返回一个指向重新分配的内存块的 void \* 类型指针。这个指针可能与原来的指针不同。

- 失败：如果内存重新分配失败，返回 NULL ，并且原来的内存块保持不变。

注意事项： realloc 在调整内存空间大小的时候，存在两种情况：

- 情况1：原有空间之后有足够大的空间，要扩展的内存就直接原来内存之后直接追加空间，原来空

间的数据不发生变化，最终返回的地址还是旧的地址。

- 情况2：原有空间之后没有足够大的空间，会在内存的堆区寻找新的满足要求的空间，返回新的起

始地址。在这个过程中会发生以下几件事：

- 寻找新的满足要求的空间

- 将旧空间的数据拷贝到新空间，保证数据不会丢失

- 释放旧的空间，返回新空间的起始地址

**课件代码 15-010 · PDF 第 5,6 页**

```c
int main()
{
    int* ptr = (int*)malloc(20);
    //...

    //扩容空间
    if(ptr != NULL)
    {
        int * tmp = (int*)realloc(ptr, 40);
        //...
    }
    return 0;
}
```

![第15章第6页配图](/images/c-language-course/15-p06-fig01.png)

由于上述的两种情况， realloc 函数的使用就要注意一些。

**课件代码 15-011 · PDF 第 6,7 页**

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *ptr = (int*)malloc(100);
    if(ptr != NULL)
    {
         //业务处理
    }
    else
    {
         return 1;
    }
    //扩展容量
    //代码1 - 直接将realloc的返回值放到ptr中
    ptr = (int*)realloc(ptr, 1000);//这样可以吗？(如果申请失败会如何？)
    //代码2 - 先将realloc函数的返回值放在p中，不为NULL，再放ptr中
    int* p = realloc(ptr, 1000);
    if(p != NULL)
    {
        ptr = p;
        p = NULL;
    }
    //业务处理
    //....
    free(ptr);
    ptr = NULL;

    return 0;
}
```

## 6. 动态内存管理常见错误

## 1. 对NULL指针的解引用操作

没有对 malloc 、 calloc 、 realloc 函数的返回值做判断，直接使用，就可能导致对NULL指针的解引用操作。

**课件代码 15-012 · PDF 第 7 页**

```c
 void test()
 {
     int *p = (int *)malloc(INT_MAX/4);
     *p = 20;//如果p的值是NULL，就会有问题
     free(p);
     p = NULL;
 }
```

## 2. 对动态开辟空间的越界访问

**课件代码 15-013 · PDF 第 7,8 页**

```c
void test()
{
    int i = 0;
    int *p = (int *)malloc(10*sizeof(int));
    if(NULL == p)
    {
        return 1;
    }
    for(i = 0; i <= 10; i++)
    {
        *(p+i) = i;//当i是10的时候越界访问
    }
    free(p);
    p = NULL;
}
```

## 3. 对非动态开辟内存使用free释放

**课件代码 15-014 · PDF 第 8 页**

```c
 void test()
 {
     int a = 10;
     int *p = &a;
     free(p);//ok?
 }
```

## 4. 使用free释放一块动态开辟内存的一部分

**课件代码 15-015 · PDF 第 8 页**

```c
 void test()
 {
     int *p = (int *)malloc(100);
     p++;
     free(p);//p不再指向动态内存的起始位置
 }
```

## 5. 对同一块动态内存多次释放

**课件代码 15-016 · PDF 第 8 页**

```c
 void test()
 {
     int *p = (int *)malloc(100);
     free(p);
     free(p);//重复释放
 }
```

## 6. 动态开辟内存忘记释放（内存泄漏）

**课件代码 15-017 · PDF 第 8,9 页**

```c
 void test()
 {
     int *p = (int *)malloc(100);
     if(NULL != p)
     {
         *p = 20;
     }
 }

 int main()
 {
     test();
     while(1);
     return 0;
 }
```

内存泄漏（Memory Leak）是指在 C 语言程序中，动态分配的内存（通过 malloc 、 calloc 、realloc 等函数获取）在使用完毕后，没有被正确释放（即没有调用 free ），导致这块内存始终被程序占用而无法再被使用，直到程序运行结束。

为什么会发生内存泄漏？

- 忘记调用 free ：分配内存后，后续代码路径没有释放它。

- 提前返回或异常退出：在分配与释放之间通过 return 、 exit 或 goto 跳过了释放语句。

- 丢失指针：将指向已分配内存的指针重新赋值或覆盖，导致原内存块的地址丢失，再也无法释放。

内存泄漏的后果：

- 资源浪费：泄漏的内存在程序生命周期内无法被回收，可用内存逐渐减少。

- 性能下降：过多内存泄漏会导致系统频繁进行内存换页、分配更大空间，程序变慢。

- 程序崩溃：当累积泄漏内存耗尽进程地址空间或系统内存时，后续 malloc 失败，程序可能异常

终止（尤其长时间运行的服务或嵌入式系统）。

- 难以调试：内存泄漏通常在运行很久后才暴露，定位困难。

## 7. 常见笔试题解析

题目1：请问运行程序，会有什么样的结果？

**课件代码 15-018 · PDF 第 9,10 页**

```c
#include <stdio.h>
#include <stdlib.h>

void GetMemory(char *p)
{
    p = (char *)malloc(100);
}

void Test(void)
{
    char *str = NULL;
    GetMemory(str);
    strcpy(str, "hello world");
    printf(str);
}

int main()
{
    Test();
    return 0;
}
```

题目2：请问运行程序，会有什么样的结果？

**课件代码 15-019 · PDF 第 10 页**

```c
#include <stdio.h>
#include <stdlib.h>

char *GetMemory(void)
{
    char p[] = "hello world";
    return p;
}

void Test(void)
{
    char *str = NULL;
    str = GetMemory();
    printf(str);
}

int main()
{
    Test();
    return 0;
}
```

题目3：请问运行程序，会有什么样的结果？

**课件代码 15-020 · PDF 第 11 页**

```c
#include <stdio.h>
#include <stdlib.h>

void GetMemory(char **p, int num)
{
    *p = (char *)malloc(num);
}
void Test(void)
{
    char *str = NULL;
    GetMemory(&str, 100);
    strcpy(str, "hello");
    printf(str);
}

int main()
{
    Test();
    return 0;
}
```

题目4：请问运行程序，会有什么样的结果？

**课件代码 15-021 · PDF 第 11,12 页**

```c
#include <stdio.h>
#include <stdlib.h>

void Test(void)
{
    char *str = (char *) malloc(100);
    strcpy(str, "hello");
    free(str);
    if(str != NULL)
    {
        strcpy(str, "world");
        printf(str);
    }
}

int main()
{
    Test();
    return 0;
}
```

## 8. 柔性数组

也许你从来没有听说过柔性数组（flexible array）这个概念，但是它确实是存在的。

C99 中，结构中的最后一个元素允许是未知大小的数组，这就叫做『柔性数组』成员。

例如：

**课件代码 15-022 · PDF 第 12 页**

```c
struct st_type
{
    int i;
    int a[0];//柔性数组成员
};
```

有些编译器会报错无法编译可以改成：

**课件代码 15-023 · PDF 第 12 页**

```c
struct st_type
{
    int i;
    int a[];//柔性数组成员
};
```

### 8.1 柔性数组的特点

- 结构中的柔性数组成员前面必须至少一个其他成员。

- sizeof 返回的这种结构大小不包括柔性数组的内存。

- 包含柔性数组成员的结构用 malloc() 函数进行内存的动态分配，并且分配的内存应该大于结构

的大小，以适应柔性数组的预期大小。

例如：

**课件代码 15-024 · PDF 第 12,13 页**

```c
struct st_type
{
    int i;
    int a[0];//柔性数组成员
};

int main()
{
    printf("%d\n", sizeof(struct st_type));//输出的是4
    return 0;
}
```

### 8.2 柔性数组的使用

**课件代码 15-025 · PDF 第 13 页**

```c
//代码1
#include <stdio.h>
#include <stdlib.h>

struct st_type
{
    int i;
    int a[0];//柔性数组成员
};

int main()
{
    int i = 0;
    struct st_type *p = (struct st_type*)malloc(sizeof(struct st_type) + 100*sizeof(int));
    if(p == NULL)
    {
        perror("malloc");
        return 1;
    }
    //业务处理
    p->i = 100;
    for(i = 0; i < 100; i++)
    {
        p->a[i] = i;
    }
    free(p);
    p = NULL;

    return 0;
}
```

这样柔性数组成员a，相当于获得了100个整型元素的连续空间，而且a数组的大小是可以动态调整的。

### 8.3 柔性数组的优势

上述的 type\_a 结构也可以设计为下面的结构，也能完成同样的效果。

**课件代码 15-026 · PDF 第 14,15 页**

```c
//代码2
#include <stdio.h>
#include <stdlib.h>

struct st_type
{
    int i;
    int *p_a;
};

int main()
{
    struct st_type *p = (struct st_type *)malloc(sizeof(struct st_type));
    if(p == NULL)
    {
        perror("malloc");
        return 1;
    }
    p->i = 100;
    int* ptr = (int *)malloc(p->i*sizeof(int));
    if(ptr == NULL)
    {
        perror("malloc 2");
        return 2;
    }
    p->p_a = ptr;
    ptr = NULL;

    //业务处理
    for(i=0; i<100; i++)
    {
        p->p_a[i] = i;
    }

    //释放空间
    free(p->p_a);
    p->p_a = NULL;
    free(p);
    p = NULL;
    return 0;
}
```

> **原例说明：** 课件循环使用了未声明的 i，应声明为 `int i = 0` 等合适类型；第二次分配失败时还需释放第一块已分配对象。

上述 代码1 和 代码2 可以完成同样的功能，但是 方法1 的实现有两个好处：

第一个好处是：方便内存释放如果我们的代码是在一个给别人用的函数中，你在里面做了二次内存分配，并把整个结构体返回给用戶。用戶调用free可以释放结构体，但是用戶并不知道这个结构体内的成员也需要free，所以你不能指望用戶来发现这个事。所以，如果我们把结构体的内存以及其成员要的内存一次性分配好了，并返回给用戶一个结构体指针，用戶做一次free就可以把所有的内存也给释放掉。

第二个好处是：这样有利于访问速度.连续的内存有益于提高访问速度，也有益于减少内存碎片。（其实，我个人觉得也没多高了，反正你跑不了要用做偏移量的加法来寻址）扩展阅读：

C语言结构体里的数组和指针

## 课件中的参考链接

- [参考资料 1](https://legacy.cplusplus.com/reference/cstdlib/malloc/?kw=malloc)
- [参考资料 2](https://legacy.cplusplus.com/reference/cstdlib/free/?kw=free)
- [参考资料 3](https://legacy.cplusplus.com/reference/cstdlib/calloc/?kw=calloc)
- [参考资料 4](https://legacy.cplusplus.com/reference/cstdlib/realloc/?kw=realloc)
- [参考资料 5](https://coolshell.cn/articles/11377.html)
