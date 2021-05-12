# dubbo-go 介绍

## github

> https://github.com/apache/dubbo-go

## apache

> https://dubbo.apache.org/zh/blog/2021/02/20/dubbo-go-%E7%99%BD%E8%AF%9D%E6%96%87/



## 安装环境

```shell
这里安装docker, docker-compose

查看是否有安装
yum list installed | grep docker

# 阿里云脚本安装
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
systemctl start docker

# 安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
```



# 入门

## docker-compose安装

```
vim /docker-compose/dubbo-go/example/docker-compose.yaml
```



```yaml
version: '3'
services:
  zookeeper:
    image: zookeeper
    ports:
      - 2181:2181
  admin:
    image: apache/dubbo-admin
    depends_on:
      - zookeeper
    ports:
      - 8080:8080
    environment:
      - admin.registry.address=zookeeper://zookeeper:2181
      - admin.config-center=zookeeper://zookeeper:2181
      - admin.metadata-report.address=zookeeper://zookeeper:2181
```

```
docker-compose up -d
```



## 管理后台

http://10.4.7.70:8080/

 ![image-20210309150727182](F:\self_dev_node\AAAAA笔记\开发课程\云原生\images\image-20210309150727182.png)



## 运行helloword案例

```
https://github.com/apache/dubbo-go-samples/tree/master/helloworld
```

下载上面的项目案例

```shell
在窗口1：
export CONF_PROVIDER_FILE_PATH="/mnt/hgfs/go_work/dubbo-go/dubbo-go-samples-master/helloworld/go-server/conf/server.yml"

export APP_LOG_CONF_FILE="/mnt/hgfs/go_work/dubbo-go/dubbo-go-samples-master/helloworld/go-server/conf/log.yml"

export CONF_CONSUMER_FILE_PATH="/mnt/hgfs/go_work/dubbo-go/dubbo-go-samples-master/helloworld/go-client/conf/client.yml"

cd /mnt/hgfs/go_work/dubbo-go/dubbo-go-samples-master/helloworld/go-server
go run cmd/server.go
```



```shell
在窗口2：
export CONF_PROVIDER_FILE_PATH="/mnt/hgfs/go_work/dubbo-go/dubbo-go-samples-master/helloworld/go-server/conf/server.yml"

export APP_LOG_CONF_FILE="/mnt/hgfs/go_work/dubbo-go/dubbo-go-samples-master/helloworld/go-server/conf/log.yml"

export CONF_CONSUMER_FILE_PATH="/mnt/hgfs/go_work/dubbo-go/dubbo-go-samples-master/helloworld/go-client/conf/client.yml"

cd /mnt/hgfs/go_work/dubbo-go/dubbo-go-samples-master/helloworld/go-client
go run cmd/client.go
```

