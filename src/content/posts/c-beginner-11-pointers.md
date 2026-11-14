---
title: C语言初阶 11：地址、指针类型与指针运算
published: 2026-11-14
description: 通过交换变量、遍历数组与字符串长度理解指针，梳理解引用、指针大小、void 指针、运算边界和二级指针。
tags: [C语言, 初阶, 指针]
category: C语言
series: C语言初阶笔记
seriesOrder: 11
draft: false
---

> 依据《（新 AI 版）鹏哥 C 语言课件》初阶第 11 章整理。

## 1. 引入：为什么交换函数改不动外面的变量

把两个整数按值传给函数，函数交换的只是副本。要让它修改原对象，必须让函数知道原对象位于哪里。

```c
#include <stdio.h>

void swap(int *left, int *right)
{
    int temporary = *left;
    *left = *right;
    *right = temporary;
}

int main(void)
{
    int a = 3;
    int b = 8;
    swap(&a, &b);
    printf("a=%d, b=%d\n", a, b); // a=8, b=3
    return 0;
}
```

这里仍是按值传递：复制的是 `a`、`b` 的地址，函数通过地址修改原对象。这种用法常被称为“传址调用”。

## 2. 地址、指针值和指针变量

可以把内存理解为可定位的一组存储单元，地址用于定位对象。指针值表示某个地址关系，指针变量则是保存这种值的对象。

```c
int number = 10;
int *p = &number;
*p = 20;
```

- `number`：普通整数对象，当前值变为 20。
- `&number`：取地址表达式，类型为 `int *`。
- `p`：保存 `number` 地址的指针变量。
- `*p`：解引用，表示 p 指向的整数对象。
- `&p`：指针变量自身的地址，类型为 `int **`。

`*` 在声明中用于构成指针类型，在表达式中可表示解引用；`&` 也要与双目按位与区分。

```c
printf("对象地址：%p\n", (void *)&number);
```

输出对象地址用 `%p` 并转换为 `void *`，不要用 `%d` 假定指针等同于整数。

## 3. 指针类型有什么意义

类型告诉编译器以何种对象类型访问地址，也决定加减整数时的步长。

```c
int values[3] = {10, 20, 30};
int *p = values;
printf("%d %d\n", *p, *(p + 1)); // 10 20
```

`p + 1` 指向下一个 `int` 元素，而不是简单增加一个字节。`double *` 加一前进一个 `double` 的空间。强行转换为不匹配的指针并解引用，还可能违反对齐和有效类型规则，不能认为“有地址就能用任何类型读写”。

指针变量的大小取决于目标实现和 ABI，常见 32 位程序为 4 字节、64 位程序为 8 字节；标准不保证所有指针种类在所有系统上大小都相同。`sizeof p` 是指针大小，`sizeof *p` 是它所指类型的大小，不要混淆。

### 声明容易踩的坑

```c
int *p, value; // 只有 p 是指针，value 是 int
int *a;
int *b;       // 分行声明通常更清楚
```

## 4. void * 与 const 的基本用法

`void *` 是通用对象指针，适合传递未知具体类型的对象地址。标准 C 不能直接解引用 `void *`，也不能直接对它做加减；先转换或赋给合适的对象指针再操作。

```c
int number = 42;
void *address = &number;
int *p = address; // C 中对象指针与 void * 可转换
printf("%d\n", *p);
```

`const int *p` 表示不能通过 p 修改整数，p 自己可改指向；`int *const p` 表示指针对象不可改指向，但可修改所指的非 const 整数。不要通过强制去掉 `const` 去修改本来就定义为 const 的对象。

## 5. 数组与指针的联系和区别

数组名在大多数表达式中会转换为指向首元素的指针，因此 `a[i]` 等价于 `*(a + i)`。但数组本身是固定元素数的对象，指针只是一个保存地址的对象。

```c
int a[4] = {1, 2, 3, 4};
int *p = a;
printf("数组字节数：%zu\n", sizeof a);
printf("指针字节数：%zu\n", sizeof p);
```

`sizeof a` 和 `&a` 等语境不发生常规首元素转换：`&a` 的类型是“指向整个四元素数组的指针”，不是 `int *`。

```c
for (int *it = a; it != a + 4; ++it) {
    printf("%d ", *it);
}
```

`a + 4` 是合法的尾后指针，可用于循环停止，但不能解引用。甚至不能随意形成超出同一数组及其尾后范围的指针，再希望“只要不读取就没关系”。

## 6. 指针减法和比较的范围

同一数组内的两个元素指针（包括尾后位置）可相减，结果是元素个数之差，类型为 `ptrdiff_t`，定义于 `<stddef.h>`。

```c
#include <stddef.h>
#include <stdio.h>

int main(void)
{
    int a[5] = {0};
    ptrdiff_t distance = &a[4] - &a[1];
    printf("%td\n", distance); // 3，不是 3*sizeof(int)
    return 0;
}
```

不要对无关数组中的指针做减法，或用大小关系推断无关对象的先后。指针相等性可用于判断是否为同一地址或空指针，但普通大小比较有更严格的适用范围。

## 7. 案例：模拟 strlen

一种方法是计数，一种方法是让末尾指针减去起点。两种都要求输入指向可访问且有终止符的字符串。

```c
#include <stddef.h>

size_t string_length(const char *text)
{
    const char *end = text;
    while (*end != '\0') {
        ++end;
    }
    return (size_t)(end - text);
}
```

起点不动，`end` 不断前进，最终距离就是终止符前的字节数。这里假定指针差可由 `ptrdiff_t` 表示；普通教学字符串满足此条件。函数不接受 `NULL`，也不能检测一个缺少终止符的字符数组是否安全，调用方必须满足契约。

## 8. 二级指针：修改指针本身

```c
#include <stdio.h>

void redirect(int **target, int *destination)
{
    *target = destination;
}

int main(void)
{
    int a = 10;
    int b = 20;
    int *p = &a;
    redirect(&p, &b);
    printf("%d\n", *p); // 20
    return 0;
}
```

如果函数只接收 `int *target`，给 `target` 赋新地址只会改形参副本。传 `&p` 后，`*target` 才代表调用者的指针变量 p。`**target` 则是沿两层指针找到最终整数。

## 9. 指针使用的三条底线

1. **初始化**：未确定地址时用 `NULL`，但空指针仍不能解引用。
2. **生命周期**：对象已结束生命周期，旧地址就不能继续使用，典型错误是返回局部变量地址。
3. **边界和权限**：访问必须在有效范围内，并使用合适的类型；只读对象不能写。

指针不是“更高级的整数”。每次使用都问：指向什么、还活着吗、能访问几项、允许写吗？

练习：用指针遍历求数组和；手画 `int n`、`int *p`、`int **pp` 的关系；比较“改 `*p` 的值”和“把 p 改为另一个地址”的差别。
