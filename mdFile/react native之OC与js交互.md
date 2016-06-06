
## react native之OC与js之间交互

## OC与js交互的基础知识

之前的[H5与native之间的通信]()中提到两者交互的一般方式，主要针对iOS早期版本的兼容。在iOS7及之后的版本中，官方开放了javascriptCore API，使得OC与js交互简单方便，同时也强大了许多。iOS7之前js到OC的调用，是通过URl进行传递，在数据类型以及传递的数据长度上有很大限制，但使用javascriptCore就不存在这些问题。

react native就是使用的javascriptCore方式，所以在分析react native之前，有必要了解javascriptCore的具体使用方法，可以参考[javascriptCore详解]()。

## react naitve中的交互思想

这里省略分析过程，直接介绍分析结果，先对整体有个概念，再深入细节分析会有更好的效果。

react native并没有把native中的模块和方法直接暴露给js调用，而是将模块和方法名称作为全局数据暴露给js端，js端所有对native的调用都是虚调用，这种纯字面上的调用都放入一个缓存队列里，待事件触发时，将整个缓存指令传递给native，native再一条条分析缓存里的指令，一次性执行完毕。这样做的好处是，减少native与js之间的环境切换，并且js并非有直接控制native执行的权限。

react native使用批量命令执行，还大量使用了懒加载模式，都是为了减少性能、资源消耗。正如官方所说，react native接近原生的性能。

## js如何调用OC

要想让js调用native，首先需要给js环境注入native的接口，react native是在RCTJSCExecutor.m中的setup方法，统一给js注入接口，核心接口有两个：

```objective-c
//RCTJSCExecutor.m

  [self addSynchronousHookWithName:@"nativeRequireModuleConfig" usingBlock:^NSString *(NSString *moduleName) {
    RCTJSCExecutor *strongSelf = weakSelf;
    if (!strongSelf.valid) {
      return nil;
    }

    RCT_PROFILE_BEGIN_EVENT(0, @"nativeRequireModuleConfig", nil);
    NSArray *config = [strongSelf->_bridge configForModuleName:moduleName];
    NSString *result = config ? RCTJSONStringify(config, NULL) : nil;
    RCT_PROFILE_END_EVENT(0, @"js_call,config", @{ @"moduleName": moduleName });
    return result;
  }];

  //js调用native总入口：js调用全部放入队列，然后批量执行。
  //calls就是js中缓存调用的队列
  [self addSynchronousHookWithName:@"nativeFlushQueueImmediate" usingBlock:^(NSArray<NSArray *> *calls){
    RCTJSCExecutor *strongSelf = weakSelf;
    if (!strongSelf.valid || !calls) {
      return;
    }

    RCT_PROFILE_BEGIN_EVENT(0, @"nativeFlushQueueImmediate", nil);
    [strongSelf->_bridge handleBuffer:calls batchEnded:NO];
    RCT_PROFILE_END_EVENT(0, @"js_call", nil);
  }];
```


一个是nativeRequireModuleConfig接口，另一个是nativeFlushQueueImmediate接口。

nativeRequireModuleConfig的功能是根据js传递给native的模块名称，返回该模块下的所有节点。那么`js传递过来的模块名称又从哪里获取的呢？`RCTBatchBridge.m中调用了RCTJSCExecutor的injectJSONText方法，注入了一个全局变量__fbBatchedBridgeConfig，这个变量里保存的就是native所有可调用的模块名称：

```objective-c
//RCTBatchBridge.m

- (void)injectJSONConfiguration:(NSString *)configJSON
                     onComplete:(void (^)(NSError *))onComplete
{
  if (!_valid) {
    return;
  }

  [_javaScriptExecutor injectJSONText:configJSON
                  asGlobalObjectNamed:@"__fbBatchedBridgeConfig"
                             callback:onComplete];
}
```


在看看js端是如何使用__fbBatchedBridgeConfig的。js端有个BatchedBridge.js用到了全局变量__fbBatchedBridgeConfig:

```js
const MessageQueue = require('MessageQueue');

const BatchedBridge = new MessageQueue(
  () => global.__fbBatchedBridgeConfig
);
```

它把__fbBatchedBridgeConfig传递给了MessageQueue模块，再看MessageQueue.js：

```js
  constructor(configProvider: () => Config) {

    ... 省略若干初始化代码

    [
      'invokeCallbackAndReturnFlushedQueue',
      'callFunctionReturnFlushedQueue',
      'flushedQueue',
    ].forEach((fn) => this[fn] = this[fn].bind(this));

    lazyProperty(this, 'RemoteModules', () => {
      let {remoteModuleConfig} = configProvider(); //返回的是global.__fbBatchedBridgeConfig
      let modulesConfig = this._genModulesConfig(remoteModuleConfig);  //过滤数据格式，返回模块数组
      let modules = this._genModules(modulesConfig); //

      this._genLookupTables(
        modulesConfig, this._remoteModuleTable, this._remoteMethodTable
      );

      return modules;
    });
  }
```

由上面代码可以看出，MessageQueue把__fbBatchedBridgeConfig数据再加工后保存到自己的`RemoteModules`属性上了。

> 注：这里的lazyProperty函数就是懒加载（绑定），只有真正访问RemoteModules属性时，才计算该属性值。

真正调用nativeRequireModuleConfig接口的是在NativeModules.js中：

```js
const NativeModules = {};
Object.keys(RemoteModules).forEach((moduleName) => {
  Object.defineProperty(NativeModules, moduleName, {
    configurable: true,
    enumerable: true,
    get: () => {
      let module = RemoteModules[moduleName];
      if (module && typeof module.moduleID === 'number' && global.nativeRequireModuleConfig) {
        const json = global.nativeRequireModuleConfig(moduleName);
        const config = json && JSON.parse(json);
        module = config && BatchedBridge.processModuleConfig(config, module.moduleID);
        RemoteModules[moduleName] = module;
      }
      Object.defineProperty(NativeModules, moduleName, {
        configurable: true,
        enumerable: true,
        value: module,
      });
      return module;
    },
  });
});
```

> 这里又用到懒加载模式，react native中随处可见。

再来看看nativeFlushQueueImmediate接口，在MessageQueue.js中有个__nativeCall方法调用了该接口，这个方法是js调用native方法的总入口：

```js
//MessageQueue.js
  __nativeCall(module, method, params, onFail, onSucc) {

    ... 省略若干代码

    //调用native方法，先把调用放入队列缓存起来，等待时机，一次把缓存方法执行完毕
    this._queue[MODULE_IDS].push(module);
    this._queue[METHOD_IDS].push(method);
    this._queue[PARAMS].push(params);

    //这里把需要调用native的函数队列传递给native，native根据模块、函数、参数规则解析调用
    var now = new Date().getTime();
    if (global.nativeFlushQueueImmediate &&
        now - this._lastFlush >= MIN_TIME_BETWEEN_FLUSHES_MS) {
      global.nativeFlushQueueImmediate(this._queue);
      this._queue = [[], [], [], this._callID];
      this._lastFlush = now;
    }
    ... 省略若干代码
  }
```

可以看到，所有的native调用都转化为统一的形式：模块+方法+参数，放入缓存队列，然后调用nativeFlushQueueImmediate时将缓存队列传递给native，由native一次处理完缓存队列调用。

由此可见，nativeRequireModuleConfig接口native给js提供基础信息，而nativeFlushQueueImmediate接口是真正js调用native的入口。

## OC如何调用js

在RCTJSCExecutor.mm中有个_executeJSCall方法，这个方法是调用js的总入口：

```js
//RCTJSCExecutor.mm

- (void)_executeJSCall:(NSString *)method
             arguments:(NSArray *)arguments
              callback:(RCTJavaScriptCallback)onComplete
{
    ...省略若干代码
    // get the BatchedBridge object
    JSStringRef moduleNameJSStringRef = JSStringCreateWithUTF8CString("__fbBatchedBridge");
    JSValueRef moduleJSRef = JSObjectGetProperty(contextJSRef, globalObjectJSRef, moduleNameJSStringRef, &errorJSRef);
    JSStringRelease(moduleNameJSStringRef);

    if (moduleJSRef != NULL && errorJSRef == NULL && !JSValueIsUndefined(contextJSRef, moduleJSRef)) {
      // get method
      JSStringRef methodNameJSStringRef = JSStringCreateWithCFString((__bridge CFStringRef)method);
      JSValueRef methodJSRef = JSObjectGetProperty(contextJSRef, (JSObjectRef)moduleJSRef, methodNameJSStringRef, &errorJSRef);
      JSStringRelease(methodNameJSStringRef);

      if (methodJSRef != NULL && errorJSRef == NULL && !JSValueIsUndefined(contextJSRef, methodJSRef)) {
        JSValueRef jsArgs[arguments.count];
        for (NSUInteger i = 0; i < arguments.count; i++) {
          jsArgs[i] = [JSValue valueWithObject:arguments[i] inContext:context].JSValueRef;
        }
        resultJSRef = JSObjectCallAsFunction(contextJSRef, (JSObjectRef)methodJSRef, (JSObjectRef)moduleJSRef, arguments.count, jsArgs, &errorJSRef);
      } else {
        if (!errorJSRef && JSValueIsUndefined(contextJSRef, methodJSRef)) {
          error = RCTErrorWithMessage([NSString stringWithFormat:@"Unable to execute JS call: method %@ is undefined", method]);
        }
      }
    } else {
      if (!errorJSRef && JSValueIsUndefined(contextJSRef, moduleJSRef)) {
        error = RCTErrorWithMessage(@"Unable to execute JS call: __fbBatchedBridge is undefined");
      }
    }

    if (errorJSRef || error) {
      if (!error) {
        error = RCTNSErrorFromJSError(contextJSRef, errorJSRef);
      }
      onComplete(nil, error);
      return;
    }

    // Looks like making lots of JSC API calls is slower than communicating by using a JSON
    // string. Also it ensures that data stuctures don't have cycles and non-serializable fields.
    // see [RCTJSCExecutorTests testDeserializationPerf]
    id objcValue;
    // We often return `null` from JS when there is nothing for native side. JSONKit takes an extra hundred microseconds
    // to handle this simple case, so we are adding a shortcut to make executeJSCall method even faster
    if (!JSValueIsNull(contextJSRef, resultJSRef)) {
      objcValue = [[JSValue valueWithJSValueRef:resultJSRef inContext:context] toObject];
    }

    onComplete(objcValue, nil);
  }), 0, @"js_call", (@{@"method": method, @"args": arguments}))];
}
```

可以看见从js全局中获取__fbBatchedBridge对象，然后调用该对象上的方法并获取返回值，在看看工程中有三处调用了该方法：

```objective-c
- (void)flushedQueue:(RCTJavaScriptCallback)onComplete
{
  // TODO: Make this function handle first class instead of dynamically dispatching it. #9317773
  [self _executeJSCall:@"flushedQueue" arguments:@[] callback:onComplete];
}

- (void)callFunctionOnModule:(NSString *)module
                      method:(NSString *)method
                   arguments:(NSArray *)args
                    callback:(RCTJavaScriptCallback)onComplete
{
  // TODO: Make this function handle first class instead of dynamically dispatching it. #9317773
  [self _executeJSCall:@"callFunctionReturnFlushedQueue" arguments:@[module, method, args] callback:onComplete];
}

- (void)invokeCallbackID:(NSNumber *)cbID
               arguments:(NSArray *)args
                callback:(RCTJavaScriptCallback)onComplete
{
  // TODO: Make this function handle first class instead of dynamically dispatching it. #9317773
  [self _executeJSCall:@"invokeCallbackAndReturnFlushedQueue" arguments:@[cbID, args] callback:onComplete];
}
```

一共调用了该对象上的三个方法flushedQueue、callFunctionReturnFlushedQueue和invokeCallbackAndReturnFlushedQueue，也就意味着js侧必然有个__fbBatchedBridge对象并实现了这三个方法：

```js
//MessageQueue.js
  callFunctionReturnFlushedQueue(module, method, args) {
    guard(() => {
      this.__callFunction(module, method, args);
      this.__callImmediates();
    });

    return this.flushedQueue();
  }

  invokeCallbackAndReturnFlushedQueue(cbID, args) {
    guard(() => {
      this.__invokeCallback(cbID, args);
      this.__callImmediates();
    });

    return this.flushedQueue();
  }

  flushedQueue() {
    this.__callImmediates();

    let queue = this._queue;
    this._queue = [[], [], [], this._callID];
    return queue[0].length ? queue : null;
  }
```

而__fbBatchedBridge是在BatchedBridge.js中定义：

```js
//BatchedBridge.js
Object.defineProperty(global, '__fbBatchedBridge', { value: BatchedBridge });
```

上面这两个接口用于模块调用，还有几个接口是用于事件分发的，即将native的事件传递给js，让js具有处理事件的能力。

```objective-c
//RCTEventDispatcher.m
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
  if (RCT_DEBUG) {
    RCTAssert([body[@"target"] isKindOfClass:[NSNumber class]],
      @"Event body dictionary must include a 'target' property containing a React tag");
  }

  name = RCTNormalizeInputEventName(name);
  [_bridge enqueueJSCall:@"RCTEventEmitter.receiveEvent"
                    args:body ? @[body[@"target"], name, body] : @[body[@"target"], name]];
}
```

分别用于传递APP事件、设备事件和输入事件。跟踪一下调用，发现最终调用的还是callFunctionReturnFlushedQueue方法，该方法在MessageQueue.js中，它又调用了下面方法：

```js
  //MessageQueue.js
  __callFunction(module: string, method: string, args: any) {
    ... 省略若干代码

    //返回可调用的对象，然后调用该对象的具体方法method
    //_callableModules是由js端注册的模块（每个native模块对应一个js模块，js模块需要调用register注册到此数组中）
    var moduleMethods = this._callableModules[module];
    invariant(
      !!moduleMethods,
      'Module %s is not a registered callable module.',
      module
    );
    moduleMethods[method].apply(moduleMethods, args);
    Systrace.endEvent();
  }
```

可以看到`__callFunction`的功能是消息分发，最终消息都发送到了各个js模块中。


## 结尾

react native中OC与js之间的交互基本就清楚了。可以看到无论native还是js，都是组织成模块的形式进行管理调用。


