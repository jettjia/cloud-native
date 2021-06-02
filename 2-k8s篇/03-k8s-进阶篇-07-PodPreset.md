# 实践篇-PodPreset

## 1. PodPreset 的作用

将一些公用的参数设置到pod中去，例如 时区统一设置为东八区等

## 2. API Server 开启PodPreset

- 编辑文件 /etc/kubernetes/manifests/kube-apiserver.yaml，添加配置 --runtime-config=settings.k8s.io/v1alpha1=true，添加PodPreset到--admission-control(新版本是--enable-admission-plugins)
- 重启kubelet服务，sudo systemctl restart kubelet

## 3. 部署统一时区的PodPreset

yaml文件如下：

```shell
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: setting-timezone
spec:
  selector:
    matchLabels:
  env:
    - name: TZ
      value: Asia/Shanghai
```

其中 selector、matchLabels是必须的，不写任何的值就代表全局启用。

## 4. 禁用PodPreset

在一些情况下，用户不希望 Pod 被 Pod Preset 所改动，这时，用户可以在 Pod spec 中添加形如 podpreset.admission.kubernetes.io/exclude: "true" 的注解。
