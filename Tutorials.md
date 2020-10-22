# Addon Repositories

Konvoy 使用 cluster.yaml 配置 clusterProvisioner 和 ClusterConfiguration

## Addon Repository Structre

```
docs-addon-repo
   |- addons
   |     |- cockroachdb
   |           |- 19.2.x                     <- appVersion <major>.<minior>.x
   |                 |- cockroachdb-1.yaml   <- Addon yaml manifest, filename with revision

   |- metadata
   |     |- root.yaml                        <- Addon metadata
   |     |- static
   |           |- cockroachdb                <- folder for logo, overview documentation, ...
   |                 |- logo.svg
   |                 |- overview.md

   |- deployments
   |     |- 1.16                                   <- Kubernetes Version
   |           |- default-addons-deployments.yaml  <-  AddonsDeployment definition
   |- repository.yaml                        <- AddonRepository definition
   |- README.md

```

* addons/ 包含 addon 资源的 manifest
* metadata/ 包含 addon 资源的 静态 metadata
* deployments/ 包含默认的 addons 的具体K8s 版本信息



## 创建 Addon Repository 在 cluster.yaml

```
...
kind: ClusterConfiguration
apiVersion: konvoy.mesosphere.io/v1beta2
metadata:
  name: y-west
  ...
spec:
  ...
  addons:
  - configRepository: https://github.com/mesosphere/kubernetes-base-addons
    configVersion: stable-1.17-2.2.0
    addonsList:
    - name: awsebscsiprovisioner
      enabled: false
    ...
  - configRepository: https://github.com/mesosphere/docs-addon-repo
    configVersion: stable-0.1
    addonsList:
    - name: awsebscsiprovisioner2
      enabled: true
    - name: cockroachdb
      enabled: true
...
```

当执行 konvoy up，输出如下：

```
STAGE [Deploying Enabled Addons]
konvoyconfig                                                           [OK]
dashboard                                                              [OK]
reloader                                                               [OK]
fluentbit                                                              [OK]
external-dns                                                           [OK]
opsportal                                                              [OK]
cert-manager                                                           [OK]
defaultstorageclass-protection                                         [OK]
gatekeeper                                                             [OK]
awsebscsiprovisioner2                  <<<                             [OK]
traefik                                                                [OK]
prometheus                                                             [OK]
cockroachdb                                                            [OK]
dex                                                                    [OK]
velero                                                                 [OK]
prometheusadapter                                                      [OK]
kube-oidc-proxy                                                        [OK]
dex-k8s-authenticator                                                  [OK]
traefik-forward-auth                                                   [OK]
kommander                                                              [OK]
elasticsearch-curator                                                  [OK]
elasticsearch                                                          [OK]
elasticsearchexporter                                                  [OK]
kibana                                                                 [OK]

Kubernetes cluster and addons deployed successfully!
```



## Addons

### Storage Provider Addons

Kind 是 ClusterAddons, ChartReference 指向 helm chart 

```
---
apiVersion: kubeaddons.mesosphere.io/v1beta1
kind: ClusterAddon
metadata:
  name: awsebscsiprovisioner2
  labels:
    kubeaddons.mesosphere.io/name: awsebscsiprovisioner2
    kubeaddons.mesosphere.io/provides: storageclass
  annotations:
    catalog.kubeaddons.mesosphere.io/addon-revision: "0.4.0-1"
    appversion.kubeaddons.mesosphere.io/awsebscsiprovisioner: "0.4.0"
    values.chart.helm.kubeaddons.mesosphere.io/awsebscsiprovisioner: "https://raw.githubusercontent.com/mesosphere/charts/6c43b8ab10108fb1adba5c6dd10e800e5f1abdd0/stable/awsebscsiprovisioner/values.yaml"
spec:
  namespace: kube-system
  requires:
    - matchLabels:
        kubeaddons.mesosphere.io/name: defaultstorageclass-protection
  kubernetes:
    minSupportedVersion: v1.15.6
  cloudProvider:
    - name: aws
      enabled: true
  chartReference:
    chart: awsebscsiprovisioner
    repo: https://mesosphere.github.io/charts/stable
    version: 0.3.3
    values: |
      ---
      resizer:
        enabled: false
      snapshotter:
        enabled: true
      provisioner:
        enableVolumeScheduling: true
      storageclass:
        isDefault: true
```



##### metadata.labels

- `kubeaddons.mesosphere.io/name` - Addon name
- `kubeaddons.mesosphere.io/provides` - Addon functionality. For example, storageclass.

##### metadata.annotations

- `catalog.kubeaddons.mesosphere.io/addon-revision` - `appVersion-<revison>`
- `appversion.kubeaddons.mesosphere.io/awsebscsiprovisioner` - Helm chart `appVersion`
- `values.chart.helm.kubeaddons.mesosphere.io/awsebscsiprovisioner` - URI to helm chart `values.yaml` file

##### spec.requires[].matchLabels

- `kubeaddons.mesosphere.io/name: defaultstorageclass-protection` - Requires the `defaulstorageclass-protection` addon



### Workload Addons

kind 是 Addon, chartReference 指向 helm chart

```
---
apiVersion: kubeaddons.mesosphere.io/v1beta1
kind: Addon
metadata:
  name: cockroachdb
  namespace: default
  labels:
    kubeaddons.mesosphere.io/name: cockroachdb
    # TODO: we're temporarily supporting dependencies on an existing default storage class
    # on the cluster, this hack will trigger re-queue on Addons until one exists
    kubeaddons.mesosphere.io/hack-requires-defaultstorageclass: "true"
  annotations:
    catalog.kubeaddons.mesosphere.io/addon-revision: "19.2.2-1"
    appversion.kubeaddons.mesosphere.io/cockroachdb: "19.2.2"
    values.chart.helm.kubeaddons.mesosphere.io/cockroachdb: "https://raw.githubusercontent.com/helm/charts/dfea2ba119be53f0d3f7d70def66e54f9e259768/stable/cockroachdb/values.yaml"
spec:
  kubernetes:
    minSupportedVersion: v1.15.0
  cloudProvider:
    - name: aws
      enabled: true
    - name: azure
      enabled: true
    - name: docker
      enabled: false
    - name: none
      enabled: true
  chartReference:
    chart: stable/cockroachdb
    version: 3.0.2

```



##### metadata.labels

- `kubeaddons.mesosphere.io/name` - Addon name
- `kubeaddons.mesosphere.io/hack-requires-defaultstorageclass` - Set to `true` if this addon requires a `default StorageClass`

##### metadata.annotations

- `catalog.kubeaddons.mesosphere.io/addon-revision`- `appVersion-<revison>`
- `appversion.kubeaddons.mesosphere.io/awsebscsiprovisioner` - Helm chart `appVersion`
- `values.chart.helm.kubeaddons.mesosphere.io/awsebscsiprovisioner` - URI to helm chart `values.yaml` file