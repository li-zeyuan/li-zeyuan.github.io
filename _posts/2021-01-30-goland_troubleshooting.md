---
title: Goland故障排查
date: 2021-01-30 00:00:00 +0800
categories: [root, goland]
tags: [goland,troubleshooting]
author: ahern
---

## 解决无法识别导入包
#### 解决
1. 检查是否开启go mod
2. 若开启了go mod，IDE的setting是否配置正确，需要删除gopath
3. go mod tidy
4. 重新go build即可

## 解决调整不正常
#### 解决
1. 可尝试删除.idea文件
2. 删除IDE缓存
3. 也可参考https://stackoverflow.com/questions/37282285/intellij-cannot-find-any-declarations

  ![IDE](https://raw.githubusercontent.com/li-zeyuan/access/master/img/20210130112938.png){:height="10%" width="50%"}

