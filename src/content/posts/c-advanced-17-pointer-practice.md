---
title: C语言进阶 17：指针的边界、const、断言与所有权
published: 2026-11-14
description: 从对象创建与销毁建立指针接口契约，区分 NULL 与有效地址、断言与运行时错误处理，实践 const 和失败清理。
tags: [C语言, 进阶, 指针, 代码规范]
category: C语言
series: C语言进阶笔记
seriesOrder: 17
draft: false
---

> 依据鹏哥 C 语言进阶第 17 章整理。

## 1. 引入：非空指针为什么仍会出错

```c
int *p = malloc(sizeof *p);
// 假定申请成功：
int *borrowed = p;
free(p);
p = NULL;
// borrowed 不再可用，检查 borrowed != NULL 不能证明对象仍存在。
```

指针安全至少包含地址来源、对象生命周期、访问范围和修改权限四个维度。判断非空只解决“不是空指针”，不解决“没有悬空、没有越界、类型正确”。

## 2. 初始化与生命周期

声明指针时让它明确指向有效对象，或用 NULL 表示当前没有对象。不要读取未初始化指针，再试图判断它是否有效。

```c
int number = 10;
int *current = &number;
int *optional = NULL;
```

释放后将拥有者置 NULL 能减少误用，但别名不会被自动更新。借用指针只应在拥有者保证对象存在期间使用，不能擅自释放。函数返回后失效的自动局部数组也不能通过旧地址继续访问。

| 返回方式 | 返回后是否可用 | 责任 |
| --- | --- | --- |
| 返回普通整数值 | 可用 | 无额外资源 |
| 返回调用者提供的缓冲区 | 可用，取决于调用者对象 | 调用者管理 |
| 返回动态分配对象 | 可用，直到释放 | 明确谁销毁 |
| 返回静态对象地址 | 对象持续存在 | 注意共享状态与并发 |
| 返回自动局部数组地址 | 不可用 | 修改设计 |

## 3. assert 解决程序员违反契约的问题

`assert` 定义在 `<assert.h>`。表达式为假时报告诊断并终止程序；定义 `NDEBUG` 后，断言表达式不求值。

```c
// 错误思想：assert(read_data() == 1);
// 若关闭断言，read_data 就完全不执行了。
```

必须避免在断言中做必需的副作用。用户输入错误、文件打不开和分配失败是可预期运行条件，应该用 if 和错误码处理，不应只靠可能被禁用的断言。

完整示例把内部不变量与外部输入检查分开：

```c
#include <stdio.h>
#include <stddef.h>
#include <assert.h>

static double sum_nonempty(const double *values, size_t count)
{
    assert(values != NULL);
    assert(count > 0);
    double sum = 0.0;
    for (size_t i = 0; i < count; ++i) {
        sum += values[i];
    }
    return sum;
}

int average(const double *values, size_t count, double *out)
{
    if (values == NULL || out == NULL || count == 0) {
        return 0;
    }
    *out = sum_nonempty(values, count) / count;
    return 1;
}

int main(void)
{
    double values[] = {70.0, 80.0, 90.0};
    double result = 0.0;
    if (!average(values, 3, &result)) {
        return 1;
    }
    printf("%.1f\n", result);
    return 0;
}
```

返回 0 表示失败，比“失败返回平均值 0”更明确，因为合法平均值也可能是 0。指针非空仍不能验证 count 与真实容量是否匹配，这是调用者必须满足的前提；示例还假定浮点累加保持在可表示范围内。

## 4. const 修饰谁，谁就受限

| 声明 | 可改指向 | 可通过它改所指整数 |
| --- | --- | --- |
| `int *p` | 可以 | 可以（对象本身可写时） |
| `const int *p` | 可以 | 不可以 |
| `int *const p` | 不可以 | 可以（对象本身可写时） |
| `const int *const p` | 不可以 | 不可以 |

`const int *` 与 `int const *` 相同；从右侧理解限定关系，别把“指针不可变”和“数据不可通过该指针修改”混为一谈。

```c
int a = 1;
int b = 2;
const int *view = &a;
view = &b; // 合法
// *view = 3; 不合法

int *const fixed = &a;
*fixed = 3; // 合法
// fixed = &b; 不合法
```

`const` 提供的是通过该访问路径的约束，不让所有别名自动变成只读。但对象若原本定义为 const，强制转换去掉 const 再修改就是未定义行为，不是推荐的“绕过限制”方法。

## 5. 数组接口同时传地址和长度

`void process(int *data, size_t count)` 只保存了一根地址与一个数。约定 count 为 0 时是否允许 NULL，count 大于 0 时必须提供足够可访问元素，写操作还要求对象可修改。

指针加减仅在同一数组及其尾后范围内进行。尾后指针可以作为结束标记，但不能读取。不要先形成非法指针，再以“还没有解引用”为理由认为安全。

长度不会随数组转换自动传给函数，`sizeof` 也无法从任意指针恢复分配大小。好的命名如 `capacity`、`length`、`bytes` 应明确区分容量、已用元素数和字节数。

## 6. 完整案例：创建和销毁 Person

名字独立分配，创建失败时清理已获得资源；创建成功后把所有权交给调用者，调用者只调用配对的销毁函数。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>

struct Person {
    char *name;
    int age;
};

struct Person *person_create(const char *name, int age)
{
    if (name == NULL || age < 0) {
        return NULL;
    }
    size_t length = strlen(name);
    if (length == SIZE_MAX) {
        return NULL;
    }
    struct Person *person = malloc(sizeof *person);
    if (person == NULL) {
        return NULL;
    }
    person->name = malloc(length + 1);
    if (person->name == NULL) {
        free(person);
        return NULL;
    }
    memcpy(person->name, name, length + 1);
    person->age = age;
    return person;
}

void person_destroy(struct Person *person)
{
    if (person != NULL) {
        free(person->name);
        free(person);
    }
}

int main(void)
{
    struct Person *person = person_create("Lin", 20);
    if (person == NULL) {
        return 1;
    }
    printf("%s %d\n", person->name, person->age);
    person_destroy(person);
    person = NULL;
    return 0;
}
```

name 参数必须指向有效字符串，NULL 检查不会替调用者找到缺失的终止符。销毁顺序先释放名字再释放包含它的结构体，反过来会在对象销毁后尝试读成员。

“谁申请谁释放”更准确地说是**资源的拥有者负责配对释放**，不要求申请和释放必须写在同一个函数。所有权可以转移，但应明确约定并保持唯一释放责任。

## 7. 多级 const 与 typedef

`int *` 可转换为 `const int *`，但不能把 `char **` 随意当作 `const char **`，多级指针的限定转换规则更严格，否则可能间接把只读对象交给可写指针。

复杂函数指针可用 `typedef int (*Compare)(const void *, const void *);` 简化，别名应突出行为而不是掩盖所有权。参数名、注释、返回码和 const 共同形成接口，缺少任何一个都可能让调用者猜测。

## 8. 实践检查与练习

每个指针接口检查：有效对象来源、最小容量、允许修改范围、借用期限、失败时状态、释放方式。分配失败应保持对象不变或清理干净，不能留下“部分初始化却当成功使用”的状态。

练习：给 Person 增加复制函数，名字也分配独立存储；测试非法年龄、空名字和创建失败路径；分别带与不带 `NDEBUG` 编译平均值程序，确认正常结果一致且必要动作未藏进断言。
