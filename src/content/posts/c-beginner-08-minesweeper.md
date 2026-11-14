---
title: C语言初阶 08：从零组织命令行扫雷项目
published: 2026-11-14
description: 完整实现基础扫雷，讲清双棋盘、边界填充、布雷去重、邻雷统计、胜负计数和三文件工程。
tags: [C语言, 初阶, 扫雷, 项目实践]
category: C语言
series: C语言初阶笔记
seriesOrder: 8
draft: false
---


## 1. 引入：先分析规则，再写 main

基础规则：9×9 棋盘随机放置 10 颗雷，输入坐标翻开格子；踩雷失败，翻开全部非雷格子获胜。无雷位置显示周围八格的雷数，重复选择已翻开的格子不计进度。

先明确数据和不变量：雷的位置开局确定后不变；玩家看到的内容逐步更新；已翻开的安全格数只在首次翻开时加一。这个版本不保证第一次点击安全，不自动展开空白区，也没有旗帜功能。

## 2. 为什么需要两个棋盘

| 数组 | 内容 | 初始状态 |
| --- | --- | --- |
| `mine` | 真实雷区：字符 `'0'`、`'1'` | 全 `'0'` |
| `show` | 玩家可见：`'*'`、`'0'`..`'8'` | 全 `'*'` |

如果只用一个数组，显示内容可能泄露答案，或把原始雷区覆盖掉。分离真实数据与展示数据，逻辑更稳定。

为了读取八个邻居，在 9×9 外加一圈，实际数组为 11×11。可见坐标使用 1..9，边界使用 0 和 10，边界全无雷。邻居循环因此不必对每一格单独判断越界，但**输入坐标仍必须检查**。

## 3. 文件职责

`game.h` 定义常量与接口；`game.c` 实现棋盘逻辑；`main.c` 管理菜单与随机种子。下面三段代码分别保存为对应文件，不要拼成包含多个头文件保护块的单个工程片段。

### game.h

```c
#ifndef GAME_H
#define GAME_H

#define ROW 9
#define COL 9
#define ROWS (ROW + 2)
#define COLS (COL + 2)
#define MINE_COUNT 10

/* 返回 0 表示输入结束或格式错误，返回 1 表示正常完成一局。 */
int play_game(void);

#endif
```

宏表达式加括号，避免后续参与乘法等运算时被优先级改变含义。布雷数量必须在 0..`ROW * COL` 范围内，否则去重布雷可能永远无法完成。

### game.c

```c
#include "game.h"
#include <stdio.h>
#include <stdlib.h>

static void init_board(char board[ROWS][COLS], char value)
{
    for (int r = 0; r < ROWS; ++r) {
        for (int c = 0; c < COLS; ++c) {
            board[r][c] = value;
        }
    }
}

static void print_board(char board[ROWS][COLS])
{
    printf("  ");
    for (int c = 1; c <= COL; ++c) {
        printf("%d ", c);
    }
    putchar('\n');
    for (int r = 1; r <= ROW; ++r) {
        printf("%d ", r);
        for (int c = 1; c <= COL; ++c) {
            printf("%c ", board[r][c]);
        }
        putchar('\n');
    }
}

static void place_mines(char mine[ROWS][COLS])
{
    int placed = 0;
    while (placed < MINE_COUNT) {
        int r = rand() % ROW + 1;
        int c = rand() % COL + 1;
        if (mine[r][c] == '0') {
            mine[r][c] = '1';
            ++placed;
        }
    }
}

static int count_neighbors(char mine[ROWS][COLS], int row, int col)
{
    int count = 0;
    for (int r = row - 1; r <= row + 1; ++r) {
        for (int c = col - 1; c <= col + 1; ++c) {
            if ((r != row || c != col) && mine[r][c] == '1') {
                ++count;
            }
        }
    }
    return count;
}

int play_game(void)
{
    char mine[ROWS][COLS];
    char show[ROWS][COLS];
    int opened = 0;

    init_board(mine, '0');
    init_board(show, '*');
    place_mines(mine);

    while (opened < ROW * COL - MINE_COUNT) {
        int row;
        int col;
        print_board(show);
        printf("输入行和列（1..9）：");
        if (scanf("%d %d", &row, &col) != 2) {
            return 0;
        }
        if (row < 1 || row > ROW || col < 1 || col > COL) {
            puts("坐标越界，请重新输入。");
            continue;
        }
        if (show[row][col] != '*') {
            puts("这一格已经翻开。");
            continue;
        }
        if (mine[row][col] == '1') {
            puts("踩雷，游戏结束。雷区如下（1 表示雷）：");
            print_board(mine);
            return 1;
        }
        show[row][col] = (char)('0' + count_neighbors(mine, row, col));
        ++opened;
    }
    print_board(show);
    puts("全部安全格已翻开，你赢了！");
    return 1;
}
```

### main.c

```c
#include "game.h"
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main(void)
{
    srand((unsigned int)time(NULL));
    for (;;) {
        int choice;
        puts("\n1. 扫雷\n0. 退出");
        if (scanf("%d", &choice) != 1) {
            puts("输入结束或格式错误。");
            return 0;
        }
        if (choice == 0) {
            return 0;
        }
        if (choice == 1) {
            if (!play_game()) {
                puts("输入结束或格式错误。");
                return 0;
            }
        } else {
            puts("无效菜单项。");
        }
    }
}
```

编译：`gcc -std=c17 -Wall -Wextra main.c game.c -o minesweeper`。Visual Studio 中将两个 `.c` 和头文件加入同一工程。本例输入按可由 `int` 表示的整数设计，非数字时退出；面向任意文本输入的版本可复用第 5 篇的逐行解析方法。

## 4. 关键逻辑为何这样写

**布雷去重**：只有把原本为 `'0'` 的格子变成 `'1'` 才增加 `placed`，否则重复随机坐标会导致实际雷数不足。

**字符与数字转换**：邻雷数返回整数 0..8，加上 `'0'` 后变成数字字符。字符数组不是整型数组，直接放整数 3 再用 `%c` 输出并不是字符 `'3'`。

**胜利计数**：只统计首次翻开的非雷格。检查重复坐标应放在 `++opened` 前，否则反复点一格也能错误获胜。

**边界缓冲**：位置 (1,1) 的邻居下标可到 0，位置 (9,9) 可到 10，都落在 11×11 数组内。边框不能布雷，也不应作为玩家可选坐标。

## 5. 按步骤调试

1. 不布雷，检查全空棋盘任意格的邻雷数是否为 0。
2. 手工固定一颗雷，检查它相邻的八格和远处格子的数值。
3. 随机布雷后，临时打印 `mine`，确认恰好 10 个 `'1'`。
4. 测试 (1,1)、(1,9)、(9,1)、(9,9)，以及 0、10 等非法坐标。
5. 重复点击同一个安全格，观察 `opened` 是否保持不变。
6. 设小棋盘和少量固定雷，分别验证踩雷和翻开全部安全格两条路径。

## 6. 扩展方向

自动展开零邻雷区域可使用递归或队列，必须先标记已访问再继续扩展，防止相邻格互相调用。首次安全可在第一次点击后再布雷并避开该格；旗帜需要增加展示状态，不能破坏 `mine`。增加难度时，常量、输入范围、胜利目标和布雷上限必须同步调整。
