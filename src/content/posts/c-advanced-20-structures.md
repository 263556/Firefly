---
title: C语言进阶 20：结构体嵌套、自引用、对齐与位域
published: 2026-11-14
description: 用学生信息与通讯录理解结构体组合和自引用，实测 sizeof 与 offsetof，区分自然对齐、打包、位域和稳定数据格式。
tags: [C语言, 进阶, 结构体, 内存对齐]
category: C语言
series: C语言进阶笔记
seriesOrder: 20
draft: false
---

> 依据鹏哥 C 语言进阶第 20 章整理。布局示例明确区分语言规则与特定 ABI 的常见结果。

## 1. 引入：学生生日也是一个对象

姓名、学号与生日共同描述学生；生日又包含年、月、日。嵌套结构体让层次清晰，也使日期类型能被其他对象复用。

```c
#include <stdio.h>

struct Date {
    int year;
    int month;
    int day;
};

typedef struct Student {
    char name[20];
    struct Date birthday;
    char id[20];
} Student;

int main(void)
{
    Student students[] = {
        {"Lin", {2008, 8, 8}, "001"},
        {"Chen", {2009, 3, 20}, "002"}
    };
    for (size_t i = 0; i < sizeof students / sizeof students[0]; ++i) {
        const Student *student = &students[i];
        printf("%s %s %d-%02d-%02d\n", student->id, student->name,
               student->birthday.year, student->birthday.month, student->birthday.day);
    }
    return 0;
}
```

`typedef` 使 `Student` 成为类型别名，不创建变量，也不改变结构体布局。学号用字符串可保留前导零，不必为了“看上去是数字”就选 int。

## 2. 自引用必须通过指针

链表节点需要保存数据并找到下一个节点：

```c
typedef struct Node {
    int value;
    struct Node *next;
} Node;
```

不能在里面写 `struct Node next;`，因为结构体尚未完整定义，且这种按值嵌套会要求无限展开。指针大小不取决于完整节点大小，因此可以指向尚未完成的本类型。

```c
#include <stdio.h>

struct Node {
    int value;
    struct Node *next;
};

int main(void)
{
    struct Node last = {30, NULL};
    struct Node middle = {20, &last};
    struct Node first = {10, &middle};
    for (const struct Node *p = &first; p != NULL; p = p->next) {
        printf("%d ", p->value);
    }
    putchar('\n');
    return 0;
}
```

这里节点是自动对象，不应 free；若改为动态节点，则需要逐个释放。遍历前进要使用当前节点的 next，不能写错成永远回到头节点。

## 3. 匿名类型与匿名成员并不相同

两次独立写出的匿名结构体定义，即使成员完全一样，也不是同一类型。需要跨函数共享时使用具名类型或 typedef。

```c
typedef struct {
    int x;
    int y;
} Point;
```

这种匿名类型有 typedef 名称，可反复声明 Point 对象，所以不能笼统说“匿名结构体只能使用一次”。没有任何名字的类型才难以在后续重新引用。

C11 的匿名结构体/联合体成员允许省略中间成员名直接访问。若子结构有成员名，如 `position`，则仍要写 `object.position.x`。匿名成员可以减少层级，但会增加命名冲突风险，不宜为了省几个字符滥用。

## 4. sizeof 为什么不等于成员大小之和

```c
struct Compact {
    char first;
    char second;
    int value;
};

struct Scattered {
    char first;
    int value;
    char second;
};
```

在 char 为 1 字节、int 为 4 字节且对齐为 4 的常见 ABI 下，Compact 通常占 8 字节，Scattered 通常占 12 字节。差异来自成员间与末尾的填充。

推导 Scattered：first 在偏移 0；补 3 字节使 int 位于偏移 4；second 位于偏移 8；末尾再补 3，使数组中下一个结构体仍满足对齐。成员声明顺序保持，编译器不会擅自把 second 搬到 first 后来省空间。

**对齐要求不等于类型大小的普遍定律。** 课件的“成员大小与默认打包值取较小值”适合解释某些编译器设置，不能推广到所有平台。使用 `_Alignof`、`offsetof` 和当前 ABI 文档核实。

```c
#include <stdio.h>
#include <stddef.h>

struct Scattered {
    char first;
    int value;
    char second;
};

int main(void)
{
    printf("size=%zu, alignment=%zu\n", sizeof(struct Scattered), _Alignof(struct Scattered));
    printf("first=%zu value=%zu second=%zu\n",
           offsetof(struct Scattered, first),
           offsetof(struct Scattered, value),
           offsetof(struct Scattered, second));
    return 0;
}
```

该例需要 C11。具体数值取决于平台，但可据此验证“成员偏移满足对齐、整体大小容许数组连续排列”的解释。

## 5. 嵌套对齐、布局优化与 pack

嵌套对象也有自己的大小和对齐要求，外层必须为它留完整空间。外层大小不能只把内层成员拆开相加。内层尾部填充仍然属于内层对象的 sizeof。

把相近对齐要求的成员集中，有时能减少空洞；但稳定接口、可读性和缓存访问模式也重要。已公开的二进制接口随意调整成员顺序可能破坏 ABI。

某些编译器支持 `#pragma pack(push, 1)` / `#pragma pack(pop)` 控制打包。这属于实现扩展，可能产生未对齐成员，拿成员地址传给要求自然对齐的接口会出问题。不要以为 pack(1) 自动让结构体成为可移植网络包：字节序、类型宽度、浮点表示等问题仍在。

文件和网络格式更适合明确字段宽度与编码，逐字段序列化，而不是直接 `fwrite(&object, sizeof object, 1, file)` 当作永久格式。

## 6. 位域：用少量位描述范围

```c
struct Flags {
    unsigned int ready : 1;
    unsigned int mode : 3;
    unsigned int : 0;
    unsigned int count : 4;
};
```

ready 表示 0..1，mode 表示 0..7。无名非零宽度字段保留位，零宽度无名字段约束后续位域的存储单元边界。位域可减少某些对象的表示空间，但实际分配方向、跨单元方式和大小依赖实现。

C17 保证的基础位域类型包含 `_Bool`、`signed int`、`unsigned int` 和 int；其他类型是否支持依赖实现。普通 int 位域的有符号性也不能盲猜，非负状态优先显式 unsigned。

不能对位域取地址，也不能对位域表达式直接使用 sizeof，因此 `scanf("%u", &flags.mode)` 不合法。应先读入普通变量、验证不超过 7，再赋给 mode。协议解码更宜使用掩码与移位，避免依赖位域的内存布局。

## 7. 通讯录案例：结构体数组与逻辑长度

课件提出静态通讯录与动态改造。本例给出静态核心：添加、查找、删除，控制台命令解析可复用字符串章节。

```c
#include <stdio.h>
#include <string.h>
#include <stddef.h>

struct Contact {
    char name[32];
    char phone[24];
};

struct Book {
    struct Contact items[8];
    size_t count;
};

int book_add(struct Book *book, const char *name, const char *phone)
{
    if (book == NULL || name == NULL || phone == NULL || book->count >= 8) {
        return 0;
    }
    size_t n = strlen(name);
    size_t p = strlen(phone);
    if (n == 0 || n >= sizeof book->items[0].name ||
        p == 0 || p >= sizeof book->items[0].phone) {
        return 0;
    }
    struct Contact *item = &book->items[book->count];
    memcpy(item->name, name, n + 1);
    memcpy(item->phone, phone, p + 1);
    ++book->count;
    return 1;
}

const struct Contact *book_find(const struct Book *book, const char *name)
{
    for (size_t i = 0; i < book->count; ++i) {
        if (strcmp(book->items[i].name, name) == 0) {
            return &book->items[i];
        }
    }
    return NULL;
}

int book_erase(struct Book *book, size_t index)
{
    if (index >= book->count) {
        return 0;
    }
    for (size_t i = index + 1; i < book->count; ++i) {
        book->items[i - 1] = book->items[i];
    }
    --book->count;
    return 1;
}

int main(void)
{
    struct Book book = {.count = 0};
    if (!book_add(&book, "Lin", "13800000000") ||
        !book_add(&book, "Chen", "13900000000")) {
        return 1;
    }
    const struct Contact *found = book_find(&book, "Chen");
    if (found != NULL) {
        printf("%s %s\n", found->name, found->phone);
    }
    if (!book_erase(&book, 0)) {
        return 1;
    }
    printf("remaining: %zu\n", book.count);
    return 0;
}
```

本例查找和删除为内部使用约定，要求传入非空且状态合法的 book，查找 name 为有效字符串。电话使用字符串，避免丢失前导零、国家区号和超范围。查找返回借用地址，删除或重排后不能假定旧地址仍代表同一个联系人。

## 8. 动态改造与练习

后续把固定数组改为“指针、size、capacity”或柔性数组：扩容先检查乘法，成功后更新拥有者，失败保持原状态。原来的借用地址在 realloc 后需要重新获取。

练习：把通讯录按姓名排序；增加重复姓名策略；分别计算成员重排前后的偏移，再用 offsetof 核实。位域练习只比较字段值，不把某次机器输出的布局当作所有平台的规范。
