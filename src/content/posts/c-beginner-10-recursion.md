---
title: C语言初阶 10：函数递归、调用展开与递归的代价
published: 2026-11-14
description: 通过阶乘、顺序打印数字、斐波那契、二分与汉诺塔理解递归，区分终止条件、问题规模和重复计算。
tags: [C语言, 初阶, 递归]
category: C语言
series: C语言学习笔记
seriesOrder: 10
draft: false
---


## 1. 引入：怎样按原顺序输出整数各位

`1234 % 10` 先取到 4，反复取余得到的是逆序。要输出 `1 2 3 4`，可以先完成“打印 123”这个较小任务，最后打印 4。较小任务与原任务形式相同，自然适合递归。

```c
#include <stdio.h>

void print_digits(unsigned int n)
{
    if (n >= 10) {
        print_digits(n / 10);
    }
    printf("%u ", n % 10);
}

int main(void)
{
    print_digits(1234);
    putchar('\n');
    return 0;
}
```

这里输入限定为非负整数，0 也会输出一位 0。若要支持负数，还需单独处理负号和最小负数取绝对值溢出的问题。

## 2. 递归的两个核心条件

递归是函数直接或间接调用自身。一个正确递归至少要说明：

1. **终止条件**：什么情况下不再递归，能直接得到答案？
2. **规模缩小**：每次调用如何严格接近终止条件？

只写“调用自己”并不构成合理算法。没有终止条件，或参数始终不变，通常会不断占用调用资源，最终失败。

对 `print_digits`：小于 10 时无需继续拆；`n / 10` 不断去掉个位，最终必会落入 0..9。

## 3. 按调用栈理解“先递后归”

调用 `print_digits(123)` 的过程可以拆成：

```text
print_digits(123)
  等待 print_digits(12)
    等待 print_digits(1)
      输出 1，返回
    输出 2，返回
  输出 3，返回
```

每次调用都有自己的参数和局部状态。调用者会停在调用点，等被调用者完成后，继续执行后面的语句。不是“递归一调用，后面的代码就消失了”。

若把 `printf` 移到递归调用前，输出顺序就变为 `3 2 1`。**处理动作放在递归前还是递归后，决定了执行顺序。**

## 4. 案例：阶乘及输入约束

定义 `0! = 1`，`n! = n × (n - 1)!`。下面把输入限定在 0..20；标准保证的 `unsigned long long` 最小范围足以容纳 20!。

```c
#include <stdio.h>

unsigned long long factorial(unsigned int n)
{
    if (n <= 1) {
        return 1;
    }
    return n * factorial(n - 1);
}

int main(void)
{
    unsigned int n = 10;
    if (n > 20) {
        puts("示例只接受 0..20。");
        return 1;
    }
    printf("%u! = %llu\n", n, factorial(n));
    return 0;
}
```

推演 `factorial(4)`：`4 * (3 * (2 * 1)) = 24`。递归深度 O(n)，乘法次数 O(n)。同样任务可用循环，额外空间 O(1)。对于阶乘，递归便于说明定义，循环通常更直接。

扩大整数类型只能延后溢出，不能消除溢出。无符号回绕有定义，也不意味着回绕后的值仍是正确阶乘。

## 5. 斐波那契：递归简洁，未必高效

设 `F(0)=0`、`F(1)=1`，则 `F(n)=F(n-1)+F(n-2)`。直接照定义写：

```c
unsigned long long fib_slow(unsigned int n)
{
    if (n < 2) {
        return n;
    }
    return fib_slow(n - 1) + fib_slow(n - 2);
}
```

求 `F(5)` 会多次求 `F(3)`、`F(2)`，重复子问题迅速增多。时间呈指数级增长，调用深度却只有 O(n)；时间复杂度和空间复杂度要分别看。

循环每次只保留前两项，时间 O(n)，额外空间 O(1)：

```c
unsigned long long fib(unsigned int n)
{
    unsigned long long previous = 0;
    unsigned long long current = 1;
    for (unsigned int i = 0; i < n; ++i) {
        unsigned long long next = previous + current;
        previous = current;
        current = next;
    }
    return previous;
}
```

本循环会计算到 `F(n+1)`，示例可将输入限制在 `n <= 92`，以适配常见及标准最小 64 位范围。另一思路是缓存已算结果（记忆化），但同时引入了缓存空间与初始化管理。

## 6. 二分查找的递归结构

前提：数组升序排列，左右端点合法。找不到时，终止条件是区间为空。

```c
int binary_search(const int a[], int left, int right, int key)
{
    if (left > right) {
        return -1;
    }
    int mid = left + (right - left) / 2;
    if (a[mid] == key) {
        return mid;
    }
    if (a[mid] < key) {
        return binary_search(a, mid + 1, right, key);
    }
    return binary_search(a, left, mid - 1, key);
}
```

每次排除中点并减半范围，时间 O(log n)，递归空间 O(log n)。调用者要把递归结果 `return` 回去，漏掉返回会丢失找到的下标。循环版能进一步省去递归栈。

## 7. 汉诺塔：递归表达任务分解

把 n 个盘从 A 移到 C，借助 B：先把 n-1 个从 A 移到 B，再把最大盘移到 C，最后把 n-1 个从 B 移到 C。

```c
void hanoi(unsigned int n, char from, char spare, char to)
{
    if (n == 0) {
        return;
    }
    hanoi(n - 1, from, to, spare);
    printf("%c -> %c\n", from, to);
    hanoi(n - 1, spare, from, to);
}
```

`hanoi(3, 'A', 'B', 'C')` 会输出 7 步。步数满足 `T(n)=2T(n-1)+1`，即 `2^n-1`，不能因为代码短就认为效率高。递归真正的价值是准确表达问题分解。

## 8. 扫雷自动展开与访问标记

点击邻雷数为 0 的位置时，可递归访问其邻居。要先检查范围、是否已访问、是否为雷，再把当前格标为已访问，最后递归邻居。若先递归再标记，相邻两格可能互相反复调用。

大棋盘的深度优先递归可能很深，可改显式栈或队列。尾递归同样不能保证“不会栈溢出”，C 不要求编译器进行尾调用优化。

## 9. 选择递归还是循环

递归适合递归定义、树形结构和分治；单向计数、线性累积通常用循环更省空间。做选择时检查最大深度、是否重复计算、结果范围和可读性，不只看行数。

练习：手写 `print_digits(507)` 的调用与返回过程；比较求 `F(30)` 时循环和朴素递归的调用工作量；用调试器的调用堆栈观察每层参数。
