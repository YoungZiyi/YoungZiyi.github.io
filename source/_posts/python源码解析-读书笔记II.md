---
title: python源码解析 读书笔记II
date: 2017-7-27 22:03:11 +0800
categories: python
tags: 读书笔记
---

# python源码解析 读书笔记II —— python虚拟机

## python执行过程

python源代码 输入 -> python解释器 编译 -> python字节码 输入 -> python虚拟机 执行

## python编译编出来的到底是什么玩意？

python是一门解释型语言，它编译后得到的不是机器码，而是叫字节码，就是以`.pyc`为后缀的文件。

python源代码中的静态信息（字符串、常量值）储存在一个运行时的对象中，这个对象就是`PyCodeObject`，python编译的结果其实就是这个对象，而`.pyc`文件就是这个对象在硬盘上的表现形式。

PyCodeObject对象的声明
code.h
```c
typedef struct{
  PyObject_HEAD
  int co_argcount; // Code block中位置参数的个数（位置参数？）
  int co_nlocals; // Code block中局部变量的个数（包括位置参数）
  int co_stacksize; // 执行这个code block需要的栈空间
  int co_flags;
  PyObject *co_code; // code block编译所得字节码的指令序列，以PyStringObject的形式存在
  PyObject *co_consts; // 保存该code block中所有常量，以PyTupleObject的形式存在
  PyObject *co_names; // 保存该code block中所有符号（？），以PyTupleObject的形式存在
  PyObject *co_varnames；// 保存该code block中左右局部变量名
  PyObject *co_freevars; // Python实现闭包需要用到的东西
  PyObject *co_cellvars; // code block中内部嵌套函数所引用的局部变量名集合
  /* THe rest doesn't count for hash/cmp */
  PyObject *co_filename; // code block中对应的py文件完整路径
  PyObject *co_name; // code block的名字，通常是函数名或类名
  int co_firstlineno; // code block对应py文件中的起始行
  PyObject *co_lnotab; // 字节码指令与py文件中source code行号的对应关系，以PyStringObject的形式存在
} PyCodeObject;
```

python代码中每一个code block都会创建一个PyCodeObject对象与之对应。code block就是一个 作用域/命名空间，例如一个 函数/类/源文件。

怎样生成`.pyc`文件
+ import一个模块时，会生成该模块的pyc文件
+ python标准库中的py_compile、compiler工具

import一个模块时，python会在设定好的path中按顺序查找以这个模块名命名的dll和pyc文件，如果只找到.py文件，那就会把源代码编译成PyCodeObject对象，并创建.pyc文件，把对象写入文件中，接下来python才会对.pyc文件进行加载操作。实际上就是把pyc文件的Code对象在内存上重新复制出来。

co_lnotab域储存的是字节码指令与py文件中source code行号的对应关系，而在python2.3以前，有一条字节码指令，叫SET_LINENO，这条字节码会记录py文件中source code的位置，python2.3之后编译不会再产生这条字节码，因为字节码代表的是运行时的行为，而记录源代码行号的动作完全可以在编译时完成，所以python会在编译时直接将这个信息记录到co_lnotab中。

python中可以访问PyCodeObject对象。
```python
source = open('demo.py').read()
co = compile(source, 'demo.py', 'exec') # co就是PyCodeObject
type(co) # <type 'code'>
```

## pyc文件的生成

python的import机制会产生pyc文件，那么import时到底做了哪些操作？
import.c
```c
static void write_compiled_module(PyCodeObject *co, char *cpathname, long mtime)
{
  FILE *fp;
  fp = open_exclusive(cpathname); // 排他性打开文件
  // 【1】写入Python magic number
  PyMarshal_WriteLongToFile(pyc_magic, fp, Py_MARSHAL_VERSION);
  // 【2】写入时间信息
  PyMarshal_WriteLongToFile(mtime, fp, Py_MARSHAL_VERSION);
  // 【3】写入PyCodeObject对象
  PyMarshal_WriteObjectToFile((PyObject *)co, fp, Py_MARSHAL_VERSION);
  
  fflush(fp);
  fclose(fp);
}
```
magic number是啥？
他是python定义的一个整数值，一般不同的版本的magic number不同，，这个值用来保证python的兼容性的。python加载pyc文件时首先会检查这个值，如果不同则会拒绝加载这个pyc文件。
另外这个值定义在import.c中。

为什么要写如时间信息？
这主要是为了保证pyc文件与源文件同步，当用户修改了源文件，python加载了较旧的pyc文件后，会检查发现pyc的时间早于py文件，于是会自动重新编译源文件，生成新的pyc文件。

PyCodeObject怎么写入文件中？
根据对象中不同类型的域调用不同的方法去写文件，在写对象之前都会先写入对象的类型，这样对pyc文件再次加载有重要作用。
