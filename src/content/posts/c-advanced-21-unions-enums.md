---
title: C语言进阶 21：联合体、枚举与带标签的数据模型
published: 2026-11-14
description: 用礼品目录与红绿灯理解共享存储和有限状态，掌握联合体大小、枚举取值、标签校验及跨平台表示边界。
tags: [C语言, 进阶, 联合体, 枚举]
category: C语言
series: C语言进阶笔记
seriesOrder: 21
draft: false
---

> 依据鹏哥 C 语言进阶第 21 章整理。

## 1. 引入：一本书不需要保存杯子的属性

礼品目录有图书、杯子和衬衫，每种都有库存、价格，但只有图书需要作者和页数，只有衬衫需要尺寸。如果把所有属性放进每个商品，既浪费空间，也容易访问与类型无关的字段。

解决思路：公共属性使用结构体同时保存，互斥属性使用 union 共用存储，再用 enum 标签记录当前是哪一种商品。

## 2. struct 与 union 的根本差异

```c
struct Both {
    int number;
    double decimal;
};

union Either {
    int number;
    double decimal;
};
```

Both 的两个成员同时存在于不同存储位置；Either 的成员共享存储，从相同起点解释内容。给 decimal 写值会覆盖 number 使用的表示，不能当成两个独立变量。

```c
#include <stdio.h>

union Value {
    int number;
    double decimal;
};

int main(void)
{
    union Value value = {.number = 10};
    printf("integer: %d\n", value.number);
    value.decimal = 2.5;
    printf("decimal: %.1f\n", value.decimal);
    printf("union=%p integer=%p decimal=%p\n",
           (void *)&value, (void *)&value.number, (void *)&value.decimal);
    return 0;
}
```

三个地址的起点相同，但对应类型不同。不要在写入 decimal 后仍把 number 当作独立保留的旧值。

## 3. 联合体大小还要满足对齐

联合体至少能容纳最大成员，整体大小还可能为了对齐补齐。不能总说“严格等于最大成员的 sizeof”。

```c
union Example {
    char bytes[5];
    int number;
};
```

若 int 大小及对齐为 4，常见结果为 8：五字节数组需要五字节，但联合体数组里的每个 number 仍要正确对齐。实际值用 sizeof、`_Alignof`（C11）核实，不从某个平台推导全部平台。

结构体和联合体内的填充、成员表示、字节序也会影响直接内存读写，不适合直接当作稳定网络格式。

## 4. 跨成员读取与字节观察

在 C 中，读取最后写入成员之外的成员涉及对对象表示的重新解释，结果依赖表示细节，某些类型组合可能得到陷阱表示；不能给任意组合承诺可移植数值。这也与 C++ 的对象规则不能简单混用。

课件通过 union 的 int 和 char 判断大小端，可帮助观察特定环境。但若目的只是查看字节，直接使用 `const unsigned char *` 观察对象表示更明确，不必引入不相关成员的解释。

union 不会自动记录最后写入了哪个成员，也不会在访问错误成员时主动报错，所以它通常需要一个显式标签。

## 5. enum 为状态命名

```c
enum Light {
    LIGHT_RED,
    LIGHT_GREEN,
    LIGHT_YELLOW,
    LIGHT_COUNT
};
```

默认从 0 开始，每项比前项大 1；指定数值后，后续继续递增。可以显式定义协议值，如 `STATUS_OK = 0`、`STATUS_FAILED = 10`，但不能再用“最大值 + 1”等同于元素数量。

枚举比到处写 0、1、2 更清楚，调试器通常也能显示符号名称。C17 中枚举常量是 int 范围内的整数常量表达式；枚举对象的兼容整数类型由实现选择，不应假定 sizeof 永远等于 int。

C 的 enum 不是严格禁止其他整数的封闭集合。输入一个整数并转换成 enum，不会自动验证它是列出的状态；用作数组下标前仍需检查范围。

枚举常量还遵循作用域规则，与 C++ 的 scoped enum 不同，常用前缀避免名称冲突。

## 6. 完整案例：带标签的礼品目录

```c
#include <stdio.h>

enum GiftType {
    GIFT_BOOK,
    GIFT_MUG,
    GIFT_SHIRT
};

struct Gift {
    enum GiftType type;
    unsigned int stock;
    unsigned int price_cents;
    union {
        struct {
            char title[32];
            char author[24];
            unsigned int pages;
        } book;
        struct {
            char design[32];
        } mug;
        struct {
            char design[32];
            char size[8];
        } shirt;
    } detail;
};

int print_gift(const struct Gift *gift)
{
    if (gift == NULL) {
        return 0;
    }
    printf("stock=%u price=%u.%02u\n", gift->stock,
           gift->price_cents / 100, gift->price_cents % 100);
    switch (gift->type) {
    case GIFT_BOOK:
        printf("book: %s / %s, %u pages\n", gift->detail.book.title,
               gift->detail.book.author, gift->detail.book.pages);
        return 1;
    case GIFT_MUG:
        printf("mug: %s\n", gift->detail.mug.design);
        return 1;
    case GIFT_SHIRT:
        printf("shirt: %s, %s\n", gift->detail.shirt.design, gift->detail.shirt.size);
        return 1;
    default:
        return 0;
    }
}

int main(void)
{
    struct Gift book = {
        .type = GIFT_BOOK,
        .stock = 5,
        .price_cents = 3900,
        .detail = {.book = {"Learn C", "Lin", 240}}
    };
    struct Gift mug = {
        .type = GIFT_MUG,
        .stock = 8,
        .price_cents = 2500,
        .detail = {.mug = {"Firefly"}}
    };
    if (!print_gift(&book) || !print_gift(&mug)) {
        return 1;
    }
    return 0;
}
```

标签与实际写入的 union 成员必须同步。不能先写 book，再只把 type 改成 GIFT_SHIRT 就认为得到一件合法衬衫。函数只能检查标签范围，不能凭空证明调用者初始化了正确的内容。

若 union 成员含拥有型指针，切换类型前还需释放旧成员资源，再建立新成员；普通整体赋值也不会自动完成深拷贝或资源管理。

## 7. 红绿灯状态机：数据与转移分开

```c
#include <stdio.h>

enum Light { LIGHT_RED, LIGHT_GREEN, LIGHT_YELLOW, LIGHT_COUNT };

enum Light next_light(enum Light current)
{
    switch (current) {
    case LIGHT_RED: return LIGHT_GREEN;
    case LIGHT_GREEN: return LIGHT_YELLOW;
    case LIGHT_YELLOW: return LIGHT_RED;
    default: return LIGHT_RED;
    }
}

int main(void)
{
    const char *names[] = {"red", "green", "yellow"};
    const int seconds[] = {10, 10, 3};
    enum Light current = LIGHT_RED;
    for (int step = 0; step < 6; ++step) {
        if ((unsigned int)current >= LIGHT_COUNT) {
            return 1;
        }
        printf("%s: %d seconds\n", names[current], seconds[current]);
        current = next_light(current);
    }
    return 0;
}
```

示例有限运行，不依赖 Windows 的 Sleep，也不让测试永久等待。显式转移函数比直接 `(current + 1) % count` 更适合状态值不连续或转移规则变复杂的情形。非法状态如何处理应由需求决定，本例统一恢复到红灯。

## 8. 选择与练习

多个字段需要同时保存用 struct；同一时刻仅一种数据形态用 union；有限状态或类别用 enum；多种独立开关应使用位掩码，不要把“可组合权限”误建模成只能选一个的状态。

练习：为礼品增加电子券类别，修改标签、数据和 switch 三处；为状态机增加“闪烁黄灯”维护状态，确认数组索引和转移规则没有依赖旧的连续编号假设。
