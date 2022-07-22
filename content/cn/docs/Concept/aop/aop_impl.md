---
title: "AOP 实现"
description: >
  基于接口代理的 AOP 实现
weight: 2
---

## 概念

一个 **AOP 实现** 是基于框架定义的 AOP 结构所创建的实例化对象。其字段包含了一类 AOP 所关注问题的解决方案。

```go
func init() {
  // 注册一个名为 “list” 的 AOP 实现到框架。该实现提供了基于 debug 端口展示所有接口方法的 gRPC 服务。
	aop.RegisterAOP(aop.AOP{
		Name: "list",
		GRPCServiceRegister: func(server *grpc.Server) {
			list.RegisterListServiceServer(server, newListServiceImpl())
		},
	})
}
```

**AOP 实现**，具体来讲是 [代理对象]() 在原始对象的基础上封装的拦截层及其周边能力。框架提供了一些基础的 AOP 代理层实现，例如上述 “list”。开发人员也可以将自定义的 AOP 代理层注册在框架上使用。一个 AOP 实现对象，关注的是一类问题的 AOP 解决方案，例如可视化、事务、链路追踪。

AOP 实现是一个对象，包含一些聚合在一起的概念。在当前版本中，它可以包含 AOP 领域的四个角度的解决方案：

- 函数调用拦截器

  可以拦截针对所有代理对象的请求。

- RPC调用拦截器

  RPC 自动装载模型是框架默认提供的扩展自动装载模型，它提供了框架原生支持的 RPC 能力，可被开发者直接选用。在 RPC 自动装载模型中，会在所有 RPC 过程中调用已注册的 AOP 实现提供的 “RPC调用拦截器”，从而拦截全量框架原生 RPC 请求。

- GRPC 服务注册器

  本框架提供了调试端口 (默认 1999)，被一个基于 gRPC 的 debug 服务监听 。AOP 实现可以将“待采集” 或 “待触发” 的被动逻辑以 gRPC Service 的形式注册在 debug 服务上，从而对外暴露。

  与 gRPC Service  相对应的是客户端，在本框架默认提供的 AOP 实现中，客户端作为指令被注册在了 iocli 工具上，其实现与 AOP 实现处于同一pkg 下。本质上，gRPC 客户端、gRPC Service 、 两个拦截器的实现形成闭环，共同解决当前 AOP 实现所关注的问题。开发者可以选择其中的一个或多个进行实现。

- ConfigLoader

  提供当前 AOP 实现的读取框架配置能力。

**AOP 结构的代码定义**

```go
type AOP struct {
	Name                  string
	InterceptorFactory    interceptorFactory // type interceptorFactory func() Interceptor
	RPCInterceptorFactory rpcInterceptorFactory // type rpcInterceptorFactory func() RPCInterceptor
	GRPCServiceRegister   gRPCServiceRegister // type gRPCServiceRegister func(server *grpc.Server)
	ConfigLoader          func(config *common.Config)
}
```



## “AOP 实现” 的工作原理

AOP 实现的加载和工作流程：

1. 在 init 函数中将 AOP 实现注册在框架上，引入这一 pkg。

   ```go
   func init(){
     aop.RegisterAOP(aop.AOP{
     ...
     })
   }
   
   ```

2. `ioc.Load()` 阶段，加载框架配置，调用 ConfigLoader 将框架配置传递给 AOP 实现。
3. 依赖注入阶段，如首次出现构建代理对象的情况，将调用所有注册在框架的 AOP 实现的 InterceptorFactory，获取到所有单例的函数拦截器。封装入代理对象。InterceptorFactory 只会被调用一次。
4. RPC 调用阶段，如首次出现 RPC 调用到情况，将调用所有注册在框架的 AOP 实现的 RPCInterceptorFactory，获取到所有单例的RPC 调用拦截器，应用在 RPC过程中。

