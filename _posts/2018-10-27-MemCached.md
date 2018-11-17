---
layout: post
title: MemCached学习笔记
date: 2018-10-27
tag: 编程语言
---

## Cache简介
- CPU缓存    
CPU的运算速度越来越快，它与存储介质（硬盘、内存）之间的差距越来越大，当CPU需要读写数据时，需要等待内存和硬盘返回或者写入数据，这样CPU的运算优势就远不能发挥出来。
CPU缓存的出现是为了解决CPU和内存读取速度不匹配的矛盾，CPU缓存学名叫“高速缓冲存储器”，它们紧紧靠近CPU，根据存储速度分为不同的等级，CPU将最近的运算结果缓存在这些硬件中，从而提高性能。

- Web缓存    
Web缓存的的出现是为了减轻Web网站数据库的负载，将热点数据存储在内存中，很大程度上提高了网站访问的速度。

## MemCached简介
> MemCached是一个自由、源码开放、高性能、分布式的分布式内存对象缓存系统，用于动态Web应用以减轻数据库的负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高了网站访问的速度。
MemCaChed是一个存储键值对的HashMap，在内存中对任意的数据（比如字符串、对象等）所使用的key-value存储，数据可以来自数据库调用、API调用，或者页面渲染的结果。MemCached设计理念就是小而强大，
它简单的设计促进了快速部署、易于开发并解决面对大规模的数据缓存的许多难题，而所开放的API使得MemCached能用于Java、C/C++/C#、Perl、Python、PHP、Ruby等大部分流行的程序语言。
> MemCache早在2004年2月就出现了，而MemCached继承了memcache优秀的一面，在这之上又增加了许多nice的api接口，批量操作和setoption非常nice，setoption增加了许多可以定义的配置，比如更优秀的压缩算法，更多的hash算法，socket链接优化，FAILURE，宕机处理方案等等。Memcached相比Memcache更优秀，但要使用它，必须基于libmemcached来编译，libmemcached的优点很多，memcached扩展支持了libmemcached的很多重要特性，比如更低的内存消耗、线程安全、更多散列算法等等，总之它更优秀。
MemCached守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。
memcached的API使用32位元的循环冗余校验（CRC-32）计算键值后，将资料分散在不同的机器上。当表格满了以后，接下来新增的资料会以LRU机制替换掉。由于memcached通常只是当作快取系统使用，所以使用memcached的应用程式在写回较慢的系统时（像是后端的数据库）需要额外的程式码更新memcached内的资料。MemCached运行时会在内存中开辟一块空间，建立一个统一的巨大的hash表，hash表能够用来存储各种格式的数据，包括图像、视频、文件以及数据库检索的结果等。


## MemCached使用场景
> 通常，我们会在访问量高的Web网站和应用中使用MemCached，用来缓解数据库的压力，并且提升网站和应用的响应速度。使用缓存时一般有一下特征：
- 访问频繁的数据库数据（身份token、首页动态）
- 访问频繁的查询条件和结果
- 作为Session的存储方式（提升Session存取性能）
- 页面缓存
- 更新频繁的非重要数据（访客量、点击次数）
- 大量的hot数据


## MemCached工作原理
> MemCached采用了C/S架构，在Server端启动后，以守护程序的方式，监听客户端的请求。启动时可以指定监听的IP（服务器的内网ip/外网ip）、端口号（所以做分布式测试时，一台服务器上可以启动多个不同端口号的MemCached进程）、使用的内存大小等关键参数。一旦启动，服务就会一直处于可用状态。
为了提高性能，MemCached缓存的数据全部存储在MemCached管理的内存中，所以重启服务器之后缓存数据会清空，不支持持久化。
Memcached的神奇来自两阶段哈希（two-stage hash）。Memcached就像一个巨大的、存储了很多<key,value>对的哈希表。通过key，可以存储或查询任意的数据。
客户端可以把数据存储在多台memcached上。当查询数据时，客户端首先参考节点列表计算出key的哈希值（阶段一哈希），进而选中一个节点；客户端将请求发送给选中的节点，然后memcached节点通过一个内部的哈希算法（阶段二哈希），查找真正的数据（item）。




------------------------------------------------------------------
参考及部分引用文档：    
<a href="https://segmentfault.com/a/1190000012950110" target="_blank">MemCache 基础介绍与工作原理</a>    
<a href="http://ju.outofmemory.cn/entry/105408" target="_blank">Memcached经验分享</a>    
<a href="https://blog.csdn.net/ywh147/article/details/47837955" target="_blank">memcached面试题集锦</a>    

