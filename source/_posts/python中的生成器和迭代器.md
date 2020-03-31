---
title: python中的迭代器和生成器
date: 2020-03-26 21:15:32
categories: python
tags: python
---

复习一下python的高级特性

## 可迭代对象

当需要迭代一个对象时, 会自动调用`iter`函数, `iter`函数会检查对象是否实现了
`__iter__`方法, 如果有则调用对象的`__iter__`方法获取一个迭代器, 没有则继续检查对象
是否实现了`__getitem__`方法, 如果有, python会自动创建一个迭代器, 使用从0开始的索引
为参数调用`__getitem__`, 该方法将返回对应索引的元素, 如果`__getitem__`方法也不存
在, 则抛出`TypeError`异常, 提示对象不可迭代

可迭代对象(iterable)就是使用内置函数`iter`可以获取迭代器的对象, 该对象要么实现了能
够直接返回迭代器的`__iter__`方法, 要么实现了接受从0开始的索引为参数的`__getitem__`
方法

由于`__iter__`方法需要返回迭代器, 先给出实现`__getitem__`方法的可迭代对象示例
```python
class it1:
    def __init__(self, text):
        self.text = text
    def __getitem__(self, index):
        return self.text[index]

obj = it1('abcdefg')
for i in obj:
    # 依次打印 a, b, c, d, e, f, g
    print(i)
```


## 迭代器

迭代器(iterator)就是实现了`__next__`方法的对象, 该方法也是返回序列的当前元素, 它跟
`__getitem__`的区别就是, 当前元素的索引是保存在迭代器对象实例中, 而`__getitem__`是
由python解释器自动创建的迭代器提供的索引

迭代器实例
```python
class it2:
    def __init__(self, text):
        self.text = text
        self.index = 0
    def __next__(self):
        try:
            curr_char = self.text[self.index]
        except IndexError:
            raise StopIteration()

obj = it2('abcdefg')
print(next(i2))# 输出 a
print(next(i2))# 输出 b
print(next(i2))# 输出 c
print(next(i2))# 输出 d
print(next(i2))# 输出 e
print(next(i2))# 输出 f
print(next(i2))# 输出 g
```

我们使用`for`语句迭代一个对象(`obj`)过程如下:
1. 使用内置函数`iter`获取`obj`对象的迭代器 (分两种情况, 第一种是通过`__iter__`直接获取对象的迭代器,
第二种是通过python解释器自动创建迭代器, 并通过`obj`对象的`__getitem__`方法获取序列元素)
2. 获得迭代器`iter`后, 循环调用`next(iter)`获取元素, 直到`__next__`方法抛出StopIteration结束循环


## 生成器

生成器是迭代器的子集, 所以它也属于迭代器, 只是它的实现方式不一样, 生成器更简洁, 而且
它不需要像迭代器一样维护一个索引, 它是通过`yield`语句来实现的

生成器函数实例
```python
def it3(text):
    for curr_char in text:
        yield curr_char

obj = it3('abcdefg')
for i in obj:
    # 依次打印 a, b, c, d, e, f, g
    print(i)

```

代码在执行到`yield`时, 将返回其后的值, 然后并不像`return`语句一样退出, 而是挂起函数
的状态, 下次再从当前胃继续执行

生成器表达式
```python
# 列表推导式
li = [x**2 for x in range(5)]
print(li)# 输出 [0, 1, 4, 9, 16]
# 生成器表达式
li = (x**2 for x in range(5))
print(li)# <generator object <genexpr> at 0x7ff563aa6938>
print(next(li))# 输出 0
print(next(li))# 输出 1
print(next(li))# 输出 4
print(next(li))# 输出 9
print(next(li))# 输出 16
print(next(li))# 自动抛出StopIteration异常
```

生成器除了实现简洁外, 它还有节省内存的优点, 它不是一次性构建整个结果列表, 它有延迟计
算的特点, 按需产生结果
