---
title: C语言初阶 02：数据类型、变量、常量与字符
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 初阶, 数据类型]
category: C语言
series: C语言学习笔记
seriesOrder: 2
draft: false
updated: 2026-11-16
---

> 本篇按《（新 AI 版）鹏哥 C 语言课件》第 2 章逐节整理。保留原课件代码、对照写法、练习及图表，仅压缩重复说明。
> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- 课件给出的类型字节数来自特定环境。C 只保证 `sizeof(char) == 1` 及各类型的最小范围，int 不保证总是 4 字节。整型大小关系应为 `sizeof(char) <= sizeof(short) <= sizeof(int) <= sizeof(long) <= sizeof(long long)`，不能普遍使用严格小于。
- 普通字符常量如 `'x'` 在 C17 中类型为 int；`long double` 也不是 C99 才首次引入。字符数组和字符串长度按字节理解，UTF-8 中文不等于一个 char。

## 本章目标

1. 掌握C语言中的基本数据类型和修饰符

2. 掌握ASCII字符集和转义字符理解

3. 学习变量和常量的知识，能在代码中熟练使用

## 1. 基本数据类型

✍ 编程是为了解决生活中的问题，那就得能描述生活中的问题，生活中有各种各样的数据

1. 商品的价格、商品好评比例

2. 购物的小票上的各种数据

...那怎么描述这些信息呢？

C语言提供了丰富的数据类型来描述生活中的各种数据。

![第2章第2页配图](/images/c-language-course/02-p02-fig01.png)

![第2章第2页配图](/images/c-language-course/02-p02-fig02.png)

购物的小票中可能包含的信息如下：

![第2章第2页图表](/images/c-language-course/02-p02-table01.png)

在这个小票当中我们包含了很多的信息，货号、品名、单位、单价、数量、小计。分析我们发现：

![第2章第2页图表](/images/c-language-course/02-p02-table02.png)

![第2章第3页图表](/images/c-language-course/02-p03-table01.png)

使用整型类型来描述整数，使用字符类型来描述字符，使用浮点型类型来描述小数。

🎈 所谓“类型”，就是相似的数据所拥有的共同特征，编译器只有知道了数据的类型，才知道

怎么操作数据。

下面盘点一下C语言提供的各种数据类型，本章节主要探讨内置数据类型。

### 1.1 字符型

字符的英文单词是 character ，在C语言中使用 char 表示字符类型。

**课件代码 02-001 · PDF 第 4 页**

```c
char    //character

char ch; //使用char创建字符变量
```

C语言中，字符是使用一对单引号括起来的，比如：'x'、'a' 、'w'，这些字符的类型就是 char 。

### 1.2 整型

整数的英文单词是 integer ，在C语言中使用 int 表示整型类型。

**课件代码 02-002 · PDF 第 4 页**

```c
int

int num; //使用int创建整型变量
```

在C语言中，100、20、35 这些整数的类型就是 int 。

### 1.3 浮点型

**课件代码 02-003 · PDF 第 4 页**

```c
float   //单精度浮点型
double  //双精度浮点型

float num1;   //使用float创建变量
double num2;  //使用double创建变量
```

在C语言中， 3.14 、 5.5 这些小数的类型默认是 double 类型， 3.14f 这种写法表示 float 类型。

### 1.4 布尔类型

C语言在C89/C90时期并没有为布尔值单独设置一个类型，使用整数 0 表示假，非零值表示真。

后来在 C99 标准中也引入了 布尔类型 ，是专门表示真假的。

**课件代码 02-004 · PDF 第 4 页**

```text
_Bool
```

布尔类型变量的取值是： true 或者 false ，这2个符号是定义在 &lt;stdbool.h&gt; 头文件中的。

**课件代码 02-005 · PDF 第 5 页**

```c
#define bool  _Bool

#define false 0
#define true  1
```

布尔类型的使用，一般得包含头文件 &lt;stdbool.h&gt;代码演示：

**课件代码 02-006 · PDF 第 5 页**

```c
#include <stdbool.h>

_Bool flag = true;
if (flag)
    printf("I like C\n");
```

一直到 C23 这个标准，才把 bool 、 true 、 false 定为C语言关键字，使用bool类型，不再依赖 &lt;stdbool.h&gt; 这个头文件了。

![第2章第5页配图](/images/c-language-course/02-p05-fig01.png)

🎈 AI提示词：

C语言中布尔类型是如何演变的？为什么 C99 中不直接使用 bool 而是使用\_Bool ?

## 2. 类型修饰符

有前面介绍的数据类型，其实已经能解决一部分问题，但是在真实的开发过程中，还是不够，为了让内置类型提供更精细的数据表示能力，C语言中，还引入了类型修饰符：

**课件代码 02-007 · PDF 第 6 页**

```text
//和内存长度有关的
short
long

//和符号相关的
signed
unsigned
```

就像，最开始人类发明了汽车，觉得很好了，但是随着汽车的普及，又衍生出了各种细分的汽车。

![第2章第6页配图](/images/c-language-course/02-p06-fig01.png)

### 2.1 short 和 long

short 和 long 是和内存长度有关的两个类型修饰符。

**课件代码 02-008 · PDF 第 6 页**

```c
int   //标准的整型

//修饰整型
short [int] //短整型
long [int]  //长的整型
long long [int] //长的整型
// []中的内容是可以省略的

//修饰浮点型
long double  //C99中引入
```

- short 和 long 是用来修饰整型类型。

- 常规的整型类型是 int ，长度是4个字节， short int 类型的长度是2个字节。

- C语言标准规定 long int 的长度是大于等于 int 的长度的。

![第2章第7页配图](/images/c-language-course/02-p07-fig01.png)

- 要想知道每个数据的类型的长度，我们可以使用 sizeof 来计算， sizeof 是C语言的一个操作

符， sizeof 可以算出一个数据类型的长度，单位是字节，使用的方法就是：

**课件代码 02-009 · PDF 第 7 页**

```text
sizeof(type)
```

- type就是对应的数据类型， sizeof 计算的结果是 size\_t 类型的，打印的时候使用应该使

用 %zu 的占位符，请看下面的代码：

**课件代码 02-010 · PDF 第 7,8 页**

```c
#include <stdio.h>
int main()
{
    printf("char:        %zu\n", sizeof(char));
    printf("_Bool:       %zu\n", sizeof(_Bool));
    printf("short:       %zu\n", sizeof(short));
    printf("int:         %zu\n", sizeof(int));
    printf("long:        %zu\n", sizeof(long));
    printf("long long:   %zu\n", sizeof(long long));

    printf("float:       %zu\n", sizeof(float));
    printf("double:      %zu\n", sizeof(double));
    printf("long double: %zu\n", sizeof(long double));
    return 0;
}
```

VS2026-msvc-输出结… mingw64 gcc 输出结… Linux gcc

**课件代码 02-011 · PDF 第 8 页**

```c
char:        1
_Bool:       1
short:       2
int:         4
long:        4
long long:   8
float:       4
double:      8
long double: 8
```

**课件代码 02-012 · PDF 第 8 页**

```c
char:        1
_Bool:       1
short:       2
int:         4
long:        4
long long:   8
float:       4
double:      8
long double: 16
```

**课件代码 02-013 · PDF 第 8 页**

```c
char:        1
_Bool:       1
short:       2
int:         4
long:        8
long long:   8
float:       4
double:      8
long double: 16
```

![第2章第8页配图](/images/c-language-course/02-p08-fig01.png)

注：long 和 long double 类型的长度，是和平台采用的数据模型有关的。

数据模型指的是在特定操作系统、CPU 架构和 ABI（应用程序二进制接口）下，C/C++ 等语言中基本整数类型（ short 、 int 、 long 、 long long ）以及指针类型的具体位宽（字节数） 的约定。

参考资料：数据类型长度和取值范围-官方介绍数据类型的长度关系如下：

**课件代码 02-014 · PDF 第 8 页**

```c
//整型家族
1 == sizeof(char) < sizeof(short) < sizeof(int) <= sizeof(long) <= sizeof(long long)

//浮点型家族
sizeof(float) < sizeof(double) <= sizeof(long double)
```

### 2.2 signed 和 unsigned

signed 和 unsigned 是和符号相关的两个类型修饰符，专门修饰 字符型和整型 类型的。

我们生活中表示的数值，有的数值是有正有负的，有的时候只有正数。

比如：

1. 温度、海拔高度、财务盈亏、电梯楼层、银行账戶流水等数据，有正也有负

2. 年龄、长度、身高、体重等数据，只有正数和0的可能

signed 关键字，表示一个类型带有正负号，可以表示正整数、负整数和0；unsigned 关键字，表示该类型不带有正负号，只能表示零和正整数。

![第2章第9页图表](/images/c-language-course/02-p09-table02.png)

对于 int 类型，默认是带有正负号的，也就是说 int 等同于 signed int 。

由于这是默认情况，关键字 signed 一般都省略不写，但是写了也不算错。

但是，C 语言中 char 有可能是 signed char ，也有可能是 unsigned char ，具体等价于哪个是编译器的具体实现定义的。这一点和 int 类型是有差异的。

我们前面通过 sizeof 算出长度的类型是 size\_t ， size\_t 就是无符号整型，因为长度不可能是负数。

🎈 sizeof 操作符的返回值，C 语言只规定是无符号整数，并没有规定具体的类型，而是留给

系统自己去决定， sizeof 到底返回什么类型。不同的系统中，返回值的类型有可能是unsigned int ，也有可能是 unsigned long ，甚至是 unsigned long long ，对应的 printf() 占位符分别是 %u 、 %lu 和 %llu 。这样不利于程序的可移植性。

C 语言提供了一个解决方法，创造了一个类型别名 size\_t ，用来统一表示 sizeof 的返回值类型，对应当前系统的 sizeof 的返回值类型，可能是 unsigned int ，也可能是unsigned long long 。

size\_t 类型的值，在打印时的占位符是%zu。

### 2.3 取值范围

我们看到不同的数据类型长度是不同的，长度不同又决定了存放的数据的范围是不同的，这里我们看一下不同数据类的取值范围。

整型家族的取值范围可以在 &lt;limits.h&gt; 这个头文件中查看浮点型家族的取值范围可以在 &lt;float.h&gt; 这个头文件中查看

![第2章第10页配图](/images/c-language-course/02-p10-fig01.png)

为了代码的可移植性，需要知道某种整数类型的极限值时，应该尽量使用这些常量。

- SCHAR\_MIN ， SCHAR\_MAX ：signed char 的最小值和最大值。

- SHRT\_MIN ， SHRT\_MAX ：short 的最小值和最大值。

- INT\_MIN ， INT\_MAX ：int 的最小值和最大值。

- LONG\_MIN ， LONG\_MAX ：long 的最小值和最大值。

- LLONG\_MIN ， LLONG\_MAX ：long long 的最小值和最大值。

- UCHAR\_MAX ：unsigned char 的最大值。

- USHRT\_MAX ：unsigned short 的最大值。

- UINT\_MAX ：unsigned int 的最大值。

- ULONG\_MAX ：unsigned long 的最大值。

- ULLONG\_MAX ：unsigned long long 的最大值。

## 3. ASCII字符集和转义字符

### 3.1 ASCII字符集

在C语言中， char 类型是字符类型，用来表示字符，但也是属于整型家族的，那这些字符的值是多少呢？

比如：'a'的值是多少？'x'的值是多少呢？

这里的值可能根据计算机的不同而不同。因为不同的机器可能使用不同的字符集，当前我们见到的Windows/Linux/macOS 平台上采用 ASCII 字符集，而在IBM大型机等老旧或专用系统还使用EBCDIC 字符集，我们主要掌握现在主流的ASCII字符集就行。

在ASCII字符集中规定了，128个字符的对应的数值，如下图，后来ASCII码常被扩展为256个字符，目的是兼容更多国家语言中的字符。

![第2章第11页配图](/images/c-language-course/02-p11-fig01.png)

ASCII编码表参考：https://zh.cppreference.com/w/cpp/language/ascii我们不需要记住所有的ASCII码表中的数字，使用时查看就可以，不过我们最好能掌握几组特殊的数据：

- 字符 A~Z 的ASCII码值从65~90，按照字母顺序是递增1

- 字符 a~z 的ASCII码值从97~122，按照字母顺序是递增1

- 对应的大小写字符(a和A)的ASCII码值的差值是32

- 数字字符0~9的ASCII码值从48~57，按照数字顺序是递增1

- 换行 \\n 的ASCII值是：10

- 在这些字符中ASCII码值从0~31 这32个字符是不可打印字符，无法打印在屏幕上观察

这里我们通过代码看一下这些字符对应的数值。

**课件代码 02-015 · PDF 第 12 页**

```c
#include <stdio.h>
int main()
{
    printf("%d\n", 'X');
    printf("%d\n", '#');
    return 0;
}
```

### 3.2 转义序列

通常情况下字符是被单引号括起来的单个字符，比如：'a', 'x'，但是ASCII码值从0~31的字符是不可打印字符，比如：换行符，无法通过单引号括起来单个字符了，这些单个字符都已经有具体的意义了，那这些字符如何表示呢？

为了使程序可以处理字符集中的每一个字符，C语言提供了一种特殊的表示法：转义序列。

转义序列有2种：字符转义序列和数字转义序列。

字符转义序列转义字符 解释\\? 在书写连续多个问号时使用，防止他们被解析成三字母词，在新的编译器上没法验证了。

\\' 用于表示字符常量'\\" 用于表示一个字符串内部的双引号

![第2章第13页图表](/images/c-language-course/02-p13-table01.png)

除了上面的这些转义字符之外，字符转义序列，不能表示其他的不可打印字符，所以就有了数字转义序列。

数字转义序列

![第2章第13页图表](/images/c-language-course/02-p13-table02.png)

数字转义序列可以表示ASCII字符集中的所有字符。

需要把一个字符写成数字转义序列，查看一下ASCII字符集中，字符对应的8进制数值和16进制数值就很好转化了。

### 3.3 练习

**课件代码 02-016 · PDF 第 13 页**

```c
#include <stdio.h>
#include <string.h> //strlen 函数需要

int main()
{
    printf("%zu\n", strlen("c:\add\test\007\vip.c"));
    return 0;
}
```

1. strlen 是求字符串长度的库函数，统计的是字符串中 '\\0' 之前的字符个数， '\\0' 是字符串

的结束标志。

2. 注意识别字符串中的转义字符，一个转义字符，在统计时只算一个字符。

## 4. 变量和常量

清楚了C语言中的数据类型后，数据类型最主要的功能就是创建变量和常量。

🎈 什么是变量？什么是常量呢？

变量和常量都是用来存储数据的 “容器”，但它们最核心的区别在于：变量的值可以改变，而常量的值一旦设定就不能再修改。

### 4.1 变量

变量创建的语法形式是这样的：

语法说明

**课件代码 02-017 · PDF 第 14 页**

```c
data_type name;
   |        |
   |        |
数据类型   变量名
```

🎈 变量命名的一般规则：

- 变量的名字起得要有意义，见名知意。

- 只能由字母（包括大写和小写）、数字和下划线（`\_`）组成。

- 不能以数字开头。

- 长度不能超过63个字符。

- 变量名中区分大小写的。

- 变量名不能使用关键字。

举例

**课件代码 02-018 · PDF 第 14,15 页**

```c
char ch;
int age;
double weight;
int 2b;//这个正确吗？
```

变量在创建的时候就给一个初始值，就叫初始化。

**课件代码 02-019 · PDF 第 15 页**

```c
char ch = 'w';
int age = 18;
double weight = 48.0;
unsigned int high = 180;
```

### 4.2 变量的分类

- 全局变量：在函数之外定义的变量称为全局变量

- 作用域：从定义位置开始，到整个源文件末尾都可见（如果其他文件想用，可通过 extern

声明）。

- 生命周期：在程序启动时创建，程序结束时才销毁（存储在内存的静态存储区/数据段）。

- 初始值：如果没有显式初始化，系统会自动初始化为0（或 '\\0' 、 NULL ）。

- 使用注意：过多使用全局变量会导致模块间耦合度高、调试困难，建议谨慎使用。

🎈 全局变量就类似马路上的共享单车，大家都可以使用。

**课件代码 02-020 · PDF 第 15 页**

```c
#include <stdio.h>
int globalVar = 100;   // 全局变量

void show()
{
    printf("全局变量值: %d\n", globalVar);
}

int main()
{
    show();
    globalVar = 200;    // 直接修改全局变量
    printf("修改后: %d\n", globalVar);
    return 0;
}
```

- 局部变量：在大括号内部定义的变量就是局部变量

- 作用域：仅限于定义它的函数或代码块内部，外部无法访问。

- 生命周期：在进入函数/代码块时创建，退出时自动销毁（存储在内存的栈区）。

- 初始值：如果没有显式初始化，其值是垃圾值（不确定的随机数）,建议对局部变量进行初始

化。

- 优先级：当局部变量与全局变量同名时，优先使用局部变量。

🎈 局部变量就类似于你们家自己的自行车，只有你家里人有权利使用它。

**课件代码 02-021 · PDF 第 16 页**

```c
#include <stdio.h>

void func()
{
    int localVar = 10;   // 局部变量
    printf("%d\n", localVar);
}

int main()
{
    // printf("%d", localVar); // 错误！main函数无法访问func中的localVar
    return 0;
}
```

🎈 AI提示词1：在C语言中，什么是作用域？什么是生命周期？

AI提示词2：软件开发中，什么是高内聚低耦合？

AI提示词3：对比一下C语言中的全局变量和局部变量？

AI提示词4：什么情况下使用全局变量？什么情况下使用局部变量？

全局变量和局部变量在内存中存储在哪里呢？

一般我们在学习C/C++语言的时候，我们会关注内存中的三个区域：栈区、堆区、静态区。

1. 局部变量是放在内存的栈区

2. 全局变量是放在内存的静态区（数据段）

3. 堆区是用来动态内存管理的（后期会介绍）

![第2章第17页配图](/images/c-language-course/02-p17-fig01.png)

其实内存区域的划分会更加细致，以后在操作系统的相关知识的时候会介绍。

### 4.3 常量

C语言中常量主要是下面这几种形式呈现的：

#### 4.3.1 字面常量

**课件代码 02-022 · PDF 第 18 页**

```c
//字符常量
'a'
'x'

//整型常量
//十进制字面常量
123，-456，0

//八进制字面常量：以 0 开头
037（等于十进制的 31）

//十六进制字面常量：以 0x 或 0X 开头
0x1F (等于进制的31)。

//后缀：
u 或 U：表示无符号整数（unsigned），如 100U。
l 或 L：表示长整型（long），如 123L。
ll 或 LL：表示长长整型（long long，C99 起），如 123LL。
可组合：100UL（unsigned long）。

//浮点型字面常量
3.14    //double类型
3.14f   //float 类型
3.14L   //long double类型
1.2e3   //科学计数法，表示1.2×10³
```

#### 4.3.2 符号常量

**课件代码 02-023 · PDF 第 18,19 页**

```c
//符号常量通常名字是全大写的
#define M 100
#define PI 3.14159

int main()
{
    int Max = M;
    printf("M   = %d\n", M);
    printf("Max = %d\n", Max);
    printf("pi = %lf\n", PI);

    return 0;
}
```

#### 4.3.3 const 修饰的常变量

使用 const 关键字修饰的变量，其值在初始化后不可修改，但本质上仍然是变量（有类型、有地址），只是编译器禁止赋值操作。

**课件代码 02-024 · PDF 第 19 页**

```c
const double e = 2.71828;   // 浮点常变量
const int days_in_week = 7;
days_in_week = 8;        // 错误：不能修改 const 变量
```

🎈 总结一下:

- 变量：值可以变，就像一个小白板，可以反复擦写。

- 常量：值不能变，就像刻在石碑上的字，一旦确定就不允许更改。

## 5. AI加餐课

2.大语言模型（LLM）

## 课件中的参考链接

- [参考资料 1](https://zh.cppreference.com/c/language/arithmetic_types#.E6.95.B0.E6.8D.AE.E6.A8.A1.E5.9E.8B)
- [参考资料 2](https://zh.cppreference.com/w/c/language/arithmetic_types#.E6.95.B0.E6.8D.AE.E6.A8.A1.E5.9E.8B)
- [参考资料 3](https://baike.baidu.com/item/ebcdic/6816534)
- [参考资料 4](https://bitejiuyeke.feishu.cn/wiki/HVy4wP6FSiPXGxkHNK5cT8Ounig)
