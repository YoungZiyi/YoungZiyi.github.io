---
title: What is metaclass
date: 2016-10-31 11:11:11 +0800
categories: python
tags: python
---

# What is metaclass?

Reference: [answer on stackoverflow](http://stackoverflow.com/questions/100003/what-is-a-metaclass-in-python/6581949#6581949)

## python中的class也是对象  
大多数的语言中，class是一段描述如何创建对象的代码，python也是这样，不过python中的class不仅仅是这样，它本身也是一个对象。  
作为一个对象，你就能对它进行如下操作:  
+ 将class对象赋给一个变量  
+ 复制它  
+ 给它添加属性  
+ 可以将它作为一个参数传递给一个函数  

## 动态的创建class  
你可以随意的创建一个class，就想任何其他对象一样的。你可以在函数中创建一个对象并将它返回，这并不是很‘动态’，因为这样只不过class的定义写进函数中而已，
当你使用class关键字时，python就会自动创建一个class对象。python提供了一个手动创建class对象的方法——`type()`  

``` python
type(object)    # 返回对象的type
```
以上是我们熟悉的用法，但是`type()`还有一个功能——创建class对象。
``` python
type(classNamem classParentTuple, classAttributeDictionary)    # 返回一个新的type（class对象）

>>> Foo = type('Foo', (), {'bar':1})
>>> f = Foo()
>>> type(f)
>>> __main__.Foo
>>> f.bar
>>> 1
```
给class对象绑定方法
``` python
>>> def echoBar(self):
>>>     print(self.bar)
>>> Foo = type('Foo', (), {'bar':1, 'echoBar':echoBar})
>>> f = Foo()
>>> f.echoBar()
>>> 1
```

## 什么是metaclass呢？
***任何对象都是由class或metaclass所创建的，class对象就是由metaclass创建的***  
> `type`类似于`str`（用来创建string实例的class），`int`（创建整数对象的class），它就是python用来创建所有class对象的metaclass  
`type`是内置的metaclass，你也可以创建自己的metaclass  

## __metaclass__属性
通过设置class的`__metaclass__`属性来指定创建class对象所使用的metaclass，__metaclass__的值可以是任何可调用的对象，包括函数，
python会去查找__metaclass__所指定的对象，如果找不到，python将会使用`type`来创建class对象。查找的顺序是1.在class内部查找；2.在模块内查找；
3.在父类中查找  
> 小心！`__metaclass__`属性不会被继承！metaclass所指定的对象必须要能创建一个class对象！  

## 定制metaclass
使用metaclass的目的是在创建class对象的时候做一些操作，就想在创建类实例的时候，自动调用__init__来做一些操作。  
举一个‘愚蠢’的例子来说明，在创建class对象的时候自动将所有自定义的attribute改成大写字母（显然没什么卵用），现在我们来通过设置metaclass来实现该目的。  
*通过函数*
``` python
def upper_case(futrue_class_name, futrue_class_parent, future_class_attribute):
    uppercase_attr = {}
    for name, val in futrue_class_attribute.items():
        if not name.startswith('__'):
            uppercase_attr[name.upper()] = val
        else:
            uppercase_attr[name] = val
            return type(futrue_class_name, futrue_class_parent, uppercase_attr)
### python2.x
__metaclass__ = upper_case
class Foo():
    bar = 1
print(hassattr(Foo, 'bar'))    # return False
print(hassattr(Foo, 'BAR'))    # return True

### python3.x    (documetation)[https://docs.python.org/3/library/2to3.html?highlight=metaclass#2to3fixer-metaclass]
class Foo(metaclass=upper_case):
    bar = 1
print(hassattr(Foo, 'bar'))    # return False
print(hassattr(Foo, 'BAR'))    # return True
```
*通过class*
``` python
class UpperAttrMetaclass(type):
    uppercase_attr = {}
    def __new__(uperattr_metaclass, futrue_class_name, futrue_class_parent, future_class_attribute):
    # def __new__(cls, clsname, bases, dct):    # this is production code
        for name, val in future_class_attribute.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val
                return type(futrue_class_name, futrue_class_parent, uppercase_attr)
        return type(future_class_name, future_class_parents, uppercase_attr)
        # or
        return type.__new__(future_class_name, future_class_parents, uppercase_attr)    # this is much OOP
        # or even cleaner
        return super(UpperAttrMetaclass, cls).__new__(cls, clsname, bases, uppercase_attr)
### python2.x
__metaclass__ = UpperAttrMetaclass
class Foo():
    bar = 1
print(hassattr(Foo, 'bar'))    # return False
print(hassattr(Foo, 'BAR'))    # return True

### python3.x    (documetation)[https://docs.python.org/3/library/2to3.html?highlight=metaclass#2to3fixer-metaclass]
class Foo(metaclass=UpperAttrMetaclass):
    bar = 1
print(hassattr(Foo, 'bar'))    # return False
print(hassattr(Foo, 'BAR'))    # return True
```
`__new__`和`__init__`差不多，后者是在创建实例时自动被调用，前者则是在创建class对象是被调用，`__new__`中第一个参数是class对象本身，
就像class方法第一个参数self是实例本身一样，定义如此，不必在意  
好了，这就是所有有关metaclass的概念了  

## 为什么使用metaclass class而不是function？
__metaclass__属性可以是任何可调用的对象，那为什么要使用class，而不是function呢？class明显要复杂的多，自己找虐吗？  
+ 使用class使目的更明显  
+ 可以使用OOP思想，使metaclass可以继承于其它metaclass，覆盖父类中的方法，甚至metaclass中还可以使用metaclass！  
+ 可以使你的代码结构更清晰，简单的代码用不着metaclass，而复杂的代码中使用class来作为metaclass可以将一组方法group起来，使代码更加易读  
+ 可以hook on `__new__``__init__``__call__`，你可以将所有的操作都放在`__new__`方法中（因为它是最先调用的方法），但是分开操作更有意义一些  
+ metaclass之所以叫metaclass这是有原因的！use class and shut up!  

## 为什么使用metaclass？
为什么使用这种不起眼而且易出错的特性？  
> 你根本不用关心metaclass，因为99%的人都用不上，如果你想知道你为什么用这个特性，那么你就是那99%中的一员...  
如果你实在想知道，可以举一个使用metaclass的例子来说明一下:  
在web框架django中，我们这样定义model:
``` python
class Person(models.Model):
    name = models.CharField(maxlength=30)
    age = models.IntegerField()
```
然后我这样使用model：
``` python
guy = Person(name="Jerry Young", age=23)
print(guy.age)    # return 23 instead of models object
```
定义时age是一个models.IntegerField对象，而打印出来却是一个int，这就是因为创建Person class对象时使用了metaclass做了一些操作。  

## 写在最后
1.你应该清楚class是对象，也可以用来创建对象；class本身就是metaclass的实例对象；python所有的东西都是对象，他们都是class或metaclass的实例，除了`type`；`type`是它自己的metaclass  
2.你应该清楚metaclass是很复杂的，你可能不会想在简单的代码逻辑中使用它，你可以使用两种类似功能的技术：  
+ monkey patching  
+ class decorators  
99%的情况下你想修改class可以使用上述两种替代方案，但是99%的情况下你不需要修改class。  








