# Python



## 多线程, 多进程以及协程

### 进程 X 线程 X 协程

__进程__: 所谓进程就是一个正在执行中的程序. 每个进程有自己的内存,数据地址等.

__线程__: 线程则属于进程的一部分,是真实的执行者. 一个进程包含一个主线程, 也可以有更多的子线程. 

__协程__: 协程则是子线程下的由程序自身控制的并发操作

### 各自的特性

__进程__: 资源隔离, 且可以充分的发挥多核的计算能力. 但是也同样因为资源隔离导致变量不能共享, 进程间交互需要打通特定的渠道.

__线程__: 资源共享, 但是由于存在全局解释器锁(GIL), python中的多线程并不能发挥出多核的性能. 所以针对计算密集型的场景, 多线程并没有什么收益. 

__协程__: 与线程类似, 但是优于线程, 由于是程序的逻辑进行切换控制不需要外部中断等待CPU调度等, 所以协程的性能是要优于线程的, 而且由于本身是单线程执行, 也不需要担心同时操作一个参数时还需要通过加锁来进行处理. 在python中使用生成器来实现协程. 




## 垃圾回收机制

### 引用计数

python主要使用引用计数来实现垃圾回收. 每一个python的对象, 内部有一个引用计数.  

下列情形会使计数增加1

- 对象创建
- 对象被引用
- 对象作为参数被传入函数中
- 对象作为一个元素被存储在容器中 (列表, 字典, 自定义类对象,  元组等)

下列情形会使计数减少1

- 对象别名被显式的销毁 del
- 对象别名被赋予新的对象
- 一个对象离开了他的作用域
- 所在容器销毁, 或被从容器中删除 



### 标记 - 清除

标记 - 清除 主要是为了处理引用计数无法解决的嵌套问题. 简单对象像int string等不需要这样. 仅针对容器对象. 

这一个算法主要分为两个步骤

1. 遍历所有的对象, 如果还有对象引用它, 就将该对象标记为可达的
2. 再次遍历对象, 如果发现某个对象没有被标记为可达, 那就将其回收

嵌套的对象是使用双端链表进行维护. 

gc开始后, 除了原本的计数器, 还会有一个gc的计数器, 遍历所有容器对象的时候, 会将gc的计数器都减一. 然后如果计数不为0, 则将该对象标记为可达, 然后如果有一个对象是可达的, 那么从该节点出发所有的节点都被标记为可达. 如此再遍历一遍, 如果还有对象没有标记为可达, 则将其回收. 

需要注意的是, 上述的垃圾回收阶段, 会暂停整个应用程序, 等待标记清楚结束之后才会恢复应用程序的运行. 




### 分代回收

由于在检测回收嵌套对象时, 整个应用程序会被暂停, 为了减少应用程序被占用的时间. Python通过分代回收来提高垃圾回收的效率. 

核心思想是: 存在越久的对象就越不可能被回收. 

Python在gc的时候, 把对象分为3个世代: 0, 1, 2. 

世代: 新生对象在世代0, 如果这个对象在一轮gc检测中活了下来. 那么它就会被移到世代1,会经受较少的检测频率, 如果下一轮依旧活了下来, 那么就会被移到世代2, 接受更少的检测. 

触发: 当某一个世代中被分配对象与释放对象的差值到达阈值的时候, 就会触发当前世代的检测, 并且比当前世代年轻的世代也会触发检测. 



### 总结

总体来说，在Python中，主要通过引用计数进行垃圾回收；通过 “标记-清除” 解决容器对象可能产生的循环引用问题；通过 “分代回收” 以空间换时间的方法提高垃圾回收效率。



## 生成器, 迭代器的概念以及含义

### 迭代器

迭代器是可以被迭代操作（for循环）的对象，对应的可以简单理解为如果实现了`__iter__`或者`__next__`方法就可以被认为是一个迭代器。迭代器可以像列表一样迭代获取其中的每一个元素。但是与list表相比，不会把所有内容一次性加载到内存中，而是延时加载。

### 生成器

生成器其实是一种特殊的迭代器，但是不需要像迭代器一样实现`__iter__`和`__next__`方法，只需要使用关键字yield就可以。

Python有两种方式提供生成器对象：

1. 生成器函数：使用yield返回而不是return，函数会返回一个生成器对象，每次返回一个结果，返回后操作会被挂起直至下一次调用。
2. 生成器表达式：类似列表推导式，形如 `(x + 1 for x in range(30))`，这样不会返回一个列表而是一个生成器对象。

协程就是通过生成器实现的。



## Python LEGB规则

### 命名空间

命名空间是Python对变量名的分组。

不同分组的同名变量被视为完全不同的变量，换句话说不同命名空间的变量可以重名。``

Python的命名空间是一个字典，字典内保存了变量名称与对象之间的映射关系



### LEGB规则

LEGB含义解释：

- L-Local(function)；函数内的名字空间
- E-Enclosing function locals；外部嵌套函数的名字空间(例如closure)
- G-Global(module)；函数定义所在模块（文件）的名字空间
- B-Builtin(Python)；Python内置模块的名字空间



Python有多个命名空间，因此，需要有规则来规定，按照怎样的顺序来查找命名空间，**LEGB就是用来规定命名空间查找顺序的规则**。

LEGB规定了查找一个名称的顺序为：local-->enclosing function locals-->global-->builtin



## Python面向对象编程

- **类(Class):** 用来描述具有相同的属性和方法的对象的集合。它定义了该集合中每个对象所共有的属性和方法。对象是类的实例。
- **类变量：**类变量在整个实例化的对象中是公用的。类变量定义在类中且在函数体之外。类变量通常不作为实例变量使用。
- **数据成员：**类变量或者实例变量, 用于处理类及其实例对象的相关的数据。
- **方法重写：**如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写。
- **局部变量：**定义在方法中的变量，只作用于当前实例的类。
- **实例变量：**在类的声明中，属性是用变量来表示的。这种变量就称为实例变量，是在类声明的内部但是在类的其他成员方法之外声明的。
- **继承：**即一个派生类（derived class）继承基类（base class）的字段和方法。继承也允许把一个派生类的对象作为一个基类对象对待。例如，有这样一个设计：一个Dog类型的对象派生自Animal类，这是模拟"是一个（is-a）"关系（例图，Dog是一个Animal）。
- **实例化：**创建一个类的实例，类的具体对象。
- **方法：**类中定义的函数。
- **对象：**通过类定义的数据结构实例。对象包括两个数据成员（类变量和实例变量）和方法。



### classmethod

类方法不需要将类实例化即可调用，第一个入参 cls 为类本身。



### staticmethod

没有任何依赖关系，只是恰好放在类中。





## 装饰器实现原理

装饰器本身是Python中一个非常重要的概念，可以动态的为一个函数或者类增加新的动作。

一种语言可以使用装饰器的前提是，可以将方法作为传参进行函数的调用。

Python中使用`@`语法糖来实现装饰器。

实际上@的实现可以理解为

```python
func = decorator(func)
```



### 闭包与自由变量

闭包就是引用了自由变量的嵌套函数。

所谓自由变量就是即使离开了创造他的环境，仍然可以存在以及生效。

```python
def foo():
    a = []

    def aaa():
        a.append(2)
        print(a)
    return aaa
  	


x = foo()
x()
```

可以调用`__closure_`查看函数是否存在自由变量，如果没有，那就是一个简单嵌套函数。



__基础装饰器__

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        print('xxxxxxx')
        result = func(*args,  **kwargs)
        print('xxxxxxx')
        return result
    return wrapper
  
@decorator
def test(a, b, c):
    pass
```



__带参数的装饰器__

```python
def decorator_with_params(a, b, c=None):
    def decorator(func):
        def wrapper(*args, **kwargs):
            print('xxxxxxx')
            result = func(*args,  **kwargs)
            print('xxxxxxx')
            print(a)
            print(b)
            print(c)
            return result
        return wrapper
    return foo
 
@decorator_with_params(1, 2)
def foo(x):
		pass

```



__使用类作为装饰器__

```python
class Decorator(object):
  	def __init__(self, func):
      	self.func = func
        
    def __call__(self, *args, **kwargs):
      	print('aaaaaa')
        self.func(*args, **kwargs)
        print('bbbbbb')

@Decorator
def foo():
  	pass
        
```



__使用对象作为装饰器__

```python
class Decorator(object):
  	def __init__(self, arg1, arg2):
      	self.arg1 = arg1
        self.arg2 = arg2
        
    def __call__(self, func):
      	def wrapper(*args, **kwargs):
            print('aaaaaa')
            self.func(*args, **kwargs)
            print('bbbbbb')
        return wrapper

@Decorator("AAA", "BBB")
def foo():
  	pass
        
```



## Python2 与 Python3的区别

字符串类型

python中有两种字符类型：字节字符串和文本字符串。

版本 	python2 	python3 
字节字符串 	str 	bytes 
文本字符串 	Unicode 	str 

2.默认字符
python2中默认的字符串类型默认是ASCII，python3中默认的字符串类型是Unicode。
3.print

python2中，print是个特殊语句，python3中print是函数。

python2：print 'hello word!'

python3：print('hello word!',file=sys.stderr)

4.除法/

python2中/的结果是整型，python3中是浮点类型。

5.导入

python2中的包导入顺序：标准库—相对倒入（即当前目录）—绝对导入（sys.path）

python3中的包导入顺序：标准库—绝对导入（如果想要相对导入，使用from .moudel）

6.类

python2中默认类是旧式类，需要显式继承新式类（object）来创建新式类。

python3中完全移除旧式类，所有类都是新式类，但仍可显式继承object类。

7.元类声明

python2中声明元类：__metaclass__ = MetaClass

python3中声明元类：class newclass(metaclass=MetaClass)：pass

8.异常

python2中引发异常：raise ValueError,'Invalid value'

python3中引发异常：raise ValueError('Invalid value')——在python2中也生效

9.处理异常

python2中处理异常：

try:

raise ValueError,'Invalid value'

except ValueError,e:

pass

python3中处理异常：

try:

raise ValueError,'Invalid value'

except ValueError as e:#在python2中也生效


pass


python2中异常链会丢失原始异常信息，即：处理B异常时引发了A异常，B异常信息会丢失。

python3中将原始异常信息赋值给__context__属性。并且可以显式指定一个异常作为另一个异常的子句：raise DatabaseError() from IOError()

10.字典

python2中的dict类中的keys、values和items均返回list对象，iterkeys、itervalues和iteritems返回生成器对象。

python3中移除了list、只返回一个生成器的对象，只保留视图（生成器），但方法名为：keys、values和items。

11.模块合并

python2中的StringIO和cStringIO合并为python3中的io

python2中的pickle和cPickle合并为python3中的pickle。

python2中的urllib、urllib2和urlparse合并为python3中的urllib

12.重命名模块

python3 	python2 
Configparser 	ConfigParser 
filter 	itertools.ifilter 
input 	raw_input 
map 	itertools.imap 
range 	xrange 
functools.reduce 	reduce 
socketserver 	SocketServer 
zip 	itertools.izip 



## Python Web框架对比

Django VS flask

Django重量级WebServer集成了几乎所有需要考虑到的功能，内置了大量的常用工具，包含了用户管理，权限控制，分页缓存以及与数据库的连接等等。

flask轻量级Web Server，本身只提供了核心的功能，但是灵活而易于拓展。



## Python 常用数据结构

###  一、字典、映射和散列表

在Python 中，字典是核心数据结构。字典可以存储任意数量的对象，每个对象都由唯一的字典键标识。

字典通常也被称为映射、散列表、查找表或关联数组。字典能够高效查找、插入和删除任何与给定键关联的对象。

**1.dict——首选字典实现**

由于字典非常重要，因此Python 直接在语言核心中实现了一个稳健的字典：dict 数据类型。

Python 还提供了一些有用的“语法糖”来处理程序中的字典。例如，用花括号字典表达式语法和字典解析式能够方便地创建新的字典对象：

```text
phonebook = {
    'bob': 7387,
    'alice': 3719,
    'jack': 7052,
}

squares = {x: x * x for x in range(6)}

>>> phonebook['alice']
3719

>>> squares
{0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

关于哪些对象可以作为字典键，有一些限制。

Python 的字典由可散列类型的键来索引。可散列对象具有在其生命周期中永远不会改变的散列值（参见__hash__），并且可以与其他对象进行比较（参见__eq__）。另外，相等的可散列对象，其散列值必然相同。

像字符串和数这样的不可变类型是可散列的，它们可以很好地用作字典键。元组对象也可以用作字典键，但这些元组本身必须只包含可散列类型。

Python 的内置字典实现可以应对大多数情况。字典是高度优化的，并且是Python 语言的基石，例如栈帧中的类属性和变量都存储在字典中。

Python 字典基于经过充分测试和精心调整过的散列表实现，提供了符合期望的性能特征。一般情况下，用于查找、插入、更新和删除操作的时间复杂度都为O(1)。

大部分情况下，应该使用Python 自带的标准字典实现。但是也存在专门的第三方字典实现，例如[跳跃表](https://link.zhihu.com/?target=http%3A//www.cl.cam.ac.uk/teaching/0506/Algorithms/skiplists.pdf)或基于B 树的字典。

除了通用的dict 对象外，Python 的标准库还包含许多特殊的字典实现。它们都基于内置的字典类，基本性能特征相同，但添加了其他一些便利特性。

下面来逐个了解一下。

**2.collections.OrderedDict——能记住键的插入顺序**

collections.OrderedDict是特殊的dict 子类，该类型会记录添加到其中的键的插入顺序。

尽管在CPython 3.6 及更高版本中，标准的字典实现也能保留键的插入顺序，但这只是CPython 实现的一个副作用，直到Python 3.7 才将这种特性固定下来了。因此，如果在自己的工作中很需要用到键顺序，最好明确使用OrderedDict 类。

顺便说一句，OrderedDict 不是内置的核心语言部分，因此必须从标准库中的collections模块导入。

```text
>>> import collections
>>> d = collections.OrderedDict(one=1, two=2, three=3)

>>> d
OrderedDict([('one', 1), ('two', 2), ('three', 3)])

>>> d['four'] = 4
>>> d
OrderedDict([('one', 1), ('two', 2),
            ('three', 3), ('four', 4)])
>>> d.keys()
odict_keys(['one', 'two', 'three', 'four'])
```

**3.collections.defaultdict——为缺失的键返回默认值**

defaultdict 是另一个dict 子类，其构造函数接受一个可调用对象，查找时如果找不到给定的键，就返回这个可调用对象。

与使用get()方法或在普通字典中捕获KeyError 异常相比，这种方式的代码较少，并能清晰地表达出程序员的意图。

```text
>>> from collections import defaultdict
>>> dd = defaultdict(list)

# 访问缺失的键就会用默认工厂方法创建它并将其初始化
# 在本例中工厂方法为list()：
>>> dd['dogs'].append('Rufus')
>>> dd['dogs'].append('Kathrin')
>>> dd['dogs'].append('Mr Sniffles')

>>> dd['dogs']
['Rufus', 'Kathrin', 'Mr Sniffles']
```

**4.collections.ChainMap——搜索多个字典**

collections.ChainMap 数据结构将多个字典分组到一个映射中，在查找时逐个搜索底层映射，直到找到一个符合条件的键。对ChainMap 进行插入、更新和删除操作，只会作用于其中的第一个字典。

```text
>>> from collections import ChainMap
>>> dict1 = {'one': 1, 'two': 2}
>>> dict2 = {'three': 3, 'four': 4}
>>> chain = ChainMap(dict1, dict2)

>>> chain
ChainMap({'one': 1, 'two': 2}, {'three': 3, 'four': 4})

# ChainMap 在内部从左到右逐个搜索，
# 直到找到对应的键或全部搜索完毕：
>>> chain['three']
3
>>> chain['one']
1
>>> chain['missing']
KeyError: 'missing'
```

**5.types.MappingProxyType——用于创建只读字典**

MappingProxyType 封装了标准的字典，为封装的字典数据提供只读视图。该类添加自Python 3.3，用来创建字典不可变的代理版本。

举例来说，如果希望返回一个字典来表示类或模块的内部状态，同时禁止向该对象写入内容，此时MappingProxyType 就能派上用场。使用MappingProxyType 无须创建完整的字典副本。

```text
>>> from types import MappingProxyType
>>> writable = {'one': 1, 'two': 2}
>>> read_only = MappingProxyType(writable)

# 代理是只读的：
>>> read_only['one']
1
>>> read_only['one'] = 23
TypeError:
"'mappingproxy' object does not support item assignment"

# 更新原字典也会影响到代理：
>>> writable['one'] = 42
>>> read_only
mappingproxy({'one': 42, 'two': 2})
```

**Python 中的字典：总结**

上面列出的所有Python 字典实现都是内置于Python 标准库中的有效实现。一般情况下，建议在自己的程序中使用内置的dict 数据类型。这是优化过的散列表实现，功能多且已被直接内置到了核心语言中。

如果你有内置dict 无法满足的特殊需求，那么建议使用文章中列出的其他数据类型。

虽然前面列出的其他字典实现均可用，但大多数情况下都应该使用Python 内置的标准dict，这样其他开发者在维护你的代码时就会轻松一点。

**关键要点**

- 字典是Python 中的核心数据结构。
- 大部分情况下，内置的dict 类型就足够了。
- Python 标准库提供了用于满足特殊需求的实现，比如只读字典或有序字典。

### 二、数组数据结构

**1.列表——可变动态数组**

列表是Python 语言核心的一部分。虽然名字叫列表，但它实际上是以动态数组实现的。这意味着列表能够添加或删除元素，还能分配或释放内存来自动调整存储空间。

Python 列表可以包含任意元素，因为Python 中一切皆为对象，连函数也是对象。因此，不同的数据类型可以混合存储在一个列表中。

这个功能很强大，但缺点是同时支持多种数据类型会导致数据存储得不是很紧凑。因此整个结构占据了更多的空间。

```text
>>> arr = ['one', 'two', 'three']
>>> arr[0]
'one'

# 列表拥有不错的__repr__方法：
>>> arr
['one', 'two', 'three']

# 列表是可变的：
>>> arr[1] = 'hello'
>>> arr
['one', 'hello', 'three']

>>> del arr[1]
>>> arr
['one', 'three']

# 列表可以含有任意类型的数据：
>>> arr.append(23)
>>> arr
['one', 'three', 23]
```

**2.元组——不可变容器**

与列表一样，元组也是Python 语言核心的一部分。与列表不同的是，Python 的元组对象是不可变的。这意味着不能动态添加或删除元素，元组中的所有元素都必须在创建时定义。

就像列表一样，元组可以包含任意数据类型的元素。这具有很强的灵活性，但也意味着数据的打包密度要比固定类型的数组小。

```text
>>> arr = 'one', 'two', 'three'
>>> arr[0]
'one'

# 元组拥有不错的__repr__方法：
>>> arr
('one', 'two', 'three')

# 元组是可变的
>>> arr[1] = 'hello'
TypeError:
"'tuple' object does not support item assignment"

>>> del arr[1]
TypeError:
"'tuple' object doesn't support item deletion"

# 元组可以持有任意类型的数据：
#（添加元素会创建新元组）
>>> arr + (23,)
('one', 'two', 'three', 23)
```

**3.array.array——基本类型数组**

Python 的array 模块占用的空间较少，用于存储C 语言风格的基本数据类型（如字节、32位整数，以及浮点数等）。

使用array.array 类创建的数组是可变的，行为与列表类似。但有一个重要的区别：这种数组是单一数据类型的“类型数组”。

由于这个限制，含有多个元素的array.array 对象比列表和元组节省空间。存储在其中的元素紧密排列，因此适合存储许多相同类型的元素。

此外，数组中有许多普通列表中也含有的方法，使用方式也相同，无须对应用程序代码进行其他更改。

```text
>>> import array
>>> arr = array.array('f', (1.0, 1.5, 2.0, 2.5))
>>> arr[1]
1.5
# 数组拥有不错的__repr__方法：
>>> arr
array('f', [1.0, 1.5, 2.0, 2.5])

# 数组是可变的：
>>> arr[1] = 23.0
>>> arr
array('f', [1.0, 23.0, 2.0, 2.5])

>>> del arr[1]
>>> arr
array('f', [1.0, 2.0, 2.5])

>>> arr.append(42.0)
>>> arr
array('f', [1.0, 2.0, 2.5, 42.0])

# 数组中元素类型是固定的：
>>> arr[1] = 'hello'
TypeError: "must be real number, not str"
```

**4.str——含有Unicode 字符的不可变数组**

Python 3.x 使用str 对象将文本数据存储为不可变的Unicode 字符序列。实际上，这意味着str 是不可变的字符数组。说来也怪，str 也是一种递归的数据结构，字符串中的每个字符都是长度为1 的str 对象。

由于字符串对象专注于单一数据类型，元组排列紧密，因此很节省空间，适合用来存储Unicode 文本。因为字符串在Python 中是不可变的，所以修改字符串需要创建一个改动副本。最接近“可变字符串”概念的是存储单个字符的列表。

```text
>>> arr = 'abcd'
>>> arr[1]
'b'

>>> arr
'abcd'

# 字符串是可变的：
>>> arr[1] = 'e'
TypeError:
"'str' object does not support item assignment"

>>> del arr[1]
TypeError:
"'str' object doesn't support item deletion"

# 字符串可以解包到列表中，从而得到可变版本：
>>> list('abcd')
['a', 'b', 'c', 'd']
>>> ''.join(list('abcd'))
'abcd'

# 字符串是递归型数据类型：
>>> type('abc')
"<class 'str'>"
>>> type('abc'[0])
"<class 'str'>"
```

**5.bytes——含有单字节的不可变数组**

bytes 对象是单字节的不可变序列，单字节为0～255（含）范围内的整数。从概念上讲，bytes 与str 对象类似，可认为是不可变的字节数组。

与字符串一样，也有专门用于创建bytes 对象的字面语法，bytes 也很节省空间。bytes对象是不可变的，但与字符串不同，还有一个名为bytearray 的专用“可变字节数组”数据类型，bytes 可以解包到bytearray 中。后面会介绍更多关于bytearray 的内容。

```text
>>> arr = bytes((0, 1, 2, 3))
>>> arr[1]
1

# bytes 有自己的语法：
>>> arr
b'\x00\x01\x02\x03'
>>> arr = b'\x00\x01\x02\x03'

# bytes 必须位于0～255：
>>> bytes((0, 300))
ValueError: "bytes must be in range(0, 256)"

# bytes 是不可变的：
>>> arr[1] = 23
TypeError:
"'bytes' object does not support item assignment"

>>> del arr[1]
TypeError:
"'bytes' object doesn't support item deletion"
```

**6.bytearray——含有单字节的可变数组**

bytearray 类型是可变整数序列，包含的整数范围在0～255（含）。bytearray 与bytes对象关系密切，主要区别在于bytearray 可以自由修改，如覆盖、删除现有元素和添加新元素，此时bytearray 对象将相应地增长和缩小。

bytearray 数可以转换回不可变的bytes 对象，但是这需要复制所存储的数据，是耗时为O(n)的慢操作。

```text
>>> arr = bytearray((0, 1, 2, 3))
>>> arr[1]
1

# bytearray 的repr：
>>> arr
bytearray(b'\x00\x01\x02\x03')

# bytearray 是可变的：
>>> arr[1] = 23
>>> arr
bytearray(b'\x00\x17\x02\x03')

>>> arr[1]
23

# bytearray 可以增长或缩小：
>>> del arr[1]
>>> arr
bytearray(b'\x00\x02\x03')

>>> arr.append(42)
>>> arr
bytearray(b'\x00\x02\x03*')

# bytearray 只能持有byte，即位于0～255 范围内的整数
>>> arr[1] = 'hello'
TypeError: "an integer is required"

>>> arr[1] = 300
ValueError: "byte must be in range(0, 256)"

# bytearray 可以转换回byte 对象，此过程会复制数据：
>>> bytes(arr)
b'\x00\x02\x03*'
```

**7.关键要点**

Python 中有多种内置数据结构可用来实现数组，上面只专注位于标准库中和核心语言特性中的数据结构。

如果不想局限于Python 标准库，那么从NumPy 这样的第三方软件包中可找到为科学计算和数据科学提供的许多快速数组实现。

对于Python 中包含的数组数据结构，选择顺序可归结如下。

**如果需要存储任意对象，且其中可能含有混合数据类型**，那么可以选择使用列表或元组，前者可变后者不可变。

**如果存储数值（整数或浮点数）数据并要求排列紧密且注重性能**，那么先尝试array.array，看能否满足要求。另外可尝试准库之外的软件包，如NumPy 或Pandas。

**如果有需要用Unicode 字符表示的文本数据**，那么可以使用Python 内置的str。如果需要用到“可变字符串”，则请使用字符列表。

**如果想存储一个连续的字节块**，不可变的请使用bytes，可变的请使用bytearray。

总之，在大多数情况下首先应尝试列表。如果在性能或存储空间上有问题，再选择其他专门的数据类型。一般像列表这样通用的数组型数据结构已经能同时兼顾开发速度和编程便利性的要求了。

强烈建议在初期使用通用数据格式，不要试图在一开始就榨干所有性能。



## 四、集合和多重集合

下面我们将用标准库中的内置数据类型和类在Python 中实现可变集合、不可变集合和多重集合（背包）数据结构。首先来快速回顾一下集合数据结构。

集合含有一组不含重复元素的无序对象。集合可用来快速检查元素的包含性，插入或删除值，计算两个集合的并集或交集。

在“合理”的集合实现中，成员检查预计耗时为O(1)。并集、交集、差集和子集操作应平均耗时为O(n)。Python 标准库中的集合实现都具有这些性能指标。

与字典一样，集合在Python 中也得到了特殊对待，有语法糖能够方便地创建集合。例如，花括号集合表达式语法和集合解析式能够方便地定义新的集合实例：

```text
vowels = {'a', 'e', 'i', 'o', 'u'}
squares = {x * x for x in range(10)}
```

但要小心，创建空集时需要调用set()构造函数。空花括号{}有歧义，会创建一个空字典。

Python 及其标准库提供了几个集合实现，让我们看看。

**1.set——首选集合实现**

set 是Python 中的内置集合实现。set 类型是可变的，能够动态插入和删除元素。

Python 的集合由dict 数据类型支持，具有相同的性能特征。所有可散列的对象都可以存储在集合中。

```text
>>> vowels = {'a', 'e', 'i', 'o', 'u'}
>>> 'e' in vowels
True

>>> letters = set('alice')
>>> letters.intersection(vowels)
{'a', 'e', 'i'}

>>> vowels.add('x')
>>> vowels
{'i', 'a', 'u', 'o', 'x', 'e'}

>>> len(vowels)
6
```

**2.frozenset——不可变集合**

frozenset 类实现了不可变版的集合，即在构造后无法更改。不可变集合是静态的，只能查询其中的元素（无法插入或删除）。因为不可变集合是静态的且可散列的，所以可以用作字典的键，也可以放置在另一个集合中，普通可变的set 对象做不到这一点。

```text
>>> vowels = frozenset({'a', 'e', 'i', 'o', 'u'})
>>> vowels.add('p')
AttributeError:
"'frozenset' object has no attribute 'add'"

# 不可变集合是可散列的，可用作字典的键
>>> d = { frozenset({1, 2, 3}): 'hello' }
>>> d[frozenset({1, 2, 3})]
'hello'
```

**3.collections.Counter——多重集合**

Python 标准库中的collections.Counter 类实现了多重集合（也称背包，bag）类型，该类型允许在集合中多次出现同一个元素。

如果既要检查元素是否为集合的一部分，又要记录元素在集合中出现的次数，那么就需要用到这个类型。

```text
>>> from collections import Counter
>>> inventory = Counter()

>>> loot = {'sword': 1, 'bread': 3}
>>> inventory.update(loot)
>>> inventory
Counter({'bread': 3, 'sword': 1})

>>> more_loot = {'sword': 1, 'apple': 1}
>>> inventory.update(more_loot)
>>> inventory
Counter({'bread': 3, 'sword': 2, 'apple': 1})
```

Counter 类有一点要注意，在计算Counter 对象中元素的数量时需要小心。调用len()返回的是多重集合中唯一元素的数量，而想获取元素的总数需要使用sum 函数：

```text
>>> len(inventory)
3 # 唯一元素的个数

>>> sum(inventory.values())
6 # 元素总数
```

**4.关键要点**

- 集合是Python 及其标准库中含有的另一种有用且常用的数据结构。
- 查找可变集合时可使用内置的set 类型。
- frozenset 对象可散列且可用作字典和集合的键。
- collections.Counter 实现了多重集合或“背包”类型的数据。

## 五、栈（后进先出）

栈是含有一组对象的容器，支持快速后进先出（LIFO）的插入和删除操作。与列表或数组不同，栈通常不允许随机访问所包含的对象。插入和删除操作通常称为入栈（push）和出栈（pop）。

现实世界中与栈数据结构相似的是一叠盘子。

> 新盘子会添加到栈的顶部。由于这些盘子非常宝贵且很重，所以只能移动最上面的盘子（后进先出）。要到达栈中位置较低的盘子，必须逐一移除最顶端的盘子。

栈和队列相似，都是线性的元素集合，但元素的访问顺序不同。

从队列删除元素时，移除的是最先添加的项（先进先出，FIFO）；而栈是移除最近添加的项（后进先出，LIFO）。

在性能方面，合理的栈实现在插入和删除操作的预期耗时是O(1)。

栈在算法中有广泛的应用，比如用于语言解析和运行时的内存管理（“调用栈”）。树或图数据结构上的深度优先搜索（DFS）是简短而美丽的算法，其中就用到了栈。

Python 中有几种栈实现，每个实现的特性略有不同。下面来分别介绍并比较各自的特性。

**1.列表——简单的内置栈**

Python 的内置列表类型能在正常的O(1)时间内完成入栈和出栈操作，因此适合作为栈数据结构。

Python 的列表在内部以动态数组实现，这意味着在添加或删除时，列表偶尔需要调整元素的存储空间大小。列表会预先分配一些后备存储空间，因此并非每个入栈或出栈操作都需要调整大小，所以这些操作的均摊时间复杂度为O(1)。

这么做的缺点是列表的性能不如基于链表的实现（如collections.deque，下面会介绍），后者能为插入和删除操作提供稳定的O(1)时间复杂度。另一方面，列表能在O(1)时间快速随机访问堆栈上的元素，这能带来额外的好处。

使用列表作为堆栈应注意下面几个重要的性能问题。

为了获得O(1)的插入和删除性能，必须使用append()方法将新项添加到列表的末尾，删除时也要使用pop()从末尾删除。为了获得最佳性能，基于Python 列表的栈应该向高索引增长并向低索引缩小。

从列表前部添加和删除元素很慢，耗时为O(n)，因为这种情况下必须移动现有元素来为新元素腾出空间。这是一个性能反模式，应尽可能避免。

```text
>>> s = []
>>> s.append('eat')
>>> s.append('sleep')
>>> s.append('code')

>>> s
['eat', 'sleep', 'code']

>>> s.pop()
'code'
>>> s.pop()
'sleep'
>>> s.pop()
'eat'

>>> s.pop()
IndexError: "pop from empty list"
```

**2.collections.deque——快速且稳健的栈**

deque 类实现了一个双端队列，支持在O(1)时间（非均摊）从两端添加和移除元素。因为双端队列支持从两端添加和删除元素，所以既可以作为队列也可以作为栈。

Python 的deque 对象以双向链表实现，这为插入和删除元素提供了出色且一致的性能，但是随机访问位于栈中间元素的性能很差，耗时为O(n)。

总之，如果想在Python 的标准库中寻找一个具有链表性能特征的栈数据结构实现，那么collections.deque 是不错的选择。

```text
>>> from collections import deque
>>> s = deque()
>>> s.append('eat')
>>> s.append('sleep')
>>> s.append('code')

>>> s
deque(['eat', 'sleep', 'code'])

>>> s.pop()
'code'
>>> s.pop()
'sleep'
>>> s.pop()
'eat'

>>> s.pop()
IndexError: "pop from an empty deque"
```

**3.queue.LifoQueue——为并行计算提供锁语义**

queue.LifoQueue 这个位于Python 标准库中的栈实现是同步的，提供了锁语义来支持多个并发的生产者和消费者。

除了LifoQueue 之外，queue 模块还包含其他几个类，都实现了用于并行计算的多生产者/多用户队列。

在不同情况下，锁语义即可能会带来帮助，也可能会导致不必要的开销。在后面这种情况下，最好使用list 或deque 作为通用栈。

```text
>>> from queue import LifoQueue
>>> s = LifoQueue()
>>> s.put('eat')
>>> s.put('sleep')
>>> s.put('code')

>>> s
<queue.LifoQueue object at 0x108298dd8>

>>> s.get()
'code'
>>> s.get()
'sleep'
>>> s.get()
'eat'

>>> s.get_nowait()
queue.Empty

>>> s.get()
# 阻塞，永远停在这里……
```

**4.比较Python 中各个栈的实现**

从上面可以看出，Python 中有多种栈数据结构的实现，各自的特性稍有区别，在性能和用途上也各有优劣。

如果不寻求并行处理支持（或者不想手动处理上锁和解锁），可选择内置列表类型或collections.deque。两者背后使用的数据结构和总体易用性有所不同。

- 列表底层是动态数组，因此适用于快速随机访问，但在添加或删除元素时偶尔需要调整大小。列表会预先分配一些备用存储空间，因此不是每个入栈或出栈操作都需要调整大小，这些操作的均摊时间复杂度为O(1)。但需要小心，只能用append()和pop()从“右侧”插入和删除元素，否则性能会下降为O(n)。
- collections.deque 底层是双向链表，为从两端的添加和删除操作进行了优化，为这些操作提供了一致的O(1)性能。collections.deque 不仅性能稳定，而且便于使用，不必担心在“错误的一端”添加或删除项。

总之，我认为collections.deque 是在Python 中实现栈（LIFO 队列）的绝佳选择。

**5.关键要点**

- Python 中有几个栈实现，每种实现的性能和使用特性略有 不同。
- collections.deque 提供安全且快速的通用栈实现。
- 内置列表类型可以作为栈使用，但要小心只能使用append()和pop()来添加和删除项，以避免性能下降。

## 六、队列（先进先出）

下面我们将介绍仅使用 Python 标准库中的内置数据类型和类来实现 FIFO 队列数据结构，首先来 回顾一下什么是队列。

队列是含有一组对象的容器，支持快速插入和删除的先进先出语义。插入和删除操作有时称为入队（enqueue）和出队（dequeue）。与列表或数组不同，队列通常不允许随机访问所包含的对象。

来看一个先进先出队列在现实中的类比。

> 想象在 PyCon 注册的第一天，一些 Python 高手等着领取会议徽章。新到的人依次 进入会场并排队领取徽章，队列后面会有其他人继续排队。移除动作发生在队列前端， 因为开发者领取徽章和会议礼品袋后就离开了。

另一种记住队列数据结构特征的方法是将其视为管道。

> 新元素（水分子、乒乓球等）从管道一端移向另一端并在那里被移除。当元素在队列中（想象成位于一根坚固的金属管中）时是无法接触的。唯一能够与队列中元素交互的方法是在管道后端添加新元素（入队）或在管道前端删除元素（出队）。

队列与栈类似，但删除元素的方式不同。

队列删除的是最先添加的项（先进先出），而栈删除的是最近添加的项（后进先出）。

在性能方面，实现合理的队列在插入和删除方面的操作预计耗时为 O(1)。插入和删除是队列 上的两个主要操作，在正确的实现中应该很快。

队列在算法中有广泛的应用，经常用于解决调度和并行编程问题。在树或图数据结构上进行 宽度优先搜索（BFS）是一种简短而美丽的算法，其中就用到了队列。

调度算法通常在内部使用优先级队列。这些是特化的队列，其中元素的顺序不是基于插入时 间，而是基于优先级。队列根据元素的键计算到每个元素的优先级。后面会详细介绍优先级队列以及它们在 Python 中的实现方式。

不过普通队列无法重新排列所包含的元素。就像在管道示例中一样，元素输入和输出的顺序 完全一致。 Python 中实现了几个队列，每种实现的特征略有不同，下面就来看看。

**1.列表——非常慢的队列**

普通列表可以作为队列，但从性能角度来看并不理想。由于在起始位置插入或删除元素需要将所有其他元素都移动一个位置，因此需要的时间为O(n)。

因此不推荐在Python 中凑合用列表作为队列使用（除非只处理少量元素）：

```text
>>> q = []
>>> q.append('eat')
>>> q.append('sleep')
>>> q.append('code')

>>> q
['eat', 'sleep', 'code']

# 小心，这种操作很慢！
>>> q.pop(0)
'eat'
```

**2.collections.deque——快速和稳健的队列**

deque 类实现了一个双端队列，支持在O(1)时间（非均摊）中从任一端添加和删除元素。由于deque 支持从两端添加和移除元素，因此既可用作队列也可用作栈。

Python 的deque 对象以双向链表实现。这为插入和删除元素提供了出色且一致的性能，但是随机访问位于栈中间元素的性能很差，耗时为O(n)。

因此，默认情况下collections.deque 是Python 标准库中不错的队列型数据结构：

```text
>>> from collections import deque
>>> q = deque()
>>> q.append('eat')
>>> q.append('sleep')
>>> q.append('code')

>>> q
deque(['eat', 'sleep', 'code'])

>>> q.popleft()
'eat'
>>> q.popleft()
'sleep'
>>> q.popleft()
'code'

>>> q.popleft()
IndexError: "pop from an empty deque"
```

**3.queue.Queue——为并行计算提供的锁语义**

queue.Queue 在Python 标准库中以同步的方式实现，提供了锁语义来支持多个并发的生产者和消费者。

queue 模块包含其他多个实现多生产者/多用户队列的类，这些队列对并行计算很有用。

在不同情况下，锁语义可能会带来帮助，也可能会导致不必要的开销。在后面这种情况下，最好使用collections.deque 作为通用队列：

```text
>>> from queue import Queue
>>> q = Queue()
>>> q.put('eat')
>>> q.put('sleep')
>>> q.put('code')

>>> q
<queue.Queue object at 0x1070f5b38>

>>> q.get()
'eat'
>>> q.get()
'sleep'
>>> q.get()
'code'

>>> q.get_nowait()
queue.Empty

>>> q.get()
# 阻塞，永远停在这里……
```

**4.multiprocessing.Queue——共享作业队列**

multiprocessing.Queue 作为共享作业队列来实现，允许多个并发worker 并行处理队列中的元素。由于CPython 中存在全局解释器锁（GIL），因此无法在单个解释器进程上执行某些并行化过程，使得大家都转向基于进程的并行化。

作为专门用于在进程间共享数据的队列实现，使用multiprocessing.Queue 能够方便地在多个进程中分派工作，以此来绕过GIL 的限制。这种类型的队列可以跨进程存储和传输任何可pickle 的对象：

```text
>>> from multiprocessing import Queue
>>> q = Queue()
>>> q.put('eat')
>>> q.put('sleep')
>>> q.put('code')

>>> q
<multiprocessing.queues.Queue object at 0x1081c12b0>

>>> q.get()
'eat'
>>> q.get()
'sleep'
>>> q.get()
'code'

>>> q.get()
# 阻塞，永远停在这里……
```

**5.关键要点**

- Python 核心语言及其标准库中含有几种队列实现。
- 列表对象可以用作队列，但由于性能较差，通常不建议这么做。
- 如果不需要支持并行处理，那么collections.deque 是Python 中实现FIFO 队列数据结构的最佳选择。collections.deque 是非常优秀的队列实现，具备期望的性能特征，并且可以用作栈（LIFO 队列）。

## 七、优先队列

优先队列是一个容器数据结构，使用具有全序关系的键（例如用数值表示的权重）来管理元素，以便快速访问容器中键值最小或最大的元素。

优先队列可被视为队列的改进版，其中元素的顺序不是基于插入时间，而是基于优先级的。对键进行处理能得到每个元素的优先级。

优先级队列通常用于处理调度问题，例如优先考虑更加紧急的任务。

来看看操作系统任务调度器的工作。

> 理想情况下，系统上的高优先级任务（如玩实时游戏）级别应高于低优先级的任务（如在后台下载更新）。优先级队列将待执行的任务根据紧急程度排列，任务调度程序能够快速选取并优先执行优先级最高的任务。

下面我们将介绍如何使用Python 语言内置或位于标准库中的数据结构来实现优先队列。每种实现都有各自的优缺点，但其中有一种实现能应对大多数常见情况，下面一起来看看。

**1.列表——手动维护有序队列**

使用有序列表能够快速识别并删除最小或最大的元素，缺点是向列表插入元素表是很慢的O(n)操作。

虽然用标准库中的bisect.insort能在O(logn)时间内找到插入位置，但缓慢的插入操作才是瓶颈。

向列表添加并重新排序来维持顺序也至少需要O(nlogn)的时间。另一个缺点是在插入新元素时，必须手动重新排列列表。缺少这一步就很容易引入bug，因此担子总是压在开发人员身上。

因此，有序列表只适合在插入次数很少的情况下充当优先队列。

```text
q = []

q.append((2, 'code'))
q.append((1, 'eat'))
q.append((3, 'sleep'))

# 注意：每当添加新元素或调用bisect.insort()时，都要重新排序。
q.sort(reverse=True)

while q:
    next_item = q.pop()
    print(next_item)

# 结果：
# (1, 'eat')
# (2, 'code')
# (3, 'sleep')
```

**2.heapq——基于列表的二叉堆**

heapq 是二叉堆，通常用普通列表实现，能在O(logn)时间内插入和获取最小的元素。

heapq 模块是在Python 中不错的优先级队列实现。由于heapq 在技术上只提供最小堆实现，因此必须添加额外步骤来确保排序稳定性，以此来获得“实际”的优先级队列中所含有的预期特性。

```text
import heapq

q = []

heapq.heappush(q, (2, 'code'))
heapq.heappush(q, (1, 'eat'))
heapq.heappush(q, (3, 'sleep'))

while q:
    next_item = heapq.heappop(q)
    print(next_item)

# 结果：
# (1, 'eat')
# (2, 'code')
# (3, 'sleep')
```

**3.queue.PriorityQueue——美丽的优先级队列**

queue.PriorityQueue 这个优先级队列的实现在内部使用了heapq，时间和空间复杂度与heapq 相同。

区别在于PriorityQueue 是同步的，提供了锁语义来支持多个并发的生产者和消费者。

在不同情况下，锁语义可能会带来帮助，也可能会导致不必要的开销。不管哪种情况，你都可能更喜欢PriorityQueue 提供的基于类的接口，而不是使用heapq 提供的基于函数的接口。

```text
from queue import PriorityQueue

q = PriorityQueue()

q.put((2, 'code'))
q.put((1, 'eat'))
q.put((3, 'sleep'))

while not q.empty():
    next_item = q.get()
    print(next_item)

# 结果：
# (1, 'eat')
# (2, 'code')
# (3, 'sleep')
```

**4.关键要点**

- Python 提供了几种优先队列实现可以使用。
- queue.PriorityQueue 是其中的首选，具有良好的面向对象的接口，从名称就能明白其用途。
- 如果想避免queue.PriorityQueue 的锁开销，那么建议直接使用heapq 模块。



