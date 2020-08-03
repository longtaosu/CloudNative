# 概念

我们可以通过手动执行kubectl scale命令和在Dashboard上操作实现Pod的扩缩容。

Kubernetes提供了这样一个资源对象：Horizontal  Pod Autoscaling（Pod水平自动伸缩），简称HPA。HPA通过监控分析RC或者Deployment控制的所有Pod的负载变化情况来确定是否需要调整Pod的副本数量，这是HPA最基本的原理。

创建了HPA后，HPA会从Heapster或者用户自定义的RestClient端获取每一个Pod利用率或原始值的平均值，然后和HPA中定义的指标进行对比，同时计算出需要伸缩的具体值并进行相应的操作。目前，HPA可以从两个地方获取数据：

- Heapster：仅支持CPU使用率
- 自定义监控：

可以前往Heapster的github页面

[https://github.com/kubernetes/heapster](https://github.com/kubernetes/heapster/tree/v1.4.2/deploy/kube-config/influxdb)

将该目录下的yaml文件保存到集群上，使用kubectl命令行工具创建。创建完成后，如果需要在Dashboard当中看到监控图表，还需要在Dashboard中配置heapsterhost。

# 创建HPA

创建一个Deployment管理的Nginx Pod，利用HPA来进行自动扩缩容。定义Deployment的YAML文件（hap-deploy-demo）如下：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-nginx-deploy
  labels:
    app: nginx-demo
spec:
  revisionHistoryLimit: 15
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

创建Deployment：

```shell
kubectl create -f hpa-deploy-demo.yaml
```

![](04images-HPA\01hpa-create.png)

创建一个HPA，使用kubectl autoscale来创建：

```shell
kubectl autoscale deployment hpa-nginx-deploy --cpu-percent=10 --min=1 --max=10
```

> 此命令创建了一个关联资源hpa-nginx-deploy的HPA，最小的Pod副本数为1，最大为10。HPA会根据设定的CPU使用率（10%）动态的增加或者减少pod的数量。

查看HPA

```shell
kubectl get hpa
```

![](04images-HPA\02hpa-autoscale.png)

除了kubectl autoscale命令，也可以通过Yaml文件创建HPA资源对象。可以通过命令获取创建的HPA的Yaml文件：

```shell
kubectl get hpa hpa-nginx-deploy -o yaml
```

![](04images-HPA\03hpa-yaml.png)



# 参考

[https://www.qikqiak.com/k8s-book/docs/25.Pod%20%E6%B0%B4%E5%B9%B3%E8%87%AA%E5%8A%A8%E4%BC%B8%E7%BC%A9.html](https://www.qikqiak.com/k8s-book/docs/25.Pod 水平自动伸缩.html)