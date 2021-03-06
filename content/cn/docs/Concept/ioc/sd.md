---
title: "结构描述符"
date: 2017-01-05
weight: 3
---

### 概念

**结构描述符(Struct Descriptor, SD)** 用于描述一个被开发者定义的结构，包含对象生命周期的全部信息，例如结构类型是什么，依赖哪些参数，如何被构造等等信息。

SD可以通过 [注解](/docs/concept/ioc/annotation) 的方式使用工具 [自动生成](/docs/reference/iocli/#结构注解与sdcndocsconceptsd代码生成) 。但还是推荐开发人员了解本框架定义的结构生命周期和结构描述信息，以便更清晰地了解加载过程。

### 对象生命周期

开发人员在 Go 语言开发过程中需要时时关注对象的生命周期，一个常见的对象生命周期如下：

1. 对象定义：开发人员编码，编写结构，实现接口，确定模型（单例、多例..) 。
2. 加载全部依赖：依赖的下游对象创建、配置读入等。
3. 对象创建：产生一个基于该对象的指针
4. 对象构造：获取全部依赖，并将其组装到空对象中，产生可用对象。
5. 对象使用：调用对象的方法，读写对象字段。
6. 对象销毁：销毁对象，销毁无需再使用的依赖对象。

### 参数

本框架的“参数”概念，是一个结构体，该结构体包含了创建一个对象所需全部依赖，并提供了构造方法。

例如：

```go
type Config struct {
	Host      string
	Port      string
	Username  string
	Password  string
}

func (c *Config) New(mysqlImpl *Impl) (*Impl, error) {
	var err error
	mysqlImpl.db, err = gorm.Open(mysql.Open(getMysqlLinkStr(c)), &gorm.Config{})
	mysqlImpl.tableName = c.TableName
	return mysqlImpl, err
}
```

Config 结构即为 Impl 结构的“参数”。其包含了产生 Impl 结构的全部信息。

### 结构描述符 （Struct Descriptor）

定义的结构描述符如下：摘自 [autowire/model.go](https://github.com/alibaba/IOC-golang/blob/master/autowire/model.go#L86)

```go
type StructDescriptor struct {
	Factory       func() interface{} // raw struct
	ParamFactory  func() interface{}
	ParamLoader   ParamLoader
	ConstructFunc func(impl interface{}, param interface{}) (interface{}, error) 
	DestroyFunc   func(impl interface{})
	Alias         string // alias of SDID
	TransactionMethodsMap map[string]string // transaction

	impledStructPtr interface{} // impledStructPtr is only used to get name
}
```


- Factory【必要】

  结构的工厂函数，返回值是未经初始化的空结构指针，例如 

  ```go
  func () interface{}{
  	return &App{}
  }
  ```

- ParamFactory【非必要】

  参数的工厂函数，返回值是未经初始化的空参数指针，例如 

  ```go
  func () interface{}{
  	return &Config{}
  }
  ```

- ParamLoader【非必要】

  参数加载器定义了参数的各个字段如何被加载，是从注入标签传入、还是从配置读入、或是以一些定制化的方式。

  框架提供了默认的参数加载器，详情参阅 [参数加载器概念](/cn/docs/concept/param_loader) 

- Constructor【非必要】

  构造函数定义了对象被组装的过程。

  入参为对象指针和参数指针，其中对象指针的所有依赖标签**都已被注入下游对象**，可以在构造函数中调用下游对象。参数指针的所有字段**都已经按照参数加载器的要求加载好**。构造函数只负责拼装。

  返回值为经过拼装后的指针。例如：

  ```go
  func(i interface{}, p interface{}) (interface{}, error) {
    param := p.(*Config)
    impl := i.(*Impl)
    return param.New(impl)
  },
  ```

- DestroyFunc 【非必要】

  定义了对象的销毁过程，入参为对象指针。

- Alias 【非必要】

  由于 [结构ID](/docs/concept/ioc/sd/#结构id) 一般较长，可以在这里指定结构的别名，可以通过这一别名替换 [结构ID](3/docs/concept/ioc/sd/#结构id)，调用 [对象获取 API](/docs/examples/di/api/)。

- TransactionMethodsMap 【非必要】

  基于 Saga 模型的事务函数声明， 这一 Map 的 Key 为需要使用事务能力的方法名，Value 为该方法的回滚函数，如果 Value 为空，则无回滚逻辑。参考事务例子 [example/transaction](https://github.com/alibaba/IOC-golang/tree/master/example/aop/transaction)

### 结构ID

- 定义

结构(描述) ID定义为："${包名}.${结构名}"

结构(描述) ID （Struct Description Identification) 在本文档和项目中多处被缩写为 SDID。

SDID 是唯一的，用于索引结构的键，类型为字符串。


- 使用

例如，开发人员在 [使用 API 获取对象](/docs/examples/di/api/) 时，需要针对使用的自动装载模型，传入 SDID 来定位结构，从而获取对象。

