---
author: wulei
comments: true
date: 2019-09-11 18:21:00+00:00
slug: python_tricks_two
title: Python Tricks (two)
tags:
- Python
- lambda
- 装饰器
---

这一节只要讨论Python中和函数相关的一些技巧。首先，函数是Python的头等对象，可以把函数分配给变量、存储在数据结构中、作为参数传递给其他函数以及作为其他函数的返回值。

# 函数可携带局部状态

函数可以包含函数，可以从父函数返回，并且可以携带父函数的一些状态，即可以携带这个函数创建的环境中的状态。这种函数称为词法闭包(lexical closure)或者简称为闭包。


```python
def get_speak_func(text, volumn):
    return (lambda : text.lower() + '...') if volumn < 0.5 else lambda : text.upper() + '!'
```


```python
get_speak_func('Hello, World', 0.7)()
```




    'HELLO, WORLD!'



可以看到返回的函数中是没有传入text参数的，但是函数依然可以运行，因为返回的函数携带了它被创建的环境中的状态

# 对象也可作为函数使用

Python中所有的函数都是对象，但反之却不成立。有些对象不是函数，但可以像函数一样被调用，此时可以当作函数对待。如果一个对象是可调用的，说明这个对象可以使用圆括号函数调用语法，并且可以传入参数，这是因为对象实现了\_\_call\_\_函数：


```python
class Adder:
    def __init__(self):
        pass
    
    def __call__(self, x, y):
        return x + y
```


```python
adder = Adder()
adder(3, 4)
```




    7



# 不应过度使用lambda

在某些情况下，lambda语句的使用可以简化代码，使代码逻辑更清晰，关于lambda的益处就不说了。这里主要记录一下lambda表达式不该使用的地方，比如在定义类的时候：


```python
class Car:
    rev = lambda self : print('Wroom!')
    crash = lambda self : print('Boom!')
    
my_car = Car()
my_car.crash()
```

    Boom!


在这种情景下使用lambda虽然使代码更简洁和fancy，但是使可读性大大降低了。有些时候在使用map()函数和filter()函数的时候，使用列表解析式或者生成器表达式也会使代码清晰一些：


```python
# 不够清晰
list(filter(lambda x : x % 2 == 0, range(16)))
```




    [0, 2, 4, 6, 8, 10, 12, 14]




```python
# 清晰明了
[x for x in range(16) if x % 2 == 0]
```




    [0, 2, 4, 6, 8, 10, 12, 14]



# 装饰器

装饰器用来包装一个函数，改变或者拓展函数的行为，而无须改变函数本身的代码。装饰器的基本原理是：装饰器是一个函数，接受一个可调用对象，对其进行包装，最后返回一个包装好的另一个可调用对象，程序执行这个包装过的可调用对象。最简单的装饰器的例子：


```python
def uppercase(func):
    def wrapper():
        original_result = func()
        modified_result = original_result.upper()
        return modified_result
    return wrapper
```

使用装饰器有两种方法，第一种直接调用装饰函数来修饰对象，第二种是使用Python的装饰器语法：


```python
# 第一种方法
def greet1():
    return 'Hello'

print(uppercase(greet1)())

# 第二种方法
@uppercase
def greet2():
    return 'Hello'

print(greet2())
```

    HELLO
    HELLO


## 将多个装饰器应用于一个函数

多个装饰器可以作用于同一个函数，并且叠加自己的作用，所有装饰器才可以以组件的形式重复使用。装饰器作用的顺序是自上而下，如下面的例子


```python
def strong(func):
    return lambda : '<strong>' + func() + '</strong>'

def emphasis(func):
    return lambda : '<emphasis>' + func() + '</emphasis>'
```


```python
@strong
@emphasis
def greet():
    return 'Hello!'

print(greet())
```

    <strong><emphasis>Hello!</emphasis></strong>


如果将上面的例子使用传统方式来改写，那么装饰函数的调用链如下


```python
print(strong(emphasis(lambda : 'Hello!'))())
```

    <strong><emphasis>Hello!</emphasis></strong>


## 装饰接受参数的函数

之前的例子中要装饰的函数都是没有参数的，而大多数情况下函数都接受参数，这个时候需要使用Python的变长参数和关键字参数，以及\*和\*\*参数解包的特性，比如一个空的装饰器：


```python
def proxy(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@proxy
def greet3(*args, **kwargs):
    if args:
        print(args)
    if kwargs:
        print(kwargs)

greet3(1, 2, 3, str = 'Hello!')
```

    (1, 2, 3)
    {'str': 'Hello!'}


# 函数的空值返回

Python在所有函数的末尾都隐形添加了`return None`语句，默认情况下所有函数都可以返回空值，我们也可以使用显式的`return`语句来替代隐式的`return None`语句。那么什么是有应该显式的返回空值，什么时候应该让解释器默认返回空值呢？如果一个函数我们不是关注其返回值，没有返回值的时候，就使用隐性的空值返回。
