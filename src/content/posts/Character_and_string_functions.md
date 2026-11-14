---
title: 字符和字符串函数
published: 2026-10-30
tags: ["分享"]
category: C语言
cover: "" # 可以填入图片路径或URL
description: 为了方便操作字符和字符串，C语言标准库中提供了一系列库函数，本便文章将介绍28个库函数，有关字符的函数声明在ctype.h文件中，字符串有关声明在string.h中（C语言）
draft: false
pinned: false
---

##1.字符函数

###1.1字符分类函数

下面的函数是专门做字符分类的，判断一个字符是什么类型的。

iscntrl:任何控制字符

isspace：空白字符：空格' ',换页'\f',换行'\n',回车'\r',制表符'\t'或者锤子制表符'\v'

isdigit:十进制数字'0'~'9'

isxdigit:十六进制数字，包括所有十进制数字字符，小写字母a~f,大写字母A~F

islower：小写字母a~z

isupper:大写字母A~Z

isalpha:字母a~z或A~Z

isalnum：字母或者数字，a~z，A~Z，0~9

ispunct：标点符号，任何不属于数字或者字母的图形字符(可打印)

isgraph:任何图形字符

isprint：任何可打印字符，包括图形字符和空格字符

举例一个函数：

```c
int islower ( int c );
```

islower是能够判断参数部分的c是否是小写字母的，通过返回值来说明是否是小写，是小写返回非0，不是小写返回0。

练习：写一个代码，将字符串中的小写字符改写为大写字符，其他不变。

```c
#include<stdio.h>
#include<ctype.h>
int main()
{
    char arr[] = "Hello World!\n";
    char c = 0;
    int i = 0;
    while(arr[i])
    {
        c = arr[i];
        if(islower(c))
           c -= 32;
        putchar(c);
        i++;
    }
    return 0;
}
```

###1.2字符转换函数（C语言中有两个转换函数）

```c
int tolower( int c );//将传进去的大写字母转换为小写字母
int toupper( int c );//将传进去的小写字母转换为大写字母
```
我们将小写转换为大写，需要-32来完成。但是有了转换函数，我们可以使用tolower来完成

```c
#include<stdio.h>
#include<ctype.h>
int main()
{
    int i = 0;
    char arr[] = "Test String.\n"
    char c;
    while(arr[])
    {
        c = arr[i];
        if(islower(c))
            c = toupper(c);
        putchar(c);
        i++;
    }
    return 0;
}
```
##2.strlen

```c
代码块
size_t strlen (const char * str );
```

功能：统计参数str指向的字符串的长度。统计的是字符串中'\0'之前的字符的个数。

参数：

str：指针，指向了要统计长度的字符串。

返回值：返回了str指向的字符串的长度，返回的长度不会是负数，所以返回类型是size_t.

头文件：<string.h>

###2.1代码演示

```c
#include<stdio.h>
$include<string,h>
int main()
{
    const char* str = "abcdef";
    printf("%zu\n",strlen(str));
    return 0;
}
```

注意事项：a.字符串以'\0'作为结束标志，strlen函数返回的是字符串中'\0'前面的字符个数（不包含\0）

b.参数指向的字符串必须要以'\0'结束，否则结果未知

c.注意函数的返回值为size_t，是无符号的（重要）

###2.2strlen的返回值

```c
代码块
#include<stdio.h>
#include<string.h>
int main()
{
    const char* str1 ="abcdef";
    const char* str2 = "abc";
    if(strlen(str1) - strlen(str2) > 0)
    {
        printf("str1 > str2\n");
    }
    else
    {
        printf("str1 < str2\n"0;)
    }
    return 0;
}
```

###2.3strlen的模拟实现（递归）

一般来说题目不会直接说使用递归，而是：模拟实现strlen，不创建临时变量，来求字符串的长度

```c
代码块
size_t my_strlen(const char*str)
{
    assert(str != NULL)
    if(*str == '\0')
        retrun 0;
    else
        return 1 + my_strlen(str + 1);
}
```

##3.strcpy

```c
代码块
char* strcpy(char * destination,const char* source);
```

功能：字符拷贝，将source指向的字符串，拷贝到destination指向的目标空间中。

参数：destination：指针，指向目的地空间

source：指针，指向源头文件

返回值：strcpy函数返回的目标空间的起始地址

头文件：<string.h>

###3.1代码演示

```c
代码块
#include<stdio.h>
#include<string.h>
int main()
{
    char arr1[10] = {0};
    char arr2[] = "Hello";
    strcpy(arr1,arr2);
    printf("%S\n",arr1);
    return 0;
}
```

注意事项：源字符串必须以'\0'结束，并且会将源字符串中的'\0'拷贝到目标空间。

目标空间必须足够大，以确保能存放源字符串。目标空间必须可修改。

###3.2模拟实现

```c
#include<stdio.h>
#include<string.h>
char* my_strcpy(char*dest,char*str)
{
    char* ret = dest;
    assert(dest != NULL);
    assert(str != NULL);
    while(*dest++ == *str++)
    {
        ;
    }
    retrurn ret;
}
int maain()
{
    char arr1[10] = {0};
    char arr2[] = "Hello";
    my_strcpy(arr1,arr2);
    printf("%s\n",arr1)
    return 0;
}
```

##4.strncpy

```c
代码块
char* strncpy(char * destination,const char* source,size_t num);
```

功能：字符串拷贝。将source指向的字符串，拷贝到destination指向的空间中，最多拷贝num个字符。

功能：字符拷贝，将source指向的字符串，拷贝到destination指向的目标空间中。

参数：destination：指针，指向目的地空间

source：指针，指向源头文件

num：source指向的字符串中最多拷贝的字符个数

返回值：strcpy函数返回的目标空间的起始地址

###4.1代码演示
```c
代码块
#include<stdio.h>
int main()
{
    char arr1[20] ="-------------------";
    char arr2[] = "Little_Trumpet";
    char* str = strnpy(arr1,arr2,5)
    printf("%s\n",arr1);//arr1==Littl
    printf("%s\n",str);
    return 0;
}
```

###4.2比较strcpy和strncpy

strcpy函数把源字符串数据拷贝到\0为止，如果目标空间不够的话，容易出现月结的行为。

strncpy函数指定了拷贝的长度，源字符串不一定要有\0，党源字符串长度>=n时，目标空间不会以\0结尾，后续如果直接当作字符串使用，会存在越界的可能。所以有时需要手动补上\0

##5.strcat

```c
代码块
char* strcat(char* destination,const char* source);
```

功能：字符串追加；将source指向的字符串的内容，追加到destination指向的空间中。

参数：• destination ：指针，指向⽬的地空间• source ：指针，指向源头数据

返回值： strcat 函数返回的⽬标空间的起始地址

头文件：<string.h>

```c
#include <stdio.h> 
#include <string.h> 
int main() 
{ 
    char arr1[20] = "hello "; 
    char arr2[] = "world"; 
    strcat(arr1, arr2); 
    printf("%s\n", arr1); 
    return 0; 
} 
```

使⽤注意事项： • 源字符串必须以 '\0' 结束。 

• ⽬标字符串中也得有 \0 ，在⽬标字符串的 \0 位置开始追加源字符串内容。

• ⽬标空间必须有⾜够的⼤，能容纳下源字符串的内容。 

• ⽬标空间必须可修改.

###5.2 模拟实现

```c
#include<stdio.h>
#include<assert.h>
char* my_strcat(char* dest,const char* str)
{
    char* ret = dest;
    assert(dest != NULL);
    assert(srt != NULL);
    while(*dest)
    {
        dest++;
    }
    while((*dest++ = *srt++))
    {
        ;
    }
    return ret;
}
int main()
{
    char arr1[20] = "Hello ";
    char arr2[] = "World!";
    my_strcat(arr1,arr2);
    printf("%s\n",arr1);
    return 0;
}
```

##6.strncat

```c
代码块
char* strncat(char* destination,const char* source,size_t num)
```

功能：字符串追加；将source指向的字符串的内容，追加到destination指向的空间中,最多追加num个字符。

参数：• destination ：指针，指向⽬的地空间• source ：指针，指向源头数据

返回值： strcat 函数返回的⽬标空间的起始地址

###6.1代码演示

```c
代码块
#include<stdio.h>
#include<string.h>

int main()
{
    char arr1[20] = "Hello ";
    cahr arr[] = "World";
    char* ret = strncat(arr1,arr2,5);
    printf("%s\n",arr1);
    printf("%s\n",ret);
    return 0;
}
```

###6.2strcat和strncat对比

• 参数不同， strncat 多了⼀个参数 

• strcat 函数在追加的时候要将源字符串的所有内容，包含 \0 都追加过去，但是 strncat 函 数指定了追加的⻓度。 

• strncat 函数中源字符串中不⼀定要有 \0 了，如果 num < 源字符串的⻓度，则遇不到 \0 的。 

• strncat 函数不管追加⼏个字符，都会再加上 \0 结尾。 

• strncat 更加灵活，也更加安全。

##7.strcmp

```c
代码块
int strcmp ( const char * str1, const char * str2 );
```

功能：⽤来⽐较 str1 和 str2 指向的字符串，从两个字符串的第⼀个字符开始⽐较，如果两个字符 的ASCII码值相等，就⽐较下⼀个字符。直到遇到不相等的两个字符，或者字符串结束。

参数： • str1 ：指针，指向要⽐较的第⼀个字符串 

• str2 ：指针，指向要⽐较的第⼆个字符串

返回值：标准规定： • 第⼀个字符串⼤于第⼆个字符串，则返回⼤于0的数字 

• 第⼀个字符串等于第⼆个字符串，则返回0 

• 第⼀个字符串⼩于第⼆个字符串，则返回⼩于0的数字

###7.1 代码演⽰：

```c
代码块
#include <stdio.h> 
#include <string.h> 
int main() 
{ 
    char arr1[] = "abcdef"; 
    char arr2[] = "abq"; 
    int ret = strcmp(arr1, arr2); 
    printf("%d\n", ret); 
    if(ret > 0) 
      printf("arr1 > arr2\n"); 
    else if(ret == 0) 
      printf("arr1 == arr2\n"); 
    else
      printf("arr1 < arr2\n");

 return 0;
}
```

###7.2 模拟实现：

 ```c
 代码块
 int my_strcmp (const char * str1, const char * str2) 
 { 
    int ret = 0 ; 
    assert(str1 != NULL); 
    assert(str2 != NULL); 
    while(*str1 == *str2) 
    { 
        if(*str1 == '\0') 
        return 0; 
        str1++; 
        str2++; 
    } 
    return (unsigned char)*str1 - (unsigned char)*str2; 
}
 ```

 ##8. strncmp
```c
代码块
char* strncmp(const char * str1, const char * str2 ,size_t num);
```

功能：字符串⽐较；⽐较 str1 和 str2 指向的两个字符串的内容，最多⽐较 num 字符。

参数： • str1 ：指针，指向⼀个⽐较的字符串 

• str2 ：指针，指向另外⼀个⽐较的字符串 

• num ：最多⽐较的字符个数 

返回值： • 标准规定： 

◦ 第⼀个字符串⼤于第⼆个字符串，则返回⼤于0的数字 

◦ 第⼀个字符串等于第⼆个字符串，则返回0 

◦ 第⼀个字符串⼩于第⼆个字符串，则返回⼩于0的数字

###8.1 代码演⽰

```c
代码块
#include <stdio.h> 
#include <string.h> 
int main() 
{ 
    char arr1[] = "abcdef"; 
    char arr2[] = "abq"; 
    int ret = strcmp(arr1, arr2，4); 
    printf("%d\n", ret); 

 return 0;
}
```

###8.2 strcmp和strncmp⽐较 
• 参数不同 

• strncmp 可以⽐较任意⻓度了 

• strncmp 函数更加灵活，更加安全，推荐使⽤

##9. 软件开发中的建议

前⾯学过的这些函数，其实可以分为两组： 

⻓度不受限制的字符串函数 strcpy，strcat，strcmp

长度不受限制的字符串函数 strncpy，strncat，strncmp

这两组函数这么相似，在实际软件开发中建议使⽤哪⼀组呢？

strcpy/strcat : 最好不用，无长度限制，缓冲溢出即安全漏洞，CWE-676

strncpy：谨慎使用，不保证'\0'终止，需要手动buf[n-1]='\0'

strncat:谨慎使用，参数n含义是“还能追加多少”，容易写错

strncmp：推荐使用，无安全风险，比strcmp更加可控

snprintf：强烈推荐使用替代strcpy/strcat，安全+自带'\0'+可以格式化

其中，CWE-676 就是使⽤潜在危险函数 使⽤潜在危险的函数，可能会导致缓冲区溢出的⻛险，攻击者可通过精确控制覆盖内容，将返回地址 改写为恶意代码的地址，从⽽劫持程序执⾏流。

```c
⼀个缓冲区溢出的代码 
#include <stdio.h> 
int main() 
{ 
    char arr1[5] = {0}; 
    char arr2[10] = "abcdef"; 
    printf("%d\n", arr2 - &arr1[4]); 
    strcpy(arr1, "--------------------------------------"); 
    //拷⻉的数据⾜够⻓，就会覆盖到arr2数组的内容 
    printf("%s\n", arr2); 
    return 0; 
}
```

在了解snprintf函数前需要了解sprintf和sscanf

```c
代码块
int printf ( const char * format, ... ); 
int sprintf ( char * str, const char * format, ... );
int snprintf ( char * s, size_t n, const char * format, ... ); 
int scanf ( const char * format, ... ); 
int sscanf ( const char * s, const char * format, ...);
```

##10. sprintf 和 snprintf

###

```c
代码块
int sprintf ( char * str, const char * format, ... );
```

功能：将格式化数据写⼊字符数组（字符串）。它类似于 printf ，但输出⽬标不是控制台或⽂件， ⽽是⽤⼾指定的内存缓冲区。常⽤于动态⽣成字符串、拼接数据或转换数据格式。简⽽⾔之就是将格 式化的数据转换成⼀个字符串。

参数： • str ：指向字符数组的指针，⽤于存储⽣成的字符串（需确保⾜够⼤以防⽌溢出）。 

• format ：格式化字符串，定义输出格式（如 %d 、 %f 、 %s 等）。

• ... ：可变参数列表，提供与格式字符串中说明符对应的数据。

返回值： • 成功时：返回写⼊ buffer 的字符数（不包括结尾的空字符 \0 ）。

• 失败时：返回负值。

```c
代码演⽰： 
代码块
#include <stdio.h> 
struct Stu 
{ 
    char name[30]; 
    int age; 
    float score; 
};
int main() 
{ 
    struct Stu s = {"zhangsan", 100, 85.5f}; 
    printf("%s %d %.1f\n", s.name, s.age, s.score); 
    //打印到屏幕上了 
    //将s中的结构数据转换成⼀个字符串 
    char arr[100] = {0}; 
    sprintf(arr, "%s %d %.1f", s.name, s.age, s.score);
    printf("%s\n", arr); 
     return 0; 
}
```

###10.2 snprintf

```c
代码块
int snprintf ( char * s, size_t n, const char * format, ... );
```

功能：将可变参数按照 format 格式化，并将结果字符串安全地写⼊到 s 指向的缓冲区中。与 sprintf 的最⼤区别是：它强制受 n 限制，绝不会写⼊超过缓冲区实际⼤⼩的内容。

参数： • s : ⽬标缓冲区指针，⽤于存放结果字符串。 
• n : 缓冲区 s 的总字节数。函数最多写⼊ n-1 个字符，并保证以 '\0' 结尾（只要 size > 0 ）。 

• format : 格式化控制字符串，与 printf 系函数规则相同。 

• ... : 可变参数列表，与 format 中的格式说明符⼀ 对应。 

返回值： • 返回如果缓冲区⾜够⼤时，应该写⼊的字符总数（不含末尾的 '\0' ）。 

• 若返回值 ret 满⾜ 0 <= ret < size ，说明输出数据完整写⼊。 

• 若 ret >= size ，说明输出数据被截断，缓冲区中仅包含前 size-1 个字符和⼀个 '\0' 。

```c
代码块
#include <stdio.h> 
struct Stu 
{ 
    char name[30]; 
    int age; 
    float score;
};
int main() 
{ 
    struct Stu s = {"zhangsan", 100, 85.5f}; 
    printf("%s %d %.1f\n", s.name, s.age, s.score); //打印到屏幕上了 
    //将s中的结构数据转换成⼀个字符串 
    char arr[100] = {0}; 
    int r = snprintf(arr, sizeof(arr), "%s %d %.1f", s.name, s.age, s.score); 
    printf("%s\n", arr); printf("%d\n", r); 
    return 0; 
}
```

###10.3 snprintf 替代 strcpy/strcat

```c
代码块
#include <string.h> 
#include <stdio.h> 
int main() 
{ 
    char buf[20] = "----------"; 
    char src[] = "abcdef"; 
    // 1.复制字符串 
    snprintf(buf, sizeof(buf), "%s", src); //strcpy 
    printf("%s\n", buf); 
    
    // 2.拼接 
    size_t len = strlen(buf); 
    snprintf(buf + len, sizeof(buf) - len, "%s", src); //strcat 
    printf("%s\n", buf); 
    return 0;
}
```

##11.sscanf

```c
代码块
int sscanf ( const char * str, const char * format, ...);
```

功能：从字符串中读取格式化数据。它与 scanf 类似，但输⼊源是内存中的字符串⽽⾮控制台或⽂ 件。常⽤于解析字符串中的结构化数据（如提取数字、分割⽂本等）。

参数： • str ：要解析的源字符串（输⼊数据来源）。 

• format ：格式化字符串，定义如何解析数据（如 %d 、 %f 、 %s 等）。 

• ... ：可变参数列表，提供存储数据的变量地址（需与格式字符串中的说明符匹配） 

返回值： • 成功时：返回成功解析并赋值的参数数量（⾮负值）。

• 失败或未匹配任何数据：若输⼊结束或解析失败，返回 EOF （通常是 -1 ）。

```c
代码演⽰： 
代码块
#include <stdio.h> 
struct Stu 
{ 
    char name[30]; 
    int age; 
    float score; 
};
    int main() 
    { 
        char str[] = "zhangsan 100 85.5"; 
        struct Stu s = {0}; 
        //从str中的字符串内中解析出格式化的数据，存放在结构体中 
        sscanf(str, "%s %d %f", s.name, &(s.age), &(s.score)); 
        //打印结构体s中的数据 
        printf("%s %d %.1f\n", s.name, s.age, s.score); 
        return 0;
    }
```

##12.strstr

```c
代码块
char* strstr(const char* str1,const char* str2);
```

功能： strstr 函数，查找 str2 指向的字符串在 str1 指向的字符串中第⼀次出现的位置。 简⽽⾔之：在⼀个字符串中查找⼦字符串。

参数： • str1 ：指针，指向了被查找的字符串 

• str2 ：指针，指向了要查找的字符串

返回值： • 如果str1指向的字符串中存在str2指向的字符串，那么返回第⼀次出现位置的指针 

• 如果str1指向的字符串中不存在str2指向的字符串，那么返回NULL 

头⽂件： <string.h>

###12.1 代码演⽰

```c
代码块
/* strstr example */ 
#include <stdio.h> 
#include <string.h> 
int main () 
{ 
    char str[] ="This is a simple string"; 
    char * pch; 
    pch = strstr (str,"simple"); 
    if (pch != NULL) 
    printf("%s\n", pch); 
    else
    printf("查找的字符串不存在\n"); 
    return 0; 
}
```

###12.2 strstr的模拟实现

```c
代码块
char* strstr(const char* str1,const char* str2)
{
    char *cp = (char* )str1;
    char *s1,*s2;
    if(!*str2)
    {
        return((char*)str1);
    }
    while(*cp)
    {
        s1 = cp;
        s2 =(char* )str2;

        while(* s1 && * s2 &&!(* s1 - * s2))
        {
            s1++;s2++;
        }
        if(1*s2)
        return(cp);
    cp++;
    }
    return(NULL);
}
```

strstr 函数的实现有多种，可以暴⼒查找，也有⼀种⾼效⼀些的算法：KMP，有兴趣的可以去学习.

