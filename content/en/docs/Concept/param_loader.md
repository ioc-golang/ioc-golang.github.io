---
title: "参数加载器"
date: 2017-01-05
weight: 4
---

### 概念

参数加载器描述了依赖参数如何在对象构造之前被加载，包括但不限于从配置加载、从[标签](/cn/docs/reference/tag_format)参数加载等等。

参数加载器作为 [SD(结构描述符)](/cn/docs/concept/sd)的一部分，可以被结构提供方定制化，也可以使用自动装载模型提供的默认参数加载器。

### 默认参数加载器

任何 SD 内定义参数加载器均被优先执行，如果加载失败，则尝试使用默认参数加载器加载。

默认参数加载器被两个基础自动装载模型（singleton、normal）引入。依此采用三种方式加载参数，如果三种方式均加载失败，则抛出错误。

- 方式1 从标签指向的对象名加载参数，参考  [配置文件结构规范](/cn/docs/reference/yaml_structure)

  ````
  Load support load struct described like:
  ```go
  normal.RegisterStructDescriptor(&autowire.StructDescriptor{
  		Factory:   func() interface{}{
  			return &Impl{}
  		},
  		ParamFactory: func() interface{}{
  			return &Config{}
  		},
  		ConstructFunc: func(i interface{}, p interface{}) (interface{}, error) {
  			return i, nil
  		},
  	})
  }
  
  type Config struct {
  	Address  string
  	Password string
  	DB       string
  }
  
  ```
  with
  Autowire type 'normal'
  StructName 'Impl'
  Field:
  	MyRedis Redis `normal:"github.com/alibaba/ioc-golang/extension/normal/redis.Impl, redis-1"`
  
  from:
  
  ```yaml
  extension:
    normal:
      github.com/alibaba/ioc-golang/extension/normal/redis.Impl:
          redis-1:
            param:
              address: 127.0.0.1
              password: xxx
              db: 0
  ````

- 方式2 从标签加载参数

  ````
  Load support load param like:
  ```go
  type Config struct {
  	Address  string
  	Password string
  	DB       string
  }
  ```
  
  from field:
  
  ```go
  NormalRedis  normalRedis.Redis  `normal:"github.com/alibaba/ioc-golang/extension/normal/redis.Impl,address=127.0.0.1&password=xxx&db=0"`
  ```
  ````

- 方式3 从配置加载参数，参考 [配置文件结构规范](/cn/docs/reference/yaml_structure)

  ````
  Load support load struct described like:
  ```go
  normal.RegisterStructDescriptor(&autowire.StructDescriptor{
  		Factory:   func() interface{}{
  			return &Impl{}
  		},
  		ParamFactory: func() interface{}{
  			return &Config{}
  		},
  		ConstructFunc: func(i interface{}, p interface{}) (interface{}, error) {
  			return i, nil
  		},
  	})
  }
  
  type Config struct {
  	Address  string
  	Password string
  	DB       string
  }
  ```
  with
  Autowire type 'normal'
  StructName 'Impl'
  
  from:
  
  ```yaml
  autowire:
    normal:
        github.com/alibaba/ioc-golang/extension/normal/redis.Impl:
          param:
            address: 127.0.0.1
            password: xxx
            db: 0
  ````

  



