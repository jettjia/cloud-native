# redis安装
注意事项:
> 1.安装redis前请先安装好docker。
> 2.文档中IP地址为示意地址，安装时请替换为实际生产地址。
> 3.本文档不要一次性执行一个命令框（灰色框）内的全部命令，应按照步骤说明分步执行。

## 准备工作

规划机器，redis版本5.0.10，操作系统版本centos7.5+
~~~bash
192.168.2.11 #安装2个redis实例，端口为7000，7001
192.168.2.12 #安装2个redis实例，端口为7000，7001
192.168.2.13 #安装2个redis实例，端口为7000，7001
~~~

设置主机hostname

~~~bash
#机器192.168.2.11执行
hostnamectl --static set-hostname  redis-11

#机器192.168.2.12执行
hostnamectl --static set-hostname  redis-12

#机器192.168.2.13执行
hostnamectl --static set-hostname  redis-13
~~~
为每台机器创建redis账号，并设置密码
~~~bash
groupadd redis
useradd -g redis redis
passwd redis #修改密码为redis@1234，生产环境修改为其他密码
echo 'redis     ALL=(ALL)NOPASSWD:ALL' >> /etc/sudoers
sed -i 's/Defaults    requiretty/#Defaults    requiretty/g' /etc/sudoers
cat  /etc/sudoers
~~~
为每台机器规划目录

~~~bash
sudo mkdir -p /app/redis
sudo chown -R redis:redis /app/redis/
~~~

## 源码安装

*注意：以下操作在3台机器上都要执行*

**下载地址：**http://redis.io/download

```bash
#下载稳定版5.0.10
wget https://download.redis.io/releases/redis-5.0.10.tar.gz
#内网地址
wget http://10.7.102.125:8000/downloads/redis-5.0.10.tar.gz
#解压缩，并编译
tar -zxvf redis-5.0.10.tar.gz -C /app/redis
cd /app/redis/redis-5.0.10
make

#如果遇到无法编译的情况之下如下命令
yum -y install gcc gcc-c++ libstdc++-devel
make CFLAGS="-march=x86-64"
```

执行完 **make** 命令后，redis-5.0.10 的 **src** 目录下会出现编译后的 redis程序redis-server, redis-cli,redis-sentinel等，把它们拷贝到/usr/local/bin下，执行如下命令

~~~bash
cp /app/redis/redis-5.0.10/src/redis-server /usr/local/bin/
cp /app/redis/redis-5.0.10/src/redis-cli /usr/local/bin/
cp /app/redis/redis-5.0.10/src/redis-sentinel /usr/local/bin/
cp /app/redis/redis-5.0.10/src/redis-trib.rb /usr/local/bin/
cp /app/redis/redis-5.0.10/src/redis-benchmark /usr/local/bin/
cp /app/redis/redis-5.0.10/src/redis-check-aof /usr/local/bin/
cp /app/redis/redis-5.0.10/src/redis-check-rdb /usr/local/bin/
~~~

## 分片集群安装
**第一步 创建实例** 

*注意：以下操作在3台机器上都要执行*

节点7000
~~~bash
#7000
mkdir -p /app/redis/cluster/7000
cp /app/redis/redis-5.0.10/redis.conf /app/redis/cluster/7000/redis.conf
~~~
~~~bash
vi /app/redis/cluster/7000/redis.conf

#修改如下内容
#bind 127.0.0.1
protected-mode no
port  7000
daemonize    yes                       //redis后台运行
pidfile  /var/run/redis_7000.pid
cluster-enabled  yes                   //开启集群
cluster-config-file  nodes_7000.conf   //集群的配置
cluster-node-timeout  15000            //请求超时  默认15秒，可自行设置
appendonly  yes                        //aof日志开启，它会每次写操作都记录一条日志
appendfilename "appendonly7000.aof"
dbfilename dump7000.rdb
~~~


节点7001
~~~bash
#7001
mkdir -p /app/redis/cluster/7001
cp /app/redis/redis-5.0.10/redis.conf /app/redis/cluster/7001/redis.conf
~~~
~~~bash
vim /app/redis/cluster/7001/redis.conf

#修改如下内容
#bind 127.0.0.1
protected-mode no
port  7001
daemonize    yes                       //redis后台运行
pidfile  /var/run/redis_7001.pid
cluster-enabled  yes                   //开启集群
cluster-config-file  nodes_7001.conf   //集群的配置
cluster-node-timeout  15000            //请求超时  默认15秒，可自行设置
appendonly  yes                        //aof日志开启，它会每次写操作都记录一条日志
appendfilename "appendonly7001.aof"
dbfilename dump7001.rdb
~~~

编辑启动脚本（7001同样操作编辑start.sh）

~~~bash
mkdir /app/redis/cluster/7000/bin/
vim /app/redis/cluster/7000/bin/start.sh
~~~
~~~bash
#!/bin/bash

cd /app/redis/cluster/7000/
/usr/local/bin/redis-server /app/redis/cluster/7000/redis.conf
~~~

编辑停止脚本（7001同样操作编辑stop.sh）
~~~bash
vim /app/redis/cluster/7000/bin/stop.sh 
~~~
~~~bash
#!/bin/bash

kill  `ps -ef | grep redis-server| grep 7000 | awk '{print $2}'`
~~~


启动并检查服务

~~~bash
#启动实例7000，7001
sh /app/redis/cluster/7000/bin/start.sh
sh /app/redis/cluster/7001/bin/start.sh

#查看进程，出现如下提示说明实例启动成功
$ ps -ef|grep redis
root     26994     1  0 11:11 ?        00:00:00 /usr/local/bin/redis-server *:7000 [cluster]
root     27024     1  0 11:15 ?        00:00:00 /usr/local/bin/redis-server *:7001 [cluster]
~~~

**第二步 创建集群** 

在一台机器使用下面命令创建集群（注意，只能在一台机器上创建集群）

```bash
#执行创建集群命令
/usr/local/bin/redis-cli --cluster-replicas 1 --cluster create 192.168.68.131:7000 192.168.68.133:7000 192.168.68.136:7000 192.168.68.131:7001 192.168.68.133:7001 192.168.68.136:7001
```

如果出现如下提示说明集群创建成功

~~~bash
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
~~~

**第三步 集群设置密码**

~~~bash
#以下命令6个redis实例都要执行
/usr/local/bin/redis-cli -c -p 7000

#输入以上命令后执行如下命令
192.168.2.11:7000> config set masterauth Redis123
OK
192.168.2.11:7000> config set requirepass Redis123
OK
192.168.2.11:7000> auth Redis123
OK
192.168.2.11:7000> config rewrite
OK
192.168.2.11:7000> exit
~~~

安装完毕！