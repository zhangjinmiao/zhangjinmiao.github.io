[TOC]



# Python学习

## 简介

Python是跨平台语言

### 优点

1. **代码库丰富**：基础代码库完善，覆盖了网络、文件、GUI、数据库、文本等大量内容，被形象地称作“内置电池（batteries included）”。第三方代码库也挺多。
2. **“优雅”、“明确”、“简单”**，Python的哲学是简单优雅，写既少又易懂的代码。

### 缺点

1. **运行速度慢**，因为Python是解释型语言，代码在执行的时候会一行一行的翻译成CPU能理解的机器码，这个过程比较耗时，而C语言是运行前直接编译成CPU能执行的机器码，所以非常快。但是有时候人类对于0.01秒和1秒来说是感觉不出来的，所以不必担心。
2. **代码不能加密**，发布程序时，发布的是源代码，这是解释型语言的弊端，而C语言和Java一样，作为编译型语言，发布程序时只发布编译后的结果即可。（C语言Windows上常见的.exe文件；java发布.class文件）。

### 应用

1. **网络应用**：包括网站和后台服务等。许多大型网站就是用Python开发的，例如YouTube、[Instagram](http://instagram.com/)，还有国内的[豆瓣](http://www.douban.com/)。
2. **日常小工具**：包括系统管理员需要的脚本任务等等
3. **包装其他语言开发的程序**，方便使用


## 安装



## 运行

### 直接运行python文件

1. windows下不行，linux和Mac下可以，但需在文件开头加入注释

   >\# !/usr/bin/env python3
   >
   >print('hello,world')

2. 通过命令给hello.py赋执行权限

   > $ chmod a+x hello.py

3. 然后就可以运行了

### Python的交互模式和直接运行`.py`文件的区别

直接输入`python`进入交互模式，相当于启动了Python解释器，但是等待一行一行地输入源代码，每输入一行就执行一行。

直接运行`.py`文件相当于启动了Python解释器，然后一次性把`.py`文件的源代码给执行了，没有机会以交互的方式输入源代码的。

用Python开发程序，完全可以一边在文本编辑器里写代码，一边开一个交互式命令窗口，在写代码的过程中，把部分代码粘到命令行去验证，事半功倍。



### Python代码运行助手

learning.py是一个python代码助手，启动后看到`Ready for Python code on port 39093...`表示运行成功，不要关闭命令行窗口，最小化放到后台运行即可。

然后到https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432523496782e0946b0f454549c0888d05959b99860f000  ，输入代码即可。



## python 基础



python是大小写敏感的。

### 数据类型和变量

python支持的数据类型有`整数`，`浮点数`，`字符串`，`布尔值`，`空值`，`常量`

**字符串:**

1. 字符串，如需转义，使用`\`，为了简化，Python还允许用`r''`表示`''`内部的字符串默认不转义
2. 如果字符串内部有很多换行，用`\n`写在一行里不好阅读，为了简化，Python允许用`'''...'''`的格式表示多行内容

**空值:**

空值是Python里一个特殊的值，用`None`表示。`None`不能理解为`0`，因为`0`是有意义的，而`None`是一个特殊的空值。



### 字符串和编码

#### 字符串

在最新的Python 3版本中，字符串是以Unicode编码的，也就是说，Python的字符串支持多语言

> - `ord()`函数：获取字符的整数表示 
>
> ```python
> >>> ord('A')
> 65
> >>> ord('中')
> 20013
> ```
>
> ​
>
> - `chr()`函数：把编码转换为对应的字符
>
> ```python
> >>> chr(66)
> 'B'
> ```
>
> ​
>
> - Python对`bytes`类型的数据用带`b`前缀的单引号或双引号表示：x = b'ABC'
>
> 如果`bytes`中只有一小部分无效的字节，可以传入`errors='ignore'`忽略错误的字节
>
> ```python
> >>> b'\xe4\xb8\xad\xff'.decode('utf-8', errors='ignore')
> '中'
> ```
>
> 要计算`str`包含多少个字符，可以用`len()`函数：
>
> ```python
> >>> len('ABC')
> 3
> >>> len('中文')
> 2
> ```
>
> 换成`bytes`，`len()`函数就计算字节数：
>
> ```python
> >>> len(b'ABC')
> 3
> >>> len(b'\xe4\xb8\xad\xe6\x96\x87')
> 6
> >>> len('中文'.encode('utf-8'))
> 6
> ```
>
> **1个中文字符经过UTF-8编码后通常会占用3个字节，而1个英文字符只占用1个字节**
>
> ​

一般在python文件开头，加入注释

```python
#!/usr/bin/env python3   告诉Linux/OS X系统，这是一个Python可执行程序，Windows系统会忽略这个注释；
# -*- coding: utf-8 -*-  告诉Python解释器，按照UTF-8编码读取源代码
```

要确保文本编辑器正在使用UTF-8 without BOM编码

#### 格式化

| 占位符  | 替换内容   |
| ---- | ------ |
| %d   | 整数     |
| %f   | 浮点数    |
| %s   | 字符串    |
| %x   | 十六进制整数 |

转义：%%，输出%；

```python
>>> 'growth rate: %d %%' % 7
'growth rate: 7 %'
```

不确定类型时，使用%s，%s会把任何数据类型转换为字符串

```python
>>> 'Age: %s. Gender: %s' % (25, True)
'Age: 25. Gender: True'
```

**字符串`format()`方法**

```python
>>> 'Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125)
'Hello, 小明, 成绩提升了 17.1%'
```



### 使用list和tuple

**list**

```python
>>>classmates = ['Michael', 'Bob', 'Tracy']

// 获取长度,下标从`0`开始
>>> len(classmates)
3
// `-`开头，获取倒数第几个
>>> classmates[-1]
'Tracy'

// 添加元素
>>> classmates.append('Adam')
>>> classmates
['Michael', 'Bob', 'Tracy', 'Adam']

// 插入指定位置
>>> classmates.insert(1, 'Jack')
>>> classmates
['Michael', 'Jack', 'Bob', 'Tracy', 'Adam']

// 删除末尾元素
>>> classmates.pop()
'Adam'
>>> classmates
['Michael', 'Jack', 'Bob', 'Tracy']

// 删除指定位置元素pop(i)
>>> classmates.pop(1)
'Jack'
>>> classmates
['Michael', 'Bob', 'Tracy']

// 替换元素，直接赋值给对应下标位置
>>> classmates[1] = 'Sarah'
>>> classmates
['Michael', 'Sarah', 'Tracy']

// list中套list
>>> s = ['python', 'java', ['asp', 'php'], 'scheme']
>>> len(s)
4

// 更直观一点
>>> p = ['asp', 'php']
>>> s = ['python', 'java', p, 'scheme']

获取php，可以p[1]或s[2][1]


// 空list长度为0
>>> L = []
>>> len(L)
0
```



**tuple**

有序列表叫元组，初始化后不可变，没有append()，insert()，取值同list。

```python
// 定义
>>> t = (1, 2)
>>> t
(1, 2)

// 定义空的tuple
>>> t = ()
>>> t
()

// 定义1各元素时，后边加`,`
>>> t = (1,)
>>> t
(1,)



```

### 条件判断

使用`if`，判断是true的话，执行缩进的代码，注意if 后面的`:`，如果判断为true，即使后边还有true，也不执行。



```python
age = 3
if age >= 18:
    print('your age is', age)
    print('adult')
else:
    print('your age is', age)
    print('teenager')
```

语法：

```python
if <条件判断1>:
    <执行1>
elif <条件判断2>:
    <执行2>
elif <条件判断3>:
    <执行3>
else:
    <执行4>
```

### 循环

#### `for x in ...`循环

把每个元素代入变量`x`，然后执行缩进块的语句

```python
names = ['Michael', 'Bob', 'Tracy']
for name in names:
    print(name)
```

**`range()`函数**

生成一个整数序列



**`list()`函数**

转换为list

```python
>>> list(range(5))
[0, 1, 2, 3, 4]
```

#### `while`循环

```python
sum = 0
n = 99
while n > 0:
    sum = sum + n
    n = n - 2
print(sum)
```



#### break

提前退出循环

```python
n = 1
while n <= 100:
    if n > 10: # 当n = 11时，条件满足，执行break语句
        break # break语句会结束当前循环
    print(n)
    n = n + 1
print('END')
```



#### continue

跳过当前的这次循环，直接开始下一次循环。

```python
n = 0
while n < 10:
    n = n + 1
    if n % 2 == 0: # 如果n是偶数，执行continue语句
        continue # continue语句会直接继续下一轮循环，后续的print()语句不会执行
    print(n)
```



### 使用dict和set

#### dict

dict全称dictionary，在其他语言中也称为map，使用键-值（key-value）存储，具有极快的查找速度。

```python
>>> d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
>>> d['Michael']
95
```

##### 判断key是否存在

通过`in`判断key是否存在

```python
>>> 'Thomas' in d
False
```

通过`get()`方法，如果key不存在，可以返回`None`，或者自己指定的value：

```python
>>> d.get('Thomas')
>>> d.get('Thomas', -1)
-1
```

##### pop(key)：删除key

```python
>>> d.pop('Bob')
75
>>> d
{'Michael': 95, 'Tracy': 85}
```

**总结：**

***dict***：

1. 查找和插入的速度极快，不会随着key的增加而变慢；

2. 需要占用大量的内存，内存浪费多。

   注：dict可以用在需要高速查找的很多地方，dict的key必须是**不可变对象**，在python中，字符串，整数都是不可变对象

***list:***

1. 查找和插入的时间随着元素的增加而增加；
2. 占用空间小，浪费内存很少。



#### set

key不可重复，无须，构建set时，使用list作为输入集合

```python
>>> s = set([1, 2, 3])
>>> s
{1, 2, 3}
```

##### add(key)：添加元素

```python
>>> s.add(4)
>>> s
{1, 2, 3, 4}
>>> s.add(4)
>>> s
{1, 2, 3, 4}
```



##### remove(key)：删除元素

```python
>>> s.remove(4)
>>> s
{1, 2, 3}
```



## 函数







## python内置方法

input()：客户端输入函数



