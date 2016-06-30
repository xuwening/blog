
# UITextField和UITextView输入长度的限制

## 前言

项目中经常会对文本输入的长度做出限制，这种限制通常有两种形式：一种是用户输入完毕进入下一步操作时，程序对文本做校验，如果校验不通过给用户提示；另一种是对输入直接做限制，也就是不给用户输入限制外的机会。

这里我们讨论第二种形式，这种方式用户体验相对来说更好些。技术实现上也也没有什么难度，系统提供了文本变化的通知：UITextField对应的通知是`UITextFieldTextDidChangeNotification`，UITextView对应的是`UITextViewTextDidChangeNotification`。只要我们注册该通知，实时对变化的文本进行校验即可。

这里有个问题，就是一个app中文本框的数量可能非常多，如果对每个控件都写一份代码，就算`copy代码`也是不小的工作量，而且维护起来比较困难。于是想做到统一去处理，最后写了一个简单的库，专门用来限制输入文本的长度。

几年过去了，也不知道现在是不是有更好更强大的库来处理长度限制。这个库早就放在了github上，也没有做宣传，用的人也不多，现在进行纪念一下，因为这是我做iOS实现的第一个库。

库的地址：[TextInputLimit](https://github.com/xuwening/textInputLimit)

## 监控所有的UITextField和UITextView

其实监控UITextField和UITextView输入是系统的工作，只是提供了用户干预的手段，就是前面说的通知。第一步先注册通知，进行监控：

```objective-c
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textFieldViewDidChange:) name:UITextFieldTextDidChangeNotification object: nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textViewDidChange:) name:UITextViewTextDidChangeNotification object: nil];

```

具体观察者是谁呢？这是实现全局监控的核心，为了监控所有的输入统一处理，就不要和任何具体页面以及具体ViewController相关，直接放到一个单例上，单独做这件事。那如何知道当前输入框是不是自己监控对象，以及区分每个输入框的长度限制？在接收到文本变化通知，具体输入框会作为参数被携带过来，因此只要将限制的长度绑定到具体输入框上，然后在接收到通知时就不用关心具体是哪个输入框。

以UITextField为例：

```objective-c
-(void)textFieldViewDidChange:(NSNotification*)notification {
    
    UITextField *textField = (UITextField *)notification.object;
    //获取输入框的长度限制
    NSNumber *number = [textField valueForKey:PROPERTY_NAME];
    if (number && textField.text.length > [number integerValue] {
        //做长度限制处理
    }
}
```

这里为什么用valueForKey而不直接访问对象属性？因为要进行解耦，即尽量剥离与其他类的关联，这样库才能独立。对输入框加上`限制长度属性`可以采用运行时关联对象的特性：

```objective-c
-(id)valueForUndefinedKey:(NSString *)key {
    if ([key isEqualToString:propertyName]) {
        return objc_getAssociatedObject(self, key.UTF8String);
    }
    return nil;
}
-(void)setValue:(id)value forUndefinedKey:(NSString *)key {
    if ([key isEqualToString:propertyName]) {
        objc_setAssociatedObject(self, key.UTF8String, value, OBJC_ASSOCIATION_RETAIN);
    }
}
```

所以给具体UITextView设置限制长度属性时，是动态添加上去的。当用户输入长度超过限制时，对超出的文本自动删除，这样就实现还有个好处：`即时拷贝过长文本也会自动截断进行限制`。

```objective-c
-(void)textFieldViewDidChange:(NSNotification*)notification {
    
    UITextField *textField = (UITextField *)notification.object;
    
    NSNumber *number = [textField valueForKey:PROPERTY_NAME];
    if (number && textField.text.length > [number integerValue] && textField.markedTextRange == nil) {
        textField.text = [textField.text substringWithRange: NSMakeRange(0, [number integerValue])];
    }
}
```

`textField.markedTextRange == nil`的判断是为了兼容中文输入处理，否则输入中文会引起崩溃。

最后，为了彻底解耦，防止头文件依赖，让观察者在类加载时自动创建，这样连初始化类实例都省了。


## 到达限制长度时的反馈

实现长度自动截取，本身对用户来说就是一种反馈：`你输入的数据只能到这儿了。`但有时可能还需要其他的特殊处理，比如弹个toast什么的。于是在超过限制时，再发个通知由接收者接收，如果接收者不想额外处理这种反馈，也不需要做任何操作，需要时才注册通知进行接收。

这种回馈可能采用协议更合理些，只是为了实现完全解耦，防止头文件依赖，直接采用通知的方式了。完整的处理：

```objective-c
-(void)textFieldViewDidChange:(NSNotification*)notification {
    if (!self.enableLimitCount) return;
    UITextField *textField = (UITextField *)notification.object;
    
    NSNumber *number = [textField valueForKey:PROPERTY_NAME];
    if (number && textField.text.length > [number integerValue] && textField.markedTextRange == nil) {
        textField.text = [textField.text substringWithRange: NSMakeRange(0, [number integerValue])];
        [[NSNotificationCenter defaultCenter] postNotificationName:@"acceptLimitLength" object: textField];
    }
}
```

## 如何使用

只需要把`LimitInput.h`和`LimitInput.m`引用到工程中，然后在需要进行长度限制的地方设置限制长度：

```objective-c
//限制该输入框长度为4
[self.textfield setValue:@4 forKey:@"limit"];
```

这样就完成了整个限制流程，很方便不是。`注意这里使用setValue: forKey`。

如果需要特殊处理，可以注册通知`acceptLimitLength`，当输入超出限制时会接收到通知，在通知处理回调中判断：是否本输入框超出限制，如果是就做特殊处理。

```objective-c
//注册通知
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(textLimitLenght:) name:@"acceptLimitLength" object:nil];

//接收到长度超出通知
-(void) textLimitLenght: (NSNotification *) notification {
    
    NSObject *object = notification.object;
    
    if ([object isEqual: self.textview]) {
        //收到来自textview的输入限制
    }
    
    if ([object isEqual: self.textfield]) {
        //收到来自textfield的输入限制
    }
    //提示
    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"" message:@"您输入的长度过长，自动被截断。" delegate:self cancelButtonTitle:@"确定" otherButtonTitles:nil, nil];
    [alert show];
}
```

## 后续

这个库的好处时，在你需要的地方设置长度限制，不需要的地方不受任何影响。也就是说你把库添加到工程中，或从工程中移除都没有任何影响，项目都会正常编译和运行。比如你在项目中使用了该库，并在某处设置`[self.textfield setValue:@4 forKey:@"limit"];`，然后移除该库，项目照样正常运行（只是长度限制不再起作用而已）。

另外这个库非常小，也很容易修改实现订制需求，比如对数字或表情符号处理。如果有需要，尽管拿去用吧，别客气！

喜欢的话请给颗星吧：[TextInputLimit](https://github.com/xuwening/textInputLimit)



