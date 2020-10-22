# 深入浅出Istio

## 服务网格的历史

* Spring cloud
* Linkerd
  * 2016 年
  * 2018 年 Linkerd 2.0
* Istio
  * 2016 年， envoy
  * 2017 年，Istio



## 服务网格的基本特性

服务网格是一个独立的基础设施层，用来处理服务之间的通信。

Istio的特点总结：

* 连接
* 安全
* 策略
* 观察

1. 连接

* 服务注册与发现
* 负载均衡策略
* 服务流量特征
* 动态流量分配



2. 安全

提供网格内部的安全保障，具备服务通信加密、服务身份认证和服务访问控制（授权和鉴权）功能、数字证书的管理。

3. 策略

控制方面，例如对调用频率的限制、对服务互访的控制、以及针对流量的一些限制和变更能力等。

Mixer 作为策略的执行者，拥有对流量的部分控制能力，在 Istio 中还有为数众多的内部适配器及进程外适配器，以及和外部软件设施一同完成策略的制定和执行。

4. 观察

调用成功率、响应时间、调用量、传输量、分布式跟踪。



## Istio 的基本介绍

设计目标是在kubernetes 的基础上，以非侵入的方式为运行在集群中的微服务提供流量管理、安全加固、服务监控和策略管理等功能。

### Istio 的核心组件

* 数据面，Sidecar
* 控制面

#### Pilot

* 从kubernetes 或者其他平台的注册中心获取服务信息，完成服务过程；
* 读取 Istio 的各项控制配置，在进行转换之后，将其发给数据面进行实施。

Pilot 的工作流程

* 用户通过kubectl 或 istioctl 在 kubernetes 上创建 CRD 资源，对 Istio 控制平面发出指令；
* Pilot 监听 CRD 中的 config、rbac、networking 及 authentication 资源，在检测到资源对象的变更之后，针对其中涉及的服务，发出指令给对应服务的 Sidecar；
* Sidecar 根据这些指令更新自身配置，根据配置修正通信行为。



#### Mixer

Mixer 的工作流：

* 用户讲 Mixer 配置发送到 kubernetes
* Mixer 通过对 Kubernetes 资源的监听，获知配置的变化
* 网格中的服务在每次调用之前，都向 Mixer 发出预检请求，查看调用是否允许执行。每次调用之后，都发出报告信息，向 Mixer 汇报在调用过程中产生的监控跟踪数据。



#### Citadel

用于证书管理



#### Sidecar （Envoy）

Sidecar 就是 Istio 的数据面，负责控制面对网格控制的实际执行。

Sidecar 加入之后，原有的源容器 -> 目标容器的直接通信方式，变成了源容器->sidecar->sidecar->目标容器的模式。Sidecar 是用来接受控制面板组件的操作的，这样一来，就让通信过程中的控制和观察成为可能。



### 核心配置对象

Istio 在安装过程中会进行 CRD 的初始化，在 Kubernetes 集群中注册一系列的 CRD，注册成功后，会建立一些基础对象，完成 Istio 的初始设置。

Istio 中的资源分为三组进行管理，分别是 networking.istio.io, config.istio.io , authentication.istion.io ，下面将分别进行介绍。

#### networking.istio.io

networking.istio.io 系列对象在 Istio 中可能是使用频率最高的， Istio 的流量管理功能就是用这一组对象完成的，这里选择其中最常用的对象进行简单介绍。

VirtualService 是一个控制中心。



1. Gateway

网络边缘的 Ingress 流量会通过对应的 Istio Ingress Gateway Controller 进入，网格内部的服务互访，则是通过虚拟的 mesh 网关进行。

Pilot 会根据 Gateway 和主机名进行检索，如果存在对应的 VirtualService，则交由 VirtualServce 处理，如果是 Mesh Gateway 且不存在对应这一主机名的VirtualService，则尝试调用 Kubernetes Service；如果不存在，则发生 404 错误。



2. VirtualService

* host，主机名称，如果在 Kubernetes 集群中，则这个主机名可以是服务名。
* Gateway，流量的来源网管。默认的网格内部服务互联所用的网关。
* 路由对象，网格中的流量，可以对 HTTP 协议进行更细致的控制。



3. TCP/TLS/HTTP Route

路由对象目前可以是HTTP、TCP 或者 TLS 中的一个，分别针对不同的协议进行工作。每种路由对象都至少包括两部分： 匹配条件和目的路由。



4. DestinationWeight

各协议路由的目标定义是一致的，都由 DestinationWeight 对象数组来完成。



5. Destination

目标对象由Subset 和 Port 两个元素组成。



#### config.istio.io

config.istio.io 中的对象用于为 Mixer 组件提供配置。Mixer 提供了预检和报告两个功能。

1. Rule

Rule 对象是Mixer 的入口，其中包含一个 match 成员和一个逻辑表达式，只有符合表达式判断的数据才会被交给 Action 处理。逻辑表达式中的变量被称为 attribute，其中的内容来自 envoy 提交的数据。

2. Action

Action负责解决的问题就是：将符合入口标准的数据，在用什么方式加工之后，交给哪个适配器进行处理。

Action 包含2个成员对象。1，Instance；2，handler



3. Instance

Instance 主要用于为进入的数据选择一个模板，并在数据中抽取某些字段作为模版的擦数，传输给模版进行处理。



4. Adapter

Adapter 在 Istio 中被定义为一个行为规范，而一些必要的实例化数据是需要再次进行初始化的。

经过 Handler 实例化之后的 Adapter，就具备了工作功能。

Envoy 传出的数据将会通过这些具体运行的 Adapter 的处理，得到预检结果，或者输出各种监控、日志及跟踪数据。

5. Template

Template 是一个模板，用于对接收到的数据进行再加工。

进入Mixer 中的数据都来自于 Sidecar，但是各种适配器应对的需求各有千秋，甚至同样一个适配器，也可能接受各种不同形式的数据。Envoy 提供的原始数据和适配器所需要的输入数据存在格式上的差别，因此需要对原始数据进行再加工。

Template 就是这样一种工具，在用户编制模版对象之后，经过模版处理的原始数据会被转换为符合适配器输入要求的数据格式，这样就可以在 Instance 字段中引用了。

6. Handler

Handler 对象用于对 Adapter 进行实例化。







