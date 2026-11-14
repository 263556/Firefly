---
title: C语言初阶 07：函数设计、传参与多文件程序
published: 2026-11-14
description: 用素数判断和数组统计理解接口设计，掌握库函数、实参与形参、返回值、数组传参、声明定义、static 与 extern。
tags: [C语言, 初阶, 函数]
category: C语言
series: C语言学习笔记
seriesOrder: 7
draft: false
---


## 1. 引入：把“判断素数”变成工具

如果多个地方都需要判断素数，不必复制整段循环。把它封装成函数，调用者只关心输入和结果。

```c
#include <stdio.h>

int is_prime(int n)
{
    if (n < 2) {
        return 0;
    }
    for (int d = 2; d <= n / d; ++d) {
        if (n % d == 0) {
            return 0;
        }
    }
    return 1;
}

int main(void)
{
    for (int n = 1; n <= 30; ++n) {
        if (is_prime(n)) {
            printf("%d ", n);
        }
    }
    putchar('\n');
    return 0;
}
```

接口含义明确：输入一个 `int`，素数返回 1，否则返回 0。负数、0、1 都有明确结果。函数既能复用，也把一个大任务拆成可单独验证的小任务。

## 2. 函数的组成与库函数学习方法

```c
返回类型 函数名(参数列表)
{
    函数体
}
```

例如 `double average(int a, int b)` 的输入是两个整数，输出是浮点数。写成 `(double)a / 2 + (double)b / 2` 可避免先做 `a + b` 的有符号溢出；对于普通小成绩，也可写 `((double)a + b) / 2.0`。

库函数学习时查清：所属头文件、原型、参数约束、返回值和失败表示。例如 `strlen`：头文件 `<string.h>`、参数必须指向有效的以 `\0` 结束的字符串、返回类型为 `size_t`，不计终止符。

不要只背函数名。知道 `printf` 能输出，却不检查格式与参数类型，仍然会写出错误代码。

## 3. 实参、形参与按值传递

```c
void set_to_zero(int value)
{
    value = 0;
}

// 调用处：
int x = 10;
set_to_zero(x);
printf("%d\n", x); // 10
```

调用中的 `x` 是实参，函数定义中的 `value` 是形参。函数得到实参值的副本，修改副本不会改变 `x`。形参名可以与实参名相同，但它们仍是不同对象。

需要改动调用者的数据时传地址：

```c
void set_to_zero(int *value)
{
    *value = 0;
}
// 调用：set_to_zero(&x);
```

C 仍然是按值传递：这次复制的是地址值。不要误认为出现指针后语言换成了真正的“引用传递”。调用者必须提供有效地址，函数的接口也应说明这一前提。

## 4. return 与单一职责

`return expression;` 立即结束当前函数并把结果传回；`return;` 用于返回类型为 `void` 的函数。返回类型不是 `void` 的普通函数应保证需要返回的路径都有合法返回值。

```c
int max_of_two(int a, int b)
{
    if (a > b) {
        return a;
    }
    return b;
}
```

一个函数可有多个 `return`，用于清晰表达早退出。`return` 不能一次返回多个独立值，逗号表达式也不会变成多返回值；需要多个结果时可使用指针输出参数或结构体。

不要返回指向自动局部变量的地址，因为函数返回后对象生命周期已经结束。返回局部整数的**值**则没有这个问题。

## 5. 数组传参必须同时约定长度

```c
#include <stddef.h>

long long sum_array(const int data[], size_t length)
{
    long long sum = 0;
    for (size_t i = 0; i < length; ++i) {
        sum += data[i];
    }
    return sum;
}
```

`const int data[]` 在函数参数中调整为 `const int *data`。函数内 `sizeof data` 是指针大小，不能求出原数组长度。调用者传 `sum_array(values, sizeof values / sizeof values[0])`，并保证这个长度不超出有效元素数、总和可由 `long long` 表示。

`const` 表示函数不会通过该参数修改元素。即使数组整体没有被复制，也要分清“改元素”与“重新赋值给形参指针”的区别。

二维数组参数需要知道行宽，例如 `void print_board(char board[][11], int rows);`，编译器才能定位第 `r` 行。

## 6. 嵌套调用与链式使用结果

```c
printf("较大值：%d\n", max_of_two(12, 8));
```

内层函数的返回值作为外层函数的参数。函数可以在函数体内调用其他函数，但标准 C 不支持在函数体内再定义普通函数。

不要依赖多个函数实参之间的固定求值顺序，例如同时在两个参数中修改同一状态。复杂步骤先保存到变量，再调用，更容易调试。

## 7. 声明、定义与多文件组织

声明告诉编译器“有这个函数”，定义提供实现。调用前应能看到正确声明。多个源文件需要共享声明时放进头文件，并用头文件保护避免重复包含。

`math_utils.h`：

```c
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

int max_of_two(int a, int b);

#endif
```

`math_utils.c`：

```c
#include "math_utils.h"

int max_of_two(int a, int b)
{
    return a > b ? a : b;
}
```

`main.c`：

```c
#include <stdio.h>
#include "math_utils.h"

int main(void)
{
    printf("%d\n", max_of_two(3, 8));
    return 0;
}
```

用 GCC 可运行 `gcc -std=c17 -Wall -Wextra main.c math_utils.c -o demo`。若忘记编译 `math_utils.c`，声明虽然能使编译通过，链接时仍可能找不到定义。通常不要用 `#include "math_utils.c"` 代替正确的工程组织。

## 8. static 与 extern 的三种关键场景

| 用法 | 作用 |
| --- | --- |
| 局部 `static` 对象 | 静态存储期，跨调用保留值；名字仍是局部的 |
| 文件作用域 `static` 对象 | 名称具有内部链接，只在本翻译单元使用 |
| `static` 函数 | 名称具有内部链接，隐藏内部实现 |
| `extern int count;` | 声明外部对象，定义通常在另一个源文件中 |

```c
int next_id(void)
{
    static int id = 0;
    return ++id;
}
```

连续调用结果为 1、2、3，而不是每次都为 1。局部静态对象只初始化一次，不是每次进入函数都重新赋值。`extern` 声明不负责神奇地找到或创建另一个定义，工程仍需提供定义并完成链接。

## 9. 练习

把成绩统计拆成“求和、最高分、及格人数”三个函数，统一约定空数组如何处理。再把函数声明和定义拆成 `.h`、`.c`，检查每个函数的输入、输出、可修改数据和失败条件是否说得清楚。
