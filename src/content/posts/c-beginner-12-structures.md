---
title: C语言初阶 12：用结构体描述复杂对象
published: 2026-11-14
description: 用学生记录理解结构体声明、变量初始化、整体赋值、点与箭头、值传参与指针传参，并完成成绩统计案例。
tags: [C语言, 初阶, 结构体]
category: C语言
series: C语言初阶笔记
seriesOrder: 12
draft: false
---



## 1. 引入：一个学生不只是一个分数

学生包含学号、姓名、成绩。把它们分别存在独立数组中，一旦排序或删除容易失去对应关系。结构体把相关但不同类型的成员组合成一个对象。

```c
struct Student {
    int id;
    char name[32];
    double score;
};
```

这里声明了结构体类型，没有创建学生对象。结束的分号不能漏。C 中使用该类型通常写 `struct Student`，只写 `Student` 需要另外用 `typedef` 定义别名。

## 2. 创建变量与初始化

```c
struct Student first = {1001, "Lin", 92.5};
struct Student second = {0};
struct Student third = {
    .id = 1003,
    .name = "Chen",
    .score = 87.0
};
```

第一种按成员声明顺序初始化，第三种是 C99 起支持的指定成员初始化，可读性更好。`{0}` 将成员初始化为相应零值，字符数组元素也为 0。`struct Student student;` 若为自动局部对象且没有初始化，其成员不能假设为零。

也可在类型声明后直接创建变量，但把类型与对象分开通常更便于复用。匿名结构体适合少量局部情形，不便于跨函数共享类型，不宜作为初学工程的主要接口。

## 3. 成员访问：点号与箭头

```c
first.score = 95.0;
printf("%d %s %.1f\n", first.id, first.name, first.score);

struct Student *p = &first;
p->score = 96.0;
printf("%.1f\n", p->score);
```

结构体对象用 `.`，指向结构体的指针用 `->`。`p->score` 等价于 `(*p).score`；不能写 `*p.score`，因为成员访问的结合方式会改变表达式含义。

需要多层访问时，按每一层实际类型选择操作符。嵌套结构体的成员访问并没有新规则，例如 `student.address.city`。

## 4. 字符数组成员不能事后整体赋值

```c
// 错误：first.name = "Wang";
```

数组初始化和赋值是两回事。创建时可以用字符串初始化数组，创建后不能用 `=` 替换整个数组。可以在确认容量足够后复制内容。

```c
#include <string.h>

const char new_name[] = "Wang";
memcpy(first.name, new_name, sizeof new_name);
```

这里 `sizeof new_name` 包含终止符，且显然不超过 32。实际处理任意外部姓名时，应先检查字节长度或使用受控格式化，不能把这个固定短字符串例子推广为“不检查也安全”。UTF-8 中文姓名同样要按字节容量考虑。

## 5. 结构体可整体赋值，但不等于深拷贝

```c
struct Student backup = first;
backup.score = 60.0;
```

同一结构体类型的对象可以整体复制，`backup.name` 的数组成员也被复制，改 `backup.score` 不影响 `first.score`。这与数组本身不能整体赋值不同。

如果成员含指针，复制的是指针值，两个对象可能仍指向同一份外部数据：

```c
struct View {
    int *data;
};
int number = 10;
struct View a = {&number};
struct View b = a;
*b.data = 20; // number 也变成 20
```

所以整体赋值不自动复制指针指向的内容。数据由谁拥有、何时有效，仍要单独设计。

结构体不能用 `==` 逐成员比较；也不要用 `memcmp` 代替逻辑相等，因为填充字节和浮点表示等细节可能干扰比较。按需要比较各个成员。

## 6. 结构体传参：复制对象还是传地址

```c
void set_score_wrong(struct Student student, double score)
{
    student.score = score; // 只修改副本
}

void set_score(struct Student *student, double score)
{
    student->score = score; // 修改调用者的对象
}
```

调用分别为 `set_score_wrong(first, 80.0)` 和 `set_score(&first, 80.0)`。结构体可以按值传参和返回，语义很清楚；较大对象或需要修改原对象时适合传指针。只读函数常写 `const struct Student *`，表达不会通过该参数修改对象。

传指针并不自动更安全，仍需保证地址非空、对象有效。选择接口时优先看语义，再考虑复制成本。

## 7. 完整案例：学生成绩列表与最高分

```c
#include <stdio.h>
#include <stddef.h>

struct Student {
    int id;
    char name[32];
    double score;
};

static void print_student(const struct Student *student)
{
    printf("%d %-8s %.1f\n", student->id, student->name, student->score);
}

/* 空列表返回 NULL；非空时返回原数组中的最高分对象地址。 */
static const struct Student *find_top(const struct Student data[], size_t count)
{
    if (count == 0) {
        return NULL;
    }
    const struct Student *top = &data[0];
    for (size_t i = 1; i < count; ++i) {
        if (data[i].score > top->score) {
            top = &data[i];
        }
    }
    return top;
}

int main(void)
{
    struct Student students[] = {
        {1001, "Lin", 92.5},
        {1002, "Wang", 86.0},
        {1003, "Chen", 95.0}
    };
    size_t count = sizeof students / sizeof students[0];
    for (size_t i = 0; i < count; ++i) {
        print_student(&students[i]);
    }
    const struct Student *top = find_top(students, count);
    if (top != NULL) {
        puts("最高分：");
        print_student(top);
    }
    return 0;
}
```

最高分相同时保留先出现的学生。函数返回的是原数组元素的地址，调用者使用它时数组必须仍然存活；这里数组在 `main` 中，使用期间有效。不要改成返回函数内部局部 `struct Student` 对象的地址。

## 8. 大小与内存布局的初步认识

结构体的 `sizeof` 不一定等于各成员大小相加，编译器可能插入填充满足对齐要求。可以用 `sizeof(struct Student)` 观察总大小，使用 `<stddef.h>` 的 `offsetof` 查询成员偏移，不要靠猜测字节偏移读写成员。

初阶先掌握通过成员名访问；布局、对齐以及文件序列化留待进阶。直接把结构体内存写入文件，也不能自动成为跨平台稳定的数据格式。

## 9. 自测与综合练习

1. 解释 `student.score`、`p->score`、`(*p).score` 各自要求的类型。
2. 给学生添加班级字段，更新初始化和输出函数，检查接口是否集中修改。
3. 编写只读的平均分函数，明确空列表的结果或失败返回方式。
4. 给列表按成绩排序，交换完整结构体，避免只交换分数而使姓名与成绩错位。

到这里，数组负责组织多个同类对象，结构体描述一个对象的多个字段，函数组织行为，指针描述访问关系。这四者结合，就能开始设计通讯录、学生管理和棋盘游戏等小程序。
