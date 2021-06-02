# k8s 安装 rabbitmq

## rabbitmq-cluster-operator 方式安装

文档: https://github.com/rabbitmq/cluster-operator



## StatefulSet 方式安装

```shell
mkdir -p /root/install-some-apps/ && cd /root/install-some-apps
mkdir rabbitmq-cluster
cd rabbitmq-cluster
```

使用参考：https://www.cnblogs.com/dukuan/p/9897443.html



```shell
# Pull image
docker pull rabbitmq:3.8.3-management

docker save rabbitmq:3.8.3-management -o /tmp/
scp -r /tmp/rabbit.tar root@10.4.7.108:/tmp/
scp -r /tmp/rabbit.tar k8s-node01:/tmp/
# login k8s-node01
docker load -i /tmp/rabbit.tar
```



下面是分布，自己手动创建

### configmap

```shell
vim rabbitmq-configmap.yaml
```

```shell
kind: ConfigMap
apiVersion: v1
metadata:
  name: rmq-cluster-config
  namespace: public-service
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
data:
    enabled_plugins: |
      [rabbitmq_management,rabbitmq_peer_discovery_k8s].
    rabbitmq.conf: |
      loopback_users.guest = false
      username: RABBITMQ_USER
      password: RABBITMQ_PASS
      ## Clustering
      cluster_formation.peer_discovery_backend = rabbit_peer_discovery_k8s
      cluster_formation.k8s.host = kubernetes.default.svc.cluster.local
      cluster_formation.k8s.address_type = hostname
      #################################################
      # public-service is rabbitmq-cluster's namespace#
      #################################################
      cluster_formation.k8s.hostname_suffix = .rmq-cluster.public-service.svc.cluster.local
      cluster_formation.node_cleanup.interval = 10
      cluster_formation.node_cleanup.only_log_warning = true
      cluster_partition_handling = autoheal
      ## queue master locator
      queue_master_locator=min-masters
```

```shell
kubectl create ns public-service
kubectl create configmap rabbitmq-configmap.yaml
```

### **secret**

```shell
vim rabbitmq-secret.yaml
```

```shell
  
kind: Secret
apiVersion: v1
metadata:
  name: rmq-cluster-secret
  namespace: public-service
stringData:
  cookie: ERLANG_COOKIE
  password: RABBITMQ_PASS
  url: amqp://RABBITMQ_USER:RABBITMQ_PASS@rmq-cluster-balancer
  username: RABBITMQ_USER
type: Opaque
```

```shell
kubectl create -f rabbitmq-secret.yaml
```



### service

```shell
vim rabbitmq-svc.yaml
```

```shell 
kind: Service
apiVersion: v1
metadata:
  labels:
    app: rmq-cluster
  name: rmq-cluster
  namespace: public-service
spec:
  clusterIP: None
  ports:
  - name: amqp
    port: 5672
    targetPort: 5672
  selector:
    app: rmq-cluster
---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: rmq-cluster
    type: LoadBalancer
  name: rmq-cluster-balancer
  namespace: public-service
spec:
  ports:
  - name: http
    port: 15672
    protocol: TCP
    targetPort: 15672
  - name: amqp
    port: 5672
    protocol: TCP
    targetPort: 5672
  selector:
    app: rmq-cluster
  type: NodePort
```

```shell
kubectl create -f rabbitmq-svc.yaml
```



### rbac

```shell
vim rabbitmq-rbac.yaml
```

```shell
apiVersion: v1
kind: ServiceAccount
metadata:
  name: rmq-cluster
  namespace: public-service
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: rmq-cluster
  namespace: public-service
rules:
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: rmq-cluster
  namespace: public-service
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: rmq-cluster
subjects:
- kind: ServiceAccount
  name: rmq-cluster
  namespace: public-service
```

```shell
kubect create -f rabbitmq-rbac.yaml
```



### sts

```shell
vim rabbitmq-sts.yaml
```

```yaml
kind: StatefulSet
apiVersion: apps/v1
metadata:
  labels:
    app: rmq-cluster
  name: rmq-cluster
  namespace: public-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rmq-cluster
  serviceName: rmq-cluster
  template:
    metadata:
      labels:
        app: rmq-cluster
    spec:
      containers:
      - args:
        - -c
        - cp -v /etc/rabbitmq/rabbitmq.conf ${RABBITMQ_CONFIG_FILE}; exec docker-entrypoint.sh
          rabbitmq-server
        command:
        - sh
        env:
        - name: RABBITMQ_DEFAULT_USER
          valueFrom:
            secretKeyRef:
              key: username
              name: rmq-cluster-secret
        - name: RABBITMQ_DEFAULT_PASS
          valueFrom:
            secretKeyRef:
              key: password
              name: rmq-cluster-secret
        - name: RABBITMQ_ERLANG_COOKIE
          valueFrom:
            secretKeyRef:
              key: cookie
              name: rmq-cluster-secret
        - name: K8S_SERVICE_NAME
          value: rmq-cluster
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: RABBITMQ_USE_LONGNAME
          value: "true"
        - name: RABBITMQ_NODENAME
          value: rabbit@$(POD_NAME).rmq-cluster.$(POD_NAMESPACE).svc.cluster.local
        - name: RABBITMQ_CONFIG_FILE
          value: /var/lib/rabbitmq/rabbitmq.conf
        image: rabbitmq:3.8.3-management
        imagePullPolicy: IfNotPresent
        livenessProbe:
          exec:
            command:
            - rabbitmqctl
            - status
          initialDelaySeconds: 30
          timeoutSeconds: 10
        name: rabbitmq
        ports:
        - containerPort: 15672
          name: http
          protocol: TCP
        - containerPort: 5672
          name: amqp
          protocol: TCP
        readinessProbe:
          exec:
            command:
            - rabbitmqctl
            - status
          initialDelaySeconds: 10
          timeoutSeconds: 10
        volumeMounts:
        - mountPath: /etc/rabbitmq
          name: config-volume
          readOnly: false
        - mountPath: /var/lib/rabbitmq
          name: rabbitmq-storage
          readOnly: false
      serviceAccountName: rmq-cluster
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          items:
          - key: rabbitmq.conf
            path: rabbitmq.conf
          - key: enabled_plugins
            path: enabled_plugins
          name: rmq-cluster-config
        name: config-volume
      - name: rabbitmq-storage
        emptyDir: {}
```

```shell
kubectl create rabbitmq-sts.yaml
```



### 查看

```shell
kubectl get po -n public-service

# entry
kubectl exec -it rmq-cluster-0 -n public-service -- bash
more /var/lib/rabbitmq/rabbitmq.conf


kubectl get ep -n !$

# login rabbitmq web
http://10.4.7.107:31497

username: RABBITMQ_USER
password: RABBITMQ_PASS
```



  ![image-20210602151124959](images/image-20210602151124959.png)



### 扩容、缩容

```shell
kubectl get sts -n public-service

# 扩容
kubectl scale sts rmq-cluster --replicas=4 -n public-service

kubeclt get po -n !$

# 缩容
kubectl scale sts rmq-cluster --replicas=3 -n public-service
```

