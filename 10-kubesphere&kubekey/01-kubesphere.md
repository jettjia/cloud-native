# KubeSphere安装

## 利用Kubekey安装 kubeSphere

参考: 02-kubekey.md



## 在k8s上，安装kubeSphere

参考：阿里云托管的k8s集群

https://kubesphere.io/zh/docs/installing-on-kubernetes/hosted-kubernetes/install-kubesphere-on-ack/

参考：本地自己安装的k8s集群

https://www.yuque.com/leifengyang/oncloud/gz1sls#flJaV 

# KubeSphere多租户

## 多组合图示

 ![img](assets/1631589337850-64ced113-11ed-4c25-99b0-5b101995cecf.png)



角色说明

| 内置角色             | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| `workspaces-manager` | 企业空间管理员，管理平台所有企业空间。                       |
| `users-manager`      | 用户管理员，管理平台所有用户。                               |
| `platform-regular`   | 平台普通用户，在被邀请加入企业空间或集群之前没有任何资源操作权限。 |
| `platform-admin`     | 平台管理员，可以管理平台内的所有资源。                       |



## 架构说明

KubeSphere 的多租户系统分**三个**层级，即集群、企业空间和项目。KubeSphere 中的项目等同于 Kubernetes 的[命名空间](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/namespaces/)。



## 实践

实践用户列表

| 帐户              | 角色                 | 描述                                                         |
| :---------------- | :------------------- | :----------------------------------------------------------- |
| `ws-manager`      | `workspaces-manager` | 创建和管理所有企业空间。                                     |
| `ws-admin`        | `platform-regular`   | 管理指定企业空间中的所有资源（在此示例中，此帐户用于邀请新成员加入该企业空间）。 |
| `project-admin`   | `platform-regular`   | 创建和管理项目以及 DevOps 项目，并邀请新成员加入项目。       |
| `project-regular` | `platform-regular`   | `project-regular` 将由 `project-admin` 邀请至项目或 DevOps 项目。该帐户将用于在指定项目中创建工作负载、流水线和其他资源。 |

测试密码全部设置为： Test123

admin初始的账号，已经修改为：Test123456

### 创建用户

点击左上角的**平台管理**，然后选择**访问控制**。在左侧导航栏中，选择**平台角色**。这里是查看角色

点击左上角的**平台管理**，然后选择**访问控制**。在左侧导航栏中，选择**用户**。创建用户，根据上面的图示，创建对应的用户来测试。



### 创建企业空间

1. 以 `ws-manager` 身份登录 KubeSphere，它具有管理平台上所有企业空间的权限。点击左上角的**平台管理**，选择**访问控制**。在**企业空间**中，可以看到仅列出了一个默认企业空间 `system-workspace`，即系统企业空间，其中运行着与系统相关的组件和服务，您无法删除该企业空间。
2. 点击右侧的**创建**，将新企业空间命名为 `demo-workspace`，并将用户 `ws-admin` 设置为企业空间管理员。完成后，点击**创建**。

3. 登出控制台，然后以 `ws-admin` 身份重新登录。在**企业空间设置**中，选择**企业空间成员**，然后点击**邀请**。

4. 邀请 `project-admin` 和 `project-regular` 进入企业空间，分别授予 `workspace-self-provisioner` 和 `workspace-viewer` 角色，点击**确定**。

5. 将 `project-admin` 和 `project-regular` 都添加到企业空间后，点击**确定**。在**企业空间**中，您可以看到列出的三名成员。

| 帐户              | 角色                         | 描述                                                         |
| :---------------- | :--------------------------- | :----------------------------------------------------------- |
| `ws-admin`        | `workspace-admin`            | 管理指定企业空间中的所有资源（在此示例中，此帐户用于邀请新成员加入企业空间）。 |
| `project-admin`   | `workspace-self-provisioner` | 创建和管理项目以及 DevOps 项目，并邀请新成员加入项目。       |
| `project-regular` | `workspace-viewer`           | `project-regular` 将由 `project-admin` 邀请至项目或 DevOps 项目。该帐户将用于在指定项目中创建工作负载、流水线和其他资源。 |



### 创建项目

1. 以 `project-admin` 身份登录 KubeSphere Web 控制台，在**项目**中，点击**创建**。

2. 输入项目名称（例如 `demo-project`），然后点击**确定**完成，您还可以为项目添加别名和描述。

3. 在**项目**中，点击刚创建的项目查看其详情页面。

4. 在项目的**概览**页面，默认情况下未设置项目配额。您可以点击**编辑配额**并根据需要指定[资源请求和限制](https://kubesphere.io/zh/docs/workspace-administration/project-quotas/)（例如：CPU 和内存的限制分别设为 1 Core 和 1000 Gi）。

5. 在**项目设置** > **项目成员**中，邀请 `project-regular` 至该项目，并授予该用户 `operator` 角色。

   信息

   具有 `operator` 角色的用户是项目维护者，可以管理项目中除用户和角色以外的资源。

6. 在创建[应用路由](https://kubesphere.io/zh/docs/project-user-guide/application-workloads/routes/)（即 Kubernetes 中的 [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)）之前，需要启用该项目的网关。网关是在项目中运行的 [NGINX Ingress 控制器](https://github.com/kubernetes/ingress-nginx)。若要设置网关，请转到**项目设置**中的**网关设置**，然后点击**设置网关**。此步骤中仍使用帐户 `project-admin`。

7. 选择访问方式 **NodePort**，然后点击**确定**。

8. 在**网关设置**下，可以在页面上看到网关地址以及 http/https 的端口。



### 创建DevOps项目

若要创建 DevOps 项目，需要预先启用 KubeSphere DevOps 系统，该系统是个可插拔的组件

1. 以 `project-admin` 身份登录控制台，在 **DevOps 项目**中，点击**创建**。
2. 输入 DevOps 项目名称（例如 `demo-devops`），然后点击**确定**，也可以为该项目添加别名和描述。
3. 点击刚创建的项目查看其详细页面。
4. 转到 **DevOps 项目设置**，然后选择 **DevOps 项目成员**。点击**邀请**授予 `project-regular` 用户 `operator` 的角色，允许其创建流水线和凭证。

# KubeSphere中间件

## 部署中间件要素

应用部署需要关注的信息【应用部署三要素】

1、应用的部署方式

2、应用的数据挂载（数据，配置文件）

3、应用的可访问性

 ![image-20220104144228746](assets/image-20220104144228746.png)



## KubeShpere-k8s-mysql

### 基础准备

#### 1）部署分析

 ![image-20220105101226579](assets/image-20220105101226579.png)

a.需要指定环境变量 MYSQL_ROOT_PASSWORD

b.需要有mysql的配置内容 my.cnf

c.需要指定mysql的存储 pvc



#### 2）找到安装mysql的命令

https://registry.hub.docker.com/_/mysql

```shell
docker run -p 3306:3306 --name mysql-01 \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root \
--restart=always \
-d mysql:5.7 
```

#### 3）mysql配置的内容

```
[client]
default-character-set=utf8mb4
 
[mysql]
default-character-set=utf8mb4
 
[mysqld]
init_connect='SET collation_connection = utf8mb4_unicode_ci'
init_connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```



### 配置

配置的就是ConfigMap的信息，我们会把mysql的配置内容，定义在ConfigMap中。



1）进入配置页面；平台管理->集群管理->配置->配置字典

2）配置

 ![image-20220104163124362](assets/image-20220104163124362.png)

3）下一步，添加数据

说明

​	键：

​		my.cnf 就是会在容器里的名称；

​	值：

​		这里就是配置mysql的内容

 ![image-20220104163242025](assets/image-20220104163242025.png)



### 存储

pvc，用来存储数据的，比如这里会把单独申请一个存储空间来保存mysql的内容。

注意：这里如果是提前申请的 pvc，也就是在 存储->存储卷中申请。后续mysql扩容的话，会都使用这一个存储卷。

一般是在应用负载配置的时候，一同申请。这样如果mysql扩容，也会一同申请新的pvc。

1）进入页面

平台管理->集群管理->存储->存储卷

2）配置

 ![image-20220104164105857](assets/image-20220104164105857.png)

### 发布和暴露服务

#### 1）进入配置

进入配置页面；平台管理->集群管理->应用负载->工作负载

#### 2）选择有状态副本集

#### 3）创建基本信息

![image-20220104171300904](assets/image-20220104171300904.png)

#### 4）容器组配置-添加容器

 ![image-20220104171501428](assets/image-20220104171501428.png)

 ![image-20220104171612052](assets/image-20220104171612052.png)

 设置资源限制![image-20220104171728530](assets/image-20220104171728530.png)

 

设置环境变量

 ![image-20220104171844708](assets/image-20220104171844708.png)

同步主机时区

 ![image-20220104171936961](assets/image-20220104171936961.png)

#### 5）存储卷设置-挂载存储卷

![image-20220104172234232](assets/image-20220104172234232.png)

 /var/lib/mysql ![image-20220104173923973](assets/image-20220104173923973.png)



#### 6）存储卷设置-挂载配置字典

 ![image-20220104172816084](assets/image-20220104172816084.png)

 ![image-20220104173017733](assets/image-20220104173017733.png)

#### 7）确认配置内容，并创建

#### 8）进入终端，查看

 ![image-20220104174256417](assets/image-20220104174256417.png)

#### 9）对外提供服务

应用负载->服务

1.在容器内登录mysql查看![image-20220104175055135](assets/image-20220104175055135.png)

 ![image-20220104175431878](assets/image-20220104175431878.png)



2.对外发布服务

应用负载->服务->创建

 ![image-20220104180016881](assets/image-20220104180016881.png)

 ![image-20220104185404727](assets/image-20220104185404727.png)



 ![image-20220104185446848](assets/image-20220104185446848.png)

测试链接

 ![image-20220104185635160](assets/image-20220104185635160.png)



## KubeShpere-k8s-redis

### 基础准备

#### 1）部署分析

  ![image-20220104190630326](assets/image-20220104190630326.png) 

a.需要redis的配置内容， redins.conf

b.需要指定redis的存储 , pvc

c.redis有启动命令 redis-server /etc/redis/redis.conf



#### 2）找到安装 redis 的命令

https://registry.hub.docker.com/_/redis

#### 3）配置内容

```
#创建配置文件
## 1、准备redis配置文件内容
mkdir -p /mydata/redis/conf && vim /mydata/redis/conf/redis.conf

##配置示例
appendonly yes
port 6379
bind 0.0.0.0

#docker启动redis
docker run -d -p 6379:6379 --restart=always \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-v /mydata/redis-01/data:/data \
 --name redis-01 redis:6.2.5 \
 redis-server /etc/redis/redis.conf
```



### 配置

1）进入配置页面；平台管理->集群管理->配置->配置字典

2）配置

 ![image-20220104192448555](assets/image-20220104192448555.png)

 ![image-20220104192610804](assets/image-20220104192610804.png)





### 发布和暴露服务

#### 1）进入配置

进入配置页面；平台管理->集群管理->应用负载->工作负载

#### 2）选择有状态副本集

#### 3）创建基本信息

 ![image-20220104192821033](assets/image-20220104192821033.png)

#### 4）容器组配置-添加容器

 ![image-20220104193203231](assets/image-20220104193203231.png)

 ![image-20220104193220108](assets/image-20220104193220108.png)

 ![image-20220104193335610](assets/image-20220104193335610.png)



#### 5）存储卷设置-挂载存储卷

这里，没有提前准备pvc，所以直接操作

 ![image-20220104193456582](assets/image-20220104193456582.png)

 ![image-20220104193626975](assets/image-20220104193626975.png)

#### 6）存储卷设置-挂载配置字典

 ![image-20220105094219896](assets/image-20220105094219896.png)

#### 7）确认配置内容，并创建

#### 8）进入终端，查看

#### 9）对外提供服务

应用负载->服务->创建

![image-20220105095852516](assets/image-20220105095852516.png)

 ![image-20220105095920844](assets/image-20220105095920844.png)

 ![image-20220105095953982](assets/image-20220105095953982.png)



## KubeShpere-k8s-es

### 基础准备

#### 1）部署分析

 ![image-20220105100823039](assets/image-20220105100823039.png)

a.需要指定es的配置， elasticsearch.yml, jvm.options

b.需要指定es的存储，pvc

c.需要指定环境变量, ES_JAVA_OPTS

d.这里会把配置内容挂载到 容器中， 其实 /usr/share/elasticsearch/config里有多个配置，我们这里要挂载进去的只有 elasticsearch.yml,  jvm.options

#### 2）找到安装 es 的命令

https://registry.hub.docker.com/_/elasticsearch

#### 3）配置内容

```
# 创建数据目录
mkdir -p /mydata/es-01/data && chmod 777 -R /mydata/es-01/data

# 容器启动
docker run --restart=always -d -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
-v es-config:/usr/share/elasticsearch/config \
-v /mydata/es-01/data:/usr/share/elasticsearch/data \
--name es-01 \
elasticsearch:7.13.4

```

elasticsearch.yml

```
cluster.name: "docker-cluster"
network.host: 0.0.0.0
```

jvm.options

```
################################################################
##
## JVM configuration
##
################################################################
##
## WARNING: DO NOT EDIT THIS FILE. If you want to override the
## JVM options in this file, or set any additional options, you
## should create one or more files in the jvm.options.d
## directory containing your adjustments.
##
## See https://www.elastic.co/guide/en/elasticsearch/reference/current/jvm-options.html
## for more information.
##
################################################################



################################################################
## IMPORTANT: JVM heap size
################################################################
##
## The heap size is automatically configured by Elasticsearch
## based on the available memory in your system and the roles
## each node is configured to fulfill. If specifying heap is
## required, it should be done through a file in jvm.options.d,
## and the min and max should be set to the same value. For
## example, to set the heap to 4 GB, create a new file in the
## jvm.options.d directory containing these lines:
##
## -Xms4g
## -Xmx4g
##
## See https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html
## for more information
##
################################################################


################################################################
## Expert settings
################################################################
##
## All settings below here are considered expert settings. Do
## not adjust them unless you understand what you are doing. Do
## not edit them in this file; instead, create a new file in the
## jvm.options.d directory containing your adjustments.
##
################################################################

## GC configuration
8-13:-XX:+UseConcMarkSweepGC
8-13:-XX:CMSInitiatingOccupancyFraction=75
8-13:-XX:+UseCMSInitiatingOccupancyOnly

## G1GC Configuration
# NOTE: G1 GC is only supported on JDK version 10 or later
# to use G1GC, uncomment the next two lines and update the version on the
# following three lines to your version of the JDK
# 10-13:-XX:-UseConcMarkSweepGC
# 10-13:-XX:-UseCMSInitiatingOccupancyOnly
14-:-XX:+UseG1GC

## JVM temporary directory
-Djava.io.tmpdir=${ES_TMPDIR}

## heap dumps

# generate a heap dump when an allocation from the Java heap fails; heap dumps
# are created in the working directory of the JVM unless an alternative path is
# specified
-XX:+HeapDumpOnOutOfMemoryError

# specify an alternative path for heap dumps; ensure the directory exists and
# has sufficient space
-XX:HeapDumpPath=data

# specify an alternative path for JVM fatal error logs
-XX:ErrorFile=logs/hs_err_pid%p.log

## JDK 8 GC logging
8:-XX:+PrintGCDetails
8:-XX:+PrintGCDateStamps
8:-XX:+PrintTenuringDistribution
8:-XX:+PrintGCApplicationStoppedTime
8:-Xloggc:logs/gc.log
8:-XX:+UseGCLogFileRotation
8:-XX:NumberOfGCLogFiles=32
8:-XX:GCLogFileSize=64m

# JDK 9+ GC logging
9-:-Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m
```



### 配置

1）进入配置页面；平台管理->集群管理->配置->配置字典

2）配置

 ![image-20220105102821237](assets/image-20220105102821237.png)

 ![image-20220105104754157](assets/image-20220105104754157.png)





### 发布和暴露服务

#### 1）进入配置

进入配置页面；平台管理->集群管理->应用负载->工作负载

#### 2）选择有状态副本集

#### 3）创建基本信息

 ![image-20220105110429057](assets/image-20220105110429057.png)



#### 4）容器组配置-添加容器

 ![image-20220105110702896](assets/image-20220105110702896.png)

 ![image-20220105110726328](assets/image-20220105110726328.png)

 ![image-20220105110856310](assets/image-20220105110856310.png)

#### 5）存储卷设置-挂载存储卷

 ![image-20220105111041044](assets/image-20220105111041044.png)

#### 6）存储卷设置-挂载配置字典

注意：这里用的是子路径的配置方式

a.配置elasticsearch.yml

配置里的内容是： /usr/share/elasticsearch/config/elasticsearch.yml

![image-20220105111753184](assets/image-20220105111753184.png)

 ![image-20220105111914752](assets/image-20220105111914752.png)

b.配置 jvm.options

配置里的内容是： /usr/share/elasticsearch/config/jvm.options

 ![image-20220105112252027](assets/image-20220105112252027.png)

#### 7）确认配置内容，并创建

#### 8）进入终端，查看

#### 9）对外提供服务

应用负载->服务->创建



## KubeShpere-应用商店-rabbitmq

### 启用应用商店

https://kubesphere.io/zh/docs/pluggable-components/app-store/#%E5%9C%A8%E5%AE%89%E8%A3%85%E5%90%8E%E5%90%AF%E7%94%A8%E5%BA%94%E7%94%A8%E5%95%86%E5%BA%97



### 应用商店-rabbitmq

应用商店-消息队列-rabbitmq

 ![image-20220105135908032](assets/image-20220105135908032.png)

 ![image-20220105135945784](assets/image-20220105135945784.png)



## KubeShpere-应用仓库-zk

### 安装仓库

helm：k8s的管理

https://helm.sh/

https://artifacthub.io/

 ![image-20220105141647551](assets/image-20220105141647551.png)

添加仓库

应用管理->应用仓库

 注意：要用空间管理员登录，我们这里是用ws-admin来登录![image-20220105141130123](assets/image-20220105141130123.png)

 ![image-20220105141808740](assets/image-20220105141808740.png)



### 部署应用

 ![image-20220105142201960](assets/image-20220105142201960.png)

 ![image-20220105142300605](assets/image-20220105142300605.png)

 ![image-20220105143102738](assets/image-20220105143102738.png)

## go 微服务中间件

### consul

​	bitnami中安装

### nacos

​	https://bytectl.github.io/helm-charts

 ![image-20220105144040096](assets/image-20220105144040096.png)

### jaeger

​	https://jaegertracing.github.io/helm-charts

  ![image-20220105144142876](assets/image-20220105144142876.png)

### kong

​	bitnami中安装

# DevOps-golang

## 启用DevOps

1. 以 `admin` 身份登录控制台，点击左上角的**平台管理**，选择**集群管理**。

2. 点击 **CRD**，在搜索栏中输入 `clusterconfiguration`，点击搜索结果查看其详细页面。

3. 在**自定义资源**中，点击 `ks-installer` 右侧的，选择**编辑 YAML**。

4. 在该 YAML 文件中，搜寻到 `devops`，将 `enabled` 的 `false` 改为 `true`。完成后，点击右下角的**确定**，保存配置。

   ```
   devops:
     enabled: true # 将“false”更改为“true”。
   ```

5. 您可以使用 Web Kubectl 工具执行以下命令来检查安装过程：

   ```
   kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
   ```

6. 查看是否安装成功

   ```
   kubectl get pod -n kubesphere-devops-system
   ```

7. 安装以后，登录后台查看

   ![image-20220105153455703](assets/image-20220105153455703.png)



## 创建DevOps项目

若要创建 DevOps 项目，需要预先启用 KubeSphere DevOps 系统，该系统是个可插拔的组件

1. 以 `project-admin` 身份登录控制台，在 **DevOps 项目**中，点击**创建**。
2. 输入 DevOps 项目名称（例如 `demo-devops`），然后点击**确定**，也可以为该项目添加别名和描述。
3. 点击刚创建的项目查看其详细页面。
4. 转到 **DevOps 项目设置**，然后选择 **DevOps 项目成员**。点击**邀请**授予 `project-regular` 用户 `operator` 的角色，允许其创建流水线和凭证。



## 官方介绍

 ![img](assets/1632308791299-29ecf126-98a4-474e-9140-5b9ec101cf45.png)

官方文档：https://kubesphere.com.cn/docs/devops-user-guide/understand-and-manage-devops-projects/overview/

Jenkins Agent: https://kubesphere.com.cn/docs/devops-user-guide/how-to-use/choose-jenkins-agent/#内置-podtemplate



## 快速入门

DevOps 项目->选中devops项目->流水线->创建

进入流水线，编辑测试

 ![image-20220105165154867](assets/image-20220105165154867.png)

 ![image-20220105165224526](assets/image-20220105165224526.png)



## go-micro-frame 微服务发布

### 项目架构

![架构图](assets/架构图.png)

代码目录

```
mall-code
|---mall-common                             	//通用模块
|---mall-srv                             		//微服务层-数据层
|------common-srv                         	    //公共微服务				[50050] 
|------goods-srv                         	    //商品微服务				[50052]
|---mall-web                             		//微服务层-业务层
|------api-backend-general-system-web			//总后台				[8021]  
|------api-mini-program-user-web				//小程序				[8020]  
```



### 中间件

| 中间件 | 集群内地址 | 外部访问地址 |
| ------ | ---------- | ------------ |
| Mysql  |            |              |
| Redis  |            |              |
|        |            |              |
| Nacos  |            |              |
| Consul |            |              |
| Jaeger |            |              |
| Kong   |            |              |
|        |            |              |
| Gitlab |            |              |
| Harbor |            |              |



### 发布准备

#### 发布准备



#### 配置抽取



### 项目发布

#### 拉取代码

#### 项目编译

#### 构建镜像

##### 基本设置

##### 并发构建

#### 推送镜像

##### 基础操作

##### 并发推送



#### 部署到k8s-dev环境

#### 系统邮件通知



#### webhook



# DevOps-vue



# ServiceMesh-istio



# k8s-ceph

