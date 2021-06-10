# rabbitmq安装
注意事项:
> 1.安装rabbitmq前请先安装好docker，具体安装方法参考文档<<docker生产安装.md>>。
> 2.文档中IP地址为示意地址，安装时请替换为实际生产地址。
> 3.本文档不要一次性执行一个命令框（灰色框）内的全部命令，请按照步骤说明分步执行。
> 4.文档中主机名称mq-11，mq-12，mq-13也需要根据生产环境ip设置为对应的主机名称。


## 准备工作

*注意：以下操作都在3台机器上分别执行*

**规划机器，rabbitmq版本3.8.9**

~~~
192.168.68.11 mq-11
192.168.68.12 mq-12
192.168.68.13 mq-13
~~~

**设置主机hostname**

~~~
#机器192.168.68.11执行
hostnamectl --static set-hostname  mq-11

#机器192.168.68.12执行
hostnamectl --static set-hostname  mq-12

#机器192.168.68.13执行
hostnamectl --static set-hostname  mq-13
~~~
**为每台机器创建rabbitmq账号，并设置密码**

~~~
groupadd rabbitmq
useradd -g rabbitmq rabbitmq
#修改账号rabbitmq密码为rabbitmq@1234，生产环境请替换为其他密码
echo rabbitmq:rabbitmq@1234|chpasswd

echo 'rabbitmq     ALL=(ALL)NOPASSWD:ALL' >> /etc/sudoers
sed -i 's/Defaults    requiretty/#Defaults    requiretty/g' /etc/sudoers
cat  /etc/sudoers
~~~
**下载稳定版安装包**

官方地址：https://www.rabbitmq.com/download.html

```
#socat
wget http://www.rpmfind.net/linux/centos/7.9.2009/os/x86_64/Packages/socat-1.7.3.2-2.el7.x86_64.rpm

#epel
wget https://mirrors.tuna.tsinghua.edu.cn/epel/7/x86_64/Packages/e/epel-release-7-13.noarch.rpm

#erland
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v23.2.1/erlang-23.2.1-1.el7.x86_64.rpm

#rabbitmq
wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.8.9/rabbitmq-server-3.8.9-1.el7.noarch.rpm

#内网地址
wget http://10.7.102.125:8000/downloads/rabbitmq/socat-1.7.3.2-2.el7.x86_64.rpm
wget http://10.7.102.125:8000/downloads/rabbitmq/epel-release-7-13.noarch.rpm
wget http://10.7.102.125:8000/downloads/rabbitmq/erlang-23.2.1-1.el7.x86_64.rpm
wget http://10.7.102.125:8000/downloads/rabbitmq/rabbitmq-server-3.8.9-1.el7.noarch.rpm
```
## 安装步骤

*注意：以下操作在3台机器上都要执行*

**socat安装**

```
sudo yum -y install socat-1.7.3.2-2.el7.x86_64.rpm
```

**epel安装**

```
sudo yum -y install epel-release-7-13.noarch.rpm
```

**erlang安装**

```
sudo yum -y install erlang-23.2.1-1.el7.x86_64.rpm
```

**rabbitmq安装**

```
sudo rpm -ivh rabbitmq-server-3.8.9-1.el7.noarch.rpm
#验证是否成功：
sudo systemctl start rabbitmq-server 
sudo systemctl status rabbitmq-server
#停止服务：
sudo systemctl stop rabbitmq-server
```

同步.erlang.cookie文件，保持3台机器上.erlang.cookie内容一样

~~~
#备注：官方在介绍集群的文档中提到过.erlang.cookie一般会存在两个地址：
#第一个是$home/.erlang.cookie，第二个地方就是/var/lib/rabbitmq/.erlang.cookie
~~~

## 集群搭建

*注意：以下操作都在3台机器上分别执行*

1.修改/etc/hosts文件，添加IP和节点名称

~~~bash
#sudo vim /etc/hosts
~~~
~~~
sudo cat <<EOF >> /etc/hosts
192.168.68.11 mq-1
192.168.68.12 mq-2
192.168.68.13 mq-3
EOF

cat /etc/hosts
~~~

2.首先启动3个节点上的RabbitMQ服务

> 创建启动脚本

~~~bash
#/home/rabbitmq/startRabbitmq.sh
~~~
~~~
cat <<EOF >  /home/rabbitmq/startRabbitmq.sh
#!/bin/bash
sudo systemctl start rabbitmq-server
EOF
~~~

> 创建停止脚本

~~~bash
#/home/rabbitmq/stopRabbitmq.sh
~~~
~~~
cat <<EOF >  /home/rabbitmq/stopRabbitmq.sh
#!/bin/bash
sudo systemctl stop rabbitmq-server
EOF
~~~

> 创建查看状态脚本

~~~bash
#/home/rabbitmq/statusRabbitmq.sh
~~~
~~~
cat <<EOF >  /home/rabbitmq/statusRabbitmq.sh
#!/bin/bash
sudo systemctl status rabbitmq-server
sudo rabbitmqctl cluster_status
EOF
~~~

启动服务

```bash
#启动服务
./startRabbitmq.sh
#查看各节点的集群状态
./statusRabbitmq.sh
```

3.以mq-11为基准，将mq-12、mq-13加入到集群中，在机器mq-12，mq-13上操作如下命令(*以下操作只在12，13两台从机器上执行*)

```bash
sudo rabbitmqctl stop_app
sudo rabbitmqctl reset
sudo rabbitmqctl join_cluster rabbit@mq-1
sudo rabbitmqctl start_app

#故障节点剔除
rabbitmqctl  -n rabbit@mq-1  forget_cluster_node rabbit@mq-2
#重置节点数据，执行前停止rabbitmq
rm -rf /var/lib/rabbitmq/mnesia
```

4.检查集群状态(*以下操作在主服务器mq-11上操作*)

> 注意：如果关闭了集群中的所有节点，确保启动时最后一个关闭的节点第一个启动。

~~~bash
sudo rabbitmqctl cluster_status
#启动管理插件
sudo rabbitmq-plugins enable rabbitmq_management

#添加admin用户,否则用guest登陆会报User can only log in via localhost
sudo rabbitmqctl add_user admin admin
sudo rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
sudo rabbitmqctl set_user_tags admin administrator

#访问控制管理页面
http://192.168.2.11:15672/
~~~

## 设置镜像队列

*注意：以下操作只在主服务器mq-11上操作*

针对每一个镜像队列都包含一个master节点和多个slave节点。如果master不工作，那么镜像队列最早的salve会升级为master。

镜像队列的配置主要是通过添加相应的 Policy 来完成，命令如下 ：

```
rabbitmqctl set_policy [-p vhost] [--priority priority] [--apply-to apply-to] {name} {pattern} {definition}
```

> 对队列名称以*字母或数字*开头的所有队列进行镜像，并在集群的所有节点上完成镜像。

```shell
sudo rabbitmqctl set_policy -p /prod --priority 0 --apply-to queues mirror_queue "^[0-9a-zA-Z]" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
```

**验证**

使用新建的admin用户登录远程的机器 http://192.168.2.11:15672/

创建一个队列，例如以***"queue_"***开头，如果管理界面能看到队列Features状态显示为***“D HA”***说明镜像队列创建成功

安装完毕！


参考： https://mp.weixin.qq.com/s/IWFZBfyHaSsQFjX7oMa_Bg

