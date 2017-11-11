# tensorflow之hello world

## 环境

mac osx
python3
tensorflow1.4

## 运行mnist

tensorflow的demo放在源码中，所以首先要把整个tensorflow下载下来：

```js
git clone https://github.com/tensorflow/tensorflow.git

cd tensorflow/examples/tutorials/mnist
```

因为源码中的demo是最新版本，最好切换到和本地库版本一致。查看你安装的tensorflow版本：

```js
python3
import tensorflow as tf
tf.__version__
```

我这里是tensorflow1.4，所以切换分之并运行：

```js
git checkout r1.4
python3 fully_connected_feed.py 
```

因为demo需要先下载模型训练文件，首次运行稍微有些慢，多等待一会儿。正确运行会输出一下内容：

```js
Extracting /tmp/tensorflow/mnist/input_data/train-images-idx3-ubyte.gz
Extracting /tmp/tensorflow/mnist/input_data/train-labels-idx1-ubyte.gz
Extracting /tmp/tensorflow/mnist/input_data/t10k-images-idx3-ubyte.gz
Extracting /tmp/tensorflow/mnist/input_data/t10k-labels-idx1-ubyte.gz

Step 0: loss = 2.34 (0.123 sec)
Step 100: loss = 2.19 (0.002 sec)
Step 200: loss = 1.94 (0.003 sec)
Step 300: loss = 1.58 (0.002 sec)

......
```

