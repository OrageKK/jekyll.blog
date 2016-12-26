---
layout:     post
title:      "iOS - NSOperatioin"
subtitle:   "多线程，iOS开发，Demo 归纳笔记"
date:       2016-12-26 13:39:59
author:     "oragekk"
header-img: "img/post-bg-ios-demo-testcallphone.jpg"
tags:

    - iOS
    - 工具集
    - 多线程    
     
---
> # 一. NSOperatioin

#### 1.目的
- 开启线程
- 提供一些GCD不具备的功能
- OC框架,内部封装的是GCD

####2.区别
- GCD执行效率高于NSOperation
- NSOperation提供了一些GCD中不具备的功能(暂停/恢复/取消)---管理操作-NSOperation的高级用法


####3.NSOperation本身是一个抽象类不可以直接使用,实际开发中使用其子类
- 苹果提供了两个原生子类
  - NSInvocationOperation 
  <pre><code>NSInvocationOperation *op = [[NSInvocationOperation alloc]<br />initWithTarget:self selector:@selector(test) object:nil];</code></pre>
  - NSBlockOperation
    - 将操作封装在block中
      <pre>NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        
        NSLog(@"操作1---------%@",[NSThread currentThread]);
    }];</pre>
    - 可以追加操作(无限)---如果在非主队列执行操作,原操作和追加的操作都会开启多条线程去执行
    <pre>
[op1 addExecutionBlock:^{
        
        NSLog(@"追加操作2-----------%@",[NSThread currentThread]);
    }];</pre>
    - 缺点:
				1) 有过在主队列执行操作,有一个操作在主线程执行(随机),其他操作在子线程执行
				2) 如果直接调用 start 方法执行操作,无法确定每一个操作在哪条线程执行
				3) 一般不要追加操作,除非就是在非主队列执行
    - 操作依赖 (串行/线程同步技术)添加数量不要太多
          1)对于不同的操作队列中的操作依然有效
          2)添加操作依赖要在添加操作队列之前
          3)只能对添加在操作队列的操作添加依赖
          4)一定不要添加循环依赖
------------------------
> # 一. NSOperatioinQueue

#### 1.主队列
  - 放在主队列中的操作,都在主线程执行
<pre><code>NSOPerationQueue mainQueue</code></pre>

#### 2.非主队列
  - 放在非主队列中的操作,都在子线程执行
<pre><code>[[NSOPerationQueue alloc] init]</code></pre>

#### 3.一般定义成全局属性
-----------------
#### 4.每一个操作都有一个start方法,用来在当前线程执行

- 本质:将操作添加到操作队列之后,内部会自动调用内部start方法,操作就会自动执行

- 主队列和非主队列决定操作在哪条线程执行(在哪条线程启动操作的start方法)
