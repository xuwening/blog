
# POW工作量证明

## 什么是共识机制

共识：也就是我们常说的达成共识，比特币是去中心化的，没有中心决策，因此就需要群众进行众议，达成一致决策，这就是共识。

共识机制：共识机制就是众议的规则、制度，也可以称作协议。比如投票制度就是一种共识机制，比如工作量证明就是一种共识机制。

POW（Proof Of Work），即工作量证明，是共识机制的一种具体实现，比特币采用了该共识机制。POW意思很明显，就是表明通过何种机制来证明你确实做过一定量的工作（好给你结算工资）。

## POW的基本原理


比特币说白了就是计算机系统中的一种虚拟货币，在计算机系统中证明工作量只有自身`资源`可用，如运算资源（`CPU`、`GPU`），存储资源：`磁盘空间`、`内存空间`。POW使用的是运算资源。

先从简单说起，给你两个数，帮我做加法运算，这就是你做了定量的工作：

```python
def proofOfWork(x, y):
    return x + y
```

这里存在两个问题：

1. 计算难度太低，很难区分到底谁先计算完成；
2. 如何验证结果的正确性？如果验证也需要运行得出结果，那么就失去委托工作的意义；

问题1的解决思路比较简单，增加计算复杂度就行了，比如计算一个较大的素数或素数因子分解。问题2需要设计一个正向验证简单，反向运算比较复杂的公式。

POW如何实现的呢？

POW采用单项散列算法(SHA-256)，就是给你一组数据，让你计算出其散列值。大家都用过md5，知道单项散列计算复杂度并不高，因此POW又给出了限制条件，即计算出的散列值必须小于某个数值，这就大大增加的复杂度。

对单项散列不熟悉的话可以参考：[单向散列算法简介](https://github.com/xuwening/blog/blob/master/mdFile/%E5%8D%95%E5%90%91%E6%95%A3%E5%88%97%E7%AE%97%E6%B3%95%E7%AE%80%E4%BB%8B.md)

具体运算是这样，对给定数据进行hash，其结果必定唯一。现引入一个随机数和限定值，使得：

```js
hash(数据 + 随机数) < 限定值
```

则，运算出的散列值才是符合条件的。

限定值的思路是这样的，对字符串"abc"进行md5后的值为：`7ac66c0f148de9519b8bd264312c4d64`，我们知道按照散列原理，散列值的大小是完全不可预估的，一旦对散列结果进行限制则会增大计算复杂度，比如前4位必须为0：`0000xxxxxxxxxxxxxxxxxxxxxxxxxxxx`。我们该如何计算出需要的结果呢？没有好的办法，只能对随机数进行递增，循环计算新的散列值是否符合条件。

前缀0的个数越多，说明限定值越小，计算难度越大。

难度的问题解决了，结果验证比较简单，只要提供正确的随机数，只要执行一次比较`hash(数据 + 随机数) < 限定值`就能验证是否正确。


为简单起见，我们简化POW的运算过程：

```python
import hashlib


def proofOfWork(data, limit):
    randnum = 0
    hashvalue = 0
    while True:
        hashvalue = hashlib.sha256(data + bytes(randnum)).hexdigest()
        if hashvalue < limit:
            break
        else:
            randnum += 1

        print(hashvalue)

    return randnum, hashvalue

if __name__ == '__main__':
    rnd, hvalue = proofOfWork(b'abcd', '00FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF')
    print(rnd, hvalue)

//output
366 0037cc3b922a68ef1c794e3971ccaa43b5acfb7178e94be93ddbeb960f456518
```

可以看到，我们设置了两个00，随机数遍历到366时，才得出符合要求的运算值。再将前缀设置为0000增加难度`0000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF`：

```python
//output
98786 00007e16a7fea47a521a3279e3a0d4a7f7aa13768b026b365aca9bb83e4f22c4
```

运算到98786次才得出符合要求的散列值，需要的时间要久的多。采用此种方式还有个好处，我们调整0的个数，就可以控制计算散列值的复杂度及大概运算时间范围，可以量化。

验证环节就很简单了，直接通过提供的原始数据+随机数进行sha-256，然后和限制数进行比较（为了方便，示例中直接用字符串来进行比较）：

```python
In [31]: hashlib.sha256(b'abcd' + bytes(366)).hexdigest()
Out[31]: '0037cc3b922a68ef1c794e3971ccaa43b5acfb7178e94be93ddbeb960f456518'

In [32]: '0037cc3b922a68ef1c794e3971ccaa43b5acfb7178e94be93ddbeb960f456518' < '00FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF'
Out[32]: True
```

POW的基本原理就差不多是这样了。这个单项散列计算其实就叫挖矿，运行POW的客户端、计算机或所属人就是矿工。

看到这里，也许你会想有没有简单的方式可以计算出符合需求的散列值呢，比如找其规律等等。这其中有几个难点：

1. data是不停变化的，实际应用中散列值经过双重散列计算，复杂度更高
2. 根据散列特征，要破解必须破解sha-256算法
3. 比特币稳定运行10年了……


比特币在设计过程中，难度系数是动态调整的，使计算难度一致保持在大约10分钟左右。

通过上面的基本原理介绍，你会发现POW实现比较简单，经过上线多年也比较稳定和安全。缺点是，计算资源浪费严重，也就是说POW是为了解决某些方面问题而人为设置的障碍。

