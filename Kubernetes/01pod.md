# 创建Pod

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: kube100-site
  labels:
    app: web
spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
    - name: flaskapp-demo
      image: jcdemo/flaskapp
      ports:
        - containerPort: 5000
```

说明

- apiVersion：这个值需要根据安装的kubernetes版本和资源类型进行变化，并不是写死的

- kind：可以使Pod、Deployment、Job、Ingress、Service
- metadata：包含了Pod的一些meta信息，比如名称、namespace、标签等
- spec：包括一些containers、storage、volumes等参数



# 定义容器

```yaml
…spec:
  containers:
    - name: front-end
      image: nginx
      ports:
        - containerPort: 80
…
```

该容器基于nginx镜像，容器会监听端口80.

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





https://www.qikqiak.com/k8s-book/docs/18.YAML%20%E6%96%87%E4%BB%B6.html