# React Native之javascript

## 前言

之前写过一篇 [理解javascript对象、Function和原型](https://github.com/xuwening/blog/blob/master/mdFile/%E7%90%86%E8%A7%A3javascript%E5%AF%B9%E8%B1%A1%E3%80%81Function%E5%92%8C%E5%8E%9F%E5%9E%8B.md)，基本把javascript说明白了。那么，为什么又要写一篇呢？

因为React Native支持ES6了，ES6写起来代码更清爽些，所以还是有必要学习一下。因此，我们这里着重介绍ES6的新语法。

## 开始之前

正确的学习方式，要理解现有javascript的不足，ES6就是针对这些不足做出了部分改善。

之前的javascript有何不足呢？

* 首先，没有块作用域，这个是最不能让人理解和接受的。
* 其次，不能导入其他文件。
* 最后，没有命名空间。

至于没有类或其他什么的，到不是太重要。

## 增加块作用域

有了块作用域，就不再用技巧来防止作用域的问题。

```js
{
	let a = 1;  //块级别作用域
	var b = 2;
}
```

注意：`块作用域变量必须使用let声明。`也就是说使用var声明的变量依然没有块作用域。

这是为哪般呢？当然是为了向前兼容，总不能语言改变一次，旧代码就跑蹦一次。

顺带着，这次也增加了常量声明const。

## 模块与命名空间

一个js文件默认就是一个模块，而要导入一个模块使用`import`关键字。

```js
//从fs模块导入stat、exists和readFile
import { stat, exists, readFile } from 'fs';
```

我们可以从一个模块中同时导入任意个，导出的关键字与模块中要一致（为防止命名冲突可以使用`as`起个别名）。

模块本身就是命名空间，所有内容都被模块封闭着，如果希望被外部使用，必须先导出（使用`export`关键字）。

```js
export { xxx };
```

## Promise

Promise的出现是解决javascript异步操作困难的问题，这种困难是开发者的困难。异步操作的代码确实难以调试和维护，偏偏javascript是单线程，碰到IO操作又必须异步操作，Promise的解决方式是以同步语法规则实现异步操作。

```js
var promise = new Promise(function(resolve, reject) {
  if (/* 异步操作成功 */){
    resolve(value);		//通知执行success操作
  } else {
    reject(error);     //通知执行failure操作
  }
});

promise.then(function(value) {
  // success
}, function(value) {
  // failure
});
```

再看看fetch的操作：

```js
fetch("/data.json").then(function(res) {
  
  if (res.ok) {
  		//success
  } else {
     //error
  }
}, function(e) {
	   //failure
});
```

再想想objective c的block。

## class

不得不说，短小的代码不需要class，但上规模的代码没有规范封装，维护起来会很吃力。

class就是来解决这个事儿的。

## 其他特性

1、Destructuring，从名称看就知道是struct的反向操作，举个例子：

```js
var obj = {
	name: 'xiaoming',
	age: 11
}；

var {name, age} = obj;
```

从示例可以看出很方便提取对象中的属性值，当然，它也可以用于数组等可遍历的数据类型。

2、Set、Map

Set类似没有重复值的数组。
Map扩展了key的数据类型，可以使用任意对象作为key，而不仅仅是字符串。

3、各种数据类型的扩展函数

增强数据类型的操作功能。

4、Iterator

增加对对象的遍历操作。

5、yield

增加对函数内部状态的控制（可参见其他语言如python和ruby对yield的应用）。

## 后记

javascript很难用于大型项目，但按照ES6这种规范增强，再来几波的话结果很难说，再联想至几年前的node，隐约感到了某种野心（javascript不发展，node这种东西永远成不了主流）。


