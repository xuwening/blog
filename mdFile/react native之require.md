
## react native之require和module

## javascript没有模块

javascript在设计之初仅为了嵌入浏览器中执行的代码片段，因此应用范围一旦扩大些，就显现出短板，称之为曰：`设计缺陷。`

估计javascript的作者也很郁闷，谁也没有想到javascript在如今撑起这么一片天空。

javascript`缺陷`之一就是没有模块的概念，说白了就是无法引入其他js文件。javascript也没有命名空间，不过可以通过函数作用域解决。

## node.js中的模块

node.js立志在服务端打一片天下，解决模块化成为首要任务，node.js如何解决的呢，通过源码可以看出大体思想：

```javascript
//require.js
module.exports = function(file, opts) {
  file = path.join(path.dirname(module.parent.filename), file)
  if (!fs.existsSync(file) && fs.existsSync(file+'.schema')) file += '.schema'
  if (!fs.existsSync(file) && fs.existsSync(file+'.json')) file += '.json'
  return compile(fs.readFileSync(file, 'utf-8'), opts)
}
```

从这段代码中基本可以看出思路：按照js文件作为模块划分，require一个模块就是加载一个js文件，让模块的作用域局限在module变量中。

***注意：node.js和react native都实现了require.js，代码要从node.js中找。***

## react native中的模块

我们再来看下react native中的require实现。

```js
//react native中的require.js
global.require = require;
global.__d = define;

function define(moduleId, factory) {...}
function require(moduleId) {...}
```

首先在全局作用域定义了两个函数：`require`和`__d`，其后定义了函数的实现。

先暂时不管具体实现，先看如何应用。react native无论开发模式还是生产模式，都是把所有的js打成一个包main.jsbundle，其实就是一个完整的js文件。可以通过：

`curl -o main.jsbundle http://localhost:8081/index.ios.bundle?platform=ios&dev=true`

或在项目根目录中执行：

`react-native bundle --entry-file index.ios.js --bundle-output ./main.jsbundle --platform ios --dev true`

生成main.jsbundle文件，然后用文本编辑器打开，虽然代码量很多，但可以看见后半部分代码都是相同的形式：


```js
//main.jsbundle
__d(422 /* object-assign/index.js */, function(global, require, module, exports) {
    ...这里省略代码若干
});
```

这里的__d就是上面的全局函数，调用的是require.js中的define方法。如果你仔细观察__d中省略的代码，就会发现一个秘密：`省略的代码其实就是422代表的js源码object-assign/index.js`，到这里估计已经明白了，__d函数就是用来定义一个模块的，而require函数必然是用来引用模块。

为了验证我们的想法，继续看代码，define函数的实现：

```js
//require.js
const modules = Object.create(null);

function define(moduleId, factory) {
  if (moduleId in modules) {
    // prevent repeated calls to `global.nativeRequire` to overwrite modules
    // that are already loaded
    return;
  }
  modules[moduleId] = {
    factory,            //key value简写
    hasError: false,
    isInitialized: false,
    exports: undefined,
  };
}
```

这段代码很简单，定义了一个数组，根据moduleId保存一个独立模块，模块有一个factory工厂函数，还有`hasError、isInitialized、exports`三个模块状态属性。结合上面`main.jsbundle`中的语句，可以知道：

`react native中的模块，沿用node.js方式，即利用node.js的工程环境。`

在react native的编译运行时，会把所有模块文件合并，引入到一个总的js文件(main.jsbundle)，所有模块都用__d函数组织起来，并注册到modules数组中。模块之间的作用域也都使用__d函数封闭起来，成为局部作用域，不会造成命名空间污染。到这里，模块的概念已经建立完毕。

下来再看看模块的引入，require函数的实现：

```js
//require.js
function require(moduleId) {

  //require时调用factory函数，才真正加载模块
  const module = __DEV__
    ? modules[moduleId] || modules[verboseNamesToModuleIds[moduleId]]
    : modules[moduleId];
  return module && module.isInitialized
    ? module.exports
    : guardedLoadModule(moduleId, module);
}

function guardedLoadModule(moduleId, module) {
    ...省略若干代码
    loadModuleImplementation(moduleId, module);
}

function loadModuleImplementation(moduleId, module) {

    ...省略若干代码
    // keep args in sync with with defineModuleCode in
    // packager/react-packager/src/Resolver/index.js
    factory(global, require, moduleObject, exports);
}
```

require函数代码有点多，但核心代码只有几句，我把非核心代码都省略掉了，有利聚焦。当调用require时，会从modules数组中查找已经注册的模块，如果该模块还没有加载，则使用factory加载该模块。从main.jsbundle中知道factory其实就是真正的模块代码。最终模块都保存在modules中，而导出的属性或函数以module.exports方式供外部访问。

## react native模块的导出

通过上面的内容，其实已经可以看出模块如何导出和引入的，这里单独梳理一下。

首先模块是以这样的形式给出的：

```js
  modules[moduleId] = {
    factory,            //key value简写
    hasError: false,
    isInitialized: false,
    exports: undefined,
  };
```

其中exports就是用来暴露导出的内容。那具体什么时候导出的呢？其实在factory中，回忆下你写的react native模块中，需要暴露的都是用：`module.exports = xxx` 修饰。

module又是从哪里传递给你写的模块呢？再来看下main.jsbundle中__d函数的factory函数的定义，以及require函数的加载。

```js
//main.jsbundle
__d(422 /* object-assign/index.js */, function(global, require, module, exports) {
    
});

//require.js

factory(global, require, moduleObject, exports);

```

看到module和exports了没，就是require时调用factory函数时将module传递进去的。弄来弄去其实都是函数调用，将模块代码封装到函数中，只不过模块化还是借助了node.js。

## 结尾

到这里模块化基本就说完了，本人对javascript不太熟悉，有任何疏忽之处，请以邮件告知。





