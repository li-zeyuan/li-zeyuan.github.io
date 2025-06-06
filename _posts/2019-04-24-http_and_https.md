---
title: http与https
date: 2019-04-24 00:00:00 +0800
categories: [root, protocol]
tags: [http]
author: ahern
---

### 浏览器输入url的过程

- 1、DNS域名解析
- 2、建立TCP连接
- 3、发送HTTP请求
- 4、服务器处理、响应请求
- 5、关闭TCP连接
- 6、浏览器渲染

### http

- 无状态，80端口
- 明文传输

### https

https://zhuanlan.zhihu.com/p/43789231

- 443端口
- 是http+SSL/TLS
- 需要CA证书
- 采用对称加密和非对称加密的方式

通讯过程

![](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210321135107.png){:height="10%" width="50%"}

- 客户端请求建立SSL连接

- 服务端返回证书信息（服务端公钥）

- 客户端SSL/TLS校验证书

- 客户端生成会话秘钥（对称秘钥），用公钥加密发送给服务端

- 服务端用私钥解密得到会话秘钥

- 客户端、服务端通讯用会话秘钥加密

客户端如何验证数字证书（服务端公钥）

- 1、客户端预装CA机构公钥
- 2、用CA机构公钥解密数字签名得 S
- 3、对数字证书hash得值 T
- 4、对比T、S值

### https2.0

- 帧:数据通讯的最小单位
- 流：二进制编码流传输，http1.1文本传输
- 多路复用：同一个域名，只占用一个tcp连接
- 服务端主动推
