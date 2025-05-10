---
layout: post
title: Px4-Gazebo-Mavros联合仿真环境的搭建
subtitle: Each post also has a subtitle
categories: ROS
tags: ROS 
---

# Px4-Gazebo-Mavros

Px4-Gazebo-Mavros联合仿真环境的搭建（Ubuntu20.04+ROS noetic）

### ROS的安装（鱼香ROS）

```shell
wget http://fishros.com/install -O fishros && . fishros
```

参考文档：www.fishros.com

### QGC的安装（官网）

```shell
sudo usermod -a -G dialout $USER
sudo apt-get remove modemmanager -y
sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y
sudo apt install libqt5gui5 -y
sudo apt install libfuse2 -y
wget https://d176tv9ibo4jno.cloudfront.net/latest/QGroundControl.AppImage
sudo chmod 777 QGroundControl.AppImage
./QGroundControl.AppImage
```

参考文档：https://docs.qgroundcontrol.com/master/en/getting_started/download_and_install.html

### MAVROS安装配置

```shell
sudo apt-get update
sudo apt-get install python3 python3-pip ros-noetic-mavros* -y
cd /opt/ros/noetic/lib/mavros
sudo ./install_geographiclib_datasets.sh
```

### PX4的安装配置

```shell
mkdir -p ${HOME}/repos && cd ${HOME}/repos
git clone https://github.com/PX4/PX4-Autopilot.git -b v1.13.0 --recursive
cd PX4-Autopilot
bash ./Tools/setup/ubuntu.sh
```

重启电脑

```shell
cd ${HOME}/repos/PX4-Autopilot
DONT_RUN=1 make px4_sitl_default gazebo
source Tools/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)/Tools/sitl_gazebo
```

运行仿真

```shell
roslaunch px4 mavros_posix_sitl.launch
```

### LibRealsense安装配置（带T265，可选）

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

