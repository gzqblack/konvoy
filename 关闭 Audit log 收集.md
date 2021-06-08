# 关闭 Audit log collection

Audit logs 是借助于 FluentBit Addon 来实现的，默认是开启收集的。有些情况下是没有必要的，接下来的流程来展示如何关闭 Audit Log 收集。

## Disable Audit Log Collection !

1. 更新 `cluster.yaml` 

```
---
kind: ClusterConfiguration
apiVersion: konvoy.mesosphere.io/v1beta1
metadata:
name: my-cluster
creationTimestamp: "2020-06-05T16:57:18Z"
spec:
kubernetes:
    version: 1.16.8
…
addons:
- configRepository: https://github.com/mesosphere/kubernetes-base-addons
    …
    addonsList:
    …
    - name: fluentbit
      enabled: true
      values: |
        audit:
          enabled: false
    …

```

如果你想删除 Kibana 的 Audit Log Dashboard ，你可以 update `cluster.yaml`

```
…
addons:
- configRepository: https://github.com/mesosphere/kubernetes-base-addons
    …
    addonsList:
    …
    - name: kibana
      enabled: true
      values: |
        dashboardImport:
          enabled: false
    …
...
```



2. 执行命令

```
konvoy up -y
```

