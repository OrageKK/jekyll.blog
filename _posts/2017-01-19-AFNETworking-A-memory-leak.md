---
layout:     post
title:      "AFNetworking A memory leak"
subtitle:   "Bug，iOS"
date:       2017-01-19
author:     "oragekk"
header-img: "img/post-bg-unix-linux.jpg"
tags:

    - iOS
    - Bug录 
---

>细心的你是否也发现了AFN的内存泄漏的问题了呢.

### 解决方法
    + (AFHTTPSessionManager *)sharedHTTPSession{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        manager = [AFHTTPSessionManager manager];
        manager.requestSerializer.timeoutInterval = 30;
        [manager.requestSerializer  setValue:@"XMLHttpRequest" forHTTPHeaderField:@"X-Requested-With"];
    });
    return manager;
    }

    + (AFURLSessionManager *)sharedURLSession{
    static dispatch_once_t onceToken2;
    dispatch_once(&onceToken2, ^{
        urlsession = [[AFURLSessionManager alloc] initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    });
    return urlsession;
    }
    
>将有问题的语句全部替换成单例后，再用instruments检查，再也没有出现泄漏的红叉了。O(∩_∩)O哈哈~