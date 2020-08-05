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

