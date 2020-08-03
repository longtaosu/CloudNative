# 概念

RC/RS是Kubernetes的一个核心概念，用于保证应用能够持续稳定的运行，主要功能如下：

- 保证Pod数量：确保Kubernetes中有指定数量的Pod在运行，如果少于指定数量的Pod，RC就会创建新的。反之会删除多余的，保证Pod的副本数量不变。
- 确保Pod健康：当Pod不健康（无法提供正常服务时），RC也会杀死不健康的Pod，重新创建新的
- 弹性伸缩：在业务高峰或者低峰时，通过RC来动态的调整Pod数量来提供资源的利用率，当然也可以通过HPA这种资源对象方式实现自动伸缩。
- 滚动升级：滚动升级是一种平滑的升级方式，通过逐步替换的策略，保证整体系统的稳定性。

Deployment同样也是Kubernetes系统的一个核心概念，职责和RC一样都是保证Pod的数量和健康，二者大部分功能完全一致，其新特性如下：

- RC的全部功能：Deployment具备上面描述的RC的全部功能。
- 事件和状态查看：可以查看Deployment的升级详细进度和状态
- 回滚：当升级Pod的时候如果出现问题，可以使用回滚操作回滚到之前的任一版本
- 版本记录：每一次对Deployment的操作，都能够保存下来，这也是保证可以回滚到任一版本的基础
- 暂停和启动：对于每一次升级都能够随时暂停和启动

作为对比，**Deployment作为新一代的RC，不仅功能更为丰富，也是官方的推荐管理Pod的方式**。官方组件如kube-dns、kube-proxy都使用Deployment来管理。

# 创建Deployment

创建一个Deployment，它创建了一个Replica Set来启动3个Nginx pod，yaml文件如下：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    k8s-app: nginx-demo
spec:
  replicas: 3
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
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

将上面的内容保存为：deployment-nginx.yaml文件，执行命令：

```shell
kubectl create -f deployment-nginx.yaml
```

![](03images-Deployment\01deployment-create.png)

查看刚刚创建的Deployment

```shell
kubectl get deployments
```

![](03images-Deployment\02deployment-get.png)

查看Replica Set

可以看到已经创建了1个Replica Set

```shell
//查看rs
kubectl get rs
//查看pod
kubectl get pod --show-labels
```

![](03images-Deployment\03deployment-rs.png)

![](03images-Deployment\04deployment-pod.png)

上面的yaml文件中的replicas：3将会保证我们始终有3个Pod在运行。



# 滚动升级

将yaml文件中的nginx镜像改为nginx:1.13.3，然后再spec下面添加滚动升级策略：

```yaml
    spec:
      containers:
      - name: nginx
        image: nginx:1.13.3
        ports:
        - containerPort: 80
      minReadySeconds: 5
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
```

- **minReadySeconds**

1. Kubernetes在等待设置的时间后才进行升级
2. 如果没有设置该值，Kubernetes会假设该容器启动起来后就提供服务了
3. 如果没有设置该值，在某些极端情况下可能会造成服务不正常运行

- **maxSurge**

1. 升级过程中最多可以比原先设置多出的Pod数量
2. 例如：maxSurage=1，replicas=5，则表示Kubernetes会先启动1个新的Pod后才删除1个旧的Pod，整个升级过程中最多会有5+1个Pod

- **maxUnavaible**

1. 升级过程中最多有多少个Pod处于无法提供服务的状态
2. 当maxSurge不为0时，该值也不能为0
3. 例如：maxUnavaible=1，则表示Kubernetes整个升级过程中最多会有一个Pod处于无法服务的状态

执行命令

```shell
kubectl apply -f deployment-nginx.yaml
```

![](03images-Deployment\05deployment-update.png)

查看升级状态

```shell
kubectl rollout status deployment/nginx-deploy
```

暂停升级

```shell
kubectl rollout pause deployment <deployment>
```

继续升级

```shell
kubectl rollout resume deployment <deployment>
```

升级后，查看rs状态

```shell
kubectl get rs
```



# 回滚

查看Deployment升级历史

```shell
kubectl rollout history deployment nginx-deploy
```

![](03images-Deployment\06deployment-history.png)

回退到前一个版本

```shell
kubectl rollout undo deployment nginx-deploy
```

通过revision回退到指定版本

```shell
kubectl rollout undo deployment nginx-deploy --to-revision=2
```

