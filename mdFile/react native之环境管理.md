# react native之环境管理

## 前言

react native环境牵扯到多方面，包括node.js，以及node版本管理工具nvm或brew，还包括npm、react-native、react-native-cli以及npm管理的js库和react-native管理的本地库。由于react native版本迭代比较快，相应的环境就需要更新，这篇文章就来谈谈整体环境如何升级，本文的目的是梳理各个工具之间的关系，当某工具需要更新时，该如何操作了然于胸，不必每次上网查询。

注：本篇文章针对不了解node的客户端人员，搞node开发的就不用看了。

## react native为什么需要node

要搞清楚这个问题，就需要对react native技术原理有简单的了解。简单来说，react native是使用js调用原生能力的一套库，因此涉及到原生代码管理和js代码管理，react native要自动化管理两者代码就需要一套工具，由于node天然的js亲和性，同时node可以调用系统操作，于是选择了node。当然，这只属个人猜测。因此，react native需要一套自动化管理工具，只是facebook选择使用node而已。

## 如何安装、管理node版本

方式一：手动管理，直接源码编译安装。这是最原始的一种方式,因为node是开源的，所以可以使用这种方式，对于使用者而言，不推荐这种方式。
方式二：nvm管理，安装、卸载、多版本管理都很方便。nvm的好处是平台通用，坏处是需要单独安装nvm。
方式三：brew管理，同nvm一样方便，不同的是brew针对所有软件，nvm仅用于管理node。坏处是brew是针对mac平台。

如果你不确定是通过哪个安装的，可以通过`which node`和`brew list`查看。

## 原生程序和脚本程序

无论node还是nvm、brew，都是原生程序，而npm、react-native以及react-native-cli等都是脚本程序，这个概念很重要。原生程序的管理要比脚本程序管理起来复杂，因为它依赖系统环境，而脚本程序仅仅是脚本文件，随便替换改写就可以更换行为。因此脚本程序的升级更简单些。

如何查看一个命令是脚本还是原生？可通过查看文件是否二进制文件。

```
file `which npm` 通过系统命令file查看文件格式
```

或者直接用文本编辑器打开文件查看。


## npm包管理

如果你已经装好node，那么npm已经自动集成到你的环境，可以直接使用。npm是node进行包管理的工具，支持node的第三方模块都可以使用npm来安装，类似android的gradle和ios的cocoaPods。

## react-native-cli

react-native-cli是方便开发、编译、打包react native工程的开发工具，减少手工操作环节。在终端执行的命令`react-native`实际上就是react-native-cli。

## react-native

react-native则是具体的工程元素，也就是具体代码和库，所有工具命令都是围绕它在转。在这里我们可以把它作为一个模板，即`react-native init`创建工程的模板，就是采用这个版本的react-natvie。想查看当前react-native的版本，可以通过命令`react-native --version`，它会显示react-native和react-native-cli的版本。

注：react-native版本显示的是当前工程使用的版本，所以只有在工程根目录执行命令才会显示react-native版本。

## 依赖关系及具体操作

很明显，源头是nvm和brew，他们不是强依赖的，只是方便我们管理node。所以nvm和brew的版本和node无关，也就是说升级node不必升级它们。

再来看npm，npm与node的关系比较弱化，node是一个js的运行环境，而npm是用来管理运行在这个环境之上的所有包，两者互不干扰。但运行`ls -l `which npm``可以发现npm是node_modules中的一个模块，也就是说npm工具是用node实现的，运行在node环境之上，所以两者之间还是有些弱化关系，例如，某天npm新版本可能使用了node的新特性，则想升级到npm最新版本，首先必须升级node。

react-native-cli、react-native同样是node的包，所以都可以通过npm管理器来升级和安装：`npm install`。

## 写在最后

明白它们之间关系，操作起来就很容易了，要更新某包或软件，只要找到它的上级管理器即可。具体操作命令查看命令帮助，要善于使用命令帮助。所有命令规则基本都一致：

```
command --help
command --version
command subcommand --help
```


