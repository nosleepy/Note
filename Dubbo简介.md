---
title: Dubbo简介
date: 2020-05-24 09:39:25
tags:
categories:
- 框架
---

## 什么是Dubbo

​Dubbo 是阿里巴巴提供的开源的 SOA 服务化治理的技术框架。它最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦合）。从服务模型的角度来看，Dubbo 采用的是一种非常简单的模型，要么是提供方提供服务，要么是消费方消费服务，所以基于这一点可以抽象出服务提供方（Provider）和服务消费方（Consumer）两个角色。 除了提供**服务**之外，它还提供了**负载均衡**，**监控中心**和**调度中心**。

下图展示了它涉及的服务治理：

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/Dubbo服务治理.png" width="360px"/>

​在大规模服务化之前，应用可能只是通过 RMI 或 Hessian 等工具，简单的暴露和引用远程服务，通过配置服务的URL地址进行调用，通过 F5 等硬件进行负载均衡。当服务越来越多时，服务 URL 配置管理变得非常困难，F5 硬件负载均衡器的单点压力也越来越大。 此时需要一个服务注册中心，动态的注册和发现服务，使服务的位置透明。并通过在消费方获取服务提供方地址列表，实现软负载均衡和 Failover，降低对 F5 硬件负载均衡器的依赖，也能减少部分成本。

## 如何使用Dubbo

### Dubbo的架构

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/Dubbo架构.png" width="320px"/>

节点角色说明：
   + Provider：暴露服务的服务提供方。
   + Consumer：调用远程服务的服务消费方。
   + Registry：服务注册与发现的注册中心。
   + Monitor：统计服务的调用次数和调用时间的监控中心。
   + Container：服务运行容器。

调用关系说明：
   + 服务容器负责启动，加载，运行服务提供者。
   + 服务提供者在启动时，向注册中心注册自己提供的服务。
   + 服务消费者在启动时，向注册中心订阅自己所需的服务。
   + 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
   + 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
   + 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

我们从上面的架构可以看出，使用 dubbo 之后我们不需要关注服务是如何被远程调用，服务之间调用好像在本地调用一样，充分发掘其强大的服务治理，我们可以减少运维成本，将注意力专注于业务本身的开发。

### Dubbo注册中心

​Zookeeper 是 hadoop 的一个子项目，是分布式的，开放源码的分布式应用程序协调服务，它包含一个简单的原语集，分布式应用程序可以基于它实现同步服务，配置维护和命名服务等。在与 dubbo 结合使用时，作为其注册中心，负责 dubbo 的所有服务地址列表维护，并且可以通过在 ZooKeeper 节点中设置相应的值来实现对这个服务的权重、优先级、是否可用、路由、权限等的控制。

下图是 zookeeper 的结构图

<img src="https://cdn.jsdelivr.net/gh/zhx2020/picture/img/Zookeeper结构图.png" width="320px"/>

## 参考

+ [微服务架构之Dubbo简介](http://luckylau.tech/2018/01/09/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%9E%B6%E6%9E%84%E4%B9%8BDubbo-1/)