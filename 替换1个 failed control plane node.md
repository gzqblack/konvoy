# 替换 1个 failed control node

1. 在 AWS 的控制台结束 failed control plane node
2. Exec 进入工作的 control plane node

```
kubectl exec -ti -n kube-system etcd-ip-10-0-193-25.us-west-2.compute.internal sh
```

3. 通过`control plane shell` 运行如下命令，列出所有的 etcd member，注意 `member id ` ，本例子中的failed etcd 是 `adcebc0f12cd5b7f`

```
ETCDCTL_API=3 etcdctl --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints=https://127.0.0.1:2379 member list

61a7a86f030d44b6, started, ip-10-0-197-223.us-west-2.compute.internal, https://10.0.197.223:2380, https://10.0.197.223:2379
adcebc0f12cd5b7f, started, ip-10-0-200-149.us-west-2.compute.internal, https://10.0.200.149:2380, https://10.0.200.149:2379
fce222be536f51c5, started, ip-10-0-193-25.us-west-2.compute.internal, https://10.0.193.25:2380, https://10.0.193.25:2379
```

4. 通过`control plane shell ` 删除failed member

```
ETCDCTL_API=3 etcdctl --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --cacert=/etc/kubernetes/pki/etcd/ca.crt --endpoints=https://127.0.0.1:2379 member remove adcebc0f12cd5b7f

Member adcebc0f12cd5b7f removed from cluster 7631c48ef36ea956
```

5. 推出 `control plane shell`, 执行如下命令，增加 control plane node

```
konvoy up
```

