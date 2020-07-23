## 版本

Konvoy 1.5

## 介绍Konvoy

### 特征和功能

* 简单安装
* 基础设施部署
* 生产级别的networking 和 storage 核心 addons
  * calic
  * traefik
  * local storage class

* metrics 和 logging 核心 addons
  * Prometheus, AlertManager, Grafana, Telegraf
  * Logging using Fluent Bit, Elasticsearch, Kibana
* 运维基础设施的 addons
* 运维 dashboard 
* 生命周期管理

### Benefis

* 提供灵活简单和灵活的平台来创建 applications
* 减少创建集群所使用的命令
* 通过使用单独的文件和单独的命令来安装、启动和定制化集群组件来简化部署和集群配置流程

* 借助 CNCF 技术来交付端到端的解决方案
* 减少构建部署多个基础设施所需要的时间、技能、成本和资源



## 概念和架构

![Architectural overview](https://docs.d2iq.com/ksphere/konvoy/1.5/img/Konvoy-arch-diagram.png)



### Kubernetes control plane 的 master components

#### 原生 Kubernetes cluster 的 master components 如下：

* kube-apiserver
* etcd
* kube-scheduler

* Kube-controller-manager

### worker nodes

* kubelet

* kube-proxy

* containerd

### Platform service addons

原生的K8s 支持众多 addons，addons 使用K8s 资源来实现具体的集群级别的特性，addons 是被定义在`kube-system` namespace.

作为生产级别的方案，Konvoy 默认提供 velero addon，支持对K8s 集群和持久化卷的 backup 和 restore 的操作。

### Konvoy port

Konvoy 的组件在每个节点上监听多个ports，为了安装成功，这些 ports 必须在安装的时候是可用的 

#### 开始之前

