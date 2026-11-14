---
title: C语言进阶 23：宏、条件编译与预处理陷阱
published: 2026-11-14
description: 从平方宏的错误展开出发，理解括号、副作用、预定义符号、头文件保护、字符串化与记号拼接，实践多版本计算器。
tags: [C语言, 进阶, 预处理, 宏]
category: C语言
series: C语言学习笔记
seriesOrder: 23
draft: false
---

> 依据鹏哥 C 语言进阶第 23 章整理。

## 1. 引入：SQUARE(a + 1) 为什么不是平方

```c
#define SQUARE_BAD(x) x * x
```

`SQUARE_BAD(a + 1)` 展开为 `a + 1 * a + 1`，a 为 5 时得到 11。宏不是先计算实参再调用，而是进行预处理记号替换。

```c
#define SQUARE(x) ((x) * (x))
```

每个参数出现处加括号，整体表达式再加括号，才能同时保护参数内部和外部运算的优先级。但这仍不能解决重复求值，后面会继续分析。

## 2. 预定义符号与对象式宏

| 符号 | 含义 |
| --- | --- |
| `__FILE__` | 当前源文件名信息 |
| `__LINE__` | 当前逻辑行号 |
| `__DATE__`、`__TIME__` | 翻译时日期和时间，不是程序启动时间 |
| `__STDC__` | 实现声明符合标准的标识 |
| `__STDC_VERSION__` | 支持相应 C 标准版本的标识，具体可用性看模式 |

```c
#include <stdio.h>

#define MAX_ITEMS 16

int main(void)
{
    printf("%s:%d capacity=%d\n", __FILE__, __LINE__, MAX_ITEMS);
    return 0;
}
```

对象式宏替换记号，不分配一个变量。定义数值常量时通常不在末尾放分号，否则 `if/else` 等语句展开后可能多出空语句。字符串字面量内部不会因为出现同名文字就被宏替换。

多行宏使用行尾反斜杠续行，反斜杠后不要再放注释或空格。`#undef NAME` 取消后续的宏定义，不会追溯改变已经处理的代码。

用户自定义名称避免双下划线，以及以下划线加大写字母开头等保留名称。使用 `APP_DEBUG` 比模仿实现内部名字更合适。

## 3. 函数式宏与副作用

宏名与定义中的左括号必须紧邻：`#define MAX(a, b) ...` 是函数式宏，`#define MAX (a, b) ...` 则是不同的对象式宏。

```c
#define MAX_VALUE(a, b) ((a) > (b) ? (a) : (b))
```

如果写 `MAX_VALUE(x++, y++)`，比较阶段会各自递增一次，被选择的分支还会再次递增相应变量，产生与函数不同的行为。`SQUARE(i++)` 更会对同一对象进行未排序的多次修改，导致未定义行为。括号不会消除这些问题。

优先使用有类型、参数只求值一次的小函数：

```c
static inline int max_int(int a, int b)
{
    return a > b ? a : b;
}
```

现代编译器可能内联函数，因此不能笼统说“宏一定更快、函数一定有调用开销”。宏的价值在于条件选择、类型/记号生成等预处理能力，不是保证性能。

## 4. 多语句宏为何常用 do while

```c
#define TRACE_VALUE(value) \
    do { \
        fprintf(stderr, "%s:%d value=%d\n", __FILE__, __LINE__, (value)); \
    } while (0)
```

把它当作一条语句调用：`TRACE_VALUE(number);`，可以在 if/else 中保持正确结构。宏内部若包含 break/continue，实际控制的是这个 do while，不能假定跳出调用点外层循环。

示例中的 value 只出现于一次运行时求值，但这不是所有宏天然具有的属性。定义宏时要检查每个参数出现次数，并把副作用规则写清楚。

## 5. 条件编译与运行时 if 的区别

```c
#ifndef APP_DEBUG
#define APP_DEBUG 0
#endif

#if APP_DEBUG
    /* 只在调试配置中进入翻译单元的代码 */
#endif
```

条件编译在翻译前决定哪些记号保留；运行时 if 的分支仍需在相应语境中通过语法和类型检查，之后才按运行值选择。

`#ifdef APP_DEBUG` 判断是否定义，不关心定义为 0 还是 1；`#if APP_DEBUG` 按数值判断。`#if` 中未定义标识符通常按 0 处理，不能使用普通变量，也不能把 `sizeof` 当成预处理算术。

多分支可用 `#if / #elif / #else / #endif`。配置不合理时用 `#error` 让构建尽早失败，而不是编译出含糊行为的程序。

## 6. 头文件保护与搜索路径

```c
#ifndef MYPROJECT_CALCULATOR_H
#define MYPROJECT_CALCULATOR_H

int calculate(int a, int b);

#endif
```

保护宏应足够独特，防止不同文件误用同名宏而隐藏内容。它解决同一翻译单元的重复包含，不解决不同源文件中的重复外部定义。

`#include "local.h"` 通常先在源文件附近等实现规定的位置查找，再按工具链规则继续；`#include <stdio.h>` 按配置的系统/包含路径搜索。精确搜索顺序由实现和编译选项决定，不要硬编码某个版本 IDE 的安装目录当作通用规则。

`#pragma once` 常见但不是 ISO C 的统一标准指令；传统 include guard 可移植性更明确。`#pragma pack` 同样依赖编译器，使用后应恢复原状态，避免影响后面的类型。

## 7. # 与 ##：字符串化和记号拼接

`#` 把宏参数的记号变成字符串；`##` 将两侧记号拼成一个有效预处理记号。它不是运行时字符串连接，拼接结果必须合法。

```c
#include <stdio.h>

#define STRING_RAW(x) #x
#define STRING(x) STRING_RAW(x)
#define JOIN_RAW(a, b) a##b
#define JOIN(a, b) JOIN_RAW(a, b)
#define VERSION 3

int main(void)
{
    int JOIN(value_, VERSION) = 42;
    puts(STRING_RAW(VERSION)); // VERSION
    puts(STRING(VERSION));     // 3
    printf("%d\n", value_3);  // 42
    return 0;
}
```

紧邻 `#` 或 `##` 的参数不按普通参数规则先展开；多加一层宏可以先展开再字符串化/拼接。一般宏参数在代入前展开，展开结果还会再次扫描，但自递归宏不会像函数那样无限递归执行。

这类技巧适合生成重复声明，过度使用会让错误定位困难。真正的字符串拼接在 C 中可用相邻字面量或运行时缓冲区处理，不能混为一谈。

## 8. 完整案例：按配置构建不同版本计算器

普通版支持加减，Pro 版增加乘除；语言由构建宏选择，调试输出可关闭。固定操作数为 12 和 3，重点展示配置，而不是把任意整数溢出处理混入本例。

```c
#include <stdio.h>

#ifndef APP_PRO
#define APP_PRO 0
#endif
#ifndef APP_ENGLISH
#define APP_ENGLISH 0
#endif
#ifndef APP_DEBUG
#define APP_DEBUG 0
#endif

#if (APP_PRO != 0 && APP_PRO != 1) || \
    (APP_ENGLISH != 0 && APP_ENGLISH != 1) || \
    (APP_DEBUG != 0 && APP_DEBUG != 1)
#error "Configuration values must be 0 or 1"
#endif

#if APP_ENGLISH
#define MSG_UNSUPPORTED "operation not available"
#define MSG_ZERO "division by zero"
#else
#define MSG_UNSUPPORTED "当前版本不支持此操作"
#define MSG_ZERO "除数不能为零"
#endif

static int calculate(char operation, int a, int b, int *out)
{
    switch (operation) {
    case '+': *out = a + b; return 1;
    case '-': *out = a - b; return 1;
#if APP_PRO
    case '*': *out = a * b; return 1;
    case '/':
        if (b == 0) {
            puts(MSG_ZERO);
            return 0;
        }
        *out = a / b;
        return 1;
#endif
    default:
        puts(MSG_UNSUPPORTED);
        return 0;
    }
}

int main(void)
{
    const char operations[] = "+-*/";
    for (size_t i = 0; operations[i] != '\0'; ++i) {
        int result;
        if (calculate(operations[i], 12, 3, &result)) {
            printf("12 %c 3 = %d\n", operations[i], result);
        }
    }
#if APP_DEBUG
    fprintf(stderr, "%s:%d pro=%d english=%d\n",
            __FILE__, __LINE__, APP_PRO, APP_ENGLISH);
#endif
    return 0;
}
```

默认编译 `gcc -std=c17 calculator.c -o calculator`，只成功输出加减结果；另两项提示不支持。完整英文调试版：

```sh
gcc -std=c17 -DAPP_PRO=1 -DAPP_ENGLISH=1 -DAPP_DEBUG=1 calculator.c -o calculator-pro
```

这个计算函数的前提是传入非空 out，且所有运算结果可表示；扩展到任意输入时还需检查溢出，尤其除法的最小负数除以 -1。条件编译只控制功能是否存在，不替代算术验证。

## 9. 观察展开与练习

用 `gcc -E calculator.c -o calculator.i`，再给 Pro 选项预处理，比较乘除分支是否存在。分别构建 2×2 的版本/语言组合，不能只测开发时的一个配置。

练习：给宏调用传入带加法的表达式，观察括号作用；把 APP_DEBUG 定义为 0，比较 ifdef 与 if；给字符串化宏去掉中间一层，解释输出差别。看到复杂宏先展开，再讨论运行行为。
