---
title: C语言初阶 01：初识 C 语言与第一个程序
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 初阶, 编程入门]
category: C语言
series: C语言学习笔记
seriesOrder: 1
draft: false
updated: 2026-11-16
---

> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- `int main()` 是课件原写法；面向 C17 的新代码建议使用 `int main(void)` 明确无参数。课件代码中的缺失头文件、多个 main 等应按独立示例理解，不要把整章代码拼成一个源文件。

## 本章目标

1. 认识C语言，了解C语言的历史和辉煌

2. 知道C语言学习必备的工具和选择

3. 编写并剖析第一个C语言程序，掌握C语言中的一些常识

先问一个"终极"问题：

📍 AI提示词： AI都能写代码了，现在计算机的大学生还需要学习C语言吗？

可以问问deepseek：https://chat.deepseek.com/

## 1. C语言是什么？

💻 C语言是一门计算机语言，是人和计算机交流的语言。

人和人交流使用的是自然语言，如：汉语、英语、日语。

那人和计算机是怎么交流的呢？使用计算机语言；人是通过计算机语言写的程序，给计算机下达指令，让计算机来帮我们完成各种工作。

C语言就是众多计算机语言中的一种，当然C++/Java/Go/Python都是计算机语言。

## 2. C语言的历史和辉煌

### 2.1 历史

C 语言最初是作为 Unix 系统的开发工具而发明的。

![第1章第2页配图](/images/c-language-course/01-p02-fig01.png)

![第1章第3页配图](/images/c-language-course/01-p03-fig01.png)

C语言之父

### 2.2 C语言的活跃度

![第1章第3页配图](/images/c-language-course/01-p03-fig02.png)

TIOBE 编程社区指数是衡量编程语言流行程度的指标参考链接：https://www.tiobe.com/tiobe-index/

### 2.3 C语言的贡献

Linux

![第1章第4页配图](/images/c-language-course/01-p04-fig01.png)

![第1章第4页配图](/images/c-language-course/01-p04-fig02.png)

FreeRTOS-Kernel

![第1章第4页配图](/images/c-language-course/01-p04-fig03.png)

redis

![第1章第4页配图](/images/c-language-course/01-p04-fig04.png)

mysql-server在偏底层的软硬件开发中，比如：系统软件开发、驱动开发、嵌入式开发，C语言当仁不让。

## 3. 学习C语言需要什么工具？

![第1章第5页配图](/images/c-language-course/01-p05-fig01.png)

### 3.1 翻译工具：编译器+链接器

C语言是一门编译型计算机语言，C语言源代码都是文本文件，C语言代码是放在 .c 为后缀的文件中的，文本文件本身无法执行，必须通过编译器和链接器的处理，生成二进制的可执行文件（包含二进制指令），可执行文件才能在计算机上执行，来完成任务。

![第1章第6页配图](/images/c-language-course/01-p06-fig01.png)

![第1章第6页配图](/images/c-language-course/01-p06-fig02.png)

VS2026项目中的.c文件 .c文件的展示

![第1章第6页配图](/images/c-language-course/01-p06-fig03.png)

C语言代码一个工程一般都会有多个源文件组成，如下图所示，演示了源程序经过编译器和链接器处理的过程。

test.c test.obj编译器add.c add.obj cl.exe链接器 可执行程序link.exe xxx.exe xxx.c xxx.obj链接库编译 链接注：

1. 每个源文件(.c)单独经过编译器处理生成对应的目标文件(.obj为后缀的文件)

2. 多个目标文件和库文件经过链接器处理生成对应的可执行程序(.exe文件)

这就是，在 Windows 电脑上C语言程序生成的 exe 可执行文件

![第1章第7页配图](/images/c-language-course/01-p07-fig01.png)

可执行程序

### 3.2 编译器的对比

那我们常见的C语言编译器都有哪些呢？

比如： msvc 、 clang 、 gcc 就是一些常见的编译器编译器 特点 维护方GCC 跨平台、成熟、优化稳定、支持语言多 开源社区维护Clang 基于 LLVM，错误提示更友好，编译速度快，macOS 默认 苹果MSVC Windows 平台官方 C++ 编译器，集成在 Visual Studio 中 微软直接使用上面提到的编译器工具不够方便，学习成本高一些；所以一些集成开发环境 如：VS2022、XCode、CodeBlocks、DevC++、Clion，都在其中集成了这些编译器套件，使用起来就非常方便了。

集成开发环境（IDE）用于提供程序开发环境的应用程序，一般包括代码编辑器、 编译器 、 调试器和 图形用戶界面 等工具；集成了代码编写、代码分析、代码编译、代码调试、构建等功能。

![第1章第8页配图](/images/c-language-course/01-p08-fig01.png)

![第1章第8页配图](/images/c-language-course/01-p08-fig02.png)

![第1章第8页配图](/images/c-language-course/01-p08-fig03.png)

![第1章第8页配图](/images/c-language-course/01-p08-fig04.png)

![第1章第8页配图](/images/c-language-course/01-p08-fig05.png)

VS2022 XCode CodeBlocks DevC++ Clion

- VS2022 集成了MSVC（安装包较大一些，安装简单，无需多余配置，使用起来非常方便）

- XCode 集成了clang（苹果电脑上的开发工具）

- CodeBlocks 集成了gcc（这个工具比较小众，需要配置环境，不太推荐）

- DevC++ 集成了gcc（小巧，但是工具过于简单，对于代码风格的养成不好，一些竞赛使用）

- Clion 是默认使用CMake构建，编译器是可以配置的（个人可以申请免费使用）

整体考虑，推荐大家安装 VS2022/VS2026 的社区版本学习，免费，使用方便，工作中常见。

VS2026 的安装教程：https://www.bilibili.com/video/BV1vE6gBHETe VS2022 的安装教程：https://www.bilibili.com/video/BV11R4y1s7jz/我上课使用的是VS2026，为了方便学习，可以保持一致。

### 3.3 VS2026 的优缺点

![第1章第8页图表](/images/c-language-course/01-p08-table01.png)

🙋 使用 VSCode 可以吗？

1. 如果你已经熟练使用，当然可以的，后期上课的过程中，我也会使用

2. VSCode本质上是一个编辑器，支持各种插件扩展

3. 可以使用VSCode + 一些插件 + gcc/clang来搭建 C/C++的开发环境

4. 学习成本较高一些，初学者不是很建议直接使用VSCode

VSCode上搭建C/C++开发环境教程：https://www.bilibili.com/video/BV1shLw69Ed1/

✍ 当然在AI时代，能不能让AI帮我们搭建一个C/C++的开发环境呢？

答案是可以的，请参考：https://b23.tv/bqF8XZe但是先得把编程智能体使用起来，而且你得有手动搭建的经验，才能理解AI是怎么做的？为什么这么做？这些都是需要慢慢学习的。

## 4. 第一个C语言程序

### 4.1 VS项目 和 源文件、头文件介绍

在VS2026上写代码，我们是需要创建项目的，直接新建项目就可以了。

在项目中就可以添加源文件和头文件。

C语言把 .c 为后缀的文件称为源文件，把 .h 为后缀的文件称为头文件。

![第1章第10页配图](/images/c-language-course/01-p10-fig01.png)

![第1章第10页配图](/images/c-language-course/01-p10-fig02.png)

头文件和源文件在VS中展示 头文件和源文件在磁盘上

### 4.2 第一个C语言程序

🎈 AI提示器：帮我写一个简单的C语言程序，功能是在屏幕上输出："hello C".

**课件代码 01-001 · PDF 第 10 页**

```c
#include <stdio.h>

int main()
{
    printf("hello C\n");
    return 0;
}
```

### 4.3 main函数

每个 C 语言程序不管有多少行代码，都是从 main 函数开始执行的， main 函数是程序的入口，main 函数也被叫做：主函数。 main 前面的 int 是 integer 的缩写，意思是：整型类型，表示main 函数执行结束的时候返回一个整型类型的值，也就是整数。所以在 main 函数的最后写return 0; 正好前后呼应。

main 函数的模板如下：

**课件代码 01-002 · PDF 第 11 页**

```c
int main()
{

    return 0;
}
```

- main 函数是程序的入口

- main 函数有且仅有一个

- 一个项目中可以有多个 .c 文件，但是只能有一个 main 函数（因为程序的入口只能有一个）

⚠ 第一次写代码，一些常见的错误如下

- main 被写成了mian

- main后边的()漏掉了

- 代码中不能使用中文符号，比如括号和分号

- 一条语句结束后，有分号

🧑‍💻 初学的时候，因为不熟悉，犯错误是非常常见的，慢慢就会好起来的。

### 4.4 关键字

❓ 为什么在前面代码中， int 、 return 可以直接使用呢？

其实C语言中有一批保留的名字的符号，比如： int 、 if 、 return ，这些符号被称为保留字或者关键字。

- 关键字都有特殊的意义，是保留给C语言使用的

- 程序员自己在创建标识符的时候是不能和关键字重复的

- 关键字也是不能自己创建的

C 语言的32个常见关键字如下：

**课件代码 01-003 · PDF 第 11,12 页**

```c
auto  break   case  char  const   continue  default  do   double else  enum extern
float  for   goto  if   int   long  register    return   short  signed  sizeof static
struct  switch  typedef union  unsigned   void  volatile  while
```

注：在C99标准中加入了 inline 、 restrict 、 \_Bool 、 \_Complex 、 \_Imaginary 等关键字。

一些关键字可以去了解一下，不过使用最多的还是上面的32个关键字。

https://zh.cppreference.com/c/keyword（C语言关键字的全部介绍）后期我们讲课的过程中，会慢慢介绍这些常用的关键字的。

### 4.5 库函数和头文件

在上面的代码中有一句代码如下：

**课件代码 01-004 · PDF 第 12 页**

```c
printf("hello C\n");
```

printf 是一个库函数，功能是在标准输出设备（一般指屏幕）上的打印信息。

上面的代码是使用 printf 函数打印一串字符，只要把想要打印的一串字符放在双引号中，就会被打印到屏幕上。字符串末尾的 \\n 是换行的意思，打印完数据后就会自动换行到下一行，后续如果还有数据打印的话，就在下一行上打印了。

❓ 双引号中的内容可以替换成你自己想打印的内容吗？可以试试哦。

✍ C语言中把双引号括起来的一串字符，称为字符串, 比如："abc"

C语言中把单引号括起来的一个字符，称为字符，比如：'x'，'a'printf 函数也可以用来打印其他类型的数据，比如：

**课件代码 01-005 · PDF 第 12 页**

```c
printf("%d\n", 100);     //printf打印整型
printf("%c\n", 'q');     //printf打印字符
printf("%f\n", 3.14);   //printf打印双精度浮点型
```

这里的 %d , %c , %f 等是占位符，会被后边的值替换。（后期课程会反复使用，慢慢就会熟悉）同时我们在使用库函数的时候，是需要包含库函数对应的头文件（.h为后缀的一种文件），比如：

printf 函数需要包含的就是 stdio.h 这个头文件，具体的方法就是：

**课件代码 01-006 · PDF 第 13 页**

```c
#include <stdio.h>
```

stdio 的是 standard input ouput 的缩写，是标准输入输出的意思，在C语言中只要和输入输出有关的操作，都需要包含这个头文件。

那什么是库函数呢？

1. 预先编写好、经过测试、打包成库（Library）并供开发者直接调用的函数就是库函数，它们不属于

编程语言本身，而是语言配套的标准库或第三方提供的代码集合。这里专门指的是C语言配套的标准库中的函数。

2. 一个系列的库函数一般会声明在同一个头文件中，所以库函数的使用，要包含对应的头文件。

3. 库函数比较多，后期慢慢来介绍，提前了解可参考链接：

https://cplusplus.com/reference/clibrary/

### 4.6 注释

注释是写给程序员看的一些内容，用于解释代码的逻辑、功能或注意事项。

编译器在编译时会完全忽略注释，因此注释不会影响程序的实际运行。

写注释一定程度上反应了程序作者的素质，建议大家写必要的注释，在未来找工作的时候，写代码时留下必要的注释也会给面试官留下更好的印象。

#### 4.6.1 为什么使用注释？

- 提高代码可读性，方便自己和他人理解。

- 临时屏蔽某段代码（调试时常用）。

#### 4.6.2 注释的形式

## 1. /\*\*/ 的注释形式

- 以 /\* 开始，以 \*/ 结束，中间所有内容（可跨多行）都是注释。

- 这种注释不能嵌套，以 /\* 开始，遇到第一个 \*/ 注释就结束了。

**课件代码 01-007 · PDF 第 13,14 页**

```c
/*
这是一个多行注释
可以用来写较长的说明
比如函数功能、参数含义等
*/

/* 也可以临时屏蔽一大段代码
int b = 20;
printf("%d", b);
*/

/*
功能：整型加法操作
参数：加法的2个操作数，都是整数
返回值：计算的结果
*/

int add(int x, int y)
{
    return x + y;
}
```

## 2. // 的注释形式

- 从 // 开始直到该行末尾的内容都是注释

- // 更简洁，适合单行或行尾注解

- 现代C语言（C99及以后）普遍支持 //

**课件代码 01-008 · PDF 第 14 页**

```c
printf("%d", 100);  // 这是一个单行注释，调用printf函数打印100这个整数
// 下面这行代码被注释掉了，不会执行
// printf("Hello");
```

🎈 AI提示器：在软件开发中，什么情况下是需要写注释的？写注释的原则是啥？

## 5. AI加餐课

1. 软件开发流程和软件工程

## 课件中的参考链接

- [参考资料 1](https://github.com/torvalds/linux)
- [参考资料 2](https://github.com/FreeRTOS/FreeRTOS-Kernel)
- [参考资料 3](https://github.com/redis/redis)
- [参考资料 4](https://github.com/mysql/mysql-server)
- [参考资料 5](https://visualstudio.microsoft.com/zh-hans/vs/)
- [参考资料 6](https://developer.apple.com/cn/xcode/)
- [参考资料 7](http://www.codeblocks.org/)
- [参考资料 8](https://sourceforge.net/projects/orwelldevcpp/)
- [参考资料 9](https://www.jetbrains.com/clion/)
- [参考资料 10](https://bitejiuyeke.feishu.cn/wiki/D2REwSS0xicMIak24PUcEUsungb)
