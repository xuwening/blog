# javascript数组与字符串的妙用


## 字符串与数组转换

这个功能是我最喜欢的功能，也是所有脚本都应该必备的功能之一：

```js
//string --> array
const arr = 'hello world'.split('')
//array --> string
const str = arr.join('')
```

当然，我们还可以这样转换：

```js
//这两种方式没有办法指定分隔符，所以没有split好用
const arr = [...'hello world']
const arr = Array.from('hello world')
const arr = Object.values('hello world')
```

它能干什么呢？我们知道计算机处理中，一般字符串处理占绝大比重，而字符串本身库函数处理能力较弱，现在可以无缝转换为数组，意味着将数组处理的功能都可以施加在字符串身上，无形中将js处理字符串的能力提升好大一截儿。

数组有什么特性函数呢？`map reduce filter sort every some reverse`等等。好了，现在字符串也可以使用这些功能函数了。

#### 对字符串进行加密

每个字符向后移动两位：a -> c， b -> d，这样就实现简单的加密，无法直接看出明文。

```js
//hello world被转换成了jgnnq"yqtnf
'hello world'.split('').map(c => String.fromCharCode(c.charCodeAt() + 2)).join('')  
```

代码说明：

要将字符向后移动两位，只需要将字符的`ASCII`值加`2`即可，这里我们使用了`charCodeAt`和`fromCharCode`进行字符与ASCII进行转换。因此，只需要对每个字符遍历，然后执行加法操作即可，这里就利用了数组的`map`函数。


#### 找出字符串中最大单词长度

例如字符串：`life was like a box of chocolates`，字符串长度最大的就是`chocolates`，长度为10。

```js
const s = `life was like a box of chocolates`
s.split(' ').reduce((pre, cur) => pre.length > cur.length ? pre : cur ).length
```

代码说明：

这次转换不是按照字符，而是按照`word`进行转换，所以分隔符是空格。既然要求最长者，意味着只有一个，使用`reduce`可以达到这个目的。


#### 统计一篇文章中某关键字出现的次数

例如关键字：`girls`出现的频次

```js
const chapter = 'xxx'
chapter.split(' ').filter( word => word === 'girls').length
```

#### 对字符串进行排序

例如：`chocolates`按照字母从小到大排序

```js
'chocolates'.split('').sort().join('') //accehloost
```

#### 判断字符串是否全部是1到5

例如：`234238123`

```js
`234232623`.split('').every(ch => parseInt(ch) > 0 && parseInt(ch) < 6)
```

#### 判断字符串是否包含数字

例如：`aavvd2`

```js
`aavvd2`.split('').some(ch => parseInt(ch))
```

#### 判断字符串是否回文

例如：`abccba`

```js
'abccba'.split('').reverse().join('') === 'abccba'
```

#### 生成序列值

一直对python中的range记忆深刻，js中没找到对应的函数，不过可以模拟一个。例如：生成0到99的序列值。

```js
//这里用到了扩展运算符
[ ...new Array(100).keys()]
```

如果指定区间值，如：20到40

```js
[...new Array(100).keys()].slice(20, 40)
```

如果再指定步长，如：20到40，步长为3

```js
[...new Array(100).keys()].slice(20, 40).filter((val, idx) => (idx+1) % 4)
```

## 待续。。。

按理说字符可以通过数组扩展自身功能，反过来也成立，不过场景就没这么多了，有时间再说……


