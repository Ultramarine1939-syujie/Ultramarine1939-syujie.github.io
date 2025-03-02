---
layout: post
title: Vins-fusion 安装教程
subtitle: Each post also has a subtitle
categories: vins
tags: vins
---

参考来源：https://github.com/Ultramarine1939-syujie/VINS-Fusion
### 使用方法

#### 安装ceres-solver1.14.0

1、安装依赖

```bash
sudo apt install libgoogle-glog-dev libatlas-base-dev libeigen3-dev libsuitesparse-dev liblapack-dev libcxsparse3 libgflags-dev libgtest-dev
```

2、下载ceres并解压 / 解压`3rd_party.zip`压缩包

```bash
#在压缩包在的目录下打开终端，输入如下命令解压
tar zxf ceres-solver-1.14.0.tar.gz
```

3、在解压后的同级目录下，编译安装

```bash
mkdir ceres-bin
cd ceres-bin
cmake ../ceres-solver-1.14.0
make -j8
sudo make install
```

#### 安装glog

- 解压`3rd_party.zip`压缩包
- 进入glog文件夹打开终端
- `./autogen.sh && ./configure && make && sudo make install`
- `sudo apt-get install liblapack-dev libsuitesparse-dev libcxsparse3.1.2 libgflags-dev libgoogle-glog-dev libgtest-dev`

#### 下载编译Vins-Fusion

1、创建工作空间vins/src，在src目录下下载Vins-Fusion

```bash
git clone https://github.com/HKUST-Aerial-Robotics/VINS-Fusion.git
```

2、在vins目录下编译

```bash
catkin_make
```

#### 运行VINS-Fusion

**Monocualr camera + IMU**

```shell
roslaunch vins vins_rviz.launch

rosrun vins vins_node ~/vins-fusion/src/VINS-Fusion/config/euroc/euroc_mono_imu_config.yaml 

(optional) rosrun loop_fusion loop_fusion_node ~/vins-fusion/src/VINS-Fusion/config/euroc/euroc_mono_imu_config.yaml 

rosbag play ~/SLAM/test_data/MH_01_easy.bag
```

**Stereo cameras + IMU**

```shell
roslaunch vins vins_rviz.launch

rosrun vins vins_node ~/vins-fusion/src/VINS-Fusion/config/euroc/euroc_stereo_imu_config.yaml 

(optional) rosrun loop_fusion loop_fusion_node ~/vins-fusion/src/VINS-Fusion/config/euroc/euroc_stereo_imu_config.yaml 

rosbag play ~/SLAM/test_data/MH_01_easy.bag
```

**Stereo cameras**

```shell
roslaunch vins vins_rviz.launch

rosrun vins vins_node ~/vins-fusion/src/VINS-Fusion/config/euroc/euroc_stereo_config.yaml 

(optional) rosrun loop_fusion loop_fusion_node ~/vins-fusion/src/VINS-Fusion/config/euroc/euroc_stereo_config.yaml 

rosbag play ~/SLAM/test_data/MH_01_easy.bag
```

**Stereo cameras + GPS**

```shell
roslaunch vins vins_rviz.launch

rosrun vins kitti_gps_test ~/catkin_ws/src/VINS-Fusion/config/kitti_raw/kitti_10_03_config.yaml YOUR_DATASET_FOLDER/2011_10_03_drive_0027_sync/ 

rosrun global_fusion global_fusion_node
```
