# Isito Canary deployment on Konvoy

## 构建 K8s 集群

1. Konvoy 集群安装完毕。

2. 因为konvoy 1.5.2 版本提供的 flagger 版本较低，可以不安装konvoy addons的 flagger，在cluster.yaml 设置为 false。

3. 安装Istio集群，在cluster.yaml 中设置为true。

4. 安装kubectl，确认与集群连接成功。

5. 安装helm client。

## 搭建 Istio ingress gateway

1. 找到 Istio  分配的 external IP

```
kubectl -n istio-system get svc|grep istio-ingressgateway
istio-ingressgateway        LoadBalancer   10.0.41.229   a699c30f48a314069858ff9706928463-1082224507.us-west-2.elb.amazonaws.com  15021:30663/TCP,80:30873/TCP,443:31938/TCP,15443:32730/TCP   41h
```

2. 注册两个域名`app.istio.example.com`, `grafana.istio.example.com` 解析至 external IP.

如果是内部测试，可以将解析写入`/etc/hosts`。

```
## istio testing
54.191.238.104 grafana.istio.example.com
54.191.238.104 app.istio.example.com
```

3. 创建一个通用Istio gateway，并向外网暴露HTTP服务

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: public-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "*"
```

将内容保存在gateway.yaml，然后执行

```
kubectl apply -f gateway.yaml
```



## 安装 Flagger

1. 添加 Flagger Helm仓库

```
helm repo add flagger https://flagger.app
```

2. 安装 Flagger canary CRD

```
kubectl apply -f https://raw.githubusercontent.com/weaveworks/flagger/master/artifacts/flagger/crd.yaml
```

3. 安装 Flagger for Istio

```
helm upgrade -i flagger flagger/flagger \
--namespace=istio-system \
--set crd.create=false \
--set meshProvider=istio \
--set metricsServer=http://<prometheus>:9090
## 注意 prometheus 的地址要填写正确
```

可以在任何 namespace 下安装 flagger，只要它可以访问 istio prometheus service 的 9090。Flagger 附带 Grafana dashbaord，用于金丝雀分析。在 istio-system namespace 下部署 grafana

```
helm upgrade -i flagger-grafana flagger/grafana \
--namespace=istio-system \
--set url=http://<prometheus.istio-system>:9090 \
--set user=admin \
--set password=change-me
## 注意 prometheus 的地址要填写正确
```

4. 创建 virtual service，用于暴露 Grafana

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: grafana
  namespace: istio-system
spec:
  hosts:
    - "grafana.istio.example.com"
  gateways:
    - public-gateway.istio-system.svc.cluster.local
  http:
    - route:
        - destination:
            host: flagger-grafana
```

将内容保存为 grafana-virtual-service.yaml, 然后执行

```
kubectl apply -f ./grafana-virtual-service.yaml
```

现在可以在浏览器直接访问`http://grafana.istio.example.com` 来查看Grafana的登陆页面。

## 使用 Flagger 部署 web 应用

Flagger包含一个Kubernetes deployment和一个可选的horizontal pod autoscaler（HPA），然后创建一些资源对象（Kubernetes deployments， ClusterIP services和Istio virtual services）。这些资源对象会在网络上暴露应用并实现金丝雀分析和升级。

![img](https://raw.githubusercontent.com/servicemesher/website/master/content/blog/automated-canary-deployments-with-flagger-and-istio/0071hauBly1g1u72wr801j30rs0cdq4w.jpg)

1. 创建 test namespace，并执行 istio sidecar injection

```
kubectl create ns test
kubectl label namespace test istio-injection=enabled
```

2. 创建 deployment 和 horizontal pod autoscaler

```
kubectl apply -k github.com/weaveworks/flagger//kustomize/podinfo
```

3, 部署 load testing service ，用于在 canary analysis 过程中产生流量

```
kubectl apply -k github.com/weaveworks/flagger//kustomize/tester
```

4. 创建一个金丝雀 custom resource （可以使用自己的域名替换`spec.service.hosts`）

```
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: podinfo
  namespace: test
spec:
  # deployment reference
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: podinfo
  # the maximum time in seconds for the canary deployment
  # to make progress before it is rollback (default 600s)
  progressDeadlineSeconds: 60
  # HPA reference (optional)
  autoscalerRef:
    apiVersion: autoscaling/v2beta1
    kind: HorizontalPodAutoscaler
    name: podinfo
  service:
    # service port number
    port: 9898
    # container port number or name (optional)
    targetPort: 9898
    # Istio gateways (optional)
    gateways:
    - public-gateway.istio-system.svc.cluster.local
    # Istio virtual service host names (optional)
    hosts:
    - app.istio.example.com
    # Istio traffic policy (optional)
    trafficPolicy:
      tls:
        # use ISTIO_MUTUAL when mTLS is enabled
        mode: DISABLE
    # Istio retry policy (optional)
    retries:
      attempts: 3
      perTryTimeout: 1s
      retryOn: "gateway-error,connect-failure,refused-stream"
  analysis:
    # schedule interval (default 60s)
    interval: 1m
    # max number of failed metric checks before rollback
    threshold: 5
    # max traffic percentage routed to canary
    # percentage (0-100)
    maxWeight: 50
    # canary increment step
    # percentage (0-100)
    stepWeight: 10
    metrics:
    - name: request-success-rate
      # minimum req success rate (non 5xx responses)
      # percentage (0-100)
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      # maximum req duration P99
      # milliseconds
      thresholdRange:
        max: 500
      interval: 30s
    # testing (optional)
    webhooks:
      - name: acceptance-test
        type: pre-rollout
        url: http://flagger-loadtester.test/
        timeout: 30s
        metadata:
          type: bash
          cmd: "curl -sd 'test' http://podinfo-canary:9898/token | grep token"
      - name: load-test
        url: http://flagger-loadtester.test/
        timeout: 5s
        metadata:
          cmd: "hey -z 1m -q 10 -c 2 http://podinfo-canary.test:9898/"
```

保存以上内容为`podinfo-canary.yaml` ，部署：

```
kubectl apply -f ./podinfo-canary.yaml
```

当 canary analysis 开始后，flagger 在路由 traffic 至 canary 之前调用 pre-rollout webhooks。 Canary analysis 将会运行 5 分钟来验证 HTTP metrics ，并每分钟 rollout hooks。

![img](https://raw.githubusercontent.com/weaveworks/flagger/master/docs/diagrams/flagger-canary-hpa.png)

几秒钟后，Flagger 将会创建 canary objects

```
# applied 
deployment.apps/podinfo
horizontalpodautoscaler.autoscaling/podinfo
canary.flagger.app/podinfo

# generated 
deployment.apps/podinfo-primary
horizontalpodautoscaler.autoscaling/podinfo-primary
service/podinfo
service/podinfo-canary
service/podinfo-primary
destinationrule.networking.istio.io/podinfo-canary
destinationrule.networking.istio.io/podinfo-primary
virtualservice.networking.istio.io/podinfo
```

打开浏览器访问`app.istio.example.com`, 可以看到 app 的 版本

## 自动金丝雀分析和升级

Flagger实现了一个控制循环，逐渐将流量转移到金丝雀，同时测量HTTP请求成功率等关键性能指标，请求平均持续时间以及pod健康状态。根据对KPI的分析，升级或中止金丝雀部署，也可以将分析结果发送到Slack。

![img](https://raw.githubusercontent.com/servicemesher/website/master/content/blog/automated-canary-deployments-with-flagger-and-istio/0071hauBgy1g1uawf9vhqj30rs0a976b.jpg)

以下对象的更改会触发金丝雀部署：

- Deployment PodSpec（容器image，command，ports，env等）
- ConfigMaps作为卷挂载或映射到环境变量
- Secrets作为卷挂载或映射到环境变量

1. 可以通过更新容器image 触发金丝雀部署

```
kubectl -n test set image deployment/podinfo \
podinfod=quay.io/stefanprodan/podinfo:1.4.1
```

2. Flagger检测到deployment的版本已更新，于是开始分析它：

```
kubectl -n test describe canary/podinfo

Events:

New revision detected podinfo.test
Scaling up podinfo.test
Waiting for podinfo.test rollout to finish: 0 of 1 updated replicas are available
Advance podinfo.test canary weight 5
Advance podinfo.test canary weight 10
Advance podinfo.test canary weight 15
Advance podinfo.test canary weight 20
Advance podinfo.test canary weight 25
Advance podinfo.test canary weight 30
Advance podinfo.test canary weight 35
Advance podinfo.test canary weight 40
Advance podinfo.test canary weight 45
Advance podinfo.test canary weight 50
Copying podinfo.test template spec to podinfo-primary.test
Waiting for podinfo-primary.test rollout to finish: 1 of 2 updated replicas are available
Promotion completed! Scaling down podinfo.test
```

3. 在分析过程中，可以使用 grafana 监控金丝雀的进展

![img](https://raw.githubusercontent.com/servicemesher/website/master/content/blog/automated-canary-deployments-with-flagger-and-istio/0071hauBly1g1ubacxpukj30rs0mhjvg.jpg)

4. 列出集群中所有的金丝雀：

```
NAMESPACE   NAME      STATUS      WEIGHT   LASTTRANSITIONTIME
test        podinfo   Promoting   0        2020-09-03T14:34:23Z
```

5. 如果启用了Slack 通知功能，则会收到以下消息：

![img](https://raw.githubusercontent.com/servicemesher/website/master/content/blog/automated-canary-deployments-with-flagger-and-istio/0071hauBly1g1ubdv033ej30rs0ap400.jpg)



## 自动回滚

在金丝雀分析期间，可以生成HTTP 500错误和高响应延迟，以测试Flagger是否暂停升级。

1. 触发另外一个 canary deployment



```
kubectl -n test set image deployment/podinfo \
podinfod=stefanprodan/podinfo:3.1.2
```

2. 在 load test pod 中执行

```
kubectl -n test exec -it flagger-loadtester-xx-xx sh
```

3. 生成 HTTP 500 errors

```
watch curl http://podinfo-canary:9898/status/500
```

4. 生成 latency

```
watch curl http://podinfo-canary:9898/delay/1
```

5. 当一定数量的 failed check 达到了 canary analysis 的 threshold，traffic 会重新路由只 primary，canary 会 scaled 至 0，rollout 呗标记为 failed

## 流量镜像

![img](https://raw.githubusercontent.com/weaveworks/flagger/master/docs/diagrams/flagger-canary-traffic-mirroring.png)

