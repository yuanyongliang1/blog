---
title: 指令信息
date: 2023-09-16 14:01:56
tags: 
 - 指令
categories: 
 - 加密
password: yuan
abstract: 这里有一些加密的东西,密码需要继续阅读。
message: 加密文章，请向管理员索取密码
wrong_pass_message: 密码无效。请检查一下再试一次。
---

### 服务器指令：

#### 防火墙指令：

查询指定端口是否已开 firewall-cmd --query-port=666/tcp

查询所有开启的端口  firewall-cmd --list-port

查看版本： firewall-cmd --version

显示状态： firewall-cmd --state

开启防火墙 systemctl start firewalld

(若遇到无法开启先用：systemctl unmask firewalld.service

然后：systemctl start firewalld.service)

关闭防火墙 systemctl stop firewalld

开启端口命令  

添加  firewall-cmd --zone=public --add-port=8001/tcp --permanent

重新载入  firewall-cmd --reload

查看某个端口是否开启    firewall-cmd --zone= public --query-port=8001/tcp

删除    firewall-cmd --zone= public --remove-port=80/tcp --permanent



#### nacos指令：

查看nacos集群启动nacos个数

ps -ef |grep nacos |grep -v grep|wc -l

复制cluster.conf.example并重命名为cluster.conf

cp cluster.conf.example cluster.conf



#### nginx指令:

(sbin目录下)

查看nginx版本号:	./nginx -v

关闭nginx:	./nginx -s stop

启动nginx:	./nginx

重新加载：./nginx -s reload

#### mq指令：

安装mq后测试发送消息和接收消息

发送消息方：

​	1.设置环境变量

​	export NAMESRV_ADDR=localhost:9876

​	2.使用安装包的Demo发送消息

​	sh tools.sh org.apache.rocketmq.example.quickstart.Producer

接收消息方：

​	1.设置环境变量

​	export NAMESRV_ADDR=localhost:9876

​	2.接收消息	

​	sh tools.sh org.apache.rocketmq.example.quickstart.Consumer

### cmd指令

#### cmd查看WiFi密码

展示连接所有的wifi：netsh wlan show profiles

查看连接wifi密码：netsh wlan show profiles name="wifi名称" key=clear

