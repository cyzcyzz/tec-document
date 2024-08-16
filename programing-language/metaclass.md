---
title: "元类"
date: 2020-07-11T10:19:03+08:00
description: ""
draft: false
tags: []
categories: [python]
---

动态语言在创建类的时候，是动态创建的

```python
class Hello(object):
		def hello(self, name='world'):
		    print("hello")
```

上边是我们创建的一个类，我们可以使用

```python
>>> type(Hello)
<class 'type'>
```

我们使用type插件类的类型的时候，我们发现，他返回的是type类型的。

```python
>>> h=Hello()
>>> type(h)
<class '__main__.Hello'>
```

实例化以后，我们可以看到，h的类型是一个Hello。

实际上，当我们使用class关键字创建类的时候，背后也是调用type函数来创建处class。

元类是类的类，类定义类实例也就是类对象的行为，元类定义类本身的行为。一个类是一个元类的实例。

也就是通常我们使用`class`关键字来定义一个类，实际上是实例化了一个元类的对象。

梳理一下关系：先定义元类，接下来使用type或者class创建类，然后实例化创建的类，也就是成为一个对象了。可以说是非常的绕了。

如何定义一个简单的元类：

```python
 # 注意这里继承的是type，元类必须从type中派生
  class ListMetaclass(type)
    def __new__(cls, name, bases, sttrs):
       attrs['add'] = lambda self, vlaue: self.append(value)
        return type.__new__(cls, name, basesm, attrs)
```

如何使用自定义元类，定制创建的类？

```python
class MyList(list, metaclass=ListMetaclass):
  pass
```

当我们传入关键字参数metaclass的时候，魔术就生效了，他会告诉解释器，使用ListMetaclass.___new___() （markdowm的问题，下划线不显示），来创建，解析下几个参数，

- 1，准备创建的类的对象 
- 2，名称，
- 3，类继承的父类集合，
- 4，类的方法集合

在一般的使用场景中，我们一般不会用到这种方式来自己构建一个元类，但是在其他的场景中，使用元类可能会更简单，如ORM。一行映射为一个对象，一个类映射为一个表。

```python
class User(Model):
    # 定义类的属性到列的映射：
    id = IntegerField('id')
    name = StringField('username')
    email = StringField('email')
    password = StringField('password')

# 创建一个实例：
u = User(id=12345, name='Michael', email='test@orm.org', password='my-pwd')
# 保存到数据库：
u.save()
```

上面的属性类型是有框架提供，save方法是由元类提供

```python
class Fied(object):
  def.__init__(self, name, type):
    self.name = name
    self.type = type
    
   def __str__(self):
    return '<%s:%s>' % (self.__class__.__name__, self.name)
```

基于父类定义各种的常见类型：

```python
class StringFied(Field):
  def __init__(self, name):
    super(StringField, self).__init__(name, 'varchar(100)')
```

