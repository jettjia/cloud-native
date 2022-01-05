## k8s部署 redis

### redis-cluster-operator 安装

文档：https://github.com/ucloud/redis-cluster-operator

master01

```shell
$ mkdir install-some-apps
$ cd install-some-apps
$ git clone git@github.com:ucloud/redis-cluster-operator.git

$ cd redis-cluster-operator
$ kubectl create -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
$ kubectl create -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml
$ kubectl create -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml

$ kubectl create -f deploy/service_account.yaml
$ kubectl create -f deploy/cluster/cluster_role.yaml
$ kubectl create -f deploy/cluster/cluster_role_binding.yaml
$ kubectl create -f deploy/cluster/operator.yaml

$ kubectl get deployment
$  kubectl get po

# install
$ kubectl apply -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml

# view
$ kubectl get distributedrediscluster
$ kubectl get all -l redis.kun/name=example-distributedrediscluster

$  kubectl get po
$  kubectl get svc


# entry
$ kubectl get svc
$ kubectl exec -it pod/drc-example-distributedrediscluster-0-0 -- sh
$ redis-cli -h example-distributedrediscluster
$ redis-cli -h 10.244.58.255
$ 10.244.58.255:6379> set a b
$ 10.244.58.255:6379> get a
```



### redis-cluster扩容缩容

扩容

```shell
kubectl edit Distributedrediscluster example-distributedrediscluster
```



```shell
apiVersion: redis.kun/v1alpha1
kind: DistributedRedisCluster
metadata:
  annotations:
    # if your operator run as cluster-scoped, add this annotations
    redis.kun/scope: cluster-scoped
  name: example-distributedrediscluster
spec:
  # Increase the masterSize to trigger the scaling.
  masterSize: 4 # 找到这个，修改为4
  ClusterReplicas: 1
  image: redis:5.0.4-alpine
```



缩容

```shell
apiVersion: redis.kun/v1alpha1
kind: DistributedRedisCluster
metadata:
  annotations:
    # if your operator run as cluster-scoped, add this annotations
    redis.kun/scope: cluster-scoped
  name: example-distributedrediscluster
spec:
  # Increase the masterSize to trigger the scaling.
  masterSize: 3  # 找到这个，修改为3
  ClusterReplicas: 1
  image: redis:5.0.4-alpine

```





### 清除

```shell
$ kubectl delete -f deploy/example/redis.kun_v1alpha1_distributedrediscluster_cr.yaml

$ kubectl delete -f deploy/cluster/operator.yaml
$ kubectl delete -f deploy/cluster/cluster_role.yaml
$ kubectl delete -f deploy/cluster/cluster_role_binding.yaml
$ kubectl delete -f deploy/service_account.yaml

$ kubectl delete -f deploy/crds/redis.kun_redisclusterbackups_crd.yaml
$ kubectl delete -f deploy/crds/redis.kun_distributedredisclusters_crd.yaml
```

