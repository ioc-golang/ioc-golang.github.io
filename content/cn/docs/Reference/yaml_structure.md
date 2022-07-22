---
title: "配置文件格式规范"
linkTitle: "配置文件格式规范"
date: 2017-01-05
weight: 3
---

## 默认配置格式与参数加载位置

配置文件 ioc_golang.yaml 的默认参数默认配置格式为：

```
autowire:
  ${自动装载模型名}:
    ${SDID}:
      ${多例模式下的索引key1}:
        param:
          field1: value1 # 多例模式创建对象的构造参数
      ${多例模式下的索引key2}:
        param:
          field1: value2 # 多例模式创建对象的构造参数
      param:
        field1: value3 # 单例模式或多例模式未指定索引key 创建对象的构造参数
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

拥有 [注入标签](../tag_format) `normal:",db2-redis"` 的字段，会从配置中读入参数，位置为标签指向的对象名db2-redis，db 为2。

```go
NormalDB2Redis normalRedis.RedisIOCInterface `normal:",db2-redis"`

address: localhost:6379
db: 2
```

拥有标签 `normal:""` 的字段，会从配置中读入参数，db 为 0。

```go
NormalDBRedis normalRedis.RedisIOCInterface `normal:""`

address: localhost:6379
db: 0
```

## 定制化参数加载器

[参数加载器(param_loader) ](/docs/concept/ioc/param_loader/)可以在自动装载模型或 [SD （结构描述符）](/docs/concept/ioc/sd/) 中定制化，例如一个基于 grpc 自动装载模型注入的客户端，我们已经在 grpc 自动装载模型中 [定制化](https://github.com/alibaba/IOC-golang/blob/master/extension/autowire/grpc/param_loader.go#L29) 了参数加载位置，为 `autowire.grpc.${tag-value}` ，因此对于如下字段：

```go
HelloServiceClient api.HelloServiceClient `grpc:"hello-service"`
```

会从如下位置读取参数：address 设置为  "localhost:8080"

```yaml
autowire:
  grpc:
    hello-service:
      address: localhost:8080
```

同理，也可以由服务提供者在 SD 中传入参数加载器，进行定制化参数加载源。

## 配置文件中的特殊引用

### 声明环境变量

在配置文件中，可以使用 `${ENV_KEY}` 的格式，引入环境变量。当环境变量存在时，将会被替换为环境变量的值。

```yaml
config:
  app:
    config-value: myValue
    config-value-from-env: ${MY_CONFIG_ENV_KEY} # 如环境变量存在，读取环境变量替换该字段
```

### 声明配置引用（nested）

```yaml
config:
  app:
    config-value: myValue
    config-value-from-env: ${MY_CONFIG_ENV_KEY}  # 如环境变量存在，读取环境变量替换该字段
    nested-config-value: ${config.app.config-value} # 引用 config.app.config-value 的值，为 myValue
    nested-config-value-from-env: ${config.app.config-value-from-env} # 引用 config.app.config-value-from-env 的值，为环境变量替换后的值
```

更多配置文件的加载与使用例子，参考 [example/config_file](https://github.com/alibaba/IOC-golang/tree/master/example/config_file)



