# tensorflow基础概念的用法

本示例对应[tensorflow中的基本概念](https://github.com/xuwening/blog/blob/master/mdFile/tensorflow中的基础概念.md)

主要是对概念的简单代码示例，用于加深对概念的理解，不对具体函数进行完整说明。

## 常量、会话

使用tensorflow实现`2+2=4`的计算。

```python
import tensorflow as tf

a = tf.constant(2)
b = tf.constant(2)
output = tf.add(a, b)

sess = tf.Session()
print(sess.run(output))
```

常量使用`tf.constant`函数创建，经过tensorflow的封装，返回的`a`是个`tensor`对象。在python交互解释器中查看它的说明：

```python
In [71]: a
Out[71]: <tf.Tensor 'Const_6:0' shape=() dtype=int32>
```

可以看到`a`是个`tensor`对象，不可变，标量，数据类型是`int32`。并且输出描述里没有a的具体值，因为`tensor`是引用对象，必须经过sess.run计算后才知道其值。

sess.run可以计算任意节点，我们最终的目的是为了计算`output`节点，但也可以单独计算其他节点：

```python
In [76]: sess.run(a)
Out[76]: 2
```

`a`被计算出来，是2没有问题。

`sess.run(output)`会自定计算被依赖的节点，因为计算`output`依赖`a`和`b`节点，因此，a和b不用显式指定sess.run。

## 张量

真正计算时，往往都是复杂数据，通常用`向量`、`矩阵`表示，甚至多维数组。`tensor`就是能覆盖标量、向量、矩阵、多维数组的对象。

```python
import tensorflow as tf

matrix1 = tf.constant([[11, 12], [21, 22]])
matrix2 = tf.constant([[31, 32], [41, 42]])

product = tf.matmul(matrix1, matrix2)

sess = tf.Session()
print(sess.run(product))
```

这次定义的是2x2的矩阵相乘。查看matrix1的描述：

```python
In [78]: matrix1
Out[78]: <tf.Tensor 'Const_7:0' shape=(2, 2) dtype=int32>
```

可以看到依然是`tensor`对象。tensorflow中的数据表示方式就是用tensor。

## 图

上面的例子都没有创建图graph，所以那些节点都添加到了默认图中，可以查看`tensor`所属的图：

```python
In [80]: a.graph
Out[80]: <tensorflow.python.framework.ops.Graph at 0x11077d780>
```

再看下默认的图，可以看到a所在的图和默认图的地址是一样的，都是`0x11077d780`，也可以判断是否相等来确认地址一致：

```python
In [82]: tf.get_default_graph()
Out[82]: <tensorflow.python.framework.ops.Graph at 0x11077d780>

In [83]: a.graph == tf.get_default_graph()
Out[83]: True
```

在小一些的程序中，默认图已经够用，如果需要在不同的图中构建不同的任务，可以手动创建图并添加不同的节点，进行任务的划分：

```python
import tensorflow as tf

graph1 = tf.Graph()

with graph1.as_default():
    a = tf.constant(2)
    b = tf.constant(2)
    output = tf.add(a, b)

with tf.Session(graph=graph1) as sess:
    print(sess.run(output))

graph2 = tf.Graph()
with graph2.as_default():
    a = tf.constant(6)
    b = tf.constant(2)
    output = tf.div(a, b)

with tf.Session(graph=graph2) as sess:
    print(sess.run(output))
```

我们建立了两个graph，分别在其中定义了不同的运算，并为之创建不同的session进行处理运算。

## 变量

程序总是需要变量，用来记录状态的变换迁移。变量使用`tf.Variable`来创建，下面演示给一个变量循环+1的演示：

```python
import tensorflow as tf

state = tf.Variable(0, name="counter")

one = tf.constant(1)
new_value = tf.add(one, state)
update = tf.assign(state, new_value)

init_op = tf.global_variables_initializer()

with tf.Session() as sess:
    sess.run(init_op)
    print(sess.run(state))

    for _ in range(3):
        sess.run(update)
        print(sess.run(state))
```

state中记录每次计算的状态。`tf.assign`是赋值操作，每执行一次`sess.run(update)`给state赋值一次。

变量需要用`init_op = tf.global_variables_initializer`及`sess.run(init_op)`进行初始化才可以参加运算，前者类似声明，后者类似定义。可以看到，任何节点和操作必须经过run才会真正计算。

> 注意：不同的版本初始化变量的函数不一样。

## 占位符

采用placeholder，可以分方便的进行不同样本的训练。

```python
import tensorflow as tf

input1 = tf.placeholder(tf.float32)
input2 = tf.placeholder(tf.float32)
output = input1 * input2

with tf.Session() as sess:
    print(sess.run([output], feed_dict={input1:[7.], input2: [2.]}))
```

如果要有多组参数，可以循环调用sess.run并把参数通过`feed_dict`传递给`placeholder`。

