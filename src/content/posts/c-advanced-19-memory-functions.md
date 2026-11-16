---
title: C语言进阶 19：memcpy、memmove、memset 与 memcmp
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 进阶, 内存函数]
category: C语言
series: C语言学习笔记
seriesOrder: 19
draft: false
updated: 2026-11-16
---

> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- memcpy 不允许区域重叠；memmove 允许。长度均为字节，不是元素数。教材通过无关指针大小比较选择拷贝方向的实现并不普遍可移植，实际使用标准库 memmove。
- memset 按字节填充值，不是按元素赋值；memcmp 比较对象表示，不等于整数数值比较，也不适合直接判断含填充结构体的逻辑相等。

## 本章目标

1. 掌握C语言中常见的内存函数

2. 有能力模拟实现memcpy、memmove函数

## 1. memcpy

**课件代码 19-001 · PDF 第 1 页**

```c
void * memcpy ( void * destination, const void * source, size_t num );
```

功能：

- memcpy 是完成内存块拷贝的，不关注内存中存放的数据是啥

- 函数 memcpy 从 source 的位置开始向后复制 num 个字节的数据到 destination 指向的内存

位置。

- 如果 source 和 destination 有任何的重叠，复制的结果都是未定义的。

内存重叠的情况，需要使用 memmove 函数参数：

- destination ：指针，指向目标空间，拷贝的数据存放在这里

- source ：指针，指向源空间，要拷贝的数据从这里来

- num ：要拷贝的数据占据的字节数

返回值：拷贝完成后，返回目标空间的起始地址头文件： &lt;string.h&gt;

### 1.1 代码演示

**课件代码 19-002 · PDF 第 1,2 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    int arr1[] = { 1,2,3,4,5,6,7,8,9,10 };
    int arr2[10] = { 0 };
    memcpy(arr2, arr1, 20);
    int i = 0;
    for (i = 0; i < 10; i++)
    {
        printf("%d ", arr2[i]);
    }
    return 0;
}
```

### 1.2 模拟实现

**课件代码 19-003 · PDF 第 2 页**

```c
void * memcpy ( void * dst, const void * src, size_t count)
{
    void * ret = dst;
    assert(dst);
    assert(src);
    /*
     * copy from lower addresses to higher addresses
     */
    while (count--) {
            *(char *)dst = *(char *)src;
            dst = (char *)dst + 1;
            src = (char *)src + 1;
    }

    return(ret);
}
```

## 2. memmove

**课件代码 19-004 · PDF 第 2 页**

```c
void * memmove ( void * destination, const void * source, size_t num );
```

功能：

- memmove函数也是完成内存块拷贝的

- 和memcpy的差别就是memmove函数处理的源内存块和目标内存块是可以重叠的。

- memmove的使用需要包含&lt;string.h&gt;

参数：

destination ：指针，指向目标空间，拷贝的数据存放在这里source ：指针，指向源空间，要拷贝的数据从这里来num ：要拷贝的数据占据的字节数返回值：

拷贝完成后，返回目标空间的起始地址

### 2.1 代码演示

**课件代码 19-005 · PDF 第 3 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    int arr1[] = { 1,2,3,4,5,6,7,8,9,10 };
    memmove(arr1 + 2, arr1, 20);
    int i = 0;
    for (i = 0; i < 10; i++)
    {
        printf("%d ", arr1[i]);
    }
    return 0;
}
```

输出的结果：

**课件代码 19-006 · PDF 第 3 页**

```text
      1 2 1 2 3 4 5 8 9 10
```

### 2.2 模拟实现

**课件代码 19-007 · PDF 第 4 页**

```c
void * memmove ( void * dst, const void * src, size_t count)
{
    void * ret = dst;
    if (dst <= src || (char *)dst >= ((char *)src + count)) {
        /*
         * Non-Overlapping Buffers
         * copy from lower addresses to higher addresses
         */
        while (count--) {
                *(char *)dst = *(char *)src;
                dst = (char *)dst + 1;
                src = (char *)src + 1;
        }
    }
    else {
        /*
         * Overlapping Buffers
         * copy from higher addresses to lower addresses
         */
        dst = (char *)dst + count - 1;
        src = (char *)src + count - 1;

        while (count--) {
                *(char *)dst = *(char *)src;
                dst = (char *)dst - 1;
                src = (char *)src - 1;
        }
    }

    return(ret);
}
```

📌 AI提示词：对比一下memcpy和memmove这两个函数，实际开发中建议使用哪个？怎么抉

择？

## 3. memset

**课件代码 19-008 · PDF 第 4 页**

```c
void * memset ( void * ptr, int value, size_t num );
```

功能：

- memset 函数是用来设置内存块的内容的，将内存中指定长度的空间设置为特定的内容。

- memset 的使用需要包含 &lt;string.h&gt;

参数：

ptr ：指针，指向要设置的内存空间，也就是存放了要设置的内存空间的起始地址。

value ：要设置的值，函数将会把 value 值转换成 unsigned char 的数据进行设置的。 也就是以字节为单位来设置内存块的。

num ：要设置的内存长度，单位是字节。

返回值：返回的是要设置的内存空间的起始地址。

### 3.1 代码演示

**课件代码 19-009 · PDF 第 5 页**

```c
#include <stdio.h>
#include <string.h>
int main ()
{
    char str[] = "hello world";
    memset (str, 'x', 6);
    printf(str);
    return 0;
}
```

输出的结果：

**课件代码 19-010 · PDF 第 5 页**

```text
xxxxxxworld
```

### 3.2 总结：

当有一块内存空间需要设置内容的时候，就可以使用memset函数，值得注意的是memset函数对内存单元的设置是以字节为单位的。

## 4. memcmp

**课件代码 19-011 · PDF 第 6 页**

```c
int memcmp ( const void * ptr1, const void * ptr2, size_t num );
```

功能：

比较指定的两块内存块的内容，比较从ptr1和ptr2指针指向的位置开始，向后的num个字节memcmp 的使用需要包含 &lt;string.h&gt;参数：

ptr1 ：指针，指向一块待比较的内存块ptr2 ：指针，指向另外一块待比较的内存块num ：指定的比较长度，单位是字节返回值：

![第19章第6页配图](/images/c-language-course/19-p06-fig01.png)

### 4.1 代码演示

**课件代码 19-012 · PDF 第 6,7 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
        char buffer1[] = "DWgaOtP12df0";
        char buffer2[] = "DWGAOTP12DF0";
        int n;
        n = memcmp(buffer1, buffer2, sizeof(buffer1));

        if (n > 0)
            printf("'%s' 大于 '%s'.\n", buffer1, buffer2);
        else if (n < 0)
            printf("'%s' 小于 '%s'.\n", buffer1, buffer2);
        else
            printf("'%s' 和 '%s' 一样.\n", buffer1, buffer2);

        return 0;
}
```

### 4.2 总结

如果要比较2块内存空间的数据的大小，可以使用 memcmp 函数，这个函数的特点就是可以指定比较长度。

memcmp 函数是通过返回值告知大小关系的。

## 课件中的参考链接

- [参考资料 1](https://cplusplus.com/reference/cstring/memcpy/?kw=memcpy)
- [参考资料 2](https://legacy.cplusplus.com/reference/cstring/memmove/)
- [参考资料 3](https://legacy.cplusplus.com/reference/cstring/memset/?kw=memset)
- [参考资料 4](https://cplusplus.com/reference/cstring/memcmp/?kw=memcmp)
