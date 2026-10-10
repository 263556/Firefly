---
title: 我的第一篇文章
published: 2026-10-10
tags: ["分享"]
categories: 学习
cover: "" # 可以填入图片路径或URL
description: 扫雷游戏的分享（C语言）
draft: false
pinned: false
---
**game.h
#pragma once

#include<stdio.h>
#include<stdlib.h>
#include<time.h>

#define ROW 9
#define COL 9

#define ROWS ROW+2
#define COLS COL+2

#define EASY_COUNT 10
//函数的声明。初始化棋盘
void InitBoard(char board[ROWS][COLS], int r, int c,char set);

//打印棋盘
void DisplayBoard(char board[ROWS][COLS], int r, int c);

//布置雷
void SetMine(char mine[ROWS][COLS], int r, int c);

//排查雷
void FindMine(char mine[ROWS][COLS],char show[ROWS][COLS], int r, int c);

**game.c
#define _CRT_SECURE_NO_WARNINGS

#include"game.h"

//函数的定义

void InitBoard(char board[ROWS][COLS], int r, int c, char set)
{
	int i = 0;
	for (i = 0;i < r;i++)
	{
		int j = 0;
		for (j = 0;j < c;j++)
		{
			board[i][j] = set;
		}
	}
}

void DisplayBoard(char board[ROWS][COLS], int r, int c)
{
	int i = 0;
	int j = 0;
	//打印列数
	for (j = 0;j <= c;j++)
	{
		printf("%d ", j);
	}
	printf("\n");
	for (i = 1;i <= r;i++)
	{
		//打印行数
		printf("%d ", i);
		for (j = 1;j <= c;j++)
		{
			printf("%c ", board[i][j]);
		}
		printf("\n");
	}
	printf("\n");
}

void SetMine(char board[ROWS][COLS], int r, int c)
{
	int count = EASY_COUNT;
	while (count)//>=EASY_COUNT
	{
		int x = rand() % r + 1;
		int y = rand() % c + 1;
		if (board[x][y] != '1')
		{
			board[x][y] = '1';
			count--;
		}
	}
}

//这个函数仅仅是为了支持FindMine函数的实现而存在的，所以不需要将其放入game.h中，其他不i走并不需要
//这个函数变成一内部函数
static size_t GetMineCount(char mine[ROWS][COLS],int x, int y)
{
	//周围八个坐标相加的值，减去8*'0'，就能得出周围类的个数
	return mine[x - 1][y] + mine[x - 1][y - 1] + mine[x][y - 1] + mine[x + 1][y] + mine[x + 1][y] + mine[x + 1][y + 1] +
		mine[x][y + 1] + mine[x - 1][y + 1] - 8*'0';
}
//计算周围雷的个数的宁一个方法
//static size_t GetMineCount(char mine[ROWS][COLS], int x, int y)
//{
//	int i = 0;
//	int j = 0;
//	for (i = -1;i <= 1;i++)
//	{
//		for (j = -1;j <= 1;j++)
//		{
//			count +=(mine[x+i][y+j]-'0');
//		}
//	}
// return count;
//}


void FindMine(char mine[ROWS][COLS], char show[ROWS][COLS], int r, int c)
{
	int x = 0;
	int y = 0;
	int win = 0;
	while (win < r*c- EASY_COUNT)
	{
		printf("请输入要排查的坐标：");
		scanf("%d%d", &x, &y);
		//判断输入坐标的合法性
		if (x >= 1 && x <= r && y >= 1 && y <= c)
		{
			//判断该位置是否被排查过
			if (show[x][y] == '*')
			{
				//该位置是否是雷
				if (mine[x][y] == '0')//不是雷
				{
					size_t count = GetMineCount(mine, x, y);//统计mine数组中x，y周围雷的个数
					show[x][y] = (char)count + '0';
					DisplayBoard(show, ROW, COL);
					win++;
				}
				else
				{
					printf("很遗憾，你已经被炸死了\n");
					DisplayBoard(mine, ROW, COL);
					break;
				}
			}
			else
			{
				printf("输入坐标已经被排查过，请重新输入坐标");
			}
		}
		else
		{
			printf("输入坐标违法，请重新输入\n");
		}
	}
	//1.被炸死
	//2.排完雷了
	if (win == r * c - EASY_COUNT)
	{
		printf("恭喜你排雷成功\n");                                                                      
		DisplayBoard(mine, ROW, COL);
	}
}

**Minesweeper.c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include"game.h"

void menu()//菜单函数的编写
{
	printf("------------------------\n");
	printf("-----  1.play  ---------\n");
	printf("-----  0.exit  ---------\n");
	printf("------------------------\n");
}

void game()//扫雷游戏函数的编写大纲，细节编写放于game.c中
{
	//需要存储数据的数组
	char mine[ROWS][COLS];//布置好的雷的信息
	char show[ROWS][COLS];//排查出的雷的信息
	//写一个函数专门来初始化游戏区
	InitBoard(mine, ROWS, COLS,'0');//'0'
	InitBoard(show, ROWS, COLS,'*');//'*'
    //打印棋盘（游戏区）
	//DisplayBoard(mine, ROW, COL);
	DisplayBoard(show, ROW, COL);
	//布置雷
	SetMine(mine, ROW, COL);
	//DisplayBoard(mine, ROW, COL);
	//排查雷
	FindMine(mine, show, ROW, COL);
}
int main()
{
	int input = 0;//定义input来选择0，1
	srand((unsigned int)time(NULL));//随机数种子
	do
	{
		menu();//打印菜单
		printf("请选择：\n");
		scanf("%d", &input);//玩家输入选择
		switch (input)
		{
		case 1:
			game();//扫雷游戏
			break;
		case 0:
			printf("退出游戏，下次再来\n");
			break;
		default:
			printf("选择错误，请重新选择\n");
			break;
		}
		
	} while (input);//只有选择才不打印菜单
	return 0;
}

**这代码分3个部分构成。一个头文件，两个源文件