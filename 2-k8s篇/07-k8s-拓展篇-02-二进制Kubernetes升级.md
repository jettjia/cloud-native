# 二进制Kubernetes升级

## 升级Master节点

下载对应要升级的版本

 ![image-20210610163437243](images/image-20210610163437243.png)



### 升级master01

```shell
$ tar -xf kubernetes-server-linux-amd64.tar.gz

$ cd /kubernetes/server/bin
$ ./kubectl version

```



```shell
# 升级 apiserver
$ cd /kubernetes/server/bin

$ systemctl stop kube-apiserver
$ which kube-apiserver
/usr/local/bin/kube-apiserver
$ cp -rp  kube-apiserver /usr/local/bin/kube-apiserver
$ /usr/local/bin/kube-apiserver --version

$ systemctl daemon-reload
$ systemctl restart kube-apiserver
$ tail -f /var/log/messages
```



```shell
# 升级 kube-controller-manager kube-scheduler
$ cd /kubernetes/server/bin

$ systemctl stop kube-controller-manager kube-scheduler
$ \cp -rp  kube-controller-manager kube-scheduler /usr/local/bin/
$ systemctl restart kube-controller-manager
$ systemctl status kube-controller-manager
$ tail -f /var/log/messages

$ systemctl restart kube-scheduler
$ tail -f /var/log/messages

```



```shell
# 升级 kube-proxy
$ cd /kubernetes/server/bin
$ systemctl stop kube-proxy
$ \cp -rp kube-proxy /usr/local/bin/
$ systemctl restart kube-proxy
$ systemctl status kube-proxy
```

```shell
# 升级 kubectl
$ cd /kubernetes/server/bin
$ cp -rp kubectl /usr/local/bin/
```



### 升级其他master 节点

master01上拷贝到其他master节点

```shell
$ scp kube-apiserver kube-controller-manager kube-scheduler  kube-proxy kubectl k8s-master02:/tmp/
$ scp kube-apiserver kube-controller-manager kube-scheduler  kube-proxy kubectl k8s-master03:/tmp/
```

master02, master03机器操作，同上



## 升级Node节点和Calico

```
建议：kubelet和 calico一起升级，每次升级一个节点
```



### master02

```shell
# 下线Node节点
$ kubectl drain k8s-master02 --delete-local-data --force --ignore-daemonsets

$ systemctl stop kubelet
$ cp -rp kubelet /usr/local/bin/kubelet

```



calico升级

文档：https://docs.projectcalico.org/maintenance/kubernetes-upgrade

安装：https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises

master02升级

```shell
# master01
$ curl https://docs.projectcalico.org/manifests/calico.yaml -O
$ vim calico.yaml
	# 修改如下
	updateStrategy
	  type: OnDelete # 修改成这个
$ kubectl apply -f calico.yaml
$ kubectl get po -n kube-system -owide

# 恢复 master02
$ kubectl uncordon k8s-master02

# 去master02
$ systemclt restart kubelet


# master01上操作
# 查看master02上的 calico
$ kubectl get po -n kuby-system -owide
$ kubectl describe po calico-node-dwgwe  ## 这个对应的是master02节点的
	#发现 node上的节点开始拉取新的calico

```



master01升级

```shell
$ systemctl stop kubelet
$ cp -rp kubelet /user/local/bin/
$ systemctl start kubelet
$ systemctl status kubelet

$ kubectl get node
```

master 03同上

