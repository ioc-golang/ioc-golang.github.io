---
title: "项目结构"
date: 2017-01-05
weight: 1
description: >
  IOC-golang 框架项目结构
---

- **autowire：** 提供单例模型、多例模型两种基本注入模型
- **config：** 配置加载模块，负责解析用户yaml配置文件
- **debug：** 调试模块：提供调试 API、提供调试注入层实现
- **extension：** 组件扩展目录：提供基于多种注入模型的预置实现结构：

  - autowire：自动装载模型扩展
    
    - grpc：grpc 客户端模型定义
    
    - config：配置模型定义

  - config：配置注入模型扩展结构
  
    - string,int,map,slice
    
  - normal：多例模型扩展结构

    - redis
    
    - mysql
    
    - rocketmq
    
    - nacos
    
  - singleton：单例模型扩展结构
  
    - http-server
  
- **example：** 示例仓库

- **iocli：** 代码生成/程序调试 工具

  提供基于注解的结构描述信息自动生成能力
