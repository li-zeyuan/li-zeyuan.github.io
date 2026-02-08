---
title: Go Module包管理
date: 2020-04-01 00:00:00 +0800
categories: [language, golang]
tags: [golang]
author: ahern
---

## 包管理简史

![8e57f2d4beac40d115496a787e2f6015](https://raw.githubusercontent.com/li-zeyuan/access/master/img/8e57f2d4beac40d115496a787e2f6015.png){:height="10%" width="50%"}

## GOPATH

GOROOT：调用官方包，会从`$GOROOT/src`目录下寻找该包，一个栗子：

- ![78a4f7a893b514a71a4db89f347fca42](https://raw.githubusercontent.com/li-zeyuan/access/master/img/78a4f7a893b514a71a4db89f347fca42.png){:height="10%" width="50%"}

GOPATH：一个项目一个GOPATH，go get第三方包时，会保存在`$GOPATH/src`目录下寻找包，一个栗子： 

- ![d4674b2200ed9b133e0df1a4fd758c22](https://raw.githubusercontent.com/li-zeyuan/access/master/img/d4674b2200ed9b133e0df1a4fd758c22.png){:height="10%" width="50%"}

缺点： 

- 没有依赖列表，只能一个一个go get
- 依赖代码根项目的代码混到`$GOPATH/src`

## vendor机制

- 优先从vendor目录中寻找包
- 再从$GOPATH/src/ 寻找

优点:

- 依赖放到vendor目录中管理，解决GOPATH机制下项目代码和依赖混淆问题
- go build 或 go run 时；不需要重新go get

缺点:

- 还是依赖$GOPATH
- 当你想升级依赖包的时候，就只能手动升级了

## Module

官方指定包管理工具,依赖包存放在$GOPATH/pkg/mod

#### go.mod

![debc22975db9378cbcab18c831b9c994](https://raw.githubusercontent.com/li-zeyuan/access/master/img/debc22975db9378cbcab18c831b9c994.png){:height="10%" width="50%"}

- module path：第一行；项目中import的包以module path开头，到此go.mod所在的目录查找包
- go directive：第二行；指定go的最低版本
- require：依赖列表
  - 版本规范依赖：`github.com/360EntSecGroup-Skylar/excelize/v2 v2.3.1`
  - 伪版本号(pseudo-version)：`github.com/bradfitz/gomemcache v0.0.0-20190913173617-a41fca850d0b`
  - incompatible；不兼容的意思。go规范中主版本>=2时，模块路径以/vn结尾（如：github.com/foo/bar/v2）。若未遵循该规范，go mod将出现incompatible，如：`github.com/dgrijalva/jwt-go v3.2.0+incompatible`
- indirect：间接依赖
- exclude：go get时这些版本不在考虑范围
- replace：重定向包路径

#### go.sum

格式：

```
<module> <version> <hash>
<module> <version>/go.mod <hash>

github.com/HdrHistogram/hdrhistogram-go v1.1.2 h1:5IcZpTvzydCQeHzK4Ef/D5rrSqwxob0t8PQPMybUNFM=
github.com/HdrHistogram/hdrhistogram-go v1.1.2/go.mod h1:yDgFjdqOqDEKOvasDdhWNXYg9BVp4O+o5f6V/ehm6Oo=
```

作用：

- 提供分布式环境下的包管理依赖内容校验；go没有类似pip这样的中心仓库，而是采用分布式包管理，github上的发布包，发布者可以修改包打上相关的tag。所以就需要一个checksum来防篡改
- 作为 transparent log 来加强安全性；go.sum是一个Append Only 的日志记录；可追溯，提高篡改者的作案成本。

缺点：

- 容易产生合并冲突
- 没有从根本上解决防篡改问题；如果发生篡改，会构建失败，更多是起到提示作用

#### 原理

- `go get`默认使用[semantic version](https://semver.org/)算法
- 如本地存在B1.1，那么`go get` 不会更新到B 1.2 

[Minimal Version Selection](https://research.swtch.com/vgo-mvs)

- `go get -u`或`go get @xx`采用此算法
- 最小的修改操作
- 最小的需求列表
- 最小的模块版本

场景1：[Construct Build List](https://research.swtch.com/vgo-mvs#algorithm_1)

- 首先构建

- 依赖图
  
  ![54bf2b25073e0e2574d18b0deca85349](https://raw.githubusercontent.com/li-zeyuan/access/master/img/54bf2b25073e0e2574d18b0deca85349.png){:height="10%" width="50%"}

- 依赖结果
  
  ![85702c49a0294e40a3bf8170e3b43c34](https://raw.githubusercontent.com/li-zeyuan/access/master/img/85702c49a0294e40a3bf8170e3b43c34.png){:height="10%" width="50%"}

场景2:[Upgrade All Modules](https://research.swtch.com/vgo-mvs#algorithm_2)

- `go get -u`

- 依赖过程
  
  ![68401c38feb9b54a8acfe90941cd98e8](https://raw.githubusercontent.com/li-zeyuan/access/master/img/68401c38feb9b54a8acfe90941cd98e8.png){:height="10%" width="50%"}

- 将所有的依赖升级到最新（直接&&间接依赖）

场景3:[Upgrade One Module](https://research.swtch.com/vgo-mvs#algorithm_3)

- `go get C@1.3`

- 依赖过程
  
  ![6168c236fcfddc5ad40f13c9a2c921b8](https://raw.githubusercontent.com/li-zeyuan/access/master/img/6168c236fcfddc5ad40f13c9a2c921b8.png){:height="10%" width="50%"}

- 仅将C的依赖升级到最新（C的直接&&间接依赖）

场景4:[Downgrade One Module](https://research.swtch.com/vgo-mvs#algorithm_4)

- `go get D@1.2`

- 依赖过程 
  
  ![b3b9ca519e770991cde9b96a2f8d133a](https://raw.githubusercontent.com/li-zeyuan/access/master/img/b3b9ca519e770991cde9b96a2f8d133a.png){:height="10%" width="50%"}

- 这里将D降级为D1.2，会先删除D1.3以及D1.4模块，然后回溯删除B1.2以及C1.2模块，最终确定到B1.1以及C1.1版本(它们分别是B和C不依赖>=D1.3模块的最新版本了)

#### 优点

- 不依赖$GOPATH，项目不需要配置GOPATH
- 排除使用vendor，项目代码的体积大大减小
- go.mod记录依赖树版本，直观，方便修改

#### 缺点

- go.sum的校验checksum仍然存在风险，也是分布式包管理的通病

## 总结

- go 依赖管理的三个阶段：GOPATH->Vendor机制->Go Module
- go Module中两个重要的文件：go.mod、go.sum
- go get采用的两种算法：semantic version、minimal version seletion
- go Module、inkedep优缺点

## 参考

- go mod blog:https://blog.golang.org/using-go-modules
- Go 包管理工具 dep 安装与使用:https://learnku.com/articles/31474
- go mod:https://colobu.com/2021/06/28/dive-into-go-module-1/
- 浅谈Go Modules原理：https://duyanghao.github.io/golang-module/
