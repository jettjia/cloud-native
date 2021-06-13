# 二进制Etcd集群升级

## 步骤

```
1.备份etcd 数据
2.下载新版 etcd包
3.停止etcd
4.替换etcd和 etcdctl
5.启动
```



## 说明

需求：如果从k8s 1.20 升级到 k8s 1.21

查看官方的 更改日志

https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.21.md

找到下面这种，类似这种更新的

```
Kubeadm installs etcd v3.4.13 when creating cluster v1.19 (#97244, @pacoxu)
```



## 升级etcd操作

```shell
# v2
$ etcdctl --ca-file /etc/kubernetes/pki/etcd/etcd-ca.pem --key-file /etc/kubernetes/pki/etcd/etcd-key-pem --cert-file /etc/kubernetes/pki/etcd/etcd.pem --endpoints https://10.4.7.107:2379,https://10.4.7.108:2379,https://10.4.7.109:2379 member list


# v3
$ export ETCDCTL_API=3
$ etcdctl --cacert=/etc/kubernetes/pki/etcd/etcd-ca.pem --cert=/etc/kubernetes/pki/etcd/etcd.pem --key=/etc/kubernetes/pki/etcd/etcd-key.pem  --endpoints https://10.4.7.107:2379,https://10.4.7.108:2379,https://10.4.7.109:2379 member list
2075659d8c0e93a, started, k8s-master02, https://10.4.7.108:2380, https://10.4.7.108:2379, false
7626dc6cd4e892c1, started, k8s-master03, https://10.4.7.109:2380, https://10.4.7.109:2379, false
badc9b068aea441f, started, k8s-master01, https://10.4.7.107:2380, https://10.4.7.107:2379, false

## or
$ etcdctl --endpoints="10.4.7.109:2379,10.4.7.108:2379,10.4.7.107:2379" --cacert=/etc/kubernetes/pki/etcd/etcd-ca.pem --cert=/etc/kubernetes/pki/etcd/etcd.pem --key=/etc/kubernetes/pki/etcd/etcd-key.pem  endpoint status --write-out=table
+-----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT     |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+-----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 10.4.7.109:2379 | 7626dc6cd4e892c1 |  3.4.13 |  8.5 MB |     false |      false |       235 |      99457 |              99457 |        |
| 10.4.7.108:2379 |  2075659d8c0e93a |  3.4.13 |  8.5 MB |     false |      false |       235 |      99457 |              99457 |        |
| 10.4.7.107:2379 | badc9b068aea441f |  3.4.13 |  8.5 MB |      true |      false |       235 |      99457 |              99457 |        |
+-----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------
```

```shell
# 备份
$ etcdctl --cacert=/etc/kubernetes/pki/etcd/etcd-ca.pem --cert=/etc/kubernetes/pki/etcd/etcd.pem --key=/etc/kubernetes/pki/etcd/etcd-key.pem  --endpoints https://10.4.7.107:2379 snapshot save 20210610-2301.db
```

```shell
# master02, 停掉 etcd
$ systemctl stop etcd
$ which etcd
/usr/local/bin/etcd
```

```shell
# master01，查看 etcd集群， 发现master02已经停掉
$ etcdctl --cacert=/etc/kubernetes/pki/etcd/etcd-ca.pem --cert=/etc/kubernetes/pki/etcd/etcd.pem --key=/etc/kubernetes/pki/etcd/etcd-key.pem  --endpoints https://10.4.7.107:2379,https://10.4.7.108:2379,https://10.4.7.109:2379 endpoin health
{"level":"warn","ts":"2021-06-11T00:13:10.066+0800","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"endpoint://client-36d6a97d-a057-42a8-8079-d58b87a52b13/10.4.7.108:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: all SubConns are in TransientFailure, latest connection error: connection error: desc = \"transport: Error while dialing dial tcp 10.4.7.108:2379: connect: connection refused\""}
https://10.4.7.107:2379 is healthy: successfully committed proposal: took = 18.702919ms
https://10.4.7.109:2379 is healthy: successfully committed proposal: took = 20.945198ms
https://10.4.7.108:2379 is unhealthy: failed to commit proposal: context deadline exceeded

# tar etcd
$ tar xf etcd-v3.4.13-linux-amd64.tar.gz
$ cd etcd-v3.4.13
$ scp etcd* k8s-master02:/usr/local/bin/etcd
```

```shell
# master02, 停掉 etcd
$ etcdctl version
etcdctl version: 3.4.13
API version: 3.4

$ systemctl start etcd


# 如果启动失败
vim /etc/etcd/etcd.config.yml  ## 修改成下面这种
log-outputs: [default]

```



## 升级其他节点

```shell
# master03 升级同上面操作

# 主节点也是同上面操作
```

