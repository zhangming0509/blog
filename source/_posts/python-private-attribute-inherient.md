title: python私有同名属性的继承
date: 2016-01-29 13:26:47
tags:
  - python
---

在python中以双下划线开头的属性是私有属性，不会被子类继承，然而python中的私有不是绝对的私有，可以通过`_类名__私有属性`的方式被继承，现有如下两个类, `child`和`Father`：

- 第一种情况：子类`Child`未调用父类的构造函数，但是没有定义`get`方法，只定义了一个`__value`私有属性

```python
class Father(object):

    def __init__(self):
        self.__value = 0

    def get(self):
        print 'Father get() method'
        print self.__value


class Child(Father):

    def __init__(self):
        self.__value = 100
```

子类`Child`继承了父类`Father`的`get`方法，但是这个`get`方法打印的是父类`Father`中的私有属性`__value`，子类对象创建的过程没有调用父类的构造函数，并未继承到父类的私有属性`_Father__value`，所以子类对象调用时会抛出异常`AttributeError: 'Child' object has no attribute '_Father__value'`

<!--more-->

```bash
>>> c = Child()
>>> c.get()
Father get() method
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "fc.py", line 11, in get
    print self.__value
AttributeError: 'Child' object has no attribute '_Father__value'
>>> dir(c)
['_Child__value', '__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'get']
```

- 第二种情况：子类`Child`调用了父类的构造函数，但是没有定义`get`方法，同时定义了一个`__value`私有属性

```python
class Father(object):

    def __init__(self):
        self.__value = 0

    def get(self):
        print 'Father get() method'
        print self.__value


class Child(Father):

    def __init__(self):
        super(Child, self).__init__()
        self.__value = 100
```

子类`Child`通过调用父类`Father`的构造函数继承了父类的私有属性`__value`，同时自己也定义了一个同名的私有属性`__value`，但是继承而来的`get`方法使用的依然是父类的私有属性，即：`_Father__value`，所以`c.get()`结果为`0`

```bash
>>> c = Child()
>>> c.get()
Father get() method
0
>>> dir(c)
['_Child__value', '_Father__value', '__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'get']
```

- 第三种情况：子类`Child`调用了父类的构造函数，同时定义自己的`get`方法，但是没有定义自己`__value`私有属性

```python
class Father(object):

    def __init__(self):
        self.__value = 0

    def get(self):
        print 'Father get() method'
        print self.__value


class Child(Father):

    def __init__(self):
        super(Child, self).__init__()

    def get(self):
        print 'Child get() method'
        print self.__value
```

子类`Child`实现了自己的`get`方法，希望使用自己的私有属性`__value`，因为是私有的，所以不会是继承自父类`Father`，所以即使调用了父类的构造方法继承得到的`_Father__value`在此也是无用的。

```bash
>>> c = Child()
>>> c.get()
Child get() method
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "fc.py", line 22, in get
    print self.__value
AttributeError: 'Child' object has no attribute '_Child__value'
>>> dir(c)
['_Father__value', '__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'get']
```

- 第四种情况：子类`Child`调用了父类的构造函数，同时定义了自己的`get`方法，也定义自己`__value`私有属性，

```python
class Father(object):

    def __init__(self):
        self.__value = 0

    def get(self):
        print 'Father get() method'
        print self.__value


class Child(Father):

    def __init__(self):
        super(Child, self).__init__()
        self.__value = 100

    def get(self):
        print 'Child get() method'
        print self.__value
```

子类`Child`实现了自己的`get`方法，而且定义了自己的私有属性`__value`，所以`c.get()`获取到的是子类自己的私有属性的值。

```bash
>>> c.get()
Child get() method
100
>>> dir(c)
['_Child__value', '_Father__value', '__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'get']
>>> c._Child__value
100
>>> c._Father__value
0
```

--------------------
