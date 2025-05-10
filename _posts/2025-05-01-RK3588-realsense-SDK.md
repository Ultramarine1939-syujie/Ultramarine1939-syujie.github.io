---
layout: post
title: RK3588安装Realsense运行D435深度相机
subtitle: Each post also has a subtitle
categories: realsense
tags: realsense 
---

## RK3588安装Realsense运行D435深度相机

#### LibRealsense安装配置

LibRealsense安装（Intel® RealSense™ SDK 2.0）

```shell
git clone https://github.com/IntelRealSense/librealsense -b v2.53.1 --recursive
sudo apt-get install libssl-dev libusb-1.0-0-dev libudev-dev pkg-config libgtk-3-dev cmake -y
sudo apt-get install libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev at -y
cd librealsense
mkdir build && cd build
cmake ../
sudo make uninstall && make clean && make -j8
sudo make install
cd ..
./scripts/setup_udev_rules.sh
```

#### RealSense™ROS安装

```shell
git clone https://github.com/IntelRealSense/realsense-ros -b ros1-legacy --recursive
```

具体操作

1、Create a [catkin](http://wiki.ros.org/catkin#Installing_catkin) workspace *Ubuntu*

```
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src/
```

2、编译（以下为官方说明，实际操作中直接使用catkin_make也可正常工作）

- Make sure all dependent packages are installed. You can check .travis.yml file for reference.
- Specifically, make sure that the ros package *ddynamic_reconfigure* is installed. If *ddynamic_reconfigure* cannot be installed using APT or if you are using *Windows* you may clone it into your workspace 'catkin_ws/src/' from [here](https://github.com/pal-robotics/ddynamic_reconfigure/tree/kinetic-devel)

```shell
catkin_init_workspace
cd ..
catkin_make clean
catkin_make -DCATKIN_ENABLE_TESTING=False -DCMAKE_BUILD_TYPE=Release
catkin_make install
```

3、source

```shell
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

