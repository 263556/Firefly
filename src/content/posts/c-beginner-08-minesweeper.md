---
title: C语言初阶 08：从零组织命令行扫雷项目
published: 2026-11-14
description: 依照原课件逐节整理，完整保留代码示例、练习、对照实现与图表，精简重复说明并附易错点。
tags: [C语言, 初阶, 扫雷, 项目实践]
category: C语言
series: C语言学习笔记
seriesOrder: 8
draft: false
updated: 2026-11-16
---

> 本篇按《（新 AI 版）鹏哥 C 语言课件》第 8 章逐节整理。保留原课件代码、对照写法、练习及图表，仅压缩重复说明。
> 代码块包含可运行实现、局部片段、输出、故意出错的示例与思考题；它们是独立示例，请结合上下文阅读，不能将整章拼成一个程序。

## 阅读前先核对

- 课件扫雷代码中的字符 `'0'`、`'1'` 与整数 0、1 不同。布雷需去重；翻开的安全格只计数一次；输入坐标先检查 1..ROW、1..COL，才能利用边框访问邻居。

## 本章目标

1. 掌握数组在程序设计中的使用

2. 掌握函数在程序设计中的使用

3. 掌握如何分析、设计和封装需要的函数

4. 锻炼对200行左右程序的规划和设计能力

5. 锻炼对一个稍微复杂的代码的分析和拆解能力

6. 锻炼使用AI来补充和扩展代码的能力

使用C语言实现一个经典的扫雷游戏，暂时实现一个控制台的版本，后期我们使用AI实现一个带界面的版本。

## 1. 扫雷游戏分析和设计

### 1.1 扫雷游戏的功能说明

- 使用控制台实现经典的扫雷游戏

- 游戏可以通过菜单实现"开始游戏"或者"退出游戏"

- 扫雷的棋盘是9\*9的格子

- 默认随机布置10个雷

- 可以排查雷

- 如果位置不是雷，就显示周围有几个雷

- 如果位置是雷，就被炸死，游戏结束

- 把除10个雷之外的所有非雷都找出来，排雷成功，游戏结束

游戏的界面：

![第8章第2页配图](/images/c-language-course/08-p02-fig01.png)

![第8章第2页配图](/images/c-language-course/08-p02-fig02.png)

![第8章第2页配图](/images/c-language-course/08-p02-fig03.png)

排雷界面 排雷失败界面初始界面

### 1.2 游戏的分析和设计

#### 1.2.1 数据结构的分析

扫雷的过程中，布置的雷和排查出的雷的信息都需要存储，所以我们需要一定的数据结构来存储这些信息。

因为我们需要在9\*9的棋盘上布置雷的信息和排查雷，我们首先想到的就是创建一个9\*9的数组来存放信息。

![第8章第2页配图](/images/c-language-course/08-p02-fig04.png)

空棋盘那如果这个位置布置雷，我们就存放1，没有布置雷就存放0.

![第8章第3页配图](/images/c-language-course/08-p03-fig01.png)

假设我们排查(2,5)这个坐标时，我们访问周围的一圈8个黄色位置，统计周围雷的个数是1假设我们排查(8,6)这个坐标时，我们访问周围的一圈8个黄色位置，统计周围雷的个数时，最下面的三个坐标就会越界，为了防止越界，我们在设计的时候，给数组扩大一圈，雷还是布置在中间的9\*9的坐标上，周围一圈不去布置雷就行，这样就解决了越界的问题。所以我们将存放数据的数组创建成11\*11是比较合适。

![第8章第3页配图](/images/c-language-course/08-p03-fig02.png)

![第8章第3页配图](/images/c-language-course/08-p03-fig03.png)

排雷的假设 周围加上一圈后的棋盘再继续分析，我们在棋盘上布置了雷，棋盘上雷的信息（1）和非雷的信息（0），假设我们排查了某一个位置后，这个坐标处不是雷，这个坐标的周围有1个雷，那我们需要将排查出的雷的数量信息记录存储，并打印出来，作为排雷的重要参考信息的。那这个雷的个数信息存放在哪里呢？如果存放在布置雷的数组中，这样雷的信息和雷的个数信息就可能或产生混淆和打印上的困难。

这里我们肯定有办法解决，比如：雷和非雷的信息不要使用数字，使用某些字符就行，这样就避免冲突了，但是这样做棋盘上有雷和非雷的信息，还有排查出的雷的个数信息，就比较混杂，不够方便。

这里我们采用另外一种方案，我们专门给一个棋盘（对应一个数组mine）存放布置好的雷的信息，再给另外一个棋盘（对应另外一个数组show）存放排查出的雷的信息。这样就互不干扰了，把雷布置到mine数组，在mine数组中排查雷，排查出的数据存放在show数组，并且打印show数组的信息给后期排查参考。

同时为了保持神秘，show数组开始时初始化为字符 '\*'，为了保持两个数组的类型一致，可以使用同一套函数处理，mine数组最开始也初始化为字符'0'，布置雷改成'1'。如下如：

![第8章第4页配图](/images/c-language-course/08-p04-fig01.png)

![第8章第4页配图](/images/c-language-course/08-p04-fig02.png)

mine数组布置雷后的状态 show输出初始化的状态对应的数组应该是：

**课件代码 08-001 · PDF 第 4 页**

```c
char mine[11][11] = {0};//用来存放布置好的雷的信息
char show[11][11] = {0};//用来存放排查出的雷的个数信息
```

#### 1.2.2 文件结构设计

之前学习了多文件的形式对函数的声明和定义，这里我们实践一下，我们设计三个文件：

**课件代码 08-002 · PDF 第 4 页**

```text
test.c //文件中写游戏的测试逻辑
game.c //文件中写游戏中函数的实现等
game.h //文件中写游戏需要的数据类型和函数声明等
```

## 2. 扫雷游戏的代码实现

game.h

**课件代码 08-003 · PDF 第 5 页**

```c
#pragma once

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define EASY_COUNT 10

#define ROW 9
#define COL 9

#define ROWS ROW+2
#define COLS COL+2

//初始化棋盘
void InitBoard(char board[ROWS][COLS], int rows, int cols, char set);

//打印棋盘
void DisplayBoard(char board[ROWS][COLS], int row, int col);

//布置雷
void SetMine(char board[ROWS][COLS], int row, int col);

//排查雷
void FindMine(char mine[ROWS][COLS], char show[ROWS][COLS], int row, int col);
```

game.c

**课件代码 08-004 · PDF 第 5,6,7 页**

```c
#include "game.h"

void InitBoard(char board[ROWS][COLS], int rows, int cols, char set)
{
        int i = 0;
        for (i = 0; i < rows; i++)
        {
                int j = 0;
                for (j = 0; j < cols; j++)
                {
                        board[i][j] = set;
                }
        }
}

void DisplayBoard(char board[ROWS][COLS], int row, int col)
{
        int  i = 0;
        printf("--------扫雷游戏-------\n");
        for (i = 0; i <= col; i++)
        {
                printf("%d ", i);
        }
        printf("\n");
        for (i = 1; i <= row; i++)
        {
                printf("%d ", i);
                int j = 0;
                for (j = 1; j <= col; j++)
                {
                        printf("%c ", board[i][j]);
                }
                printf("\n");
        }
}

void SetMine(char board[ROWS][COLS], int row, int col)
{
    //布置10个雷
    //生成随机的坐标，布置雷
    int count = EASY_COUNT;
    while (count)
    {
        int x = rand() % row + 1;
        int y = rand() % col + 1;

        if (board[x][y] == '0')
        {
            board[x][y] = '1';
            count--;
        }
    }
}

static int GetMineCount(char mine[ROWS][COLS], int x, int y)
{
    return (mine[x-1][y]+mine[x-1][y-1]+mine[x][y - 1]+mine[x+1][y-1]+mine[x+1] [y]+
            mine[x+1][y+1]+mine[x][y+1]+mine[x-1][y+1] - 8 * '0');
}


void FindMine(char mine[ROWS][COLS], char show[ROWS][COLS], int row, int col)
{
    int x = 0;
    int y = 0;
    int win = 0;
    while (win < row * col- EASY_COUNT)
    {
        printf("请输入要排查的坐标:>");
        scanf("%d %d", &x, &y);
        if (x >= 1 && x <= row && y >= 1 && y <= col)
        {
            if (mine[x][y] == '1')
            {
                printf("很遗憾，你被炸死了\n");
                DisplayBoard(mine, ROW, COL);
                break;
            }
            else
            {
                //该位置不是雷，就统计这个坐标周围有几个雷
                int count = GetMineCount(mine, x, y);
                show[x][y] = count + '0';
                DisplayBoard(show, ROW, COL);
                win++;
            }
        }
        else
        {
            printf("坐标非法，重新输入\n");
        }
    }
    if (win == row * col - EASY_COUNT)
    {
        printf("恭喜你，排雷成功\n");
        DisplayBoard(mine, ROW, COL);
    }
}
```

test.c

**课件代码 08-005 · PDF 第 8,9 页**

```c
#include "game.h"

void menu()
{
    printf("***********************\n");
    printf("*****   1. play   *****\n");
    printf("*****   0. exit   *****\n");
    printf("***********************\n");
}

void game()
{
    char mine[ROWS][COLS];//存放布置好的雷
    char show[ROWS][COLS];//存放排查出的雷的信息
    //初始化棋盘
    //1. mine数组最开始是全'0'
    //2. show数组最开始是全'*'
    InitBoard(mine, ROWS, COLS, '0');
    InitBoard(show, ROWS, COLS, '*');
    //打印棋盘
    //DisplayBoard(mine, ROW, COL);
    DisplayBoard(show, ROW, COL);
    //1. 布置雷
    SetMine(mine, ROW, COL);
    //DisplayBoard(mine, ROW, COL);
    //2. 排查雷
    FindMine(mine, show, ROW, COL);
}

int main()
{
    int input = 0;
    srand((unsigned int)time(NULL));
    do
    {
            menu();
            printf("请选择:>");
            scanf("%d", &input);
            switch (input)
            {
            case 1:
                    game();
                    break;
            case 0:
                    printf("退出游戏\n");
                    break;
            default:
                    printf("选择错误，重新选择\n");
                    break;
            }
    } while (input);

    return 0;
}
```

## 3. 扫雷游戏的扩展

- 是否可以选择游戏难度

- 简单 9\*9 棋盘，10个雷

- 中等 16\*16棋盘，40个雷

- 困难 30\*16棋盘，99个雷

- 如果排查位置不是雷，周围也没有雷，可以展开周围的一片（这个需要学习函数递归，等后期让AI

拓展）

- 是否可以标记雷

- 是否可以加上排雷的时间显示

📌 AI提示词：

1. 在尽量少改动代码的基础上，给游戏增加计时功能，每次排雷后都显示当前已经消耗了多

少时间

2. 在尽量少改动代码的基础上，避免第一次排雷炸死的功能

3. 在尽量少改动代码的基础上，增加标记雷的功能

4. 在尽量少改动代码的基础上，给游戏增加难度选择

- 简单 9\*9 棋盘，10个雷

- 中等 16\*16棋盘，40个雷

- 困难 30\*16棋盘，99个雷

AI拓展代码后，我们可以去阅读和学习代码~在线扫雷游戏：http://www.minesweeper.cn/

## 4. AI加餐课

8.BS和CS软件架构和主流技术栈

## 课件中的参考链接

- [参考资料 1](https://bitejiuyeke.feishu.cn/wiki/NakbwwmO2ionu1kU47ecjGRxnsx)
