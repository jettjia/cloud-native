# 二进制CoreDNS升级

下载

master01 

```shell
$ git clone git@github.com:coredns/deployment.git
$ cd deployment
$ cd kubernetes

# 备份
$ mkdir bak
$ kubectl get cm coredns -n kube-system -oyaml > bak/coredns-cm.yaml
$ kubectl get deploy coredns -n kube-system -oyaml > bak/coredns-dp.yaml
$ kubectl get clusterrole coredns -n kube-system -oyaml > bak/coredns-cr.yaml
$ kubectl get clusterrolebinding coredns -n kube-system -oyaml > bak/coredns-crb.yaml

$ ./deploy.sh -s | kubectl apply -f -

$ kubectl get po -n kube-system

$ kubectl logs -f coredns-csegwe-zege -n kube-system

# 测试
$ kubectl get po
$ kubectl exec -it nginx-fegwe-egew -- bash
## 容器内命令
$$ curl kubernetes:443
```



