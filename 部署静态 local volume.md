# 部署 staitc local volume

`localvolumeprovisioner `  addon 使用[local volume static provisioner](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner) 来为预分配的disk管理 persistent volume。通过观察每个 host `mnt/disks` 的目录，并在`localvolumeprovisioner` storage class 中创建persistent volume。

* mount 在 `/mnt/disks`下的 'Filesystem' volume-mode persistent volume 会被发现。
*  mount 在 `/mnt/disks`下的 'Block' volume-mode persistent volume 会被发现

部署 cluster 和 volume

1. 部署 kubernetes cluster

```
konvoy provision
```

2. 部署 local volume provisioner，查看 每个 host 的`/mnt/disks`

例如，使用ansible 来部署

```
ansible -i inventory.yaml node -m shell -a "mkdir -p /mnt/disks/example-volume && mount -t tmpfs example-volume /mnt/disks/example-volume"
```

3. 部署 Konvoy 集群

```
konvoy up
```

4. 验证 persistent volume

```
kubectl get pv

NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS             REASON   AGE
local-pv-4c7fc8ba   3986Mi     RWO            Delete           Available           localvolumeprovisioner            2s
```

5. 使用 PersistentVolumeClaim 来claim persistent volume

```
cat <<EOF | kubectl create -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: example-claim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: localvolumeprovisioner
EOF
```

6. 在 pod 中使用 persistent volume claim

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-persistent-volume
spec:
  containers:
    - name: frontend
      image: nginx
      volumeMounts:
        - name: data
          mountPath: "/var/www/html"
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: example-claim
EOF
```

7. 验证 persistent volume claim

```
kubectl get pvc

NAME            STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS             AGE
example-claim   Bound    local-pv-4c7fc8ba   3986Mi     RWO            localvolumeprovisioner   78s
$ kubectl get pv
NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS             REASON   AGE
local-pv-4c7fc8ba   3986Mi     RWO            Delete           Bound       default/example-claim   localvolumeprovisioner            15m

```

