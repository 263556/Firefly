---
title: C语言进阶 14：数组名、数组指针与二维数组传参
published: 2026-11-14
description: 从排序和二维成绩表理解数组退化，区分数组指针与指针数组，推导步长、sizeof、字符串指针和多级指针表达式。
tags: [C语言, 进阶, 数组, 指针]
category: C语言
series: C语言进阶笔记
seriesOrder: 14
draft: false
---

> 依据鹏哥 C 语言进阶第 14 章整理。

## 1. 引入：把成绩表交给函数

一维数组传给排序函数，函数能改变原数组，却不能在内部用 `sizeof` 算出元素数。二维数组看起来也能写 `a[i][j]`，参数却不能随意换成 `int **`。这两个疑问都要从“表达式的类型”入手。

```c
int a[4] = {1, 2, 3, 4};
int *p = a;
int (*whole)[4] = &a;
```

`p` 指向一个 `int` 元素，`whole` 指向整个四元素数组。它们可能打印出相同的起始地址，但类型和加一的意义不同。

## 2. 数组名什么时候变成指针

数组表达式在多数语境中转换为首元素指针。常见例外是作为 `sizeof` 的操作数、作为一元 `&` 的操作数；字符串字面量用于初始化字符数组也是需要区分的场景。

| 表达式 | 类型/含义 | `sizeof` 的结果 |
| --- | --- | --- |
| `a` 用于普通表达式 | 转换为 `int *` | 单独写 `sizeof a` 时不转换 |
| `a + 0` | `int *` | 指针大小 |
| `*a` | 首元素，`int` | `sizeof(int)` |
| `&a` | `int (*)[4]` | 数组指针大小 |
| `*&a` | 原数组 | `4 * sizeof(int)` |
| `&a[0] + 1` | 第二元素地址，`int *` | 指针大小 |

`a + 1` 跳过一个元素；`&a + 1` 跳过整个数组。`&a + 1` 可形成尾后指针，但不能解引用它。不要因为观察到相同地址，就把不同指针类型视为可任意互换。

数组不是可修改的指针变量，`a = p` 不合法。数组的元素可以逐项修改，整个数组本身不能被赋予一个新地址。

## 3. 下标与解引用的对应关系

在有效范围内，`a[i]` 等价于 `*(a + i)`，`p[i]` 等价于 `*(p + i)`。这解释了指针为什么也能使用下标，但不代表数组和指针是同一种对象。

```c
#include <stdio.h>

int main(void)
{
    int a[] = {10, 20, 30};
    int *p = a;
    printf("%d %d %d\n", a[1], *(a + 1), p[1]);
    printf("array=%zu, element=%zu, pointer=%zu\n",
           sizeof a, sizeof a[0], sizeof p);
    return 0;
}
```

访问值都为 20，大小则由当前实现决定。不必为了“效率”把所有下标改成复杂指针表达式，优先选择能清晰表达数据结构的写法。

## 4. 字符数组与字符串字面量

```c
char first[] = "hello";
char second[] = "hello";
const char *view = "hello";
```

`first` 和 `second` 是两个独立数组，可修改各自元素。`view` 保存字符串字面量首字符的地址，不保存一份“装在指针里的字符串”。修改字符串字面量是未定义行为，应使用 `const char *` 表达只读意图。

内容比较用 `strcmp(first, second) == 0`，不是比较指针是否相等。两个相同字面量是否共享存储，由实现决定；不能保证两个 `const char *` 一定指向同一个位置。

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char a[] = "hello";
    char b[] = "hello";
    a[0] = 'H';
    printf("%s / %s\n", a, b); // Hello / hello
    printf("same content: %d\n", strcmp(a, b) == 0); // 0
    return 0;
}
```

## 5. 一维数组传参的本质

参数声明中的 `int data[]` 调整为 `int *data`，不会复制一个新数组。数组元素通过地址被访问，长度则必须单独约定。

```c
void print_array(const int data[], size_t count);
// 与下面的参数类型一致：
void print_array(const int *data, size_t count);
```

不能在函数体里用 `sizeof data / sizeof data[0]` 恢复调用者数组长度。只读接口使用 `const`，可变接口需要明确说明会修改原内容。

### 案例：带提前结束的冒泡排序

每趟比较相邻元素，把当前最大值移到未排序区域末尾。若某一趟完全没有交换，整个数组已经有序。

```c
#include <stdio.h>
#include <stddef.h>

void bubble_sort(int data[], size_t count)
{
    for (size_t end = count; end > 1; --end) {
        int changed = 0;
        for (size_t i = 1; i < end; ++i) {
            if (data[i - 1] > data[i]) {
                int temporary = data[i - 1];
                data[i - 1] = data[i];
                data[i] = temporary;
                changed = 1;
            }
        }
        if (!changed) {
            break;
        }
    }
}

int main(void)
{
    int data[] = {5, 1, 4, 2, 2};
    size_t count = sizeof data / sizeof data[0];
    bubble_sort(data, count);
    for (size_t i = 0; i < count; ++i) {
        printf("%d ", data[i]);
    }
    putchar('\n');
    return 0;
}
```

结果 `1 2 2 4 5`。从 `end > 1` 开始避免 `size_t` 的 `count - 1` 在空区间下溢。最坏时间 O(n²)，已有序时一趟 O(n)，额外空间 O(1)；只在严格大于时交换，因此保持相等元素的相对次序。

## 6. 数组指针与指针数组

| 声明 | 先结合什么 | 含义 |
| --- | --- | --- |
| `int *p[3]` | `p[3]` | 三元素数组，每个元素是 `int *` |
| `int (*p)[3]` | `(*p)` | 指针，指向三元素 int 数组 |

`[]` 比声明中的 `*` 结合得更紧，因此数组指针的括号必不可少。读复杂声明时从变量名向外看，先辨认“它本身是数组还是指针”，再看所包含或指向的类型。

## 7. 二维数组为什么传数组指针

```c
int table[2][3] = {{1, 2, 3}, {4, 5, 6}};
int (*row)[3] = table;
```

二维数组的首元素是第一行，类型是 `int[3]`；转换后自然是 `int (*)[3]`。`row + 1` 前进一整行。`table[r][c]` 等价于 `*(*(table + r) + c)`，两次偏移分别定位行和列。

```c
#include <stdio.h>
#include <stddef.h>

void print_table(int (*table)[3], size_t rows)
{
    for (size_t r = 0; r < rows; ++r) {
        for (size_t c = 0; c < 3; ++c) {
            printf("%d ", table[r][c]);
        }
        putchar('\n');
    }
}

int main(void)
{
    int table[2][3] = {{1, 2, 3}, {4, 5, 6}};
    print_table(table, 2);
    return 0;
}
```

参数也可写 `int table[][3]`。行数可运行时传入，但这个固定列数接口必须匹配 3 列，不能把 5 列数组强转后传入。`int **` 指向的是指针对象，真实二维数组没有每行一根指针的布局，因此不兼容。

## 8. 指针数组模拟多行数据

```c
int first[] = {1, 2};
int second[] = {3, 4, 5};
int *rows[] = {first, second};
size_t lengths[] = {2, 3};
```

`rows[0][1]` 先取第一个指针，再访问它指向的数组。每一行可以长度不同，甚至位于完全不同的存储区域；应像 `lengths` 这样同时记录各行长度，不能假定整片数据是一块连续矩形。

字符串列表同理可写 `const char *names[] = {"Lin", "Chen"};`。若使用指向列表的指针 `const char **p = names;`，`++p` 移到下一个字符串指针，`*p + 1` 才是在某个字符串中前进一个字符。

## 9. 拆表达式与练习

遇到 `**++p`，先记录 p 的类型和当前目标，再拆成 `++p;` 与 `**p` 理解。多层 `*` 不会自动“跳过几行”，每层都取决于所指对象的类型。

练习：画出真实二维数组和指针数组的存储图；测试空长度、单元素、全相等、逆序数组的冒泡排序；解释 `sizeof a`、`sizeof(a + 0)`、`sizeof *&a`。不要用越界后退、强制改类型等技巧代替合法的数组索引。
