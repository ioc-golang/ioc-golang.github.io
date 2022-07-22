---
title: "拦截器"
description: >
  跟随 AOP 对象注册的拦截器模型
weight: 3
---

## 概念

**拦截器** 是 AOP 思路的基本切面单元。在 [AOP 实现](../aop_impl) 中提到，本框架提供了针对代理对象函数调用的拦截器模型，和框架原生支持的 RPC 拦截器模型。 

需要注意，在当前版本中，函数拦截器都是**单例**的，即 AOP 实现中的工厂函数只会被调用一次。函数拦截器尚不能保证**调用顺序**。

### 拦截器接口

```go
// 函数拦截器
type Interceptor interface {
	BeforeInvoke(ctx *InvocationContext)
	AfterInvoke(ctx *InvocationContext)
}

// RPC 调用拦截器
type RPCInterceptor interface {
	BeforeClientInvoke(req *http.Request) error
	AfterClientInvoke(rsp *http.Response) error
	BeforeServerInvoke(c *gin.Context) error
	AfterServerInvoke(c *gin.Context) error
}
```

开发者可以按需定制单例模式的拦截器，并通过 AOP 实现注册在框架上，以供使用。

- 函数拦截器

函数拦截器的参数为 `InvocationContext ` 其包含了一次请求的上下文信息：

```go
type InvocationContext struct {
	ProxyServicePtr interface{} // 被调用的代理对象
	SDID            string // 原始对象ID
	MethodName      string // 被调用的方法名
  MethodFullName  string // 被调用方法全名，包含了包名、结构名、方法名，例如 github.com/alibaba/ioc-golang/test.(*App).Run
	Params          []reflect.Value // 请求参数列表
	ReturnValues    []reflect.Value // 返回参数列表
	GrID            int64 // 当前 goroutine ID
}
```

在调用原始对象的函数之前，所有注册在框架等函数拦截器的 `BeforeInvoke` 方法将依此被调用，此时上下文中 `ReturnValues`字段为空 ；在调用原始对象的函数之后，所有注册在框架等函数拦截器的 `BeforeInvoke` 方法将依此被调用。

### 拦截器注册 API

在 init 方法中跟随 AOP 对象注册至框架，例如：

```go
func init() {
	aop.RegisterAOP(aop.AOP{
		Name: "monitor",
		InterceptorFactory: func() aop.Interceptor {
			return getMonitorInterceptorSingleton() // 函数调用拦截器注册
		},
		GRPCServiceRegister: func(server *grpc.Server) {
			monitorPB.RegisterMonitorServiceServer(server, newMonitorService())
		},
	})
}
```



