---
title: C语言进阶 18：字符与字符串函数的完整使用规则
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 进阶, 字符串, 标准库]
category: C语言
series: C语言学习笔记
seriesOrder: 18
draft: false
updated: 2026-11-16
---

> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- 小写转大写应使用 toupper，tolower 是转小写。ctype 参数必须为 EOF 或可表示为 unsigned char 的值，处理 char 字节时先转 unsigned char。
- strlen 的 size_t 输出用 `%zu`；比较长度直接用大小关系，避免无符号相减。strncpy 不保证终止，strncat 的 n 是最多追加字符数，不是目标总容量。
- snprintf 只有在传入真实容量且无非法重叠时才能限制写入；必须检查负返回和截断。sscanf 失败可能返回 0，不是全部失败都返回 EOF。

## 本章目标

1. 掌握常见的字符分类函数和转换函数

2. 熟练使用常见的字符串相关函数，能模拟实现重点函数

3. 熟悉相关函数的差异和使用建议

4. 掌握在软件开发中库函数的使用建议

在编程的过程中，我们经常要处理字符和字符串，为了方便操作字符和字符串，C语言标准库中提供了一系列库函数，接下来我们就学习一下这些函数，本章节，我们会介绍28个库函数。

有关字符的函数是声明在 ctype.h 文件中的，和字符串相关的函数声明在 string.h 头文件中的。

## 1. 字符函数

### 1.1 字符分类函数

C语言中有一系列的函数是专门做字符分类的，也就是判断一个字符是属于什么类型的字符的。

![第18章第1页配图](/images/c-language-course/18-p01-fig01.png)

这些函数的使用方法非常类似，我们就讲解一个函数的事情，其他的非常类似：

**课件代码 18-001 · PDF 第 2 页**

```c
int islower ( int c );
```

islower 是能够判断参数部分的 c 是否是小写字母的。

通过返回值来说明是否是小写字母，如果是小写字母就返回非0的整数，如果不是小写字母，则返回0。

练习：

写一个代码，将字符串中的小写字母转大写，其他字符不变。

**课件代码 18-002 · PDF 第 2 页**

```c
#include <stdio.h>
#include <ctype.h>
int main ()
{
    char str[] = "Test String.\n";
    char c = 0;
    int i = 0;
    while (str[i])
    {
        c = str[i];
        if (islower(c))
            c -= 32;
        putchar(c);
        i++;
    }
    return 0;
}
```

### 1.2 字符转换函数

C语言提供了2个字符转换函数：

**课件代码 18-003 · PDF 第 2 页**

```c
int tolower ( int c ); //将参数传进去的大写字母转小写
int toupper ( int c ); //将参数传进去的小写字母转大写
```

上面的代码，我们将小写转大写，是-32完成的效果，有了转换函数，就可以直接使用 tolower 函数。

**课件代码 18-004 · PDF 第 3 页**

```c
#include <stdio.h>
#include <ctype.h>

int main ()
{
    int i = 0;
    char str[] = "Test String.\n";
    char c;
    while (str[i])
    {
        c = str[i];
        if (islower(c))
            c = toupper(c);
        putchar(c);
        i++;
    }
    return 0;
}
```

## 2. strlen

**课件代码 18-005 · PDF 第 3 页**

```c
size_t strlen ( const char * str );
```

功能：统计参数 str 指向的字符串的长度。统计的是字符串中 '\\0' 之前的字符的个数。

参数：

- str ：指针，指向了要统计长度的字符串。

返回值：返回了 str 指向的字符串的长度，返回的长度不会是负数，所以返回类型是 size\_t 。

头文件： &lt;string.h&gt;

### 2.1 代码演示

**课件代码 18-006 · PDF 第 3,4 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    const char* str = "abcdef";
    printf("%zd\n", strlen(str));
    return 0;
}
```

使用注意事项：

- 字符串以 '\\0' 作为结束标志， strlen 函数返回的是在字符串中 '\\0' 前面出现的字符个数

（不包含 '\\0' )。

- 参数指向的字符串必须要以 '\\0' 结束，否则结果是未知数。

- 注意函数的返回值为 size\_t ，是无符号的（ 易错 ）

### 2.2 strlen返回值

**课件代码 18-007 · PDF 第 4 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    const char* str1 = "abcdef";
    const char* str2 = "bbb";
    if(strlen(str2) - strlen(str1) > 0)
    {
        printf("str2 > str1\n");
    }
    else
    {
        printf("srt1 > str2\n");
    }
    return 0;
}
```

### 2.3 strlen的模拟实现

之前我们讲解过了两种实现方式了，这里再给大家讲解第三种实现，递归的方式。

一般别人出题不会直接说让你使用递归，而是这样说：

模拟实现strlen，不能创建临时变量，来求字符串的长度

**课件代码 18-008 · PDF 第 4,5 页**

```c
size_t my_strlen(const char * str)
{
    assert(str); //思考：这里需要断言吗？
    if(*str == '\0')
        return 0;
    else
        return 1 + my_strlen(str+1);
}
```

## 3. strcpy

**课件代码 18-009 · PDF 第 5 页**

```c
char* strcpy(char * destination, const char * source );
```

功能：字符串拷贝，将 source 指向的字符串，拷贝到 destination 指向的目标空间中。

参数：

- destination ：指针，指向目的地空间

- source ：指针，指向源头数据

返回值： strcpy 函数返回的目标空间的起始地址头文件： &lt;string.h&gt;

### 3.1 代码演示

**课件代码 18-010 · PDF 第 5 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char arr1[10] = {0};
    char arr2[] = "hello";
    strcpy(arr1, arr2);
    printf("%s\n", arr1);

    return 0;
}
```

使用注意事项：

- 源字符串必须以 '\\0' 结束，并且会将源字符串中的 '\\0' 拷贝到目标空间。

- 目标空间必须足够大，以确保能存放源字符串。

- 目标空间必须可修改。

### 3.2 模拟实现

**课件代码 18-011 · PDF 第 6 页**

```c
#include <stdio.h>
#include <assert.h>

char* my_strcpy(char *dest, const char*src)
{
    char *ret = dest;
    assert(dest != NULL); //思考：这里需要断言吗？
    assert(src != NULL);

    while((*dest++ = *src++))
    {
        ;
    }
    return ret;
}

int main()
{
    char arr1[10] = {0};
    char arr2[] = "hello";
    my_strcpy(arr1, arr2);
    printf("%s\n", arr1);

    return 0;
}
```

## 4. strncpy

**课件代码 18-012 · PDF 第 6 页**

```c
char * strncpy ( char * destination, const char * source, size_t num );
```

功能：字符串拷贝；将 source 指向的字符串拷贝到 destination 指向的空间中，最多拷贝 num个字符。

参数：

- destination ：指针，指向目的地空间

- source ：指针，指向源头数据

- num ：从 source 指向的字符串中最多拷贝的字符个数

返回值： strncpy 函数返回的目标空间的起始地址

### 4.1 代码演示

**课件代码 18-013 · PDF 第 7 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char arr1[20] = "----------";
    char arr2[] = "abcdefghi";
    char* str = strncpy(arr1, arr2, 5);
    printf("%s\n", arr1);
    printf("%s\n", str);

    return 0;
}
```

### 4.2 比较strcpy和strncpy函数

- strcpy 函数把源字符串数据拷贝到 \\0 为止，如果目标空间不够的话，容易出现越界行为。

- strncpy 函数指定了拷贝的长度，源字符串不一定要有 \\0 ，当源字符串长度 ≥ n 时，目标空

间不会以 \\0 结尾，后续如果直接当作字符串使用，会存在越界的可能。所以有时候需要手动补上 \\0 。

📌 AI提示词：模拟实现一下C语言的库函数：strncpy，加上必要的注释

## 5. strcat

**课件代码 18-014 · PDF 第 7 页**

```c
char * strcat ( char * destination, const char * source );
```

功能：字符串追加，把 source 指向的源字符串中的所有字符都追加到 destination 指向的空间中。

参数：

- destination ：指针，指向目的地空间

- source ：指针，指向源头数据

返回值： strcat 函数返回的目标空间的起始地址头文件：&lt;string.h&gt;

### 5.1 代码演示

**课件代码 18-015 · PDF 第 8 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char arr1[20] = "hello ";
    char arr2[] = "world";
    strcat(arr1, arr2);
    printf("%s\n", arr1);
    return 0;
}
```

使用注意事项：

- 源字符串必须以 '\\0' 结束。

- 目标字符串中也得有 \\0 ，在目标字符串的 \\0 位置开始追加源字符串内容。

- 目标空间必须有足够的大，能容纳下源字符串的内容。

- 目标空间必须可修改。

### 5.2 模拟实现

**课件代码 18-016 · PDF 第 8,9 页**

```c
#include <stdio.h>
#include <assert.h>

char* my_strcat(char *dest, const char*src)
{
    char *ret = dest;
    assert(dest != NULL);
    assert(src != NULL);

    while(*dest)
    {
        dest++;
    }
    while((*dest++ = *src++))
    {
        ;
    }
    return ret;
}

int main()
{
    char arr1[20] = "hello ";
    char arr2[] = "world";
    my_strcat(arr1, arr2);
    printf("%s\n", arr1);
    return 0;
}
```

## 6. strncat

**课件代码 18-017 · PDF 第 9 页**

```c
char * strncat ( char * destination, const char * source, size_t num );
```

功能：字符串追加；将 source 指向的字符串的内容，追加到 destination 指向的空间，最多追加 num 个字符。

参数：

- destination ：指针，指向了目标空间

- source ：指针，指向了源头数据

- num ：表示最多追加的字符的个数， num 的值要 &lt; 目标空间中剩余的长度

返回值：返回的是目标空间的起始地址

### 6.1 代码演示

**课件代码 18-018 · PDF 第 10 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char arr1[20] = "hello ";
    char arr2[] = "world";
    char* str = strncat(arr1, arr2, 5);
    printf("%s\n", arr1);
    printf("%s\n", str);

    return 0;
}
```

### 6.2 strcat和strncat对比

- 参数不同， strncat 多了一个参数

- strcat 函数在追加的时候要将源字符串的所有内容，包含 \\0 都追加过去，但是 strncat 函

数指定了追加的长度。

- strncat 函数中源字符串中不一定要有 \\0 了，如果 num &lt; 源字符串的长度，则遇不到 \\0

的。

- strncat 函数不管追加几个字符，都会再加上 \\0 结尾。

- strncat 更加灵活，也更加安全。

📌 AI提示词：模拟实现一下C语言的库函数：strncat，加上必要的注释

## 7. strcmp

**课件代码 18-019 · PDF 第 10 页**

```c
int strcmp ( const char * str1, const char * str2 );
```

功能：用来比较 str1 和 str2 指向的字符串，从两个字符串的第一个字符开始比较，如果两个字符的ASCII码值相等，就比较下一个字符。直到遇到不相等的两个字符，或者字符串结束。

参数：

- str1 ：指针，指向要比较的第一个字符串

- str2 ：指针，指向要比较的第二个字符串

返回值：标准规定：

- 第一个字符串大于第二个字符串，则返回大于0的数字

- 第一个字符串等于第二个字符串，则返回0

- 第一个字符串小于第二个字符串，则返回小于0的数字

### 7.1 代码演示：

**课件代码 18-020 · PDF 第 11 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char arr1[] = "abcdef";
    char arr2[] = "abq";
    int ret = strcmp(arr1, arr2);
    printf("%d\n", ret);

    if(ret > 0)
        printf("arr1 > arr2\n");
    else if(ret == 0)
        printf("arr1 == arr2\n");
    else
        printf("arr1 < arr2\n");

    return 0;
}
```

### 7.2 模拟实现：

**课件代码 18-021 · PDF 第 11,12 页**

```c
int my_strcmp (const char * str1, const char * str2)
{
    int ret = 0 ;
    assert(str1 != NULL);
    assert(str2 != NULL);
    while(*str1 == *str2)
    {
        if(*str1 == '\0')
            return 0;
        str1++;
        str2++;
    }
    return (unsigned char)*str1 - (unsigned char)*str2;
}
```

## 8. strncmp

**课件代码 18-022 · PDF 第 12 页**

```c
int strncmp ( const char * str1, const char * str2, size_t num );
```

功能：字符串比较；比较 str1 和 str2 指向的两个字符串的内容，最多比较 num 字符。

参数：

- str1 ：指针，指向一个比较的字符串

- str2 ：指针，指向另外一个比较的字符串

- num ：最多比较的字符个数

返回值：

- 标准规定：

- 第一个字符串大于第二个字符串，则返回大于0的数字

- 第一个字符串等于第二个字符串，则返回0

- 第一个字符串小于第二个字符串，则返回小于0的数字

![第18章第12页配图](/images/c-language-course/18-p12-fig01.png)

### 8.1 代码演示

**课件代码 18-023 · PDF 第 12,13 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char arr1[] = "abcdef";
    char arr2[] = "abcqw";

    int ret1 = strncmp(arr1, arr2, 3);
    printf("%d\n", ret1);
    int ret2 = strncmp(arr1, arr2, 4);
    printf("%d\n", ret2);

    return 0;
}
```

### 8.2 strcmp和strncmp比较

- 参数不同

- strncmp 可以比较任意长度了

- strncmp 函数更加灵活，更加安全，推荐使用

📌 AI提示词：模拟实现一下C语言的库函数：strncmp，加上必要的注释

## 9. 软件开发中的建议

前面学过的这些函数，其实可以分为两组：

![第18章第13页图表](/images/c-language-course/18-p13-table02.png)

这两组函数这么相似，在实际软件开发中建议使用哪一组呢？

📌 AI提示词：

strcpy、strcat、strcmp和strncpy、strncat、strncmp 两组函数，在软件开发过程中建议怎么使用？

![第18章第14页配图](/images/c-language-course/18-p14-fig01.png)

CWE-676 就是使用潜在危险函数使用潜在危险的函数，可能会导致缓冲区溢出的风险，攻击者可通过精确控制覆盖内容，将返回地址改写为恶意代码的地址，从而劫持程序执行流。

一个缓冲区溢出的代码

**课件代码 18-024 · PDF 第 14 页**

```c
#include <stdio.h>

int main()
{
    char arr1[5] = {0};
    char arr2[10] = "abcdef";
    printf("%d\n", arr2 - &arr1[4]);
    strcpy(arr1, "--------------------------------------");
    //拷贝的数据足够长，就会覆盖到arr2数组的内容
    printf("%s\n", arr2);

    return 0;
}
```

snprintf 又是什么函数呢？

它怎么就能替代 strncpy 和 strncat 函数了？这得先从 sprintf 和 sscanf 函数讲起。

**课件代码 18-025 · PDF 第 14,15 页**

```c
int printf ( const char * format, ... );
int sprintf ( char * str, const char * format, ... );
int snprintf ( char * s, size_t n, const char * format, ... );

int scanf ( const char * format, ... );
int sscanf ( const char * s, const char * format, ...);
```

## 10. sprintf 和 snprintf

### 10.1 sprintf

**课件代码 18-026 · PDF 第 15 页**

```c
int sprintf ( char * str, const char * format, ... );
```

功能：将格式化数据写入字符数组（字符串）。它类似于 printf ，但输出目标不是控制台或文件，而是用戶指定的内存缓冲区。常用于动态生成字符串、拼接数据或转换数据格式。简而言之就是将格式化的数据转换成一个字符串。

参数：

- str ：指向字符数组的指针，用于存储生成的字符串（需确保足够大以防止溢出）。

- format ：格式化字符串，定义输出格式（如 %d 、 %f 、 %s 等）。

- ... ：可变参数列表，提供与格式字符串中说明符对应的数据。

返回值：

- 成功时：返回写入 buffer 的字符数（不包括结尾的空字符 \\0 ）。

- 失败时：返回负值。

代码演示：

**课件代码 18-027 · PDF 第 15,16 页**

```c
#include <stdio.h>

struct Stu
{
    char name[30];
    int age;
    float score;
};

int main()
{
    struct Stu s = {"zhangsan", 100, 85.5f};
    printf("%s %d %.1f\n", s.name, s.age, s.score); //打印到屏幕上了

    //将s中的结构数据转换成一个字符串
    char arr[100] = {0};
    sprintf(arr, "%s %d %.1f", s.name, s.age, s.score);
    printf("%s\n", arr);
    return 0;
}
```

### 10.2 snprintf

**课件代码 18-028 · PDF 第 16 页**

```c
int snprintf ( char * s, size_t n, const char * format, ... );
```

功能：将可变参数按照 format 格式化，并将结果字符串安全地写入到 s 指向的缓冲区中。与sprintf 的最大区别是：它强制受 n 限制，绝不会写入超过缓冲区实际大小的内容。

参数：

- s : 目标缓冲区指针，用于存放结果字符串。

- n : 缓冲区 s 的总字节数。函数最多写入 n-1 个字符，并保证以 '\\0' 结尾（只要 size

&gt; 0 ）。

- format : 格式化控制字符串，与 printf 系函数规则相同。

- ... : 可变参数列表，与 format 中的格式说明符一一对应。

返回值：

- 返回如果缓冲区足够大时，应该写入的字符总数（不含末尾的 '\\0' ）。

- 若返回值 ret 满足 0 &lt;= ret &lt; size ，说明输出数据完整写入。

- 若 ret &gt;= size ，说明输出数据被截断，缓冲区中仅包含前 size-1 个字符和一个 '\\0' 。

**课件代码 18-029 · PDF 第 16,17 页**

```c
#include <stdio.h>

struct Stu
{
    char name[30];
    int age;
    float score;
};

int main()
{
    struct Stu s = {"zhangsan", 100, 85.5f};
    printf("%s %d %.1f\n", s.name, s.age, s.score); //打印到屏幕上了
    //将s中的结构数据转换成一个字符串
    char arr[100] = {0};
    int r = snprintf(arr, sizeof(arr), "%s %d %.1f", s.name, s.age, s.score);
    printf("%s\n", arr);
    printf("%d\n", r);
    return 0;
}
```

### 10.3 snprintf 替代 strcpy/strcat

**课件代码 18-030 · PDF 第 17 页**

```c
#include <string.h>
#include <stdio.h>
int main()
{
    char buf[20] = "----------";
    char src[] = "abcdef";
    // 1.复制字符串
    snprintf(buf, sizeof(buf), "%s", src); //strcpy
    printf("%s\n", buf);

    // 2.拼接
    size_t len = strlen(buf);
    snprintf(buf + len, sizeof(buf) - len, "%s", src); //strcat
    printf("%s\n", buf);

    return 0;
}
```

## 11. sscanf

**课件代码 18-031 · PDF 第 17 页**

```c
int sscanf ( const char * str, const char * format, ...);
```

功能：从字符串中读取格式化数据。它与 scanf 类似，但输入源是内存中的字符串而非控制台或文件。常用于解析字符串中的结构化数据（如提取数字、分割文本等）。

参数：

- str ：要解析的源字符串（输入数据来源）。

- format ：格式化字符串，定义如何解析数据（如 %d 、 %f 、 %s 等）。

- ... ：可变参数列表，提供存储数据的变量地址（需与格式字符串中的说明符匹配）

返回值：

- 成功时：返回成功解析并赋值的参数数量（非负值）。

- 失败或未匹配任何数据：若输入结束或解析失败，返回 EOF （通常是 -1 ）。

代码演示：

**课件代码 18-032 · PDF 第 18 页**

```c
#include <stdio.h>

struct Stu
{
    char name[30];
    int age;
    float score;
};

int main()
{
    char str[] = "zhangsan 100 85.5";
    struct Stu s = {0};
    //从str中的字符串内中解析出格式化的数据，存放在结构体中
    sscanf(str, "%s %d %f", s.name, &(s.age), &(s.score));

    //打印结构体s中的数据
    printf("%s %d %.1f\n", s.name, s.age, s.score);
    return 0;
}
```

## 12. strstr

**课件代码 18-033 · PDF 第 18 页**

```c
char * strstr (const char * str1, const char * str2);
```

功能： strstr 函数，查找 str2 指向的字符串在 str1 指向的字符串中第一次出现的位置。

简而言之：在一个字符串中查找子字符串。

参数：

- str1 ：指针，指向了被查找的字符串

- str2 ：指针，指向了要查找的字符串

返回值：

- 如果str1指向的字符串中存在str2指向的字符串，那么返回第一次出现位置的指针

- 如果str1指向的字符串中不存在str2指向的字符串，那么返回NULL

头文件： &lt;string.h&gt;

### 12.1 代码演示

**课件代码 18-034 · PDF 第 19 页**

```c
/* strstr example */
#include <stdio.h>
#include <string.h>
int main ()
{
    char str[] ="This is a simple string";
    char * pch;
    pch = strstr (str,"simple");
    if (pch != NULL)
        printf("%s\n", pch);
    else
        printf("查找的字符串不存在\n");
    return 0;
}
```

### 12.2 strstr的模拟实现

**课件代码 18-035 · PDF 第 19,20 页**

```c
char * strstr (const char * str1, const char * str2)
{
    char *cp = (char *) str1;
    char *s1, *s2;
    //特殊情况：str2是空字符串时，直接返回str1
    if ( !*str2 )
        return((char *)str1);

    while (*cp)
    {
        s1 = cp;
        s2 = (char *) str2;

        while ( *s1 && *s2 && !(*s1 - *s2) )
            s1++, s2++;

        if (!*s2)
            return(cp); //返回第一次出现的起始

        cp++;
    }

    return(NULL); //找不到则返回NULL
}
```

strstr 函数的实现有多种，可以暴力查找，也有一种高效一些的算法：KMP，有兴趣的可以去学习。

📌 AI提示词：模拟实现一下C语言的库函数：strstr，加上必要的注释

## 13. strtok 函数的使用

**课件代码 18-036 · PDF 第 20 页**

```c
char* strtok(char * str, const char *delim);
```

功能：分割字符串：根据 delim 参数中指定的分隔符，将输入字符串 str 拆分成多个子字符串。

- 修改原始字符串： strtok 会直接在原始字符串中插入 '\\0' 终止符，替换分隔符的位置，因

此原始字符串会被修改。

参数：

- str ：首次调用时传入待分割的字符串；后续调用传入 NULL ，表示继续分割同一个字符串。

- delim ：包含所有可能分隔符的字符串（每个字符均视为独立的分隔符）。

返回值

- 成功时返回指向当前子字符串的指针。

- 没有更多子字符串时返回 NULL 。

使用步骤

1. 首次调用：传入待分割字符串和分隔符。

2. 后续调用：传入 NULL 和相同的分隔符，继续分割。

3. 结束条件：当返回 NULL 时，表示分割完成。

### 13.1 代码演示

**课件代码 18-037 · PDF 第 21 页**

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char arr[] = "192.168.6.111";
    const char* sep = ".";
    const char* str = NULL;

    char  buf[30] = {0};
    strcpy(buf, arr); //将arr中的字符串拷贝到buf中，对buf的内容进行切割
    for (str = strtok(buf, sep); str != NULL; str = strtok(NULL, sep))
    {
        printf("%s\n", str);
    }
    return 0;
}
```

### 13.2 注意事项

- 破坏性操作： strtok 会直接修改原始字符串，将其中的分隔符替换为 '\\0' 。如果需要保留原

字符串，应先拷贝一份。

- 连续分隔符：多个连续的分隔符会被视为单个分隔符，不会返回空字符串。

- 空指针处理：如果输入的 str 为 NULL 且没有前序调用，行为未定义。

## 14. strerror 函数的使用

**课件代码 18-038 · PDF 第 21 页**

```c
char* strerror ( int errnum );
```

功能：

1. strerror 函数可以通过参数部分的 errnum 表示错误码，得到对应的错误信息，并且返回这

个错误信息字符串首字符的地址。

2. strerror 函数只针对标准库中的函数发生错误后设置的错误码的转换。

3. strerror 的使用需要包含&lt;string.h&gt;

在不同的系统和C语言标准库的实现中都规定了一些错误码，一般是放在 errno.h 这个头文件中说明的，C语言程序启动的时候就会使用一个全局的变量 errno 来记录程序的当前错误码，只不过程序启动的时候 errno 是 0，表示没有错误，当我们在使用标准库中的函数的时候发生了某种错误，就会将对应的错误码，存放在 errno 中，而一个错误码的数字是整数，很难理解是什么意思，所以每一个错误码都是有对应的错误信息的。strerror函数就可以将错误码对应的错误信息字符串的地址返回。

参数：

errnum ：表示错误码这个错误码一般传递的是 errno 这个变量的值，在C语言有一个全局的变量叫： errno ，当库函数的调用发生错误的时候，就会将本次错误的错误码存放在 errno 这个变量中，使用这个全局变量需要包含一个头文件 errno.h 。

返回值：

函数返回通过错误码得到的错误信息字符串的首字符的地址。

### 14.1 代码演示

**课件代码 18-039 · PDF 第 22 页**

```c
#include <errno.h>
#include <string.h>
#include <stdio.h>

//我们打印一下0~10这些错误码对应的信息
int main()
{
    int i = 0;
    for (i = 0; i <= 10; i++) {
        printf("%d: %s\n", i,  strerror(i));
    }
    return 0;
}
```

在Windows11 + VS2022环境下输出的结果如下：

**课件代码 18-040 · PDF 第 22,23 页**

```text
0: No error
1: Operation not permitted
2: No such file or directory
3: No such process
4: Interrupted function call
5: Input/output error
6: No such device or address
7: Arg list too long
8: Exec format error
9: Bad file descriptor
10: No child processes
```

举例：

**课件代码 18-041 · PDF 第 23 页**

```c
#include <stdio.h>
#include <string.h>
#include <errno.h>
int main ()
{
    FILE * pFile = NULL;
    //fopen函数以读的形式打开文件，如果文件不存在，则打开失败。
    pFile = fopen ("unexist.ent", "r");
    if (pFile == NULL)
    {
        printf ("错误信息是：%s\n", strerror(errno));
        return 1;//错误返回
    }

    return 0;
}
```

输出：

**课件代码 18-042 · PDF 第 23 页**

```text
错误信息是：No such file or directory
```

### 14.2 perror

**课件代码 18-043 · PDF 第 23 页**

```c
void perror ( const char * str );
```

也可以了解一下 perror 函数， perror 函数相当于一次将上述代码中的第11行完成了，直接将错误信息打印出来。 perror 函数打印完参数部分的字符串后，再打印一个冒号和一个空格，再打印错误信息。

**课件代码 18-044 · PDF 第 24 页**

```c
#include <stdio.h>
#include <string.h>
#include <errno.h>

int main ()
{
    FILE * pFile = NULL;
    pFile = fopen ("unexist.ent", "r");
    if (pFile == NULL)
    {
        perror("错误信息是");
        return 1;
    }
    return 0;
}
```

输出：

**课件代码 18-045 · PDF 第 24 页**

```text
错误信息是: No such file or directory
```

## 15. 代码实践：命令行待办事项

📌 AI提示词：

我想使用C语言实现一个命令行待办事项。

主要使用数组、字符串操作函数、函数封装的知识，所有任务存在一个字符串数组里希望支持以下功能：

1. 添加任务， 比如：“add 学C语言”

2. 列出所有任务 ， 比如：“list”

3. 标记第x项完成 , 比如："done 2"

这里主要锻炼我们描述任务的能力和阅读代码的能力。

## 课件中的参考链接

- [参考资料 1](https://legacy.cplusplus.com/reference/cctype/)
- [参考资料 2](https://legacy.cplusplus.com/reference/cstring/)
- [参考资料 3](http://www.cplusplus.com/reference/cstring/strlen/?kw=strlen)
- [参考资料 4](https://cplusplus.com/reference/cstring/strcpy/?kw=strcpy)
- [参考资料 5](https://cplusplus.com/reference/cstring/strncpy/?kw=strncpy)
- [参考资料 6](https://cplusplus.com/reference/cstring/strcat/?kw=strcat)
- [参考资料 7](https://cplusplus.com/reference/cstring/strncat/?kw=strncat)
- [参考资料 8](https://cplusplus.com/reference/cstring/strcmp/?kw=strcmp)
- [参考资料 9](https://cplusplus.com/reference/cstring/strncmp/?kw=strncmp)
- [参考资料 10](https://legacy.cplusplus.com/reference/cstdio/sprintf/?kw=sprintf)
- [参考资料 11](https://legacy.cplusplus.com/reference/cstdio/snprintf/?kw=snprintf)
- [参考资料 12](https://legacy.cplusplus.com/reference/cstdio/sscanf/?kw=sscanf)
- [参考资料 13](https://cplusplus.com/reference/cstring/strstr/?kw=strstr)
- [参考资料 14](https://www.bilibili.com/video/BV1AN9QBJEqq)
- [参考资料 15](https://legacy.cplusplus.com/reference/cstring/strtok/?kw=strtok)
- [参考资料 16](https://legacy.cplusplus.com/reference/cstring/strerror/?kw=strerror)
- [参考资料 17](https://legacy.cplusplus.com/reference/cstdio/perror/?kw=perror)
