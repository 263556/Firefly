---
title: C语言初阶 02：数据类型、变量、常量与字符
published: 2026-11-14
description: 用购物小票理解类型选择，梳理整数修饰符、类型大小、ASCII、转义字符、变量作用域和常量。
tags: [C语言, 初阶, 数据类型]
category: C语言
series: C语言学习笔记
seriesOrder: 2
draft: false
---


## 1. 引入：一张购物小票需要哪些数据

商品有名称、单价、数量和小计。数量通常是整数，价格有小数，名称是一串字符。数据类型描述数据的表示方式、范围以及可执行的操作。

```c
#include <stdio.h>

int main(void)
{
    char grade = 'A';
    int quantity = 3;
    double price = 12.50;
    const double discount = 0.9;

    printf("等级：%c，件数：%d\n", grade, quantity);
    printf("折后总价：%.2f\n", price * quantity * discount);
    return 0;
}
```

`quantity` 是变量名，`int` 是类型，`3` 是初始值。变量应先初始化再使用；未初始化的自动局部变量不能假设为 0。

## 2. 基本类型与格式说明符

| 类型 | 典型用途 | printf | scanf |
| --- | --- | --- | --- |
| `char` | 单字节字符或小整数 | `%c` | `%c` |
| `int` | 常规整数 | `%d` | `%d` |
| `unsigned int` | 非负整数及位运算 | `%u` | `%u` |
| `long long` | 更大范围的整数 | `%lld` | `%lld` |
| `float` | 单精度浮点 | `%f` | `%f` |
| `double` | 双精度浮点 | `%f` | `%lf` |
| `size_t` | 大小、长度 | `%zu` | `%zu` |

`printf` 的可变参数会把 `float` 提升为 `double`，因此输出两者都可用 `%f`；`scanf` 接收指针，`float *` 与 `double *` 不同，格式必须区分。`%.2f` 表示输出时保留两位小数，并不把变量内部存储变成精确的两位十进制数。

```c
float f = 3.14f;    // f 后缀指定 float 字面量
double d = 3.14;    // 默认是 double
long long n = 10000000000LL;
```

浮点数通常不能精确表示所有十进制小数。金额计算常选整数“分”，而不是直接假设 `0.1 + 0.2` 与 `0.3` 在二进制中完全相同。

## 3. short、long、signed、unsigned

整数类型可按大小和是否带符号组合。`short int` 可以简写为 `short`，`long int` 可简写为 `long`。

- `signed`：允许负值。
- `unsigned`：只表示非负值，运算按相应模数回绕。
- `short`、`long`、`long long`：提供不同的最小范围保证。
- 普通 `char` 是带符号还是无符号，由实现决定，不能默认等同于 `signed char`。

**不要把“常见机器上 int 是 4 字节”误当作语言对所有平台的保证。** 标准规定 `sizeof(char) == 1`；这个 C 字节有多少位看 `CHAR_BIT`，常见值是 8。`int` 至少能表示 -32767 到 32767，具体范围查 `<limits.h>`。

```c
#include <stdio.h>
#include <limits.h>

int main(void)
{
    printf("一个字节有 %d 位\n", CHAR_BIT);
    printf("int：%zu 字节，范围 %d 到 %d\n", sizeof(int), INT_MIN, INT_MAX);
    printf("unsigned int 最大值：%u\n", UINT_MAX);
    return 0;
}
```

有符号整数溢出是未定义行为，不能依靠“变成负数”。无符号虽然有回绕规则，也不意味着可以忽视范围检查。需要指定宽度时，可查看 `<stdint.h>` 的 `int32_t`、`uint32_t` 等类型；精确宽度类型是否提供取决于实现是否支持相应表示。

## 4. 字符、ASCII 与字符串

字符本质上按数值编码表示。在 ASCII 中，`'0'` 到 `'9'` 连续，数字字符可以用 `ch - '0'` 转为对应数值。

```c
char ch = '7';
int value = ch - '0';
printf("字符：%c，数字：%d\n", ch, value); // 字符：7，数字：7
```

`'7'`、整数 `7` 和字符串 `"7"` 是不同概念。在 C17 中，普通字符常量如 `'A'` 的类型实际是 `int`，常用它给 `char` 变量赋值。

| 转义写法 | 含义 |
| --- | --- |
| `\n` | 换行 |
| `\t` | 水平制表 |
| `\\` | 反斜杠 |
| `\"` | 双引号 |
| `\'` | 单引号 |
| `\0` | 空字符，数值为 0 |

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char text[] = "A\nB";
    printf("strlen = %zu, sizeof = %zu\n", strlen(text), sizeof text);
    return 0;
}
```

结果是 `strlen = 3, sizeof = 4`。`\n` 是一个字符，结尾还有自动附加的 `\0`。`strlen` 不把结束符计入长度，`sizeof` 统计整个数组的字节数。

中文等字符可能占多个字节；普通 `strlen` 统计终止符前的字节数，不是屏幕上看见的汉字个数。`'\0'` 与 `'0'` 也不同，后者是数字字符。

## 5. 变量的作用域和生命周期

局部变量在函数或代码块内声明，名称通常只在相应块内可见；全局变量在所有函数外定义。作用域回答“哪里能用这个名字”，生命周期回答“对象什么时候存在”，二者不能混为一谈。

```c
#include <stdio.h>

int total = 100; // 文件作用域；静态存储期

int main(void)
{
    int count = 3; // 自动局部变量
    {
        int count = 8; // 内层同名变量遮蔽外层变量
        printf("%d\n", count); // 8
    }
    printf("%d %d\n", count, total); // 3 100
    return 0;
}
```

初学时尽量避免同名遮蔽和不必要的全局变量。具有静态存储期、未显式初始化的对象会被零初始化；不要把这个规则套到普通自动局部变量上。课件中常用“栈区、静态区”辅助理解，但语言本身不要求所有实现都使用某种具体内存布局。

## 6. 三种常量写法

```c
#define DAYS_PER_WEEK 7

const double pi = 3.141592653589793;
int weeks = 2;
int days = weeks * DAYS_PER_WEEK;
```

- **字面常量**：`10`、`3.14`、`'A'`、`"C"` 等直接写出的值。整数还可用八进制、十六进制等表示，如 `010` 是八进制 8，`0x10` 是十进制 16。
- **宏常量**：预处理文本替换，没有独立对象，也不按变量的方式检查类型。
- **const 对象**：有类型、占据存储，其值不能通过该名字修改。C 中 `const int n = 10` 不会自动使 `n` 成为所有语境中的整数常量表达式。

C99 可使用 `<stdbool.h>` 提供的 `bool`、`true`、`false`，底层类型是 `_Bool`；C23 将这些名称纳入关键字体系。本文示例以 C17 为基准，需要时包含该头文件。

## 7. 自测与易错点

1. 预测 `sizeof "abc"` 和 `strlen("abc")`：分别为 4、3。
2. 为什么 `scanf("%f", &double_value)` 不正确？指针类型与格式不符，应为 `%lf`。
3. 如何选择商品编号的类型？若需要保留前导 0 或不参与运算，通常应保存为字符串。
4. 本地打印 `sizeof(long)`，再解释为什么不能根据自己的结果推断其他系统。

类型选择要从数据含义和范围出发，而不是只看变量当前存了多大的数。
