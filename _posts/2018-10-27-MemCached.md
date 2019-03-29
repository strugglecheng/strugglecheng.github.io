---
layout: post
title: MemCached学习笔记
date: 2018-10-27
tag: 编程语言
---

## MemCached简介
百度百科上对Memcached的介绍：    
> Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储键/值对的hashmap。其守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。这是一套开放源代码软件，以BSD license授权协议发布。memcached缺乏认证以及安全管制，这代表应该将memcached服务器放置在防火墙后。
memcached的API使用32位元的循环冗余校验（CRC-32）计算键值后，将资料分散在不同的机器上。当表格满了以后，接下来新增的资料会以LRU机制替换掉。由于memcached通常只是当作快取系统使用，所以使用memcached的应用程式在写回较慢的系统时（像是后端的数据库）需要额外的程式码更新memcached内的资料。MemCached运行时会在内存中开辟一块空间，建立一个统一的巨大的hash表，hash表能够用来存储各种格式的数据，包括图像、视频、文件以及数据库检索的结果等，由于是基于内存管理数据，所以存储速度非常快。

## MemCached内存模型




------------------------------------------------------------------
参考及部分引用文档：    
<a href="https://segmentfault.com/a/1190000012950110" target="_blank">MemCache 基础介绍与工作原理</a>    
<a href="http://ju.outofmemory.cn/entry/105408" target="_blank">Memcached经验分享</a>    
<a href="https://blog.csdn.net/ywh147/article/details/47837955" target="_blank">memcached面试题集锦</a>    

