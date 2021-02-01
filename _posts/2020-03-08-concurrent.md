---
layout : post
title : "并发"
category : 并发
duoshuo: true
date : 2020-03-08
tags : [并发]
---
1. ConcurrentLinkedQueue : 
    - 1.1 数据结构 ： 单项链表
    - 1.2 初始化 ： header -> tail -> null
    - 1.3 offer 操作
        - 1.3.1 判断tail的下个节点是否空， 如果是空就往尾部插入当前节点
        - 1.3.2 如果不为空，说明有其他线程操作，则重新获取获取尾部节点 然后重新操作1.3.1
