---
title: "注解"
date: 2017-01-05
weight: 5
---

### 概念

注解(annotation) 是 Go 代码中符合特定格式的注释，一般位于结构定义之前。可被命令行工具扫描，从而获取结构信息，自动生成需要的代码。
例如[快速开始](/cn/docs/getting-started/tutorial)示例中的
```go
// +ioc:autowire=true
// +ioc:autowire:type=singleton

type App struct {
	...
}

```

注解在 Go 应用开发的过程中，是一种直观、清晰的结构描述方式，通过使用注解进行标注，使用工具自动生成相关代码，可减少开发人员的工作量，降低代码重复率。

### 注解与代码生成

注解与代码生成能力，是为了让开发者无需关心[SD(结构描述符)](/cn/docs/concept/sd)的组装和注册。开发者只需定义好结构，正确标注注解，即可使用 iocli 工具自动生成当前目录和子目录下的 SD 。

### iocli 工具支持的注解

详情参阅 [iocli #结构注解](/cn/docs/reference/iocli#结构注解与sdcndocsconceptsd代码生成)

