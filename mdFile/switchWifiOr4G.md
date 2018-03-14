# 切换WiFi和4G网络

## 原始需求

原始需求是如何避免用户手动输入手机号，实现自动登录，所谓自动登录的前提就是可以自动获取手机号。

以下三种方式要实现自动登录：

1. 隐式短信（android）
2. 4G网络状态下
3. WiFi和4G同开的状态下

手机号的获取，大家都知道android和iOS都是不可以的，但是android可以通过隐式发送短信，短信网关可以获取到手机号，然后和后台对接推送到后台从而实现手机号的获取。iOS就不行了，因为iOS无法进行隐式发短信。4G网络方式下可以通过移动基站获取手机号码，前提是移动基站要和后台对接推送手机号。

短信自动登录早已实现，现在需要实现4G网络状态下的自动登录，好消息是移动基站和后台对接已经建设完毕，纯4G状态下流程验证没有问题。剩下的问题就是如何在WiFi和4G同开的情况下，让指定的接口走4G网络发送，其他接口走WiFi。


## 基本原理

手机中有wifi模块和移动网络模块，可以看做两个独立网卡，每个网卡对应一个ip地址，分别是以10开头移动网络ip，和以192开头的wifi局域网ip。也就是说wifi和4G同开的情况下，会有两个ip。
在网络通信中，我们必须指定服务器地址，而本地地址一般交给系统自动去选择。要实现单独接口走4G网络，只需要指定本地采用哪个网卡发送数据即可，而可以指定本地ip发送数据，只有传输层可以做到，也就是socket。


## 实现步骤

#### 第一步：获取4G网络的IP地址

遍历本地网络名称为`pdp_ip0`即4G地址，获取网络信息函数为系统函数`getifaddrs`。

#### 第二步：socket绑定4G网络的IP地址

```c
//创建socket
int clientSocketId = socket(AF_INET, SOCK_STREAM, 0);
//将socket和4G网络ip绑定（addr中需要填充ip）
bind(clientSocketId, (const struct sockaddr *)&addr, sizeof(addr));
```

#### 第三步：使用socket发送数据

```c
//使用绑定4G ip的socket发送数据
send(clientSocketId, data, strlen(data), 0);
```


## 更好的实现

因为服务接口为http协议，采用socket需要自己解析http协议，并且需要处理一堆的网络异常问题，使用C/C++写过网络编程的应该都有体会，因此应采用第三方库来实现该需求，减轻工作量。

#### 获取4G ip的第三方库：

[System Services](https://github.com/Shmoopi/iOS-System-Services)

#### socket网络库：

[CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket)

#### 这样只要很少的代码就可以实现需求：

```objective-c
//获取4G IP
NSString *cellip = [[SystemServices sharedServices] cellIPAddress];    
//通过4G网络链接服务
if (![asyncSocket connectToHost:WWW_HOST onPort:port viaInterface:cellip withTimeout:-1 error:&error]) {

}

//然后再实现CocoaAsyncSocket的几个协议接口就可以了，协议接口有：

didConnectToHost  //链接上服务器后做什么处理，一般我们发送数据请求
didReadData	      //接收到服务器的响应数据如何处理
socketDidDisconnect  //发生网络错误，如何处理
```

