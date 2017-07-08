# 理解 javascript 中的 this 以及 bind

## 什么是 this

在面向对象语言中都有 this 的概念。在编译型语言中，如 C++、java，this 相对比较好理解。

先来看一段 java 代码。

```java
class Person {

    private String name;
    private int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    String descriptions() {
        return "name:" + this.name + "age:" + this.age;
    }
}

public class TestDemo {

    public static void main(String argv[]) {
        Person person = new Persion("xiaoming", 19);
        System.out.println(person.descriptions());
    }
}
```

在面向对象语言中，都有类的概念，类的实例称之为对象。类中所有方法都属于对象，只有对象才能调用方法，对象之外是无法直接访问对象方法的。（类方法例外，也正因为类方法不依赖实例，所以没有this对象）

我们知道机器执行时，是没有类和对象的概念的，只有函数调用。也就是说，无论哪种语言，面向对象的调用方法最终还是会转换为普通函数调用。
以上代码为例，在运行执行person.descriptions()时，最终会转化为如下形式：

```java
descriptions(person);  //对象作为函数descriptions的 this参数
```

也就是说，每个对象的方法，都会有一个隐含的参数，一般作为参数列表中的第一个，这个参数就是 this。C语言中没有 this，那么用 C 语言模拟 C++方法调用则更容易理解 this：

```c

typedef struct tag_Person {

    char *name;
    int age;
    char *(*descriptions)(struct tag_Person *this);
} Person;

char *descFunc(struct tag_Person *this) {
    return sprintf("%s%s%s%d", "name:", this->name, "age:", this->age);
}

Person person;
person.descriptions = descFunc;
person.name = "xiaoming";
person.age = 19;

person.descriptions(&person);  //模拟 C++方法调用，传递person对象作为 this


```

综上所述，this 的作用就是让函数作用在哪个对象上。因为java 和 C++等语言，函数均和调用对象自动绑定，所以容易理解。

## javascript 中的 this

之所以有上面的铺垫，实则因为 javascrpt 的 this 绑定方式有所不同，导致 javascript 方法调用时经常出错，也较为难以理解。

先来看段代码:

```javascript
function test(x,y) {
    this.x = x;
    this.y = y;
}
```

有两种执行方式：

```javascript
//对象调用
var o = new test(1, 2);
o.x  //1
o.y  //2

//函数调用
test(1, 2);
o.x //undefined
o.y //undefined
x   //1
y   //2
```

可以看到，执行对象调用时，符合我们的预期。函数调用时，结果会出乎意料之外。究其原因，javascript 的 this 绑定方式与其他语言不一样。javascirpt函数绑定方式属于运行时绑定，也就是说，this引用的对象与调用者相关。

在执行函数调用时，test的调用者是全局对象，执行`this.x = x`时，相当于给全局对象添加了一个`x`，该值为1。而对象调用方式，调用对象是`o`，则会给`o`正确赋值。

如果不能理解`new test(1, 2)`为什么作用对象是`o`，需要了解一下 javascript 创建对象的过程，大致流程如下：

```javascript
var o = {};
o.__proto__ = test.prototype;
test.call(o);   //构造对象时，绑定 o 对象
```

## 嵌套函数中的 this

嵌套函数中使用 this，是更复杂一些的场景。

```javascript
function test(x, y) {
    this.x = 0;
    this.y = 0;

    var f1 = function(x, y) {
        this.x = x;
        this.y = y;
    }

    f1(x, y);
}

var o = new test(1, 2);
o.x //0
o.y //0
x   //1
y   //2
```

现在按照对象调用方式也不正确了，原因在于使用了内嵌函数。test 函数虽然已经绑定到 o 对象上，但 f1函数却未绑定到 o，于是默认绑定到外层对象，也就是全局对象。这个坑就比较大了。

通常解决方法：

```javascript
function test(x, y) {
    this.x = 0;
    this.y = 0;

    var that = this;  //保存 this变量，供内部函数使用
    var f1 = function(x, y) {
        this.x = x;
        this.y = y;
    }

    f1(x, y);
}

var o = new test(1, 2);
o.x //1
o.y //2
```

当然也可以采用 call 和 apply 来动态绑定对象，更好的方式是采用 ES5的特性 bind。

```javascript
function test(x, y) {
    this.x = 0;
    this.y = 0;
    
    var f1 = function(x, y) {
        this.x = x;
        this.y = y;
    }

    f1.bind(this)(x, y);
}
```

## bind 函数

bind的作用是将某函数绑定到固定对象上，准确来说，是返回一个绑定到固定对象上的函数。 为什么需要 bind 呢？就像上面示例一样，如果不 bind，使用 this 时会出现很多奇怪的问题。

先定义一个函数

```javascript
function test(x, y) {
    this.x = x;
    this.y = y;
}
```

然后定义一个对象，并对该对象绑定函数。

```javascript
var a = {};
t1 = test.bind(a);  //返回绑定对象 a 的函数t1，从此 t1中的this 永远指向 a 对象

t1(1,2);
a.x   //1
a.y   //2

t1(3, 4)
a.x   //3
a.y   //4

```

再来对 t1执行call 来改变作用对象试试

```javascript
b = {}
t1.call(b, 7, 8)
a.x    //7
a.y    //8
b.x    //undefined
b.y    //undefined
```

可见即便调用 call 也依然改变不了绑定对象，这就是 bind 的作用。


## React Native 中为何需要频繁 bind 函数

先看RN 中 bind 示例：

```javascript
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    //对当前对象的方法做 bind 处理
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```

主要分析几个问题：`为什么要做 bind？对谁做 bind？何时做 bind？如何做 bind？`

1. 为什么要做 bind？

该代码的功能是点击 button后，执行Toggle的handleClick方法。可以看到handleClick方法中使用了 this引用，而onClick 属于 button 对象，如果不做 bind 处理，则handleClick中的 this 最终绑定的对象是 button，而非Toggle。

2. 对谁做 bind？

我们知道 bind 主要是为了将函数绑定正确的对象上，所以要对handleClick方法做绑定，该函数使用了 this 引用。

3. 何时做 bind？

bind 会生成一个新的方法，因此需要在将handleClick方法赋值给onClick事件之前执行，否则onClick就会被赋值旧的方法。因此最好在构造Toggle对象时bind。

4. 如何做 bind？

bind 会生成新的方法，所以做 bind 有两种方式。一是将 bind 后的方法起个新名字；二是用新的方法替换旧的方法（bind 后的方法替换绑定前的方法）。因为我们写 class 的目的就是想像其他语言一样使用对象方法，所以采用第二种方式更符合我们的要求（如本示例）。缺点就是不能再将handleClick方法作用到其他对象上，如`toggle.handleClick.call(other)`。







