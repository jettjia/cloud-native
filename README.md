```
云原生方案

1.运维篇；CI/CD，持续集成【已完结】
	知识点：gitlab, jenkins, docker, harbor
	
2.k8s
	基础篇，单机、高可用集群部署【已完结】
		知识点：k8s二进制高可用集群安装
	
	进阶和高级篇，从0到k8s架构 【已完结】
		知识点：
		基础组件介绍(APIServer、Schedule、Controller Manager、ETCD、Kubelet、Kube-proxy);
		Pod知识
		资源调度：Deployment、StatefulSet、DaemonSet
		服务发布：Label && Selector、Service、Ingress、HPA
		配置管理：ConfigMap、Secret、ConfigMap&Secret使用SubPath、ConfigMap&Secret 热更新、ConfigMap&Secret 不可变
		存储：Volumes、PV&PVC
		高级调度：CronJob、污点Taint和容忍Toleration、Init Container、Affinity、Topology拓扑域、临时容器
		RBAC：基于角色（Role）的访问控制
		准入控制：LimitRange、ResourceQuot
		Qos：QoS类是Kubernetes用来决定Pod的调度和驱逐的策略
		PodPreset: 将一些公用的参数设置到pod中去，例如 时区统一设置为东八区
		Ceph: Rook、Ceph
		Helm: Kubernetes 的包管理器
		k8s-elk: elk日志收集
		Prometheus: k8s集群监控
		Ingress: k8s, ingress使用
		
3.istio 【已完结】
	知识点：istio基础知识介绍、组件介绍：Proxy、Mixer、Pilot、Citadel、Galley;
	httpbin案例、Bookinfo案例、灰度发布、可视化监控

4.实践篇-go【规划中...】

5.实践篇-php【规划中...】

6.实践篇-java 【已完结】
	知识点：
		springcloud
		gitlab
		docker
		k8s集群
		harbor
		jenkins-master, jenkins-slave
		CI/CD
		
7.中间件集群篇，【更新中...】
    mysql-cluster
    redis-cluster 【已完结】
    rabbitmq-cluster 【已完结】
    es-cluster 【已完结】
    clickhouse-cluster

8.k8s&istio集群管理平台 【规划中...】
	项目是Kubernetes资源管理平台，基于API开发，可以管理k8s的Deployment、DaemonSet、StatefulSet、Service、Ingress、Pods、	  Nodes、Role、Secret、ConfigMap、PV、PVC等.
	istio管理后台，灰度发布后台。

9. kubernetes 权威指南整理笔记 【更新中...】
```

