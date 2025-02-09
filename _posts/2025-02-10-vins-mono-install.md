---
layout: post
title: Vins-mono 安装教程
subtitle: Each post also has a subtitle
categories: vins
tags: vins
---
参考代码：https://github.com/Ultramarine1939-syujie/VINS-Mono

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

#### VINS-Mono代码修改

1.camera_model/include/camodocal/calib/CameraCalibration.h

```cpp
//add
#include <opencv2/imgproc/types_c.h>
#include <opencv2/imgproc/imgproc_c.h>
```

2.camera_model/include/camodocal/chessboard/Chessboard.h

```cpp
//add
#include <opencv2/imgproc/types_c.h>
#include <opencv2/calib3d/calib3d_c.h>
```

3.pose_graph/src/keyframe.h

```cpp
//add
#include <opencv2/highgui.hpp>
#include <opencv2/cvconfig.h>
```

4.pose_graph/src/pose_graph.h

```cpp
//add
#include <opencv2/highgui.hpp>
#include <opencv2/cvconfig.h>
```

5.pose_graph/src/ThirdParty/DVision/BRIEF.h

```cpp
//add
#include <opencv2/highgui.hpp>
#include <opencv2/cvconfig.h>
#include <opencv2/imgproc/types_c.h>
#include <opencv2/imgproc/imgproc_c.h>
```

#### 运行VINS-Mono

在catkin_ws目录下开三个命令行终端窗口

```shell
#第一个窗口
source devel/setup.bash
roslaunch vins_estimator euroc.launch 
#第二个窗口
source devel/setup.bash
roslaunch vins_estimator vins_rviz.launch
#第三个窗口
source devel/setup.bash
rosbag play MH_01_easy.bag
```