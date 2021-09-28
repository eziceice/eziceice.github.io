---
title: Thinking in Python
date: 2019-04-30 12:19:29
tags: python
categories:
- programming languages
---
# Python函数的参数
##### 1. 默认参数
比如两个参数的函数:
 ```
  def power(x, n = 2):
 ```
当以上函数被调用时,如果只传入x的值,则n的值默认为2.若n和x的值都被传入,则使用传入的值.默认参数的数量是无限制的.
可以不按照顺序提供默认参数,但是必须将变量的名字写上.
 ```
  def enroll(name, gender, age=6, city='Beijing')：
  //必须这样调用
  enroll('Adam', 'M', city='Tianjin')
 ```
**默认参数必须指向不可变得对象,不能指向list之类的数据结构,因为默认参数的值在一开始就被计算出来了.**
**必选参数在前，默认参数在后.**
##### 2. 可变参数
在函数的parameter之前添加一个\*可以将函数变为支持可变参数的函数,在函数内部,可变参数将会转化为一个tuple.同样,在list和tuple之前加一个\*便可以将list和tuple转化为可变参数.
##### 3. 关键字参数
关键字参数的函数允许传入任意个含参数名的参数，这些关键字参数在函数内部自动组装成一个dict.
 ```
  def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
  >>> person('Michael', 30)
  name: Michael age: 30 other: {}
 ```
kw必定是一个dict的数据结构,注意如果传入一个kw的dict结构,获得的将是一份拷贝而非原来的kw,因此方法内部对kw的改造并不影响之前的object.
##### 4. 命名关键字参数
命名关键字参数允许函数的参数只接受需要的关键字参数.
 ```
  def person(name, age, *, city, job):
    print(name, age, city, job)
  >>> person('Jack', 24, city='Beijing', job='Engineer')
  Jack 24 Beijing Enginee
 ```
该函数只可以接受city和job作为key的参数.注意中间需要用\*分隔开.命名关键字参数必须要传入参数名.
#### 参数部分总结
**\*args是可变参数,接受的是一个tuple.<br>
\*\*kw是关键字参数,接受的是一个dict.<br>
可变参数和关键字参数都可以直接传入数值,或先组装成\*tuple和\*\*tw在传到方法里.<br>
命名关键字参数是为了限制调用者可以传入的参数名,同时也可以提供默认值.<br>
命名关键字参数必须要在中间添加\*.**<br>


# 递归函数

## 递归
Python中的递归函数和Java类似,也应当注意防止栈溢出的问题.<br>
栈溢出是指，在计算机中，所有的方法调用都是通过栈(stack)这种数据结构实现的,每当进入一个函数调用,就增加一层栈帧,每当一个函数返回,就减少一层栈帧.由于栈的大小并不是无限的,所以当递归函数的调用次数过多时,就会出现栈溢出(stackoverflow).<br>
因此,解决栈溢出的方法是采用尾递归优化,尾递归是指,在函数返回时,调用函数本身,同时return语句中不能包含表达式.这样,无论编译器如何调用,递归函数始终只占用一个栈帧.
```
# 递归函数优化之前，容易出现栈溢出的问题
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)
''' 
计算过程
===> fact(5)
===> 5 * fact(4)
===> 5 * (4 * fact(3))
===> 5 * (4 * (3 * fact(2)))
===> 5 * (4 * (3 * (2 * fact(1))))
===> 5 * (4 * (3 * (2 * 1)))
===> 5 * (4 * (3 * 2))
===> 5 * (4 * 6)
===> 5 * 24
===> 120
'''
# 优化后
def fact(n):
    return fact_iter(n, 1)

def fact_iter(num, product):
    if num == 1:
        return product
    return fact_iter(num - 1, num * product)

''' 
计算过程
===> fact_iter(5, 1)
===> fact_iter(4, 5)
===> fact_iter(3, 20)
===> fact_iter(2, 60)
===> fact_iter(1, 120)
===> 120
'''
```
从上面我们可以看出,栈的调用情况被简化了.

## 递归二分查找
```
def search(squence, number, lower, upper):
    if (lower == upper):
        return upper
    else:
        middle = lower + (upper - lower)//2
        if number > middle:
            search(squence, number, middle + 1, upper)
        else:
            search(squence, number, lower, middle)
```


# 高级特性
##### 1. 切片
不同于Java,Python允许操作list就像操作字符串,对于一个list,可以直接使用切片取其中的一段元素.
 ```
 L = list(range(100))
 L[:10]
 L[5:50]
 L[-1:-20] #python支持负index,最后一个数的index是-1
 L[:10:2] #从0到10每2个数取一个,后面的值为步长,默认为1,步长可以使负数,此时分片从右往左提取元素
 ```
**对于正数步长,Python会从序列的头部开始向右提取元素,直到最后一个元素;而对于负数步长,则是从序列的尾部开始向左提取元素,
直到第一个元素.**<br>
切片可以运用于list,tuple和字符串,Java中的substring就类似于Python中切片对字符串的操作.
如果分片中最左边的索引比最右边的索引出现的晚(左边索引的值必须大于右边索引),则得到一个空切片.
 ```
  numbers = [1,2,3,4,5,6]
  print(numbers[-3:0]) #结果为[]
  print(numbers[-3:]) #结果为[4,5,6]
  print(numbers[:3]) #结果为[1,2,3]
 ```
##### 2. 迭代
Python的迭代不仅可用于list,也可以用于tuple和dict.
```
 d = {'a' : 1, 'b' : 2, 'c' : 3}
 for key in d:
    print(key)

 for value in d.values():
    print(value)

 for k, v in d.items():
    print(k,v)
```
通过collections模块的Iterable类型来判断一个对象是否可以被迭代对象:
```
 from collections import Iterable
 isinstance('abc',Iterable)
```
Python中也有类似Java里的下表循环,需要使用enumerate函数将list变成索引-元素对.
```
 for i,value in enumerate(['A','B','C']):
    print(i,value)
 ```
##### 3. 列表生成式 (List Comprehensions)
```
[x * x for x in range(1,10)] #可以直接生成包含1-10平方值的list.
```
列表生成式同时也可以加入if条件
```
[x * x for x in range(1,10) if x % 2 == 0] #生成一个1-100平方数都可以被2整除的list
[m + n for m in 'ABC' for n in 'XYZ'] #生成全排列 结果: ['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
L1 = ['Hello', 'World', 18, 'Apple', None]
L2 = [s for s in L1 if isinstance(s,str)] #输出只有string的list
```


# 常用API
1. input()函数通常需要用户来输入一个带''的值，这是不现实的.如果要想获取用户输入通常使用raw_input()函数.<br>
2. 使用'''可以将输出的超长字符串进行换行.<br>
   同时,使用原始字符串,将会输出原始字符串,意味着不会对\进行转义操作,原始字符串在使用正则表达式时会比较有用.
```
 print(r'C:\nowhere')
 输出结果: C:\nowhere
```
3. len()获取长度,list()可以将任何一个类型的序列转化为List,而''.join(alist)则可以将任何List转换为字符串.
```
 list('hello') #结果为['h','e','l','l','o']
 ''.join(list('hello')) #结果为'hello'
```


# List & Tuple

**List & Tuple**
1. List使用[]表示,而Tuple使用()表示.
2. 使用+运算符可以将两个List连接在一起,而使用乘法*将会把一个List乘以n次得到新的List,其中原来的List被重复了n次.
```
 [1,2,3] + [4,5,6] #结果为[1,2,3,4,5,6]
 ['python'] * 5 #结果为['python','python','python','python','python']
```
3. List中的删除
```
 L = [1,2,3]
 del L[1] #2将被移出L.
```
4. 给切片赋值,可以通过List的切片,对List进行插入和删除操作.

**List常用方法**

- append(),将一个item添加至list末尾.

- clear(),清楚list中的所有内容.

- copy(),该方法将复制一个list的副本,而不是单纯的复制引用.此种情况下更改初始list的值并不会造成list副本值的变化. 使用切片[:]或者list()方法,会得到和copy()方法同样的效果.

- count(),计算指定元素出现了的次数.

- extend(),该方法能将多个值加入到原list的末尾.该方法和+不同的是,它将直接改变原来list的长度,而使用+需要进行赋值操作,效率比extend()低.

- index(),刚方法返回所求值在list中的序列位置.(若无法找到,将会抛出exception而不是返回-1).

- insert(),在指定位置插入一个item.

- pop(),将List中最后的值弹出.(获得最后的值并且将该值从list删除),若有index传入,则pop出该index的值.

- remove(),用于移除list中某个匹配值得第一项.

- reverse(),用于将列表中的元素反向存放.刚方法也会改变原来的list,若只是单纯想得到反序列的值,可以使用reversed()方法.
```
list(reversed(x))
```

- sort(),将列表中的元素排序,此方法也会改变list中的序列,并且没有返回值.若想得到具有返回值的排序列表副本,建议使用sorted()函数.同时,和Java类似,sort()方法也可以传入自定义的接口,在Python中可以传入cmp()方法和key来提供排列顺序,同时也可以通过传入reverse的布尔值来表示是否需要逆排序.
```
numbers.sort(cmp)
numbers.sort(key=len)
numbers.sort(reverse=True)
```
**Tuple**

- 建立一个tuple通常使用(),建立一个只含有一个值的tuple需要在末尾添加,.
```
(1,2,3)
(41,)#空的Tuple
3*(40+2) #表达式
3*(40+2,)#Tuple,结果为(42,42,42)
```
- tuple()方法可以将一个参数转换为tuple.

- Tuple也可以像list一样进行切片和index访问.
> **Tuple和List的一个重要区别就在于Tuple可以作为dict中的Key存在,因为它是不可变对象.**


# String

## String
- find(),可以在一个长的string中查找子串,并且返回第一个找到的字符串的最左边字母的index.如果找不到便返回-1.该方法还可以提供起始点和结束点,相当于在substring中查找值.
```
subject.find('x',1,10) #从字符串1-10的位置中查找x.
```
- join(),该方法可以将string连接起来.不同的是,join方法将会对b中的每个字符添加a.
```
a = '123'
b = '456'
a.join(b) #412351236123

a = ['1','2','3']
b = '+'
b.join(a) #'1+2+3'
```
- lower(),将字符串化为小写字母,类似于Java中的toLowerCase().

- replace(),将匹配的字符进行替换.

- split(),join()的逆方法,将字符串分隔开.如果不提供任何字符,程序会根据所有空格将字符串分隔开.

- strip(),去除字符串前后端的空格(不包括中间的).也可以用来去除指定的字符,只要将他们列为参数即可,但是只能去除**字符串前后的对应字符**.

- translate()方法和replace方法一样,但是其每次只可以替换单个字符,有些时候该方法比replace()更有效率.


# Dict

## Dict

- 可以通过dict函数,将其他字典或者键值对序列建立成字典.
```
items = [('name', 'Gumby'), ['age', 42]]
d = dict(items)
print(d) # {'age':42, 'name':'Gumby'}

d = dict(name = 'Gumby', age = 42)
print(d) # {'age':42, 'name':'Gumby'}
```

- 键类型: 字典的键不一定是整型数据,但一定是不可变类型.
```
x = []
x[42] = 'test' #错误, out of range
x = {}
x[42] = 'test' #正确
print(x) # {42:'test'}
```

- 成员资格: 表达式k in d查找的是键,而不是值.表达式v in l查找的是值,不是索引.

## 常用API

- clear(),清楚字典中的所有数据. (无返回值)

- copy(),copy方法返回一个具有相同键-值对的新dict(这个方法实现的是浅复制 - shallow copy,因此值本身是相同的,而不是副本).**copy.copy 浅拷贝-只拷贝父对象,不会拷贝对象的内部的子对象. copy.deepcopy 深拷贝-拷贝对象及其子对象.**
```
x = {'username': 'admin', 'machines': ['foo', 'bar', 'baz']}
y = x.copy()
y['username'] = 'mlh'
y['machines'].remove('bar')

print(y)
print(x)
# {'username': 'mlh', 'machines': ['foo', 'baz']}
# {'username': 'admin', 'machines': ['foo', 'baz']}
```
从以上代码中可以看到,当在副本中替换值的时候,原始字典不受影响,但是,如果**修改**某个值(原地修改,非替换),则原始的字典也会被改变,因为同样的值也存在于原字典中(就像上面例子中的machines列表一样).**浅复制之后值仍然是共享的**.<br>
如果需要避免这种问题,可以使用深复制(deep copy),刚方法会复制所有的值.
```
from copy import deepcopy

d = {}
d['names'] = ['Alfred', 'Betrand']
c = d.copy()
dc = deepcopy(d)
d['names'].append('Clive')

print(d) # {'names': ['Alfred', 'Betrand', 'Clive']}
print(c) # {'names': ['Alfred', 'Betrand', 'Clive']}
print(dc) # {'names': ['Alfred', 'Betrand']}
```
- fromkeys(),使用给定的键建立新的字典,每个键都对应一个默认的None.

- get(),通常使用索引访问字典中不存在的项时会出错,而使用get就不会.
```
d = {}
print d['name'] #报错
d.get('name') #返回none
d.get('name', 'not exist') #返回not exist,自定义找不到时候的输出.
```

- has_key(),等同于k in d.

- items()返回无特定次序的key-value的list,而iteritems()返回的结果和items()相同,只不过iteritems()返回的是一个iterator,需要使用list()函数将结果转化为list.**在很多情况下使用iteritems()更为高效**.

- keys(),等同于items(),但是只返回key,iterkeys()同理.

- pop(),用于获得给定的value,同时将该key-value在dict中移除.

- popitems(),随机弹出dict中的项,可以用于一个一个移除dict中的项.

- setdefault(),类似于get()中设默认值的方法,可以将不存在的键的值设为默认的值而不是返回None.

- update(),将一个字典项更新到另一个字典里.
```
d = {'title': 'name'}
x = {'title': 'name2'}
print(d) # {'title': 'name'}
d.update(x)
print(d) # {'title': 'name2'}
```
-  values(),等同于items(),但是只返回value, itervalues()同理.


# If & Loop Statement

## If & Loop & Other Statement

- Import包可以使用别名.
```
import math as foobar
foodbar.sqrt(4)

from math import sqrt as foobar
print(foobar(4))
```

- 序列解包(sequence unpacking),也叫做递归解包,将多个值的序列解开,然后放到变量的序列中.解包允许函数返回一个以上的值并且打包成元组, 然后通过一个赋值语句很容易进行访问.所解包的序列中的元素数量必须和放置在赋值符号=左边的变量数量完全一致.
```
x, y, z = 1, 2, 3
print(x,y,z) # (1,2,3)

x, y = y, x
print(x,y,z) # (2,1,3)

values = 1,2,3
x,y,z = values
print(x,y,z) # (1,2,3)
```

- 链式赋值(chained assignment),是将同一个值赋给多个变量的捷径.看起来有些像并行赋值,但是实际上只是处理了一个值.
```
x = y = somefunction()
#等价于
y = somefunction()
x = y
#不一定等价于
x = somefunction()
y = somefunction()
```

- 在Python中,冒号(:)用来标识语句块的开始,块中的每一个语句的缩进量都是相同的.当回退到和已经闭合的块一样的缩进量时,就表示当时块已经结束了.**Python标准推荐的方式是只用空格,每个缩进需要4个空格,并不是使用tab,因为tab是8个空格.**

- Python的解释器将会把下面的值当做布尔表达式中的假(False). <br>
  **False None 0 "" '' () [] {}**
```
print(True + False + 42) // True = 1 False = 0
```
bool()方法可以用来将值和布尔值进行转换. **注意,这里并不意味着其他这些等同于False的值是相等的.**

- elif子句,Python中的else if使用elif代替.
```
if x:
    dosomething()
elif:
    dosomethingelse()
else:
    doit()
```

- 比较运算符和Java中的类似,多了以下两种.
```
x is y # x和y是同一个对象
x is not y # x和y不是同一个对象
x in y # x在容器y中
x not in y # x不在容器y中

# 同时, python中的比较运算符是可以连接使用的
if 0 < x < 100:
    dosomething()
```

- ==运算符和Java中类似,通常用于比较两个值是否相同,而x is y则比较两个值是否为同一个,和equals()有些类似.**使用==运算符来判定两个对象是否相等,使用is判定两者是否等同(同一个对象).**

- 比较运算符可以用于比较字符串和序列,基本上来说就是一个一个来比较.
```
print('abc' < 'bcd') # True

print([1,2,3] < [1,2,5]) # True
```

- Python中的and和or运算符就是使用and和or,并不像Java使用&&和||.
  **惰性求知(lazy evaluation)或者短路逻辑(short-circuit logic), 就是指只有在需要的时候才求值,如果第一个表达式不成立,则后面的表达式直接不考虑(略过).**

- Assert,可以在条件不成立是抛出exception使程序崩溃.
```
a = 50
assert a > 100, 'a must be larger than 50' #AssertionError: a must be larger than 50.
```

## 常见的loop工具

- range()和xrange()的区别,range()一次生成全部的序列,而xrange()每次只生成一个数.因此,当需要生成的range非常大的时候,建议使用xrange().

- 并行迭代,程序可以同时迭代两个序列. zip()方法可以作用于任意多的序列,也可以处理不等长的序列,当最短的序列用完时就会停止.
```
names = ['test1','test2','test3','test4']
ages = [12,13,14,15]

for i in range(len(names)):
    print(ages[i], names[i])

 # zip函数可以将两个序列压缩在一起，组成一个tuple的list.
for name, age in zip(names, ages): # 解包元组 也可以使用 for i in zip(names, ages):
    print(name, age)
```

- Python中的按索引迭代通常使用enumerate()
```
for index, item in enumerate(templist):
     if 'abc' in templist:
          templist[index] = 'opq'
```

- sorted()和reversed()方法,可以作用于任何序列或者可迭代对象,不是原地修改,而是返回翻转或者排序之后的序列.**sorted方法返回列表,reversed方法返回一个可迭代对象(需要配合list()或者循环方法使用)**

- for和while循环中都可以使用continue, break和**else**语句, else语句只在没有调用break的时候才执行.
```
for i in range(0,100):
    if i == 101: #若i等于101,执行else.若i=0-99中的某个数,调用break并且跳过else.
        print('break')
        break
else:
    print('I am in else')
```

- 列表推导式(list comprehension),利用其他列表创建新列表.该方法比使用循环更加简洁.
```
print([x * x for x in range(0,10)])

print([x * x for x in range(0,10) if x % 3 == 0])

print([(x, y) for x in range(0,10) for y in range(5)])
```

- pass,如果不要程序做什么,可以使用pass,和Java中的Todo差不多.先空下今后再开发.

- del方法不仅会移除对象的引用,也会移除对象本身. (不需要过多考虑,python有垃圾回收的机制.)实际上删除的仅仅是名称而已,真正的对象删除靠的是GC的机制.Python的解释器会负责这些GC的处理.

- exec(),可以直接执行一个字符串,也可以动态地穿件代码字符串.

- eval(),求值函数.可以直接计算一个python表达式.


# Abstract

## 抽象

- 文档化函数.在Python中除了添加注释以外(\#开头),也可以在函数的开头写下字符串,它就会作为函数的一部分进行储存,这被称为文档字符串.同时,Python内置的help()函数可以得到关于函数的一些有用的信息,具体如下.
```
def test(name):
    'this is a test'

print(test.__doc__)

print('***')

help(test)

#输出结果
this is a test
***
Help on function test in module __main__:

test(name)
    this is a test
```

- 所有的函数的确都有返回值,当函数中没有存在返回值的时候,就返回None.

- 类似于Java,函数的参数如果是基本类型(字符串,数字以及元组),则传入的实际上为拷贝的值,在函数内改变该参数的值并不会影响原来的参数.而当传入Object或者list之类的时候,在其中改变参数的值会影响原来的list或者object.如果不想对该类参数进行修改,可以使用切片或者deepcopy().
## 关键字参数和默认值

- 除了支持位置参数之外,Python还支持关键字参数,可以提供参数的名字在方法中,让函数明白参数的值是哪一个.
```
def helloworld(greeting, name):
    print(greeting + ', ' + name)

helloworld(greeting='hello', name='world')
```
- 关键字参数最厉害的地方在于可以给参数提供默认值.
```
def helloworld_1(greeting='Hello',name='world'):
    print(greeting + ', ' + name)

helloworld_1()
```
- 位置参数和关键字参数可以同时使用,只需要将位置参数放在前面就可以了.如果不这样做,解释器将会报错.**除非完全清楚程序的功能和参数的意义,否则应该避免混合使用位置参数和关键字参数,一般来说,只有强制要求的参数个数比可修改的具有默认值的参数个数少的时候,可以使用.**

- 可以在参数中使用*来传递一系列的参数,*后面的参数将会被当做元组来处理. *的意思可以看做为'收集其余的位置的参数',如果不提供参数,将输出一个空tuple().
```
def print_params(*params):
    print(params)

print_params('1') # (1,) - 注意,如果参数中只有一个元素输出结果将会是(1,)

print_params('1','2',3) # ('1','2',3)
```

- 在参数中使用\**可以收集参数组装成字典.
```
def print_params(*params, **keypar):
    print(params, keypar)

print_params('1','2', key1=['1234'], key2=2) # (('1', '2'), {'key2': 2, 'key1': ['1234']})
```

## 变量

- 变量是什么?可以把变量看作是值的名字.变量和所对应值用的是一个不可见的字典.内建的vars()可以返回这个字典.该类不可见的字典可以叫做命名空间或者作用域.
```
x = 1 #此处可以看做把名称x引用到值1上,这就像一个字典,键是x,值为1.
scope = vars()
print(scope['x'])
scope['x'] += 1
print(scope['x'])
```

- 局部变量只在方法内部受影响,而不会影响全局变量的值.该处和Java中的处理机制略有不同.那么如何想要在方法中使用全局变量呢?一是保证方法中的变量名和全局变量名不同,则可以正常使用全局变量.二是如果全局变量和局部变量名相同的话,全局变量将会被局部变量屏蔽,这里可以使用global()来获取全局变量的值.
```
globals()['parameter'] #可以获取全局变量
```

- 如何重新绑定全局变量呢？在函数内部将值赋予一个变量,那么该值会自动变为局部变量.此时我们如果想改变全局变量的值,需要使用global parameter才可以改变全局变量的值.
```
def changeglobal():
      global x
      x = x + 1
```

- Python的函数是可以嵌套的,也就是说可以将一个函数放在另一个里面. 如果一个函数位于另外一个里面,外层函数返回里层函数,就可以将变量赋值为一个函数. 类似于mutiplyByFactor()存储子封闭作用域的行为叫做闭包(closure).
```
def multiplier(factor):
    def mutiplyByFactor(number):
        return factor*number
    return mutiplyByFactor #此处返回内层函数本身,不可以加()

double = multiplier(3)
print(double(4))
```


# Further Abstract

## 更加抽象的内容

- Python也是OOP的编程语言,因此封装继承多态也是Python中的重要特性.

- **多态**. 事实上,唯一能够毁掉多态的就是使用函数进行显式的类型检查,比如type,isinstance以及issubclass函数等.如果可能的话,应该尽量避免使用这些毁掉多态的方式.真正重要的是如何让对象按照你所希望的方式工作,不管它是否是正确的类型.

- **封装**.每一个object都有自己的状态,使用者不需要关心这个object的各种attribute是怎样进行设定的.

- **继承**.更好的设计,减少代码量.

- 类(class)是指一种类型,而object都是某一个类的instance.
```
__metaclass__ = type #确定使用新式类,旧式类和新式类之间是有区别的. 
                     #新式类的语法中需要放置该赋值语句,但不是必要的显示地包含这行语句.

class Person:
    name = ''
    def setName(self, name):
        self.name = name
    def getName(self):
        return self.name
    def greet(self):
        print("hello world " + self.name)
        
```

- 方法(绑定方法)将它们第一个参数绑定到所属的实例上,因此不需要提供该参数(self).Python与Java不同的是可以将方法方法绑定到另一个方法上,这样原有的方法就会被改变了. Python中的方法绑定换了实际就是将参数中需要的self更换了.
```
class Test:
    def method(self):
        print('12345')

def method2():
    print('678910')

a = Test()
a.method() #12345
a.method = method2 #对方法进行绑定,可以理解为对方法进行赋值.
a.method() #678910
b = a.method
b() #678910
```

- Python并不直接支持私有方式(类似于Java中的private),可以通过添加双下划线_来将方法或者特性变为私有. 不过通过object._Class名__私有方法或者attribute也是可以访问到私有的property的,但是不建议这样做.**简而言之,Python中并不能确保私有的方法或者变量无法被外界访问(类似于Java反射也可以访问私有的变量或者方法),但是十分不建议developer这样做.**
```
class Private:
    __name = '12345' #设为私有
    def __getName(self): #设为私有
        print(self.__name)

    def getName(self):
        self.__getName()

p = Private()
p.getName()
p.__name #无法访问,报变量不存在的exception
p.__getName() #无法访问,报方法不存在的exception
p._Private__getName() #可以访问,但是并不建议
```

- 类的命名空间(class namespace),所有位于class语句中的代码都在特殊的命名空间中执行,这种特殊的命名空间被称为类命名空间.这个命名空间可由类内所有成员访问.
```
class C:
    print(1234)
C #1234

class MemberCount:
    members = 0 #该attribute为所有object共享
    def init(self): #该方法只为每个self object使用
        MemberCount.members += 1

a = MemberCount()
a.init()
print(a.members) #1

b = MemberCount()
b.init()
print(b.members) #2 - members为所有类型共享的属性

a.members = 'k2'
print(a.members) #k2 - 新members的值被写到了a的特性中
print(b.members) #2 - b的特性中的members并没有被改变,因为上一个过程中发生了类范围内的变量屏蔽.
#类似于如果给共享的全局变量赋值,该变量就变成了该instance的局部变量.
```

- Python中的继承使用方法.
```
class Filter:
    def init(self):
        self.blocked = []
    def filter(self, sequence):
        return [x for x in sequence if x not in self.blocked]

class SPAMFilter(Filter): #SPAMFilter 继承自Filter
    def init(self):
        self.blocked = ['SPAM']
```

- issubclass()方法可以判断一个类是否为另一个类的子类. 如果想知道一个类的基类,可以访问其特殊属性__bases__. 同样,可以通过__class__来获取对象属于哪个类. 对于所有的新式类,type()方法也可以获取所属的类.
```
issubclass(SPAMFilter, Filter)
print(SPAMFilter.__bases__)
s = SPAMFilter()
print(s.__class__)
type(s)
```

- Python支持**多重继承**, 但是应当尽量避免使用多重继承,因为容易带来比较混乱的情况.**同时有一点需要格外注意的是,如果多个超类以不同的方式实现了同一个方法-多个同名方法,必须在class语句中小心排列这些超类,因为位于前面的类的方法将覆盖位于后面的类的方法.**
```
class Calculator:
    def calculate(self, expression):
        self.value = eval(expression)

class Talker:
    def talk(self):
        print('Hi, my value is', self.value)

class TalkingCalculator(Calculator, Talker): #如果两个类中有相同的方法,则继承顺序中的第一个类覆盖
                                             #第二个类的相同的方法.继承顺序将会影响方法的访问顺序.
    pass

tc = TalkingCalculator()
tc.calculate('1 + 2 * 3')
tc.talk()
```

## 接口

- Python中接口的概念和多态息息相关,在处理多态对象时,只要关心它的接口即可,也就是公开的方法和特性.不需要像Java一样显式的制定接口,Python可以在使用对象的时候假定它可以实现你所要求的行为(方法),如果不能实现程序变失败.
```
hasattr(tc, 'talk') #可以用来检测该object中是否包含后面的method或者attribute
hasattr(method, '__call__') #可以用来判断方法是否可以被调用,method需要先被定义
getattr(instance, 'name', 'default_value') #获取instance中的方法或者属性,
                                           #如果不存在便使用后面的默认值
setattr(instance, 'name', 'value') #在instance中添加一个方法或者属性(或者覆盖),后面为默认值.
```


# 异常处理

## Exception

- Python用异常对象(exception object)来表示异常情况.遇到错误后,会引发异常,如果异常对象没有被捕捉或者处理,程序就会用所谓的回溯(traceback,一种错误信息)终止执行-类似于Java中的stacktrace. 自定义异常类只要继承自exception就可以了.
```
raise Exception('test')

import exceptions
dir(exceptions) #列出python中所有种类的异常

class CustomException(Exception): #自定义异常继承自Exception
pass
```

- 捕捉异常使用try/except语句来实现.
```
try:
    x = input('1: ')
    y = input('2: ')
    print(x/y)
except ZeroDivisionError:
    print('1234')
```

- 如果捕捉到了异常,但是又想重新引发它(抛出异常或者传递异常),那么可以调用不带任何参数的raise. 同时,可以添加多个语句捕捉到多种异常.也可以用一个块捕捉多个异常.
```
try:
    x = input('1: ')
    y = input('2: ')
    print(x/y)
except ZeroDivisionError:
    raise 
except TypeError:
    raise
except (ZeroDivisionError, TypeError, NameError):
    raise
except (ZeroDivisionError, TypeError, NameError) as e: #捕捉异常并在console中输出
    print(e)
except: #捕捉全部异常,但需要谨慎使用,有时候有些错误会被忽略
    print('error')
except Exception as e: #用这个except捕捉全部异常好一些
    print(e)
```

- try/except可以和else一起使用.只有当exception不被触发的时候,else语句才会被调用.
```
try:
    print('123')
except:
    print('456')
else:
    print('bye')
```

- finally可以放在try/except最后,finally中的语句将在异常发生或者不发生的情况下都会被执行.

## Warning

- 可以引用模块warnings中的函数warn来对不正确的地方发出警告.同时,warnings中的函数filterwarnings可以来抑制发出的警告,并制定要采取的措施.另外,还可以根据异常来过滤掉特定类型的警告.
```
from warnings import warn
warn('this is a warning')

from warnings import filterwarnings
filterwarnings('ignore')  #ignore发出的警告
warn('123') #被ignore
filterwarnings('error') #mark警告为error
warn('123') #抛出警告-等同于抛出异常
filterwarnings('ignore', category=DeprecationWarning) #忽略特定种类的警告
warn('123', DeprecationWarning) #对于特定种类的警告抛出
```


# Property and Iterator

## 构造函数

- 在Python3中将不会存在旧式类,所有的类都将隐式地继承object.

- __init__是Python中的构造函数,它将会在函数创建后被自动调用.
```
class Test:
    def __init__(self, value=42):
        print('12345',value)

t = Test('kkk') #输出12345 kkk
```

- 析构函数(destructor)的方法在Python中也被提供了,这个方法可以在对象被销毁前调用,但需要准确的知道调用时间,否则容易引起错误.
```
class Test:
    def __init__(self, value=42):
        print('12345',value)
    def __del__(self):
        print('123456')
t = Test('kkk') #输出12345 kkk 123456 - 说明t被销毁了
```

- 覆盖重写子类的constructor的时候,必须先调用超类的constructor,否则有可能无法正确地初始化. 通常是使用super()方法,不过对于一些老版本的Python代码,也可以直接调用超类的constructor.
```
class Bird:
    def __init__(self):
        self.hungry = True
    def eat(self):
        if self.hungry:
            self.hungry = False
            print('I am Full now')
        else:
            print('I am not hungry')

class SongBird(Bird):
    def __init__(self):
        super().__init__()
        # Bird.__init__(self) 旧版方法-通过类调用方法,就不会有实例与其相关联,因此便可以随意设置参数self.
        # 这种方法调用的模式称为未关联方法调用.
s = SongBird()
s.eat()
s.eat()
```

- 使用super()方法的优点是显而易见的. 即使一个类有多个超类,也只需要调用super()一次(条件是所有超类的构造函数也使用super).同时,super()将返回一个超类的对象(类似于包含所有超类的合集的对象),当你尝试去访问super()返回的对象的某个属性或者方法时,Python将会在所有的超类中进行查找,直到找到该属性或者引发AttributeError异常.

## 元素访问

- 在Python中,协议(protocol)通常指的是规范行为的规则,其指定应实现哪些方法以及这些方法应该做什么(类似于Java中的接口). 只要该类实现了某种协议,便可以遵从该种协议具有一系列相同的行为.

- list和dict基本上都是item的集合,要实现它们的基本协议,不可变对象需要实现2个方法,而可变对象需要实现4个方法.
    1. \_\_len__(self): 集合包含的项数,对序列来说为元素的个数,对映射来说是键值对的数量.
    2. \_\_getitem__(self, key): 返回对应的值,根据序列位置对应的元素或者dict的key.
    3. \_\_setitem__(self, key, value): 添加元素,根据序列的位置或者dict的键值对. - **只有可变对象需要**
    4. \_\_delitem__(self, key): 删除元素,根据提供的key或序列的位置. - **只有可变对象需要**
```
def check_index(key):
    if not isinstance(key, int):
        raise TypeError
    if key < 0:
        raise IndexError

class ArithmeticSequence:
    def __init__(self, start = 0, step = 1):
        self.start = start
        self.step = step
        self.changed = {}

    def __getitem__(self, key):
        check_index(key)
        try:
            return self.changed[key]
        except KeyError:
            return self.start + key * self.step

    def __setitem__(self, key, value):
        check_index(key)
        self.changed[key] = value
s = ArithmeticSequence()
#以上的类建立之后可以使用s[0], s[1]来访问元素,类似list或者dict的行为
```

- 有时候需要重写的方法太多,工作量很大,而如果仅仅是想实现list或者dict的行为,可以直接使用继承.
```
class CouterList(list):
    def __init__(self, *args):
        super().__init__(*args)
        self.counter = 0
    def __getitem__(self, index):
        self.counter += 1
        return super().__getitem__(index)
```

## Property - 特性

- property()函数可以将一个类中通过计算的值很容易的通过instance.field的表现形式进行获取. property()函数只可以运用于新式类,否则setter可能会不正常,getter依然可以正常运行.
```
class Rectangle:
    def __init__(self):
        self.width = 10
        self.height = 5
    def set_size(self, size):
        self.width, self.height = size
    def get_size(self):
        return self.width, self.height
    size = property(get_size,set_size) #getter在前,setter在后

r = Rectangle()
print(r.width)
print(r.height)
print(r.size)

r.size = (200,300)
print(r.width)
print(r.height)
print(r.size)
```

- property()函数可以不指定参数到指定一个参数/两个/三个/四个参数.如果没有指定参数,则特性不可读也不可写,一个参数(getter)只可以读.第三个参数可选的,为删除属性的方法.第四个参数也是可选的,指定一个文档字符串. property()参数的名字分别为**fget, fset, fdel和doc**. 如果需要创建一个只可写并且带文档的property,只需要使用以上关键字参数即可.

## 静态方法和类成员方法

- 静态方法和类成员方法分别在创建时被装入staticmethod类型和classmethod类型的对象中.静态方法的定义没有self参数,且能够被类本身直接调用.类方法在定义时需要名为cls的类似于self的参数,类成员方法可以直接用类的具体对象调用-但cls参数是自动绑定到类的. 可以使用annotation直接绑定静态方法或者class方法.
```
class MyClass:
    @staticmethod
    def smeth():
        print('this is a static method')
    @classmethod
    def cmeth(cls):
        print('this is a class method')
MyClass.smeth()
MyClass.cmeth()
```

- Intercept(拦截)是指对对象的访问特性访问的时候进行一定的规则并且得到一系列处理后的值(不再是简单的获取想要的值).常用的拦截方法为:

    1. \_\_getattribute__(self, name):当特性name被访问时自动被调用.(只能在新式类中使用)
    2. \_\_getattr__(self, name):当特性name被访问时且对象没有相应的property时被自动调用.
    3. \_\_setattr__(self, name, value): 当试图给特性name赋值时会被自动调用.
    4. \_\_delattr__(self, name): 当试图删除特性name时被自动调用.

**尽量避免在getattribute, getattr和setattr中使用self,会让程序产生死循环.在使用setattr是注意使用特殊方法__dict__,该特殊方法包含一个字典,字典里是所有实例的属性.在使用getattribute时,会拦截所有特性的访问,同时也拦截对__dict__的访问,因此安全的做法是访问super().__getattribute__(\*args)方法**

```
class People:
    def __init__(self, age, name):
        self.age = age
        self.name = name

    def __getattribute__(self, obj):
        if obj == 'age':
            print("被询问了年龄:")
            return super().__getattribute__(obj) #使用超类的getattribute方法,避免死循环
        elif obj == 'name':
            print('被询问了名字')
            return super().__getattribute__(obj)
        else:
            return super().__getattribute__(obj)
     def __setattr__(self, obj, value):
         if obj == 'size':
            self.age, self. name = value
         else:
            self.__dict__[obj] = value #避免死循环,将不为同名的键值对放入字典中

p = People(17, 'Test')
print(p.age)
print(p.name)

```

## 迭代器 - Iterator

- \_\_iter__返回一个迭代器,它是包含方法_\_next__的对象,调用这个方法是可以不提供参数.当调用next方法时,迭代器返回其下一个值.如果迭代器没有提供可返回的值,应引发StopIteration异常. 如果只想每次获取一个值,那么使用iterator比使用列表方便,因为使用列表你需要一次性初始化所有的值,占用内存较多.如果需要获取的值为无穷大的,list更加无法使用.
```
class Fibs:
    def __init__(self):
        self.a = 0
        self.b = 1
    def __next__(self):
        self.a, self.b = self.b, self.a + self.b
        return self.a
    def __iter__(self):
        return self
```

- 比较正式的说法是,一个实现了\_\_iter__()的对象是可迭代的,一个实现了next方法的对象则是迭代器. iter方法和next方法通常需要一起实现的.否则并不能进行迭代.
```
#内置函数iter()也可以在可迭代对象中获得迭代器
it = iter([1,2,3])
```

- 序列和可迭代对象基本可以互相转换. 在大部分情况下,两者可以互相替换.
```
class TestIterator:
    value = 0
    def __next__(self):
        self.value += 1
        if self.value > 10:
            raise StopIteration
        return self.value
    def __iter__(self):
        return self

t = TestIterator()
print(list(t))
```

## 生成器 - Generator

- 任何包含yield语句的函数被称为生成器.yield和普通函数的区别是它不像return那样返回值,而是每次产生多个值.每次使用yield语句,函数就会被冻结:即函数停在那点等待被重新唤醒,函数被重新唤醒后就从停止的那点开始执行.
```
def flatten(nested):
    for sublist in nested:
        for element in sublist:
            yield element

nested = [[1,2],[3,4],[5]]
for num in flatten(nested):
    print(num)
```

- 在python中,对字符串进行迭代实际上会导致无穷递归,因为一个字符串的第一个元素是另一个长度为1的字符串,而长度为1的字符串的第一个元素就是字符串本身.
```
def flatten(nested):
    try:
        try:
            nested + '' #其他类型和字符串相加会报错-python特性
        except TypeError:
            pass
        else:
            raise TypeError('No String')
        for sublist in nested:
            for element in flatten(sublist):
                yield element
    except TypeError:
        yield nested
```

- 生成器是一个包含yield关键字的函数.当它被调用时,在函数体中的代码不会被执行,而会返回一个迭代器.每次请求一个值,就会执行生成器中的代码,直到遇到一个yield或者return语句,yield语句意味着应该生成一个值,而return语句意味着生成器停止执行. **换句话说,生成器是有两部分组成的-生成器的函数和生成器的迭代器.生成器的函数是用def语句定义的,包含yield部分,生成器的迭代器是这个函数的返回的部分.**
```
def simplegenerator():
    yield print(1)

print(simplegenerator()) #<generator object simplegenerator at 0x104574eb8> 调用返回迭代器
for i in simplegenerator(): #当迭代器被使用时,代码会被执行,同时yield将会被记录位置
    pass
```

- **生成器方法** 基本上来说就是一个方法如果有yield函数,调用时首先只返回一个生成器(可迭代),使用next函数时开始执行函数体,遇到yield返回.下一次执行函数体(需要调用next或者send方法)时从上一次执行的yield处可是继续执行,直到遇到下一个yield或者代码执行完毕.

    - 外部作用域访问生成器的send方法,就像访问next方法一样,只不过send方法需要提供一个参数(要发送的消息,可以是任意对象).

    - 在内部则挂起生成器, yield现在作为表达式而不是语句使用,换句话说,当生成器重新运行的时候,yield方法返回一个值.也就是外部通过send方法发送的值.如果next方法被使用,那么yield方法返回None.

    - 注意,使用send方法只有在生成器挂起之后才有意义(也就是说yield函数第一次被执行之后).如果在此前需要给生成器提供更多信息,那么只需要使用生成器函数的参数即可.
```
def repeater(value):
    while True:
        new = (yield value) # 使用yield返回值当做语句时,请务必添加括号
        if new is not None:
            value = new

r = repeater(50) #获得生成器(可迭代)
print(r.send(None)) #发送None可以获得默认值,通常为参数
print(r) #生成器对象
print(r.__next__()) #在yield处停止,返回value
print(r.send('hello')) #从上一个yield处开始执行,遇到yield然后返回hello
                       #此处是一个infinite loop因此必须在下一个yield处停止

```

- 生成器还包含另外两个方法, throw()和close().

    1. throw(): 用于在生成器中(yield表达式处)引发异常,调用时可以提供一个异常类型,一个可选值和一个traceback对象.
    2. close(): 用于停止生成器,调用时无需提供任何参数 - 该方法实际上也是基于异常的,在yield处引发GeneratorExit的异常.因此如果要在生成器中提供一些清理代码,可以将yield放在一条try/finally语句中. 调用close()后如果再次试图从generator获取值将会获得RuntimeError异常.

## 八皇后 - Generator

- 首先,使用tuple或者list来表示当前皇后的状态. state[0] = 3表示第一个皇后的位置位于第一行第四列.
```
def conflict(state, nextX):
    nextY = len(state)
    for i in range(nextY):
        '''
           判断新增加的皇后和之前的是否冲突:
           冲突条件1: 在同一行 state[i] = state[nextX]
           冲突条件2: 在同一对角线 abs(state[i] - state[nextX]) = nextX - i
        '''
        if state[i] == state[nextX] or abs(state[i] - state[nextX]) == nextX - i:
            return True
    return False

#如果只剩下一个皇后没有位置,遍历所有可能的位置并且返回最后皇后可以放得位置.
def queens(num, state):
    if len(state) == num - 1:
        for pos in range(num):
            if not conflict(state, pos):
                yield pos
```

- 递归解法 - 程序从前面的皇后得到了包含位置信息的元组,并且要为后面的皇后提供当前皇后的每种合法的位置信息.
```
def queensrecurision(num=8,state=()):
    for pos in range(num):
        if not conflict(state, pos):
            if len(state) == num - 1:
                yield (pos,)
            else:
                #将当前皇后可能的位置传给下一个皇后
                for result in queensrecurision(num, state + (pos,)):
                    yield (pos,) + result
```

## 总结

- property()方法可以将内部方法获取的值转换为在外界调用时类的内部的属性.

- 构造方法__init()__.

- 实现list或者dict只需要在类中定义__getitem__和__setitem__特殊方法,或者直接继承list和dict.

- 迭代器需要在类中实现__next__和__iter__方法. 迭代器可以在循环语句中使用.

- 生成器是指包含yield函数的类,注意生成器中代码的运行顺序. 单纯调用生成器只会获得生成器的instance, 需调用next或者send方法来使生成器类运行.生成器类运行时会运行至yield然后暂停函数,下次执行时会从上一个yield处继续.


# Python的标准库

## 模块

- 模块实际上就是将各个代码分类,达到复用的目的. 在当前类中调用代码时,\_\_name__会被设定为\_\_main__,但是如果在其他类中引用了当前类并调用时,当前类的__name__会被设定为类名.简单来说,只有当前类的name会被定义为__main__.
```
# helloworld.py
def helloworld():
    print('hello world')

$ testhelloworld.py
import helloworld
helloworld.helloworld()
```

- 目录的列表可以在sys模块中的path变量中找到: 通常来说,只要将模块放入site-packages之中,所有的程序就都能将其导入.
```
import sys, pprint #pprint提供更加智能的打印输出
pprint.pprint(sys.path) # Documents/Github/ThinkinginPython/venv/lib/python3.6/site-packages
```

- 为了组织好模块,你可以将它们分组为包(package).包基本就是另外一类模块,不同的地方就是它们能够包含其他模块.为了让Python能够识别包,包必须包含一个命名为\_\_init__.py的文件(模块).
```
import drawing #引drawing包,这时只有__init__模块中的内容是可用的,shapes & colors不可用
import drawing.colors # colors模块可用,但是必须通过全名drawing.colors使用
from drawing import shapes # shapes模块可用,并且可以通过短名shapes进行调用
```

## 探究模块

- dir()可以查看模块中包含的内容, 显示所有特性.
```
import copy
print(dir(copy)) #输出全部特性
print(copy.__all__) #输出module的全部public interface
#如果没有设置__all__,用import *语句默认将会导入模块中所有不以下划线开头的全局名称
```

- help()可以获取函数的帮助.
```
print(help(copy)) #输出函数基本帮助
print(range.__doc__) #输出函数文档,如果有字符串定义在函数或者类开头的话
```

## 标准库

- sys module可以让你能够访问与Python解释器联系紧密的变量和函数. 在Windows中,由os.system或者os.startfile启动了外部程序,Python程序仍然继续运行.但在Unix中,程序则会中止,等待os.system命令完成.
```
import sys
print(sys.argv) #命令行参数,包括脚本名称
print(sys.modules) #映射模块名字到载入模块的字典
print(sys.path) #查找模块所在的目录的目录名列表
print(sys.platform) #平台标识符 - 操作系统或虚拟机
print(sys.stdin) #标准输入流
print(sys.stdout) #标准输出流
print(sys.stderr) #标准错误流
sys.exit(0) #退出当前程序
```

- os module提供了访问多个操作系统服务的功能.
```
import os
print(os.environ) #对环境变量进行映射
print(os.system('cd')) #在子shell中执行操作系统命令
print(os.sep) #路径中的分隔符
print(os.pathsep) #分隔路径中的分隔符
print(os.linesep) #行分隔符
print(os.urandom(5)) #返回n字节的加密强随机数据

import webbrowser
webbrowser.open('http://www.google.com')
```

- fileinput module可以让你能够轻松遍历文本文件的所有行. fileinput.input是其中最重要的函数, 它能够返回用于for循环遍历的对象(迭代器). 如果不想使用默认行为(fileinput查找需要循环遍历的文件),那么可以给函数提供(序列形式的)一个或多个文件名. 同时,还可以使用inplace参数将函数进行原地处理 - 对于要访问的每一行,需要打印出替代的内容,以返回到当前的输入文件中.
```
import fileinput
print(fileinput.input()) #遍历多个输入流的行
print(fileinput.filename()) #当前文件名
print(fileinput.lineno()) #当前累计行数 - 更换文件时并不会清除数量
print(fileinput.filelineno()) #当前文件总行数 
print(fileinput.isfirstline()) #是否为文件内的第一行
print(fileinput.isstdin()) #检查最后一行是否来自sys.stdin
print(fileinput.nextfile()) #关闭当前文件,移动到下一个
print(fileinput.close()) #关闭文件
```


## Set & Heap & Queue

- Set是**由序列或者其他可迭代对象**构建的. set('只接受可迭代对象或者序列').
```
a = frozenset([5,6,7,8]) #使用frozenset()可以将集合变为不可变对象 - 可做dict中的key.
```

- Heap在Python中可以看成是优先队列的一种. 使用优先队列能够以任意的顺序增加对象,并且能在任何时间找到最小的元素 - 比列表的min方法有效率.
```
from heapq import *
from random import shuffle

'''
heappush(heap,x) 将x入堆
heappop(heap) 将堆中最小的元素弹出
heapify(heap) 将heap中的属性强制应用到任意一个列表
heapreplace(heap,x) 将堆中最小的元素弹出,同时将x入堆
nlargest(n, iter) 返回iter中第n大的元素  - 比排序方法更有效率
nsmallest(n, iter) 返回iter中第n小的元素 - 比排序方法更有效率
'''
data = list(range(10))
shuffle(data)

heap = []
for i in data:
    heappush(heap, i)
print(heap)
heapreplace(heap, 0.2)
print(heap)
```

- Deque(double - ended queue),双端队列. 顾名思义,可以在队列两端加入值或者弹出值. 通常appendleft指在队列左边加入值,append为队列右边入值. 同样,pop默认为队列右边末尾的值,而popleft则为队列第一个值. 双端队列还有一个方法为rotate(int) - 能够使队列中的item进行左移或者右移. 注意, **extendleft使用的可迭代对象中的元素将会反序出现在双端队列中.**
```
from collections import deque
q = deque(range(5))
print(q) # deque([0, 1, 2, 3, 4])
temp = list(range(5))
q.extendleft(temp)
print(q) # deque([4, 3, 2, 1, 0, 0, 1, 2, 3, 4])
```

## Time

- Python可以用元组表示时间. 例如(2008, 1, 21, 12, 2, 56, 0, 21, 0)表示为2008年1月21日12时2分56秒，星期一，并且是当年的第21天(无夏令时).

    1. 第7个索引表示儒历日 - 通常为当年的第多少天 范围1-366.
    2. 第8个索引表示夏令时 - 值为1, 0和-1.

```
import time
t = (2008, 1, 21, 12, 2, 56, 0, 21, 0)
print(time.asctime(t)) #将时间元组转化为字符串
print(time.localtime(time.time())) #将秒数转换为日期元组,以本地时间为准
print(time.mktime(t)) #将时间元组转化为本地时间
print(time.sleep(1)) #休眠secs秒
print(time.strptime('Mon Jan 21 12:02:56 2008')) #将字符串解析为时间元组
print(time.time()) #当前时间 
```

## Random

- random模块用来产生随机数,同Java一样,该模块生成的实际上都是伪随机数.若想使用真的随机,应该使用os module的urandom()或者random module内的SystemRandom类,可以让结果接近真正随机.
```
import random
print(random.random()) #返回0-1(包括1)直接的随机数
print(random.getrandbits(233)) #以长整型形式返回n个随机位 - 处理真正的随机事物,比如加密
print(random.uniform(0,100)) #返回随机实数n,其中 a < n <= b
print(random.randrange(0, 10, 2)) #返回随机整数,最后一个参数为step,可以用来控制获得奇数或偶数
print(random.choice([1,2,5,6])) #从一个序列中返回随机元素
a = [1,2,5,6]
random.shuffle(a) #对于给定的一个序列进行原地shuffle,返回none
print(a)
print(random.sample(a, 2)) #从给定的序列中随机返回n个元素
```
## Shelve

- Shelve为Python提供了一个简单的存储方案,调用open()方法是会获得一个shelf对象,操作该对象的方法可以参考普通的dict,只不过dict的键为string.保存的时候调用close()方法即可. open()方法中的writeback参数设为true时,数据变化将会发生回写, 同时读取或赋值到shelf的数据结构都会保存在缓存中.只不过这样操作会比较占用内存.
```
import shelve
s = shelve.open('a file path')
s.get('a').append('d')
s.get('a') #d并没有加进去因为该值并没有保存
#当在shelve对象中进行查找时,这个对象都会根据已经存储的版本进行重新构建. 若想改变值请使用赋值操作.

a = shelve.open('a file path', True)
a.get('a').append('d') #writeback为True,当数据发生变化时会发生回写
a.close()
```

## 正则表达式

- 通配符(wildcard),‘.’.

- 转义(escape), '\\' 或使用原始字符串r'\'.

- 字符集(character set), '[a-zA-z]'. 反转字符集,在开头使用^, '[^abc]'.

- 管道符号(|)也可以称作subparttern,使用方法类似或, 'python|perl'.

- (pattern)?可以出现一次或不出现, (pattern)*可以出现0次或多次, (pattern)+可以出现一次或多次, (pattern){m,n}可以出现m-n次.

- 如果不是字符集的话,正常的正则表达式以^开头,以$结尾. 在使用match函数时,如果要对整个字符串进行匹配,一定要在正则表达式中以$结尾,该种方法将会将匹配顺延至字符串末尾.

- re.escape()函数可以将字符串中所有可能被解释为正则运算符的字符进行转义.
```
from re import *
print(escape('www.string.text.com'))
#www\.string\.text\.com
```


# 文件和流

## Python中的IO

- open(name[ , mode[ , buffering]]). name为文件名称,mode(模式)和buffering(缓冲)为可选参数.

    1. model - 文件模式,默认为读模式.但是通过改变参数也可以变成写模式.'+'读和写都是允许的-可以添加到其他模式中. ‘r’读模式, 'w'写模式, 'a'追加模式, 'b'二进制模式 - 可添加到其他模式中使用.
    2. buffering - 缓冲. 0或者False,代表无缓冲,所有读写操作都直接针对硬盘. 1或者True, 有缓冲, 代表Python将使用内存来代替硬盘,此时程序会更快,只有使用flush()或者close()才会更新硬盘. 大于1的数字代表缓冲区的大小(单位是字节), -1或者任何负数代表使用默认的缓冲区大小.

```
f = open('resource/IOTEST.txt', 'r+', True)
print(f.read()) #可以在read中指定数字来标注需要读取的字符数
f.write('this is more test\r')
f.seek(int) #找到文件中字符的index并且从此处开始
f.tell() #可以获得当前游标的位置
f.write('123455') #从当前游标的位置开始写入,注意,会覆盖后面的部分
f.readline(int) #int代表当前行中你想获取的字符数.
f.readlines() #可以读取一个文件中的所有行并将其作为list返回.
```

- pipeline(管道), ‘|’. 管道符号可以将一个命令标准输出和下一个命令的标准输出连在一起.

- 换行符, mac(\r), windosw(\r\n).

- writelines()可以接受一个字符串的列表或者任何可迭代的对象或者序列,它会把所有接受的对象写入文件(stream)中. write()则可以使用为一行一行的写入.
```
import os
f = open('resource/IOTEST.txt', 'r+', True)
f.read() #read中可以加int表示要读取的字符数

f.write('\rthis is a test')
f.writelines(['\rthis is', '\ra stupid', '\r12345165151'])
``` 
- 在文件写入过后必须要使用close()进行关闭文件,如果因为使用缓存而又因为一些原因程序崩溃,那么你所写入的数据将会丢失 - 切记使用close(). Python中有两种方法可以关闭一个文件或流.在任何情况下,能关闭文件直接关闭文件,如果不能便使用flush().
```
#使用try close
try:
    f = open('resource/IOTEST.txt', 'r+', True)
    f.write('123545')
finally:
    f.close()

#使用with statement,文件或者流将在语句结束后自动关闭.即便存在异常也会关闭.
with open('resource/IOTEST.txt', 'r+', True) as testclose:
    testclose.write()
    testclose.read()

```

- with statement其实是使用了一种很通用的结构,允许使用所谓的上下文管理器(context manager). 该管理器支持\_\_enter__和\_\_exit__这两个方法的对象. 文件可以被用作与这种上下文管理器, enter方法返回文件本身, exist方法关闭文件.

## 使用File的基本方法

- 按字节处理. 最常见的对文件内容进行迭代的方法就是在while loop中使用read方法.
```
with open('resource/somefile.txt', 'r+', True) as testclose:
    while True:
        line = testclose.readline()
        if not line:
            break
        print(line)
```

- 如果文件不是很大,那么可以一次性读取文件中的所有内容,但是文件如果很大的话并不建议这样做.

- 如果文件很大时, readlines()方法将会占用很多的内存.通常我们选择用while loop和readline()方法来代替.也可以使用**懒惰行迭代**的方法,懒惰是因为它只是读取实际需要的部分.
```
import fileinput
for line in fileinput.input('resource/somefile.txt'):
    print(line)
```

- 文件迭代器. 在Python的近几个新版本当中, 文件对象被转化成一种可以迭代的对象,这就意味着可以使用for循环来迭代对象.
```
f = open('resource/somefile.txt')
for line in f:
    print(line)
f.close()
```


# Database

## Python的数据库接口

- 任何支持2.0版本的DB API的数据库模块都必须定义3个描述模块特性的全局变量.

    1. apilevel - 所使用的Python DB API版本.
    2. threadsatety - 模块的线程安全等级. 0表示线程完全不共享模块,3表示模块式完全线程安全的. 1表示线程本身可以共享模块,但不对应连接共享. 如果不使用多线程,则可以忽略这个变量.
    3. paramstyle - 在SQL查询中使用的参数风格. 在执行多次类似查询的时候,参数是如何被拼接到SQL查询中的.

- 为了使用底层的数据库系统,首先必须连接到它,使用connect(). connect()函数的常用参数如下.

    1.  dsn - 数据源名称,给出该参数表示数据库依赖.
    2. user - 用户名
    3. password - 用户密码
    4. host - 主机名
    5. database - 数据库名

- connect()函数将返回一个connect对象,该对象表示目前和数据库的会话. 支持的方法如下.

    1.  close() - 关闭和数据库的连接, 连接对象和游标都不可使用.
    2. commit() - 如果支持的话就将提交挂起的事物, 否则不做任何事.
    3. rollback() - 回滚挂起的事物(可能不可用).
    4. cursor() - 返回连接的游标对象.


- cursor()方法将返回一个游标对象.通过游标执行SQL查询并检查结果. 常用的方法如下.

    1. callproc(name[, params]) - 使用给定的名称和参数调用已命名的数据库过程.
    2. close() - 关闭游标,之后游标不可用.
    3. execute(oper[, params]) - 执行一个SQL,可能带有参数.
    4. executemany(oper, pseq) - 对序列中的每一个参数集执行SQL操作.
    5. fetchone() - 把查询的结果集中的下一行保存为序列, 或者None.
    6. fetchmany([size]) - 获取查询结果集中的多行,默认尺寸为arraysize.
    7. fetchall() - 将所有(剩余)的行作为序列的序列.
    8. nextset() - 跳至下一个可用的结果集(可选).
    9. setinputsizes(sizes) - 为参数预先定义内存区域.
    10. setoutputsizes(size[, col]) - 为获取的大数据的值设定缓冲区尺寸.


- cursor对象的特性(property).

    1. description - 结果列描述的序列,只读.
    2. rowcount - 结果中的行数,只读.
    3. arraysize - fetchmany中返回的行数,默认为1.

- 底层数据库可能需要一些特殊的类型的值,如果需要和Python的类型对应,需要使用对应的api将Python的对象封装成数据库支持的类型.
```
import sqlite3
with sqlite3.connect('somedatabase.db') as db:
    db.cursor()
    db.commit()
```

- 使用cursor执行select语句时, 先使用execute()执行语句,然后用fetchall()获得结果.

- 如果不是大量的数据或者有大型数据库服务器的需求,可以使用sqlite3 in Python. 该数据库为Python内置的服务器,可以满足一些小型的需求. 使用比较方便.


# 网络编程

## Socket

- 网络编程中的一个基本组建是socket, 它是一个信息通道, 两端各有一个程序. 这些程序可能位于(通过网络相连的)不同的计算机上, 通过socket向对方发送信息. 在Python中, 大多数网络编程都隐藏了模块socket的基本工作原理, 不与socket直接交互.

    1. Socket分为两类, Server socket和Client socket. 创建了服务端的socket后,就可以让它等待连接请求的到来. 这样, 它将在某个网络地址(由IP和Port组成)处监听, 直到客户端的socket与之建立连接. 这样客户端和服务端就能与之通信了.

    2. 通常客户端的socket处理起来比较容易,因为它只需要完成任务并且断开连接. 而服务端的socket略微复杂一些, 因为服务器必须准备随时处理客户端的连接,并且还必须可以处理多个连接.

    3. Python中的socket存在于socket module中, 实例化socket时最多可指定三个参数: 一个地址族(默认为socket.AF_INET), 一个是流socket(socket.SOCK_STREAM,默认设置)还是数据报socket(socket.SOCK_DGRAM), 还有一个是协议(默认值0). 创建普通socket时,不需要提供任何参数.

    4. 服务器端的socket先调用bind方法绑定host IP和Port, 再使用listen来监听特定的地址. 然后, 客户端的socket就可以连接至服务器了, 办法是调用方法connect并提供调用方式bind时指定的地址(在服务器端, 可使用方法socket.gethostname获取当前机器的主机名). 这里的地址是一个格式为(host ,port)的tuple, host为主机名(如www.example.com), 而port是端口号(一个整数). 方法listen接受一个参数 - 待办任务清单的长度(即最多可有多少个连接在队列中等待接纳, 到达这个数量后将开始拒绝连接).

    5. 服务器socket开始监听后, 就可以接受客户端的连接了, 这是使用方法accept来完成的. 这个方法将阻断(同步式的)到客户端连接的到来, 然后返回一个格式为(client, address)的tuple. 其中client是一个客户端的socket, 而address是前面的地址解释过的地址. 服务器能以其认为合适的方式处理客户端的连接,然后再次调用accept以接着等待新连接的到来. 这通常是在一个无限循环中完成的.

    6. 为了在服务端和客户端传输数据, socket主要有两种方法: send和recv. 发送数据可使用send并提供一个字符串; 而接收数据, 可以调用recv并指定最多接收多少个字节的数据.

    7. 在Linux或者Unix系统中, 需要有管理员权限才能使用1024以下的port, 这些较小的port都是提供标准服务使用的. 比如port 80是提供web服务器的使用.

```
import socket
import threading

class MyThread(threading.Thread):
    def run(self):
        create_server()

def create_server():
    s = socket.socket()
    host = socket.gethostname()
    port = 1234
    s.bind((host, port))
    s.listen(5)
    while True:
        c, addr = s.accept()
        print('Got connection from', addr)
        message = 'Thanks for your messages'
        c.send(message.encode())
        c.close()

def create_client():
    s = socket.socket()
    host = socket.gethostname()
    port = 1234
    s.connect((host, port))
    print(s.recv(1024))

def main():
    t = MyThread()
    t.start()
    create_client()

if __name__ == '__main__':
    main()

```

## urllib和urllib2

- 使用这两个module几乎可以像打开本地文件一样打开远程文件,差别是只能使用读取模式, 以及使用模块urllib.request中的函数urlopen, 而不是open. urlopen返回的类似于文件的对象支持方法close, read, readline和readlines - 并且支持迭代.
```
from urllib.request import urlopen
webpage = urlopen('https://www.google.com')
```

- 如果想获取或者下载远程的文件, 可以使用urlretrieve()方法. 这个方法不返回一个类似于文件的对象, 而返回一个格式为(filename, headers)的元组, 其中filename是本地文件的名称(由urllib自动创建), 而headers包含一些有关远程文件的信息. 如果要给下载的副本指定文件名, 可以通过第二个参数.
```
from urllib.request import *
webpage = urlretrieve('https://www.google.com')
print(webpage) 
# ('C:\\Users\\yutianli\\AppData\\Local\\Temp\\tmp2lsl3d81', <http.client.HTTPMessage object at 0x02B60330>)
urlcleanup() #清理文件
```

## SocketServer

- 简单的socket server并不难编写. 但是如果想要创建更加复杂的服务器, 则需要使用服务器的模块. SocketServer是Python服务器框架的基石库. 这个框架包括 - BaseHTTPServer, SimpleHTTPServer, CGIHTTPServer, SimpleXMLRPCServer和DocXMLRPCServer. 这几种服务器都是在基本服务器上添加了各种功能.

- SocketServer包含4个基本的服务器: TCPServer(支持TCP socket), UDPServer(支持UDP socket), 以及更难懂的UnixStreamServer和UnixDatagramServer. 使用SocketServer编写服务器时, 大部分代码都位于**请求处理器**中. 每当服务器收集到客户端的连接请求, 都会实例化一个请求处理程序, 并对其调用各种处理方法来处理请求. 具体调用哪些方法取决于使用的服务器类和请求处理程序类, 还可以从这些请求处理类派生出子类, 从而让服务器调用一组自定义的处理方法.

- 基本请求处理程序类BaseRequestHandler将所有操作都放在同一个方法中—服务器调用的方法handle. 这个方法可以通过属性self.request来访问客户端的socket. 如果处理的是流(使用TCPServer时很可能如此), 可以使用StreamRequestHandler类, 它包含另外的两个属性: self.rfile(用于读)和self.wfile(用于写). 可以使用这两个类似的文件对象和客户端进行通信.

```
from socketserver import TCPServer, StreamRequestHandler
import socket

class Handler(StreamRequestHandler):
    def handle(self):
        addr = self.request.getpeername()
        print('Got connection from', addr)
        self.wfile.write('Thank you for connecting'.encode()) #str is not supported, encode it first.

server = TCPServer((socket.gethostname(),1234), Handler)
server.serve_forever()
```

## 多个连接

- Python中也有支持同时处理多个连接的方式,并且不会阻塞任何程序. 主要有三种方式, 分别为forking(分叉), 线程化(threading)和异步I/O(asynchronous I/O). 通过结合使用SocketServer中的混合类和服务器类, 很容易实现forking和线程化. 但是forking和线程化也有缺点, forking占用的资源较多, 并且在客户端很多时的伸缩性不佳(但只要客户端数量适中, forking在UNIX和Linux系统中的效率很高. 如果有多个CPU, 效率就更高了); 而线程化容易带来同步问题.

- forking是一个Unix术语. 对进程进行分叉时, 基本上是复制它, 而这样得到的两个进程都将从当前位置开始继续往下执行, 并且每个进程都有自己独立的内存副本(变量等-内存空间). 原来的进程为父进程, 复制的进程为子进程. 进程能够判断它们到底是原始进程还是子进程(通过查看函数fork的返回值), 因此能够执行不同的操作. **在forking服务器中, 对于每个客户端的连接, 都将通过分叉创建一个子进程. 父进程继续监听新连接, 而子进程负责处理客户请求. 客户端请求处理结束后, 子进程直接退出. 由于forking出来的进程并行地运行, 因此客户端无需等待.** Windows暂时不支持forking.

```
from socketserver import TCPServer, ForkingMixIn, StreamRequestHandler
import socket

class Server(ForkingMixIn, TCPServer):
    pass

class Handler(StreamRequestHandler):
    def handle(self):
        addr = self.request.getpeername()
        print('Got connection from', addr)
        self.wfile.wirte('Thank you for connecting'.encode())

server = Server((socket.gethostname(), 1234), Handler)
server.serve_forever()
```

- 线程可以解决forking占用资源较多的问题, 因为所有位于同一个进程中的线程将共享内存. 这将会减少占用的资源, 但是同时又带来了另一个问题, 你必须确保每个线程不会彼此干扰或者同时修改一项数据, 否则会引起混乱. 因此使用线程时必须去处理同步问题.
```
from socketserver import TCPServer, ThreadingMixIn, StreamRequestHandler
import socket
class Server(ThreadingMixIn, TCPServer):
    pass

class Handler(StreamRequestHandler):
    def handle(self):
        addr = self.request.getpeername()
        print('Got connection from', addr)
        self.wfile.write('Thank you for connecting'.encode())

server = Server((socket.gethostname(), 1234), Handler)
server.serve_forever()
```
- 还有一种避免线程和forking的方法是使用Stackless Python, 它是一个能够快速而轻松地在不同上下文之间切换的Python版本, 支持一种类似于线程的并行方式, 名为微线程, 其伸缩性比真正的线程高的多.

- 在较低层次使用异步I/O较为困难, 其基本机制是在模块select中使用select方法. 不过现在有用于实现异步I/O的高级框架, 使其能够通过简单而抽象的接口使用可伸缩性的强大机制. 该框架由asyncore和asynchat组成.

- 使用select和poll实现异步I/O, 当服务器与客户端通信时, 来自客户端的数据可能时断时续. 如果使用了forking和线程化, 就可以解决这种问题, 因为总有一个进程或线程在处理数据, 其他进程或线程可继续处理其客户端. **另一种做法是只处理当前正在通信的客户端, 你甚至无需不断监听, 只需监听后将客户端加入队列即可**.这就是框架asyncore/asynchat和Twisted采取的方法. 这种功能的基石是函数select或poll, poll的伸缩性更高, 但只有Unix支持(Windows不支持).

- select()方法需要3个序列作为它的必选参数,此外还有一个可选的以秒为单位的超时时间作为第四个参数. 3个序列用于输入, 输出以及异常情况. 如果没有给定超时时间, select会阻塞(处于等待状态), 直到其中的一个文件描述符已经为行动做好了准备; 如果给定了超时时间, select最多阻塞给定的超时时间, 如果给定的超时时间是0, 那么就给出一个连续的poll(既不阻塞). select的返回值是三个序列, 每个代表相应参数的一个活动子集.
```
import socket, select
s = socket.socket()
host = socket.gethostname()
port = 1234
s.bind((host, port))
s.listen(5)
inputs = [s]
while True:
    rs, ws, es = select.select(inputs,[],[]) #如果没有输入会阻塞
    for r in rs:
        if r is s: #如果是新的服务器,那么就进行监听
            c, addr = s.accept()
            print('Got connection from ', addr)
            inputs.append(r) #将新连接过来的client放入输出list
        else: #如果client端准备就绪,则尝试读取数据
            try:
                data = r.receive(1024)
                disconnected = not data
            except socket.error:
                disconnected = True
            if disconnected:
                print(r.getpeername(), ' disconnected')
                inputs.remove(r)
            else:
                print(data)
```

- poll()方法使用起来比select简单一些. 在调用poll时, 会得到一个poll对象. 然后就可以使用poll对象的register方法注册一个文件描述符(或者是带有fileno方法的对象). 注册后可以使用unregister方法移除注册的对象. 注册了一些对象(比如socket)之后, 就可以调用poll方法并且得到一个(fd, event)的tuple列表(可能为空), 其中fd是文件描述符, event则告诉你发生了什么.event是一个位掩码(bitmask), 意思是它是一个整数,这个整数的每个位对应不同的事件. 所以我们需要在这里使用位操作符(&).
```
import socket, select
s = socket.socket()
host = socket.gethostname()
port = 1234
s.bind((host, port))
s.listen(5)
fdmap = {s.fileno(): s}
p = select.poll()
p.register(s)
while True:
    events = p.poll()
    for fd, event in events:
        if fd == s.fileno():
            c, addr = s.accept()
            print('Got connection from ', addr)
            p.register(c)
            fdmap[c.fileno()] = c
        elif event & select.POLLIN:
            data = fdmap[fd].recv(1024)
            if not data:
                print(fdmap[fd].getpeername(), ' disconnected')
                p.unregister(fd)
                del fdmap[fd]
        else:
            print(data)
```

## 小结

- Socket以及Socket模块, 进程之间进行通信的信息通道可能会通过网络来通信. socket模块给提供了对客户端和服务端socket的低级访问功能. 服务器端socket会在指定的地址监听客户端的连接, 而客户机是直接连接的.

- urllib和urllib2, 这些模块可以在给出数据源url时让从不同的服务器读取和下载数据. urllib模块是一个简单一些的实现, 而urllib2是可扩展的, 而且十分强大.

- SocketServer框架, 这是一个同步的网络服务器基类. 位于标准库中, 使用它可以很容易地编写服务器. 它甚至用CGI支持简单的Web服务(Http). 如果想同时处理多个连接, 可以使用forking或者threading来处理混入类.

- select和poll, 这两个函数让你可以考虑一组连接并且能找出已经准备好读取或者写入的连接. 这就意味着能为通过时间片轮转来为几个连接提供服务. 看起来就像是同时处理几个连接. 尽管代码可能更复杂一点, 但在伸缩性和效率上要比threading或者forking好得多.

- Twisted, to be continued. 


# Python & Web

## 屏幕抓取

- 屏幕抓取是程序下载网页并且从中提取信息的过程. 如果你想在你的程序中使用在线的网页所包含的信息, 就可以使用这个技术. 如果所涉及的网页是动态的那就更有用了, 也就是说网页是不停变化的. 不然就要每次都下载网页, 然后手动提取信息才行.  Example: 使用urllib获取网页的html源码, 然后使用正则表达式提取信息. 简单的urllib抓取有很多问题, 两个比较好的解决方案是使用程序调用**Tidy(Python库),** 进行XHTML解析. 另一个是使用**Beautiful Soup库,** 它专门为了屏幕抓取设计.
```
#抓取google首页中有多少google单词出现.
from urllib import request
import re
p = re.compile('google')
text = request.urlopen('http://www.google.com').readline()
text = bytes.decode(text)
print(p.findall(text).__sizeof__()) # 408
```

## Tidy和XHTML解析

- XHTML是HTML最新的方言, 是XML的一种形式. 由于浏览器可以宽容的接受许多种类的html错误并且尽可能的对其进行解析, 导致网络上许多的html网页的格式有很多问题, 这对于屏幕抓取是非常不方便的.

- XHTML和HTML的主要区别有两个, 首先XHTML对于显示关闭所有元素要求更加严格, 必须使用开始和关闭tag才能关闭一段元素. 这种行为让XHTML变得非常容易解析, 因为可以直接告诉程序什么时候进入或离开各种元素. 另外一个区别就是XHTML是XML的一种, 所以可以对他使用XML工具, 比如XPATH.

- Tidy是用来修复不规范且有些随意的html文档的工具. 它能够以相当智能的方法修复一般的错误, 做那些你不愿意做的事情. 它也是可设置的, 也可以打开或者关闭各种修改选项.

- 使用HTMLParser(Python标准库)可以处理XHTML. 当使用XHTML时, 需要生成一个它的子类, 并且对handle_starttag和handle_data之类的事件处理方法进行override.
```
from urllib import request
from html.parser import HTMLParser

class Scraper(HTMLParser):
    in_h3 = False
    in_link = False

    def handle_starttag(self, tag, attrs):
        attrs = dict(attrs)
        if tag == 'h3':
            self.in_h3 = True

        if tag == 'a' and 'herf' in attrs:
            self.in_link = True
            self.chunks = []
            self.url = attrs['href']

    def handle_data(self, data):
        if self.in_link:
            self.chunks.append(data)

    def handle_endtag(self, tag):
        if tag == 'h3':
            self.in_h3 = False
        if tag == 'a':
            if self.in_h3 and self.in_link:
                print(self.chunks, self.url)
                self.in_link = False
                
text = request.urlopen('https://www.google.com').read()
text = bytes.decode(text)
parser = Scraper()
parser.feed(text) #解析HTML
parser.close()
```

## Beautiful Soup

- BeatifulSoup类可以对获得的HTML进行缩进和解析, 并且可以通过使用类似访问property的方式访问HTML中的各种tag. 使用该类来进行屏幕抓取非常方便以及容易.
```
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')
print(soup.title)
print(soup.head)
print(soup.title.string)
print(soup.find_all('a'))
```

## 使用CGI技术创建动态网页

- CGI(Common Gateway Interface)通用网关接口,是网络服务器可以将查询(一般来说是web表单)传递到专门的程序中并且在网页上显示结果的标准机制. 它是创建web应用程序的一种简便的方法, 无需编写特殊的应用服务器.

- Python CGI程序设计的关键工具是cgi模块, cgitb是另外一个在CGI脚本开发过程中的有用的模块. 使用CGI脚本可通过网络访问(运行)之前, 需要将它们放到网络服务器可以访问的地方, 并且加入pound bang行, 设置合适的文件权限.

- Step 1: 准备网络服务器, 一般来说就是把需要的文件放置在网络上. CGI程序也必须放在通过网络可以访问的目录中, 并且必须将他们标识为CGI脚本. 这样网络服务器就不会将普通的源代码作为网页处理. 有两个方法可以实现这个功能:

    - 将脚本放在叫做cgi-bin的子目录中

    - 把脚本文件扩展名改为.cgi

- Step 2: 加入Pound Bang行, 当把脚本放在正确位置后, 需要在脚本的开始处增加pound bang行. 通常来说, 只要把下面这行加到脚本开始处就可以了: 注意, 它一定要是第一行. 如果不能正常工作, 需要查看Python可执行文件的确切位置.
```
#!/usr/bin/env python‘

# Windows: 一定要使用Python的全路径
#!C:\Python22\python.exe 
```

- Step 3: 设置文件权限, 最后一件事是设置合适的文件权限.要确保每个人都可以读取和执行脚本文件, 但是还要确保只有你可以写入. 修改文件权限的UNIX命令是chmod. 只要运行以下的命令即可(如果脚本叫做somescript.cgi), 使用普通的用户账户, 或者为这类Web任务特别建立的账户: 在做好所有这些准备后, 应该能将脚本作为网页打开并且执行.
```
chmod 755 somescript.cgi
```

- 一般来说不允许CGI脚本修改计算机上的任何文件. 如果想要它修改文件, 必须显式地给它设置相应的权限. 这时有两个选项, 如果有root(系统管理员)权限的话, 那么可以为你的脚本创建一个用户账户, 改变需要修改的文件的所有权. 如果没有root权限, 则可以为文件设置文件权限, 这样系统上的所有用户(包括网络服务器用来运行CGI脚本的用户)都被允许写文件.
```
chmod 666 editable_file.txt #使用666可能会导致潜在危险
```

## Web服务

- Web服务一般应用于很高层次的抽象, 它使用HTTP(Web协议)作为底层协议, 上面则是更多面向内容的协议. 比如WSDL, 是一种描述通过服务可获得何种方法以及方法的参数和返回值等内容的XML格式的语言.

- RSS代表富站点摘要(Rich Site Summary), RDF站点摘要或简易信息聚合(Really Simple Syndication), 是在XML中列出新闻项目的最简单的格式. RSS文档(或称feed)更像服务而不是静态文档的原因是它们是定期更新的, 甚至可以对它们执行动态计算.

- 如今常用的Web服务: XML-RPC, 可以看做是使用简单的XML在系统之间通信, 现在已经不常用了. REST,  使用JSON在系统之间通信, 效率很高非常优雅, 但安全性一般. SOAP, 使用标准化格式的XML在系统之间通信, 比较庞大, 但是安全性较高. 


# Test

## 测试驱动编程

- 在Java这种编译型语言中, 通常的代码构建流程是编辑, 编译和运行. 有许多问题可以再编译过程中被发现. 而Python是没有编译步骤的, 所以只需要编辑和运行即可. 运行程序就是测试的过程.

- 单元测试是指对程序的各个部分依次建立测试, 这是非常重要的. 现在有许多开发团队推崇测试驱动编程, 也就是所谓的先测试, 后开发. 测试驱动编程的理念是从编写测试程序开始, 然后编写可以通过测试的程序. 测试程序就是程序的需求说明, 它帮助程序员在开发程序的时候不偏离需求. 这种思想不仅在初始开发程序时非常有用, 在以后扩展和维护代码的时候也很有用.

- 自动化测试除了在编写程序上给予巨大的帮助外, 还可以避免在修改过程中引入错误. 代码的一部分改变的时候, 很有可能会引入几个不可预料的错误. 如果程序设计的足够好(使用大量的抽象和封装), 改变产生的影响就应该是局部的, 并且只影响一小部分代码.

- Coverage(覆盖度)是测试中一个非常重要的概念. 保证你可以尽量多的提供测试代码以及测试数据, 可以对程序有着更高的覆盖度. 事实上遵守测试驱动编程的原则就是确保拥有好的测试覆盖度的方法之一. 如果能保证在写函数之前就编写了测试代码, 那么就可以肯定每个函数都被测试过了.

## 测试的四个步骤

1. 指出需要的新特性, 可以记录下来, 然后为其编写一个测试.

1. 编写特性的骨架代码, 这样程序就可以运行, 不存在任何语法等方面的错误, 但是测试会失败. 看到测试失败是很重要的, 这样就可以确定所编写的测试代码是正常运行的. 在试图让测试成功之前, 先要看到它失败.

1. 为特性的骨架编写dummy code, 能够满足测试的要求就行. 不用准确地实现功能, 只要保证测试可以通过即可.

1. 重构代码, 使它完成自己应该做的事.

- Python标准库中的测试工具分为unittest(通用测试框架)和doctest(检查文档用的, 但也可以用来编写单元测试).

## doctest

- 交互式解释器的会话可以是将文档字符串(docstring)写入文档的一种有用的形式.

```
def square(x):
    '''
    Squares a number and returns the result:
    >>> square(2)
    4
    >>> square(4)
    16
    '''
    return x*x

if __name__ == '__main__':
    import doctest, this
    doctest.testmod(this) #testmod函数检查模块的文档字符串, 以及函数的文档字符串, 并运行所有存在的测试. 
```

## unittest

- Python中的unittest和Java中的JUnit有着异曲同工之处. unittest.main函数负责实际运行测试, 它会实例化所有TestCase的子类, 运行所有名字以test开头的方法. 如果定义了setUp和tearDown的方法, 它们就会在运行每个测试方法之前和之后执行, 这样就可以用这些方法提供测试代码数据的初始化和清理代码. 这些被称为测试装置(test fixture).

```
import unittest, Test.Simple_doctest

def product(x, y):
    return x*y

class ProducTestCase(unittest.TestCase):

    def testIntegers(self):
        for x in range(-10,10):
            for y in range(-10,10):
                p = Test.Simple_doctest.square(x, y)
                self.failUnless(p == x*y, 'Integer multiplication failed')
                
    def testFloats(self):
        for x in range(-10,10):
            for y in range(-10,10):
                x = x/10.0
                y = y/10.0
                p = Test.Simple_doctest.square(x, y)
                self.failUnless(p == x*y, 'Float multiplication failed')
    
if __name__ == '__main__':
    unittest.main()
```

## 探索(Probulating)程序

- 除了测试以外, Python中还有源代码检查以及性能分析(profiling). 源代码检查是一种寻找代码中普通错误或者问题的方法(有点像Java中的编译器, 但功能远不止如此). 性能分析则是查明程序到底可以跑多快的方法. 简单来说, 单元测试可以让程序工作, 源代码检查可以让程序更好, 最后, 性能分析会让程序运行的更快.

- Pychecker和PyLint可以用来检查源代码.

- 标准库中有一个profile的分析模块, 可以使用字符串参数调用它的run方法来分析程序的效率. 或者使用更快的嵌入式C语言的版本, hotshot.


# 程序设计

## 配置文件

- 可以将所有固定的常量放置到一个配置文件中, 这样用户就可以比较容易的直接更改程序中常量的值.
```
from configparser import *

CONFIGFILE = 'config.txt'
config = ConfigParser()
config.read(CONFIGFILE)

print(config.get('numbers', 'pi'))
print(config.get('messages', 'greeting')) #第一个参数是所要获取的区块, 第二个参数则是config的名字
```

## 日志记录

- Python标准库中的日志文件就可以记录程序运行中的各种数据.
```
import logging
logging.basicConfig(level=logging.INFO, filename='mylog.log')
logging.info('program start')
logging.info('Trying to divide 1 by 0')
print(1/0)

```


# 用户密码与Python

用户名密码是用户身份认证的关键, 它的重要性不言而喻. 通常大多数用户都会在各种应用中使用相似的密码, 一旦某一个应用的密码被破解, 很可能用户的其他应用也相对危险。通常有以下几种模式来存储和保护用户密码.

- **密码明文直接存储中字系统中**: 这种方法下的密码安全性比系统本身还低, 管理员能查看所有用户的密码明文. 除非你要盗取用户密码, 否则不能使用这种方式.

- **密码明文经过转换后再存储:** 与直接存储明文的方式区别不大, 任何知道或破解转换方法的人都可以逆转得到密码明文.

- **密码经过对称加密后再储存**: 密码明文的安全性等同于加密密钥本身的安全性. 对称加密的密钥可同时用于加密和解密. 一般它会直接出现在加密代码中, 破解的可能性相当大. 而且一般系统管理员很可能知道密钥, 进而算出密码原文.

- **密码经过非对称加密后再储存**: 密码的安全性等同于私钥的安全性. 密码明文经过公钥加密. 要还原密码明文, 必须要相应的私钥才行. 因此只要保证私钥的安全, 密码明文就安全. 私钥可以由某个受信任的人活着机构来掌管, 身份验证只需要公钥就可以了. 实际上，这就是HTTPS/SSL的理论基础. 这里的关键是私钥的安全, 如果私钥泄露, 那么密码明文就危险了.

因此，**密码最好是以不可还原明文的方式来保存**. 通常利用**哈希算法的单向性**来保证明文以不可以还原的有损方式进行存储. 安全性由**低到高**依次为:

- 使用自己独创的哈希算法对密码进行哈希, 存储哈希过的值. - 独创对理论要求很高, 通常不建议.

- 使用MD5或SHA-1哈希算法 - MD5和SHA-1已经被破解. 虽然不能还原明文, 但很容易找到能生成相同哈希值的替代明文, 而且这两个算法速度较快, 暴力破解相对省时, 建议不使用.

- 使用更加安全的SHA-256等成熟算 - 更加复杂的算法增加了暴力破解的难度. 但是如果遇到简单的密码, 用彩虹字典的暴力破解法, 也很快可以找到密码原文.

- 加入随机salt的哈希算法 - 密码原文(或经过hash后的值)和随机生成的salt字符串混淆, 然后再进行hash, 最后把hash值和salt值一起存储. 验证密码的时候只要用salt再与密码原文做一次相同步骤的运算, 比较结果与存储的hash值就可以了. 这样一样哪怕是简单的密码, 进行salt混淆后产生的也是很不常见的字符串, 根本不会出现在彩虹字典中. salt越长暴力破解难度越大. 具体的hash过程也可以进行若干次的迭代, 虽然迭代会增加碰撞概率, 但也增加暴力破解的资源消耗. 就算真破解了, 黑客掌握的也只是这个随机salt混淆过的密码, 用户的原始密码依然安全. 
