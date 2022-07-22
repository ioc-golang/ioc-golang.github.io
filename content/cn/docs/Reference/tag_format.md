---
title: "注入标签格式规范"
linkTitle: "注入标签格式规范"
date: 2017-01-05
weight: 2
---

IOC-golang 依赖 Go 原生支持的字段标签来作为依赖注入标签，标签的格式定义如下

拥有注入标签的结构，必须要在 ioc 框架的管理下才能正确注入，因此需要在结构体上方增加注解 ` +ioc:autowire ...`=

```go
// +ioc:autowire=true
// +ioc:autowire:type=singleton

type DemoStruct struct{
  ${大写字母开头的字段名} ${字段类型} `${自动装载模型}:"${结构ID},${结构key}或${参数字段1}=${参数值1}&..."`
}
```

### ${字段类型}

其中`${字段类型}` 可有如下选择

- 接口

  - [结构专属接口](/docs/concept/aop/proxy/#3-结构专属接口)

    结构专属接口已经包含了结构ID信息，其标签内 `${结构ID}` 为空即可。

  - 开发者提供的接口

    ---
    
    由于结构专属接口与结构体实现处在同一 pkg 下，如果期望通过依赖注入进行接口-实现分离，则需要由开发者给出接口，同时必须给出期望注入的`${结构ID}` 。并保证程序在任何位置引入了实现类pkg，保证在程序启动时，期望注入的结构可被注册，从而被框架所管理，完成注入逻辑。
    
    ---

- 结构体指针。

  结构体指针已经包含了结构ID信息，其标签内 `${结构ID}` 为空即可。

### ${自动装载模型}

内置：autowire, normal, grpc, rpc-client, config

如开发者提供了 [扩展的自动装载模型](/docs/developer/develop_autowire/)，也可以在这里指定。

### ${结构ID}

结构ID字符串，参考 [结构ID](/docs/concept/ioc/sd/#结构id)。

如注入字段的 `${字段类型}` 为结构体指针或者 [结构专属接口](/docs/concept/aop/proxy/#3-结构专属接口)，`${结构ID}` 为空即可。

### ${结构key}或${参数字段1}=${参数值1}&...

指定构造参数的值

- 如果为多例模型，在此位置指定结构key，参考 [默认配置格式与参数加载位置](/docs/reference/yaml_structure/#默认配置格式与参数加载位置) 中的例子
- 如果在此位置给出 `${参数字段1}=${参数值1}&${参数字段2}=${参数值2}` 则会以此处的参数值构造依赖对象并注入。参数值可以从配置文件中读取，也可以从环境变量中读取，一个复杂的例子如下：

参考 [example/third_party/state/redis](https://github.com/alibaba/IOC-golang/blob/master/example/third_party/state/redis/cmd/main.go#L37)

```go

// +ioc:autowire=true
// +ioc:autowire:type=singleton
// +ioc:autowire:paramType=Param
// +ioc:autowire:constructFunc=Init
// +ioc:autowire:alias=AppAlias

type App struct {
	NormalRedis    normalRedis.RedisIOCInterface `normal:""` // 配置文件默认位置加载参数
	NormalDB1Redis normalRedis.RedisIOCInterface `normal:",db1-redis"` // 配置文件当前结构的索引 db1-redis 位置加载参数
	NormalDB2Redis normalRedis.RedisIOCInterface `normal:",db2-redis"` // 配置文件当前结构的索引 db2-redis 位置加载参数
	NormalDB3Redis normalRedis.RedisIOCInterface `normal:",address=127.0.0.1:6379&db=3"` // 直接给出静态参数
	NormalDB4Redis normalRedis.RedisIOCInterface `normal:",address=${REDIS_ADDRESS_EXPAND}&db=5"`  // 从环境变量中读取参数
	NormalDB5Redis normalRedis.RedisIOCInterface `normal:",address=${autowire.normal.<github.com/alibaba/ioc-golang/extension/state/redis.Redis>.nested.address}&db=15"`  // 从配置文件的给定位置中读取参数

	privateClient *redis.Client
}
```

其配置文件如下：

```yaml
autowire:
  normal:
    github.com/alibaba/ioc-golang/extension/state/redis.Redis:
      db1-redis: 
        param: # NormalDB1Redis 读取参数位置
          address: localhost:6379
          db: 1
      db2-redis: # NormalDB2Redis 读取参数位置
        param:
          address: localhost:6379
          db: 2
      param: # NormalRedis 读取参数位置
        address: localhost:6379
        db: 0
      expand:
        address: ${REDIS_ADDRESS_EXPAND}
        db: 15
      nested: # NormalDB5Redis 读取参数位置
        address: ${autowire.normal.<github.com/alibaba/ioc-golang/extension/state/redis.Redis>.expand.address}
        db: ${autowire.normal.<github.com/alibaba/ioc-golang/extension/state/redis.Redis>.expand.db}
```





