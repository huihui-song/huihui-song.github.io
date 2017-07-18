---
layout: post
title:  "python: Dict To Object"
date:   2017-07-18 +0800
category: code
---

# Dict To Object

## Question

实际应用中，为了方便用户书写或者说统一借口规范，在进行程序开发时通常会提供给用户对象接口，这样用户可以通过访问对象属性直接获取某些必要的值，程序显得清晰美观。

而在自定义数据时，为了方便实现以及灵活变化，开发者更倾向于定义dict数据结构，由于dict与object在属性访问方式上的不同，所以二者的转化就显得有必要。

期望实现两个自定义函数：<br/>
1. dict to object: to_object() <br/>
  假设有dict: d = {'x': 1}  <br/>
  期望在to_object方法后可以得到对象obj_d, obj_d.x == 1 <br/>
2. object to dict: to_dict() <br/>
  用法与上述相反

## Attributes of Object

在python2.6之后，新式类都推荐继承自object，我们看下一个空的object属性

```python
In [13]: dir(object)
Out[13]:
['__class__',
 '__delattr__',
 '__doc__',
 '__format__',
 '__getattribute__',
 '__hash__',
 '__init__',
 '__new__',
 '__reduce__',
 '__reduce_ex__',
 '__repr__',
 '__setattr__',
 '__sizeof__',
 '__str__',
 '__subclasshook__']
```

可见任意继承自object的对象都会具有"__setattr__"属性，这是为对象条件属性的接口，"__setattr__"属性对应的building方法为"setattr".

再看一个有自定义方法和成员的类对象的属性：

```python
In [14]: class A(object):
   ....:     x = 1
   ....:     def __init__(self):
   ....:         self.y = 2
   ....:     def foo(self):
   ....:         self.z = 3
   ....:         

In [15]: a = A()

In [16]: dir(a)
Out[16]:
['__class__',
 '__delattr__',
 '__dict__',
 '__doc__',
 '__format__',
 '__getattribute__',
 '__hash__',
 '__init__',
 '__module__',
 '__new__',
 '__reduce__',
 '__reduce_ex__',
 '__repr__',
 '__setattr__',
 '__sizeof__',
 '__str__',
 '__subclasshook__',
 '__weakref__',
 'foo',
 'x',
 'y']

In [17]: a.__dict__
Out[17]: {'y': 2}

In [18]: a.foo()

In [19]: a.__dict__
Out[19]: {'y': 2, 'z': 3}
```

由上可知，自定义对象相比object而言增加自定义方法、自定义类变量、自定义成员变量属性，其中成员变量属性只在明确调用后产生，而"__dict__"属性会包含已经产生过的自定义成员变量属性。所以如果我们直接更新"__dict__"属性应该可以更新对象具有的成员变量。

## Dict To object

### use setattr
依据上述两种思路，可以得到两种dict to object的方法：
1. setattr设置object属性

```python
In [20]: class Obj(object):
   ....:     def to_object(self, dictionary):
   ....:         for key in dicttionary:
   ....:             setattr(self, key, dictionary[key])

```

2. 更改__dict__属性间接设置object属性

```python
In [21]: class Obj(object):
   ....:     def to_object(self, **dictionary):
   ....:         self.__dict__.update(dictionary)
   ....:         

```

注：上述方法中的参数均可用**dictionary或dictionary，只是传参时方法不同罢了。

### use namedtuple

除了直接设置object的属性之外，也可以通过生成特别的namedtuple来达到目的

```python
from collection import namedtuple

Obj = namedtuple('Obj', 'x y z')
# also can be: Obj = namedtuple('Obj', ('x', 'y', 'z'))
a = Obj(x=1, y=2, z=3)
```

这样以来也得到了拥有x、y、z属性的Obj。

### multi layers dict

当dict具有多层嵌套结构时，最直接的想法是采用递归方式调用。

```python
class obj(object):
    def __init__(self, d):
        self.to_object(d)

    def to_object(self, d):
        for a, b in d.items():
            if isinstance(b, (list, tuple)):
               setattr(self, a, [obj(x) if isinstance(x, dict) else x for x in b])
            else:
               setattr(self, a, obj(b) if isinstance(b, dict) else b)
```

上述实现是在obj的__init__方法中传入参数直接调用to_object，主要是方便obj[x]生成新的object从而实现属性嵌套，如果直接设置to_object为classmethod然后调用时，需注意要在每次嵌套设置时生成新的object。

也可以采用json的loads函数中的object_hook方法实现，参考如下：

```python
from collections import namedtuple

row = '''{
    "url" : {
        "domain" : "dmyz.org"
    },
    "page_rank" : 2,
    "geography" : {
        "city" : "Beijing",
        "country" : "China"
    }
}'''

x = json.loads(row, object_hook=lambda d: namedtuple('X', d.keys())(*d.values()))

print x.url.domain
```

## Object to Dict

相对而言，这种就比较简单了，可以直接利用object的__dict__属性，

```python
def to_dict(obj):  
    pr = {}  
    for name in dir(obj):  
        value = getattr(obj, name)  
        if not.name.startswith('__') and not callable(value):  
            pr[name] = value  
    return pr  
```

## References

[dmyz: Python的dict转object](http://dmyz.org/archives/516) <br/>
[stackoverflow: Convert Python dict to object](https://stackoverflow.com/questions/1305532/convert-python-dict-to-object) <br/>
[csdn: python中 class 或对象属性转化成dict 、dict转换成对象 ](http://blog.csdn.net/chenyulancn/article/details/8203763) <br/>
