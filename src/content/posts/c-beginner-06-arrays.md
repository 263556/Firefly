---
title: C语言初阶 06：一维数组、二维数组与字符串
published: 2026-11-14
description: 用成绩统计、二分查找和棋盘理解数组，梳理初始化、遍历、内存布局、字符串终止符、sizeof 与 strlen、变长数组。
tags: [C语言, 初阶, 数组]
category: C语言
series: C语言初阶笔记
seriesOrder: 6
draft: false
---


## 1. 引入：用一组成绩代替一堆变量

如果有 30 名同学，逐个声明 `score1`、`score2` 不便遍历。数组把同类型元素连续存放，用一个名字和下标定位元素。

```c
#include <stdio.h>

int main(void)
{
    int score[] = {78, 92, 85, 67, 88};
    size_t count = sizeof score / sizeof score[0];
    int sum = 0;
    int max = score[0];

    for (size_t i = 0; i < count; ++i) {
        sum += score[i];
        if (score[i] > max) {
            max = score[i];
        }
    }
    printf("人数：%zu，最高：%d，平均：%.1f\n", count, max, (double)sum / count);
    return 0;
}
```

下标从 0 开始，5 个元素的合法下标为 0..4。最大值从首元素初始化，避免“全为负数时把 0 当最大值”的通用错误；本例数组非空，才能读取 `score[0]`。

## 2. 创建和初始化

```c
int a[5];               // 自动局部数组：元素值不确定
int b[5] = {1, 2};      // 1, 2, 0, 0, 0
int c[5] = {0};         // 全部为 0
int d[] = {10, 20, 30}; // 根据初始化项推断长度为 3
```

数组长度描述元素个数，不是总字节数。定义之后不能通过 `a = b` 整体赋值，也不能直接用 `a == b` 比较所有元素；需要逐元素处理或采用合适的库函数。

`int a[5]` 的类型是“包含 5 个 int 的数组”，不是 `int *`。数组元素连续存放，因此总大小为 `5 * sizeof(int)`。

## 3. 遍历与边界

```c
for (size_t i = 0; i < count; ++i) {
    printf("%d ", score[i]);
}
```

条件必须是 `< count`，不是 `<= count`。越界访问是未定义行为，运行时未崩溃也不能说明合法。

倒序遍历若使用 `size_t`，不能写 `i >= 0`，因为无符号数不会小于 0。可这样写：

```c
for (size_t i = count; i > 0; --i) {
    printf("%d ", score[i - 1]);
}
```

`sizeof arr / sizeof arr[0]` 只适用于此处的 `arr` 仍是真正数组。数组参数会调整为指针，函数内部必须由调用者另外传长度。

## 4. 字符数组与字符串

```c
char a[] = "abc";              // 4 个元素：a b c \0
char b[] = {'a', 'b', 'c'};     // 3 个元素，不是 C 字符串
char c[] = {'a', 'b', 'c', 0};  // 有终止符，是字符串
```

字符串以第一个 `\0` 结束；字符数组不一定是字符串。`printf("%s", b)` 或 `strlen(b)` 会越过数组寻找终止符，导致未定义行为。

| 项目 | `sizeof` | `strlen` |
| --- | --- | --- |
| 性质 | 操作符 | `<string.h>` 中的函数 |
| 计算内容 | 对象或类型占的字节数 | 终止符前的字节数 |
| 是否需要终止符 | 否 | 是 |
| 是否包含 `\0` | 数组中的终止符也占空间 | 不包含 |

```c
char s[10] = "abc";
printf("%zu %zu\n", sizeof s, strlen(s)); // 10 3
```

数组容量、当前字符串长度是两回事。`"ab\0cd"` 的数组大小为 6，字符串长度为 2，因为第一个空字符就终止了字符串。

## 5. 安全读取一行

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char name[32];
    if (fgets(name, sizeof name, stdin) == NULL) {
        return 1;
    }
    name[strcspn(name, "\n")] = '\0';
    printf("你好，%s！\n", name);
    return 0;
}
```

`fgets` 最多读容量减一的字符并追加 `\0`，能读入空格。输入过长时，余下部分仍在输入流里，实际交互程序还需要检测并清理这一行。若用 `scanf("%31s", name)`，会以空白分隔，不能读取含空格的完整姓名。字符串数组名在这里已转换为地址，不再写 `&name`。

## 6. 二维数组与行优先存储

```c
int board[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};
```

可以把它理解为“2 个元素的一维数组，每个元素又是长度为 3 的 int 数组”。连续内存顺序是 `1, 2, 3, 4, 5, 6`，先一行再下一行。

```c
for (int row = 0; row < 2; ++row) {
    for (int col = 0; col < 3; ++col) {
        printf("%d ", board[row][col]);
    }
    putchar('\n');
}
```

初始化时可省略第一维，如 `int a[][3] = {{1, 2, 3}, {4, 5, 6}};`，不能同时省略第二维，因为编译器需要知道一行占多少空间。二维数组传参时也必须正确提供列维度，不能当作 `int **`。

## 7. 案例：在有序数组中二分查找

前提是数组已经升序排列。比较中间元素后，舍弃不可能存在答案的一半区间。

```c
#include <stdio.h>

int main(void)
{
    int data[] = {1, 3, 5, 7, 9, 11, 13};
    int target = 9;
    int left = 0;
    int right = (int)(sizeof data / sizeof data[0]) - 1;
    int position = -1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (data[mid] == target) {
            position = mid;
            break;
        }
        if (data[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    printf("下标：%d\n", position); // 4；找不到为 -1
    return 0;
}
```

区间是闭区间 `[left, right]`，更新要排除已经检查的 `mid`，否则可能死循环。每轮候选范围约减半，时间复杂度为 O(log n)，顺序查找则为 O(n)。本例使用小数组和 `int` 下标，超大数组需选择能表示其长度的索引类型。

## 8. 变长数组与应用延伸

C99 引入长度由运行时值决定的变长数组（VLA），如函数内 `int a[n];`，要求计算出的长度为正。后续标准的支持情况和编译器实现存在差异，MSVC 常见 C 模式不支持 VLA。它也不是可随时扩容的动态数组，大尺寸可能消耗大量自动存储。

初学阶段跨平台示例可用固定上限数组；需要运行时分配容量时，后续学习动态内存。课件的随机点名可用二维字符数组保存姓名；若要求抽两名不同学生，还需要避免抽到同一个下标。

## 9. 自测

1. `char s[5] = "hello"` 为什么不能用于 `%s` 输出？没有为终止符留下第六个元素。
2. 改写成绩统计，另外统计及格人数，确保每个元素只遍历一次。
3. 分别查找二分数组的首元素、尾元素、不存在的元素和只有一个元素的数组。
4. 写 3×3 棋盘的输入输出，明确行列坐标与数组下标如何转换。
