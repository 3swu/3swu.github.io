---
author: wulei
comments: true
date: 2019-09-08 16:0:00+00:00
slug: python_tricks_one
title: Python Tricks (one)
tags:
- Python
- 断言
- with
- 字符串
---

记录一下Python一些有趣实用的tricks

# 断言
Python的断言是一种调试工具，用来测试某个断言条件。如果断言为真，则程序继续执行，如果断言为假，则程序会抛出`AssertionError`异常并且打印异常信息。

## 示例
假设需要编写一个为商品添加打折信息的函数


```python
def apply_discount(product, discount):
    price = int(product['price'] * (1 - discount))
    assert 0 <= price <= product['price'], 'invalid price'
    return price
```

其中使用断言来验证折扣后价格的正确性，接下来试一下有用的和无用的折扣


```python
shoes = {'name': 'fancy shoes', 'price': 14900}
```


```python
apply_discount(shoes, 0.25)
```




    11175




```python
apply_discount(shoes, 2.0)
```


    ---------------------------------------------------------------------------

    AssertionError                            Traceback (most recent call last)

    <ipython-input-4-e53e95c2a957> in <module>
    ----> 1 apply_discount(shoes, 2.0)
    

    <ipython-input-1-dd6c13736311> in apply_discount(product, discount)
          1 def apply_discount(product, discount):
          2     price = int(product['price'] * (1 - discount))
    ----> 3     assert 0 <= price <= product['price'], 'invalid price'
          4     return price


    AssertionError: invalid price


可以看到，在断言条件为假时，程序抛出了`AssertionError`异常并且打印了自定义的错误信息

## Tricks

### 不要使用断言验证数据
要注意的一个重点是，若在命令行中使用-O和-OO标识，或者修改CPython中的`PYTHONOPTIMIZE`环境变量，都会全局禁用断言。此时程序会跳过断言，所有的断言都会失效。因此使用断言来验证数据非常的危险，断言仅应该用于调试内部bug
### 存在永不失败的断言
例如这个断言
``` python
assert (1 == 2, 'msg')
```
错误的语法使用使得assert语句的第一个参数（即条件表达式）是一个元组，而非空元组总为真值，所以这个断言永远不会触发

---

# with语句和上下文管理器

with语句是Python一个非常有用的特性，有助于编写更清晰易读的代码。它能够简化一些通用的资源管理模式，最常见的就是文件操作利用这个特性：
``` python
with open('hello.txt', 'w') as f:
    f.write('hello, world!')
```
这样能够确保打开的文件描述符能够在执行完毕后被自动关闭。上面的代码可以转换为下面的代码：
``` python
f = open('hello.txt', 'w')
try:
    f.write('hello, world!')
finally:
    f.close()
```
with语句不仅让处理系统资源的代码更易读，还可以避免bug和资源泄露

## 在自定义对象中支持with
为什么open()函数能够使用with语句，是因为他们实现了上下文管理器(Context Manager)，只要在自己的对象中实现了上下文管理器，就可以和with语句一起使用。简单来说，上下文管理器就是一个简单的协议或者说接口，实现了这个协议或者接口，那么这个对象就是一个上下文管理器。而这个协议的具体内同就是在对象中实现`__enter__`方法和`__exit__`方法，当执行流程进入with语句上下文时，Python会调用`__enter__`获得资源，离开with上下文时，Python会调用`__exit__`来释放资源，下面是一个open()上下文管理器的一个简单实现


```python
class ManagedFile:
    def __init__(self, name):
        self.name = name
    
    def __enter__(self):
        self.file = open(self.name, 'w')
        return self.file
    
    def __exit__(self, *args):
        if self.file:
            self.file.close()
```


```python
with ManagedFile('hello.txt') as f:
    f.write('hello, world!')
    f.write('bye now')
```

---

# 有关下划线的命名

单下划线和双下划线在Python的变量和方法命名中非常常见，这里列举五种情况讨论他们的特殊含义，有一些是约定俗成，而有一些能够影响Python的行为
+ 前置单下划线：_var
+ 后置单下划线：var_
+ 前置双下划线：__var
+ 前后双下划线：\_\_var\_\_
+ 单下划线：_

## 前置单下划线：_var
前置单下划线只有约定意义，意思是提醒开发人员，以单下划线开头的变量或者方法只在对象的内部使用。Python与Java不同，没有语法上明确的私有和公共属性，所以使用这个约定，但是如果非要访问也是没有问题的


```python
class Test:
    def __init__(self):
        self.x = 100
        self._y = 200
```


```python
t = Test()
print(t.x, t._y)
```

    100 200


虽然在变量和方法名上前置单下划线是一个约定的使用方式，但会影响从模块中导入名称的方式，在使用通配符导入所有名称的过程中，前置单下划线的名称不会被导入，假设有一个叫my_module的模块，其中有这些代码：
``` python
# my_module.py

def external_func():
    return 1

def _internal_func():
    return 2
```
然后使用通配符导入所有的名称，可见第二个方法不会被导入


```python
from my_module import *
```


```python
external_func()
```




    1




```python
_internal_func()
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-23-3757d8ee357f> in <module>
    ----> 1 _internal_func()
    

    NameError: name '_internal_func' is not defined


通常来说，应当避免使用通配符导入名称，因为这样就不清楚当前命名空间中存在哪些名称了

## 后置单下划线：var_
后置单下划线也是约定的使用方式，通常用来绕过命名冲突


```python
def make_object(name, class):
    pass
```


      File "<ipython-input-24-88a174f47223>", line 1
        def make_object(name, class):
                                  ^
    SyntaxError: invalid syntax




```python
def make_object(name, class_):
    pass
```

## 前置双下划线：__var
与前面不同，这种命名方式会让Python解释器重写属性名称，以避免子类中的命名冲突，这也叫名称改写(name mangling)，即解释器会更改变量的名称，以便再稍后拓展类时不发生命名冲突，使用一个例子来实验：


```python
class Test():
    def __init__(self):
        self.foo = 11
        self._bar = 23
        self.__baz = 42
```

然后使用dir()函数来看一下这个类的对象的属性：


```python
dir(Test())
```




    ['_Test__baz',
     '__class__',
     '__delattr__',
     '__dict__',
     '__dir__',
     '__doc__',
     '__eq__',
     '__format__',
     '__ge__',
     '__getattribute__',
     '__gt__',
     '__hash__',
     '__init__',
     '__init_subclass__',
     '__le__',
     '__lt__',
     '__module__',
     '__ne__',
     '__new__',
     '__reduce__',
     '__reduce_ex__',
     '__repr__',
     '__setattr__',
     '__sizeof__',
     '__str__',
     '__subclasshook__',
     '__weakref__',
     '_bar',
     'foo']



可以发现，self.foo和self.\_bar属性都没有改变，而\_\_baz属性不见了，而是出现了一个\_Test\_\_baz属性，这就是解释器为这个属性改的名称

### What is dunder？
有些Python高手会使用dunder这个词，什么意思呢？因为Python代码中经常出现双下划线，所以为了简化发音，通常会将“双下划线”(double underscore)简称为dunder，比如\_\_baz读作dunderbaz，\_\_init\_\_读作dunderinit，只是一个暗号

## 前后双下划线：\_\_var\_\_

前置双下划线会使解释器改写名称，然而前后双下划线则不会使解释器改写名称。但是前后双下划线命名有特殊的用途，比如\_\_init\_\_和\_\_call\_\_这些内置函数一样，都遵循这个规则，这些前后双下划线命名的方法称为**魔法方法**

但是并没有什么魔法，不应该畏惧它


## 单下划线：_
按照约定，单下划线变量表示变量是临时的或者无关紧要的，比如在循环中不需要循环索引时，或者解包表达式时略过不需要的值


```python
for _ in range(10):
    pass
```

除了用作临时变量之外，_在大多数的Python的REPL中还是一个特殊的变量，用来表示解释器上一个计算的表达式的结果，比如:
``` python
>>> 1 + 2
3
>>> _
3
>>> list()
[]
>>> _.append(1)
>>> _.append('a')
>>> _
[1, 'a']
>>> 
```
如上所示，还可以不指定名称实时创建对象并与之交互

# 字符串格式化

字符串格式化是经常碰到的工作，记录四种字符串格式化的方式，假如需要格式化一个错误提示信息，先预先定义需要的变量


```python
errno, name = 50159747054, 'Bob'
```

## 第一种 “老旧”的方式
使用%操作符可以方便的格式化字符串，就像C语言中printf风格的函数一样，使用格式说明符占位


```python
'Hello, %s' % name
```




    'Hello, Bob'



还可以使用格式说明符对变量类型进行转换


```python
'%x' % errno
```




    'badc0ffee'



如果需要格式化多个变量，则将变量封装为一个元组


```python
'Hey %s, there is a 0x%x error!' % (name, errno)
```




    'Hey Bob, there is a 0xbadc0ffee error!'



## 第二种 “新式”的方式
使用format()函数来取代%操作符，使得语法更加规整


```python
'Hello, {}'.format(name)
```




    'Hello, Bob'




```python
'Hey {name}, there is a 0x{errno:x} error!'.format(name = name, errno = errno)
```




    'Hey Bob, there is a 0xbadc0ffee error!'



## 第三种 更fancy的方式
使用f字符串进行字面值插值的方式


```python
f'Hello {name}!'
```




    'Hello Bob!'




```python
f'Hey {name}, there is a {errno:#x} error!'
```




    'Hey Bob, there is a 0xbadc0ffee error!'



## 第四种 模板字符串
最后一种格式化技术是模板字符串，这种机制很简单但是某些情况下更加安全


```python
from string import Template

t = Template('Hey $name!')
t.substitute(name = name)
```




    'Hey Bob!'



---

第一部分笔记到此结束，第二部分正在路上......
