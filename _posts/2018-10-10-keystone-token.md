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
1. 用户先携带用户名密码申请Token，(V3版本中可以显示使用unscoped的验证方式)，得到临时Token（V2版本的Keystone返回的Token的id在响应体中，用'id'标识，V3版本的Keystone返回的Token的id在响应头上，用X-Subject-Token标识），下次访问其他服务时可以在请求头上带上"X-Auth-Token: xxxxx"的方式访问。V3版本请求Token时有多种限定范围的请求方式。
2. keystone的V3之前的版本，获取Token的时候，同时会返回一系列的endpoint，这些endpoint有简单的项目说明，用户选择相应的endpoint，使用endpoint中的publicURL对相关项目进行访问即可，如下表所示：

```
{
    "access": {
        "token": {
            "issued_at": "2019-02-19T11:22:05.000000Z",
            "expires": "2019-02-19T12:22:04.000000Z",
            "id": "gAAAAABca-bdx5ddGWxryIg9vzNmztCJ0GqfzE_140z2wYmekpnnBn9WOig-aKqpwbkE3AWfGAJNs2uOzRvTZOygJqJqdYp1GLTLj7Yg5KuDHMOKHLvoV3_ENbzMeuathKN9610v-LOxEQUCHcycxZ5jHSpGpZLPqPO8uXFSddFZu3AnQsPQQqg",
            "tenant": {
                "description": "admin tenant",
                "enabled": true,
                "id": "27a0ecb9058b447c9a5af2513bb8bf99",
                "name": "admin"
            },
            "audit_ids": [
                "2rzwPTEISbiVzpVWUyF_2A"
            ]
        },
        "serviceCatalog": [
            {
                "endpoints": [
                    {
                        "adminURL": "http://192.168.204.2:8778",
                        "region": "RegionOne",
                        "internalURL": "http://192.168.204.2:8778",
                        "id": "4ec5c67c7854422c8ab5a5f71151db34",
                        "publicURL": "http://10.190.62.120:8778"
                    }
                ],
                "endpoints_links": [],
                "type": "placement",
                "name": "placement"
            },
            {
                "endpoints": [
                    {
                        "adminURL": "http://192.168.204.2:9696",
                        "region": "RegionOne",
                        "internalURL": "http://192.168.204.2:9696",
                        "id": "24f14bba0dbd4ce4893f0059139f256b",
                        "publicURL": "http://10.190.62.120:9696"
                    }
                ],
                "endpoints_links": [],
                "type": "network",
                "name": "neutron"
            },
            {
                "endpoints": [
                    {
                        "adminURL": "http://192.168.205.4:9292",
                        "region": "RegionOne",
                        "internalURL": "http://192.168.205.4:9292",
                        "id": "6be7bc45e1d74fd28ce7b2ebe97dddb1",
                        "publicURL": "http://10.190.62.120:9292"
                    }
                ],
                "endpoints_links": [],
                "type": "image",
                "name": "glance"
            },
            {
                "endpoints": [
                    {
                        "adminURL": "http://192.168.204.2:8777",
                        "region": "RegionOne",
                        "internalURL": "http://192.168.204.2:8777",
                        "id": "cc3ebdb78a7a41cc84c7f8cb44f6d03d",
                        "publicURL": "http://10.190.62.120:8777"
                    }
                ],
                "endpoints_links": [],
                "type": "metering",
                "name": "ceilometer"
            },
            .
            .
            .
            .
            .
            .
        ],
        "user": {
            "username": "admin",
            "roles_links": [],
            "id": "412a4f332a864e1c86faccbc55a9c165",
            "roles": [
                {
                    "name": "admin"
                },
                {
                    "name": "_member_"
                },
                {
                    "name": "heat_stack_owner"
                }
            ],
            "name": "admin"
        },
        "metadata": {
            "is_admin": 0,
            "roles": [
                "2283ac71045947c2bfbf29d0b2733c8b",
                "9fe2ff9ee4384b1894a90878d3e92bab",
                "35be0a21cf61491e83544a88b995d1c4"
            ]
        }
    }
}
```
3. V3版本的keystone可以先请求相应的临时token，再使用该临时token去请求指定scope的token，V3版本的Keystone返回的Token的id在响应头上，用X-Subject-Token标识。
  
## Token的四种生成方式
- UUID Token类型
UUID token 是随机字符串，不包含任何内容，所以不支持本地校验

- PKI Token类型
PKI token 使用非对称加密方式，携带更多用户信息的同时还附上了数字签名，以支持本地认证

- PKIZ Token类型
PKIZ 在 PKI 的基础上做了压缩处理，但是压缩的效果极其有限，一般情况下，压缩后的大小为 PKI token 的 90 % 左右，所以 PKIZ 不能友好的解决 token size 太大问题。

- Fernet Token类型
社区提出了 Fernet token ，它采用 cryptography 对称加密库(symmetric cryptography，加密密钥和解密密钥相同) 加密 token，具体由 AES-CBC 加密和散列函数 SHA256 签名。 Fernet 是专为 API token 设计的一种轻量级安全消息格式，不需要存储于数据库，减少了磁盘的 IO，带来了一定的 性能提升 。为了提高安全性，需要采用 Key Rotation 更换密钥。

可见每种方式都有优缺点，如果希望少去和Keystone交互，减少认证请求带宽，可以采用PKI、PKIZ、Fernet的方式，而这几种方式的逻辑各有差异，涉及到加密方法的安全性，CA的权威性。UUID模式是最简单的模式，Token中不包含用户信息，只是简单的一串UUID，所以验证身份时需要频繁地请求Auth组件，在管理多个微服务的安全验证时会有性能方面的隐患。PKI方式的Token采用非对称加密，PKIZ类型与PKI类型相似，只是加了压缩，这两种模式的Token安全性非常高，但是计算速度相对慢一点。VNFM安全组件AUTH组件选择的是Fernet token，Fernet是专为API token 设计的一种轻量级安全消息格式，采用对称加密算法，计算只比UUID的token慢，但快于其他两种Token，且不需要存储于数据库，减少了磁盘的IO，避免随着时间的增加，数据库数据变得庞大查找耗时导致验证速度大幅下降的尴尬。

------------------------------------------------------------------
参考及部分引用文档：     
<a href="https://docs.openstack.org" target="_blank">OpenStack官方文档</a>
<a href="https://baijiahao.baidu.com/s?id=1574064596917586&wfr=spider&for=pc" target="_blank">理解Keystone的四种Token</a>
 
