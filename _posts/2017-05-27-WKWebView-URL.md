---
layout:     post
title:      "WKWebView拦截URL"
subtitle:   "iOS，前端开发，杂货铺"
date:       2017-05-27
author:     "oragekk"
header-img: "img/post-bg-2016-11-3.jpg"
tags:

    - iOS
    - 前端开发
    - 杂货铺 
---
> 本文介绍使用WKWebView拦截url进行原生界面跳转

[![3.md.gif](https://storage1.cuntuku.com/2017/05/27/3.md.gif)](https://cuntuku.com/image/48Ct0)

* 使用代理方法decidePolicyForNavigationAction

``` objc
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    // 获取完整url并进行UTF-8转码
    NSString *strRequest = [navigationAction.request.URL.absoluteString stringByRemovingPercentEncoding];
    if ([strRequest hasPrefix:@"app://"]) {
        // 拦截点击链接
        [self handleCustomAction:strRequest];
        // 不允许跳转
      	decisionHandler(WKNavigationActionPolicyCancel);
    }else {
    	// 允许跳转
        decisionHandler(WKNavigationActionPolicyAllow);
        
    }
}
```

* 自定义方法传过来url进行判断，需要html元素本身就有跳转链接，才可以拦截，如没有，拦截不到。下文app://xxx链接为自定义链接

``` objc
- (void)handleCustomAction:(NSString *)URL
{
    // 判断跳转
    NSString *link_id = @"";
    if ([URL hasPrefix:@"app://video"]) {
        // 视频
        MMLog(@"点击了视频%@",link_id);
    }else if ([URL hasPrefix:@"app://item"]) {
        // 单品
        MMLog(@"点击了单品%@",link_id);
    }else if ([URL hasPrefix:@"app://brand"]) {
        // 品牌
        link_id = [URL substringFromIndex:[NSString stringWithFormat:@"app://brand"].length];
        MMLog(@"点击了品牌%@",link_id);
    }
}
```