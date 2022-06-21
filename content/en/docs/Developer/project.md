---
title: "Project Structure"
date: 2017-01-05
weight: 1
description: >
  IOC-golang Project Structure
---

- **autowire:** Provides two basic injection models: singleton model and multi-instance model
- **config:** Configuration loading module, responsible for parsing user yaml configuration files
- **debug:** Debug module: Provide debugging API, provide debugging injection layer implementation
- **extension:** Component extension directory: Provides preset implementation structures based on various injection models:

  - autowire: autoload model extensions

    - grpc: grpc client model definition

    - config: configure the model definition

  - config: configuration injection model extension structure

    - string,int,map,slice

  - normal: multi-instance model extension structure

    - redis

    - mysql

    - rocketmq

    - nacos

  - singleton: singleton model extension structure

    - http-server

- **example:** example repository

- **iocli:** code generation/program debugging tool

  Provides the ability to automatically generate annotation-based structural description information