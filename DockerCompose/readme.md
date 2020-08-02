# 安装

安装 python-pip

```shell
yum -y install epel-release

pip -V
yum -y install pip
//如果提示没有可用软件包pip，则使用下面的命令
yum -y install python-pip
```

升级pip

```shell
pip install --upgrade pip
```

安装Docker-compose

```
pip install docker-compose
docker-compose -version
```



# yml文件

```yaml
version: '3'
services:
  web:    
  build: .    
  ports:    
  - "5000:5000"
  volumes:
       - .:/code
  redis:    
  image: "redis:alpine"
```



# Compose

