
# react native之线程

## javascript线程

javascript属于解释执行的语言，本身处于一个线程中，只能异步不能并发执行，落实到具体执行指的就是javascirpt解释器。也就是说javasccript所处的线程等同于解释器所在的线程，解释器具体在哪个线程自行指定。

既然可以指定解释器在某个具体线程，那么可以创建多个解释器，分别指定在多个线程，从而实现javascript的并发。只是到目前为止，客户端侧的性能集中在UI渲染，UI渲染又是本地语言的事情，所以实现javascript的并发并没有太大意义，何况实现的成本也不低。

react native对于这块的处理方式是，将javascript与native之间的交互单开一个线程处理，这样就不抢占UI线程的资源，牵扯到UI更新时，将这部分逻辑加入到UI线程队列即可。

实现这部分交互大部分都在RCTJSCExecutor这个类中实现，首先是开启新线程，一般称之为work线程：

```objective-c

//线程函数，事件循环
+ (void)runRunLoopThread
{
  @autoreleasepool {
    // copy thread name to pthread name
    pthread_setname_np([NSThread currentThread].name.UTF8String);

    //创建source避免线程空循环，但不使用source处理事件
    CFRunLoopSourceContext noSpinCtx = {0, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL};
    CFRunLoopSourceRef noSpinSource = CFRunLoopSourceCreate(NULL, 0, &noSpinCtx);
    CFRunLoopAddSource(CFRunLoopGetCurrent(), noSpinSource, kCFRunLoopDefaultMode);
    CFRelease(noSpinSource);

    //事件监听循环
    while (kCFRunLoopRunStopped != CFRunLoopRunInMode(kCFRunLoopDefaultMode, ((NSDate *)[NSDate distantFuture]).timeIntervalSinceReferenceDate, NO)) {
      RCTAssert(NO, @"not reached assertion"); // runloop spun. that's bad.
    }
  }
}

- (instancetype)init
{
  if (self = [super init]) {
    _valid = YES;

    //创建work线程，线程函数：runRunLoopThread
    _javaScriptThread = [[NSThread alloc] initWithTarget:[self class]
                                                selector:@selector(runRunLoopThread)
                                                  object:nil];
    _javaScriptThread.name = RCTJSCThreadName;

    if ([_javaScriptThread respondsToSelector:@selector(setQualityOfService:)]) {
      [_javaScriptThread setQualityOfService:NSOperationQualityOfServiceUserInteractive];
    } else {
      _javaScriptThread.threadPriority = [NSThread mainThread].threadPriority;
    }

    [_javaScriptThread start];
  }

  return self;
}
```

这里除了创建一个work线程之外，还设定了事件循环，因为要永久监听来自native或javascript命令事件。各个平台对事件循环的具体方式不太一样，但思想基本一致：`要永久循环，空闲时处于休眠状态，可由外部事件唤醒并处理该事件，处理完毕继续进入休眠状态。`

Runloop是iOS下的事件监听机制，这里只是使用source事件防止空循环，并没有使用source做事件处理。那么该线程是如何被唤醒并处理调用流程呢？executeBlockOnJavaScriptQueue函数就是用来指定在该线程执行：

```objective-c
- (void)executeBlockOnJavaScriptQueue:(dispatch_block_t)block {

  if ([NSThread currentThread] != _javaScriptThread) {
    [self performSelector:@selector(executeBlockOnJavaScriptQueue:)
                 onThread:_javaScriptThread withObject:block waitUntilDone:NO];
  } else {
    block();
  }
}
```

`performSelector: onThread:`可以指定某个方法在指定线程执行，这样就会唤醒线程执行任务。

javascript所有处理都会通过该线程处理，调用UI更新的部分在RCTBatchedBridge的handleBuffer中进行线程切换：

```objective-c
- (void)handleBuffer:(NSArray *)buffer {
    
    ...省略若干代码

    //组装block
    dispatch_block_t block = ^{
        //...
    };

    if (queue == RCTJSThread) {
        //work线程处理
      [_javaScriptExecutor executeBlockOnJavaScriptQueue:block];
    } else if (queue) {
        //依据各模块定义队列执行
      dispatch_async(queue, block);
    }
}
```

各模块都可以覆盖一个methodQueue方法，返回该模块导出的函数需要运行的队列，这里的handleBuffer把函数分类到各个指定的队列上执行。

上面是js调用native的线程处理，下面是native调用js的线程处理：

```objective-c
//RCTJSCExecutor.m
//调用js的总入口
- (void)_executeJSCall:(NSString *)method
             arguments:(NSArray *)arguments
              callback:(RCTJavaScriptCallback)onComplete
{
  //这里将main线程的调用转换到work线程上执行
  [self executeBlockOnJavaScriptQueue:RCTProfileBlock((^{

    //...省略具体调用

    }))];
}
```

以上就是UI线程与work线程之间的切换。

## native组件执行的线程

组件必须实现RCTBridgeModule协议，有个可选项`methodQueue`用来指定导出函数运行的线程队列：

```objective-c
@protocol RCTBridgeModule <NSObject>

//模块化的自动注册过程
#define RCT_EXPORT_MODULE(js_name) \
RCT_EXTERN void RCTRegisterModule(Class); \
+ (NSString *)moduleName { return @#js_name; } \
+ (void)load { RCTRegisterModule(self); }

// Implemented by RCT_EXPORT_MODULE
+ (NSString *)moduleName;

@optional
//指定导出函数运行的线程队列
@property (nonatomic, strong, readonly) dispatch_queue_t methodQueue;

@end
```

也就是说，react native不知道组件运行在哪个线程环境，需要定义者自己指定。既然是可选协议，组件定义者也可以不指定`methodQueue`，默认运行在work线程。从哪里看出的呢？依然在上面的`handleBuffer`函数中：

```objective-c
- (void)handleBuffer:(NSArray *)buffer {

    //这里RCTJSThread的值为kCFNull，未指定的queue值也为kCFNull
    if (queue == RCTJSThread) {
        //work线程处理
      [_javaScriptExecutor executeBlockOnJavaScriptQueue:block];
    } else if (queue) {
        //依据各模块定义队列执行
      dispatch_async(queue, block);
    }
}
```

可以看到没有指定methodQueue的值就为NSNull，就在work线程上处理。







