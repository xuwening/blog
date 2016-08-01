
# react-native动态修改server host

## 前言

大家都知道android端的react native可以运行时修改server host，开发模式下摇一摇设备，呼出调试菜单，就可以修改server host，不需要重新打包很方便。不知道为什么，iOS环境缺没有提供相关功能。

于是这个工具就出来了：[react-native-debug-server-host](https://github.com/xuwening/react-native-debug-server-host)

## 如何使用

使用方式简单的不能再简单了，只要把`DebugServerHost`整个目录引用到xcode工程中，恭喜你，你已经安装完毕。

运行一下，看你的调试菜单是不是多了一项：

![](https://raw.githubusercontent.com/xuwening/react-native-debug-server-host/master/image/1.png)

修改`server host`可以通过手工输入，也就是直接在文本框中手工打字，原则上不建议这么做，太虐心了。因为提供了更方便的修改途径：`扫描二维码。`

![](https://raw.githubusercontent.com/xuwening/react-native-debug-server-host/master/image/2.png)

具体操作步骤：

1. 将服务器地址通过二维码生成工具，生成二维码。

2. 点击`Input URL With QRScan`，打开扫一扫工具，扫描二维码。

3. 点击`OK`，会自动执行`reload`动作。


很方便是吧。集成和使用说完了，下面说下具体实现。只关心使用的朋友就不用继续往下看了。

> 该库中使用了二维码扫描库`QRCodeReaderViewController`，如果你的工程中也使用了这个库，保留工程中的，删除库中的源文件即可。


## 实现原理

调试菜单的实现在`RCTDevMenu`这个类中，每次打开调试菜单时，都会调用`menuItems`这个方法，它是用来创建菜单选项的，所以我们要添加自己的调试菜单，只需要在末尾追加就可以了。

考虑到react native更新频率较快，并且直接修改源码的方式不太科学，因此创建`RCTDevMenu`的分类，添加自定义菜单，然后使用swizzling技术替换原有方法。swizzling在react native工具类`RCTUtils`中已经实现。


```objective-c
@implementation RCTDevMenu (serverAddr)

- (NSArray<RCTDevMenuItem *> *)newMenuItems {
  
  NSMutableArray<RCTDevMenuItem *> *items = (NSMutableArray *)[self newMenuItems];
  
  RCTDevMenuItem *item = [RCTDevMenuItem buttonItemWithTitle:@"Debug server host" handler:^{
    
    [[NSNotificationCenter defaultCenter] postNotificationName:ChangeServerAddrNotification
                                                        object:nil
                                                      userInfo:nil];
    
  }];
  
  [items addObject: item];
  return items;
}

@end
```


可以看到，我们添加了一项菜单`Debug server host`，处理hander发送了一个通知，用来告知处理类打开UI面板，让用户设置server host。

```objective-c
- (void)changeServerAddr:(NSNotification *)notification {
  
  dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    
    [[UIApplication sharedApplication].keyWindow.rootViewController presentViewController:_serverAddrviewController animated:YES completion:^{
      NSLog(@"");
    }];
  }); 
}
```

到这里，已经实现了调试菜单，并让用户修改server host选项。下一步，就要告知`RCTBridge`新的server host，因为加载的动作是`RCTBridge`执行的。

`RCTBridge`有个`RCTBridgeDelegate`，用来告知server host是哪个，所以只要实现该协议，并指定`RCTBridge`的具体delegate即可。

```objective-c

//修改RCTBridge的delegate为自定义对象
- (void)setBridge:(RCTBridge *)bridge {
  
  _bridge = bridge;
  
  if ([_bridge isKindOfClass:[RCTBatchedBridge class]]) {
    
    RCTBatchedBridge *batched = (RCTBatchedBridge *)_bridge;
    [batched.parentBridge setValue:self forKey:@"delegate"];
  } else {
    
    [_bridge setValue:self forKey:@"delegate"];
  }
}
```

```objective-c
//为简单起见，server host传递放在全局配置文件中
- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge {
  
  NSURL *serverUrl = nil;
  NSString *url = [[NSUserDefaults standardUserDefaults] objectForKey:@"RCT_SERVER_ADDR_URL"];
  if (url != nil && url.length > 1) {
    
    serverUrl = [NSURL URLWithString: url];
  }
  
  return serverUrl;
}
```

核心实现基本就差不多了，剩下没什么好说的，有兴趣直接看源码吧。

[源码地址](https://github.com/xuwening/react-native-debug-server-host)

如果有帮助到你就给颗星吧，^_^
