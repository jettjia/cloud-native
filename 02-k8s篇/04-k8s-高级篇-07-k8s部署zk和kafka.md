# k8s 部署zookeeper&kafka

文档: https://docs.bitnami.com/tutorials/deploy-scalable-kafka-zookeeper-cluster-kubernetes/

## zk

方式1：

```shell
cd /root/install-some-apps

# list
helm repo list

# 如果没有，添加
helm repo add bitnami https://charts.bitnami.com/bitnami

# search
helm  search repo zookeeper
```



```shell
helm install zookeeper bitnami/zookeeper 
  --set replicaCount=1
  --set auth.enabled=false 
  --set allowAnonymousLogin=true
```

生产

```
helm install zookeeper bitnami/zookeeper 
  --set replicaCount=3 
  --set auth.enabled=true 
  --set allowAnonymousLogin=true
```



方式2：

```shell
helm pull bitnami/zookeeper
tar xf zookeeper-5.11.0.tgz
cd zookeeper

# 关掉存储卷
vim values.yaml
enabled: false
storageClass: "-" #上面一行的 enabled: 设置为 false

# install
helm install zookeeper -n public-service --set auth.enabled=false --set allowAnonymousLogin=true

kubectl get po -n public-service
```



## kafka

方式1：

```shell
helm install kafka bitnami/kafka 
  --set zookeeper.enabled=false 
  --set replicaCount=3
  --set externalZookeeper.servers=ZOOKEEPER-SERVICE-NAME
```



方式2：

```shell
# 到下载的kafka当前目录，这里的zk用的是上面安装的zk
helm pull bitnami/kafka
tar xf kafka-10.2.1.tgz
cd kafka && ls


# install
helm install kafka -n public-service --set zookeeper.enabled=false --set replicaCount=1 --set externalZookeeper.servers=zookeeper .
  
kubectl ge po -n public-service
kubectl get svc -n public-service
```





## 测试zk & kafka

文档：https://docs.bitnami.com/tutorials/deploy-scalable-kafka-zookeeper-cluster-kubernetes/

```shell
kubectl get po -n public-service

# entry
kubectl exec -it kafka-0 -n public-service -- bash

# Create a topic

kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic mytopic

# Start a Kafka message consumer
kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic mytopic --consumer.config /opt/bitnami/kafka/conf/consumer.properties &
# 如果上面失效，下面的
kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic mytopic

# Using a different console, start a Kafka message producer
kubectl get po -n public-service
kubectl default exec -it kafka-0 -n public-service -- bash
kafka-console-producer.sh --broker-list kafka:9092 --topic mytopic
# 在这里创建消息，然后在消费者窗口查看消息
```



## zk & kafka 扩容缩容

```shell
helm upgrade kafka bitnami/kafka
  --set zookeeper.enabled=false 
  --set replicaCount=7
  --set externalZookeeper.servers=ZOOKEEPER-SERVICE-NAME
```

```shell
helm upgrade zookeeper bitnami/zookeeper
  --set replicaCount=5 
  --set auth.enabled=false 
  --set allowAnonymousLogin=true
```

