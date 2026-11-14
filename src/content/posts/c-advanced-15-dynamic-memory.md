---
title: C语言进阶 15：动态内存、realloc 与柔性数组
published: 2026-11-14
description: 用可增长成绩表理解 malloc、calloc、realloc、free，建立容量检查与所有权规则，并实现柔性数组对象。
tags: [C语言, 进阶, 动态内存]
category: C语言
series: C语言进阶笔记
seriesOrder: 15
draft: false
---

> 依据鹏哥 C 语言进阶第 15 章整理。以下零大小分配规则以 C17 为基准；示例主动避开零大小 realloc。

## 1. 引入：人数运行时才知道怎么办

固定数组容量会过小或浪费空间，自动局部数组在作用域结束后也不能继续使用。动态内存让程序按运行时需求申请存储，并明确决定何时释放。

分配函数通常由实现的堆分配器管理，但 C 语言重点规定的是分配与生命周期语义，不要求每个平台都有完全相同的“栈区/堆区”布局。获得内存后，程序同时承担边界、初始化和释放责任。

## 2. 四个接口与关键契约

接口位于 `<stdlib.h>`：

| 函数 | 用途 | 需要注意 |
| --- | --- | --- |
| `malloc(bytes)` | 分配指定字节数 | 内容未初始化；失败返回 NULL |
| `calloc(count, size)` | 为多个元素分配并把字节清零 | 全零字节不普遍等于所有类型的“零值” |
| `realloc(p, bytes)` | 调整已有分配大小 | 成功可能迁移；失败保留原块 |
| `free(p)` | 释放整个分配块 | 参数须为有效分配起点或 NULL |

`void *` 在 C 中可转换为对象指针，因此 `int *p = malloc(n * sizeof *p);` 不需要显式强转。一定包含正确头文件，强转不能补救缺少声明。

分配大小单位是字节，不是元素个数。写 `sizeof *p` 比硬编码 4 更便于维护，类型改动时不必到处改大小。

## 3. 分配前先检查乘法

若 `n * sizeof *p` 已经因 `size_t` 溢出而变小，分配可能成功，但后面按 n 个元素访问会越界。检查必须发生在乘法之前。

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

int main(void)
{
    size_t count = 5;
    int *scores = NULL;
    if (count == 0 || count > SIZE_MAX / sizeof *scores) {
        return 1;
    }
    scores = malloc(count * sizeof *scores);
    if (scores == NULL) {
        fputs("分配失败\n", stderr);
        return 1;
    }
    for (size_t i = 0; i < count; ++i) {
        scores[i] = 60 + (int)i;
        printf("%d ", scores[i]);
    }
    putchar('\n');
    free(scores);
    scores = NULL;
    return 0;
}
```

`malloc` 成功不代表所有元素已有值，必须先写后读。`calloc` 清零适合整数零初始化，但不能泛化为“所有指针都是空指针、所有浮点数都是数值 0”，因为语言不保证它们的零值表示均为全零字节。

C17 的 `malloc(0)` 不是一概未定义，存在实现定义的结果；即使返回非空，也不能据此访问对象。实际接口最好明确规定空集合如何处理，避免依赖这个边角行为。

## 4. realloc：只在成功后替换拥有者

```c
int *temporary = realloc(scores, new_count * sizeof *scores);
if (temporary != NULL) {
    scores = temporary;
} else {
    // scores 仍持有原分配，可以继续使用或释放。
}
```

先检查新大小乘法可表示。不能直接 `scores = realloc(scores, bytes)`，否则失败返回 NULL 后可能丢失原块的唯一地址，造成泄漏。

当新大小非零：成功后旧对象生命周期结束，应只使用新返回的指针，所有旧元素指针和别名都需重新获取，即使数值地址看起来没变；失败则原对象保持不变。新增区域不自动初始化，缩小会丢弃尾部数据。

`realloc(NULL, n)` 类似 `malloc(n)`；需要释放时直接用 `free`，不要把 `realloc(p, 0)` 当作统一释放方案，其跨标准行为不同。

### 完整案例：成绩表由 3 项扩到 5 项

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

int main(void)
{
    size_t count = 3;
    size_t new_count = 5;
    int *scores = malloc(count * sizeof *scores);
    if (scores == NULL) {
        return 1;
    }
    for (size_t i = 0; i < count; ++i) {
        scores[i] = 70 + (int)i;
    }

    if (new_count > SIZE_MAX / sizeof *scores) {
        free(scores);
        return 1;
    }
    int *temporary = realloc(scores, new_count * sizeof *scores);
    if (temporary == NULL) {
        free(scores);
        return 1;
    }
    scores = temporary;
    for (size_t i = count; i < new_count; ++i) {
        scores[i] = 0;
    }
    count = new_count;
    for (size_t i = 0; i < count; ++i) {
        printf("%d ", scores[i]);
    }
    putchar('\n');
    free(scores);
    return 0;
}
```

输出 `70 71 72 0 0`。更完整的容器还应区分已用元素数 `size` 与分配容量 `capacity`，通常按倍数扩容，避免每添加一项都重新分配。

## 5. free、别名与所有权

`free` 不会自动把指针变量设为 NULL，也不保证立刻把物理内存还给操作系统。它结束对应分配对象的生命周期，使存储可被分配器再利用。

```c
int *owner = malloc(sizeof *owner);
// 检查分配成功后：
int *alias = owner;
free(owner);
owner = NULL;
// alias 也已失效，不能再解引用或再次 free。
```

置 NULL 只更新当前变量，不能修复其他别名。良好接口应说明谁拥有内存、谁只是临时借用、谁负责释放。`free(NULL)` 合法且不做事，但 `free` 不能检测任意非空地址是否可释放。

| 常见错误 | 原因 |
| --- | --- |
| 不检查分配结果就解引用 | 可能访问 NULL |
| 循环使用 `i <= count` | 多访问一个元素 |
| `free(&local)` | 不是动态分配起点 |
| `free(p + 1)` | 不是原块起始指针 |
| 两次 `free(p)` | 已释放对象被再次释放 |
| 提前 return 未清理 | 内存泄漏 |
| 返回局部数组地址 | 返回后对象已失效 |

`void create(char *p)` 中给 p 分配内存只改变形参副本。可以返回新地址，或接收 `char **` 修改调用者指针；不论哪种方式，都要约定失败和释放责任。

## 6. 柔性数组：元数据与内容一次分配

C99 允许结构体最后一个成员是未知长度数组，前面至少有一个其他具名成员：

```c
struct Scores {
    size_t count;
    int values[];
};
```

标准写法是 `[]`，`[0]` 是部分编译器扩展。`sizeof(struct Scores)` 不包含实际元素存储，但可能包含填充；需在基础大小后追加元素空间。

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

struct Scores {
    size_t count;
    int values[];
};

int main(void)
{
    size_t n = 4;
    struct Scores *scores;
    if (n > (SIZE_MAX - sizeof *scores) / sizeof scores->values[0]) {
        return 1;
    }
    scores = malloc(sizeof *scores + n * sizeof scores->values[0]);
    if (scores == NULL) {
        return 1;
    }
    scores->count = n;
    for (size_t i = 0; i < scores->count; ++i) {
        scores->values[i] = 80 + (int)i;
        printf("%d ", scores->values[i]);
    }
    putchar('\n');
    free(scores);
    return 0;
}
```

它只需一次申请、一次释放，数据相邻，适合“头部信息 + 可变长度内容”。但结构体整体赋值不会复制柔性尾部内容；扩容也可能迁移整个对象，旧成员指针全部失效。

## 7. 验证与练习

测试零元素约定、单元素、扩容、缩容和分配失败清理路径；使用支持的 AddressSanitizer 或内存工具排查越界、释放后使用和泄漏。仅测试“申请成功的路径”远远不够。

练习：实现 `create_scores(count)` 与 `destroy_scores`；给成绩表增加追加接口，成功更新状态，失败保持原对象不变。每个接口都写一句所有权说明，再开始写代码。
