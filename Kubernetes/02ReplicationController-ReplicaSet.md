Kubernetes提供的资源对象

- Replication Controller：用来部署、升级Pod
- Replica Set：下一代的Replication Controller
- Deployment：可以更加方便的管理Pod和Replica Set



# Replication Controller

Replication Controller简称RC，是Kubernetes系统中的核心概念之一。

RC可以保证在任意时间运行运行的`Pod`的副本数量，能够保证`Pod`总是可用的。如果实际Pod数量比指定的多那就结束掉多余的，如果实际数量比指定的少就启动一些Pod。当Pod失败、被删除或者挂掉后，RC都会去自动创建新的Pod来保证副本数量，所以即使只有一个Pod，我们也应该使用RC来管理我们的Pod。

使用RC来管理使用的Nginx的Pod，Yaml文件如下：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-demo
  labels:
    name: rc
spec:
  replicas: 3
  selector:
    name: rc
  template:
    metadata:
     labels:
       name: rc
    spec:
     containers:
     - name: nginx-demo
       image: nginx
       ports:
       - containerPort: 80
```

- Kind：ReplicationController
- spec.replicas：指定Pod副本数量，默认为1
- spec.selector：RC通过该属性来筛选要控制的Pod
- spec.template：这里的定义和之前Pod定义一样，不需要`aspVersion`和`kind`。
- spec.template.metadata.labels：这里的`Pod`的`labels`要和`spec.selector`相同，这样RC就可以来控制当前这个`Pod`了。

这个Yaml文件定义了一个RC资源对象，名字叫做rc-demo，保证一直会有3个Pod运行，Pod的镜像是nginx镜像。

> 注意spec.selector和spec.template.metadata.labels这两个字段必须相同，否则会创建失败。
>
> 可以不写spec.selector，这样就默认与Pod模板中的metadata.labels相同。

创建RC对象（保存为replication-nginx.yaml）

```shell
kubectl create -f replication-nginx.yaml
```

![](02images-ReplicationController-ReplicaSet\01rc-create.png)

查看RC

```shell
kubectl get rc
```

![](02images-ReplicationController-ReplicaSet\02rc-get.png)

查看具体信息

```shell
kubectl describe rc rc-demo
```

![](02images-ReplicationController-ReplicaSet\03rc-describe.png)

修改Yaml文件，将Pod的副本数改为2：

```shell
kubectl apply -f replication-nginx.yaml
```

![](02images-ReplicationController-ReplicaSet\04rc-replicas.png)

使用RC来进行滚动升级，将镜像地址更改为`nginx:1.7.9`

```shell
kubectl set image rc rc-demo nginx-demo=nginx:1.7.9
```

> rc-demo是replicationControllers的名字
>
> nginx-demo是container的名字
>
> nginx:1.7.9是指定新的镜像版本

![](02images-ReplicationController-ReplicaSet\05rc-updateimage.png)

如果Pod中多个容器的话，就需要通过Yaml文件进行修改

```shell
kubectl apply -f replication-nginx.yaml
```



# Replication Set

Replication Set简称RS，随着Kubernetes的高速发展，官方推荐使用RS和Deployment来代替RC，而RS和RC功能基本一致，**区别在于RC只支持基于等式的selector（env=dev或者environment!=qa），但是RS还支持基于集合的selector**，这对于复杂的管理很方便。

RS被Deployment这个更加高层的资源对象使用，一般情况下推荐使用Deployment而不是直接使用RS。

RS/RC总结：

- 大部分情况下，可以通过定义一个RC实现Pod的创建和副本数量的控制
- RC中包含一个完整的Pod定义模块（不包含apiversion和kind）
- RC通过label selector机制来实现对Pod副本的控制
- 通过改变RC里面Pod副本数量，可以实现Pod的扩缩容功能
- 通过改变RC里面的Pod模板中的镜像版本，可以实现Pod的滚动升级功能（但是不支持一键回滚）