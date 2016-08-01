
# react native之App集成以及高度自适应

## 集成

与现有App集成，以及高度适应问题，官方文档中都有说明，单拉一篇文章出来，主要是想记录一下其中的问题点，给大家做些参考。

集成到现有App，无非是把react界面放到现有App中的某个页面中，那么第一点是要把react native加到现有工程，第二点创建RCTRootView并添加到某页面。

react native添加到现有工程，有两种方式：

```
1、使用cocoapods，自动化管理；
2、手工集成，即将react native各个子工程手动添加到项目中；
```

但是，在此之前，还有一件至关重要的事：`下载react native工程`。

react native使用node.js作为工程集成环境，包管理使用npm工具，所以这一步也比较简单：

```js
//在现有App工程目录中创建package.json文件，执行npm install：
{
  "name": "reactDemo",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "dependencies": {
    "react": "^15.2.1",
    "react-native": "^0.29.2"
  }
}
```

当前目录就回创建node_modules，react native所有依赖项都存在此目录中。

使用cocoapods集成比较方便：

```js
//创建podfile，配置工程依赖项，注意:path路径要设置自己的node_modules正确路径

target 'App工程名称' do

  pod 'React', :path => './node_modules/react-native', :subspecs => [
    'Core',
    'RCTText',
    'RCTWebSocket', # needed for debugging
    # Add any other subspecs you want to use in your project
    'RCTImage',
    'RCTNetwork',
    'RCTActionSheet',
    'RCTGeolocation',
    'RCTLinkingIOS',
    'RCTVibration',
    'RCTSettings',
  ]

end
```

`执行: pod install， 即可集成react native到现有工程。`

手动添加react native，就要麻烦一下，进入node_modules目录查找相应react工程，即project，将project添加到现有工程中即可。

下一步，创建react页面，并作为子页面添加到现有项目中：

```js
    NSURL *jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.ios.bundle?platform=ios&dev=true"];

    jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
    
    _rctView = [[RCTRootView alloc] initWithBundleURL: jsCodeLocation moduleName:@"reactDemo" initialProperties:@{} launchOptions:nil];
    
    [self.view addSubview: _rctView];

```

到这一步，算是整体集成完毕，可以启动服务看页面是否能正常展示。

## react高度适应

react页面高度是不定的，这就给react页面的父页面设定带来一些问题。官方也给出了解决方案，就是父页面使用UIScrollView，并监听react页面高度变化，使父页面随react页面变化而变化。

设置监听RCTRootView的高度变化：

```objective-c

    _rctView.delegate = self;
    _rctView.sizeFlexibility = RCTRootViewSizeFlexibilityHeight;

#pragma mark - RCTRootViewDelegate
- (void)rootViewDidChangeIntrinsicSize:(RCTRootView *)rootView {

    CGRect newFrame = rootView.frame;
    newFrame.size = rootView.intrinsicSize;
    
    rootView.frame = newFrame;
    self.view.frame = newFrame;
    
    if (self.delegate) {
        [self.delegate didChangeHeight: newFrame.size.height];
    }
}    
```

ok，集成和高度适应都完毕了，下来还有什么问题？

## touchable与UIScrollView事件冲突问题

这里说的事件冲突，倒不是说UIScrollView截获了Touchable的点击事件，而是Touchable会把UIScrollView的滚动事件误认为是点击事件。所以造成的现象就是，当滚动UIScrollView时，同时会出发Touchable点击事件，这种体验糟糕的不要不要的。

react native自身也封装了ScrollView，在使用react native的ScrollView时不会存在这个问题，明显react native发现并解决了该问题。react native的点击事件又必须依赖Touchable，我们又不能使用react native的ScrollView，所以这个问题不能靠框架解决。

解决思路：

Touchable是完全依赖事件模拟的点击操作，发生混乱的原因是UIScrollView滚动相关事件传递给了js侧的Touchable，所以只要把滚动的相关事件屏蔽调就可以实现目标。

解决方法：

RCTRootView是native与js侧的桥梁，所有UI触摸事件都是通过该View传递给js，该View有个cancelTouches方法，用来取消当前的触摸事件，所以，只要在`scrollViewWillBeginDragging`方法中调用`[_rctView cancelTouches]`即可。

## 交互与扩展

既然集成react native到本地项目中，那么必然还存在两者之间的交互。react native核心技术是javascriptCore，交互必然也是通过它了。与WebView不同，javascriptCore是独立存在项目中，整个react native都是通过一个单独的jscontext建立之间的联系，也就无法截获WebView的请求消息进行通信。好消息是，无论旧的WebView方式还是新的javascriptCore方式，只需要封装一下甚至无需封装，就可以将旧的jsbridge接口移植到react native项目中。

有关jsbridge和javascriptCore可以参考之前的文章：

[H5与native之间的通信](https://github.com/xuwening/blog/blob/master/mdFile/H5%E4%B8%8Enative%E4%B9%8B%E9%97%B4%E7%9A%84%E9%80%9A%E4%BF%A1.md)

[javascriptCore 详解](https://github.com/xuwening/blog/blob/master/mdFile/javascriptCore%E8%AF%A6%E8%A7%A3.md)

如何注入到react native可以参考这篇文章：

[react native之OC与js之间交互](https://github.com/xuwening/blog/blob/master/mdFile/react%20native%E4%B9%8BOC%E4%B8%8Ejs%E4%BA%A4%E4%BA%92.md)

如何注入到react native中的WebView，可以参考这篇文章：

[react native之WebView中注入js接口](https://github.com/xuwening/blog/blob/master/mdFile/react%20native%E4%B9%8BWebView%E4%B8%AD%E6%B3%A8%E5%85%A5js%E6%8E%A5%E5%8F%A3.md)

如何让你的项目可以随意更换react native服务器地址，可以参考这个工具：

[react-native-debug-server-host](https://github.com/xuwening/react-native-debug-server-host)

需要注意一点：react native的jscontext是在子线程中创建执行，所以如果你的接口中有UI的操作，需要手动指定到UI线程执行。





