---
title: C语言进阶 16：函数指针、回调与通用排序
published: 2026-11-14
description: 用运算分发表和 qsort 理解函数地址、typedef、函数指针数组、回调契约与按字节交换的通用排序。
tags: [C语言, 进阶, 函数指针, 回调]
category: C语言
series: C语言进阶笔记
seriesOrder: 16
draft: false
---

> 依据鹏哥 C 语言进阶第 16 章整理。

## 1. 引入：排序过程相同，比较方法不同

给整数排序，可以直接比较数值；给学生排序，可能按年龄，也可能按姓名。排序算法负责“何时比较和交换”，使用者负责“怎样判断先后”。函数指针让这两件事分离，回调则是把某个行为作为参数传给另一段程序。

先看最简单的函数指针：

```c
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int main(void)
{
    int (*operation)(int, int) = add;
    printf("%d %d\n", operation(2, 3), (*operation)(4, 5));
    return 0;
}
```

输出 `5 9`，两种调用形式都合法。函数名在这里转换为函数指针，`&add` 也可用于初始化同类型指针。

## 2. 从声明读出完整类型

```c
int (*operation)(int, int);
```

从 operation 向外读：先与 `*` 结合，是指针；指向的函数接收两个 `int`，返回 `int`。括号必需，否则 `int *operation(int, int)` 声明的是一个返回 `int *` 的函数。

函数指针的参数与返回类型必须兼容，不能用强制转换把不兼容接口“凑成可调用”。转换不改变真实函数的调用契约，错误类型调用会导致未定义行为。

无参数函数在 C17 中写 `void f(void)`；`void f()` 表示未提供参数类型信息的旧式声明，不能简单当成同一规则。示例统一使用原型。

标准 C 没有把函数指针当作 `void *` 对象指针的通用保证，`%p` 也要求 `void *`。不要通过强转并打印函数地址来推导可移植接口。零值转换得到空函数指针，调用它是未定义行为，不能把课件中的语法拆解题当作可运行的“调用零地址函数”。

## 3. typedef 把复杂声明变成接口名称

```c
typedef int (*BinaryOperation)(int, int);
BinaryOperation operation = add;
```

别名没有创建新函数，也没有额外运行开销，只是让类型更容易阅读。类似地，`typedef int (*RowPointer)[5];` 是数组指针别名。

对课件中 signal 风格的声明，可以先定义回调类型：

```c
typedef void (*Handler)(int);
Handler choose_handler(int code, Handler replacement);
```

这样就看出：函数接收一个整数和一个 Handler，返回一个 Handler。拆解思路是找到函数名、参数列表，再解释返回类型，而不是从最左边逐字符硬背。

## 4. 函数指针数组：把选择映射到行为

```c
#include <stdio.h>
#include <stddef.h>

typedef int (*BinaryOperation)(int, int);

int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }
int multiply(int a, int b) { return a * b; }

int main(void)
{
    BinaryOperation table[] = {add, subtract, multiply};
    const char *names[] = {"add", "subtract", "multiply"};
    size_t choice = 2;
    size_t count = sizeof table / sizeof table[0];
    if (choice >= count) {
        return 1;
    }
    printf("%s: %d\n", names[choice], table[choice](6, 3));
    return 0;
}
```

输出 `multiply: 18`。分发表把大量重复调用分支改为索引选择，但索引检查不可省略。操作函数也要处理自身契约：真实计算器需要检查溢出，除法还需检查零除和 `INT_MIN / -1` 等情况。

## 5. 回调的契约不只是一根地址

把比较函数交给排序算法后，排序算法会在需要时调用它。要确认参数指向什么、返回值含义、是否允许修改数据、回调指针要保持有效多久。

回调可以同步调用，也可能被其他 API 保存到稍后使用；不能因为当前例子立即调用，就假设所有回调都没有生命周期问题。需要额外上下文的接口常采用“函数指针 + `void *context`”，避免依靠全局状态。

## 6. qsort 四个参数各自负责什么

`qsort` 在 `<stdlib.h>` 中声明，概念原型如下：

```c
void qsort(void *base, size_t count, size_t size,
           int (*compare)(const void *, const void *));
```

- base：待排序数组的起点。
- count：元素个数，不是末尾下标。
- size：单个元素的字节数。
- compare：两个元素的先后关系，返回负值、0、正值分别表示前者更小、相等、更大。

排序会移动元素，比较器应一致、可重复，不应修改正在排序的元素。`qsort` 不保证稳定排序，也不保证具体采用快速排序算法。

### 完整案例：按年龄、再按姓名排序

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Student {
    char name[20];
    int age;
};

int compare_students(const void *left, const void *right)
{
    const struct Student *a = left;
    const struct Student *b = right;
    int age_order = (a->age > b->age) - (a->age < b->age);
    if (age_order != 0) {
        return age_order;
    }
    return strcmp(a->name, b->name);
}

int main(void)
{
    struct Student students[] = {{"Wang", 20}, {"Lin", 18}, {"Chen", 20}};
    size_t count = sizeof students / sizeof students[0];
    qsort(students, count, sizeof students[0], compare_students);
    for (size_t i = 0; i < count; ++i) {
        printf("%s %d\n", students[i].name, students[i].age);
    }
    return 0;
}
```

顺序为 Lin 18、Chen 20、Wang 20。不能普遍用 `return a->age - b->age;`，两个合法 int 的差也可能溢出；关系比较再相减只得到 -1、0、1，不发生该问题。

若排序字符串指针数组，比较器收到的是“数组元素的地址”，应解释为指向字符串指针的指针，而不是直接当作字符串：

```c
int compare_names(const void *left, const void *right)
{
    const char *const *a = left;
    const char *const *b = right;
    return strcmp(*a, *b);
}
```

## 7. 模拟通用排序：字节定位与回调比较

下面复用冒泡思想，教学重点是接口，不是声称复现标准库内部算法。`unsigned char *` 按字节访问对象表示；每个元素位置是 `base + index * size`。

```c
#include <stdio.h>
#include <stddef.h>
#include <stdint.h>

int generic_bubble(void *base, size_t count, size_t size,
                   int (*compare)(const void *, const void *))
{
    if (size == 0 || count > SIZE_MAX / size) {
        return 0;
    }
    if (count < 2) {
        return 1;
    }
    if (base == NULL || compare == NULL) {
        return 0;
    }
    unsigned char *bytes = base;
    for (size_t end = count; end > 1; --end) {
        int changed = 0;
        for (size_t i = 1; i < end; ++i) {
            unsigned char *a = bytes + (i - 1) * size;
            unsigned char *b = bytes + i * size;
            if (compare(a, b) > 0) {
                for (size_t k = 0; k < size; ++k) {
                    unsigned char temporary = a[k];
                    a[k] = b[k];
                    b[k] = temporary;
                }
                changed = 1;
            }
        }
        if (!changed) {
            break;
        }
    }
    return 1;
}

int compare_ints(const void *left, const void *right)
{
    int a = *(const int *)left;
    int b = *(const int *)right;
    return (a > b) - (a < b);
}

int main(void)
{
    int values[] = {3, -2, 3, 0};
    size_t count = sizeof values / sizeof values[0];
    if (!generic_bubble(values, count, sizeof values[0], compare_ints)) {
        return 1;
    }
    for (size_t i = 0; i < count; ++i) {
        printf("%d ", values[i]);
    }
    putchar('\n');
    return 0;
}
```

输出 `-2 0 3 3`。调用者仍必须保证 base 指向真实的 count 个元素，函数无法只靠地址验证分配容量。传错元素宽度，即使乘法未溢出，也会破坏对象边界。

## 8. 练习

将学生排序改为年龄降序；用 `INT_MIN`、0、`INT_MAX` 检验整数比较器；测试通用排序的空长度、单元素和重复元素。每次换类型时，分别检查起点、数量、宽度、比较器四个参数，不要只改 compare 就结束。
