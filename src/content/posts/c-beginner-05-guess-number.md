---
title: C语言初阶 05：用猜数字游戏串起分支与循环
published: 2026-11-14
description: 理解 rand、srand、time 和随机范围，完成支持菜单、次数限制、输入检查与计时的猜数字游戏。
tags: [C语言, 初阶, 猜数字, 项目实践]
category: C语言
series: C语言初阶笔记
seriesOrder: 5
draft: false
---

> 依据《（新 AI 版）鹏哥 C 语言课件》初阶第 5 讲整理。本篇把课件的基本游戏和扩展要求合成一个完整示例。

## 1. 引入：把“重复猜”变成程序

游戏规则：电脑选定 1 到 100 的整数，玩家输入猜测，电脑提示“大了”“小了”，猜中结束。外层菜单允许重复开局，内层循环完成一局游戏。

先拆职责：`main` 管理菜单和初始化，`play_game` 管理一局，`read_int` 负责读取一个整数。函数让“输入失败”和“猜错了”成为不同情况，不会混在一起。

## 2. rand 产生的是伪随机序列

```c
#include <stdlib.h>

int number = rand(); // 0 到 RAND_MAX，包含两个端点
```

`rand()` 不是读取真正的随机现象，而是按算法从内部状态生成序列。相同实现、相同种子，会重复同一序列。`RAND_MAX` 的具体值由实现定义，至少为 32767，不能认定所有编译器都一样。

```c
#include <stdlib.h>
#include <time.h>

srand((unsigned int)time(NULL));
```

`srand` 设置种子，一般在 `main` 开始时调用一次。不要在每次 `rand` 前或每次开局时反复播种：同一秒内获得相同时间，可能反复重置成同一序列。

`time(NULL)` 返回当前日历时间的表示值，常见系统以秒时间戳表示。它的 C 类型为 `time_t`，不应假设所有平台都把它实现成某个固定整数类型。测试时使用 `srand(1)` 可重现序列，排错后再切回时间种子。

## 3. 限定随机范围

```c
rand() % 100;       // 0..99
rand() % 100 + 1;   // 1..100
100 + rand() % 101; // 100..200
```

小范围教学示例可用 `a + rand() % (b - a + 1)`，前提是 `a <= b`、区间宽度及结果可表示，且随机源足以覆盖目标范围。取模法可能产生轻微分布偏差，不适合需要严格均匀或密码学安全的场景；本篇游戏不依赖这些性质。

## 4. 一局游戏的状态

一局需要三个主要值：`target`（答案）、`guess`（猜测）、`attempt`（次数）。答案只在开局生成一次，循环中不能重新生成，否则玩家面对的目标会不断变化。

```text
生成答案 → 读入猜测 → 比较
                   ├─ 相等：获胜并返回
                   ├─ 太小：继续
                   └─ 太大：继续
超过次数限制 → 公布答案并返回
```

边界输入需要明确处理：非数字输入、超出 1..100、结束输入。下例约定：范围外输入不消耗次数；输入格式错误或 EOF 结束当前程序。这种规则简单且便于验证。

## 5. 完整可运行代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <limits.h>
#include <errno.h>
#include <ctype.h>
#include <string.h>

/* 一行恰好一个 int；不接受尾随的非空白字符。 */
static int read_int(int *out)
{
    char line[128];
    char *end;
    long value;

    if (fgets(line, sizeof line, stdin) == NULL) {
        return 0;
    }
    if (strchr(line, '\n') == NULL && !feof(stdin)) {
        int ch;
        while ((ch = getchar()) != '\n' && ch != EOF) {
        }
        return 0; // 行太长，拒绝截断输入
    }

    errno = 0;
    value = strtol(line, &end, 10);
    if (end == line || errno == ERANGE || value < INT_MIN || value > INT_MAX) {
        return 0;
    }
    while (isspace((unsigned char)*end)) {
        ++end;
    }
    if (*end != '\0') {
        return 0;
    }
    *out = (int)value;
    return 1;
}

/* 正常结束一局返回 1；输入失败返回 0。 */
static int play_game(void)
{
    const int target = rand() % 100 + 1;
    const int limit = 7;
    time_t start = time(NULL);

    for (int attempt = 1; attempt <= limit; ) {
        int guess;
        printf("第 %d/%d 次，猜 1..100：", attempt, limit);
        if (!read_int(&guess)) {
            return 0;
        }
        if (guess < 1 || guess > 100) {
            puts("超出范围，不计次数。");
            continue;
        }
        if (guess == target) {
            printf("猜对了！用了 %d 次。\n", attempt);
            time_t finish = time(NULL);
            if (start != (time_t)-1 && finish != (time_t)-1) {
                printf("用时 %.0f 秒。\n", difftime(finish, start));
            }
            return 1;
        }
        puts(guess < target ? "猜小了。" : "猜大了。");
        ++attempt;
    }
    printf("次数用完，答案是 %d。\n", target);
    return 1;
}

int main(void)
{
    srand((unsigned int)time(NULL));
    for (;;) {
        int choice;
        puts("\n1. 开始游戏\n0. 退出");
        if (!read_int(&choice)) {
            puts("输入结束或格式无效，程序退出。");
            return 0;
        }
        switch (choice) {
        case 1:
            if (!play_game()) {
                puts("输入结束或格式无效，程序退出。");
                return 0;
            }
            break;
        case 0:
            puts("再见！");
            return 0;
        default:
            puts("请选择 0 或 1。");
            break;
        }
    }
}
```

输入辅助函数涉及后续才会深入学习的指针和字符串，可先把它当作“成功读到整数返回 1”的接口。游戏主流程仍由已经学过的条件和循环组成。

## 6. 关键设计解释

- `for` 不在第三项自动递增，因为范围错误不应消耗机会；有效且没猜中的分支才递增。
- 猜对后直接 `return`，结束本局但不结束整个菜单。
- 菜单中的 `break` 只结束 `switch`，随后仍会进入下一轮菜单。
- 计时用 `difftime` 求时间差，不直接假定 `time_t` 的底层类型。
- 输入通过 `fgets` 和 `strtol` 分成“读取一行”和“验证整数”两步，避免失败字符留在输入流中反复触发错误。

## 7. 怎么测才有意义

先固定种子，并在本地调试时暂时打印答案，测试猜大、猜小、第一次猜中和最后一次猜中；再测试用完机会、范围外值、输入字母、空行和 EOF。测试完成后删除显示答案的日志。

合理策略是每次猜剩余区间中点：1..100 猜 50，得到“小了”后范围变为 51..100，再猜 75。这就是逐步缩小候选范围的思想，与后续二分查找相通。

扩展练习：增加难度（范围与次数）、记录最佳成绩、每次失败显示剩余次数。每次只改一个功能，先写规则再实现。
