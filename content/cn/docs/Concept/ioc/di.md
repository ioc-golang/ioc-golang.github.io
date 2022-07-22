---
title: "依赖注入"
date: 2017-01-05
weight: 1
---

### 概念

**依赖注入**(Dependency Injection，DI) 是指开发人员无需显式地创建对象，而是通过特定标注（在本框架中为 [标签](/docs/reference/tag_format) ）的方式声明字段，由框架在对象加载阶段，将实例化对象注入该字段，以供后续使用。

### 优点

依赖注入可以降低代码的耦合度，减少冗余代码量，优化代码逻辑，使得开发人员可以只关心业务逻辑，帮助面向对象编程，面向接口编程。

### IOC-golang 的依赖注入能力

本框架从两个角度实现依赖注入：

- 结构如何被提供

  开发人员需要准备好需要被注入的结构，将它注册到框架。注册到框架的代码可以由开发人员手动编写，开发人员也可以通过使用[注解](../annotation)标记结构，使用 [iocli](http://localhost:1313/cn/docs/reference/iocli/#结构注解与sdcndocsconceptsd代码生成) 的代码生成能力自动生成注册代码，从而减少工作量。

  一个带有 [注解](../annotation) 的结构

  ```go
  // +ioc:autowire=true
  // +ioc:autowire:type=singleton
  
  type App struct {}
  ```

  由 iocli 工具生成，或用户手动编写的注册代码。摘自[ example/helloworld/zz_generated.ioc.go#L21](https://github.com/alibaba/IOC-golang/blob/master/example/helloworld/zz_generated.ioc.go#L21)

  ```go
  func init() {
  	singleton.RegisterStructDescriptor(&autowire.StructDescriptor{
  		Factory: func() interface{} {
  			return &App{}
  		},
  	})
  }
  ```
  
- 结构如何被使用

  - 通过标签注入

    开发人员需要通过标签来标记需要注入的结构字段。

    需要通过标签注入依赖至字段的结构，其本身必须也注册在框架上。

    ```go
    // +ioc:autowire=true
    // +ioc:autowire:type=singleton
    
    type App struct {
    	MySubService Service `singleton:"main.ServiceImpl1"` 
    }
    ```

    

  - 通过 API 可获取对象，入参为 [结构 ID (结构描述 ID，SDID)](/docs/concept/ioc/sd/#%E7%BB%93%E6%9E%84%E6%8F%8F%E8%BF%B0id) 和构造参数(如需要)。

    通过 API 获取对象的过程会默认通过代码生成。参考 [example/helloworld/zz_generated.ioc.go#L190](https://github.com/alibaba/IOC-golang/blob/master/example/helloworld/zz_generated.ioc.go#L190)
    
	  ```go
    appInterface, err := singleton.GetImpl("main.App")
	  if err != nil {
	    panic(err)
    }
    app := appInterface.(*App)
    
    redisInterface, err := normal.GetImpl("github.com/alibaba/ioc-golang/extension/normal/redis.Impl", &Config{
	      Address: "localhost:6379"
    })
	  if err != nil {
	    panic(err)
    }
    redisClient := redisInterface.(*Impl)
    ```
    
    
