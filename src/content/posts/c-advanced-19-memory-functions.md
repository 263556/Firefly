---
title: C语言进阶 19：memcpy、memmove、memset 与 memcmp
published: 2026-11-14
description: 用数组复制和删除元素理解按字节操作，推导重叠拷贝方向，模拟内存函数并区分对象值与字节表示。
tags: [C语言, 进阶, 内存函数]
category: C语言
series: C语言学习笔记
seriesOrder: 19
draft: false
---

> 四个接口均声明于 `<string.h>`，长度单位均为字节。

## 1. 引入：为什么 strcpy 不能复制整数数组

`strcpy` 直到第一个空字符才停止，适用于字符串。整数、结构体和任意二进制数据中可能含零字节，它们的结束位置不能由 `\0` 决定。内存函数接受明确的字节数，不解释数据的业务含义。

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    int source[] = {10, 0, 30, 40};
    int destination[4];
    memcpy(destination, source, sizeof source);
    for (size_t i = 0; i < 4; ++i) {
        printf("%d ", destination[i]);
    }
    putchar('\n');
    return 0;
}
```

输出 `10 0 30 40`。`sizeof source` 包含整个数组，不要硬编码 16，也不要误传元素个数 4。通过函数参数接收 source 时，它可能已调整为指针，不能继续照搬这个 sizeof。

## 2. 接口对照

| 接口 | 动作 | 返回值 | 主要前提 |
| --- | --- | --- | --- |
| `memcpy(dst, src, n)` | 复制 n 字节 | dst | 源目标不重叠 |
| `memmove(dst, src, n)` | 复制 n 字节 | dst | 允许重叠，仍须有足够容量 |
| `memset(dst, value, n)` | 重复写同一个字节 | dst | 目标可写且足够长 |
| `memcmp(a, b, n)` | 按 unsigned char 比较 | 负、零、正 | 两边都有 n 个可读字节 |

它们不会自动分配内存，不保证“字节数写大也安全”，也不会自动补字符串终止符。对标准接口，不要假定 n 为 0 就一定允许传入无效或空指针；自定义接口可另行明确空长度规则。

## 3. memcpy：复制的是表示，不是逻辑对象图

```c
void *memcpy(void *destination, const void *source, size_t count);
```

目的区域至少有 count 个可写字节，源区域至少有 count 个可读字节，且实际区域不重叠。复制到相同位置也不要拿 memcpy 的重叠约束冒险，直接不做操作或使用 memmove。

如果结构体含指针，memcpy 只复制指针值，不会为指向的数据建立副本；这与结构体整体赋值的浅拷贝问题相同。保存跨平台文件时，复制内存也不能自动解决字节序、填充和指针无意义的问题。

### 模拟实现的关键

将两边转换为字符指针，逐字节复制，因为标准 C 不支持直接对 void 指针做算术。使用 unsigned char 可观察和复制任意对象表示。

```c
void *copy_bytes(void *destination, const void *source, size_t count)
{
    unsigned char *out = destination;
    const unsigned char *in = source;
    for (size_t i = 0; i < count; ++i) {
        out[i] = in[i];
    }
    return destination;
}
```

这是练习版本，不要用同名函数覆盖标准库。接口仍要求有效范围和不重叠，真实库往往使用更高效的平台相关实现。

## 4. memmove：重叠时为什么要倒着复制

把 `ABCDE` 前四个字符复制到向右一位的位置，若从左向右：先写 B 为 A，再从已经被改过的 B 读取，最终可能得到一串 A。

```text
原始：A B C D E
目标：A A B C D
正确方向：先移动 D，再 C，再 B，再 A
```

当目标落在源区间后部时，应从后向前；向左移动可以从前向后。标准 memmove 的语义等价于先保存到临时区域再写入，不要求实现真的分配一个临时数组。

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char text[] = "ABCDE";
    memmove(text + 1, text, 4);
    puts(text); // AABCD，原来的结尾终止符保持不变
    return 0;
}
```

memmove 不会替你增长数组。若连终止符也往后移动，需要额外容量；不要根据上例就把 4 随意改成 `sizeof text`。

## 5. 不比较无关指针大小的练习实现

常见教材代码直接 `if (destination < source)` 判断方向，但对不相关对象做指针大小比较没有普遍的标准保证。下面用相等性判断目标是否落在源区间后部，避免这一问题；仍要求两边各有 count 个有效字节。

```c
#include <stdio.h>
#include <stddef.h>

void *move_bytes(void *destination, const void *source, size_t count)
{
    unsigned char *out = destination;
    const unsigned char *in = source;
    if (count == 0 || out == in) {
        return destination;
    }
    int copy_backward = 0;
    for (size_t i = 1; i < count; ++i) {
        if (out == in + i) {
            copy_backward = 1;
            break;
        }
    }
    if (copy_backward) {
        for (size_t i = count; i > 0; --i) {
            out[i - 1] = in[i - 1];
        }
    } else {
        for (size_t i = 0; i < count; ++i) {
            out[i] = in[i];
        }
    }
    return destination;
}

int main(void)
{
    char text[] = "ABCDE";
    move_bytes(text + 1, text, 4);
    puts(text); // AABCD
    move_bytes(text, text + 1, 4);
    text[4] = '\0';
    puts(text); // ABCD
    return 0;
}
```

这种写法为了说明可移植判断，增加了一次线性扫描。工程中直接调用标准 memmove，避免为了“手写优化”引入更复杂的边界问题。

## 6. memset：设置每个字节，而不是每个元素

`memset(p, value, n)` 将 value 转换为 unsigned char，再在 n 个字节中重复它。

```c
int values[3];
memset(values, 1, sizeof values);
```

这不会把每个 int 设为 1。在常见 8 位字节、32 位 int 环境中，会形成每字节都是 01 的表示。需要每项为 1，应写逐元素循环。

整型全零字节可表示 0，但不能泛化为所有指针类型的 NULL 或所有浮点类型的 0.0。初始化结构体优先使用 `struct Item item = {0};`，由语言按成员类型完成零初始化。

memset 也不是通用“安全擦除秘密”的保证：编译器可能消除后续不再使用的写入。需要敏感数据清理时选择平台或标准提供的专用接口，并核对可用性。

## 7. memcmp：字节字典序不等于数值大小

memcmp 找第一个不同字节，按 unsigned char 解释后决定符号，返回值不保证是 -1 或 1。它不会在零字节处停止。

```c
unsigned char first[] = {1, 0, 9};
unsigned char second[] = {1, 0, 8};
int result = memcmp(first, second, sizeof first); // 大于 0
```

不能用 memcmp 判断两个整数的数值先后：小端下低位字节先被比较，可能与数值大小相反。结构体可能含填充，两个逻辑相同的结构体也不一定有相同字节表示；浮点数还涉及正负零、NaN 等情况。需要逻辑相等时，按字段或元素比较。

## 8. 案例：从数组中删除一项

删除第 index 项，后面的元素整体左移一格，源目标重叠，所以用 memmove。

```c
#include <stdio.h>
#include <string.h>
#include <stddef.h>

int erase_int(int values[], size_t *count, size_t index)
{
    if (values == NULL || count == NULL || index >= *count) {
        return 0;
    }
    size_t tail = *count - index - 1;
    if (tail != 0) {
        memmove(values + index, values + index + 1, tail * sizeof values[0]);
    }
    --*count;
    return 1;
}

int main(void)
{
    int values[] = {10, 20, 30, 40};
    size_t count = 4;
    if (!erase_int(values, &count, 1)) {
        return 1;
    }
    for (size_t i = 0; i < count; ++i) {
        printf("%d ", values[i]);
    }
    putchar('\n');
    return 0;
}
```

输出 `10 30 40`。物理数组容量仍为 4，逻辑长度变成 3，末尾旧值不属于有效元素。调用者须保证 count 不超过实际容量；这个函数不会缩小分配。

## 9. 练习

对 move_bytes 测试左右重叠、完全相同、不重叠和零长度；对删除函数测试首项、尾项、唯一一项和非法下标。写下每次传入的字节数公式，再检查是否误把元素数当成字节数。
