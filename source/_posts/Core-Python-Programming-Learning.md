---
title: Python核心编程学习笔记
date: 2020-02-06 16:00:00
tags:
  - 读书笔记
  - 学习笔记
categories:
  - Python
---
# Python核心编程
** 第二版学习笔记 **

## 快速入门

Python是一门解释性语言，作为对比JavaScript也是解释性语言。学习的前提需要到官网(https://www.python.org) 下载并安装最新版本。Python源文件一般用.py作为扩展名。

```python
#script.py
#井号是注释符号

#定义一个变量
aValue = 'Hello World!'

#打印一个变量
print(aValue)

#定义一个函数
def print_string(astr):
    '打印一个字符串'  #文档字符串(注释)
    print(astr)

#调用函数
print_string(aValue)
```

将示例文件另存为：script.py, 在Windows命令行中执行python script.py,  可以看到如下信息。

```
Hello World!
Hello World!
```


## Python基础

### 语句和语法
* 井号(#)之后的字符为Python的注释;
* 换行(\n)是标准的行分隔符，一般一行一条语句;
* 反斜杠(\)表示继续上一行，一行过长的语句使用反斜杠（\）可以分解成多行；
* 分号(;)将两个语句连接在一起，如果需要将两条语句放到同一行中，需要使用分号隔开；
* 冒号(:)将代码块的头和体分开，像if、while、def、class复合语句，首行以关键字开始，以冒号（：）结束，该行之后的一行或多行构成代码组，首行和代码组合起来称为一个子句；
* 语句(代码块)用缩进的方式体现，不同的缩进深度分割不同的代码块，缩进相同的一组语句构成一个代码块，建议每个缩进使用4个空格表示；
* Python文件以模块的形式进行组织。

### 变量赋值
等号是主要的赋值操作符。
增量操作符和C语言类似。
Python使用下划线作为变量前缀和后缀时为特殊变量。
Python支持多元赋值，示例如下：
~~~python
x, y, z = 1, 2, 'Hello World!' 
~~~


### 标识符
标识符定义规则：第一个字符必须是字母或下划线，剩下的字符可以说字符、数字和下划线。标识符大小写敏感。

```python
#不要使用dict、list、file、bool、str、input、len等内置类型作为变量名。
```

## Python对象

### 基本数据类型
* 数字
* Integer 整型
* Boolean 布尔型
* Long integer 长整型
* Floating point real number 浮点型
* Complex number 复数类型
* String 字符串
* List 列表
* Tuple 元组
* Dictionary 字典

所有类型对象的类型都是type，这个概念有点类似于C#中所有类型都是由object派生出的一样。
Python中的特殊类型是None, 表示Null对象或NoneType. None类型类似于C#中的void, 值类似于null. None的布尔值总是为False.

### 操作符
对象值可以使用：>、<、==、>=、<=、!=等操作符进行比较。
对象引用方式根据赋值方式，有所不同。
~~~python
#示例
#a1和a1引用的相同对象
a1 = a2 = 123

#b1和b2引用的不同对象
b1=123
b2=123 
~~~
使用is、is not可以比较是否引用的同一对象。
~~~python
#结果：True
a1 is a1 

#结果：False
b1 is b2 
~~~

布尔类型操作符：not、and、or.

### 内置函数

| 函数           | 功能                                    |
| -------------- | ------------- |
| cmp(obj1,obj2) | 比较obj1和obj2，根据比较结果返回整型值 |
| repr(obj)      | 返回一个对象的字符表示，类似C#中ToString方法  |
| str(obj)       | 返回可读性好的对象字符串，类似C#中ToString带格式的方法 |
| type(obj)      | 获取对象的类型，返回相应的type对象 |

### 类型分类

| 数据类型 | 存储类型 | 更新模型 | 访问模型 |
| -------- | -------- | -------- | -------- |
| 数字 | 标量 | 不可更改 | 直接访问 |
| 字符串 | 标量 | 不可更改 | 顺序访问 |
| 列表 | 容器 | 可更改 | 顺序访问 |
| 元组 | 容器 | 不可更改 | 顺序访问 |
| 字典 | 容器 | 可更改 | 映射访问 |

标量：一个保存单个字面对象的类型，类似于C#中的值类型和字符串。
容器：可存储多个对象（对象可以有不同的类型）的类型，类似C#集合。
可变类型：允许值被更新,每次修改后新值替换旧值。类似C#引用类型。
不可变类型：不允许值被更改，每次修改后使用新的值替代；旧值被丢弃，等待垃圾回收器处理回收对象。类似C#值类型。
直接访问:对数值直接进行访问，类似C#中栈。
顺序访问：可对容器按索引进行访问元素，类似C#中索引。
映射访问：元素无序存放，通过唯一Key访问，类似C#中哈希。

## 数字

Python的整型包含：布尔型、标准整型、长整型。布尔型只有两个取值：True和False, 对应整型的1和0. 标准整型和C# Int32和Int64表示范围相同，在32位机器上范围为Int32, 64位则为Int64. 长整型的范围超过Int64, 具体范围与（虚拟）内存大小有关，长整型一般在数值后面加一个大写的L. Python会自动转换整型和长整型。
复数由实数部分和虚数部分构成，虚数语法：real+imagj. 实数和虚数部分都是浮点类型，虚数部分后缀必须为j或J.
两个连续的星号(\*\*)表示幂运算.
~~~python
3 ** 2 #结果为：9
~~~
整除操作符：//, 也称“地板除”，除法不管操作数为何种数值类型，总是舍去小数部分，返回数字序列总别真正商小的最接近的数字。
~~~python
1//2       #结果为：0
1.0//2.0   #结果为：0.0
-1//2      #结果为：-1
~~~

数值工厂函数

| 函数                                   | 操作                |
| -------------------------------------- | ------------------ |
| bool(obj)                              | 返回obj对象的布尔值 |
| int(x [,base\])                        | 将x转换为一个整型   |
| long(x [,base\] )                      | 将x转换为一个长整型 |
| float(x)                               | 将x转换到一个浮点型 |
| complex(real [,imag\]) or complex(str) | 创建一个复数        |

仅适用于整型的内置函数

| 函数        | 操作                                                      |
| ----------- | --------------------------------------------------------- |
| hex(num)    | 将数字转换成十六进制数并以字符串形式返回                  |
| oct(num)    | 将数字转换成八进制数并以字符串形式返回                    |
| chr(num)    | 将ASCII值的数字转换成ASCII字符，范围：0<=num<=255         |
| ord(chr)    | 传入长度为1的字符串，返回相应的ASCII整数值或Unicode整数值 |
| unichr(num) | 将一个整数转换为Unicode字符                               |

## 序列：字符串、列表和元组

### 序列

序列元素顺序

```
            0      1      2           N-2     N-1
Sequence    ☐     ☐      ☐   ● ● ●   ☐      ☐ 
           -N   -(N-1)  -(N-2)        -2     -1
```


序列类型操作符

| 序列操作符         | 作用                               |
| ------------------ | ---------------------------------- |
| seq[index]         | 获取下标为index的与元素            |
| seq[index1:index2] | 获取下标从index1到index2之间的元素 |
| seq * expr         | 序列重复expr次                     |
| seq1 + seq2        | 连接序列seq1和seq2                 |
| obj in seq         | 判断元素obj是否包含在seq中         |
| obj not in seq     | 判断元素obj是否不包含在seq中       |

切片操作符：[]、[:]、[::]

```python
#切片示例
s = 'abcdefgh'

s = 'abcdefgh'
print(s)           #abcdefgh 打印原始字符串
print(s[::-1])     #hgfedcba 字符串翻转操作
print(s[::2])      #aceg     隔一个取一个的操作
print(s[1:3])      #bc       获取索引下标1到2之间的元素
print(s[3:])       #defgh    获取索引下标从3开始的所有元素
print(s[-3:])      #fgh      获取索引-3开始之后的所有元素
print(s[:3])       #abc      从右往左获取前3个元素
print(s[:-3])      #abcde    从右往左获取前5个元素（长度8 - 3）
```



### 字符串

Python中使用单引号或双引号创建字符串，使用del语句删除字符串。

字符串进行比较操作时，按照ASCII值的大小进行比较。

字符串格式化操作符：%.

字符串格式化符号

| 格式化符 号 | 转换方式                             |
| :---------- | :----------------------------------- |
| %c          | 格式化字符及其ASCII码                |
| %s          | 格式化字符串                         |
| %d          | 格式化整数                           |
| %u          | 格式化无符号整型                     |
| %o          | 格式化无符号八进制数                 |
| %x          | 格式化无符号十六进制数               |
| %X          | 格式化无符号十六进制数（大写）       |
| %f          | 格式化浮点数字，可指定小数点后的精度 |
| %e          | 用科学计数法格式化浮点数             |
| %E          | 作用同%e，用科学计数法格式化浮点数   |
| %g          | %f和%e的简写                         |
| %G          | %F 和 %E 的简写                      |
| %p          | 用十六进制数格式化变量的地址         |

格式化操作符辅助指令

| 符号  | 作用                                                         |
| :---- | :----------------------------------------------------------- |
| *     | 定义宽度或者小数点精度                                       |
| -     | 用做左对齐                                                   |
| +     | 在正数前面显示加号( + )                                      |
| <sp>  | 在正数前面显示空格                                           |
| #     | 在八进制数前面显示零('0')，在十六进制前面显示'0x'或者'0X'(取决于用的是'x'还是'X') |
| 0     | 显示的数字前面填充'0'而不是默认的空格                        |
| %     | '%%'输出一个单一的'%'                                        |
| (var) | 映射变量(字典参数)                                           |
| m.n.  | m 是显示的最小总宽度,n 是小数点后的位数(如果可用的话)        |


```python
#格式示例
print('%x' % 100)                 #64
print('%#x' % 100)                #0x64
print('%c' % 100)                 #d
print('value is %d' % 100)        #value is 100
print('value is %s' % '100')      #value is 100
```

字符串格式化的高级用法是使用字符串模板，引入Template模块对象，调用substitute()和safe_substitute()方法进行格式化。

```python
from string import Template
s = Template('Today is ${year}.${month}.${day}!')
print (s.substitute(year=2020, month=2, day=6))         #Today is 2020.2.6!
```

注意：字符串中的模板必须与变量一一对应，否则会出现KeyError异常。而使用safe_substitute()在缺少key时可以将字符串原封不动的打印出来。

Unicode字符串在字符串前加大写U或小写u. 加前缀u表示告诉Python后面的字符串要编码成Unicode字符串。

```
print(u'\u8881')     #袁
```

在控制台界面中可以使用input(~~raw_input()~~为python2中的函数python3中不存在)函数进行输入字符串，类似于C#中的Console.ReadLine().

使用反斜杠(\)加一个单一字符可以表示一个特殊字符。

三引号语法允许一个字符串跨多行，字符串中可以包含换行符、制表符以及其他特殊字符。语法类似C# @"".

```python
#示例
print('''Hello

World''')
```

### 列表

列表使用方括号([])进行定义。

```python
aList = [123, 'Hello', 'abc', 123.456, ['abcd', 100]]
print(aList)           #[123, 'Hello', 'abc', 123.456, ['abcd', 100]]

#访问列表中值
print(aList[0])        #123

#更新列表中值
aList[1] = 'abcdefg'
print(aList)           #[123, 'abcdefg', 'abc', 123.456, ['abcd', 100]]

#删除列表元素
del aList[0]
#或使用remove()方法
aList.remove('abc')
print(aList)           #['abcdefg', 123.456, ['abcd', 100]]

#删除整个列表
del aList
```

### 元组

元组的功能和列表类似，元组使用圆括号进行定义。元组属于不可变类型，类似于C#中只读集合。元组中的元素不可跟新和删除，只能删除整个元组。

```python
aTuple = (123, 'Hello', 'abc', 123.456, ['abcd', 100])

#访问元组中的元素
print(aTuple[1])        #Hello

#删除元组
del aTuple
```

## 字典和集合类型

### 字典

字典是Python语言中唯一的映射类型，可以简单理解为Hash集合。字典初始化有两种方式：{}和dict()函数。字典和JavaScript中的对象类似，都是使用大括号({})将key和value值进行包裹。

```python
#字典示例
dict1 = {}
dict2 = {'name': 'wenwen', 'year': '2020', 'age': 3}
dict3 = dict((['x', 1], ['y', 2]))

#循环遍历字典中的值
for key in dict2:
    print('key=%s, value=%s' % (key, dict2[key]))
    #key=name, value=wenwen
    #key=year, value=2020
    #key=age, value=3

#获取特定元素
print(dict2['name'])      #wenwen

#更新字典
dict2['year'] = 2000      #更新已有key值
dict2['sex'] = 'girl'     #新增key value

#删除字典元素
del dict2['year']           #删除key为"year"的项
dict2.clear()               #删除dict2中所有的项
del dict2                   #删除整个dict2字典
```

不允许一个键对应多个值，当键冲突时，取最后（最近）的赋值。键必须是可哈希的，一般使用数字、字符串作为字典键。键必须是可哈希的原因是：解释器调用哈希函数，根据键中的值来计算存储位置，如果键是可变对象，它的值可能发生变化，导致哈希函数无法映射到原有的地址，无法获取数据。

字典获取特定元素时，若key不存在则会抛出：”name 'dict' is not defined“异常。使用setdefault()方法可以避免key不存在的问题。

```python
print(dict2.setdefault('name', 'yuan'))      #wenwen
print(dict2.setdefault('height', 100))       #100
```

### 集合

集合元素是一组无序排列的可哈希的值。集合有两种类型：可变集合(set)和不可变集合(frozenset)。可变集合不能作为字典的键，也不能作为其他集合的元素。创建集合只能使用集合的工厂方法set()和frozenset().

集合类型操作符包括(所有集合类型)：联合(|)、交集(&)、差补/相对补集(-)、对称差分(^)。

```python
s = set('abcdefgh')
t = frozenset('python')

print(s)            #{'g', 'h', 'b', 'c', 'd', 'e', 'f', 'a'}
print(t)            #frozenset({'p', 'n', 'h', 'o', 'y', 't'})

#集合类型操作符

#联合(|)
print(s | t)        #{'g', 'p', 'n', 'h', 'b', 'c', 'd', 'e', 'f', 'o', 'y', 'a', 't'}

#交集(&)
print(s & t)        #{'h'}

#差补/相对补集(-)
print(s - t)        #{'g', 'b', 'c', 'd', 'e', 'f', 'a'}

#对称差分(^)
print(s ^ t)        #{'g', 'p', 'b', 'f', 'd', 'y', 'a', 'n', 'c', 'o', 'e', 't'}
```

仅适用于可变集合的操作符：|=、&=、-=、^=，功能和示例代码类似，不再做演示。

## 条件和循环

if语句语法

```python
if expression:
    exp_true_suite
else:
    exp_false_suite
```

Python支持elif关键字，用于表示else-if.

```python
if expression1:
    expr1_true_suite
elif expression2:
    expr2_true_suite
                #.
        		#.
elif expressionN:
    exprN_true_suite
else:
    none_of_the_above_suite    
```

条件表达式（三元操作符）语法：X if C else Y

```
#C、C++、C#、Java 三元操作符示例
int x = 1;
int y = 2;
int smaller = x < y ? x : y;

#Python 三元操作符示例
x, y = 1, 2
smaller = x if x < y else y
```

while语句语法

```python
while expression:
    suite_to_repeat
```

for语句类似于C#中foreach，语法如下：

```python
for iter_var in iterable:
    suite_to_repeat
```

使用enumerate()函数可获取索引和项。

```python
#enumerate()函数示例
aList = [123, 'Hello', 'abc', 123.456, 'World']
for i, item in enumerate(aList):
    print('Index:%d Item:%s' % (i, item))    
    #Index:0 Item:123
	#Index:1 Item:Hello
	#Index:2 Item:abc
	#Index:3 Item:123.456
	#Index:4 Item:World
```

range()函数语法：range(start, end, step=1), range()会返回一个包含所有k的列表，start <= k < end, 从start到end,  k每次递增step. step不可为0.

```python
aList = range(3, 19, 5)
for item in aList:
    print('Value:%d' % item)
    #Value:3
    #Value:8
    #Value:13
    #Value:18
```

Python中break和continue语句用法和其他语言类似。

Python提供pass语句，它不做任何事情，用于开发调试。

```python
#pass语句示例
if age == 5:
    pass
else:
    pass
```

Python支持迭代器，为类序列对象提供一个类序列的接口，类似C#中IEnumerator.

```python
#iter()函数示例
aList = [123, 'Hello', 'abc', 123.456, 'World']
item = iter(aList)
print(item.__next__())        #123
print(item.__next__())        #Hello
print(item.__next__())        #abc

#备注：原文中使用的next()方法，在Python3中不存在。
```
