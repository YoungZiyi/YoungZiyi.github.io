---
title: How to read statement in C
date: 2016-11-25 11:11:11 +0800
categories: 技术
tags: C
---

# How to read statement in C?

Reference: [解读C的声明](http://avnpc.com/pages/c-pointer)

发现一个解读C语言声明的好方法， 记录下来以备后患.  
## 使用英语解读C语言声明  
先上一组解读例子：  
``` c
int hoge;
// eng: hoge is variable of int
// chn: hoge 是 int（类型） 的 变量
int hoge[10];
// eng: hoge is array[10] of int
// chn: hoge 是 int（类型）  的 数组【10个元素】
int hoge[10][3];
// eng: hoge is array[10] of array[3] of int
// chn: hoge 是 int（类型）  的 数组【10】  的 数组【3】
int *hoge[10];
// eng: hoge is array[10] of pointer to int
// chn: hoge 是 指向int（类型）指针 的 数组【10】
int (*hoge)[3];
// eng: hoge is pointer to array[3] of int
// chn: hoge 是 指向 int（类型）的数组  的 指针
int func(int a);
// eng: func is function(int a) returning int
// chn: func 是 返回int（类型） 的 函数（int a）
int (*func)(int a);
// eng: func is pointer to function(int a) returning int
// chn: func 是 指向 返回int（类型）的函数（int a） 的 指针
```
这种格式化的翻译不是真正的英语， 前期靠这种方式理解C语言声明确实很low， 不要着急， 熟练之后再鄙视吧。

## 翻译规则
基本元素的翻译：  
+ aaa（变量名hoge、函数名func）      => aaa is  | aaa 是  
+ aaa                           => variable of ...  | 
+ bbb[10]                       => array of ...  
+ ccc()                         => function returning ...  
+ *ddd                          => pointer to ...  
翻译顺序：
以 `int (*func)(int a)` 为例  
1. 首先翻译表示符： func is  
2. 优先翻译标示符所在括弧中的内容： func is pointer to  
3. 翻译用来表示表示符类型的符号（例如用来表示数组的[]， 用来表示函数的()）： func is pointer to function returning  
4. 最后， 追加类型指定符（最左边用来指定类型）： func is pointer to function returning int  
最后的翻译结果就是：  
> func is pointer to function returning int  
这句话也不好理解， 再稍加润色一下就是：  
> func is a poniter that point to the function that returning int  
翻译成中文就是：
> func 是 指向 返回int的函数 的指针  

练习：  
`char *name[10]`  
> name is array[10] of pointer to char  
> name 是 指向char 的 数组  
`double *d`  
> d is pointer to double  
> d 是 指向 double类型 的 指针  
。。。
