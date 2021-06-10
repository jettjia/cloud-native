# helm 部署rabbitmq集群



## 自己制作helm发布

```
cd /root/install-some-apps/rabbit-cluster
```



可以参考下面的内容





## 官网helm

```shell
[root@k8s-master01 ~]# helm repo list 
NAME         	URL                                                   
aliyuncs     	https://apphub.aliyuncs.com                           
stable       	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
ingress-nginx	https://kubernetes.github.io/ingress-nginx 
```

### 查看可使用tabbitmq-ha的版本

```shell
[root@k8s-master01 ~]# helm search repo rabbitmq-ha 
NAME                	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyuncs/rabbitmq-ha	1.39.0       	3.8.0      	Highly available RabbitMQ cluster, the open sou...
stable/rabbitmq-ha  	1.0.0        	3.7.3      	Highly available RabbitMQ cluster, the open sou...
[root@k8s-master01 ~]# helm search repo rabbitmq-ha --versions
NAME                	CHART VERSION	APP VERSION	DESCRIPTION                                       
aliyuncs/rabbitmq-ha	1.39.0       	3.8.0      	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.38.2       	3.8.0      	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.38.1       	3.8.0      	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.36.4       	3.8.0      	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.36.3       	3.8.0      	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.36.0       	3.8.0      	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.34.1       	3.7.19     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.34.0       	3.7.19     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.33.0       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.32.4       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.32.3       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.32.2       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.32.0       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.31.0       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.30.0       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.29.1       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.29.0       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.28.0       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.27.2       	3.7.15     	Highly available RabbitMQ cluster, the open sou...
aliyuncs/rabbitmq-ha	1.27.1       	3.7.12     	Highly available RabbitMQ cluster, the open sou...
stable/rabbitmq-ha  	1.0.0        	3.7.3      	Highly available RabbitMQ cluster, the open sou...
stable/rabbitmq-ha  	0.1.1        	3.7.0      	Highly available RabbitMQ cluster, the open sou...
```

### 拉取指定版本的配置

```shell
[root@k8s-master01 ~]# helm pull aliyuncs/rabbitmq-ha --version=1.33.0
[root@k8s-master01 ~]# ls
helm  old  rabbitmq-cluster  rabbitmq-ha-1.33.0.tgz #这个就只拉去的文件，可以解压
```

### 查询配置文件

```shell
[root@k8s-master01 ~]# tar -xf rabbitmq-ha-1.33.0.tgz 
[root@k8s-master01 ~]# ls
helm  old  rabbitmq-cluster  rabbitmq-ha  rabbitmq-ha-1.33.0.tgz
[root@k8s-master01 ~]# tree rabbitmq-ha
rabbitmq-ha
├── Chart.yaml
├── OWNERS
├── README.md
├── templates
│   ├── alerts.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── pdb.yaml
│   ├── rolebinding.yaml
│   ├── role.yaml
│   ├── secret.yaml
│   ├── serviceaccount.yaml
│   ├── service-discovery.yaml
│   ├── servicemonitor.yaml
│   ├── service.yaml
│   └── statefulset.yaml
└── values.yaml

1 directory, 18 files
```

参考

```shell
├── charts # 依赖文件
├── Chart.yaml # 这个chart的版本信息
├── templates #模板
│   ├── deployment.yaml
│   ├── _helpers.tpl # 自定义的模板或者函数
│   ├── ingress.yaml
│   ├── NOTES.txt #这个chart的信息
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml #配置全局变量或者一些参数
```

### 创建namespaces

helm指定namespaces，如果没有namespaces空间，就会报错。需要提前创建

```shell
[root@k8s-master01 ~]# helm create rabbitmq-cluster
Creating rabbitmq-cluster
```

### 运行rabbitmq集群

```shell
[root@k8s-master01 rabbitmq-ha]# pwd
/root/rabbitmq-ha
[root@k8s-master01 rabbitmq-ha]# ls
Chart.yaml  OWNERS  README.md  templates  values.yaml
[root@k8s-master01 rabbitmq-ha]# helm install rabbitmq --namespace  rabbitmq-cluster --set ingress.enabled=true,ingress.hostName=rabbitmq.akiraka.net --set rabbitmqUsername=aka,rabbitmqPassword=rabbitmq,managementPassword=rabbitmq,rabbitmqErlangCookie=secretcookie .

NAME: rabbitmq
LAST DEPLOYED: Sun Mar 14 17:25:11 2021
NAMESPACE: rabbitmq-cluster
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

  Credentials:

    Username            : aka
    Password            : $(kubectl get secret --namespace rabbitmq-cluster rabbitmq-rabbitmq-ha -o jsonpath="{.data.rabbitmq-password}" | base64 --decode)
    Management username : management
    Management password : $(kubectl get secret --namespace rabbitmq-cluster rabbitmq-rabbitmq-ha -o jsonpath="{.data.rabbitmq-management-password}" | base64 --decode)
    ErLang Cookie       : $(kubectl get secret --namespace rabbitmq-cluster rabbitmq-rabbitmq-ha -o jsonpath="{.data.rabbitmq-erlang-cookie}" | base64 --decode)

  RabbitMQ can be accessed within the cluster on port 5672 at rabbitmq-rabbitmq-ha.rabbitmq-cluster.svc.cluster.local

  To access the cluster externally execute the following commands:

    export POD_NAME=$(kubectl get pods --namespace rabbitmq-cluster -l "app=rabbitmq-ha" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward $POD_NAME --namespace rabbitmq-cluster 5672:5672 15672:15672

  To Access the RabbitMQ AMQP port:

    amqp://127.0.0.1:5672/ 

  To Access the RabbitMQ Management interface:

    URL : http://127.0.0.1:15672
    

```

### 查看集群

```shell
[root@k8s-master01 rabbitmq-ha]# kubectl get svc,pod,ingress -n rabbitmq-cluster  
NAME                                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                       AGE
service/rabbitmq-rabbitmq-ha             ClusterIP   None         <none>        15672/TCP,5672/TCP,4369/TCP   7m23s
service/rabbitmq-rabbitmq-ha-discovery   ClusterIP   None         <none>        15672/TCP,5672/TCP,4369/TCP   7m23s

NAME                         READY   STATUS    RESTARTS   AGE
pod/rabbitmq-rabbitmq-ha-0   1/1     Running   0          7m23s
pod/rabbitmq-rabbitmq-ha-1   1/1     Running   0          6m56s
pod/rabbitmq-rabbitmq-ha-2   1/1     Running   0          6m34s

NAME                                      CLASS    HOSTS                  ADDRESS        PORTS   AGE
ingress.extensions/rabbitmq-rabbitmq-ha   <none>   rabbitmq.akiraka.net   10.96.107.62   80      7m23s
```

 ![image-20210603100608788](images/image-20210603100608788.png)



### 补充

```shell
helm install helm-test2 --set fullnameOverride=aaaaaaa --dry-run .
这个是模拟运行
```

```shell
删除helm uninstall rabbitmq-cluster -n public-service  # helm v3 --keep-history
	helm delete/del NAME --purge 
升级helm upgrade rabbitmq-cluster -n public-service .
```

卸载保留历史记录

```shell
[root@k8s-master01 rabbitmq-ha]# helm  uninstall rabbitmq -n rabbitmq-cluster --keep-history 
release "rabbitmq" uninstalled
[root@k8s-master01 rabbitmq-ha]# helm list  -n rabbitmq-cluster 
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
[root@k8s-master01 rabbitmq-ha]# helm list  
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
[root@k8s-master01 rabbitmq-ha]# helm  ls
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
[root@k8s-master01 rabbitmq-ha]#  helm status rabbitmq -n rabbitmq-cluster
NAME: rabbitmq
LAST DEPLOYED: Sun Mar 14 17:25:11 2021
NAMESPACE: rabbitmq-cluster
STATUS: uninstalled
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

  Credentials:

    Username            : aka
    Password            : $(kubectl get secret --namespace rabbitmq-cluster rabbitmq-rabbitmq-ha -o jsonpath="{.data.rabbitmq-password}" | base64 --decode)
    Management username : management
    Management password : $(kubectl get secret --namespace rabbitmq-cluster rabbitmq-rabbitmq-ha -o jsonpath="{.data.rabbitmq-management-password}" | base64 --decode)
    ErLang Cookie       : $(kubectl get secret --namespace rabbitmq-cluster rabbitmq-rabbitmq-ha -o jsonpath="{.data.rabbitmq-erlang-cookie}" | base64 --decode)

  RabbitMQ can be accessed within the cluster on port 5672 at rabbitmq-rabbitmq-ha.rabbitmq-cluster.svc.cluster.local

  To access the cluster externally execute the following commands:

    export POD_NAME=$(kubectl get pods --namespace rabbitmq-cluster -l "app=rabbitmq-ha" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward $POD_NAME --namespace rabbitmq-cluster 5672:5672 15672:15672

  To Access the RabbitMQ AMQP port:

    amqp://127.0.0.1:5672/ 

  To Access the RabbitMQ Management interface:

    URL : http://127.0.0.1:15672
```

