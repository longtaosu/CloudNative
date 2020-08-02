DockerFile是一个文本文件，其中包含了一条条的指令（Instruction），每条命令构建一层，因此每条指令，就是描述该层如何构建。



# 指令

## FROM

指定基础镜像



## RUN

> 每一个run都会建立一层镜像。下面的例子，会将run命令当做一条命令，减少镜像构建的层数

```shell
RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && apt-get purge -y --aut-remove $buildDeps
```





# 构建镜像

在Dockerfile文件所在的目录下

```shell
docker build -t 仓库名[:标签] .
```



# 镜像仓库

## 拉取镜像

通过 `docker search` 命令查找官方仓库中的镜像

```shell
docker search centos
```

找到后通过pull命令拉取镜像

```shell
docker pull centos
```



## 推送镜像

为镜像添加tag（username需要换成自己Docker的账号）

```shell
docker tag ubuntu:17.10 username/ubuntu:17.10
```

推送镜像

```shell
docker push username/ubuntu:17.10
```



## 私有仓库

可以通过docker-registry单间私有仓库

```shell
docker run -d -p 5000:5000 --restart=always --name registry registry
```

为镜像添加tag

```shell
docker tag ubuntu:latest 127.0.0.1:5000/ubuntu:latest
```

推送镜像

```shell
docker push 127.0.0.1:5000/ubuntu:17.10
```



# 多阶段构建

在Docker 17.05版本之后，官方提供了一个新的特性：Multi-stage builds（多阶段构建）。使用多阶段构建，可以通过在一个Dockerfile文件中使用多个FROM语句。

每个FROM 指令都可以使用不同的基础镜像，并表示开始一个新的构建阶段。这样可以方便的将一个阶段的文件复制到另外一个阶段，将最终的镜像仅保留需要的内容即可。

```shell
FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM microsoft/dotnet:2.1-sdk AS build
WORKDIR /src
COPY ["DockerSample/DockerSample.csproj", "DockerSample/"]
RUN dotnet restore "DockerSample/DockerSample.csproj"
COPY . .
WORKDIR "/src/DockerSample"
RUN dotnet build "DockerSample.csproj" -c Release -o /app

FROM build AS publish
RUN dotnet publish "DockerSample.csproj" -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
```

