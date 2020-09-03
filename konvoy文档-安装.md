## 支持的操作系统

### CentOS

| OS Release                                                   | Kernel Version              |
| ------------------------------------------------------------ | --------------------------- |
| [CentOS 7.7](https://wiki.centos.org/action/show/Manuals/ReleaseNotes/CentOS7.2003) | 3.10.0-1062.12.1.el7.x86_64 |
| [CentOS 7.8](https://console.cloud.google.com/marketplace/details/centos-cloud/centos-7) (Only on GCP) | 3.10.0-1127.8.2.el7.x86_6   |

### RHEL

| OS Release                                                   | Kernel Version              |
| ------------------------------------------------------------ | --------------------------- |
| [RHEL_7.7](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/7.7_release_notes/index) | 3.10.0-1062.12.1.el7.x86_64 |

### Ubuntu

| OS Release                                                   | Kernel Version |
| ------------------------------------------------------------ | -------------- |
| [Ubuntu 16.04 (xenial)](https://wiki.ubuntu.com/XenialXerus/ReleaseNotes) | 4.4.0-1087     |
| [Ubuntu 18.04 (bionicbeaver)](https://wiki.ubuntu.com/BionicBeaver/ReleaseNotes) | 5.0            |

### Debian

| OS Release                                                   | Kernel Version |
| ------------------------------------------------------------ | -------------- |
| [Debian 9 (stretch)](https://www.debian.org/releases/stretch/releasenotes) | 4.9.0-9        |
| [Debian 10 (buster)](https://www.debian.org/releases/buster/releasenotes) | 4.19.67-2      |



### SUSE Linux Enterprise Sever

| OS Release                                       |
| ------------------------------------------------ |
| [15](https://documentation.suse.com/sles/15-SP1) |



## 在AWS 上安装

### 开始之前

* aws cli
* Docker 18.09.2+
* Kubectl v1.17.8+
* 有效的AWS account

### 安装

验证完 prerequisites 之后，可以试用`konvoy up` 来创建集群，这个命令会创建Amazon EC2 实例，安装 Kubernetes，安装默认 addons 来支持 Kubernetes 集群。

> 如果你想定制化你的安装，你可以执行 `konvoy init` 命令，编辑已经创建的`cluster.yaml`文件

`konvoy up `命令执行如下：

- Provisions three `m5.xlarge` EC2 instances as Kubernetes master nodes
- Provisions four `m5.2xlarge` EC2 instances as Kubernetes worker nodes
- Deploys all of the following default addons:
  - Calico
  - CoreDNS
  - Helm
  - AWS EBS CSI driver
  - Elasticsearch (including Elasticsearch Exporter)
  - Fluent Bit
  - Kibana
  - Prometheus operator (including Grafana, AlertManager and Prometheus Adapter)
  - Traefik
  - Kubernetes dashboard
  - Operations portal
  - Velero
  - Dex identity service
  - Dex Kubernetes client authenticator
  - Traefik forward authorization proxy
  - Kommander

### 修改集群名字

默认情况下，集群的名字就是你执行konvoy命令所在目录的名字。定义集群名字可以使用如下命令。

```
konvoy up --cluster-name <YOUR_SPECIFIED_NAME>
```

> 注意： 集群的名字包含如下字符：`a-z,0-9,.-and_`



### 显示预计的基础设施的改变

```
$ konvoy provision --plan-only
...
Plan: 41 to add, 0 to change, 0 to destroy.
```

### Control plane 和 worker nodes

Control plane 包括 etcd，kube-apiserver，kube-scheduler，kube-controller-manager。3个control plane 保证高可用。worker node 运行 container，pods。

### Default addons

默认的 addons 帮助你管理集群，包括 monitoring(Prometheus)， logging(Elasticsearch), dashboard (Kubernetes Dashboard), ingress (Traefik) 和其他的 services.

### 查看安装操作

执行`konvoy up` 命令，你将会看到操作输出，前面的信息是由 terraform 产生的，用于部署节点。

node 部署完毕后，ansible 连接 EC2 实例安装 Kubernetes ，在输出的最后是addons 安装。

### 查看集群的操作

可以通过 Operations Portal 访问用户界面来监视集群。在执行完 `konvoy up` 命令后，如果安装成功，命令输出类似于：

```
Kubernetes cluster and addons deployed successfully!

Run `./konvoy apply kubeconfig` to update kubectl credentials.

Run `./konvoy check` to verify that the cluster has reached a steady state and all deployments have finished.

Navigate to the URL below to access various services running in the cluster.
  https://lb_addr-12345.us-west-2.elb.amazonaws.com/ops/landing
And login using the credentials below.
  Username: AUTO_GENERATED_USERNAME
  Password: SOME_AUTO_GENERATED_PASSWORD_12345

If the cluster was recently created, the dashboard and services may take a few minutes to be accessible.
```

### 检查安装的文件











