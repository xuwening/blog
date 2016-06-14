
# react native事件及手势响应系统

## 事件

BatchedBridge提供了一个调用js模块的接口`enqueueJSCall`，虽然根据`module.method`命名去调用js，其实只是进行了封装，实际还是通过js的总入口`callFunctionReturnFlushedQueue`进行的调用，然后在js侧再分发到各个js模块，具体详情请参考[react native之OC与js之间交互](https://github.com/xuwening/blog/blob/master/mdFile/react%20native%E4%B9%8BOC%E4%B8%8Ejs%E4%BA%A4%E4%BA%92.md)。

这一章节主要介绍react native的事件处理。那么提到模块调用干什么呢？因为react的事件就是基于模块调用方式实现。

在RCTEventDispatcher.m中有以下事件接口定义：

```objective-c
- (void)sendAppEventWithName:(NSString *)name body:(id)body
{
  [_bridge enqueueJSCall:@"RCTNativeAppEventEmitter.emit"
                    args:body ? @[name, body] : @[name]];
}

- (void)sendDeviceEventWithName:(NSString *)name body:(id)body
{
  [_bridge enqueueJSCall:@"RCTDeviceEventEmitter.emit"
                    args:body ? @[name, body] : @[name]];
}

- (void)sendInputEventWithName:(NSString *)name body:(NSDictionary *)body
{
  name = RCTNormalizeInputEventName(name);
  [_bridge enqueueJSCall:@"RCTEventEmitter.receiveEvent"
                    args:body ? @[body[@"target"], name, body] : @[body[@"target"], name]];
}
```

基本事件分为3种：APP事件、设备事件和输入事件。当需要传递事件到js侧，可根据事件类型使用上面的接口发送，js端对应的模块去处理响应的事件。

查看RCTNativeAppEventEmitter.js代码和RCTDeviceEventEmitter.js代码，可以发现它就是使用了EventEmitter模块。而RCTEventEmitter使用的是ReactNativeEventEmitter模块。

EventEmitter模块处理比较简单，直接调用注册的监听器。而ReactNativeEventEmitter模块处理相对比较复杂些，主要针对输入的高性能处理。

## 手势响应系统

手势响应也属于输入事件，不过更麻烦。

TouchableOpacity属于Touchable系列组件中的一种，还有其他组件如：TouchableHighlight、TouchableWithoutFeedback和PanResponder。这些组件是对手势响应的高级封装，更易用，低级一些的事件如touch相关事件需要用户自己处理和判断手势。

我们来看一下touch事件如何传递的，首先native实现了一个自定义手势RCTTouchHandler，并添加到了react的content view上实时监听触摸事件。RCTTouchHandler在初始化时设置了一个属性：

```objective-c
self.cancelsTouchesInView = NO;
```

这个属性默认为YES，意思是当系统根据touch点判断出用户的具体手势后，会自动发送touch cancel事件，不再继续触摸探测，并调用手势回调函数执行业务处理。现在设置为NO，则手势处理不中断触摸探测，两者同时执行:

```objective-c
//[super initWithTarget:self action:@selector(handleGestureUpdate:)]

//手势回调函数
- (void)handleGestureUpdate:(__unused UIGestureRecognizer *)gesture
{
  if (self.state == UIGestureRecognizerStateBegan) {

    [self _updateAndDispatchTouches:_nativeTouches.set eventName:@"topTouchStart" originatingTime:0];

    _dispatchedInitialTouches = YES;
  }
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
  [super touchesBegan:touches withEvent:event];

  _coalescingKey++;
  
  //记录touch操作
  [self _recordNewTouches:touches];
  
  if (_dispatchedInitialTouches) {
    [self _updateAndDispatchTouches:touches eventName:@"touchStart" originatingTime:event.timestamp];
    self.state = UIGestureRecognizerStateChanged;
  } else {
    self.state = UIGestureRecognizerStateBegan;
  }
}
```

可以看到创建手势时定义了手势操作函数handleGestureUpdate，该函数把记录的touch操作发送给js侧去处理，标记手势的开始`topTouchStart`，但并不中断触摸探测，其后继续发送`touchStart、touchMove、touchEnd`触摸事件。

touch事件的封装：

传递给js的事件使用RCTTouchEvent封装，该类指定了通过哪个js模块接口处理touch事件：

```objective-c
+ (NSString *)moduleDotMethod
{
  return @"RCTEventEmitter.receiveTouches";
}
```

touch事件和上面的输入事件使用的同一个接受模块`RCTEventEmitter`。

要想使js接收touch事件，在需要监听的view上注册监听事件:

```js
<View onStartShouldSetResponder={this.shouldResponder}
        onResponderGrant={this.responseGrant} 
        onResponderMove={this.responseMove}/>
```

onStartShouldSetResponder用来确认view是否成为响应者，必须成为响应者才能接收touch事件。事件经过js侧的如下流程：

```js
ReactNativeEventEmitter.js
    receiveEvent-handleTopLevel
ReactEventEmitterMixin.js
    handleTopLevel
    runEventQueueInBatch
EventPluginHub.js
    processEventQueue
    executeDispatchesAndReleaseTopLevel
    executeDispatchesAndRelease
EventPluginUtils.js
    executeDispatchesInOrder
    executeDispatch
Touchable.js
    touchableHandleResponderRelease
    _performSideEffectsForTransition
TouchableOpacity.js
    touchableHandlePress
```

最终回调处理函数成功。

Touchable系列的高级事件处理，是对touch进行了封装简化处理，本质还是经过touch事件传递。


## 总结

react native将native touch事件透传给js处理，这个实时性以及性能要求较高，很担心在复杂view操作时卡顿。



