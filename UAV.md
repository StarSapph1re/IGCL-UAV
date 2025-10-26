# IGCL UAV 入门指南
> Version: 1.0  
> Author: hanyx  
> Contibutor: 

欢迎加入IGCL课题组，参与智能无人机方向的研究！本指南将带领你快速入门无人机算法的基础开发流程，也请你在阅读、环境配置和代码实现的过程中，对于其中的错误或不完善之处进行自己的修正与补充，当你按照本指南步骤操作，遇到指南中未列出的问题时，请务必将问题内容和解决的过程直接增补到本指南中。前人栽树，后人乘凉！  

前置要求：  
* 了解linux操作系统的使用方法  
* 掌握C++语言的基础语法与开发技能
* 掌握Python语言的基础语法与开发技能

配套材料（由于部分内部资料不方便公开到github，请联系实验室相关人员领取）：
* Amov 科研无人机代码 (p450_experiment, prometheus px4)

## Task1 搭建Linux开发环境  
现代机器人广泛使用ROS（Robot Operating System）作为核心框架，无人机也不例外。ROS目前对windows的支持较差，对无人机所有的开发都只能在Linux系统中进行。因此，首先你需要在电脑上配置Linux操作系统，由于ROS Noetic为本指南以及目前无人机上主要采用的ROS版本，安装Ubuntu操作系统时版本请务必选择**Ubuntu 20.04**。下面介绍两种配置方式及其优缺点。
#### 1. 单独安装Linux操作系统（建议使用）
在同一个磁盘上分配一定的空间，并在其中安装完整的Ubuntu系统，由于Airsim仿真环境的场景文件较大，这里推荐分配至少300GB以上，以防止磁盘空间紧张的情况。
优点：
* 图形界面更流畅，方便操作
* 仿真环境的渲染帧率高，体验更好
缺点：
* 需要大量磁盘空间
* 来回切换需要重启电脑
安装方式：参考视频

#### 2. 在Windows上安装WSL（不推荐）
利用微软提供的WSL，在Windows中直接运行Linux内核，这种方式可能有诸多问题，除非只想做一些初步的尝试，否则不推荐长期开发使用。
优点：
* 对磁盘空间需求相对较少
* 传输文件方便
缺点：
* 可能遇到原生Linux没有的问题
* 仿真环境渲染质量与流畅程度极低，甚至影响仿真运行
安装方式：
  
安装完成后，确认系统能够正常运行，推荐安装一些基础软件：
* 搜狗输入法
* QQ/Wechat for Linux
* VSCode
* 梯子
* ncdu（磁盘空间监测工具）  

请稍微熟悉一下Linux的操作方式，然后进入Task2。

## Task2 安装ROS
机器人操作系统（ROS，Robot Operating System）是一个专为机器人软件开发设计的开源框架，其核心目标是提供标准化工具和模块化设计，以促进代码复用和跨平台协作‌，将功能分解为独立的结点（Node）是其最鲜明的特征。每个节点通过发布-订阅（Pub/Sub）或服务（Service）机制通信，支持多设备分布式计算‌，例如传感器数据采集结点可与运动控制结点异步交互。ROS主要支持C++和Python语言，对Java等语言也有一定兼容，请至少熟练掌握其中一种语言的基础开发技能。  
目前ROS有两个大版本：ROS1和ROS2，其中ROS1更加稳定，主要用于学术研究；ROS2则因为有更好的特性、支持更多平台，一般用于工业界。ROS1的版本代号按字母顺序命名，每两年发布一个‌长期支持版本（LTS）‌，主要区别在于依赖的Ubuntu版本和功能更新，例如ROS Melodic （Ubuntu 18.04），ROS Noetic（Ubuntu 20.04），现在科研无人机主要采用ROS Noetic。    
现在，你需要在Ubuntu系统上安装ROS Noetic的开发环境，推荐使用鱼香ROS社区提供的一键安装程序，详细步骤如下。
#### 1. 更换系统源
```
wget http://fishros.com/install -O fishros && . fishros
```
在终端中执行以上命令，并依次选择5，2，1.
如果出现证书、数字签名无效等问题，执行以下命令。
```
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```

#### 2. 安装ros-noetic
```
wget http://fishros.com/install -O fishros && . fishros
```
执行以上命令，并依次选择1，2，3，1.
#### 3. 验证安装
打开一个终端，输入以下命令启动ros核心。
```
roscore
```
再打开一个终端，运行海龟仿真结点。
```
rosrun turtlesim turtlesim_node
```
如有正常画面显示（小海龟），则ros安装成功，进入Task3。

## Task3 安装Airsim仿真环境
AirSim是由微软开源的跨平台高保真仿真平台，基于Unreal Engine‌ 4构建，专注于无人机和自动驾驶车辆的物理与视觉仿真‌。核心优势包括：  
* ‌跨平台部署‌：支持 Windows/Linux/macOS，并提供 Python/C++ API 接口‌。
* 逼真视觉渲染‌：利用游戏引擎实现光照、天气（雨雪雾）、动态场景等真实效果，适合视觉算法验证‌。
* 多模态支持‌：兼容无人机（PX4/ArduPilot飞控）和车辆模式，提供传感器数据（摄像头、激光雷达、IMU等）‌。  
注意，Airsim的仿真对显卡的渲染能力有一定需求，在RTX4080下渲染非常流畅，更低的显卡配置表现未知，**欢迎补充**。
首先，你需要按照官方文档在Ubuntu上进行【Unreal Engine 4.27】的源代码编译。
文档：

参考这篇博客安装Airsim：

启动Unreal Engine并进入Airsim提供的测试场景Block，点击运行后，如果出现无人机，则说明安装成功。启动仿真以后，默认视角会跟随无人机移动，依次按下B和M键可以进入自由视角，此时按WASD和方向键可以控制视角在场景中自由移动。在Task x之前我们不会使用Airsim作为实验的仿真环境，但请尽早确认Airsim在你的电脑上的渲染表现，如果渲染的帧率无法接受或有明显卡顿现象，请尽早考虑自行更换或向实验室申请配置更好的设备。在此期间，你依然可以使用这台设备的环境进行学习并完成后续任务。确认仿真能稳定运行后，进入Task 4。

## Task4 学习ROS开发
在学习之前，先简要介绍ROS的两个核心特性，时刻保持对两个特性的意识有利于在开发前对系统进行更好的设计，以及开发时根据问题进行动态调整，下面的介绍完全根据个人开发的心得，如有错误欢迎修正。
#### 结点分布
ROS的运行时其实是启动了若干个结点，有一个Master结点作为ROS的核心管理者，也就是通过roscore启动的结点。这些结点都是完全独立运行自己的逻辑，他们的代码可以由不同的语言编写，也可以分布在不同的设备上。例如在无人机避障的任务中，结点A负责读取雷达的数据，结点B负责将雷达电云转换为体素地图，结点C负责根据当前的地图生成飞行轨迹。因此，在开发过程中，请事先做好对任务的拆分和结点的设计，避免单一结点执行过多逻辑，给系统带来额外的复杂度。当结点数量多时，推荐画图来明确各自的作用和协作关系。
#### 话题通讯
ROS结点之间通过话题（Topic）进行通讯，每一个话题的要素包括发布者（Publisher）、订阅者（Subscriber）以及话题的消息类型（Message Type）。结点的核心逻辑包括主循环（spinning）和回调函数（Callback）。主循环是结点在【没有收到话题消息时】所执行的逻辑，例如避障任务中结点A在主循环中不断根据当前生成的轨迹来计算下一时刻无人机所执行的飞行指令。同时，结点会为监听的每一个话题注册回调函数（Callback），来设置【收到这一话题的消息时】所执行的逻辑，结点会有单独的线程监听话题时，当新的话题消息收到时，这个线程通常会基于话题消息作一些处理，例如避障任务中结点A注册回调函数trajectory_callback来收集planner规划的最新一条轨迹，从而在主循环中根据最新的轨迹来生成下一时刻命令。另外，也有负责【每隔一定时间】执行某些逻辑的计时器（Timer），其回调函数在计时器到了预订时间时执行一次。在开发时，根据结点所需的数据设计所订阅的话题，并根据结点的功能将处理后的结果通过话题发送给下一个结点供其使用。

现在，你可以自行寻找教程学习，这里强烈推荐古月居的ROS网课，按照他的流程写一遍，你应该能够掌握后续Task所需的基础开发技能。
视频：
请确保自己已经熟悉如何在ROS中实现以下功能后，进入Task5：
* 创建一个新的功能包
* 在功能包中创建结点（C++/rospy）
* 定义新的话题消息类型
* 令结点发布与订阅自定义消息类型的话题
* 通过roslaunch为结点的启动配置参数
* 编译、运行与调试自己写好的结点
* ...

## Task5 配置阿木实验室仿真环境
Task5－Task8，采用的是阿木实验室提供的【P450】科研无人机，这款飞机为我实习期间飞行使用，也是比较有代表性的科研用无人机，如果有时间，可以看一下这款飞机的文档。【P450文档链接】虽然机型与目前实验室使用的中航恒拓的【OWL 3L】不同，但阿木实验室提供了较成熟的、容易部署的开源仿真环境，且撰写此文档时尚未调通OWL 3L的仿真，因此还是将阿木实验室的飞机用作本指南的仿真用机，之后可能会更新OWL 3L在Airsim等仿真平台的仿真步骤。阿木实验室提供的仿真代码基于gazebo仿真器，下面先进行简单介绍。

#### gazebo仿真环境介绍
gazebo是开源的机器人仿真平台，支持机械臂、轮式机器人、无人机等多种机器人，采用多种物理引擎对刚体动力学、碰撞监测、摩擦等物理特性进行模拟，并支持各类传感器（雷达、深度相机、IMU）等等。gazebo与ROS有良好的集成，通过ROS进行数据交换与机器人控制。相比Airsim，gazebo的数据收集，例如深度渲染等快很多，对硬件要求相对较低，但场景的渲染质量远不如Airsim真实，更加适合较low－level的、对物理层仿真要求较高的算法（例如避障算法）模拟，而Airsim提供接近真实世界的图像渲染，更适合为high－level的算法（例如Visual Language Action）提供更真实的图像数据。
【仿真画面】

> 随着版本更新，以下两部分钟官方文档内容可能与配套材料代码不符，优先参考本指南。  
> 这两部分之前做的时候没有详细记录每一步，**请下一个使用本指南的同学记录一下自己的安装过程**。

### 编译P450无人机 机上代码
这部分代码见配套材料 /p450_experiment，无报错运行编译脚本compile.sh即安装成功，以下为编译之前的前置步骤以及可能遇到的一些问题。
#### 1. 安装 `multi_map_server`

```bash
sudo apt-get install ros-noetic-multi-map-server
sudo apt-get install ros-noetic-ddynamic-reconfigure
```

#### 2. 安装 CUDA、cuDNN 和 TensorRT
参考nvidia驱动 550.144.03
请参考以下链接进行安装：
[安装 CUDA, cuDNN 和 TensorRT](https://blog.csdn.net/Apple_Coco/article/details/129293019)

Unbuntu20.04中，建议cuda选择run文件安装，cudnn选择直接下载archive，tensorrt下载tar包（不建议采用deb安装文件，会有诸多问题）
TensorRT配置bashrc：
```
export TENSORRT_DIR=/usr/local/TensorRT-8.6.1.6
export PATH=${TENSORRT_DIR}/bin:$PATH
export LD_LIBRARY_PATH=${TENSORRT_DIR}/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=${TENSORRT_DIR}/lib:$LIBRARY_PATH
export CUDNN_DIR=/usr/local/cuda
```

#### 3. 安装 OpenCV 4.7.0

安装 OpenCV 4.7.0 时，务必确保同时安装 `opencv_contrib`，否则会出现如下错误：  
`#include <opencv2/aruco.hpp>` 出错。

参考文章：[安装 OpenCV 4.7.0](https://blog.csdn.net/wxyczhyza/article/details/128968849)  
安装 `opencv_contrib` 参考链接：[腾讯云文章](https://cloud.tencent.com/developer/article/1491605)

编译 OpenCV 时，需要使用以下 `cmake` 配置命令：

```bash
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_GENERATE_PKGCONFIG=ON \
      -D INSTALL_PYTHON_EXAMPLES=ON \
      -D INSTALL_C_EXAMPLES=ON \
      -D OPENCV_EXTRA_MODULES_PATH=/path/to/opencv_contrib/modules \
      -D OPENCV_EXAMPLES=ON ..
```

#### 4. 编译时遇到 `float32_t` 未定义的问题

在编译过程中，如果遇到 `float32_t` 未定义的错误，定位到文件 `p450_experiment/src/spirecv-ros/sv-rosapp/samples/gimbal_server.cpp`，直接将 `float32_t` 替换为 `float`，目前没有发现其他问题。

#### 5. 安装 RealSense SDK

克隆 RealSense SDK 仓库并进行编译安装：

```bash
git clone -b v2.55.1 https://github.com/IntelRealSense/librealsense --recursive
cd librealsense
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/home/<usrname>  # 必须安装到自己用户名对应的目录下
make
sudo make install
```

安装完成后，修改 `realsense2Config.cmake` 配置文件：

```bash
# 修改文件 /<path-to>/librealsense/build/realsense2Config.cmake
# 将第 35 行更改为：
include("/home/<usrname>/lib/cmake/realsense2/realsense2Targets.cmake")
```

如果还是报错找不到librealsense，终端中输入：
```
export CMAKE_PREFIX_PATH=/path/to/realsense:$CMAKE_PREFIX_PATH
```

#### 6. 安装 Miniconda

```bash
mkdir -p ~/build_env/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/build_env/miniconda3/miniconda.sh
bash ~/build_env/miniconda3/miniconda.sh -b -u -p ~/build_env/miniconda3
rm ~/build_env/miniconda3/miniconda.sh
```

#### 7. 解决 `catkin_make / cmake` 找不到 OpenCV 路径问题

在 `catkin_make` 或 `cmake` 时，如果出现找不到 OpenCV 路径的错误，可以通过以下命令指定 OpenCV 路径：

```bash
export OpenCV_DIR=/path/to/opencv-4.7.0/build
```

注意：不要同时在 WSL 中使用 `sudo apt-get` 安装的 OpenCV 和自己编译的 OpenCV，可能会导致头文件定义冲突。若出现此问题，卸载 `sudo apt-get` 安装的 OpenCV：

```bash
sudo apt-get remove libopencv-*
```

卸载之后因为依赖问题会把ros的一些包也卸载可以，重新用小鱼的安装器进行安装。

如果出现/usr/bin/ld: 找不到 -lnvinfer，

#### 8. 安装`cv_bridge`

如果需要在 Python 中使用 `cv_bridge`，可以按照以下步骤进行安装：

```bash
git clone -b noetic https://github.com/ros-perception/vision_opencv.git --recursive
cd vision_opencv/cv_bridge
pip setup.py install
```

#### 可能遇到的其他问题

如果在编译带有 Python 脚本的功能包时，遇到以下错误：

```
AttributeError: module 'em' has no attribute 'RAW_OPT'
```

可以通过安装指定版本的 `empy` 来解决：

```bash
pip install empy==3.3.4
```

### 配置Prometheus仿真环境
Prometheus为阿木实验室基于gazebo仿真器进行一定包装而成，本质上还是gazebo，只是加了一些场景元素和无人机模型，可以参考其文档【文档链接】。


接下来，你需要按照官网 [Prometheus 仿真环境配置](https://wiki.amovlab.com/public/prometheus-wiki/%E5%BF%AB%E9%80%9F%E4%B8%8A%E6%89%8B/Prometheus%E4%BB%BF%E7%9C%9F%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE_Ubuntu/prometheus_px4%E9%85%8D%E7%BD%AE.html) 进行配置。配置本指南所用的环境时，**建议关闭anaconda激活的环境，避免交叉污染问题**。  
以下记录编译过程中遇到的问题，以供参考，欢迎补充。

①错误：`No module named 'catkin_pkg'`
这部分会用到系统的python，所以直接安装这个包即可
```
pip install catkin_pkg
```
②错误：`/usr/include/boost/thread/pthread/thread_data.hpp:60:5: error: missing binary operator before token "("`
解决方案：

找到 `thread_data.hpp` 文件，定位到以下内容：

```cpp
#if PTHREAD_STACK_MIN>0
```

将其替换为：

```cpp
#ifdef PTHREAD_STACK_MIN
```

③编译Gazebo仿真模块时，错误：`No rule to make target '/usr/lib/x86_64-linux-gnu/libpthread.so'`


```bash
sudo apt-get install libpthread-stubs0-dev
```

检查 `/usr/lib/x86_64-linux-gnu/` 目录，确保存在 `libpthread.so.0`，如果缺少符号链接，可以手动创建：

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libpthread.so.0 /usr/lib/x86_64-linux-gnu/libpthread.so
```

④错误：`No rule to make target '/usr/lib/x86_64-linux-gnu/libdl.so'`
解决方案：

检查 `/usr/lib/x86_64-linux-gnu/` 目录，确保存在 `libdl.so.2`，如果缺少符号链接，可以手动创建：

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libdl.so.2 /usr/lib/x86_64-linux-gnu/libdl.so
```

⑤编译Fast－LIO模块时，错误：`fatal error: fast_lio/Pose6D.h: No such file or directory`
此问题是由于没有添加依赖，导致 C++ 编译时找不到消息文件。修改 `CMakeLists.txt` 文件，在 `add_executable` 行下添加：

```cmake
add_dependencies(fastlio_mapping ${PROJECT_NAME}_generate_messages_cpp)
```

在编译目标检测模块前，你需要安装SpireCV和流媒体服务器。

克隆SpireCV代码库，在 SpireCV 主目录下运行 `scripts/x86-intel/zlm-server-install.sh` 来安装流媒体服务器。只有在安装完 ZLM 后，才能实现图像推流。

#### 启动流媒体服务器

ZLM server在仿真中不是必需的，安装完 ZLM 后，运行 `SpireCV/ZLM/MediaServer` 启动流媒体服务器。默认端口为 `554`。由于权限问题，需要使用 `sudo` 来启动 MediaServer，或者可以配置命令不需要 `sudo`：

1. 执行 `sudo visudo`
2. 添加以下行：

```bash
<username> ALL=(ALL) NOPASSWD: /home/<username>/usr/SpireCV/ZLM/MediaServer
```

> 注意：`MediaServer` 会加载 `ZLM/` 目录下的配置文件。虽然配置文件可以写在其他路径，但是修改 `config` 文件后才会生效。  


#### 仿真时可能遇到的其他错误

①错误：`resource not found px4`

此问题与 `~/.bashrc` 中 `source` 命令的顺序有关。`source devel/setup.bash` 会覆盖之前的 PX4 设置。

1. 检查当前环境里是否正确配置了 PX4 环境：

```bash
rospack list | grep "px4"
```

如果没有输出：

2. 确保 `px4` 的 `source` 在 `bashrc` 的最后：

```bash
source ~/px4/ROM/setup.bash
```

3. 在运行 `sh` 文件之前，执行：

```bash
source ~/.bashrc
```
之后，需要在~/.bashrc中配置以上两个部分的相关环境变量，我的版本仅供参考，请根据实际位置修改路径：  
```bash
# ------------------------- p450_experiments -------------------------

# gazebo model path
export GAZEBO_MODEL_PATH=/home/hanyx/gazebo_models:$GAZEBO_MODEL_PATH

# prometheus simulation
source /home/hanyx/prometheus_mavros/devel/setup.bash
source ~/usr/Prometheus/devel/setup.bash

# p450_experiments
source /home/hanyx/usr/p450_experiment/devel/setup.bash

export GAZEBO_PLUGIN_PATH=$GAZEBO_PLUGIN_PATH:~/usr/Prometheus/devel/lib
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:~/usr/Prometheus/Simulator/gazebo_simulator/gazebo_models/uav_models
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:~/usr/Prometheus/Simulator/gazebo_simulator/gazebo_models/ugv_models
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:~/usr/Prometheus/Simulator/gazebo_simulator/gazebo_models/sensor_models
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:~/usr/Prometheus/Simulator/gazebo_simulator/gazebo_models/scene_models
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:~/usr/Prometheus/Simulator/gazebo_simulator/gazebo_models/texture

# Prometheus gazebo
source ~/usr/Prometheus/devel/setup.bash

# prometheus px4
source ~/usr/prometheus_px4/Tools/setup_gazebo.bash ~/usr/prometheus_px4 ~/usr/prometheus_px4/build/amovlab_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/usr/prometheus_px4                                                                  
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/usr/prometheus_px4/Tools/sitl_gazebo
```
成功编译以上两部分后，进入Task6。

## Task6 P450仿真飞行
接下来，我们将在仿真环境中进行无人机的基本飞行，无人机飞行时主要由飞控和板载计算机两部分进行控制。
  
飞控是一套嵌入式实时控制系统，也是无人机的“小脑”，核心任务只有两件：
* ①姿态稳定：把机身实际姿态（ roll/pitch/yaw ）与期望姿态之间的误差，用 PID/ADRC 等算法转成电机转速指令，毫秒级闭环。
* ②轨迹跟踪：把 GPS/视觉/惯导给出的位置速度与任务规划值比较，生成新的期望姿态，再交给内环执行，外环秒级、内环毫秒级分层控制。

细节上可以参考PX4官方文档：【PX4文档】

飞控单元比较底层，我们是开发上层算法，一般不涉及对飞控内部代码的更改，事实上科研无人机生产商也不太更改内部的代码实现，但是飞控内部有一些可以配置的参数，之后用到了会统一介绍。目前你只需要把飞控当成一个**将飞行高级命令**（如目标的x, y, z，三个方向的速度vx, vy, vz等）翻译成**具体的飞行指令**（如每个电机的电压等）的黑盒即可。

板载计算机（Nvidia Jetson Orin NX等）是上层算法的主要载体，类似无人机的“大脑”，我们部署的算法都会运行在这台计算机上。机载计算机通常为一块开发板，计算能力较低，目前也有部分开发板具有GPU，能支持轻量化深度学习模型的部署。运行的算法需要被实现为ROS结点，读取ROS话题中的无人机传感器数据（相机图像、IMU等）与当前状态（里程计位置，GPS等），并进行一定的处理（例如目标追踪，地图构建），并生成无人机下一步的高级飞行指令，由飞控翻译为具体的控制。

### 无人机飞行模式基础
#### 位置模式
这一模式由遥控器+无人机飞控内部位置估计完成无人机的飞行，例如遥控器让飞机前进，通常是给飞控发送了一个当前目标前方的目标点，由飞控控制飞机将自身的位置与目标点位置对齐。在位置模式下无人机不响应板载计算机提供的命令，只由遥控器信号控制，通常用于进入Offboard模式之前的飞机初始位置调整。

#### 姿态模式
这一模式支持最大的控制权限，完全由遥控器命令控制飞机的姿态，但控制难度高，开发与实机飞行中都不常用。

#### Offboard模式
这一模式下，无人机根据板载计算机输出的飞行指令自主飞行，不响应遥控器指令，在算法准备好和当前状态下的无人机的配合后，进入这一模式。

> 注意：位置模式和Offboard模式下里程计位置发散将导致无人机失控，在实机飞行中具有一定危险。

#### Amov P450 仿真遥控器使用说明
通过以下命令启动遥控器：
【启动指令】
P450飞机遥控器为日本手，左侧摇杆控制前后左右，右侧摇杆控制油门和偏航角。仿真中我写的遥控器界面如图所示【遥控器界面】，通过WASD模拟左侧摇杆控制，方向键模拟右侧摇杆控制。上面5 6 7 8分别4个开关，功能如下：
* 通道5为两段开关，分别为上锁/解锁
* 通道6为三段开关，分别为初始（INIT）/ 位置模式（RC_POS_CONTROL）/ Offboard模式，INIT模式下，无人机不响应遥控器和开发板的指令；
* 通道7为两段开关，控制无人机急停，当通道7拨到低位时，无人机紧急锁桨，掉落到地面，用于在实际飞行中防止无人机失控对人的伤害。
* 通道8为两段开关，当通道8拨到低位时，无人机从当前位置自主降落。

开关的控制方式为：按数字键选择对应通道，被选中的通道上方会有*提示，按i键向上拨动一格，按k键向下拨动一格。

感兴趣也可以仔细查看uav_controller结点对应的代码，以及keyboard_control是如何通过话题接收uav_controller的杆量数据并转换为无人机命令发送给飞控的，相信这对你理解无人机控制有一定帮助。

#### 仿真飞行
①运行/Prometheus/Scripts/simulation/my_demos/gazebo.sh，启动仿真环境，可以在场景中看见无人机。  
②运行/Prometheus/Scripts/simulation/my_demos/remote.sh，启动遥控器控制结点。  
③将遥控器通道5拨到低位，解锁无人机，此时无人机螺旋桨应该能够转动起来，再将通道6拨到中位进入RC_POS_CONTROL，按方向键控制无人机偏航角和高度，按WASD控制无人机前后左右飞行。通道6拨到低位进入Offboard，则进入机上算法控制模式。

在仿真环境中可以自由飞行无人机以后，进入Task7.


## Task7 Ego-planner避障实验
Ego-planner是一个被广泛使用的无人机避障工作，实现细节建议参考其论文【论文链接】。  
没有其他问题以后，进入Task8.

## Task8 [Optional] 无人机目标跟踪实战
现在，你已经在仿真环境中拥有了一架可以自由开发的无人机，并且也尝试了如何在仿真中运行别人的算法，下面是时候去开发与实现自己的算法了。Task8是可选任务，如果你对于如何将一个算法放到无人机上仍觉得无从下手，强烈建议花时间完成此任务。  
本部分将逐步带领你实现一个简单的无人机自然语言跟踪算法，我们的目标是基于已有的目标跟踪算法【UVL-Track】，在仿真中实现如下功能：
1. 无人机起飞后悬停，等待用户从控制终端输入自然语言提示。
2. 基于当前摄像头拍摄到的图像，找到用户自然语言提示对应的目标。
3. 进入跟踪模式，当目标的位置发生移动时，无人机调整自身位置，对目标进行跟踪。  
这一算法作为我实习期间的练习项目，最终部署到了真机上进行试飞，请看VCR：[真机视频]

以下是将算法部署到仿真无人机中的详细步骤，请你根据步骤依次实现，参考答案位于p450_experiment/src/uvl-track-ros

### step1：配置UVL-Track环境
仓库地址：https://github.com/OpenSpaceAI/UVLTrack  
你需要在本地搭建UVL-Track的conda环境，实现输入mp4格式的视频，输出带检测框的检测结果，运行命令和测试用例可参考p450_experiment/src/uvl-track-ros/notes.txt  
运行测试中的命令，看到算法能够框出视频中的目标，表示环境配置成功。

### step2：设计结点功能与结点关系

### step3：

## Task9 Airsim仿真P450飞机
to be continued...
