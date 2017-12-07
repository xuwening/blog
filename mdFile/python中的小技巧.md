# python中的小技巧

python非常强大，它的强大离不开自身的内置库以及更强大的第三方库，但简洁的语法更值得称道，正因为python简单直接的表达方式吸引了众多的爱好者，才能发展出蓬勃的第三方库。

正因为python的简单直接，所以我几乎没有写过python类，如果不是用python做项目工程，更建议去`用`python，而不是去`学`。对我而言，python代码写的好不好不重要，也不太注重性能问题，只要能实现需要的结果就好，跑一个结果出来少花一秒种意义不大。

但一些常用技巧还是有用的，掌握一些实用性的代码技巧，更能提高工作效率。当然，这些技巧能掌握当然是好的，掌握不了去速查也没什么打不了。这里就总结一些我使用过的比较实用的技巧。


#### range

```python
In [1]: range(10)  ##生成10以内的序列
Out[1]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [2]: range(1, 10, 3)  ##生成1到10，步长为3的序列
Out[2]: [1, 4, 7]
```

#### 查看ASCII值

用C语言的有时需要查某字符的ASCII，或者反查

```python
In [5]: ord('a')
Out[5]: 97

In [6]: chr(97)
Out[6]: 'a'
```

#### 不同进制转换

```python
In [8]: bin(10)  ##二进制
Out[8]: '0b1010'

In [9]: hex(10)  ##十六进制
Out[9]: '0xa'

In [10]: oct(10)  ##八进制
Out[10]: '012'
```

#### 字符串与列表转换


```python
In [13]: list('hello world')
Out[13]: ['h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd']

In [14]: 'hello world'.split(' ')
Out[14]: ['hello', 'world']

In [20]: ''.join(['h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd'])
Out[20]: 'hello world'

## 自定义分隔符
In [23]: '-'.join(['hello', 'world'])
Out[23]: 'hello-world'
```

#### 多项赋值

```python
In [10]: a, b, c = [1, {2,3}, 4]

In [22]: a,*b, c = [1,2,3,4,5,6]  ## b为[2, 3, 4, 5]

## 函数可以返回多个值
In [14]: def func():
    ...:     return 1, 2, 3

In [15]: a, b, c = func()

## 两变量值互换
In [19]: a, b, = b, a
```

#### 翻转字符串

利用列表切片的特性

```python
In [26]: ''.join(list('hello world')[::-1])
Out[26]: 'dlrow olleh'

## 判断是否是回文字符串
'abccba'[::-1] == 'abccba'
```

#### 排序字符串

```python
In [27]: "".join(sorted('bcadefg'))
Out[27]: 'abcdefg'
```

#### 集合操作

```python
In [99]: a = set([1, 2, 'aaa', 65, 'bad', 'badd', 4])

In [100]: b = set([1, 2, 'ccc', 'ddd'])

In [101]: a & b
Out[101]: {1, 2}

In [102]: a | b
Out[102]: {65, 1, 2, 4, 'ddd', 'bad', 'badd', 'aaa', 'ccc'}

In [103]: a ^ b
Out[103]: {65, 'ddd', 4, 'bad', 'badd', 'ccc', 'aaa'}

In [104]: a - b
Out[104]: {65, 4, 'bad', 'badd', 'aaa'}
```

#### 随机选择

```python
In [30]: random.choice(["aaa", "bbb", "ccc", "ddd"])
Out[30]: 'bbb'
```

#### lambda表达式

```python
In [31]: filter(lambda x: x%2, [1,2,3,4,5])
Out[31]: [1, 3, 5]
```

#### in运算符

```python
In [32]: 'abc' in 'aababc'
Out[32]: True
```

#### 三元表达式

```python
x if exp else y
exp and 'yes' or 'no'
```

#### string常量字符串

有了这些常量字符串，对于字符串的判断操作以及构造字符串区间非常方便

```python
In [81]: string.digits
Out[81]: '0123456789'

In [82]: string.octdigits
Out[82]: '01234567'

In [83]: string.hexdigits
Out[83]: '0123456789abcdefABCDEF'

In [84]: string.ascii_letters
Out[84]: 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'

In [85]: string.whitespace
Out[85]: ' \t\n\r\x0b\x0c'

In [86]: string.printable
Out[86]: '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~ \t\n\r\x0b\x0c'
```

#### 列表表达式

这是我最喜欢的特性，可以用极简短的语句实现强大的功能

```python
In [11]: [x**2 for x in range(10) if x%2==0] ##10以内的偶数的平方
Out[11]: [0, 4, 16, 36, 64]
```

#### 列表去重

```python
list(set([1,1,2,3]))
```

## 同时遍历两个集合

```python
for v1, v2 in zip(list1, list2)
```

#### 用两个列表构造字典

```python
In [34]: dict(zip(['red', 'blue'],[0x110000, 0x000011]))
Out[34]: {'blue': 17, 'red': 1114112}
```

#### 统计字符出现频率

```python
In [39]: Counter('hello world')
Out[39]: Counter({' ': 1, 'd': 1, 'e': 1, 'h': 1, 'l': 3, 'o': 2, 'r': 1, 'w': 1})

## 统计频次最高的前两名
In [47]: Counter('hello world').most_common(2)
Out[47]: [('l', 3), ('o', 2)]
```

#### url编解码

```python
In [49]: urllib.parse.quote('http://www.baidu.com/index.html?args=中国')
Out[49]: 'http%3A//www.baidu.com/index.html%3Fargs%3D%E4%B8%AD%E5%9B%BD'

In [50]: urllib.parse.unquote('http%3A//www.baidu.com/index.html%3Fargs%3D%E4%B8%AD%E5%9B%BD')
Out[50]: 'http://www.baidu.com/index.html?args=中国'
```

#### 函数可变参数

```python
def func(*args):
		pass
func(1,2,3,4)

def func(**args):
		pass
func(a=1, b=2)
```

#### 函数参数的扩展

```python
In [66]: def f1(a,b,c=10,d=10):
    ...:     print(a,b,c,d)
    
In [67]: a1 = [1,2,3,4]
In [68]: f1(*a1)  ##将列表展开
1 2 3 4

In [74]: a2 = {'a': 2, 'b': 4}
In [75]: f1(**d)  ##将字典展开
2 4 10 10
```

#### 平台相关

```python
os.sep  ## 文件分隔符，windows为\，unix为/
os.linesep  ## 换行符，windows为\r\n，unix为\n

```

#### 遍历文件夹及文件

```python
os.listdir(os.getcwd()) #列出当前目录所有文件夹和文件

for dirpath, dirnames, filenames in  os.walk(path) #遍历文件夹和文件
```

#### 路径操作

```python
In [93]: os.path.basename('/usr/home/test.txt') ## 取文件名
Out[93]: 'test.txt'

In [94]: os.path.dirname('/usr/home/test.txt')  ## 取路径
Out[94]: '/usr/home'

In [95]: os.path.splitext('/usr/home/test.txt')  ## 分割路径及文件后缀
Out[95]: ('/usr/home/test', '.txt')

In [96]: os.path.split('/usr/home/test.txt')  ## 分割路径及文件名
Out[96]: ('/usr/home', 'test.txt')
```

#### 调用shell命令

```python
import commands 
ret, output = commands.getstatusoutput(cmd) 
```

#### 查看帮助

```python
## 查看sys名称空间
dir(sys)

## 查看当前解释器全局空间
dir()

## 查看所有内置函数
dir(__builtin__)

## 查看模块或函数详细手册
sys?
help(sys)
sys.__doc__
```

