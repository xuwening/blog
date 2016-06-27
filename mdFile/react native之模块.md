
# react native之模块

## native模块

在native中用RCTModuleData代表一个导出模块，它包含一个模块的整体信息，包括模块名、模块类、模块的导出方法、模块的导出常量、导出属性、模块需要运行的队列、模块实例是否需要在主线程中创建等。

#### 如何导出一个模块

导出一个native模块，使用RCT_EXPORT_MODULE宏，展开是如下形式：

```objective-c
extern __attribute__((visibility("default"))) void RCTRegisterModule(Class); 

+ (NSString *)moduleName { 
    return @""; 
} 

+ (void)load { 
    RCTRegisterModule(self); 
}
```

该宏的目的是在load类时，调用RCTRegisterModule函数，注册自身，RCTRegisterModule函数定义在RCTBridge.m中：

```objective-c
static NSMutableArray<Class> *RCTModuleClasses;

void RCTRegisterModule(Class moduleClass)
{
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    RCTModuleClasses = [NSMutableArray new];
  });

  RCTAssert([moduleClass conformsToProtocol:@protocol(RCTBridgeModule)],
            @"%@ does not conform to the RCTBridgeModule protocol",
            moduleClass);

  // Register module
  [RCTModuleClasses addObject:moduleClass];
}
```

所谓注册，其实就是全局保存该类。app初始化时，会调用RCTBatchedBridge的initModulesWithDispatchGroup方法，该方法会扫描所有导出的类，并将导出类包装成RCTModuleData对象，

#### 如何导出一个模块的方法

使用RCT_EXPORT_METHOD宏，展开形式(以RCTActionSheetManager为例)：

```objective-c
+ (NSArray<NSString *> *)__rct_export__520 { 
    return @[@"", @"showActionSheetWithOptions:(NSDictionary *)options callback:(RCTResponseSenderBlock)callback"]; 
} 

- (void)showActionSheetWithOptions:(NSDictionary *)options callback:(RCTResponseSenderBlock)callback {
    ...
}
```

可以看到，RCT_EXPORT_METHOD对导出函数并没有任何改变，仅仅定义了一个以__rct_export__命名的函数，返回导出函数的字符串描述。

RCTModuleData中methods方法，是获取导出模块中的导出函数，其中判断依据就是以__rct_export__为前缀，将return的字符串作为函数名称，然后封装为RCTModuleMethod对象。

#### 如何导出一个模块的属性


1、使用RCT_EXPORT_VIEW_PROPERTY宏：

```objective-c
+ (NSArray<NSString *> *)propConfig_blurRadius { 
    return @[@"CGFloat"]; 
}
```

将属性加以前缀propConfig_，并返回该属性的数据类型。

2、使用RCT_REMAP_VIEW_PROPERTY宏：

```objective-c
+ (NSArray<NSString *> *)propConfig_defaultSource { 
    return @[@"UIImage", @"defaultImage"]; 
}
```

返回访问该属性的数据类型以及keypath。

3、使用RCT_CUSTOM_VIEW_PROPERTY宏，自定义属性：

```objective-c
+ (NSArray<NSString *> *)propConfig_tintColor { 
    return @[@"UIColor", @"__custom__"]; 
} 

- (void)set_tintColor:(id)json forView:(RCTImageView *)view withDefaultView:(RCTImageView *)defaultView {

  view.tintColor = [RCTConvert UIColor:json] ?: defaultView.tintColor;
  view.renderingMode = json ? UIImageRenderingModeAlwaysTemplate : defaultView.renderingMode;
}
```

还有一个宏是RCT_EXPORT_SHADOW_PROPERTY，用来定义shadow view属性的，功能和RCT_EXPORT_VIEW_PROPERTY宏相同。

通过propConfig_前缀是用以区分是普通方法还是导出方法，在RCTComponentData的viewConfig方法中扫描propConfig前缀，作为js可访问属性。

RCTComponentData相当于UI组件的封装，也就是对RCTViewManager的封装。

#### 如何导出常量

模块必须实现RCTBridgeModule协议，该协议的可选协议里有个constantsToExport方法，如果需要导出常量供js使用，可以实现该方法。

```objective-c
- (NSDictionary<NSString *, id> *)constantsToExport {
    ...
}
```


#### 导出方法在哪个线程执行

实现模块协议methodQueue方法，可指定执行方法的队列。

```objective-c
- (dispatch_queue_t)methodQueue {
    ...
}
```


## native模块在js侧的映射

native模块一般都会对应一个js模块，即js对native模块调用再封装，也是js侧模块和native侧模块的一一映射。而js侧的模块其实就是modules中的factory要执行的代码，这种封装方式是为了实现native接口的延迟加载，不会将native的所有模块和接口全部加载到js端。

objc在启动时给js context注册一个全局对象：__fbBatchedBridgeConfig，其中包含了所有导出对象以及导出的方法。BatchedBridge.js会创建全局对象__fbBatchedBridge，它其实就是MessageQueue的实例，MessageQueue实例化时通过__fbBatchedBridgeConfig获取native的导出对象及方法，将对象及方法转换为js端的映射存放在RemoteModules中，所有的native方法调用都统一被替换成为__nativeCall调用。

__fbBatchedBridgeConfig的展示形式如下：

```js
{
    "remoteModuleConfig": [
        null,
        [
            "CalendarManager",
            {
                "strKey": 4
            },
            [
                "addEvent",
                "addEvent",
                "addEvent",
                "findEvents",
                "findEvents"
            ],
            [
                4
            ]
        ],
        [
            "RCTUIManager",
            {
                "RCTTextView": {
                    "Manager": "RCTTextViewManager",
                    "NativeProps": {
                        "text": "NSString",
                        "color": "UIColor",
                        "clearTextOnFocus": "BOOL",
                        "autoCorrect": "BOOL",
                        "autoCapitalize": "UITextAutocapitalizationType",
                        "placeholder": "NSString",
                        "secureTextEntry": "BOOL",
                        "blurOnSubmit": "BOOL",
                        "keyboardType": "UIKeyboardType",
                        "mostRecentEventCount": "NSInteger",
                        "fontWeight": "NSString",
                        "returnKeyType": "UIReturnKeyType",
                        "fontStyle": "NSString",
                        "placeholderTextColor": "UIColor",
                        "onSelectionChange": "BOOL",
                        "fontSize": "CGFloat",
                        "enablesReturnKeyAutomatically": "BOOL",
                        "fontFamily": "NSString",
                        "editable": "BOOL",
                        "textAlign": "NSTextAlignment",
                        "keyboardAppearance": "UIKeyboardAppearance",
                        "maxLength": "NSNumber",
                        "selectionColor": "UIColor",
                        "selectTextOnFocus": "BOOL"
                    }
                },
                "RCTSwitch": {
                    "Manager": "RCTSwitchManager",
                    "NativeProps": {
                        "thumbTintColor": "UIColor",
                        "tintColor": "UIColor",
                        "onTintColor": "UIColor",
                        "value": "BOOL",
                        "onChange": "BOOL",
                        "disabled": "BOOL"
                    }
                },

                ...
            }
        ],

        ...
    ],
}
```

***注意view类的模块都在RCTUIManager下管理。Manager是管理类，NativeProps是导出的属性。***

转化native函数调用为__nativeCall：

```js
_genMethod(module, method, type) {
    
    ...省略若干代码

    //统一函数定义：即所有调用native方法都是此定义
    fn = function(...args) {

        //获取最后两个参数：如果有的话
        let lastArg = args.length > 0 ? args[args.length - 1] : null;
        let secondLastArg = args.length > 1 ? args[args.length - 2] : null;

        //如果最后两个参数是函数：主要是处理成功、失败回调
        let hasSuccCB = typeof lastArg === 'function';
        let hasErrorCB = typeof secondLastArg === 'function';
        hasErrorCB && invariant(
          hasSuccCB,
          'Cannot have a non-function arg after a function arg.'
        );

        let numCBs = hasSuccCB + hasErrorCB;
        let onSucc = hasSuccCB ? lastArg : null;
        let onFail = hasErrorCB ? secondLastArg : null;
        args = args.slice(0, args.length - numCBs);

        //所有调用native的函数均采用统一调用方式
        return self.__nativeCall(module, method, args, onFail, onSucc);
    };

    fn.type = type;
    return fn;    
}
```

## js调用native方法的具体流程

以简单调用举例，如调用系统的振动功能：

```js
Vibration.vibrate();
```

require模块Vibration时，只是进行了命名映射，实际并没有加载Vibration.js，在第一次调用Vibration时才执行factory加载动作，也就是执行Vibration.js,源码中有两条核心语句：

```js
var RCTVibration = require('NativeModules').Vibration;

RCTVibration.vibrate();
```

`Vibration.vibrate();`调用了native方法`RCTVibration.vibrate();`。RCTVibration是从NativeModules模块加载的：

```js
const NativeModules = {};

//根据RemoteModules定义懒加载模式的NativeModules
Object.keys(RemoteModules).forEach((moduleName) => {
  Object.defineProperty(NativeModules, moduleName, {
    configurable: true,
    enumerable: true,
    get: () => {

      //这里是关键
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

前面说了，native导出模块的所有方法都被保存在RemoteModules中，并且方法都被统一执行__nativeCall，NativeModules最终还是将函数调用转到RemoteModules中的对象方法，开始执行__nativeCall。

为什么要这么绕呢，还是那句话，延迟加载。

## js端的模块

js侧除了和native对应的模块之外，还有自己的模块，用于native直接调用，如事件、timer等操作。

通过前面的分析，oc调用js是通过`__callFunction`进行分发调用的：

```js
var moduleMethods = this._callableModules[module];
moduleMethods[method].apply(moduleMethods, args);
```

oc调用js需要提供module和method以及参数，这里就是通过module查找并调用。那么`_callableModules`是什么？可以看到MessageQueue中还有这样一个函数：

```js
registerCallableModule(name, methods) {
    this._callableModules[name] = methods;
}
```

`registerCallableModule`是用来注册js端的模块的，凡事被注册过的js模块，才可以被oc调用。所以自定义js处理模块需要被oc访问，需调用`BatchedBridge.registerCallableModule`对模块进行注册。










