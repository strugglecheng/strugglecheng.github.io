---
layout: post
title: Keystone的Token服务
date: 2018-10-10
tag: openstack
---

## 为什么要用Token
&emsp;&emsp;在Web交互中，如果用户每次都携带用户名密码请求Web服务，很容易泄露用户信息（例如HTTP Basic Authorization，请求头信息包含了base64编码的用户名和密码信息，很容易被截获解码出来），为了安全性，比较推荐的一种验证方式是使用Token。
Token是用户的一种身份凭证，第一次请求时需要使用相关的用户名密码等信息获取Token，后续的请求就只需在请求头携带此Token即可，Token有超时机制，且一般使用加密算法加密，需要私钥才能解开。Token被攻击截获了，也无法知晓用户的信息，很大程度上提高了安全性。
    
## 通过Token访问其他OpenStack组件的流程    
1. 用户先携带用户名密码申请Token，(V3版本中可以显示使用unscoped的验证方式)，得到临时Token（V2版本的Keystone返回的Token的id在响应体中，用'id'标识，V3版本的Keystone返回的Token的id在响应头上，用X-Subject-Token标识），下次访问其他服务时可以在请求头上带上"X-Auth-Token: xxxxx"的方式访问。V3版本请求Token时有多中限定范围的请求方式。

&emsp;&emsp;安全包含两部分：Authentication（认证）和 Authorization（鉴权），Authentication 解决的是“你是谁？”的问题，Authorization 解决的是“你能干什么？”的问题。
keystone 是借助 Role 来实现 Authorization 的。Role 本质就是一堆ACL的集合，用于划分权限，可以通过给User指定Role，使User获得Role对应的操作权限。    
&emsp;&emsp;keystone返回给User的Token包含了Role列表，被访问的Services会判断访问它的User和User提供的Token中所包含的Role, 及每个role访问资源或者进行操作的权限。
系统默认使用管理Role admin和成员Role _member_ ,也可以新增新的Role，需要修改先关配置文件使之生效。user验证时必须带有Project(Tenant)。    

------------------------------------------------------------------
参考及部分引用文档：    
<a href="https://www.cnblogs.com/CloudMan6/p/5365474.html" target="_blank">理解 Keystone 核心概念 - 每天5分钟玩转 OpenStack（18）</a>    
<a href="https://www.cnblogs.com/CloudMan6/p/5365474.html" target="_blank">openstack装B之路-------keystone</a>    
<a href="http://www.cnblogs.com/resn/p/5870264.html" target="_blank">Openstack入坑指南</a>    
