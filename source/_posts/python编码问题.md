---
title: python编码问题
date: 2020-04-02 14:49:01
categories: 技术
tags: python
---

[参考](https://nedbatchelder.com/text/unipain.html)


## 编码基础

![ascii](/img/USASCII_code_chart.png "source: upload.wikimedia.org")

这是ascii编码表, 从0到127, 每一个数字(code point)对应一个字符, 编码表就是这种对应
关系的映射表, 由于计算机只能处理字节, 所以还需要将数字保存为字节, ***ASCII编码***
规定使用一个字节

一个字节提供的编码位是从0到255, 足以存下ascii编码表中所有字符, 为了支持更多语言, 人
们使用更多的字节来定义一个字符, 例如使用两个字节的`gb2312`, `big5`,  他们分别用来处
理中文简体和中文繁体

不同的语言使用不同的编码表导致大量不同的编码表出现, 于是人们就想能不能一套编码表包含
所有语言, 于是`unicode`就出现了, `unicode`总共有1100000个编码位(code point), 目前
用到了110000个, 每种语言中的每一个字符在`unicode`中都使用一个编码位, 有了编码位与字
符之间的映射关系后, 还需要将编码位转成字节提供给计算机处理, `ASCII`编码就是简单的使
用一个字节来保存编码位, `unicode`标准定义了多种编码方式(encoding)来将编码位转成字
节, 例如`UTF-8`, `UTF-16`, `UTF-32`, `UCS-2`, `UCS-4`等等, `UTF-8`是我们常用的编
码, 它使用变长字节数来表示一个`unicode`编码位, `ASCII`中的字符在`UTF-8`中也是使用一
个字节, 且相同, 所以`ASCII`是`UTF-8`的子集


## python2中的编码

python2中有两种字符串类型, 一种是`str`类型, 它储存的是字节, 另一种是`unicode`, 它储
存的是编码位, 如下:

```python
s1 = 'hi 你好'
print(type(s1))# <type 'str'>

# use a 'u' prefix
s2 = u'hi 你好'
print(type(s2))# <type 'unicode'>
```

`str`类型的字符串保存的是字节, 所以使用`len`函数计算长度返回的将是字节数, 而`unicode`
类型的字符串保存的是编码位, 将返回编码位个数

```python
print(len(s1))# 9

print(len(s2))# 5
```

`str`和`unicode`的字符串可以互相转换, `unicode`字符串可以通过`encode`方法转变成储存
字节的`str`类型, `str`字符串可以通过`decode`转变成`unicode`

```python
print(s1.decode('utf-8'))# hi 你好

print(s2.encode('utf-8'))# hi 你好
```

`s1`是`str`类型, 保存的是计算机使用的字节码(Bytes), 它使用`decode`方法根据`UTF-8`
编码表可以解码成`unicode`, `s2`是`unicode`类型, 它可以使用`encode`方法, 同样根据
`UTF-8`编码表可以转变成字节码(Bytes), 如果使用错误的编码类型, 将返回编码错误

```python
print(s1.decode('ascii'))
# Traceback (most recent call last):
#   File "<stdin>", line 1, in <module>
# UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 3: ordinal not in range(128)

# 解释: s1 中的中文字符编码超出了ascii所能表示的范围, 所以解码失败

# 不提供编码参数的话将使用默认编码
# 查看环境默认编码: sys.getdefaultencoding(), 我这里是 ascii
print(s2.encode())
# Traceback (most recent call last):
#   File "<stdin>", line 1, in <module>
# UnicodeEncodeError: 'ascii' codec can't encode characters in position 3-4: ordinal not in range(128)

# 解释: s2中的中文编码位超过了ascii所能表示的范围, 所以编码失败

print(s1.decode('ascii', 'replace'))# hi ������
print(s2.encode('ascii', 'replace'))# hi ??

# 解释: 第二个参数使用replace将会使用'?'代替不可编码/解码的字符
# decode时返回了6个问号, 而encode只有两个, 这是因为s1字符串内部储存的是字节, 而中文
# '哈哈'使用了六个Bytes, 使用ascii解码时将六个字节当成六个字符处理, 因为超过了ascii
# 所能表示的范围, 所以返回六个问号, 而s2储存的时编码位, 其内部的中文'哈哈'是两个
# unicode编码位, 使用ascii编码时是当作两个字符处理, 超过了ascii表示范围, 返回两个问
# 号
```

python2中很多地方都会存在隐性转换, 比如尝试将`unicode`字符串和`str`类型(字节)拼接起
来, python2先会自动将`str`字节字符串decode成默认编码类型的字符串, 然后再拼接

```python
print(u'hi' + ' nihao')# hi nihao

print(u'hi' + ' 你好')
# Traceback (most recent call last):
#   File "<stdin>", line 1, in <module>
# UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 1: ordinal not in range(128)

# 解释: python2首先会将' 你好'按默认编码ASCII解码, 但是中文是超出了ASCII表示的范围,
# 所以报UnicodeDecodeError
```

python2中隐性的字符串类型转换在所有字符都是ascii时表现正常, 当存在其他编码字符时就
可能导致错误


# python3中的编码

python3中也有两种类型的字符串, 直接由引号括起来的字符串是`str`类型, 普通字符串加`b`
前缀的字符串属于`bytes`类型

```python
s1 = 'hi 你好'
print(type(s1))# <class 'str'>

s2 = b'hi how old are you'
print(type(s2))# <class 'bytes'>
```

python2中的`str` 等价于 python3中的` bytes`  
python2中的`unicode` 等价于 python3中的` str`

