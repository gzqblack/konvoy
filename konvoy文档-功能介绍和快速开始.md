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

* 为了执行安装，ansible 需要ssh 连接 22 端口
* 多种网络组件一起构成 Konvoy 的网络stack. 参考[networking](https://docs.d2iq.com/ksphere/konvoy/1.5/networking)

* 必须使用适合的网络机制来阻止针对网络节点的未授权的访问 [security]([security](https://docs.d2iq.com/ksphere/konvoy/1.5/security))
* 默认 pods 是非隔离的，可以接受任何的访问流量。通过使用网络策略，Pods 可以成为隔离的。当有任何 namespace 中的任何 NetworkPolicy 选择了 pod，pod 将会拒绝任何 NetworkPolicy 不允许的连接。具体细节可以参考 Konvoy  集成 calico。[network policies](https://docs.d2iq.com/ksphere/konvoy/1.5/networking/#network-policy) 
* 在安装的过程中，konvoy 可以配置自动增加 如下的 [iptables]([iptables](https://docs.d2iq.com/ksphere/konvoy/1.5/networking/#iptables)) 的规则

## 快速开始

Konvoy 提供开箱即用的解决方案，提供生产级别的Kubernete. This Quick Start guaid i通过你了简单的操作命令在AWS 公有云上安装 Konvoy 集群和运行最小的配置需求

### 先决条件

- Linux or MacOS
- aws cli
- docker 18.09.2+
- kubectl v1.17.8+
- aws account
  - EC2 instances
  - VPC
  - Subnets
  - ELB
  - Internet Gateway
  - NAT Gateway
  - EBS Volumes
  - Security Groups
  - Route Tables
  - IAM Role

### 安装 konvoy

1. 安装需要的软件包

`brew install kubernetes-cli awscli`

2. 检查 kubernetes client 版本

`kubectl version --short=true`

3. 下载 konvoy的安装包，[Downlaod Konvoy](https://docs.d2iq.com/ksphere/konvoy/latest/download/)

4. 采用默认安装
5. 验证你有有效的 **AWS security credentials**，如果你在 on-premises 环境中安装，可以不需要。
6. 执行如下命令，创建目录用于存储集群状态信息

```
mkdir konvoy-quickstart
cd konvoy-quickstart
```

7. 使用默认的设置和addons部署，运行如下的命令：

 ```
konvoy up
 ```

`Konvoy up` 命令执行如下任务

* 3 个control plane machines `m5.xlarge` (高可用的 control-plane API)

* 4 个worker machines `m5.2xlarge` 在 AWS

* 部署如下的default addons

  * calico
  * CoreDNS
  * Helm
  * AWS EBS CSI driver
  * Elasticsearch(including Elasticsearch exporter)
  * Kibana
  * Fluent Bit
  * Prometheus operator(including Grafana AlertManager and Prometheus Adaptor)
  * Traefik (layer 7)
  * kubernetes dashboard
  * operations portal 
  * velero
  * Dex indentity service
  * Dex Kubernetes client authenticator
  * Traefik forward authorization proxy
  * kommander

  

### 安装验证

`konvoy up` 的输出结果类似于

```
Kubernetes cluster and addons deployed successfully!

Run `konvoy apply kubeconfig` to update kubectl credentials.

Navigate to the URL below to access various services running in the cluster.
  https://lb_addr-12345.us-west-2.elb.amazonaws.com/ops/landing
And login using the credentials below.
  Username: AUTO_GENERATED_USERNAME
  Password: SOME_AUTO_GENERATED_PASSWORD_12345

The dashboard and services may take a few minutes to be accessible.
```



### 查看集群和addons

默认的 **operations portal** 为多个 service 提供dashboard 的链接，包括：

* Grafana
* Kibana
* Prometheus AlertManager dashboard
* Traefik dashboards
* Kubernetes dashboard for cluster activity

### Merge kubeconfig

集群部署完毕后，在使用`kubectl` 链接集群之前，可以存储 access 配置信息在你的重要的`kubeconfig` 文件中。

Access 配置文件包括证书和 API server endpoint。 `konvoy` 集群存储这些内部信息在 `admin.conf`，但是你可以merge他们到`kubeconfig` 文件，你可以从你的机器的别的工作目录访问集群。

merge 访问配置文件，执行如下命令：

`konvoy apply kubeconfig`

1. 指定 kubeconfig 位置

默认使用`KUBECONFIG`环境变量来声明配置文件的位置。如果 `KUBECONFIG` 为定义，默认的路径 `~/.kube/config` 会被使用。

可以采用两种方法覆盖默认的Kubernetes 配置路径：

* 指定路径

```
export KUBECONFIG="${HOME}/.kube/konvoy.conf"
konvoy apply kubeconfig
```

* 指定`KUBECONFIG`至目前配置文件的路径

```
export KUBECONFIG="${PWD}/admin.conf"
```

2. 验证merge 的配置

```
kubectl get nodes
```

输出类似于：

```
NAME                                         STATUS   ROLES    AGE   VERSION
ip-10-0-129-3.us-west-2.compute.internal     Ready    <none>   24m   v1.17.8
ip-10-0-131-215.us-west-2.compute.internal   Ready    <none>   24m   v1.17.8
ip-10-0-131-239.us-west-2.compute.internal   Ready    <none>   24m   v1.17.8
ip-10-0-131-24.us-west-2.compute.internal    Ready    <none>   24m   v1.17.8
ip-10-0-192-174.us-west-2.compute.internal   Ready    master   25m   v1.17.8
ip-10-0-194-137.us-west-2.compute.internal   Ready    master   26m   v1.17.8
ip-10-0-195-215.us-west-2.compute.internal   Ready    master   26m   v1.17.8
```

### 下一步

接下来可以试用一下集群了，可以参考：

- [Deploy a sample application](https://docs.d2iq.com/ksphere/konvoy/1.5/tutorials/deploy-sample-app/)
- [Provision a customized cluster](https://docs.d2iq.com/ksphere/konvoy/1.5/tutorials/provision-a-custom-cluster/)
- [Check component integrity](https://docs.d2iq.com/ksphere/konvoy/1.5/troubleshooting/check-components/)
- [Troubleshooting](https://docs.d2iq.com/ksphere/konvoy/1.5/troubleshooting/)

