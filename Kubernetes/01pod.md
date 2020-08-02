> Kubernetes可以创建Pod、Deployment、Job、Ingress和Service。



# 创建Pod

```yaml
…spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
…
```

**说明**

- apiVersion：这个值需要根据安装的kubernetes版本和资源类型进行变化，并不是写死的；

- kind：可以使Pod、Deployment、Job、Ingress、Service等；
- metadata：包含了Pod的一些meta信息，比如名称、namespace、标签等；
- spec：包括一些containers、storage、volumes等参数。

该容器基于nginx镜像，容器会监听端口80。

常用属性：

- name
- image
- command
- args
- workingDir
- ports
- env
- resources
- volumeMounts
- livenessProbe
- readinessProbe
- livecycle
- terminationMessagePath
- imagePullPolicy
- securityContext
- stdin
- stdinOnce
- tty

## create

完成Yaml文件的编辑后，使用kubectl创建Pod

```shell
kubecetl create -f deploy-nginx.yml
```

![](01images-pod\01kubectl-create.png)

查看Pod状态

```shell
kubectl get pods
```

![](01images-pod\02kubectl-getpods.png)

## describe

可以使用kubectl describe命令进行调试

```shell
kubectl describe pod pod-test
```

![](01images-pod\03kubectl-describe.png)

## delete

使用delete命令可以删除Pod（yaml文件并不会被删除）

```shell
kubectl delete -f deploy-nginx.yaml
//也可以通过pod的name删除
kubectl delete pod pod-test
```

![](01images-pod\04kubectl-delete.png)

![](01images-pod\05kubectl-delete.png)

# 创建Deployment

刚才只是创建了一个Pod实例，如果这个Pod出现了故障，则整个服务会挂掉，所以K8S提供了一个`Deployment`的概念，可以让K8S去管理一组Pod的副本。

**Deployment可以保证一定数量的副本一直可用，不会因为一个Pod挂掉导致整个服务挂掉。**

1. 将kind从Pod修改为Deployment
2. replicas：2，创建2个副本

```yaml
apiVersion: apps/v1

kind: Deployment
metadata: 
  name: deployment-test
  namespace: aspnetcore
  labels:
    name: deployment-test
spec:
  replicas: 2
  selector:
    matchLabels:
      name: deployment-test
  template:
    metadata:
      labels:
        name: deployment-test
    spec:
      containers: 
        - name: front-end
          image: nginx
          ports:
            - containerPort: 80
```

![](01images-pod\06Deployment.png)

同样的通过命令create运行

```shell
kubectl create -f deploy-nginx.yaml
```

此时通过 kubectl get deployment命令查看到运行状态

```shell
kubectl get deployments
```

可以通过dashboard查看

![](01images-pod\07Deployment-dashboard.png)







# 参考

[https://www.qikqiak.com/k8s-book/docs/18.YAML%20%E6%96%87%E4%BB%B6.html](https://www.qikqiak.com/k8s-book/docs/18.YAML 文件.html)