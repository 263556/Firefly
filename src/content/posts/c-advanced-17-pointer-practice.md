---
title: C语言进阶 17：指针的边界、const、断言与所有权
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 进阶, 指针, 代码规范]
category: C语言
series: C语言学习笔记
seriesOrder: 17
draft: false
updated: 2026-11-16
---

> 本篇按《（新 AI 版）鹏哥 C 语言课件》第 17 章逐节整理。保留原课件代码、对照写法、练习及图表，仅压缩重复说明。
> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- 非空不代表指针有效；free 后将拥有者置 NULL 并不能修复所有别名。assert 可被 NDEBUG 关闭，不能在其中放必需副作用，分配失败、文件不存在等用运行时检查处理。
- 对本来定义为 const 的对象强制去掉 const 再修改是未定义行为，不能作为“绕过 const 的正常方法”。

## 本章目标

1. 掌握在使用C语言指针进行程序设计中总结的各种最佳实践

2. 掌握assert断言

3. 掌握const修饰指针的基本写法

## 1. 永远初始化指针

未初始化的指针就是“野指针”，指向的内存地址不可预测，解引用即未定义行为。

**课件代码 17-001 · PDF 第 1 页**

```c
//野指针
int *p;
*p = 10;  //err

//好习惯：初始化为 NULL 或有效地址
int* p1 = NULL;
int num = 10;
int*p2 = &num;
```

## 2. 释放内存后立即置 NULL

动态申请的内存调用 free 函数释放后，指向动态内存空间的指针立马就变成悬空指针。

悬空指针虽然是非 NULL 指针，但是指向的空间已经不属于当前程序，不能再继续使用。

为了防止使用悬空指针，需要手动将指针置为 NULL ;

**课件代码 17-002 · PDF 第 1 页**

```c
int *ptr = (int*)malloc(20);
//....
free(ptr);
ptr = NULL;   // 防止误用
```

📌 • 野指针：从未有效。指针变量定义时没有初始化，值是一个随机的垃圾地址。

- 悬空指针：曾经有效，但现在失效了。指针原本指向了一块有效的内存，但该内存已被释

放或销毁。

它们都属于“错误的指针”，危害也都很严重（导致程序崩溃或数据污染），但根本区别在于指针是否曾经有效

## 3. 使用前检查非空

基于指针进行了初始化和置 NULL 的前提，我们使用的指针变量只有两类值：

- 有效的地址

- NULL 指针

指针为 NULL 的时候，表示没有指向有效的内存空间，就无法使用，所以在使用指针之前我们会进行检查。

这里检查一般会使用 if 语句判断或者 assert 断言。

1. 对于函数参数中的指针，或者函数返回值的指针，除非明确约定不可为 NULL ，否则都应该进行防

御性检查。

**课件代码 17-003 · PDF 第 2 页**

```c
//代码1
void process_data(struct buffer *buf)
{
    if (!buf)
        return;   // 或返回错误码
    // 正常处理...
}

//代码2
int*p = (int*)malloc(100);//因为malloc有可能返回NULL指针的
if(NULL == p)
{
    return;  // 或返回错误码
}
//正常处理
```

2. 对于内部函数，如果性能敏感且调用者保证非 NULL ，可使用 assert 在调试时暴露错误。

什么是内部函数呢？

一般就是static修饰的函数，或者是小范围内使用的函数，函数的使用范围是可控的，能够保证函数的正确调用。

相对应的就是外部函数，库函数、还有对外的提供的接口函数都是外部函数，这些函数的使用者很广泛，并非程序作者可控的。

**课件代码 17-004 · PDF 第 3 页**

```c
#include <assert.h>

// 内部辅助函数：求数组和。
// 调用者已保证 arr 非空且 len > 0，所以用 assert 在调试期验证即可。
static double array_sum(const double *arr, int len)
{
    assert(arr != NULL);      // 发布版无开销
    assert(len > 0);

    double total = 0.0;
    for (int i = 0; i < len; i++)
    {
        total += arr[i];
    }
    return total;
}

// 外部 API：计算平均值，调用者可能传 NULL 或 len=0。
// 需要运行时防御，返回错误码或安全默认值。
double array_average(const double *arr, int len)
{
    if (!arr || len <= 0) return 0.0;   // 运行时检查，防止崩溃

    return array_sum(arr, len) / len;   // 此时调用内部函数完全安全
}
```

### 3.1 assert是啥？

assert.h 头文件定义了宏 assert() ，用于在运行时确保程序符合指定条件，如果不符合，就报错终止运行。这个宏常常被称为“断言”。

**课件代码 17-005 · PDF 第 3 页**

```c
#include <assert.h>

assert(p != NULL);
```

上面代码在程序运行到这一行语句时，验证变量 p 是否等于 NULL 。如果 p 确实不等于 NULL ，程序继续运行，否则就会终止运行，并且给出报错信息提示。

assert() 宏接受一个表达式作为参数。如果该表达式为真（返回值非零）， assert() 不会产生任何作用，程序继续运行。如果该表达式为假（返回值为零）， assert() 就会报错，在标准错误流stderr 中写入一条错误信息，显示没有通过的表达式，以及包含这个表达式的文件名和行号。

📌 一般在 assert() 中写的表达式是"一定要为真"的表达式，这是我们期望的结果。

当然这个表达式不一定非要和指针相关，也可以其他希望为真的表达式。

assert() 的使用对程序员是非常友好的，使用 assert() 有几个好处：

1. 它不仅能自动标识文件和出问题的行号

2. 还有一种无需更改代码就能开启或关闭 assert() 的机制。如果已经确认程序没有问题，不需要

再做断言，就在 #include &lt;assert.h&gt; 语句的前面，定义一个宏 NDEBUG 。

**课件代码 17-006 · PDF 第 4 页**

```c
#define NDEBUG
#include <assert.h>
```

然后，重新编译程序，编译器就会禁用文件中所有的 assert() 语句。如果程序又出现问题，可以移除这条 #define NDEBUG 指令（或者把它注释掉），再次编译，这样就重新启用了 assert() 语句。

assert() 的缺点是，因为引入了额外的检查，增加了程序的运行时间。

一般我们可以在 Debug 中使用，在 Release 版本中选择禁用 assert 就行，在 VS 这样的集成开发环境中，在 Release 版本中，直接就是优化掉了。这样在debug版本写有利于程序员排查问题，在 Release 版本不影响用戶使用时程序的效率。

📌 什么场景下使用 assert 呢？

一般是因为程序员自己的问题，导致程序可能出bug的地方，才会使用assert检查。

如果是用戶输入非法、文件不存在、网络断开、内存分配失败等，这些不是程序员自己的问题导致的 bug，而是现实世界可能会发生的正常异常情况。这种情况下一般使用if语句来判断。

## 4. const 限定符：用编译器保护你的数据

### 4.1 const修饰的常变量

变量是可以修改的，如果把变量的地址交给一个指针变量，通过指针变量的也可以修改这个变量。

但是如果我们希望一个变量加上一些限制，不能被修改，怎么做呢？这就是const的作用。

**课件代码 17-007 · PDF 第 5 页**

```c
#include <stdio.h>
int main()
{
    int m = 0;
    m = 20;//m是可以修改的
    const int n = 0;
    n = 20;//n是不能被修改的
    return 0;
}
```

> **原例说明：** 故意展示 const 对象赋值被拒绝；这一块不是应当编译成功的正例。

上述代码中n是不能被修改的，其实n本质是变量，只不过被const修饰后，在语法上加了限制，只要我们在代码中对n进行修改，就不符合语法规则，就报错，致使没法直接修改n。

但是如果我们绕过n，使用n的地址，去修改n就能做到了，虽然这样做是在打破语法规则。

**课件代码 17-008 · PDF 第 5 页**

```c
#include <stdio.h>
int main()
{
    const int n = 0;
    printf("n = %d\n", n);
    int* p = &n;
    *p = 20;
    printf("n = %d\n", n);
    return 0;
}
```

输出结果：

![第17章第5页配图](/images/c-language-course/17-p05-fig01.png)

程序运行结果可见这里一个确实修改了，但是我们还是要思考一下，为什么n要被const修饰呢？就是为了不能被修改，如果p拿到n的地址就能修改n，这样就打破了const的限制，这是不合理的，所以应该让p拿到n的地址也不能修改n，那接下来怎么做呢？

### 4.2 const 修饰指针变量

一般来讲const修饰指针变量，可以放在\*的左边，也可以放在\*的右边，意义是不一样的。

**课件代码 17-009 · PDF 第 6 页**

```c
int * p;//没有const修饰？
int const * p;//const 放在*的左边做修饰
int * const p;//const 放在*的右边做修饰
```

我们看下面代码，来分析具体分析一下：

**课件代码 17-010 · PDF 第 6,7 页**

```c
#include <stdio.h>
//代码1 - 测试无const修饰的情况
void test1()
{
    int n = 10;
    int m = 20;
    int* p = &n;
    *p = 20;//ok?
    p = &m; //ok?
}

//代码2 - 测试const放在*的左边情况
void test2()
{
    int n = 10;
    int m = 20;
    const int* p = &n;
    *p = 20;//ok?
    p = &m; //ok?
}

//代码3 - 测试const放在*的右边情况
void test3()
{
    int n = 10;
    int m = 20;
    int * const p = &n;
    *p = 20; //ok?
    p = &m;  //ok?
}

//代码4 - 测试*的左右两边都有const
void test4()
{
    int n = 10;
    int m = 20;
    int const * const p = &n;
    *p = 20; //ok?
    p = &m;  //ok?
}

int main()
{
    //测试无const修饰的情况
    test1();
    //测试const放在*的左边情况
    test2();
    //测试const放在*的右边情况
    test3();
    //测试*的左右两边都有const
    test4();
    return 0;
}
```

> **原例说明：** 故意对照 const 位于指针星号不同位置时的限制，标成不合法的赋值应逐项理解，不要删除限制来强行编译。

结论：const修饰指针变量的时候

- const如果放在\*的左边，修饰的是指针指向的内容，保证指针指向的内容不能通过指针来改变。

但是指针变量本身的内容可变。

- const如果放在\*的右边，修饰的是指针变量本身，保证了指针变量的内容不能修改，但是指针指

向的内容，可以通过指针改变。

### 4.3 模拟实现strlen(3)

**课件代码 17-011 · PDF 第 8 页**

```c
#include <stdio.h>
//计数器的方法
size_t my_strlen(const char *str) //加上const就意味着不希望函数内部通过str修改字符串
{
   size_t count = 0;
   while(*str != '\0')
   {
       count++;
       str++;
   }
   return count;
}

int main()
{
    printf("%zu\n", my_strlen("abcdef"));
    return 0;
}
```

## 5. 动态内存管理：谁申请谁释放，所有权要清晰

每次 malloc/calloc/realloc 必须对应一个 free ，避免内存泄漏。

当分配和释放发生在不同函数时，必须明确所有权转移的约定。

**课件代码 17-012 · PDF 第 8,9 页**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>


struct person
{
    char *name;
    int   age;
};

// 创建一个“人”，返回新分配的内存，所有权交给调用者
struct person *person_create(const char *name, int age)
{
    struct person *p = malloc(sizeof(*p));
    if (!p) return NULL;

    p->name = malloc(strlen(name) + 1);
    if (!p->name) {
        free(p);
        return NULL;
    }
    strcpy(p->name, name);
    p->age = age;
    return p;   // 所有权转交给调用者
}

// 销毁“人”，调用者交出所有权，由 destroy 完成释放
void person_destroy(struct person *p)
{
    if (p) {
        free(p->name);
        free(p);
    }
}

// 使用示例
int main(void)
{
    // 我申请，我就负责释放
    //所有权交给p
    struct person *p = person_create("Alice", 30);
    if (!p) return 1;

    printf("%s is %d years old.\n", p->name, p->age);

    person_destroy(p);  // 释放所有权
    return 0;
}
```

## 6. 别让边界问题毁了你的程序

不管是数组空间，还是动态开辟的连续内存空间，使用指针来访问内存的过程中，运算必须在合法的内存边界内进行，严禁越界访问。

## 7. 数组退化为指针时，传递长度信息

数组在传参的过程中，形参可以写成数组，也可以写成指针；即使写成数组的形式，本质上还是指针。数组退化成了指针，在函数内部访问数组，没办法计算出数组的元素个数了，一般数组传参的时候，也会传递数组的大小。

## 8. 永远不要返回栈空间的地址

返回局部数组或者返回局部变量的地址，其实都是返回栈空间地址的问题。

局部变量和数组出了作用域，就会被自动回收，所以如果在回收之前返回了内存的地址，这个时候就会导致悬空指针的问题。应该避免这样的问题。

**课件代码 17-013 · PDF 第 10 页**

```c
// 错误：返回栈内存地址
char *get_name(void)
{
    char name[] = "Alice";
    return name;
}

int main()
{
    char*p_name = get_name(); //悬空指针！

    return 0;
}
```

## 9. 用 typedef 简化函数指针类型

指针类型比较复杂的时候，会严重影响代码的可读性，可以使用typedef对函数指针、数组指针类型进行重命名，这样代码的可读性会提升，方便代码理解。

参考《指针和函数》章节。
