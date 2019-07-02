---
layout: post
title:  "cartographer安装笔记"
date:   2019-6-12 00:00:00
categories: ROS
tags: 工具
excerpt: 石师兄的安装笔记
mathjax: true
---
* content
{:toc}
---



> 文档链接：https://google-cartographer.readthedocs.io/en/latest/


## cartographer_ros安装

建议单独新建一个工作空间安装cartographer，例如`mkdir catkin_google_ws`安装在这个文件夹下。<br/>

### 1.安装依赖性


```
$ sudo apt-get install -y \
    clang \
    g++ \
    git \
    google-mock \
    libboost-all-dev \
    libcairo2-dev \
    libcurl4-openssl-dev \
    libeigen3-dev \
    libgflags-dev \
    libgoogle-glog-dev \
    liblua5.2-dev \
    libsuitesparse-dev \
    ninja-build \
    python-sphinx

```


---

### 安装ceres-solver

Ceres Solver是一个用于建模和解决大型复杂优化问题的开源C ++库。它是一个功能丰富，成熟且高性能的库，自2010年以来一直在被Google所应用。Ceres Solver可以解决两种问题：<br/>
	- 具有边界约束的非线性最小二乘问题。<br/>
	- 一般无约束优化问题。<br/>


在任意路径下：<br/>

`$ git clone https://github.com/ceres-solver/ceres-solver.git `<br/>
`$ mkdir ceres-solver/build` <br/>
`$ cd ceres-solver/build` <br/>
`$ cmake ..` <br/>
`$ make -j2` <br/>
`$ sudo make install` <br/>




---

### 安装cartographer

Cartographer是一个跨多个平台和传感器配置提供2D和3D实时同步定位和建图（SLAM）的系统。

`$ git clone https://github.com/googlecartographer/cartographer.git `<br/>
`$ mkdir cartographer/build`<br/>
`$ cd cartographer/build`<br/>
`$ cmake ..`<br/>
`$ make -j2`<br/>
`$ sudo make install`<br/>

或者使用浏览器从GitHub下载后安装

```
$ cd protobuf
$ mkdir build
$ cd build
$ cmake -G Ninja \
  -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
  -DCMAKE_BUILD_TYPE=Release \
  -Dprotobuf_BUILD_TESTS=OFF \
  ../cmake
$ ninja
$ sudo ninja install


```


---


### 安装cartographer_ros

catkin_google_ws/src目录下：

`$ git clone https://github.com/googlecartographer/cartographer_ros.git` <br/>
`$ cd ..` <br/>
`$ catkin_make` <br/>


---

## 使用测试数据运行cartographer


### 运行2D测试数据

`$ wget -P ~/download/slam_data/catographer https://storage.googleapis.com/cartographer-public-data/bags/backpack_2d/cartographer_paper_deutsches_museum.bag`<br/>


`$ roslaunch cartographer_ros demo_backpack_2d.launch bag_filename:= ~/download/slam_data/ cartographer /cartographer_paper_deutsches_museum.bag`<br/>


### 运行3D测试数据

`$ wget -P ~/download/slam_data/cartographer https://storage.googleapis.com/cartographer-public-data/bags/backpack_3d/with_intensities/b3-2016-04-05-14-14-00.bag` <br/>

`$ roslaunch cartographer_ros demo_backpack_3d.launch bag_filename:= ~/download/slam_data/cartographer/ b3-2016-04-05-14-14-00.bag` <br/>

