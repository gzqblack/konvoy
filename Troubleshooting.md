### 检查组件的完整性

每个 Konvoy 集群包括几个必须的组件来确保集群的运行。 Konvoy cli 提供子命令来确保组件的完整性。

#### 检查部署环境

可以使用 `pre-flight check` 在执行`konvoy deploy` 之前验证 hosts 的配置和状态。

``` 
konvoy check preflight
```

输出如下

```
STAGE [Running Preflights]

PLAY [Bootstrap Bastion Nodes] *********************************************************************************************************************************************
skipping: no hosts matched

PLAY [Bootstrap Nodes] *****************************************************************************************************************************************************

TASK [wait-ssh : wait 180 seconds for SSH target connection to become reachable (socket)] **********************************************************************************
ok: [10.0.128.252 -> localhost]
ok: [10.0.194.33 -> localhost]
ok: [10.0.128.170 -> localhost]
ok: [10.0.193.146 -> localhost]
ok: [10.0.192.97 -> localhost]
ok: [10.0.131.119 -> localhost]
ok: [10.0.131.17 -> localhost]

TASK [wait-ssh : wait 180 seconds for SSH target connection to become reachable (SSH)] *************************************************************************************
skipping: [10.0.128.170]
skipping: [10.0.131.17]
skipping: [10.0.128.252]
skipping: [10.0.131.119]
skipping: [10.0.194.33]
skipping: [10.0.193.146]
skipping: [10.0.192.97]
<... other checks>
PLAY RECAP *****************************************************************************************************************************************************************
10.0.128.170               : ok=22   changed=0    unreachable=0    failed=0
10.0.128.252               : ok=18   changed=0    unreachable=0    failed=0
10.0.131.119               : ok=18   changed=0    unreachable=0    failed=0
10.0.131.17                : ok=18   changed=0    unreachable=0    failed=0
10.0.192.97                : ok=17   changed=0    unreachable=0    failed=0
10.0.193.146               : ok=17   changed=0    unreachable=0    failed=0
10.0.194.33                : ok=17   changed=0    unreachable=0    failed=0
```

#### 检查 addons

```  
konvoy check addons
```

输出如下

```
STAGE [Checking Addons]
awsebscsiprovisioner                                                   [OK]
cert-manager                                                           [OK]
dashboard                                                              [OK]
defaultstorageclass-protection                                         [OK]
<... other addons>
```



#### 检查Kubernetes

```
konvoy check kubernetes
```

输出如下

```
STAGE [Checking Kubernetes]

PLAY [Check Control Plane Health] ********************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************
ok: [10.0.195.156]
ok: [10.0.128.10]

TASK [check-kubernetes : calico health] **************************************************************************************************************************************
ok: [10.0.128.10 -> ec2-34-222-69-18.us-west-2.compute.amazonaws.com]
ok: [10.0.195.156 -> ec2-34-222-69-18.us-west-2.compute.amazonaws.com]

TASK [check-kubernetes : coredns health] *************************************************************************************************************************************
ok: [10.0.128.10 -> ec2-34-222-69-18.us-west-2.compute.amazonaws.com]

TASK [check-kubernetes : count number of nodes] ******************************************************************************************************************************
ok: [10.0.128.10 -> ec2-34-222-69-18.us-west-2.compute.amazonaws.com]

TASK [check-kubernetes : check nodes are Ready] ******************************************************************************************************************************
ok: [10.0.128.10 -> ec2-34-222-69-18.us-west-2.compute.amazonaws.com]

PLAY RECAP *******************************************************************************************************************************************************************
10.0.128.10                : ok=5    changed=0    unreachable=0    failed=0
10.0.195.156               : ok=2    changed=0    unreachable=0    failed=0

```



#### 检查节点

```
konvoy check nodes
```

输出如下

```
STAGE [Running Preflights]

PLAY [Bootstrap Bastion Nodes] *********************************************************************************************************************************************
skipping: no hosts matched

PLAY [Bootstrap Nodes] *****************************************************************************************************************************************************

TASK [wait-ssh : wait 180 seconds for SSH target connection to become reachable (socket)] **********************************************************************************
ok: [10.0.131.119 -> localhost]
ok: [10.0.128.170 -> localhost]
ok: [10.0.193.146 -> localhost]
ok: [10.0.192.97 -> localhost]
ok: [10.0.128.252 -> localhost]
ok: [10.0.131.17 -> localhost]
ok: [10.0.194.33 -> localhost]
<... various preflight check output>

STAGE [Checking Nodes]

PLAY [Check Machine Readiness] *********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [10.0.194.33]
ok: [10.0.131.17]
ok: [10.0.128.170]
ok: [10.0.128.252]
ok: [10.0.193.146]
ok: [10.0.192.97]
ok: [10.0.131.119]
<... various node check output>

TASK [check-nodes : kube-scheduler health] *********************************************************************************************************************************
skipping: [10.0.128.170]
skipping: [10.0.131.17]
skipping: [10.0.128.252]
skipping: [10.0.131.119]
ok: [10.0.194.33]
ok: [10.0.193.146]
ok: [10.0.192.97]

PLAY RECAP *******************************************************************************************************************************************************************
10.0.128.10                : ok=12   changed=0    unreachable=0    failed=0
10.0.195.156               : ok=15   changed=0    unreachable=0    failed=0
```



### 安装失败

#### 部署失败

执行`konvoy up` 或者 `konovy deloy` 出现失败

出现部署失败的大部分原因是由于与底层基础设施的API连接出现错误，例如，如果你在公有云AWS上进行部署，失败的大多数原因是由于调用 AWS API 造成的。

因为 Konvoy 使用 `terraform` 执行部署流程，如果问题出现，可以试用 `konvoy up --verbose `命令来输出错误。

```
konvoy up --verbose
```



#### 过期或无效的credentials

错误类似

```
An error occurred (ExpiredToken) when calling the GetCallerIdentity operation: The security token included in the request is expired
Please refresh your AWS credentials; the AWS API could not be reached with your current credentials.
```

错误信息指出 `~/.aws/credentials` 和 `AWS_PROFILE` credentials 过期，需要 renew

#### hosts 在部署后失联

在部署较大的集群的时候可能会碰到此问题。通常，ansible 尝试与所有节点进行持续快速交互的时候会出现。这种情景下，Cloud providers' 网络可能会限制与多个hosts进行交互的数量。限定可能会引发ansible 部分执行失败，可以通过重复执行命令来解决。



### Addon 错误或失败

有时我们执行`konvoy up` 命令后会出现 addons 的错误

#### 检查addon 部署的状态

执行`konvoy up` 或者 `konvoy deploy` 后，有成功的情况，例如

```
STAGE [Deploying Enabled Addons]
konvoyconfig                                                           [OK]
dashboard                                                              [OK]
external-dns                                                           [OK]
reloader                                                               [OK]
opsportal                                                              [OK]
cert-manager                                                           [OK]
gatekeeper                                                             [OK]
defaultstorageclass-protection                                         [OK]
traefik                                                                [OK]
awsebscsiprovisioner                                                   [OK]
dex                                                                    [OK]
kube-oidc-proxy                                                        [OK]
traefik-forward-auth                                                   [OK]
prometheus                                                             [OK]
dex-k8s-authenticator                                                  [OK]
prometheusadapter                                                      [OK]
velero                                                                 [OK]
kommander                                                              [OK]
elasticsearch                                                          [OK]
elasticsearch-curator                                                  [OK]
elasticsearchexporter                                                  [OK]
fluentbit                                                              [OK]
kibana                                                                 [OK]
```

有失败的情况

```
traefik                                                                [ERROR]

Error: error installing: the following 1 addon failed to deploy: [ traefik ]
```

Konvoy 可能会直接把错误输出，在执行 `konvoy deploy addons` 之前

很多的addon 是通过 `helm` 部署的，可以先执行，

```
helm list
```

检查 deployments 或者 pods

检查 deployments

```
kubectl -n kubeaddons get deployment traefik-kubeaddons
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
traefik-kubeaddons   0/2     0            0           10m
```

```
kubectl -n kubeaddons get deployment traefik-kubeaddons
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
traefik-kubeaddons   0/2     0            0           10m
```



检查pods

```
kubectl -n kubeaddons get pods | grep traefik
```

检查log

```
kubectl -n kubeaddons logs traefik-kubeaddons-<id>
```



### 替换掉一个failed control plane node

1. 确认 failed control plane node  和 其它的节点没有联系

```
kubectl get nodes --output=custom-columns="NAME":".metadata.name","READY":".status.conditions[?(@.type==\"Ready\")].status"
```

这个例子中，`konvoy-example-cluster-control-plane-1 `节点没有ready 

2. 永久的删除failed 的节点

例如，这个node 是 AWS EC2 instance，使用 AWS CLI or Console 来删除这个节点

3. 验证 etcd 成员可以接收 etcd api requests

```
kubectl -n kube-system get pod --selector=tier=control-plane,component=etcd --output=custom-columns="NAME":".metadata.name","READY":".status.conditions[?(@.type==\"Ready\")].status"
```

这个例子中，plane-1，plane-2 是 ready 的。

```
NAME                                          READY
etcd-konvoy-example-cluster-control-plane-0   True
etcd-konvoy-example-cluster-control-plane-1   False
etcd-konvoy-example-cluster-control-plane-2   True
```

4. 验证 failed etcd member

找到 etcd ID

```
READY_ETCD_MEMBER="<name of etcd member from previous step>"
ETCDCTL="ETCDCTL_API=3 etcdctl --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints=https://127.0.0.1:2379"
kubectl -n kube-system exec -it "$READY_ETCD_MEMBER" -- /bin/sh -c "$ETCDCTL member list"
```

在这个例子中，failed etcd ID 是`1d021ffdd096a804`，copy一下

```
1d021ffdd096a804, started, konvoy-example-cluster-control-plane-1, https://172.17.0.6:2380, https://172.17.0.6:2379, false
40fd14fa28910cab, started, konvoy-example-cluster-control-plane-0, https://172.17.0.4:2380, https://172.17.0.4:2379, false
87651970646a8073, started, konvoy-example-cluster-control-plane-2, https://172.17.0.5:2380, https://172.17.0.5:2379, false
```

5. 删除failed etcd 成员

```
READY_ETCD_MEMBER="<name of etcd member from previous steps>"
ETCD_ID_TO_REMOVE="<etcd member ID from previous step>"
ETCDCTL="ETCDCTL_API=3 etcdctl --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints=https://127.0.0.1:2379"
kubectl -n kube-system exec -it "$READY_ETCD_MEMBER" -- /bin/sh -c "$ETCDCTL member remove $ETCD_ID_TO_REMOVE"
--------------------
Member 1d021ffdd096a804 removed from cluster a6ea9ad1b116d02f
```

6. 创建新的 control plan node 来替代 failed node

```
konvoy up
```



### 替换 failed node

使用`kubectl`命令确认节点可访问

```
kubectl get nodes
```

如果failed node 是Ready的，应该被drained。

```
kubectl drain <worker-node-id>
```

可能会出现类似错误

```
$ kubectl drain ip-10-0-128-170.us-west-2.compute.internal

node/ip-10-0-128-170.us-west-2.compute.internal cordoned
error: unable to drain node "ip-10-0-128-170.us-west-2.compute.internal", aborting command...

There are pending nodes to be drained:
 ip-10-0-128-170.us-west-2.compute.internal
cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-system/calico-node-8j5qz, kube-system/ebs-csi-node-ft6b6, kube-system/kube-proxy-9vpxs, kubeaddons/fluentbit-kubeaddons-fluent-bit-k5q7f, kubeaddons/prometheus-kubeaddons-prometheus-node-exporter-xbpsn
cannot delete Pods with local storage (use --delete-local-data to override): kubeaddons/alertmanager-prometheus-kubeaddons-prom-alertmanager-0, kubeaddons/prometheusadapter-kubeaddons-prometheus-adapter-6b8975fc48d6h2b, velero/velero-kubeaddons-5d85fcdcb9-762ll
```

可以增加 flags

```
kubectl drain ip-10-0-128-170.us-west-2.compute.internal --ignore-daemonsets --delete-local-data
```

如果 node 不是`Ready` 状态，你可以跳过 draining process



#### cloud provider

**AWS**

可以删除实例，可以执行

```
konvoy up
```



**On-Prem**

在inventory.yaml 删除节点IP，执行

```
konvoy up
```



### 生成 diagnostic bundle

1. 进入包含 Konvoy cluster 的 state file 的目录。
2. 执行命令，生成 diagnostic 信息的压缩包 

```
konvoy diagnose
```

3. 生成的压缩包

```
ls
20190705T114114.tar.gz
```

4. 解压后，类似于

```
$ tar -xvf 20190705T114114.tar.gz
bundles/master_172.17.0.3.tar.gz
bundles/worker_172.17.0.2.tar.gz
bundles/cluster-data.tar.gz
bundles/helm.tar.gz
...
```



```
$ tar -xvf bundles/master_172.17.0.3.tar.gz
iptables-save.txt
ctr-version.txt
containerd.service.log
containerd.service.status.txt
journalctl.log
kubelet.service.log
ansible_facts.json
timedatectl.txt
kubelet.service.status.txt
dmesg.txt
```



### SSH 连接错误

#### Too many authentication failures

```
STAGE [Running Preflights]

PLAY [Configure Cluster Prerequisites] ****************************************************************

TASK [all : wait 180 seconds for target connection to become reachable] *******************************

fatal: [10.0.129.25]: FAILED! => {
   "changed": false,
   "elapsed": 180
}

MSG:

timed out waiting for ping module test success: EOF on stream; last 300 bytes received: 'Received disconnect from 54.218.158.21 port 22:2: Too many authentication failures\r\n'
```



确认私钥已经在 SSH agent

```
ssh-add -L
```



reset SSH agent，增加私钥

```
ssh-add -D
ssh-add <PRIVATE_KEY>
```



### 更改 Node IP

Cloud provider

Konvoy 使用 public IP 来连接 nodes. konvoy 设定` ansible_host` 在 `inventory.yaml`.

更新 `inventory.yaml` 中的 IP 信息，需要运行 `konvoy up` 或者 `konvoy provision`.



On-Premises

更新`inventory.yaml` 文件。



### Troubleshooting Technique

Kubernetes 官方文档提供了大量有价值的分析问题和验证方案

- [Troubleshooting Overview](https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/)
- [Troubleshooting your Cluster](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)
- [Troubleshooting your Application](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)
- [Determine Reason for a Pod Failure](https://kubernetes.io/docs/tasks/debug-application-cluster/determine-reason-pod-failure/)
- [Debug Pods](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/)
- [Debug Services](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)
- [Debugging Nodes with `crictl`](https://kubernetes.io/docs/tasks/debug-application-cluster/crictl/)
- [Getting a Shell into a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)



#### 探寻Kubernetes 的errors信息

##### 识别错误类型

网络错误

| Symptom                                          | Diagnosis                                                    |
| ------------------------------------------------ | ------------------------------------------------------------ |
| Pods 启动，但是无法访问其他的Pods                | 使用 `kubectl logs`  检查 Pods的日志。                       |
| Pods 启动失败，处于Pending状态 或 unscheduleable | 使用 `kubectl describe pod`检查Kubernetes 状态，review Events section。 |
| Services 集群内部或者外部服务不可达              | 使用`kubectl describe service` 检查Kubernetes 的状态。service `selector` 的错误配置会导致服务成功注册，但是没有 pods 被选择作为后端的endpoints。`kubectl describe service <service_name>` 将显示该service 没有相关的endpoints。 |

存储错误

| Symptom                                          | Diagnosis                                                    |
| ------------------------------------------------ | ------------------------------------------------------------ |
| Pods 启动失败，处于Pending状态 或 unscheduleable | 使用 `kubectl describe pod`检查Kubernetes 状态，review Events section。 |
| PersistentVolumeClaims being unresolved.         | 检查 Kubernetes 相关 objects 的状态，`kubectl get sc,pv,pvc`; 可以在Pod’s Events section 看到指示信息。 |
| 应用超过了分配的存储                             | 执行`kubectl logs`，检查log。                                |

部署配置错误

| Symptom                                          | Diagnosis                                                    | Notes                                                        |
| ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Pods 启动失败，处于Pending状态 或 unscheduleable | 使用 `kubectl describe pod`检查Kubernetes 状态，review Events section。 | 通常是由于错误配置的 storage volumes 导致的, [configuration maps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/), or [secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/). |



####  Konvoy 诊断

##### 确认系统先决条件

 ```
konvoy check preflight
 ```

验证的内容包括

* 机器 SSH 可达，合适的 SSH 配置
* 已安装要求的 Python 版本
* 操作系统与平台匹配
* 节点可以访问 Docker registries 并获取需要的 images
* 操作系统的 Package registries 可达
* Swap禁用
* 兼容的网络配置（例如，避免子网配重叠）
* 节点可以在网络中互访
* 需要的网络端口是可用的

推荐执行 preflight checks 的情况：

* Kubernetes 集群 node 不可达或无响应
* Applications 失去联系
* 操作系统升级，系统配置，或网络维护可能导致服务中断

##### 验证节点健康

```
konvoy check nodes
```

* Kernel 配置是有效的，特别是 container networking
* 核心 Kubernetes 组件健康运行
* 核心系统命令可用

推荐执行节点健康检查的情况：

* Kubernets 报告某个节点不健康
* Pods 被 scheduled 到1个节点但是没有启动
* Pods 运行但是在集群内的网络中无法被访问到

注意：一些 Pod 的运行错误可以通过浏览 [local node debugging with crictl](https://kubernetes.io/docs/tasks/debug-application-cluster/crictl/).



##### 验证 Kubernetes 集群健康

```
konvoy check kubernetes
```

检查包括

* 容器网络 (calico) 健康运行在所有的节点
* 集群中所有的节点都健康
* control plane 组件版本正确

推荐执行集群健康检查的情况：

* API server 出现down，无法访问，或没有响应的情况
* 部署失败：Pods 和其它资源无法通过 API 创建
* 部署成功，但是未达到预期结果
  * 例如：Service 创建了，但是External IP 没有报给 service
* 集群的 Applicatons 无法访问其它services
* 在所有集群升级后



##### 验证安装的 addons

```
konvoy check addons
```

* 检查所有启用的 addons 都安装和运行
* 部署的 addons 和 `cluster.yaml` 文件中的配置保持一致

 推荐执行 addon 健康检查的情况：

* Konvoy addon 不可达或者无响应。

* logging 和 metrics 数据在 Kibana 和 Grafana 没有更新
* 通过 ingress 无法访问部署的applications

注意: Konvoy 中，addons 跟 applications 类似，深入研究可以查看 [the Kubernetes application troubleshooting guide](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/).

