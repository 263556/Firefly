---
title: C语言初阶 04：分支、循环与流程控制
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 初阶, 流程控制]
category: C语言
series: C语言学习笔记
seriesOrder: 4
draft: false
updated: 2026-11-16
---

> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- 数学式 `0 <= x <= 100` 在 C 中应写成 `0 <= x && x <= 100`。if/while 后多余分号、悬空 else、continue 跳过递增等均为需要辨析的错误示范。

## 本章目标

1. 掌握分支语句if.else，switch

2. 掌握关系操作符、条件操作符、逻辑操作符在代码中的应用，能够熟练使用

3. 掌握循环语句：for、while、do.while，以及break和continue

4. 理解并熟练应用循环嵌套解决问题

5. 锻炼自己阅读代码的能力

C语言是结构化的程序设计语言，这里的结构指的是顺序结构、选择结构、循环结构，C语言是能够实现这三种结构的，其实我们如果仔细分析，我们日常所见的事情都可以拆分为这三种结构或者这三种结构的组合。

我们可以使用 if 、 switch 实现分支结构，使用 for 、 while 、 do while 实现循环结构。

## 1. if语句

### 1.1 if

if 语句的语法形式如下:

**课件代码 04-001 · PDF 第 1 页**

```text
if ( 表达式 )
    语句
```

表达式成立（为真），则语句执行，表达式不成立（为假），则语句不执行在C语言中，0为假，非0表示真，也就是表达式的结果如果是0，则语句不执行，表达式的结果如果不是0，则语句执行。

例子：输入一个整数，判断是否为奇数

**课件代码 04-002 · PDF 第 1,2 页**

```c
#include <stdio.h>
int main()
{
    int num = 0;
    scanf("%d", &num);
    if(num % 2 == 1)
         printf("%d 是奇数\n", num);
    return 0;
}
```

非0 ==1(真)表达式 num%2语句 打印为0 ==0（假）结束 结束if语句执行流程 奇数判断

### 1.2 else

一个正整数不是奇数，那就是偶数了，现在给定一个任意的正整数，要判断是奇数还是偶数，怎么表示呢？

这里就需要 if...else... 语句了，语法形式如下：

**课件代码 04-003 · PDF 第 2 页**

```text
if ( 表达式 )
    语句1
else
    语句2
```

举例：输入一个正整数，如果是奇数打印是奇数，否则打印偶数。

**课件代码 04-004 · PDF 第 3 页**

```c
#include <stdio.h>
int main()
{
    int num = 0;
    scanf("%d", &num);
    if(num % 2 == 1)
        printf("%d 是奇数\n", num);
    else
        printf("%d 是偶数\n", num);
    return 0;
}
```

练习：

输入一个成绩，成绩 &gt;= 60就输出：拿捏拿捏，否则就输出：补考补考~

**课件代码 04-005 · PDF 第 3 页**

```c
#include <stdio.h>

int main()
{
    int score = 0;
    scanf("%d", &score);
    if(score >= 60)
        printf("拿捏拿捏~\n");
    else
        printf("补考补考~\n");
    return 0;
}
```

### 1.3 多条语句

默认在 if 和 else 语句中默认都只控制一条语句，比如：

**课件代码 04-006 · PDF 第 3,4 页**

```c
#include <stdio.h>
int main()
{
    int score = 0;
    scanf("%d", &score);
    if(score >= 60)
        printf("拿捏拿捏~\n");
        printf("庆祝一下\n");
    return 0;
}
```

上面的代码，你会发现输入的值不管是&gt;=60 还是&lt;60，"庆祝一下" 都会打印在屏幕上。

![第4章第4页配图](/images/c-language-course/04-p04-fig01.png)

![第4章第4页配图](/images/c-language-course/04-p04-fig02.png)

这是因为 if 语句只能控制一条语句，就是 printf("拿捏拿捏~\\n"); ，if语句为真，则打印 拿捏拿捏~ ，if语句为假，则不打印，对于 printf("庆祝一下\\n"); 是独立存在的，不管if语句的条件的真假，都会被执行。

那如果我们要 if 语句同时控制 2 条语句，怎么办呢？那就要使用 {} 将代码括起来， else 后也可以跟上大括号。如下：

**课件代码 04-007 · PDF 第 4,5 页**

```c
#include <stdio.h>
int main()
{
    int score = 0;
    scanf("%d", &score);
    if(score >= 60)
    {
        printf("拿捏拿捏~\n");
        printf("庆祝一下\n");
    }
    return 0;
}

#include <stdio.h>
int main()
{
    int score = 0;
    scanf("%d", &score);
    //if和else后面的{}可以括起来多条语句，是一个整体，这个大括号也叫程序块，或者复合语句
    if(score >= 60)
    {
        printf("拿捏拿捏~\n");
        printf("庆祝一下\n");
    }
    else
    {
        printf("补考补考~\n");
        printf("好好复习\n");
    }
    return 0;
}
```

> **原例说明：** 这个代码框并列展示两份 main 实现，练习时分别保存、分别编译，不要放进同一源文件。

### 1.4 多分支

在 if else 语句中， else 可以与另一个 if 语句连用，构成多分支的场景。

**课件代码 04-008 · PDF 第 5 页**

```c
if(表达式1)
    语句1;
else if(表达式1)
    语句2;
else if(表达式3)
    语句3;
else
    语句4;
```

比如：总价“满50减15，满30减8，满20减5，不满20元，不参与活动”，怎么实现？

**课件代码 04-009 · PDF 第 5,6 页**

```c
#include <stdio.h>

int main()
{
    float total = 0;
    scanf("%f", &total);
    if (total >= 50)
        printf("减15元，实付：%.2f\n", total - 15);
    else if (total >= 30)
        printf("减8元，实付：%.2f\n", total - 8);
    else if (total >= 20)
        printf("减5元，实付：%.2f\n", total - 5);
    else
        printf("不满足满减，去隔壁拼个单吧。\n");
    return 0;
}
```

练习：暗恋对象回消息速度和心情指数如果回复时间小于1分钟，秒回！今天一定有戏如果回复时间在1分钟到10分钟，还行，ta可能在忙如果回复时间在10分钟到1小时，ta可能没看到信息……我去打游戏了如果回复时间超过1小时，别等了，代码不香吗？

参考代码：

**课件代码 04-010 · PDF 第 6 页**

```c
#include <stdio.h>

int main()
{
    int minutes = 0; // 回复间隔
    scanf("%d", &minutes);
    if (minutes <= 1)
        printf("秒回！有戏\n");
    else if (minutes <= 10)
        printf("还行，ta可能在忙\n");
    else if (minutes <= 60)
        printf("ta可能没看到信息……我去打游戏了\n");
    else
        printf("别等了，代码不香吗？\n");

    return 0;
}
```

### 1.5 嵌套if语句

有时候，我们需要在if语句中嵌套if语句，形成多层嵌套的判断。

比如：评奖学金资格

1. 成绩 &gt;= 85，没有挂科，没有记过处分，有奖学金

2. 其中任意一个条件不满足，都不能评奖

怎么实现呢？

**课件代码 04-011 · PDF 第 6,7 页**

```c
#include <stdio.h>

int main()
{
    int gpa = 90;
    int has_fail = 0;   // 0没挂科
    int has_warn = 0;   // 0没处分

    if (gpa >= 85)
    {
        if (has_fail == 0)
        {
            if (has_warn == 0)
            {
                printf("恭喜！奖学金到手，请客！\n");
            }
            else
            {
                printf("警告处分，不能评奖\n");
            }
        }
        else
        {
            printf("挂过科，不能评奖\n");
        }
    }
    else
    {
        printf("绩点不够，继续努力。\n");
    }
    return 0;
}
```

### 1.6 悬空else问题

如果有多个 if 和 else ，可以记住这样一条规则， else 总是与前面最近的、还没有配对的 if结合。

看下面代码，执行的结果是什么？

**课件代码 04-012 · PDF 第 7,8 页**

```c
#include <stdio.h>
int main()
{
    int a = 0;
    int b = 2;
    if(a == 1)
        if(b == 2)
            printf("hehe\n");
    else
        printf("haha\n");
    return 0;
}
```

程序运行的结果是啥？

很多初学者，上来以判断 a 是 0，不等于 1 ，那就执行 else 子句，打印 haha但是当你去运行代码，输出的结果是：啥都不输出。

![第4章第8页配图](/images/c-language-course/04-p08-fig01.png)

为什么呢？

这就是悬空 else 的问题，如果有多个 if 和 else ，可以记住这样一条规则， else 总是与前面最近的、还没有配对的 if 匹配。

上面的代码排版，让 else 和第一个 if 语句对齐，让我们以为 else 是和第一个if匹配的，当 if语句不成立的时候，自然想到的就是执行 else 子句，打印 haha ，但实际上 else 是和第二个 if进行匹配的，这样后边的 if...else 语句是嵌套在第一个 if 语句中的，如果第一个 if 语句就不成立，嵌套 if 和 else 就没机会执行了，最终啥都不打印。

如果代码改成下面这样就更加容易理解了

**课件代码 04-013 · PDF 第 8,9 页**

```c
#include <stdio.h>
int main()
{
    int a = 0;
    int b = 2;
    if(a == 1)
    {
        if(b == 2)
            printf("hehe\n");
        else
            printf("haha\n");
    }
    return 0;
}
```

或者如果我们希望else确实和第一个if匹配，可以这样修改代码：

**课件代码 04-014 · PDF 第 9 页**

```c
#include <stdio.h>
int main()
{
    int a = 0;
    int b = 2;
    if(a == 1)
    {
        if(b == 2)
            printf("hehe\n");
    }
    else
    {
         printf("haha\n");
    }
    return 0;
}
```

只要带上适当的大括号，代码的逻辑就会更加的清晰，所以大家以后在写代码的时候要注意括号的使用，让代码的可读性更高。

## 2. 关系操作符

C 语言用于比较的表达式，称为 “关系表达式”（relational expression），里面使用的运算符就称为“关系运算符”（relational operator），主要有下面6个。

![第4章第10页配图](/images/c-language-course/04-p10-fig01.png)

关系表达式通常返回 0 或 1 ，表示真假。

C 语言中， 0 表示假，所有非零值表示真。比如， 20 &gt; 12 返回 1 ， 12 &gt; 20 返回 0 。

关系表达式常用于 if 或 while 结构。

**课件代码 04-015 · PDF 第 10 页**

```c
if (x == 3)
{
     printf("x is 3.\n");
}
```

📌 相等运算符 == 与赋值运算符 = 是两个不一样的运算符，不要混淆

**课件代码 04-016 · PDF 第 10 页**

```text
if (x = 3) ...
```

上面示例中，原意是 x == 3 ，但是不小心写成 x = 3 。这个式子表示对变量 x 赋值 3 ，它的返回值为 3 ，所以 if 判断总是为真。

为了防止出现这种错误，有的程序员喜欢将变量写在等号的右边。

**课件代码 04-017 · PDF 第 10 页**

```text
if (3 == x) ...
```

这样的话，如果把 == 误写成 = ，编译器就会报错。

**课件代码 04-018 · PDF 第 10,11 页**

```c
/* 报错 */
if (3 = x) ...
```

📌 多个关系运算符不宜连用！！

**课件代码 04-019 · PDF 第 11 页**

```text
i < j < k
```

上面示例中，连续使用两个小于运算符。这是合法表达式，不会报错，但是通常达不到想要的结果，即不是保证变量 j 的值在 i 和 k 之间。因为关系运算符是从左到右计算，所以实际执行的是下面的表达式。

**课件代码 04-020 · PDF 第 11 页**

```text
(i < j) < k
```

上面式子中， i &lt; j 返回 0 或 1 ，所以最终是 0 或 1 与变量 k 进行比较。如果想要判断变量 j的值是否在 i 和 k 之间，应该使用下面的写法。

**课件代码 04-021 · PDF 第 11 页**

```text
i < j && j < k       //&& 是并且的意思，下面会介绍
```

比如：我们输入一个年龄，如果年龄在18岁~36岁之间，我们输出⻘年。

如果我们这样写

**课件代码 04-022 · PDF 第 11 页**

```c
#include <stdio.h>
int main()
{
    int age = 0;
    scanf("%d", &age);
    if(18 <= age <= 36)
    {
        printf("⻘年\n");
    }
    return 0;
}
```

当我们输入10的时候，依然输出⻘年，如下图：

![第4章第12页配图](/images/c-language-course/04-p12-fig01.png)

这是因为，我们先拿18和age中存放的10比较，表达式18&lt;=10为假， 18&lt;=age 的结果是0，再拿0和36比较，0&lt;=36为真，所以打印了 ⻘年 ，所以即使当age是10的时候，也能打印 ⻘年 ，逻辑上是有问题，这个代码应该怎么写呢？

**课件代码 04-023 · PDF 第 12 页**

```c
#include <stdio.h>
int main()
{
    int age = 0;
    scanf("%d", &age);
    if(age >= 18 && age <= 36)
    {
        printf("⻘年\n");
    }
    return 0;
}
```

以上就是关系操作符，我们需要掌握的，剩下的只要按照字面意思理解使用就行，没有特别注意的。

## 3. 条件操作符

条件操作符也叫三目操作符，需要接受三个操作数的，形式如下：

**课件代码 04-024 · PDF 第 12 页**

```text
exp1 ? exp2 : exp3
```

条件操作符的计算逻辑是：如果 exp1 为真， exp2 计算，计算的结果是整个表达式的结果；如果exp1 为假， exp3 计算，计算的结果是整个表达式的结果。

练习：使用条件表达式实现找两个数中较大值。

代码块 代码块

**课件代码 04-025 · PDF 第 12,13 页**

```c
#include <stdio.h>
int main()
{
    int a = 0;
    int b = 0;
    int m = 0;
    scanf("%d %d", &a, &b);
    if(a > b)
        m = a;
    else
        m = b;
    printf("%d\n", m);

    return 0;
}
```

**课件代码 04-026 · PDF 第 12,13 页**

```c
#include <stdio.h>
int main()
{
    int a = 0;
    int b = 0;
    int m = 0;
    scanf("%d %d", &a, &b);
    m = (a > b ? a : b);
    printf("%d\n", m);

    return 0;
}
```

🎈 条件操作符也能嵌套使用，但是不建议嵌套太深，如果嵌套，要注意换行对齐的方式来写。

**课件代码 04-027 · PDF 第 13 页**

```c
#include <stdio.h>

int main()
{
    int minutes = 15;
    scanf("%d", &minutes);
    printf("心情：%s\n",
        minutes <= 1  ? "秒回！有戏" :
        minutes <= 10 ? "还行，ta可能在忙" :
        minutes <= 60 ? "ta可能没看到信息……我去打游戏了" : "别等了，代码不香吗？");

    return 0;
}
```

## 4. 逻辑操作符：&amp;&amp; , \|\| , ！

逻辑运算符提供逻辑判断功能，用于构建更复杂的表达式，主要有下面三个运算符。

![第4章第13页配图](/images/c-language-course/04-p13-fig01.png)

### 4.1 逻辑取反运算符 ！

![第4章第14页配图](/images/c-language-course/04-p14-fig01.png)

逻辑反操作比如，我们有一个变量叫 flag ,如果flag为假，要做一个什么事情，就可以这样写代码：

**课件代码 04-028 · PDF 第 14 页**

```c
#include <stdio.h>
int main()
{
    int flag = 0;
    if(!flag)
    {
        printf("do something\n");
    }
    return 0;
}
```

如果 flag 为真， !flag 就是假，如果 flag 为假， !flag 就是真所以上面的代码的意思就是 flag 为假，执行if语句中的代码。

### 4.2 逻辑与运算符

![第4章第14页配图](/images/c-language-course/04-p14-fig02.png)

逻辑与&amp;&amp; 就是与运算符，也是并且的意思， &amp;&amp; 是一个双目操作符，使用的方式是 a&amp;&amp;b , &amp;&amp; 两边的表达式都是真的时候，整个表达式才为真，只要有一个是假，则整个表达式为假。

比如：如果我们说月份是3月到5月，是春天，那使用代码怎么体现呢？

**课件代码 04-029 · PDF 第 15 页**

```c
int month = 0;
scanf("%d", &month);
if(month >= 3 && month <= 5)
{
    printf("春季\n");
}
```

这里表达的意思就是month既要大于等于3，又要小于等于5，必须同时满足。

### 4.3 逻辑或运算符

![第4章第15页配图](/images/c-language-course/04-p15-fig01.png)

逻辑或\|\| 就是或运算符，也就是或者的意思， \|\| 也是一个双目操作符，使用的方式是 a \|\| b ， \|\|两边的表达式只要有一个是真，整个表达式就是真，两边的表达式都为假的时候，才为假。

比如：我们说一年中月份是12月或者1月或者2月是冬天，那么我们怎么使用代码体现呢？

**课件代码 04-030 · PDF 第 15 页**

```c
int month = 0;
scanf("%d", &month);
if(month == 12 || month == 1 || month == 2)
{
    printf("冬季\n");
}
```

### 4.4 练习1：闰年的判断

输入一个年份year，判断year是否是闰年闰年判断的规则：

1. 能被4整除并且不能被100整除是闰年

2. 能被400整除是闰年

**课件代码 04-031 · PDF 第 16 页**

```c
#include <stdio.h>
//代码1
int main()
{
    int year = 0;
    scanf("%d", &year);
    if(year % 4 == 0 && year % 100 != 0)
        printf("是闰年\n");
    else if(year % 400 == 0)
        printf("是闰年\n");

    return 0;
}

//代码2 - 使用逻辑操作符
int main()
{
    int year = 0;
    scanf("%d", &year);
    if((year % 4 == 0 && year % 100 != 0) || (year % 400 == 0))
        printf("是闰年\n");

    return 0;
}
```

> **原例说明：** 原框包含两个独立 main，用来比较控制流，请拆开运行。

### 4.5 练习2：石头剪刀布

本题目为自行阅读代码，自行练习的题目

**课件代码 04-032 · PDF 第 16,17 页**

```c
char player = 's'; // 石头
char computer = 'j'; // 剪刀

if (player == computer)
{
    printf("平局！再来一局。\n");
}
else if ((player == 's' && computer == 'j') ||
           (player == 'j' && computer == 'b') ||
           (player == 'b' && computer == 's'))
{
    printf("你赢了！\n");
}
else
{
    printf("你输了，电脑果然厉害。\n");
}
```

### 4.6 短路

C语言逻辑运算符还有一个特点，它总是先对左侧的表达式求值，再对右边的表达式求值，这个顺序是保证的。

如果左边的表达式满足逻辑运算符的条件，就不再对右边的表达式求值。这种情况称为“短路”。

如前面的代码:

**课件代码 04-033 · PDF 第 17 页**

```text
if(month >= 3 && month <= 5)
```

表达式中&amp;&amp; 的左操作数是 month &gt;= 3 ，右操作数是 month &lt;= 5 ,当左操作数 month &gt;= 3 的结果是0的时候，即使不判断 month &lt;= 5 ，整个表达式的结果也是0（不是春季）。

所以，对于&amp;&amp;操作符来说，左边操作数的结果是0的时候，右边操作数就不再执行。

对于 \|\| 操作符是怎么样呢？我们结合前面的代码：

**课件代码 04-034 · PDF 第 17 页**

```text
if(month == 12 || month == 1 || month == 2)
```

如果month == 12，则不用再判断month是否等于1或者2，整个表达式的结果也是1（是冬季）。

所以， \|\| 操作符的左操作数的结果不为0时，就无需执行右操作数。

像这种仅仅根据左操作数的结果就能知道整个表达式的结果，不再对右操作数进行计算的运算称为短路求值。

练习：阅读代码，计算代码输出的结果

**课件代码 04-035 · PDF 第 18 页**

```c
//代码1
#include <stdio.h>

int main()
{
    int i = 0, a = 0, b = 2, c = 3, d = 4;
    i = a++ && ++b && d++;
    printf("a = %d\nb = %d\nc = %d\nd = %d\n", a, b, c, d);
    return 0;
}
//上面的代码执行结果是什么？如果把a的值改成1，结果又是什么？

//代码2
#include <stdio.h>

int main()
{
    int i = 0, a = 1, b = 2, c = 3, d = 4;
    i = a++ || ++b || d++;
    printf("a = %d\nb = %d\nc = %d\nd = %d\n", a, b, c, d);
    return 0;
}
//上面的代码执行结果是什么？如果把a的值改成0，结果又是什么？
```

> **原例说明：** 原框包含两个独立 main，分别观察输出，不是一个完整源文件。

练习2：

游戏中，同时持有 ‘钥匙’ 且 血量&gt;30%，或被诅咒了，才能进入隐藏房间。

阅读代码，体会逻辑操作符和短路现象

**课件代码 04-036 · PDF 第 18,19 页**

```c
#include <stdio.h>
int main()
{
    int has_key = 1; //是否有钥匙
    int hp = 25;     //血量
    int is_cursed = 1; //是否被诅咒
    if ((has_key && hp > 30) || is_cursed)
    {
        printf("开启隐藏房间——诅咒之屋\n");
    }
    else
    {
        printf("前方的门打不开\n");
    }
    return 0;
}
```

## 5. switch 语句

除了 if 语句外，C语言还提供了 switch 语句来实现分支结构。

🎈 AI提示词：有了if语句，为什么还需要switch呢？

语法糖：指的是编程语言中那些让代码写起来更简洁、更易读，但并没有引入新功能的语法。

switch 语句是一种特殊形式的 if...else 结构，用于判断条件有多个结果的情况。它把多重的else if 改成更易用、可读性更好的形式。

**课件代码 04-037 · PDF 第 19 页**

```c
switch (表达式)
{
  case value1: 语句1
  case value2: 语句2
  ...
  default: statement
}
```

上面代码中，根据switch后边的 表达式 产生的不同的值，执行相应的 case 分支，如果找不到对应的值，就执行 default 分支。

🎈 • switch 后的 表达式 必须是整型表达式

- case 后的值，必须是整形常量表达式

### 5.1 if语句和switch语句的对比

练习：输入任意一个整数值，计算除3之后的余数如果使用if语句完成，如下：

**课件代码 04-038 · PDF 第 20 页**

```c
#include <stdio.h>

int main()
{
    int n = 0;
    scanf("%d", &n);
    if(n % 3 == 0)
        printf("整除，余数为0\n");
    else if(n % 3 == 1)
        printf("余数是1\n");
    else
        printf("余数是2\n");
    return 0;
}
```

如果使用 switch 语句改写，就可以是这样的：

**课件代码 04-039 · PDF 第 20 页**

```c
#include <stdio.h>

int main()
{
    int n = 0;
    scanf("%d", &n);
    switch(n % 3)
    {
    case 0:
        printf("整除，余数为0\n");
        break;
    case 1:
        printf("余数是1\n");
        break;
    case 2:
        printf("余数是2\n");
        break;
    }
    return 0;
}
```

上述的代码中，我们要注意的点有：

1. case 和后边的数字之间必须有空格

2. 每一个 case 语句中的代码执行完成后，需要加上 break ，才能跳出这个 switch 语句。

### 5.2 switch语句中的break

前面的代码中，如果我们去掉case语句中的break，会出现什么情况呢？

**课件代码 04-040 · PDF 第 21 页**

```c
#include <stdio.h>

int main()
{
    int n = 0;
    scanf("%d", &n);
    switch(n % 3)
    {
    case 0:
        printf("整除，余数为0\n");
    case 1:
        printf("余数是1\n");
    case 2:
        printf("余数是2\n");
    }
    return 0;
}
```

测试一组，看一下结果：

![第4章第21页配图](/images/c-language-course/04-p21-fig01.png)

我们发现，7除以3本来余数是1，但是我们发现程序运行的结果多了一行“余数是2”的打印。

这是为什么呢？

原因是 switch 语句也是分支效果的，只有在 switch 语句中使用 break 才能在跳出 switch 语句，如果某一个 case 语句的后边没有 break 语句，代码会继续往下执行，有可能执行其他 case语句中的代码，直到遇到 break 语句或者 switch 语句结束。就比如上面的代码就执行了 case 2 中的语句。

所以在 switch 语句中 break 语句是非常重要的，能实现真正的分支效果。

当然， break 也不是每个 case 语句都得有，这就得根据实际情况来看了。

练习：

输入一个1~7的数字，打印对应的星期几例如：

输入：1 输出：星期一输入：2 输出：星期二输入：3 输出：星期三输入：4 输出：星期四输入：5 输出：星期五输入：6 输出：星期六输入：7 输出：星期天参考代码如下：

**课件代码 04-041 · PDF 第 22,23 页**

```c
#include <stdio.h>
int main()
{
    int day = 0;
    scanf("%d", &day);
    switch(day)
    {
        case 1：
            printf("星期一\n");
            break;
        case 2:
            printf("星期二\n");
            break;
        case 3:
            printf("星期三\n");
            break;
        case 4:
            printf("星期四\n");
            break;
        case 5:
            printf("星期五\n");
            break;
        case 6:
            printf("星期六\n");
            break;
        case 7:
            printf("星期天\n");
            break;
    }
    return 0;
}
```

> **原例说明：** 课件 case 标签中使用了全角冒号 `：`；实际编译时改为半角 `:`。原例保留以便对照。

如果需求发生变化，变为：

1. 输入1~5，输出的是“早八人，早八魂。”;

2. 输入6~7，输出“周末！关掉闹钟，睡到11点。”

那参考代码如下：

**课件代码 04-042 · PDF 第 23 页**

```c
#include <stdio.h>

int main()
{
    int day = 0;
    scanf("%d", &day);

    switch(day)
    {
        case 1：
        case 2:
        case 3:
        case 4:
        case 5:
            printf("早八人，早八魂\n");
            break;
        case 6:
        case 7:
            printf("周末！关掉闹钟，睡到11点\n");
            break;
    }
    return 0;
}
```

> **原例说明：** 课件 case 标签中的全角 `：` 应改为 C 语法使用的 `:`。

上面的练习中，我们发现应该根据实际的情况，来在代码中觉得是否使用 break ，或者在哪里使用break ，才能正确完成实际的需求。

### 5.3 switch语句中的default

在使用 switch 语句的时候，我们经常可能遇到一种情况，比如 switch 后的表达式中的值无法匹配代码中的 case 语句的时候，这时候要不就不做处理，要不就得在 switch 语句中加入default 子句。

**课件代码 04-043 · PDF 第 24 页**

```c
switch (expression)
{
  case value1: statement
  case value2: statement
  default: statement
}
```

switch 后边的 expression 的结果不是 value1 ，也不是 value2 的时候，就会执行default 子句。

就比如前面做的打印星期的练习，如果 day 的输入不是1~7的值，如果我们要提示：输入错误，则可以这样完成代码：

**课件代码 04-044 · PDF 第 24,25 页**

```c
#include <stdio.h>

int main()
{
    int day = 0;
    scanf("%d", &day);

    switch(day)
    {
        case 1：
        case 2:
        case 3:
        case 4:
        case 5:
            printf("早八人，早八魂\n");
            break;
        case 6:
        case 7:
            printf("周末！关掉闹钟，睡到11点\n");
            break;
        default:
            printf("输入错误\n");
            break;
    }
    return 0;
}
```

> **原例说明：** case 标签中的冒号应为半角 `:`；多条 case 连续匹配时还需根据目标检查 break。

### 5.4 switch语句中的case和default的顺序问题

在 switch 语句中 case 子句和 default 子句有要求顺序吗？ default 只能放在最后吗？

其实，在 switch 语句中 case 语句和 default 语句是没有顺序要求的，只要你的顺序是满足实际需求的就可以。

不过我们通常是把 default 子句放在最后处理的。

## 6. while循环

C语言提供了3种循环语句， while 就是其中一种，接下来就介绍一下 while 语句。

while 语句的语法结构和 if 语句非常相似。

### 6.1 if 和 while的对比

代码块 代码块

**课件代码 04-045 · PDF 第 25 页**

```c
if(表达式)
    语句;

if(表达式)
{
    语句;
}
```

**课件代码 04-046 · PDF 第 25 页**

```c
while(表达式)
    语句;

while(表达式)
{
    语句;//如果循环体想包含更多的语句，可以加上大 括号
}
```

你可以对比来看一下，具体写个代码吧代码块 代码块

**课件代码 04-047 · PDF 第 25,26 页**

```c
//代码1
#include <stdio.h>

int main()
{
    //if后边条件满足，打印一次 hehe
    if(1)
        printf("hehe\n");
    return 0;
}
```

**课件代码 04-048 · PDF 第 25,26 页**

```c
//代码2
#include <stdio.h>

int main()
{
    //while后边的条件恒成立，死循环的打印 hehe
    while(1)
        printf("hehe\n");
    return 0;
}
```

这就是他们的区别， while 语句是可以实现循环效果的。

### 6.2 while语句的执行流程

值为0表达式值不为0 break continue语句结束while语句的执行流程图首先上来就是执行判断表达式，表达式的值为0，循环直接结束；表达式的值不为0，则执行循环语句，语句执行完后再继续判断，是否进行下一次判断。

### 6.3 while 循环的实践

练习：在屏幕上打印 1~10 的值参考代码：

**课件代码 04-049 · PDF 第 26,27 页**

```c
#include <stdio.h>
int main()
{
    int i = 1;
    while(i <= 10)
    {
        printf("%d ", i);
        i = i + 1;
    }
    return 0;
}
```

🎈 while循环的特点就类似于：

1. 你有一个女朋友，有一天她生气了，你只有买奶茶她才可能不生气

2. 但是具体买多少次奶茶，不知道，你得一直买到她不生气为止

3. 为了防止买多了，你每次买奶茶之前都问她还生气吗？

**课件代码 04-050 · PDF 第 27 页**

```c
while(还生气呢)
{
     买奶茶;
     心情好一点;
}
```

### 6.4 练习1

输入一个正的整数，逆序打印这个整数的每一位例如：

输入：1234，输出：4 3 2 1输入：521，输出：1 2 5题目解析

1. 要想得到n的最低位，可以使用n%10的运算，得到的余数就是最低位，如：1234%10得到4

2. 要想去掉n的最低位，找出倒数第二位，则使用 n = n / 10 操作就可以去掉最低位的，如：

n=1234/10得到123，123相较于1234就去掉了最低位，123%10就得到倒数第二位3。

3. 循环1和2两个步骤，在n变成0之前，就能到所有的位。

参考代码

**课件代码 04-051 · PDF 第 27,28 页**

```c
#include <stdio.h>

int main()
{
    int n = 0;
    scanf("%d", &n);
    while(n > 0)
    {
        printf("%d ", n % 10);
        n /= 10;
    }
    return 0;
}
```

### 6.5 练习2

输入一个正整数，打印所有小于或者等于这个数的，正偶数本题目为自行阅读代码，自行练习的题目参考代码：

**课件代码 04-052 · PDF 第 28 页**

```c
#include <stdio.h>

int main()
{
    int n = 0;
    scanf("%d", &n);//存放输入的数字

    int i = 2;  // 从最小的正偶数开始
    while (i <= n)
    {
        printf("%d ", i);
        i += 2;  // 每次跳2，保证全是偶数
    }
    printf("\n");

    return 0;
}
```

### 6.6 练习3

模拟游戏打boss时，攻击掉血的场景本题目为自行阅读代码，自行练习的题目参考代码：

**课件代码 04-053 · PDF 第 29 页**

```c
#include <stdio.h>
int main()
{
    int boss_hp = 1000; //血量值
    int attack = 150; //每次攻击掉血
    while (boss_hp > 0)
    {
        //防止最后一次攻击，血量变成负值
        int real_attack = (boss_hp < attack) ? boss_hp : attack;
        printf("你对Boss发动攻击，造成 %d 点伤害！\n", real_attack);
        boss_hp -= real_attack;
        printf("Boss剩余血量：%d\n", boss_hp);
    }
    printf("Boss倒下了！掉落传说装备！\n");
    return 0;
}
```

## 7. for循环

### 7.1 语法形式

for 循环是三种循环中使用最多的， for 循环的语法形式如下：

**课件代码 04-054 · PDF 第 29 页**

```c
//形式1
for(表达式1; 表达式2; 表达式3)
    语句;

//形式2
for(表达式1; 表达式2; 表达式3)
{
    语句;//如果循环体想包含更多的语句，可以加上大括号
}
```

表达式1 用于循环变量的初始化表达式2 用于循环结束条件的判断表达式3 用于循环变量的调整

### 7.2 for循环的执行流程

表达式1：初始化==0（假）表达式2：判断!=0 表达式3：调整（真）break continue循环语句结束for循环的执行流程首先执行 表达式1 初始化循环变量，接下来就是执行 表达式2 的判断部分， 表达式2 的结果如果==0，则循环结束； 表达式2 的结果如果!=0则执行循环语句，循环语句执行完后，再去执行表 达式3 ,调整循环变量，然后再去 表达式2 的地方执行判断， 表达式2 的结果是否为0，决定循环是否继续。

整个循环的过程中，表达式1初始化部分只被执行1次，剩下的就是表达式2、循环语句、表达式3在循环。

### 7.3 for循环的实践

练习：在屏幕上打印1~10的值参考代码：

**课件代码 04-055 · PDF 第 30,31 页**

```c
#include <stdio.h>

int main()
{
    int i = 0;
    for(i = 1; i <= 10; i++)
    {
        printf("%d ", i);
    }

    return 0;
}
```

🎈 for循环的特点就类似于：

1. 你有一个女朋友，有一天她生气了，你只有买奶茶她才可能不生气

2. 但是她一般会告你，买够几杯就不生气了

3. 只要你买够了，就不生气了

### 7.4 while和for对比

![第4章第31页配图](/images/c-language-course/04-p31-fig01.png)

![第4章第31页配图](/images/c-language-course/04-p31-fig02.png)

for 和 while 在实现循环的过程中都有初始化、判断、调整这三个部分，但是 for 循环的三个部分非常集中，便于代码的维护，而如果代码较多的时候 while 循环的三个部分就比较分散，所以从形式上 for 循环要更优一些。

### 7.5 for循环的特殊写法

1. for循环中的三个表达式可以选择性省略，判断部分被省略，意味着判断部分恒成立，可能会造成死

循环。

代码块 代码块

**课件代码 04-056 · PDF 第 31,32 页**

```c
#include <stdio.h>

int main()
{
    for(;;)
    {
        printf("hehe\n");
    }
    return 0;
}
```

**课件代码 04-057 · PDF 第 31,32 页**

```c
#include <stdio.h>
int main()
{
    int i = 1;
    for(; i <= 10; i++)
    {
        printf("%d ", i);
    }
    return 0;
}
```

2. C99标准中，for循环支持新写法，循环变量可以在初始化部分来创建，这样写的循环变量，只能在

循环内部使用，出了循环就不能使用了。多个循环如果使用同一个循环变量就不会干扰了。

**课件代码 04-058 · PDF 第 32 页**

```c
#include <stdio.h>
int main()
{
    for(int i = 1; i <= 10; i++)
    {
        printf("%d ", i);
    }
    return 0;
}
```

3. 多个变量控制for循环

**课件代码 04-059 · PDF 第 32 页**

```c
#include <stdio.h>
int main()
{
    int a = 0;
    int b = 0;
    for (b = 0, a = 10; b < a; b++, a--)
    {
        printf("b=%d, a=%d\n", b, a);
    }
    return 0;
}
```

初始化部分的 b = 0, a = 10 和调整部分的 b++, a-- 是逗号表达式。

逗号表达式，就是用逗号隔开的多个表达式。

**课件代码 04-060 · PDF 第 32 页**

```text
exp1, exp2, exp3, …expN
```

逗号表达式从左向右依次执行；整个表达式的结果是最后一个表达式的结果。

**课件代码 04-061 · PDF 第 33 页**

```c
//代码1
int a = 1;
int b = 2;
int c = (b = 4, a = b + 10, b = a + 1);//逗号表达式
c是多少？

//代码2
if (a = b + 1, c = a / 2, d > 0) //从左向右计算，起判断作用的是最右边的d>0
```

### 7.6 练习1

计算1~100之间3的倍数的数字之和

**课件代码 04-062 · PDF 第 33,34 页**

```c
#include <stdio.h>

int main()
{
    int i = 0;
    int sum = 0;
    for(i = 1; i <= 100; i++)
    {
        if(i % 3 == 0)
            sum += i;
    }
    printf("%d\n", sum);
    return 0;
}


//小小的优化
//如果能直接产生3的倍数的数字就省去了多余的循环和判断
#include <stdio.h>

int main()
{
    int i = 0;
    int sum = 0;
    for(i = 3; i <= 100; i += 3)
    {
         sum += i;
    }
    printf("%d\n", sum);
    return 0;
}
```

> **原例说明：** 这个框包含两份独立 main，用于比较实现；请拆开编译。

### 7.7 练习2

计算机银行存款结余银行利息是3%，本金10000元计算出5年中，每一年年底，本金 + 利息的结余本题目为自行阅读代码，自行练习的题目参考代码

**课件代码 04-063 · PDF 第 34 页**

```c
#include <stdio.h>
int main()
{
    double money = 10000;
    int year = 0;
    for (year = 1; year <= 5; year++)
    {
        money *= 1.03;  // 年利率3%
        printf("第 %d 年：%.2f 元\n", year, money);
    }

    return 0;
}
```

## 8. do-while 循环

### 8.1 语法形式

在循环语句中 do while 语句的使用最少，它的语法如下：

**课件代码 04-064 · PDF 第 34 页**

```c
do
    语句;
while(表达式);
```

while 和 for 这两种循环都是先判断，条件如果满足就进入循环，执行循环语句，如果不满足就跳出循环；而 do while 循环则是先直接进入循环体，执行循环语句，然后再执行 while 后的判断表达式，表达式为真，就会进行下一次，表达式为假，则不再继续循环。

### 8.2 do while循环的执行流程

break continue语句!=0（真）判断表达式==0（假）结束do while 循环的执行流程在 do while 循环中先执行图上的“语句”，执行完语句，在去执行“判断表达式”，判断表达式的结果是!=0，则继续循环，执行循环语句；判断表达式的结果==0，则循环结束。

所以在 do while 语句中循环体是至少执行一次的，这是 do while 循环比较特殊的地方。

### 8.3 do while循环的实例

在屏幕上打印1~10的值

**课件代码 04-065 · PDF 第 35,36 页**

```c
#include <stdio.h>
int main()
{
    int i = 1;
    do
    {
        printf("%d ", i);
        i = i + 1;
    }while(i <= 10);

    return 0;
}
```

![第4章第36页配图](/images/c-language-course/04-p36-fig01.png)

一般 do while 使用在循环体至少被执行一次的场景下，所以较少一些。

🎈 do while循环的特点就类似于：

1. 你有一个女朋友，有一天她生气了，你只有买奶茶她才可能不生气

2. 你不管三七二十一，上来就先买奶茶

3. 买完后，再看是不是还生气，如果还生气，就继续买

### 8.4 练习1

输入一个正整数，计算这个整数是几位数？

例如：

输入：1234 输出：4输入：12 输出：2输入：0 输出：1

**课件代码 04-066 · PDF 第 36,37 页**

```c
#include <stdio.h>

int main()
{
    int n = 0;
    scanf("%d", &n);
    int cnt = 0;
    do
    {
        cnt++;
        n = n / 10;
    } while (n);
    printf("%d\n", cnt);

    return 0;
}
```

这里并非必须使用 do while 语句，但是这个代码就比较适合使用 do while 循环，因为n即使是0，也是1位数，要统计位数的。

### 8.5 练习2

1. 写代码确保只能输入一个正整数，如果是0或者负数，则提示"输入错误，请输入正整数"，直到

正确输入

2. 然后计算这个整数是几位数

**课件代码 04-067 · PDF 第 37 页**

```c
#include <stdio.h>
int main()
{
    int n = 0;
    do
    {
        scanf("%d", &n);
        if (n <= 0)
            printf("\a输入错误，请输入正整数\n");
    } while (n <= 0);

    int cnt = 0;
    do
    {
        cnt++;
        n = n / 10;
    } while (n);
    printf("%d\n", cnt);

    return 0;
}
```

## 9. break和continue语句

在循环执行的过程中，如果某些状况发生的时候，需要提前终止循环，这是非常常见的现象。C语言中提供了 break 和 continue 两个关键字，就是应用到循环中的。

- break 的作用是用于永久的终止循环，只要 break 被执行，直接就会跳出循环，继续往后执

行。

- continue 的作用是跳过本次循环中 continue 后边的代码，在 for 循环和 while 循环中有

所差异的。

🎈 生活化举例：

比如，你们在KTV唱歌，计划唱3个小时

1. 遇到不喜欢的歌，直接跳过，继续唱下一曲，就是continue

2. 如果突然停电了，即使时间还没到，没法继续再唱了，就是break

### 9.1 while循环中的break和continue

#### 9.1.1 break举例

**课件代码 04-068 · PDF 第 38 页**

```c
#include <stdio.h>
int main()
{
    int i = 1;
    while(i <= 10)
    {
        if(i == 5)
            break;//当i等于5后，就执行break，循环就终止了
        printf("%d ", i);
        i = i + 1;
    }
    return 0;
}
```

执行的结果：

![第4章第38页配图](/images/c-language-course/04-p38-fig01.png)

打印了1,2,3,4后，当i等于5的时候，循环在 break 的地方终止，不再打印，不再循环。

所以 break 的作用就是永久的终止循环，只要 break 被执行， break 外的第一层循环就终止了。

那以后我们在循环中，想在某种条件下终止循环，则可以使用 break 来完成我们想要的效果。

#### 9.1.2 continue举例

continue 是继续的意思，在循环中的作用就是跳过本次循环中 continue 后边的代码，继续进行下一次循环的判断。

上面的代码，如果把 break 换成 continue 会是什么结果呢？

**课件代码 04-069 · PDF 第 39 页**

```c
#include <stdio.h>
int main()
{
    int i = 1;
    while(i <= 10)
    {
        if(i == 5)
            continue;
            //当i等于5后，就执行continue，直接跳过continue的代码，去循环的判断的地方
            //因为这里跳过了i = i+1，所以i一直为5，程序陷入和死循环
        printf("%d ", i);
        i = i + 1;
    }
    return 0;
}
```

到这里我们就能分析出来， continue 可以帮助我们跳过某一次循环 continue 后边的代码，直接到循环的判断部分，进行下一次循环的判断，如果循环的调整是在 continue 后边的话，可能会造成死循环。

### 9.2 for循环中的break和continue

#### 9.2.1 break 举例

其实和 while 循环中的 break 一样， for 循环中的 break 也是用于终止循环的，不管循环还需要循环多少次，只要执行到了 break ，循环就彻底终止，我们上代码。

**课件代码 04-070 · PDF 第 39,40 页**

```c
#include <stdio.h>
int main()
{
    int i = 1;
    for(i = 1; i <= 10; i++)
    {
        if(i == 5)
            break;
        printf("%d ", i);
    }
    return 0;
}
```

运行结果：

![第4章第40页配图](/images/c-language-course/04-p40-fig01.png)

break 的作用是永久的终止循环，未来我们在某个条件发生的时候，不想再继续循环的时候，就可以使用 break 来完成。

#### 9.2.2 continue举例

上面的代码，如果把 break 换成 continue 会是什么结果呢？

**课件代码 04-071 · PDF 第 40 页**

```c
#include <stdio.h>
int main()
{
    int i = 1;
    for(i = 1; i <= 10; i++)
    {
        if(i == 5)
            continue;//这里continue跳过了后边的打印，来到了i++的调整部分
        printf("%d ", i);
    }
    return 0;
}
```

![第4章第40页配图](/images/c-language-course/04-p40-fig02.png)

所以在 for 循环中 continue 的作用是跳过本次循环中 continue 后的代码，直接去到循环的调整部分。未来当某个条件发生的时候，本次循环无需再执行后续某些操作的时候，就可以使用continue 来实现。

在这里我们也可以对比一下 while 循环和 for 循环中 continue 的区别：

![第4章第41页配图](/images/c-language-course/04-p41-fig01.png)

### 9.3 do while循环中的break和continue

do.while 语句中的 break 和 continue 的作用和 while 循环中几乎一模一样，大家下来可以自行测试并体会。

代码块 代码块

**课件代码 04-072 · PDF 第 41 页**

```c
#include <stdio.h>

int main()
{
    int i = 1;
    do
    {
        if(i == 5)
            break;
        printf("%d ", i);
        i = i + 1;
    }while(i <= 10);

    return 0;
}
```

**课件代码 04-073 · PDF 第 41 页**

```c
#include <stdio.h>

int main()
{
    int i = 1;
    do
    {
        if(i == 5)
            continue;
        printf("%d ", i);
        i = i + 1;
    }while(i <= 10);

    return 0;
}
```

## 10. 循环的嵌套

前面学习了三种循环 while ， do while ， for ，这三种循环往往会嵌套在一起才能更好的解决问题，就是我们所说的：循环嵌套，这里我们就看一个例子。

练习找出100~200之间的素数，并打印在屏幕上。

注：素数又称质数，只能被1和本身整除的数字。

题目解析

1. 要从100~200之间找出素数，首先得有100~200之间的数，这里可以使用循环解决。

2. 假设要判断i是否为素数，需要拿2~i-1之间的数字去试除i，需要产生2~i-1之间的数字，也可以使用

循环解决。

3. 如果2~i-1之间有数字能整除i，则i不是素数，如果都不能整除，则i是素数。

参考代码

**课件代码 04-074 · PDF 第 42 页**

```c
#include <stdio.h>

int main()
{
    int i = 0;
    //循环产生100~200的数字
    for(i = 100; i <= 200; i++)
    {
        //判断i是否为素数
        //循环产生2~i-1之间的数字
        int j = 0;
        int flag = 1;//假设i是素数
        for(j = 2; j < i; j++)
        {
            if(i % j == 0)
            {
                flag = 0;
                break;
            }
        }
        if(flag == 1)
            printf("%d ", i);
    }
    return 0;
}
```

🎈 AI提示器：这个代码中，素数判断的代码还可以优化吗？如何优化

## 11. goto 语句

C语言提供了一种非常特别的语法，就是 goto 语句和跳转标号， goto 语句可以实现在同一个函数内跳转到设置好的标号处。

例如：

**课件代码 04-075 · PDF 第 43 页**

```c
#include <stdio.h>
int main()
{
    printf("hehe\n");
    goto next;
    printf("haha\n");

next:
    printf("跳过了haha的打印\n");
    return 0;
}
```

goto 语句如果使用的不当，就会导致在函数内部随意乱跳转，打乱程序的执行流程，所以我们的建议是慎重使用，在适合的场景下使用；

- Linux之父Linus Torvalds曾说：" goto 在错误处理中使用是好的"

- SQLite文档明确："我们使用 goto 来简化错误处理"

goto 语句并非一无是处，以下场景下，goto还是经常被使用的

1. 在多层循环的代码中，如果想快速跳出，使用 goto 就非常方便

2. 统一错误处理

代码块 代码块

**课件代码 04-076 · PDF 第 43,44 页**

```c
for(...)
{
    for(...)
    {
        for(...)
        {
            if(disaster)
                goto error;
        }
    }
}

error:
    //...
```

**课件代码 04-077 · PDF 第 43,44 页**

```c
int test()
{
    void *p1 = NULL, *p2 = NULL;
    FILE *f = NULL;

    f = fopen("file", "r");
    if (!f) goto error;
    p1 = malloc(100);
    if (!p1) goto error;

    p2 = malloc(200);
    if (!p2) goto error;

    // 正常处理
    free(p1); free(p2); fclose(f);
    return 0;

error:
    if (p1) free(p1);
    if (p2) free(p2);
    if (f) fclose(f);
    return -1;
}
```

本来 for 循环想提前退出得使用 break ，一个break 只能跳出一层 for 循环，如果3层循环嵌套就得使用3个 break 才能跳出循环，所以在这种情况下我们使用 goto 语句就会更加的快捷。

在test函数内部，多个地方都能出现错误，出现错误后，想要统一处理，就可以使用goto语句。

🎈 AI提示词：

C语言中的goto语句在大型项目中会被使用吗？如果被使用，举例一些经典开源项目中goto语句的使用，最好给出原码实例和代码出处和链接。

## 12. AI加餐课

4.VibeCoding

## 课件中的参考链接

- [参考资料 1](https://bitejiuyeke.feishu.cn/wiki/G8Z1wV7loi22Oikr6R9cqDgenzc)
