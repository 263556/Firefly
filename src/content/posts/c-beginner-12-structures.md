---
title: C语言初阶 12：用结构体描述复杂对象
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 初阶, 结构体]
category: C语言
series: C语言学习笔记
seriesOrder: 12
draft: false
updated: 2026-11-16
---

> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- 结构体指针使用 `->`，普通结构体对象使用 `.`。结构体整体复制会复制数组成员，但指针成员只复制地址，不自动深拷贝。

## 本章目标

1. 掌握结构体类型的声明、变量的创建和赋值

2. 掌握结构体成员访问的方法

3. 掌握结构体传参的代码编写

4. 为后续结构体进阶打好基础

## 1. 结构体

C语言已经提供了内置类型，如：char、short、int、long、float、double等，但是只有这些内置类型还是不够的，假设我想描述学生，描述一本书，这时单一的内置类型是不行的。

- 描述一个学生需要名字、年龄、学号、身高、体重等；

- 描述一本书需要书名、作者、出版社、定价等；

C语言为了解决这个问题，增加了结构体这种自定义的数据类型，让程序员可以自己创造适合的类型。

📌 结构体是一些值的集合，这些值称为成员变量。

结构体的每个成员可以是不同类型的变量，如：标量、数组、指针，甚至是其他结构体变量。

数组也是一些值的集合，但是数组中每个元素的类型是相同的，这里可以和结构体做个对比。

### 1.1 结构体类型声明

**课件代码 12-001 · PDF 第 1 页**

```c
struct tag
{
    member-list;
} variable-list;
```

描述一个学生：

**课件代码 12-002 · PDF 第 2 页**

```c
struct Stu
{
    char name[20];//名字
    int age;//年龄
    char sex[5];//性别
    char id[20];//学号
}; //分号不能丢
```

### 1.2 结构体变量的定义

方式1 方式2

**课件代码 12-003 · PDF 第 2 页**

```c
struct Stu
{
    char name[20];//名字
    int age;//年龄
    char sex[5];//性别
    char id[20];//学号
} s1, s2, s3; //全局变量
```

**课件代码 12-004 · PDF 第 2 页**

```c
struct Stu
{
    char name[20];//名字
    int age;//年龄
    char sex[5];//性别
    char id[20];//学号
};

struct Stu s1;//全局变量

int main()
{
    struct Stu s2; //局部变量

    return 0;
}
```

对于结构体变量的创建更推荐方式2，这样更加灵活。

结构体类型和结构体变量的关系，就是设计图纸和真实建筑物的关系。

![第12章第3页配图](/images/c-language-course/12-p03-fig01.png)

### 1.3 结构体变量的初始化

结构体变量中包含了多个成员，初始化的时候也使用 {} 把初始化的值括起来的。

**课件代码 12-005 · PDF 第 3,4 页**

```c
#include <stdio.h>

struct Stu
{
    char name[20];//名字
    int age;//年龄
    char sex[5];//性别
    char id[20];//学号
};

int main()
{
    //按照结构体成员的顺序初始化
    struct Stu s1 = { "张三", 20, "男", "202818001" };

    //按照指定的顺序初始化
    struct Stu s2 = { .age = 18, .name = "lisi", .id = "202818002", .sex = "女" };

    //不完全初始化
    struct Stu s3 = {"lisi", 18}; //剩余初始化的成员默认都是0

    return 0;
}
```

### 1.4 结构体变量的赋值

数组是不能整体赋值的，但是结构体变量是可以整体赋值的。

**课件代码 12-006 · PDF 第 4 页**

```c
//代码1
int main()
{
    int arr1[10] = {1,2,3,4,5,6,7,8,9,10};
    int arr2[10] = {0};
    arr2 = arr1;//err

    return 0;
}

//代码2
struct Stu
{
    char name[20];//名字
    int age;//年龄
    char sex[5];//性别
    char id[20];//学号
};

int main()
{
    struct Stu s1 = { "张三", 20, "男", "202818001" };
    struct Stu s2 = {0};
    s2 = s1; //ok
    struct Stu s3 = s2; //ok
    return 0;
}
```

## 2. 结构成员访问操作符

### 2.1 结构体成员的直接访问

结构体成员的直接访问是通过点操作符（.）访问的；点操作符接受两个操作数。如下所示：

**课件代码 12-007 · PDF 第 5 页**

```c
#include <stdio.h>

struct Stu
{
    char name[20];//名字
    int age;//年龄
    char sex[5];//性别
    char id[20];//学号
};

int main()
{
    struct Stu s = { "张三", 20, "男", "20230818001" };
    s.age = 18;
    printf("name: %s\n", s.name);
    printf("age : %d\n", s.age);
    printf("sex : %s\n", s.sex);
    printf("id  : %s\n", s.id);
    return 0;
}
```

使用方式：结构体变量.成员名

### 2.2 结构体成员的间接访问

有时候我们得到的不是一个结构体变量，而是得到了一个指向结构体的指针。如下所示：

**课件代码 12-008 · PDF 第 5,6 页**

```c
#include <stdio.h>
struct Stu
{
    char name[20];//名字
    int age;//年龄
    char sex[5];//性别
    char id[20];//学号
};

int main()
{
    struct Stu s = { "张三", 20, "男", "20230818001" };
    struct Stu *ps = &s;
    printf("name: %s\n", ps->name);
    printf("age : %d\n", ps->age);
    printf("sex : %s\n", ps->sex);
    printf("id  : %s\n", ps->id);
    return 0;
}
```

使用方式：结构体指针-&gt;成员名

## 3. 结构体传参

在使用结构体的时候，经常会涉及到结构体传参，也就是将一个结构体的信息传递给某个函数。

结构体传参的时候，可以传递结构体变量，也可以传递结构体变量的地址。

请看下面的例子：

**课件代码 12-009 · PDF 第 6 页**

```c
struct S
{
    int data[1000];
    int num;
};

struct S s = {{1,2,3,4}, 1000};
//结构体传参
void print1(struct S s)
{
    printf("%d\n", s.num);
}
//结构体地址传参
void print2(struct S* ps)
{
    printf("%d\n", ps->num);
}

int main()
{
    print1(s);  //传结构体
    print2(&s); //传地址
    return 0;
}
```

上面的 print1 和 print2 函数哪个好些？

答案是：首选print2函数。

原因：

函数传参的时候，参数是需要压栈，会有时间和空间上的系统开销。

如果传递一个结构体对象的时候，结构体过大，参数压栈的的系统开销比较大，所以会导致性能的下降。

结论：结构体传参的时候，要传结构体的地址。

更多关于结构体的知识，后期在《第20讲：结构体进阶》中讲解。
