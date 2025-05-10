---
layout: post
title: 如何在 Ubuntu 中创建、删除和调整 SWAP 空间
subtitle: Each post also has a subtitle
categories: ubuntu
tags: ubuntu 
---

# 如何在 Ubuntu 中创建、删除和调整 SWAP 空间

转载自：https://www.sysgeek.cn/ubuntu-swap-size/

## 查看 SWAP 空间

在开始创建之前，请先使用以下命令检查您的 Ubuntu 系统是否已启用 SWAP 空间：

```shell
sudo swapon --show
```

该列表会包含所有的 SWAP 空间，包括 **SWAP 分区**和 **SWAP 文件**。如果输出为空，则表示当前系统尚未启用 SWAP 空间。

## 创建 SWAP 文件

与 SWAP 分区相比，SWAP 文件具有一个重要的优势，即文件大小可以轻松调整，而无需触及磁盘分区来更改交换空间的大小。

在本节中，我们将创建一个新的 SWAP 文件，并将其添加到当前的交换池中。

1、在「终端」中使用以下命令创建一个空白文件：

```shell
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
```

- 文件大小计算为 1M X 2048 = 2G，要创建不同大小的文件，请更改相应的`count`参数值。
- `/dev/zero`是 Linux 系统中的一个特殊块设备，在每次读取时输出零字节。
- 可以使用其他工具（例如 fallocate）来创建文件，但在某些情况下，可能会引起问题。更详细的讨论可以在[这篇 AskUbuntu 帖子](https://askubuntu.com/questions/927854/how-do-i-increase-the-size-of-swapfile-without-removing-it-in-the-terminal/1177620#1177620)中找到。

2、使用以下命令设置正确的文件权限：

```shell
sudo chmod 600 /swapfile
```

3、使用使用`mkswap`实用程序将文件格式化为 SWAP 空间：

```shell
sudo mkswap /swapfile
```

4、使用以下命令激活 swap 文件并将其添加到交换池中：

```shell
sudo swapon /swapfile
```

5、要让创建好的 SWAP 空间永久生效，需要将 swapfile 路径内容写入到`/etc/fstab`文件当中：

```shell
/swapfile swap swap defaults 0 0
```

6、使用`swapon`或`free`命令验证 SWAP 文件是否处于活动状态，如下所示：

```
sudo swapon --show
##或者
sudo free -h
```

## 调整 Swappiness 值

Swappiness 是 Linux 内核的一个属性，用于定义 Linux 系统使用 SWAP 空间的频率。`swappiness`值的范围是`0` 到`100`，较低的值会尽量减少内核对 SWAP 空间的使用，而较高的值会使 Linux 内核更积极地使用 SWAP 空间。

Ubuntu 系统的默认 Swappiness 值为`60`，您可以使用以下命令进行查看：

```shell
cat /proc/sys/vm/swappiness
```

值为`60`对于 Ubuntu Desktop 来说还可以，但对于 Ubuntu Server来说，SWAP 的使用频率就比较高了，所以您可能需要将值设置得较低一些。例如，要将`swappiness`值设置为`40`，请执行以下命令：

```shell
sudo sysctl vm.swappiness=40
```

如果要让设置在系统重启后依然有效，需要在`/etc/sysctl.conf`文件中添加以下内容：

```shell
vm.swappiness=40
```

最佳的 swappiness 值取决于 Ubuntu 系统的工作负载和内存使用方式，您应该逐渐调整这个参数，以找到最佳取值。

## 删除 SWAP 文件

要停用并删除 SWAP 文件，请按照下列步骤操作：

1、在「终端」中运行以下命令停用 SWAP 文件：

```shell
sudo swapoff -v /swapfile
```

2、在`/etc/fstab`文件中删除 swap 相关的行。

3、最后执行以下命令删除 swapfile 文件：

```shell
sudo rm /swapfile
```

## 调整 SWAP 空间大小

根据 SWAP 空间的类型（分区或文件），调整大小的过程可能会有所不同。

### 调整 SWAP 分区大小

如果分区后面有未分配的空间，才能扩展分区大小。否则，唯一的调整选项是缩小分区大小。这也同样适用于 SWAP 分区。

如果您使用的是 [GNOME 桌面环境](https://www.sysgeek.cn/linux-desktop-environments-inventory/)，「磁盘」应用程序可以提供相关信息；或者，可以使用 [GParted](https://gparted.org/) 来进行可视化。

在系统中，如果交换分区直接紧邻根分区。就没有空间来扩展 SWAP 分区了。

### 调整 SWAP 文件大小

1、要操作 SWAP 文件，请运行以下命令将其从交换池中移除：

```shell
sudo swapoff /swapfile
```

2、重新运行`dd`命令来增加文件的大小：

```shell
sudo dd if=/dev/zero of=/swapfile bs=1G count=2 oflag=append conv=notrunc
```

3、在这里，我们将交换文件增加到 2GB。接下来，使用以下命令将文件重新格式化为交换空间：

```shell
sudo mkswap /swapfile
```

4、将其作为交换文件启用：

```shell
sudo swapon /swapfile
```

