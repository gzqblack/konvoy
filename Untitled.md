# 使用 LoadBalancer service type

LoadBalancer server type 在 public cloud 中创建一个外部的 load balancer，分配一个固定的外部IP 给service.

用户可以通过暴露的 IP 地址来访问服务。

## 使用 LoadBalancer service 来暴露 Pod

1. 部署一个 Redis Pod

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: redis
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.3
    ports:
    - name: redis
      containerPort: 6379
      protocol: TCP
EOF
```

2. 使用 LoadBalancer type 创建 service

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis
  name: redis
spec:
  type: LoadBalancer
  selector:
    app: redis
  ports:
  - protocol: TCP
    port: 6379
    targetPort: 6379
EOF
```

3. 创建 service

```
kubectl get svc redis
```

```
NAME    TYPE           CLUSTER-IP   EXTERNAL-IP                                                               PORT(S)          AGE
redis   LoadBalancer   10.0.51.32   a92b6c9216ccc11e982140acb7ee21b7-1453813785.us-west-2.elb.amazonaws.com   6379:31423/TCP   43s
```

4. 验证通过 external IP 可以访问 Redis Pod

```
telnet a92b6c9216ccc11e982140acb7ee21b7-1453813785.us-west-2.elb.amazonaws.com 6379
Trying 52.27.218.48...
Connected to a92b6c9216ccc11e982140acb7ee21b7-1453813785.us-west-2.elb.amazonaws.com.
Escape character is '^]'.
quit
+OK
Connection closed by foreign host.
```

