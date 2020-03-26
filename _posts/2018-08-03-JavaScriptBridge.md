---
layout:     post
title:      "WebViewJavascriptBridge"
subtitle:   "Native & H5 The messaging"
date:       2018-08-03 09:48:33
author:     "oragekk"
header-img: "img/post-bg-ios-demo-testcallphone.jpg"
catalog: true
tags:

    - iOS
    - 前端开发
    - 杂货铺 
---
> 最近一直在忙，今天抽空写一下H5和Native的交互




## 一、选择
* 项目本身webview使用的是WKWebview，其实WKWebview自带的messageHandle也可以满足此需求
* JSContext,源自于JavaScriptCore框架中的东西，最后不使用此方案源于一下几点
  * 但是其中繁杂的字符串使用，让我觉的可能会由于粗心出现不可预知的错误
  * 加载时机的问题，当你重新loadrequest的时候，会导致js注入失败
  * 回调方法略复杂
* JavaScriptBridge，最后选择此库源于以下几点
  * 使用简单，注册完毕之后设置完代理，只需要负责注册方法和调用方法
  * 回调简单，两端回调responsecallback包含在注册的方法中。使用block
  * 三端通用，JavaScript和iOS、Android都可以（[Android版本库](https://github.com/wendux/WebViewJavascriptBridge)）
  * Ps :关于Android版本库，其中很多是按照iOS版的JavaScriptBridge改写的。但是其中有很多问题，尤其是各种调用时机问题，上面的链接是经过我旁边的Android小哥试了四五个版本之后发现的，修复了各种改写版的问题

## 二、使用

* 首先需要引入WebViewJavascriptBridge库

``` objc
#import "WebViewJavascriptBridge.h"
```
* 初始化，此处为了方便子类使用，所以在基类中注册bridge，并return bridge对象，方便子类调用

``` objc
#pragma mark - 桥接

- (void)InitializeWebViewJavascriptBridge {
    // 注册桥接
    self.bridge = [SWHybridManager setJavaScriptBridgeWithWebView:self.webView controller:self];
}

[WKWebViewJavascriptBridge enableLogging];
WKWebViewJavascriptBridge *bridge = [WKWebViewJavascriptBridge bridgeForWebView:webView];
[bridge setWebViewDelegate:controller];
```
* 注册方法供JavaScript调用

``` objc
__weak typeof(controller)wController = controller;
/****************************公共方法注册-Start*********************/
    //MARK:打开窗体
    [bridge registerHandler:HandlerFunctionNameOpenWindow handler:^(id data, WVJBResponseCallback responseCallback) {
        __strong typeof(wController)sController = wController;
        NSLog(@"\n调用了openWindow: %@", data);
        NSDictionary *dict = (NSDictionary *)data;
        SWOpenWindowModel *model = [SWOpenWindowModel yy_modelWithDictionary:dict];
        [self pushViewController:sController data:model responseCallback:responseCallback];
    }];
```
-  调用JavaScript方法

```objc
[self.bridge callHandler:callFunctionNameGetSearchKeyWord data:json];
```

## 三、方法名定义

- 因为方法名的定义是字符串，所以建议采用常量字符串，防止拼写错误
- 其次不建议采用宏定义
- 我采用以下方法
- 桥接管理类的.h

``` objc
/**打开窗体 */
FOUNDATION_EXPORT  NSString *const HandlerFunctionNameOpenWindow;
/** 关闭窗口*/
FOUNDATION_EXPORT  NSString *const HandlerFunctionNameCloseWindow;
```

- 桥接管理类的.m

``` objc
/**打开窗体 */
NSString *const HandlerFunctionNameOpenWindow = @"openWindow";
/** 关闭窗口*/
NSString *const HandlerFunctionNameCloseWindow = @"closeWindow";
```
- 使用时直接使用常量字符串即可
- 注意点：如类似我使用在基类传入控制器和webview到管理类中，在类中使用controller要注意循环引用，否则会导致控制器无法释放

``` objc
+ (WKWebViewJavascriptBridge *)setJavaScriptBridgeWithWebView:(WKWebView *)webView controller:(__kindof SWBaseWebViewController *)controller;

__weak typeof(controller)wController = controller; // 弱引用传入控制器
__strong typeof(wController)sController = wController; // 在block内部强引用
```
## 四、数据传输

- iOS端直接返回字典即可
- 我代码中是返回json字符串，为了与Android统一，方便H5解析数据

## 五、JavaScript代码

```javascript
function setupWebViewJavascriptBridge(callback) {
	if (window.WebViewJavascriptBridge) { return callback(WebViewJavascriptBridge); }
	if (window.WVJBCallbacks) { return window.WVJBCallbacks.push(callback); }
	window.WVJBCallbacks = [callback];
	var WVJBIframe = document.createElement('iframe');
	WVJBIframe.style.display = 'none';
	WVJBIframe.src = 'https://__bridge_loaded__';
	document.documentElement.appendChild(WVJBIframe);
	setTimeout(function() { document.documentElement.removeChild(WVJBIframe) }, 0)
}
```

```javascript
setupWebViewJavascriptBridge(function(bridge) {
	
	/* Initialize your app here */

	bridge.registerHandler('JS Echo', function(data, responseCallback) {
		console.log("JS Echo called with:", data)
		responseCallback(data)
	})
	bridge.callHandler('ObjC Echo', {'key':'value'}, function responseCallback(responseData) {
		console.log("JS received response:", responseData)
	})
})
```

## 六、注意事项

- 如果产生调用不通的问题，多为JavaScript调用时机问题
- 注意桥接的代理