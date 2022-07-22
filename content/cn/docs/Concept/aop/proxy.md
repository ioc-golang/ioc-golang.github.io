---
title: "代理对象"
description: >
  基于接口的代理对象的使用与设计
weight: 1
---

## 概念

**原始对象** 是通过 [结构描述符](../../ioc/sd) 直接创建的对象。

**代理对象**是对开发者提供的针对原始对象的封装，在 IOC-golang 的设计中，将“封装了 AOP 拦截器至原始结构，并赋值给接口”的对象，定义为“代理对象”。针对代理对象的函数调用在业务逻辑上完全等价于针对原始对象的函数调用，并为原始对象提供针对函数调用的 AOP 能力，这一能力可被应用在监控、可视化、事务等场景。

## 1. 代理对象由结构使用者关心

IOC-golang 在依赖注入的开发过程中存在两个视角，结构提供者和结构使用者。框架接受来自结构提供者定义的结构，并按照结构使用者的要求把结构提供出来。

结构提供者只需关注结构本体，无需关注结构实现了哪些接口。结构使用者需要关心结构的注入和使用方式，例如 [通过 API 获取](../../../examples/di/api/)，或者通过 [标签](../../../reference/tag_format/) 注入。如通过标签注入，是注入至接口，或是注入至结构体指针。

## 2. 代理对象的获取

框架会默认为 注入/获取 至接口的场景注入代理对象。

### 2.1 通过标签注入代理对象

```go
// +ioc:autowire=true
// +ioc:autowire:type=singleton

type App struct {
    // 将结构对象注入至结构体指针
    ServiceStruct *ServiceStruct `singleton:""`
  
    // 将 main.ServiceImpl1 结构封装成代理对象，并注入至接口字段
    ServiceImpl Service `singleton:"main.ServiceImpl1"`
}
```

App 的 ServiceStruct 字段是具体结构的指针，字段本身已经可以定位期望被注入的结构，因此不需要在标签中给定期望被注入的结构名。

App 的 ServiceImpl 字段是一个名为 Service 的接口，期望注入的结构指针是 main.ServiceImpl。本质上是一个从结构到接口的断言逻辑，虽然框架可以进行接口实现的校验，但仍需要结构使用者保证注入的接口实现了该方法。**对于这种注入到接口的方式，IOC-golang 框架自动为 main.ServiceImpl 结构创建代理，并将代理结构注入在 ServiceImpl 字段，因此这一接口字段具备了 AOP 能力。**

因此，ioc 更建议开发者面向接口编程，而不是直接依赖具体结构，除了 AOP 能力之外，面向接口编程也会提高 go 代码的可读性、单元测试能力、模块解耦合程度等。

### 2.2 通过 API 的方式获取代理对象

IOC-golang 框架的开发者可以通过 [API 的方式](../../../examples/di/api/) 获取结构指针，通过调用自动装载模型（例如singleton）的 `GetImpl` 方法，可以获取结构指针。可以在生成的代码中找到类似如下的函数。

```go
func GetServiceStructSingleton() (*ServiceStruct, error) {
  i, err := singleton.GetImpl("main.ServiceStruct", nil)
  if err != nil {
    return nil, err
  }
  impl := i.(*ServiceStruct)
  // 返回原始结构体指针
  return impl, nil
}
```

上述获取的是 **结构体指针**，我们更推荐开发者通过 API ，调用下面的方法获取接口对象，通过调用自动装载模型（例如singleton）的 `GetImplWithProxy` 方法，可以获取**代理**对象，该对象可被断言为一个接口供使用。

在使用 iocli 工具生成代码的时候，会默认为每个结构生成一个**结构专属接口**，可以在生成的代码中找到类似如下的函数，通过调用该函数，可以直接获取专属接口形态的**代理对象**。

```go
func GetServiceStructIOCInterfaceSingleton() (ServiceStructIOCInterface, error) {
  // 获取代理对象
  i, err := singleton.GetImplWithProxy("main.ServiceStruct", nil)
  if err != nil {
    return nil, err
  }
  // 将代理对象断言成对象专属接口 ServiceStructIOCInterface
  impl := i.(ServiceStructIOCInterface) 
  // 返回结构专属接口形态的代理对象
  return impl, nil
}
```

这两种通过 API 获取对象的方式可以由 iocli 工具自动生成。注意！这些代码的作用都是方便开发者调用 API ，减少代码编写量，而 ioc 自动装载的逻辑内核并不是由工具生成的，这是与 wire 提供的依赖注入实现思路的不同点之一，也是很多开发者误解的一点。

## 3. 结构专属接口

通过上面的介绍，我们知道 IOC-golang 框架提供了封装 AOP 层的代理对象，其注入方式是 **强依赖接口** 的。但要求开发者为自己的全部结构都手写一个与之匹配的接口出来，这会耗费大量的时间。因此 iocli 工具可以自动生成结构专属接口，减轻开发人员的代码编写量。

例如一个名为 `ServiceImpl` 的结构，其包含 `GetHelloString` 方法

```go
// +ioc:autowire=true
// +ioc:autowire:type=singleton

type ServiceImpl struct {
}

func (s *ServiceImpl) GetHelloString(name string) string {
    return fmt.Sprintf("This is ServiceImpl1, hello %s", name)
}
```

当执行 iocli gen 命令后， 会在当前目录生成一份代码zz_generated.ioc.go 其中包含该结构的“专属接口”：

```go
type ServiceImplIOCInterface interface {
    GetHelloString(name string) string
}
```

专属接口的命名为 `$(结构名)IOCInterface`，专属接口包含了结构的全部方法。专属接口的作用有二：

1、减轻开发者工作量，方便直接通过 API 的方式 Get 到代理结构，方便直接作为字段注入，见上述1.2节。

2、结构专属接口可以直接定位结构 ID，因此在注入专属接口的时候，标签无需显式指定结构类型：

```go
// +ioc:autowire=true
// +ioc:autowire:type=singleton

type App struct {
    // 注入 ServiceImpl 结构专属接口，无需在标签中指定结构ID
    ServiceOwnInterface ServiceImplIOCInterface `singleton:""`
}
```
