---
title: C语言进阶 24：文件读写、错误处理与数据持久化
published: 2026-11-14
description: 系统梳理流与打开模式、字符和块读写、EOF、定位、更新流和缓冲，并实现二进制复制与通讯录文本保存读取。
tags: [C语言, 进阶, 文件操作, 持久化]
category: C语言
series: C语言学习笔记
seriesOrder: 24
draft: false
---

示例文件名均为练习目录中的固定名称，运行前确认不会覆盖需要保留的同名文件。

## 1. 引入：让通讯录在退出后仍然存在

内存里的联系人在程序结束后消失。持久化需要定义保存格式，把数据写入文件，下一次启动再解析回来。它不仅是“加两个 fread/fwrite”，还涉及打开失败、格式损坏、部分读写和关闭失败。

文本文件通常保存编码后的字符，便于查看；二进制文件按约定保存字节序列。文本也不一定是 ASCII，UTF-8 同样是文本编码。文件扩展名不决定内容，`.txt` 中也能写二进制。

整数 10000 写成文本通常是五个数字字符；直接写本机 int 是 `sizeof(int)` 字节且包含机器表示，不保证永远为 4，也不保证换一台机器还能正确解释。

## 2. 流、FILE 与标准流

`FILE *` 标识一个由标准 I/O 库管理的流，含位置、缓冲、错误等内部状态。不要根据某个版本库的 FILE 结构定义直接修改内部字段，也不要把 FILE 指针等同于“整个文件内容的内存地址”。

宿主环境启动时提供 stdin、stdout、stderr，常见对应键盘、屏幕和错误输出，也可以被重定向。标准流和文件流共享许多 I/O 接口。

## 3. 打开模式决定会不会截断原文件

| 模式 | 能读 | 能写 | 已有内容 | 文件不存在 |
| --- | --- | --- | --- | --- |
| `r` | 是 | 否 | 保留 | 失败 |
| `w` | 否 | 是 | **截断** | 创建 |
| `a` | 否 | 是 | 保留，写入追加 | 创建 |
| `r+` | 是 | 是 | 保留 | 失败 |
| `w+` | 是 | 是 | **截断** | 创建 |
| `a+` | 是 | 是 | 保留，写入强制追加 | 创建 |

加 `b` 选择二进制流，如 `rb`、`wb`、`rb+`。在 Windows 等平台，文本模式可能转换换行和某些控制字符；处理任意字节时使用二进制模式。

追加模式的写操作进入文件末尾，不能靠一次 fseek 把它变成任意位置覆盖。需要读取追加流时显式定位，不依赖实现的初始读取位置差异。

```c
FILE *file = fopen("notes.txt", "r");
if (file == NULL) {
    // 打开失败，不能继续读取，也不能 fclose(NULL)。
}
```

相对路径相对于进程工作目录，不一定是源文件所在目录。Windows C 字符串中的反斜杠需转义，例如 `"D:\\data\\notes.txt"`。

## 4. 关闭也是可能失败的操作

`fclose(file)` 成功返回 0，失败返回 EOF。输出可能先进入缓冲，最终刷新时才发现失败，因此不能只检查 fwrite/fprintf 而忽略 fclose。

调用 fclose 后，无论返回是否成功，都不能继续使用这个 FILE 指针或再次关闭它。置 NULL 只避免本变量误用，不修复其他别名。错误路径也要关闭所有已打开的流。

## 5. 顺序 I/O 接口对照

| 数据形式 | 读取 | 写入 | 返回结果重点 |
| --- | --- | --- | --- |
| 一个字节字符 | `fgetc` | `fputc` | 用 int 保存字符或 EOF |
| 一行/字符串 | `fgets` | `fputs` | fgets 返回指针或 NULL；fputs 不自动换行 |
| 格式化数据 | `fscanf` | `fprintf` | 成功赋值数；输出字符数或负值 |
| 指定数据块 | `fread` | `fwrite` | 成功读取/写入的**元素数** |

`fread(buffer, size, count, file)` 最多写 `size * count` 字节，buffer 必须足够大；不自动补 `\0`。把 size 设为 1 时，返回值才直接等于完整读取的字节数。

fputs 只写到字符串终止符前，不把终止符写入流；puts 会额外换行，但 fputs 不会。fprintf 的格式和参数类型仍要匹配，文件输出不豁免类型错误。

## 6. EOF 不是提前预告

错误写法是 `while (!feof(file))` 然后无条件使用读取结果。文件结束标志通常在尝试读取越过末尾后才设置，不会在你刚读到最后一个字符时自动预测下次失败。

正确方式让读取返回值控制循环，循环后再区分错误与正常结束：

```c
int ch;
while ((ch = fgetc(file)) != EOF) {
    // 处理本次成功读到的字符
}
if (ferror(file)) {
    // 读取错误
} else if (feof(file)) {
    // 正常到达末尾
}
```

ch 使用 int，不能先存到 char 再比较 EOF。`clearerr` 清除流错误与结束指示器；它不自动修复底层错误或回到文件开头。

## 7. 完整案例：按块复制二进制文件

将 input.bin 复制到 output.bin，覆盖目标内容。本例同时处理短读、短写和关闭失败。

```c
#include <stdio.h>

int main(void)
{
    FILE *input = fopen("input.bin", "rb");
    if (input == NULL) {
        perror("open input");
        return 1;
    }
    FILE *output = fopen("output.bin", "wb");
    if (output == NULL) {
        perror("open output");
        fclose(input);
        return 1;
    }

    unsigned char buffer[4096];
    size_t count;
    int success = 1;
    while ((count = fread(buffer, 1, sizeof buffer, input)) != 0) {
        if (fwrite(buffer, 1, count, output) != count) {
            fputs("write failed\n", stderr);
            success = 0;
            break;
        }
    }
    if (ferror(input)) {
        fputs("read failed\n", stderr);
        success = 0;
    }
    if (fclose(input) != 0) {
        success = 0;
    }
    if (fclose(output) != 0) {
        success = 0;
    }
    return success ? 0 : 1;
}
```

只有成功读取的 count 字节会写出，不把缓冲区上次留下的尾部也复制。输入输出应为不同文件，真实命令行工具还要处理路径指向同一文件的情况。失败时 output.bin 可能只写了一部分，不能当作有效备份。

## 8. fgets 与格式化解析

fgets 最多读取容量减一字节，保留读到的换行并补终止符。过长的一行会拆成多次读取，必须自行决定拼接还是拒绝；不要把每段都当成完整记录。

```c
char line[128];
if (fgets(line, sizeof line, file) != NULL) {
    line[strcspn(line, "\r\n")] = '\0';
}
```

需要 `<string.h>`。先确认 fgets 成功，再处理缓冲区。fscanf 也必须检查赋值数，格式不匹配可能返回 0 且不推进输入，盲目重试会死循环。

字符串目标指定宽度，如 `%31s` 需要至少 32 字节；处理任意数值文本时，`fgets + strtol` 更便于报告范围错误和尾随非法字符。文件格式不可信时，还应限制行长、记录数和字段长度。

## 9. 定位、更新流与缓冲

- `ftell` 返回位置表示，失败为 -1L；文本流的位置值不能普遍当作字节数做算术。
- `fseek(file, offset, origin)` 成功为 0，失败非零；origin 为 SEEK_SET、SEEK_CUR、SEEK_END。
- 文本流可移植的定位方式受到限制：offset 为 0，或使用之前 ftell 返回的位置值并配 SEEK_SET。不要随意按字符数加减文本位置。
- `rewind` 回开头并清除 EOF 与错误指示器，但不返回失败码；需要检查定位失败时可使用 fseek。
- 成功 fseek 会清除 EOF 指示器，但不等于清除已有错误标志。

更新流读写切换规则：写后读要有 fflush 或定位操作；读后写要有定位操作，除非刚刚的读取遇到 EOF。不是“写后必须同时 fflush 再 fseek”才能合法，定位操作本身也能满足相应切换要求。

`fflush` 将输出缓冲交给底层，不保证数据已物理持久写入磁盘。不要用非标准的 `fflush(stdin)` 清理输入。行缓冲、全缓冲和无缓冲影响何时传递数据，但不能替代返回值检查和关闭操作。

## 10. 完整案例：通讯录文本保存与读取

定义简单格式：每行 `姓名<TAB>电话`，姓名和电话不允许空、制表符、换行，总计最多 8 项。load 先读入临时 Book，全部成功才覆盖调用者，避免损坏文件产生半个新列表。

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

static int valid_field(const char *text, size_t capacity)
{
    size_t length = strlen(text);
    return length > 0 && length < capacity && strpbrk(text, "\t\r\n") == NULL;
}

int save_book(const char *path, const struct Book *book)
{
    if (path == NULL || book == NULL || book->count > 8) {
        return 0;
    }
    for (size_t i = 0; i < book->count; ++i) {
        if (!valid_field(book->items[i].name, sizeof book->items[i].name) ||
            !valid_field(book->items[i].phone, sizeof book->items[i].phone)) {
            return 0;
        }
    }
    FILE *file = fopen(path, "w");
    if (file == NULL) {
        return 0;
    }
    int success = 1;
    for (size_t i = 0; i < book->count; ++i) {
        if (fprintf(file, "%s\t%s\n", book->items[i].name, book->items[i].phone) < 0) {
            success = 0;
            break;
        }
    }
    if (fclose(file) != 0) {
        success = 0;
    }
    return success;
}

int load_book(const char *path, struct Book *book)
{
    if (path == NULL || book == NULL) {
        return 0;
    }
    FILE *file = fopen(path, "r");
    if (file == NULL) {
        return 0;
    }
    struct Book temporary = {.count = 0};
    char line[128];
    int success = 1;
    while (fgets(line, sizeof line, file) != NULL) {
        if (strchr(line, '\n') == NULL && !feof(file)) {
            success = 0;
            break;
        }
        line[strcspn(line, "\r\n")] = '\0';
        char *separator = strchr(line, '\t');
        if (temporary.count == 8 || separator == NULL) {
            success = 0;
            break;
        }
        *separator = '\0';
        const char *phone = separator + 1;
        struct Contact *item = &temporary.items[temporary.count];
        if (!valid_field(line, sizeof item->name) || !valid_field(phone, sizeof item->phone)) {
            success = 0;
            break;
        }
        memcpy(item->name, line, strlen(line) + 1);
        memcpy(item->phone, phone, strlen(phone) + 1);
        ++temporary.count;
    }
    if (ferror(file)) {
        success = 0;
    }
    if (fclose(file) != 0) {
        success = 0;
    }
    if (success) {
        *book = temporary;
    }
    return success;
}

int main(void)
{
    struct Book original = {{{"Lin", "13800000000"}, {"Chen", "13900000000"}}, 2};
    struct Book restored = {.count = 0};
    if (!save_book("contacts.tsv", &original) || !load_book("contacts.tsv", &restored)) {
        fputs("保存或读取失败\n", stderr);
        return 1;
    }
    for (size_t i = 0; i < restored.count; ++i) {
        printf("%s %s\n", restored.items[i].name, restored.items[i].phone);
    }
    return 0;
}
```

接口假设内存中的 Book 已满足各字符串成员正确终止的内部约定；解析文件时再验证字段。示例面向没有嵌入 NUL 的文本格式，任意恶意二进制输入还需更严格的逐字节校验。

save 使用 w，写入前会截断旧文件，因此失败可能损失旧版本。需要可靠更新时，可先写同目录临时文件，确认写入和关闭成功后，再按目标平台的替换语义提交；跨平台原子性和崩溃持久性不能仅凭一次 rename 或 fflush 保证。

## 11. 格式设计与练习

不要把含指针的结构体直接写入文件期待下次恢复地址。稳定格式应写明版本、字段范围、字符编码、记录边界及字节序；文本便于调试，二进制可以更紧凑，但都需要验证。

练习：测试空文件、最后一行无换行、超长字段、缺制表符和超过 8 条记录；确认 load 失败时旧 Book 不变。给二进制复制例子准备含 0 字节、0xFF 和大于 4096 字节的输入，再逐字节比较输出。
