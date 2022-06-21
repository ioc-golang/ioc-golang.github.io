---
title: "配置文件结构规范"
linkTitle: "配置文件结构规范"
date: 2017-01-05
weight: 3
---

### 默认配置层级

配置文件 ioc_golang.yaml 的默认参数默认配置层级依此为：

```
autowire:
  $(自动装载模型名):
    $(SDID):
      $(多例模式下的对象名):
        param:
      param:
```



例如，一个基于多例模型的结构 Impl，实现了 Redis 接口，其多个实例参数可以配置为：

```yaml
autowire:
  normal:
    github.com/alibaba/ioc-golang/extension/normal/redis.Impl:
        db1-redis:
          param:
            address: localhost:6379
            db: 1
        db2-redis:
          param:
            address: localhost:6379
            db: 2
        param:
          address: localhost:6379
          db: 0
```

拥有标签 `normal:"Impl,db2-redis"` 的字段，会从配置中读入参数，位置为标签指向的对象名db2-redis

```go
NormalDB2Redis normalRedis.Redis `normal:"Impl,db2-redis"`

address: localhost:6379
db: 2
```

拥有标签 `normal:"Impl"` 的字段，会从配置中读入参数。

```go
NormalDBRedis normalRedis.Redis `normal:"Impl"`
address: localhost:6379
db: 0
```

### 定制化参数加载模型

参数加载模型可以由自动装载模型和 SD （结构描述符）定制化，例如一个基于 grpc 自动装载模型注入的客户端

```go
HelloServiceClient api.HelloServiceClient `grpc:"hello-service"`
```

从如下位置读取参数：

```yaml
autowire:
  grpc:
    hello-service:
      address: localhost:8080
```



