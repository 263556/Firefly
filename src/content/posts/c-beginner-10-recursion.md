---
title: C语言初阶 10：函数递归、调用展开与递归的代价
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 初阶, 递归]
category: C语言
series: C语言学习笔记
seriesOrder: 10
draft: false
updated: 2026-11-16
---

> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- 递归一定要有终止条件并让参数逼近它；阶乘、斐波那契还需检查数值溢出。C 不保证尾调用优化，尾递归仍可能耗尽调用栈。

## 本章目标

1. 理解函数递归的核心要素

2. 学会思考递归问题，拆解递归代码

3. 能够编写函数递归的程序

4. 理解递归可能存在的问题，以及递归和循环的特点

## 1. 什么是递归

递归是编程中的一种技术，指的是一个函数在其定义内部调用自身。（递归一定是依赖于函数的）史上最简单的递归程序

**课件代码 10-001 · PDF 第 1 页**

```c
#include <stdio.h>

int main()
{
    printf("hehe\n");
    main(); //main函数自己调用自己
    return 0;
}
```

这个程序是函数递归，但是是错误的递归程序，因为会导致死递归，最终出现栈溢出的现象。

![第10章第1页配图](/images/c-language-course/10-p01-fig01.png)

![第10章第1页配图](/images/c-language-course/10-p01-fig02.png)

### 1.1 递归的解释

可以把它理解为“俄罗斯套娃”或“镜子中的镜子”把一个大型复杂问题层层转化为一个与原问题相似，但规模较小的子问题来求解；直到子问题不能再被拆分，递归就结束了。（大事化小）

![第10章第2页配图](/images/c-language-course/10-p02-fig01.png)

![第10章第2页配图](/images/c-language-course/10-p02-fig02.png)

俄罗斯套娃 《盗梦空间》剧照

### 1.2 递归的核心要素

一个正确的递归函数必须包含两个关键部分：

- 递归调用（递推阶段）：函数自己调用自己，每次调用时，问题的规模都应该比上一次更小，逐步

逼近一个最简单的“基础情况”。

- 终止条件（基础情况）：一个不再进行递归调用、能直接返回结果的特定条件。如果没有终止条

件，递归会无限进行下去，最终导致栈溢出错误。

## 2. 递归举例

### 2.1 举例1：求n的阶乘

题目：计算正整数n的阶乘（不考虑溢出，假设计算机结果在int的取值范围内）一个正整数的阶乘（factorial）是所有小于及等于该数的正整数的积，并且0的阶乘为1。

自然数n的阶乘写作 n!0! = 1 1! = 1 2! = 2 \* 1 3! = 3 \* 2 \* 1 4! = 4 \* 3 \* 2 \* 1 5! = 5 \* 4 \* 3 \* 2 \* 1= 5 \* 4!

#### 2.1.1 分析和代码实现

有了上面的概念，我们很容易想到，n的阶乘的递归公式，如下：

1, n = 0⎧n! =⎨n × (n − 1)!, n ≥ 1⎩

- 当 n &gt; 0 的时候， n! = n \* (n-1)! ， n! 转换成了有关 (n-1)! 的问题；同时 (n-1)!

和 n! 是相似的问题，同时规模在变小；

- 当n在不断变小的过程中，当n==0的时候，就不再递归，0!就是1。

这就是典型的递归场景，这时候我们就写一个函数 fact(n) 来计算n!，再结合上面的公式自然地就能写出下面代码：

**课件代码 10-002 · PDF 第 3 页**

```c
#include <stdio.h>

int Fact(int n)
{
    if(n <= 0)
        return 1;
    else
        return n * Fact(n-1);
}

int main()
{
    int n = 0;
    scanf("%d", &n);
    int ret = Fact(n);
    printf("%d\n", ret);
    return 0;
}
```

在Fact函数内，每次递归调用的时候n会变成n-1，在逐渐变小，逼近n==0这个终止条件，递归就结束了。

#### 2.1.2 画图推演

当n==5，求5!时，递推和回归过程的演示：

![第10章第4页配图](/images/c-language-course/10-p04-fig01.png)

### 2.2 举例2：顺序打印一个整数的每一位

输入一个正整数m，按照顺序打印整数的每一位。

比如：

输入：1234 输出：1 2 3 4输入：520 输出：5 2 0

#### 2.2.1 分析和代码实现

这个题目，放在我们面前，首先想到的是，怎么得到这个数的每一位呢？

如果n是1位数，直接打印n就行n是超过1位数的话，就得拆分n的每一位1234%10就能得到4，然后1234/10得到123，这就相当于去掉了4然后继续对123%10，就得到了3，再除10去掉3，以此类推不断的 %10 和 /10 操作，直到1234的每一位都得到；但是这里有个问题就是得到的数字顺序是倒着的上面的推理中，我们发现其实一个数字的最低位是最容易得到的，通过%10就能得到那我们就把最后1位分离出来，把一个n位数看做: 前面的n-1位 + 最后一位比如: 1234，拆分为123 和 4，这样就把4位数，转化成3位数 + 1位数 的问题这就是递归的大事化小那我们假设想写一个函数Print来打印n的每一位，如下表示：

**课件代码 10-003 · PDF 第 4,5 页**

```c
Print(n)
如果n是1234，那Print(1234) 能打印1234的每一位

其中1234中的4可以通过%10得到，那么
Print(1234)就可以拆分为两步：
1. Print(1234/10)  //打印123的每一位
2. printf(1234%10) //打印4
完成上述2步，那就完成了1234每一位的打印

那么Print(123)又可以拆分为Print(123/10) + printf(123%10)
```

以此类推下去，就有

**课件代码 10-004 · PDF 第 5 页**

```c
   Print(1234)
==>Print(123)                       + printf(4)
==>Print(12)            + printf(3)
==>Print(1)  + printf(2)
==>printf(1)
```

直到被打印的数字变成一位数的时候，就不需要再拆分，递归结束。

那么代码完成也就比较清楚：

**课件代码 10-005 · PDF 第 5 页**

```c
void Print(int n)
{
    if(n > 9)
    {
        Print(n / 10);
    }
    printf("%d ", n % 10);
}

int main()
{
    int m = 0;
    scanf("%d", &m);
    Print(m);
    return 0;
}
```

输入和输出结果：

![第10章第6页配图](/images/c-language-course/10-p06-fig01.png)

在这个解题的过程中，我们就是使用了大事化小的思路把 Print(1234) 打印1234每一位，拆解为首先 Print(123) 打印123的每一位，再打印得到的4把 Print(123) 打印123每一位，拆解为首先 Print(12) 打印12的每一位，再打印得到的3直到 Print 打印的是一位数，直接打印就行。

#### 2.2.2 画图推演

以1234每一位的打印来推演一下

![第10章第6页配图](/images/c-language-course/10-p06-fig02.png)

### 2.3 举例3：求第n个斐波那契数

斐波那契数列大家都听过，下面这个序列就是斐波那契数列：

![第10章第6页配图](/images/c-language-course/10-p06-fig03.png)

斐波那契数列的特点是，第0个数是0，第1个数是1，往后的数字都是前2个数字之和现在给定一个n的值(n从0开始)，计算出第n个斐波那契数（不考虑溢出）。

#### 2.3.1 分析和代码实现

在斐波那契数列中，只有前2个数字是必须已知的，后期的数字都是可以计算得到的。

0, n = 0⎧F(n) = 1 ,  n = 1⎨F(n − 1) + F(n − 2), n ≥ 2⎩根据这个公式轻松就能得到下面的代码：

**课件代码 10-006 · PDF 第 7 页**

```c
#include <stdio.h>

int Fib(int n)
{
    if(n == 0)
        return 0;
    else if(n == 1)
        return 1;
    else
        return Fib(n-1) + Fib(n-2);
}

int main()
{
    int n = 0;
    scanf("%d", &n);
    int ret = Fib(n);
    printf("%d\n", ret);
    return 0;
}
```

#### 2.3.2 程序性能分析

针对上面的代码，我们去测试，如果n较小的时候，程序很正常；但是当 n 较大的时候，比如 n==50 的时候，需要很长时间才能算出结果，这个计算所花费的时间，是我们很难接受的，这也说明递归的写法是非常低效的，那是为什么呢？

![第10章第8页配图](/images/c-language-course/10-p08-fig01.png)

看上图，随着递归不断的展开，我们很容易就能发现，在递归的过程中会有重复计算，而且递归层次越深，冗余计算就会越多。我们可以写代码统计一下冗余计算的数据，会非常的惊人。

**课件代码 10-007 · PDF 第 8,9 页**

```c
#include <stdio.h>
int count = 0;

int Fib(int n)
{
    if(n == 3) //统计第3个斐波那契数被重复计算的次数
        count++;
    if(n == 0)
        return 0;
    else if(n == 1)
        return 1;
    else
        return Fib(n-1) + Fib(n-2);
}


int main()
{
    int n = 0;
    scanf("%d", &n);
    int ret = Fib(n);
    printf("%d\n", ret);
    printf("\ncount = %d\n", count);
    return 0;
}
```

![第10章第9页配图](/images/c-language-course/10-p09-fig01.png)

这里我们看到了，使用递归实现的代码在计算第40个斐波那契数的时候，第3个斐波那契数就被重复计算了39088169次，正是因为这些大量重复的计算，让程序的性能很差。那我们看到了

📌 1. 第n个斐波那契数的计算，使用递归来实现并非最佳的选择

2. 递归过程中如果反复计算子问题，会导致指数级时间复杂度，最终让程序的性能堪忧

#### 2.3.3 栈溢出

其实递归程序除了可能影响性能之外，还会存在栈溢出的风险。

📌 1. 在C语言程序中每一次函数调用，都需要为本次函数调用在内存的栈区，申请一块内存空

间来保存函数调用期间的各种局部变量的值，这块空间被称为运行时堆栈，或者函数栈帧。

2. 函数如果不返回，函数对应的栈帧空间就一直占用，所以如果函数调用中存在递归调用的

话，每一次递归函数调用都会开辟属于自己的栈帧空间，直到函数递归不再继续，开始回归，才逐层释放栈帧空间。如果采用函数递归的方式完成代码，递归层次太深，就会浪费太多的栈帧空间，也可能引起栈溢出（stack overflow）的问题。

3. 关于函数栈帧的详细内容，请看加餐内容《函数栈帧的创建和销毁》章节

**课件代码 10-008 · PDF 第 9,10 页**

```c
#include <stdio.h>

int count = 0;

void test()
{
    count++;
    printf("当前深度: %d\n", count);
    int buffer[1000] = {0};  // 占用栈空间
    test();        // 无限递归
}

int main()
{
    test();
    return 0;
}
```

### 2.4 递归和循环

我们发现递归程序有可能导致栈溢出问题或者性能的问题，那什么解决办法吗？

通常会把递归程序改造成循环的方式，比如：

#### 2.4.1 求阶乘

递归写法 循环写法

**课件代码 10-009 · PDF 第 10 页**

```c
int Fact(int n)
{
    if(n <= 0)
        return 1;
    else
        return n * Fact(n-1);
}
```

**课件代码 10-010 · PDF 第 10 页**

```c
int Fact(int n)
{
    int i = 0;
    int ret = 1;
    for(i = 1; i <= n; i++)
    {
        ret *= i;
    }
    return ret;
}
```

#### 2.4.2 求斐波那契数

递归写法 循环写法

**课件代码 10-011 · PDF 第 10 页**

```c
int Fib(int n)
{
    if(n == 0)
        return 0;
    else if(n == 1)
        return 1;
    else
        return Fib(n-1) + Fib(n- 2);
}
```

**课件代码 10-012 · PDF 第 10,11 页**

```c
int Fib(int n)
{
    int a = 1;
    int b = 1;
    int c = 1;
    while(n > 2)
    {
        c = a+b;
        a = b;
        b = c;
        n--;
    }
    return c;
}
```

当然还有一种优化递归程序中栈溢出问题的方法是，采用尾递归的方式，但是尾递归不一定可靠，有兴趣的同学下来可以研究一下。

📌 AI提示词：

1. 什么是尾递归？举例讲解C语言中尾递归代码的案例

2. 尾递归能解决递归的什么问题？一定能解决吗？

#### 2.4.3 递归和循环的选择

我们看到的许多问题是以递归的形式进行解释的，这只是因为它比非递归的形式更加清晰，但是这些问题的循环实现往往比递归实现效率更高。

📌 当一个问题非常复杂，难以使用循环的方式实现时，此时递归实现的简洁性便可以补偿它所

带来的运行时开销。一般情况下，递归的深度&lt;100层，并且不会造成大量冗余计算的时候，可以大胆地使用递归写法。

当这个问题使用递归解决存在明显缺陷的时候，就需要考虑改造成循环的方式。

递归经常会使用到：树/图遍历、分治算法、回溯算法中，大家在后期学习《数据结构和算法》的知识时候，再逐步去体会学习。

📌 AI提示词：关于递归，在C语言程序设计中有哪些使用建议？

## 3. 递归拓展学习

借助于AI研究，搞清楚算法思想尝试自行阅读代码

### 3.1 二分查找的递归实现

![第10章第12页配图](/images/c-language-course/10-p12-fig01.png)

**课件代码 10-013 · PDF 第 12,13,14 页**

```c
#include <stdio.h>

// 递归二分查找函数
// arr: 有序数组（升序）
// left: 左边界索引
// right: 右边界索引
// target: 要查找的目标值
// 返回值: 找到返回索引，未找到返回-1
int binarySearch(int arr[], int left, int right, int target)
{
    // 基本情形：未找到目标值
    if (left > right)
        return -1;
    // 计算中间索引（避免溢出）
    int mid = left + (right - left) / 2;

    if (arr[mid] == target) // 找到目标值
        return mid;
    else if (arr[mid] > target)   // 目标值在左半部分
        return binarySearch(arr, left, mid - 1, target);
    else // 目标值在右半部分
        return binarySearch(arr, mid + 1, right, target);
}

// 包装函数，简化调用
int search(int arr[], int size, int target)
{
    return binarySearch(arr, 0, size - 1, target);
}

int main()
{
    int arr[] = {1, 3, 5, 7, 9, 11, 13, 15, 17, 19};
    int size = sizeof(arr) / sizeof(arr[0]);
    int target;

    printf("有序数组: ");
    for (int i = 0; i < size; i++)
    {
        printf("%d ", arr[i]);
    }
    printf("\n");

    // 测试查找
    target = 7;
    int result = search(arr, size, target);
    if (result != -1)
    {
        printf("元素 %d 找到，索引为: %d\n", target, result);
    }
    else
    {
        printf("元素 %d 未找到\n", target);
    }

    target = 10;
    result = search(arr, size, target);
    if (result != -1)
    {
        printf("元素 %d 找到，索引为: %d\n", target, result);
    }
    else
    {
        printf("元素 %d 未找到\n", target);
    }

    return 0;
}
```

### 3.2 汉诺塔问题

A柱上有n个盘子，要借助于B柱，挪到C柱上挪动的过程中，在柱子上要保证上的盘子小，下面的盘子大如果有1个盘子：A-&gt;C如果有2个盘子：A-&gt;B，A-&gt;C, B-&gt;C如果有3个盘子：A-&gt;C, A-&gt;B，C-&gt;B, A-&gt;C, B-&gt;A，B-&gt;C，A-&gt;C如果有n个盘子：.....

![第10章第15页配图](/images/c-language-course/10-p15-fig01.png)

演示网站：https://gallery.selfboot.cn/zh/algorithms/hanoitower

**课件代码 10-014 · PDF 第 15,16 页**

```c
#include <stdio.h>

// 汉诺塔递归函数
//pos1上的n个盘子，借助于pos2，移动到pos3上
void hanoi(int n, char pos1, char pos2, char pos3)
{
    if (n == 0)
        return;
    // 将上面n-1个圆盘从起始柱移动到辅助柱
    hanoi(n - 1, pos1, pos3, pos2);
    // 将最大的圆盘从起始柱移动到目标柱
    printf("%c -> %c\n", pos1, pos3);
    // 将n-1个圆盘从辅助柱移动到目标柱
    hanoi(n - 1, pos2, pos1, pos3);
}
int main()
{
    int n = 0;

    printf("请输入汉诺塔的层数: ");
    scanf("%d", &n);

    printf("\n移动过程如下：\n");
    hanoi(n, 'A', 'B', 'C');  // A为起始柱，B为辅助柱，C为目标柱return 0;
}
```

【波哥算法】汉诺塔代码视频讲解

### 3.3 扫雷游戏的拓展

📌 AI提示词：

1. 在尽量少改动原有代码的基础上，实现在排雷过程中实现展开一片的功能。

2. 在尽量少改动代码的基础上，使用raylib库，将扫雷游戏改造成带界面的版本，raylib我已

经安装好了，路径是：C:\\raylib\\raylib-5.5\_win64\_mingw-w64 raylib库链接：https://github.com/raysan5/raylib/releases

## 4. AI加餐课

10.AI相关的岗位介绍

## 课件中的参考链接

- [参考资料 1](https://www.bilibili.com/video/BV1swVz6HE28?vd_source=b4b93e10f67adc83a17eb0a996681a93&p=2&spm_id_from=333.788.videopod.episodes)
- [参考资料 2](https://bitejiuyeke.feishu.cn/wiki/FANrwXMWkiQ9d3kfzZCci1l4nqb)
