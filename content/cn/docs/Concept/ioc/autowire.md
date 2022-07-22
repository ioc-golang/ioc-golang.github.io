---
title: "自动装载模型"
date: 2017-01-05
weight: 2
---

### 概念

**自动装载(Autowire)模型** 封装了一类对象的装载方式，是本框架对装载过程抽象出的概念和接口。

用户需要定义的结构千变万化，如果每个结构描述符都要提供完整的装载过程信息，将是一件很麻烦的事情。我们将一类相似的结构抽象出一个 **自动装载模型**，选择注册在该模型的所有结构都需要遵循该模型的加载策略，这大大降低了用户需要提供的 [结构描述符](../sd) 内的定制化信息量，从而提高开发效率。

框架内置了两个基础自动装载模型：[单例模型（singleton）](https://github.com/alibaba/IOC-golang/tree/master/autowire/singleton)，[多例模型（normal）](https://github.com/alibaba/IOC-golang/tree/master/autowire/normal)

当前版本中，框架内置了三个[扩展的自动装载模型](https://github.com/alibaba/IOC-golang/tree/master/extension#%E5%9F%BA%E4%BA%8E-%E8%87%AA%E5%8A%A8%E8%A3%85%E8%BD%BD%E6%A8%A1%E5%9E%8B-%E5%92%8C-aop-%E8%83%BD%E5%8A%9B%E7%9A%84%E6%89%A9%E5%B1%95)：配置（config），gRPC 客户端（grpc），RPC（rpc）。其中配置模型是多例模型的扩展，gRPC 客户端是单例模型的扩展，RPC 模型提供了（rpc-client 和 rpc-server）两侧的自动装载模型。关于这三个自动装载模型的应用，可以参考[example/autowire](https://github.com/alibaba/IOC-golang/tree/master/example/autowire) [example/third_party/grpc](https://github.com/alibaba/IOC-golang/tree/master/example/third_party/autowire/grpc) 中给出的例子。

基于这些自动装载模型，框架内置了基于“扩展自动装载模型”的多个结构。例如，用户可以用几行代码将 “gRPC 客户端存根”注册在 “grpc 装载模型” 之上[【示例】](/docs/examples/autowire/grpc)，再例如可以方便地从配置文件中的 [指定位置读入](/docs/reference/yaml_structure/) 数据。

