# 简介

`Job`负责处理任务，即仅执行一次的任务，它保证批处理任务的一个或多个`Pod`成功结束。

`CronJob`则就是在`Job`上加上了时间调度。



# Job

使用Job来执行一个倒计时的任务，定义yaml文件：job-demo.yaml。

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-demo
spec:
  template:
    metadata:
      name: job-demo
    spec:
      restartPolicy: Never
      containers:
      - name: counter
        image: busybox
        command:
        - "bin/sh"
        - "-c"
        - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done"
```

> **注**
>
> Job的RestartPolicy仅支持 Nerver 和 OnFailure 两种，不支持Always。

创建该 Job

```shell
$ kubectl create -f job-demo.yaml
job.batch/job-demo created
```

查看当前 Job 资源对象

```shell
$ kubectl get jobs
```

![](05images-Job\01创建Job.png)

通过 kubectl logs查看日志

```shell
$ kubectl logs job-demo-qsphw
```

![](05images-Job\02定时任务.png)

# CronJob

CronJob是在Job的基础上加上了时间调度，我们可以在给定的时间运行一个任务，也可以周期性的在给定的时间点运行。

CronTab格式如下

| 分   | 时   | 日   | 月   | 星期                  | 命令         |
| ---- | ---- | ---- | ---- | --------------------- | ------------ |
| 0~59 | 0~23 | 1~31 | 1~12 | 0~7（0、7表示星期天） | 要运行的命令 |

创建CronJob

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cronjob-demo
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: hello
            image: busybox
            args:
            - "bin/sh"
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done"
```

创建这个cronjob

```shell
$ kubectl create -f cronjob-demo.yaml
cronjob.batch/cronjob-demo created
```

查看cronjob资源

```shell
$ kubectl get cronjob
```

![](05images-Job\03CronJob.png)