---
title: C语言进阶 18：字符与字符串函数的完整使用规则
published: 2026-11-14
description: 系统梳理字符分类、字符串复制拼接比较、格式化、查找分割与错误信息，并实现安全复制和命令行待办事项。
tags: [C语言, 进阶, 字符串, 标准库]
category: C语言
series: C语言学习笔记
seriesOrder: 18
draft: false
---


## 1. 引入：处理一条命令需要哪些操作

用户输入 `add learn C`，程序要识别前缀、保存任务、检查容量并输出列表。看似简单的字符串功能，实际涉及长度、终止符、复制、比较和解析等不同契约。

首先牢记：字符串在第一个 `\0` 结束，数组容量包括终止符的位置。库函数通常不知道真实容量，传入地址并不会自动得到越界保护。

## 2. 字符分类与大小写转换

`<ctype.h>` 的分类函数，条件满足返回非零，否则返回 0；不要要求结果必须等于 1。

| 函数 | 判断 |
| --- | --- |
| `isdigit`、`isxdigit` | 十进制数字、十六进制数字 |
| `islower`、`isupper` | 小写、大写字母 |
| `isalpha`、`isalnum` | 字母、字母或数字 |
| `isspace`、`iscntrl` | 空白、控制字符 |
| `ispunct` | 标点 |
| `isprint`、`isgraph` | 可打印、可打印且非空格 |

`tolower` 转小写、`toupper` 转大写，无法转换的字符保持原值。不要写“统一加减 32”，它依赖字符编码且不能替代库函数。

```c
#include <stdio.h>
#include <ctype.h>

int main(void)
{
    char text[] = "Hello C 2026!";
    for (size_t i = 0; text[i] != '\0'; ++i) {
        text[i] = (char)toupper((unsigned char)text[i]);
    }
    puts(text);
    return 0;
}
```

参数必须是 EOF，或可表示为 `unsigned char` 的值。普通 char 可能为负，所以处理字符串字节时先转成 `unsigned char`。从 `getchar` 获得的值应保存在 int 中，先排除 EOF，再处理；不要把 EOF 先截断进 char。

## 3. strlen：无符号长度不是差值类型

`size_t strlen(const char *s)` 返回第一个 `\0` 前的字节数。没有终止符的数组不能传入。输出用 `%zu`，不能把它与 `sizeof` 混为一谈。

```c
// 正确比较长度：
if (strlen(a) > strlen(b)) {
    // a 较长
}
```

不要写 `strlen(a) - strlen(b) > 0`：较短字符串减较长字符串会无符号下溢。递归模拟为 `*s ? 1 + length(s + 1) : 0` 虽然简洁，却增加 O(n) 调用深度，循环通常更合适。

## 4. 复制和追加：n 的含义并不统一

以下函数均在 `<string.h>`：

| 函数 | 做什么 | 关键约束 |
| --- | --- | --- |
| `strcpy(dst, src)` | 复制到终止符，包括终止符 | dst 至少容纳 `strlen(src)+1` |
| `strncpy(dst, src, n)` | 最多取 n 字节，不足时补零到 n | 源长不小于 n 时不追加终止符 |
| `strcat(dst, src)` | 从 dst 的终止符处追加 | 两者是字符串，总容量足够 |
| `strncat(dst, src, n)` | 最多追加 n 个源字符，再补终止符 | n 是追加上限，不是 dst 总容量 |

这些复制/追加操作要求相应源目标区域不重叠。`strncpy` 不等于“自动安全的 strcpy”，还可能写满 n 个字节；`strncat` 的终止符另占一个位置。

例如容量 cap、已有长度 used 的缓冲区，最多还可追加 `cap - used - 1` 个字符，前提是已证明 `used < cap`。不验证终止符就先 `strlen(dst)`，也可能在计算剩余容量之前已经越界。

### 模拟复制并明确检查容量

下面自定义接口不截断：空间不足返回 0，目标不变；成功复制并返回 1。它不声称与 `strcpy` 有相同接口。

```c
#include <stdio.h>
#include <stddef.h>
#include <string.h>

int copy_text(char *destination, size_t capacity, const char *source)
{
    if (destination == NULL || source == NULL || capacity == 0) {
        return 0;
    }
    size_t length = strlen(source);
    if (length >= capacity) {
        return 0;
    }
    for (size_t i = 0; i <= length; ++i) {
        destination[i] = source[i];
    }
    return 1;
}

int main(void)
{
    char name[8] = "old";
    if (!copy_text(name, sizeof name, "Lin")) {
        return 1;
    }
    puts(name);
    printf("long copy accepted: %d\n", copy_text(name, sizeof name, "too-long-name"));
    puts(name); // 失败后仍为 Lin
    return 0;
}
```

仍要求 source 为有效字符串、source 与 destination 不重叠，capacity 不能谎报真实容量。需要移动重叠字节应使用 `memmove`。

## 5. 比较与子串查找

`strcmp(a, b)` 逐字节按 unsigned char 解释比较，返回值的**符号**表示字典序，不保证只返回 -1、0、1。`strncmp(a, b, n)` 最多比较 n 字节，遇到终止符也停止。

判断完整命令用 `strcmp(command, "list") == 0`；检查前缀用 `strncmp(command, "add ", 4) == 0`。前缀相同不代表完整字符串相同，中文词语的语言学排序也不能直接由字节比较替代。

`strstr(text, pattern)` 返回第一次匹配位置，找不到返回 NULL；空 pattern 匹配 text 起点。下面暴力查找保留只读属性：

```c
const char *find_text(const char *text, const char *pattern)
{
    if (*pattern == '\0') {
        return text;
    }
    for (; *text != '\0'; ++text) {
        const char *a = text;
        const char *b = pattern;
        while (*a != '\0' && *b != '\0' && *a == *b) {
            ++a;
            ++b;
        }
        if (*b == '\0') {
            return text;
        }
    }
    return NULL;
}
```

调用者提供两个有效字符串，需包含 `<stddef.h>` 取得 NULL。最坏时间 O(nm)，更复杂需求可学习 KMP 等算法，但不要先牺牲接口正确性。

## 6. sprintf、snprintf 和 sscanf

这些接口位于 `<stdio.h>`。`sprintf` 把格式化结果写进缓冲区，但没有容量参数；`snprintf` 额外接收允许写入的总字节数。容量必须真实，否则函数不知道你报大了。

```c
#include <stdio.h>

int main(void)
{
    char line[32];
    int result = snprintf(line, sizeof line, "%s:%d", "score", 95);
    if (result < 0 || (size_t)result >= sizeof line) {
        fputs("格式化失败或内容被截断\n", stderr);
        return 1;
    }
    puts(line);
    return 0;
}
```

返回值是完整结果本应需要的字符数，不含终止符；非负且小于容量才表示完整写入。不要用 `snprintf(buf, cap, "%s!", buf)` 原地自拼接，源目标重叠不合法。需追加时先得到合法已有长度，再向 `buf + used` 写剩余空间。

`sscanf` 从内存字符串解析，返回成功赋值项目数，可能为 0，不是所有失败都返回 EOF。字符串目标指定宽度：`sscanf(line, "%19s %d", name, &age)` 中 name 至少 20 字节。宽度不会解决数值溢出和所有格式验证；不可信整数用 `strtol` 配合范围检查更稳妥。

## 7. strtok：会修改原串且保存内部状态

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char text[] = "C,,Astro;Firefly";
    for (char *token = strtok(text, ",;"); token != NULL;
         token = strtok(NULL, ",;")) {
        puts(token);
    }
    return 0;
}
```

输出 C、Astro、Firefly。分隔符参数表示一个字符集合，不是完整分隔子串；连续分隔符被跳过，不返回空字段。它会把分隔位置改成 `\0`，因此不能传字符串字面量。嵌套分割会干扰内部状态，严格 CSV、需要空字段或并发解析时应采用更适合的解析方式。

## 8. strerror、errno 与 perror

`strerror(error_code)` 返回错误说明；`perror("prefix")` 根据当前 errno 向 stderr 输出提示。应先判断被调用函数是否失败，再在该函数契约规定设置 errno 时读取它。成功调用不一定清零 errno，不能只靠 errno 非零判断失败。

错误信息、编号和语言可能因平台而异；`errno` 应通过 `<errno.h>` 使用，不要自己声明一个 `extern int errno`。保存错误码后再做其他调用，能避免后续操作覆盖真正原因。

## 9. 完整实践：固定容量的命令行待办事项

支持 `add 文本`、`list`、`done 编号`、`quit`。每项最多 63 字节，总计 8 项；过长命令明确拒绝，不把截断片段当成完整命令执行。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <errno.h>

struct Task {
    char text[64];
    int done;
};

int main(void)
{
    struct Task tasks[8] = {{"", 0}};
    size_t count = 0;
    char line[128];
    while (fgets(line, sizeof line, stdin) != NULL) {
        if (strchr(line, '\n') == NULL && !feof(stdin)) {
            int ch;
            while ((ch = getchar()) != '\n' && ch != EOF) {
            }
            puts("命令过长。");
            continue;
        }
        line[strcspn(line, "\r\n")] = '\0';
        if (strcmp(line, "quit") == 0) {
            break;
        }
        if (strcmp(line, "list") == 0) {
            for (size_t i = 0; i < count; ++i) {
                printf("%zu [%c] %s\n", i + 1, tasks[i].done ? 'x' : ' ', tasks[i].text);
            }
        } else if (strncmp(line, "add ", 4) == 0) {
            const char *text = line + 4;
            size_t length = strlen(text);
            if (count == 8 || length == 0 || length >= sizeof tasks[0].text) {
                puts("任务为空、过长或列表已满。");
                continue;
            }
            memcpy(tasks[count].text, text, length + 1);
            tasks[count].done = 0;
            ++count;
        } else if (strncmp(line, "done ", 5) == 0) {
            char *end;
            errno = 0;
            long index = strtol(line + 5, &end, 10);
            if (end == line + 5 || errno == ERANGE) {
                puts("编号格式错误。");
                continue;
            }
            while (isspace((unsigned char)*end)) {
                ++end;
            }
            if (*end != '\0' || index < 1 || (unsigned long)index > count) {
                puts("编号无效。");
                continue;
            }
            tasks[(size_t)index - 1].done = 1;
        } else {
            puts("命令：add 文本 / list / done 编号 / quit");
        }
    }
    return ferror(stdin) ? 1 : 0;
}
```

测试 `add learn C`、`add review pointers`、`done 1`、`list`，第一项应显示 `[x]`。再测空任务、超长行、非法编号和第九个任务。内容长度按字节限制，不能把 63 字节直接解释成 63 个汉字。

## 10. 使用标准

选函数时先问“我是否知道容量、源是否终止、区域是否重叠、截断能否接受”。带 n 的名字不自动代表安全：n 对 `strncpy`、`strncat`、`snprintf` 分别有不同含义，调用处必须按各自契约计算。
