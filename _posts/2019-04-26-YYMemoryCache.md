---
layout:     post
title:      "YYMemoryCache Source Code Analysis"
subtitle:   "iOS，Source Code Analysis，杂货铺"
date:       2019-04-26 16:08:25
author:     "oragekk"
header-img: "img/post-bg-ios-demo-testcallphone.jpg"
catalog: true

tags:

    - iOS
    - Source Code Analysis
    - 杂货铺 
---

# YYMemoryCache 源码分析

#### YYMemoryCache是内存缓存，所以存取速度非常快，主要用到两种数据结构的LRU淘汰算法

1. LRU淘汰算法

   > LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。
   >
   > 最常见的实现是使用一个链表保存缓存数据
   >
   > 【命中率】
   >
   > 当存在热点数据时，LRU的效率很好，但偶发性的、周期性的批量操作会导致LRU命中率急剧下降，缓存污染情况比较严重。
   >
   > 
   >
   > Cache的容量是有限的，当Cache的空间都被占满后，如果再次发生缓存失效，就必须选择一个缓存块来替换掉。LRU法是依据各块使用的情况， 总是选择那个最长时间未被使用的块替换。这种方法比较好地反映了程序局部性规律

2. 数据结构

   - 双向链表 (Doubly Linked List)
   - 哈希表 (Dictionary)

3. 缓存操作

   - 新数据插入到链表头部；
   - 每当缓存命中（即缓存数据被访问），则将数据移到链表头部；
   - 当链表满的时候，将链表尾部的数据丢弃。

4. 分析图

   [![bpM38.png](https://storage6.cuntuku.com/2019/04/27/bpM38.png)](https://cuntuku.com/image/bpM38)

5. YYMemoryCache.m里的两个分类

   1. 链表节点 `_YYLinkedMapNode`

      ```objc
      @interface _YYLinkedMapNode : NSObject {
          @package
          // 指向前一个节点
          __unsafe_unretained _YYLinkedMapNode *_prev; // retained by dic
          // 指向后一个节点
          __unsafe_unretained _YYLinkedMapNode *_next; // retained by dic
          // 缓存key
          id _key;
          // 缓存对象
          id _value;
          // 当前缓存内存开销
          NSUInteger _cost;
          // 缓存时间
          NSTimeInterval _time;
      }
      @end
      ```

      
   
   2. 链表 `_YYLinkedMap`
   
      ```objc
      @interface _YYLinkedMap : NSObject {
          @package
          // 用字典保存所有节点_YYLinkedMapNode (为什么不用oc字典?因为用CFMutableDictionaryRef效率高，毕竟基于c)
          CFMutableDictionaryRef _dic;
          // 总缓存开销
          NSUInteger _totalCost;
          // 总缓存数量
          NSUInteger _totalCount;
          // 链表头节点
          _YYLinkedMapNode *_head;
          // 链表尾节点
          _YYLinkedMapNode *_tail;
          // 是否在主线程上，异步释放 _YYLinkedMapNode对象
          BOOL _releaseOnMainThread;
          // 是否异步释放 _YYLinkedMapNode对象
          BOOL _releaseAsynchronously;
      }
      // 添加节点到链表头节点
      - (void)insertNodeAtHead:(_YYLinkedMapNode *)node;
      // 移动当前节点到链表头节点
      - (void)bringNodeToHead:(_YYLinkedMapNode *)node;
      // 移除链表节点
      - (void)removeNode:(_YYLinkedMapNode *)node;
      // 移除链表尾节点(如果存在)
      - (_YYLinkedMapNode *)removeTailNode;
      // 移除所有缓存
      - (void)removeAll;
      @end
      ```
   
      

>  ps未完待续……