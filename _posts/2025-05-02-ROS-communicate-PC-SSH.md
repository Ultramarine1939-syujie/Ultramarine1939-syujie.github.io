---
layout: post
title: PC与ROS机器人SSH通信以及多机通信设置
subtitle: Each post also has a subtitle
categories: ROS
tags: ROS SSH
---

# PC与ROS机器人SSH通信以及多机通信设置

使用实例：用自己的电脑连接其他设备查看Rviz

由于目前我的rviz是开在我自己电脑上的，它需要一个ros master才能正常开启，要解决这个问题只要解决让它识别到一个局域网中的ros master并将其连接进去即可。方式也很简单，ROS是有自己的主从机机制的，它允许设备将自己当成主机也允许设备将自己当成从机。此时只要将设备配置成主机将电脑配置成从机即可。 

#### 通过SSH的方式实现PC和ROS机器人之间的通信

第一步，使PC和小车处于同一局域网，并获取各自的IP，在终端通过`ifconfig`命令查看ip。

第二步，修改目录/etc/hosts文件

```shell
sudo vim /etc/hosts
```

在PC端添加ROS机器人的ip和hostname,这里的hostname最好通过命令`hostname`查看

```shell
ip   hostname
```

同理，在机器人端添加PC的ip和hostname

最后双端可以通过SSH命令互相访问

```shell
ssh username@<ip>
```

#### 主从机设置

在上面设置好hosts的基础上(不设置hosts的话能实现多机查看话题列表，但不能进行消息的多机通信),在从机端，一般是PC端，设置：

```shell
echo "export ROS_HOSTNAME=本机ip" >> ~/.bashrc
echo "export ROS_MASTER_URI=主机ip" >> ~/.bashrc
```

或者直接编辑.bashrc，如下所示：

```shell
export ROS_MASTER_URI=http://192.168.233.175:11311	#机器人的IP
export ROS_HOSTNAME=192.168.233.130	#PC的IP

# >>> fishros initialize >>>
 source /opt/ros/noetic/setup.bash 
# <<< fishros initialize <<<
```

这时候在自己的电脑上打开rviz就可以正常启动并使用相关功能

不需要从机的时候需要在.bashrc文件中注释掉这两行，不然无法正常运行

SSH和ROS的本身的多机通信是有着区别的，建议在实际使用时(如建图时)PC控制ROS机器人结合使用，此外，通过vnc、scp、ftp等手段也可实现PC对ROS机器人的远程控制，文件传输