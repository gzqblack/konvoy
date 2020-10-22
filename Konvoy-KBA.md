# Kubernetes Base Addons

## KBA

KBA 提供与配置的day2 的 功能和服务，在 K8s 之上，例如 monitoring，logging 和 外部 DNS。KBA 2个月会更新一次。

新的Minor 或者 Major 版本是基于用户的需求。KBA 有自己的 release number，例如：`release-<kubernetes version>-<major>.<minor>`



## KBA 需求

| **name of Addon**              | **Description**                                              | **Default Minimum Suggested** | **Default On When konvoy init** |
| ------------------------------ | ------------------------------------------------------------ | ----------------------------- | ------------------------------- |
| awsebscsiprovisioner           | Supports persistent volumes on AWS                           |                               | Yes                             |
| awsebsprovisioner              | Legacy “in-tree” volume provisioner                          |                               | No                              |
| azuredisk-csi-driver           | Supports persistent volumes on Azure                         | cpu: 10mmemory: 20Mi          | No                              |
| azurediskprovisioner           | Legacy volume provisioner                                    |                               | No                              |
| cert-manager                   | Automates the management and issuance of TLS certificates from various issuing sources. It will ensure certificates are valid and up to date periodically, and attempt to renew certificates at an appropriate time before expiry. It has ACME integration which would allow users to get a Let’s Encrypt certificate automaticallyand then talk to Let’s Encrypt server to get a valid certificate. | cpu: 10mmemory: 32Mi          | Yes                             |
| dashboard                      | Provides a general-purpose web-based user interface for the Kubernetes cluster | cpu: 250mmemory: 300Mi        | Yes                             |
| defaultstorageclass-protection | Ensures that there is 1 default storage class (i.e. something that would provide a volume) |                               | Yes                             |
| dex                            | Provides identity service (authentication) to the Kubernetes clusters | cpu: 100mmemory: 50Mi         | Yes                             |
| dex-k8s-authenticator          | Enables authentication flow to obtain `kubectl` token for accessing the cluster. | cpu: 100mmemory: 128Mi        | Yes                             |
| elasticsearch                  | Enables scalable, high-performance logging pipeline          | cpu: 100mmemory: 1536Mi       | Yes                             |
| elasticsearch-curator          | Helps curate, or manage, your Elasticsearch indices and snapshots by obtaining the full list of indices (or snapshots) from the cluster, as the actionable list; iterate through a list of user-defined filters to progressively remove indices (or snapshots) from this actionable list as needed; and perform various actions on the items which remain in the actionable list. | cpu: 100mmemory: 128Mi        | Yes                             |
| elasticsearchexporter          | The purpose of exporters is to take data collected from any Elastic Stack source and route it to the monitoring cluster | cpu: 100mmemory: 128Mi        | Yes                             |
| external-dns                   | Makes Kubernetes resources discoverable via public DNS servers; retrieves a list of resources (Services, Ingresses, etc.) from the Kubernetes API to determine a desired list of DNS records. It's not a DNS server itself, but merely configures other DNS providers accordingly. | cpu: 10mmemory: 50Mi          | Yes                             |
| flagger                        | Automates the release process for applications running on Kubernetes | cpu: 10mmemory: 32Mi          | No                              |
| fluentbit                      | Collects and collates logs from different sources and send logged messages to multiple destinations | cpu: 200mmemory: 200Mi        | Yes                             |
| gatekeeper                     | Policy controller for Kubernetes, allowing organizations to enforce configurable policies using the Open Policy Agent, a policy engine for Cloud Native environments hosted by CNCF as an incubation-level project. | cpu: 200mmemory: 300Mi        | Yes                             |
| istio                          | Helps you manage cloud-based deployments by providing an open-source service mesh to connect, secure, control, and observe microservices. | cpu: 10mmemory: 50Mi          | No                              |
| kibana                         | Supports data visualization for content indexed by Elasticsearch | cpu: 100m                     | Yes                             |
| konvoyconfig                   | Manages installation related configuration                   |                               | Yes                             |
| kube-oidc-proxy                | Reverse proxy to authenticate to managed Kubernetes API servers via OIDC |                               | Yes                             |
| localvolumeprovisioner         | Uses the local volume static provisioner to manage persistent volumes for pre-allocated disks. It does this by watching the /mnt/disks folder on each host and creating persistent volumes in the localvolumeprovisioner storage class for each disk that is discovered in this folder. |                               | No                              |
| nvidia                         | Enables deployment of NVIDIA GPU clusters                    | cpu: 100mmemory: 128Mi        | No                              |
| opsportal                      | Centralizes access to addon dashboards                       | cpu: 100mmemory: 128Mi        | Yes                             |
| prometheus                     | Collects and evaluates metrics for monitoring and alerting   | cpu: 300mmemory: 1500Mi       | Yes                             |
| prometheusadapter              | Gathers the names of available metrics from Prometheus at a regular interval, and then only exposes metrics that follow specific forms. | cpu: 1000mmemory: 1000Mi      | Yes                             |
| reloader                       | Watches changes in `ConfigMap` and `Secret` and do rolling upgrades on Pods with their associated `DeploymentConfigs`, `Deployments`, `Daemonsets` and `Statefulsets` | cpu: 100mmemory: 128Mi        | Yes                             |
| traefik                        | Routes layer 7 traffic as a reverse proxy and load balancer. | cpu: 500m                     | Yes                             |
| traefik-forward-auth           | Provides basic authorization for Traefik ingress             | cpu: 100mmemory: 128Mi        | Yes                             |
| velero                         | Backs up and restores Kubernetes cluster resources and persistent volumes. | cpu: 250mmemory: 256Mi        | Yes                             |
| dispatch                       | D2iQ’s cloud-native GitOps platform                          | cpu: 250mmemory: 256Mi        | No                              |
| kommander                      | D2iQ's administrative cluster for multi-cluster management of Kubernetes lifecycle, governance, and workloads | cpu: 100mmemory: 256Mi        | Yes                             |



## Addon Dependencies

| **Addon**                      | **Dependencies**                                             |
| ------------------------------ | ------------------------------------------------------------ |
| awsebscsiprovisioner           | `kubeaddons.mesosphere.io/name: defaultstorageclass-protection` |
| awsebsprovisioner              | `kubeaddons.mesosphere.io/name: defaultstorageclass-protection` |
| defaultstorageclass-protection | `kubeaddons.mesosphere.io/name: cert-manager`                |
| dex                            | `kubeaddons.mesosphere.io/provides: ingresscontroller`       |
| dex-k8s-authenticator          | `kubeaddons.mesosphere.io/name: dex` `kubeaddons.mesosphere.io/provides: ingresscontroller` |
| elasticsearch-curator          | `kubeaddons.mesosphere.io/name: elasticsearch`               |
| elasticsearchexporter          | `kubeaddons.mesosphere.io/name: elasticsearch`               |
| flagger                        | `kubeaddons.mesosphere.io/name: istio`                       |
| fluentbit                      | `kubeaddons.mesosphere.io/name: elasticsearch`               |
| gatekeeper                     | `kubeaddons.mesosphere.io/name: cert-manager`                |
| gcpdisk-csi-driver             | `kubeaddons.mesosphere.io/name: defaultstorageclass-protection` |
| gcpdiskprovisioner             | `kubeaddons.mesosphere.io/name: defaultstorageclass-protection` `kubeaddons.mesosphere.io/name: gcpdisk-csi-driver` |
| istio                          | `kubeaddons.mesosphere.io/name: cert-manager`                |
| kibana                         | `kubeaddons.mesosphere.io/name: elasticsearch`               |
| kube-oidc-proxy                | `kubeaddons.mesosphere.io/provides: ingresscontroller` `kubeaddons.mesosphere.io/name: cert-manager<br/>kubeaddons.mesosphere.io/name: dex` |
| localvolumeprovisioner         | `kubeaddons.mesosphere.io/name: defaultstorageclass-protection` |
| prometheusadapter              | `kubeaddons.mesosphere.io/name: prometheus`                  |
| traefik                        | `kubeaddons.mesosphere.io/name: cert-manager`                |
| traefik-forward-auth           | `kubeaddons.mesosphere.io/name: dex` `kubeaddons.mesosphere.io/provides: ingresscontroller` |
| velero                         | `kubeaddons.mesosphere.io/provides: ingresscontroller`       |



| **`ubeaddons.mesosphere.io/provides` value** | **Addon(s)**                                                 |
| -------------------------------------------- | ------------------------------------------------------------ |
| `storageclass`                               | awsebsprovisioner azurediskprovisioner awsebscsiprovisioner gcpdiskprovisioner localvolumeprovisioner |
| `nvidia`                                     | nvidia                                                       |
| `csi-driver`                                 | azuredisk-csi-driver gcpdisk-csi-driver                      |
| `loadbalancer`                               | metallb                                                      |
| `ingresscontroller`                          | traefik                                                      |



## Remove Helm V2

从 Konvoy 1.6.0 开始，需要删除 helm version 2，使用 heml version3

删除 Tiller 需要如下的命令：

```
kubectl -n kube-system delete deployment tiller-deploy
```

 