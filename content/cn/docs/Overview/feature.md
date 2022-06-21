---
title: "功能"
date: 2017-01-05
weight: 4
description: >
  IOC-golang 框架提供的能力
---

- [依赖注入](https://ioc-golang.github.io/cn/docs/getting-started/tutorial/)

  支持任何结构、接口的依赖注入，具备完善的对象生命周期管理机制。

  可以接管对象的创建、参数注入、工厂方法。可定制化对象参数来源。

- [结构代理层](https://ioc-golang.github.io/cn/docs/getting-started/tutorial/)

  基于 AOP 的思路，为由框架接管的对象提供默认的结构代理层，在面向接口编程的情景下，可以使用基于结构代理 AOP 层扩展的丰富运维能力。例如接口查询，参数动态监听，方法粒度链路追踪，性能瓶颈分析，分布式场景下全链路方法粒度追踪等。

- [代码生成能力](https://ioc-golang.github.io/cn/docs/reference/iocli/#%E7%BB%93%E6%9E%84%E6%B3%A8%E8%A7%A3)

  我们提供了代码生成工具，开发者可以通过注解的方式标注结构，从而便捷地生成结构注册代码、结构代理、结构专属接口等。

- [可扩展能力](https://ioc-golang.github.io/cn/docs/contribution-guidelines/)

  支持被注入结构的扩展、自动装载模型的扩展、调试 AOP 层的扩展。

- [丰富的预置组件](https://ioc-golang.github.io/cn/docs/examples/)

  提供覆盖主流中间件的预制对象，方便直接注入使用。