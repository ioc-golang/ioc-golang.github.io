---
title: "环境安装"
linkTitle: "环境安装"
weight: 1
---

### 安装 Go 环境

[Go 下载地址](https://golang.google.cn/)

建议 go version >= 1.17

将 $GOPATH/bin 加入环境变量

### 安装 iocli 代码生成工具至 $GOPATH/bin

```shell
go install github.com/alibaba/ioc-golang/iocli@latest
```