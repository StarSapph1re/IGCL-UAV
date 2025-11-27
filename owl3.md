### 将机上代码配合厂家提供的闭源部分，在本地编译成功
1. sourcecode下gimbal controller复制到src中
2. 硬编码修复：/home/hanyx/usr/ros_ws/devel/share/controller_msgs/cmake/controller_msgsConfig.cmake
```
/home/visbot/VISIM_swarm/ros_ws -> /home/hanyx/usr/owl3/ros_ws
```
3. 厂家给的闭源部分文件不全，还需将无人机上/opt/ros/noetic/include/captain下的内容拷贝到/home/hanyx/usr/owl3/ros_ws/src/third_party/include/captain，然后修改/home/hanyx/usr/owl3/ros_ws/src/captain/mavros_controller/CMakeLists.txt
```
include_directories(
  ${PROJECT_SOURCE_DIR}/include
  SYSTEM
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${captain_INSTALL_PREFIX}/include/captain
  ${PCL_INCLUDE_DIRS}
  /home/hanyx/usr/owl3/ros_ws/src/third_party/include/captain  # 加上这一行
)
```
4. ros_ws/install/share/captain/launch/captain.launch simulation参数设置为true
5. 安装这个依赖：
```
sudo apt install libnanomsg-dev
```
6. 编译时要带厂家提供的install下的闭源包路径
```
catkin_make -Dcaptain_DIR=/home/hanyx/usr/ros_ws/install/share/captain/cmake -DPYTHON_EXECUTABLE=/home/hanyx/.conda/envs/airsim/bin/python
```
注意，编译一次以后会有catkin缓存，之后不需要再显式指定路径，但是如果rm -rf build了就需要再执行一次，否则会报错
```
Could not find a package configuration file provided by "captain" with any of the following names: captainConfig.cmake captain-config.cmake
```


### 编译mavros
1. 将厂家给的mavros拷贝到自己的src，不要直接在厂家的ros_ws中catkin_make，有可能破坏环境
2. 执行catkin_make

### 下载PX4 1.14.0
https://blog.csdn.net/weixin_42037083/article/details/137102858
clone 1.14 版本的代码
```
git clone --branch v1.14.0 --depth 1 https://github.com/PX4/PX4-Autopilot.git
```
下载其他依赖库
```
git submodule update --init --recursive
```
可能遇到问题curl 16 Error in the HTTP2 framing layer
```
git config --global http.version HTTP/1.1
```
再次克隆子模块，成功安装以后重置
```
git config --global http.version HTTP/2
```
编译固件
```
make px4_sitl_default gazebo
```

### 在本地运行机上结点
libSmartX1App.so放到ros_ws/install/lib下

### 仿真开发流程
1. /home/hanyx/usr/owl3/ros_ws/src/captain-plugin/config/captain_ext.json 中修改
```
"plugin_list":[/home/hanyx/usr/owl3/ros_ws/devel/lib/libcaptain_extend.so]
```
开发时编译这个以后，启动captain结点，他就能链接上自定义的Task类

2. /home/hanyx/usr/owl3/ros_ws/src/mavros/mavros/launch/node.launch 修改
```
<node pkg="mavros" type="mavros_node" name="mavros" 这里name从"drone_$(arg drone_id)_mavros"改为"mavros"
```

### 验证通讯过程
1. 启动Airsim，运行仿真
2. 运行mavros.sh (这里面包含mavros和PX4)
3. 运行controller.sh
4. 打开终端输入以下内容
```
source ~/usr/owl3/ros_ws/devel/setup.bash
rostopic pub -1 /control controller_msgs/control -f /home/hanyx/usr/owl3/ros_ws/src/captain/my_cmd/my_takeoff.msg
```
观察到无人机起飞即整个通讯过程没有问题

### 根据文档配置PX4的参数

### 下载QGC地面站
CAUSION: 这些依赖版本可能导致问题，由于配置参数不一定要经过QGC，不建议通过QGC去配置

链接：https://docs.qgroundcontrol.com/master/en/qgc-user-guide/getting_started/download_and_install.html

运行时依赖问题：
version `GLIBC_2.35' not found
参考解决方案：https://blog.csdn.net/m0_74456433/article/details/141960591

version `GLIBCXX_3.4.29' not found
参考解决方案：https://blog.csdn.net/weixin_39379635/article/details/129159713

undefined symbol: wl_proxy_marshal_flags
升级一下库：
```
sudo apt update
sudo apt install --only-upgrade libwayland-client0 libwayland-server0
```

### 注意，以上步骤可能导致原本编译通过的ros_ws再次编译出现问题，解决方法如下
/usr/include/boost/thread/pthread/thread_data.hpp:60:5: error: missing binary operator before token "("
   60 | #if PTHREAD_STACK_MIN > 0
      |     ^~~~~~~~~~~~~~~~~
找到这个文件，这一行改成 #ifdef PTHREAD_STACK_MIN

没有规则可制作目标“/usr/lib/x86_64-linux-gnu/libdl.so”
类似这种报错，在这个目录下找一个libdl.so.*，然后建立软连接即可

### 配置相关PX4参数
启动Airsim，PX4 + QGC
这里需要关闭原本的mavlink结点，否则启动QGC时会因为多个mavlink实例冲突，表现为QGC显示Comms Lost
QGC中设置：
SYS_HAS_MAG disabled
SYS_HAS_GPS disabled
试了好像在QGC的修改需要重启生效，但是仿真中QGC无法直接重启PX4，因此还是直接在PX4控制台更改：
```
param set SYS_HAS_MAG 0
param set SYS_HAS_GPS 0
param set EKF2_MAG_CHECK 0
param set EKF2_MAG_TYPE None
param set EKF2_EV_CTRL 11
param set EKF2_HGT_REF vision
param set EKF2_EV_DELAY 0ms
param save
```
重启飞控，使用param show确认参数是否被成功设置和保存，保存后，再次启动时参数不用重新设置

### 创建一个自定义Task
1. /home/hanyx/usr/owl3/ros_ws/src/captain-plugin/src/task_plugin 写好对应的cpp和hpp
2. /home/hanyx/usr/owl3/ros_ws/src/captain-plugin/config/captain_ext_sim.json 中为其赋予编号(编号需要>128, <218)
3. /home/hanyx/usr/ros_ws/install/share/captain/launch/captain_sim.launch 中，确认captain_ext_path指向json文件路径
4. /home/hanyx/usr/owl3/ros_ws/src/captain-plugin/cmake/build_lib.cmake 中将对应的cpp文件添加上（已用通配符收集所有文件），然后编译
5. /home/hanyx/usr/owl3/ros_ws/src/captain/my_cmd 下定义对应的启动msg，其中id为任务编号
6. 启动方式：rostopic pub -1 /control controller_msgs/control -f /home/hanyx/usr/owl3/ros_ws/src/captain/my_cmd/xxx.msg

### 自动启动逻辑
1. /home/hanyx/usr/ros_ws/install/share/captain/launch/captain_sim.launch中mission_name决定了captain启动时自动执行的mission
2. 去对应的mission中（一般采用ExtGeneralMission）/home/hanyx/usr/owl3/ros_ws/src/captain-plugin/src/mission_plugin/generalMission.cpp 修改对应push_back的任务名称，来实现按顺序执行这里面的任务

### 定义消息功能包
这里我是为了在Task执行前都通过一个ros结点统一进行确认后再进行执行，首先创建一个统一放置消息的功能包
```
cd ~/usr/owl3/ros_ws/src
catkin_create_pkg owl_msgs std_msgs message_generation message_runtime
cd owl_msgs
mkdir msg
```
在msg文件夹下新建一个OwlCmd.msg，只发送一个信号值
```
int32 signal
```
要编译这个消息类型，首先在CMakeLists.txt中增加如下内容：
```
add_message_files(
  FILES
  OwlCmd.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

# 这两个注释必须打开
catkin_package(
   INCLUDE_DIRS include
#  LIBRARIES owl_msgs
   CATKIN_DEPENDS std_msgs  # 只需要std_msgs
#  DEPENDS system_lib
)

install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
```
编译完成后，在其他的功能包中使用这个包中的消息，首先要加入依赖，在CMakeLists.txt中
```
find_package(catkin REQUIRED COMPONENTS
  ...
  owl_msgs
)

catkin_package(
  CATKIN_DEPENDS 
  ...
  owl_msgs
)
```
然后在package.xml中加上构建依赖和运行依赖：
```
<build_depend>owl_msgs</build_depend>
<run_depend>owl_msgs</run_depend>
```

在代码中，直接include头文件即可（仅限在同一个src目录下）
```
#include "owl_msgs/OwlCmd.h"
from owl_msgs.msg import OwlCmd
```
另外，为了防止硬编码判断，以及纯数字无语义带来的可读性问题，可以在owl_msgs/include/owl_msgs下定义SignalConstraints.h 里面设置一个自己的命名空间
```
#pragma once
namespace owl_msgs::signal
{
    static constexpr int CONFIRM = 0;
    static constexpr int STOP    = 1;
}
```
然后在其他功能包使用时，带命名空间去判断与赋值即可
```
#include <owl_msgs/SignalConstants.h>
OwlCmd cmd;
cmd.signal = owl_msgs::signal::CONFIRM;
```
这一步用.msg文件内的枚举也可以，有点多此一举。

### 定义功能包
为了在真机飞行时用一个commander结点统一为飞机将要执行的任务发送控制指令，再定义一个real功能包，方便在真机测试时使用
```
cd ~/usr/owl3/ros_ws/src
catkin_create_pkg real roscpp rospy std_msgs message_generation message_runtime
mkdir scripts

```
CMakeLists.txt中设置python文件的执行路径
```
set(PYTHON_EXECUTABLE /home/hanyx/.conda/envs/airsim/bin/python)
```
添加C++和Python实现的结点时，需要分别在CMakeLists.txt中加上
```
add_executable(xxx src/xxx.cpp)

target_link_libraries(xxx
  ${catkin_LIBRARIES}  
)

catkin_install_python(PROGRAMS scripts/yyy.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```

### 遥控器各通道对应数据
注：摇杆中位都是1500
|         | 上         | 下         | 左         | 右         |
| ------- | --------- | --------- | --------- | --------- |
| **左摇杆** | 通道2(2000) | 通道2(1000) | 通道3(1000) | 通道3(2000) |
| **右摇杆** | 通道1(1000) | 通道1(2000) | 通道0(1000) | 通道0(2000) |

|              | 上    | 中    | 下    |
| ------------ | ---- | ---- | ---- |
| **SWA(通道5)** | 1000 | /    | 2000 |
| **SWB(通道6)** | 1000 | /    | 2000 |
| **SWC(通道4)** | 1000 | 1500 | 2000 |
| **SWD(无映射)** | /    | /    | /    |

### 坐标轴
发送给waypoint Task: x 前 y 左 z 上  
读取话题/mavros/local_position/pose: x 右 y 前 z 上

### 注意点
1. 自定义Task中也可以收发ros话题，示例如下：
```C++
void WaypointTask::initRos(){
    // 获取句柄
    if (!ros::isInitialized()) {
        int argc = 0;
        char **argv = nullptr;
        ros::init(argc, argv, "waypoint_node", ros::init_options::NoSigintHandler);
    }
    nh_ = new ros::NodeHandle("~");

    // 【订阅示例】
    string waypoint_topic = "/captain/waypoint/goal";
    goal_sub_ = nh_->subscribe(waypoint_topic, 1, &WaypointTask::goalCallback, this);

    // 【发布示例】
    string goal_topic = "/captain/waypoint/goal_reach";
    goal_reach_pub_ = nh_->advertise<sim::GoalReach>(goal_topic, 1);
}

void WaypointTask::goalCallback(const sim::PoseYaw::ConstPtr& msg){
    // 回调函数处理逻辑
}
```
2. 创建轨迹
出现调用轨迹的initPrimitive导致报错，captain结点挂掉
```C++
trajectory_ = data->CreateTraj("ExtSetpointTrajectory");
```
这个名字不是随便命名的，Trajectory基类会去找traj_plugin中的对应实现，目前提供的示例实现为ExtSetpointTrajectory, ExtShapeTrajectory, ExtPolynomialTrajectory
3. 主循环
Task的主循环中，通常使用Task状态作为判断继续执行的条件，但是当循环中仅有ros::spinOnce(), usleep(10000)等【似乎不进行操作】的语句时，会出现任务启动后立即退出的情况（循环不会一直执行，即使没有手动赋值state_ = TASK_DONE），例如：
```
while(state_ != TASK_DONE){
    ros::spinOnce();
    usleep(10000);  // freq: 100Hz
  }
```
如果给循环中加打印语句，发现又可以正常进行，因此对于这种特殊情况，请使用while(true)
4. 真机上编译找不到新文件
偶尔发生，往captain-plugins/src里加入新文件以后，编译始终不管这个文件。  
原因：camke缓存没清理，估计和GLOB自动扫描有关，缓存以后不再进行扫描，需要彻底清除缓存（catkin_make clean没用）：
```
cd /home/visbot/ros_ws
rm -rf build/ devel/
```
5. 真机飞行时用tail查看发现print语句输出有延迟
在Task中手动设置不使用stdout的缓冲区（在所有print之前调用即可）
```
setvbuf(stdout, nullptr, _IONBF, 0);
```

#### 奇怪的问题
python实现的结点，运行报错：ModuleNotFoundError: No module named 'numpy.core._multiarray_umath' 
在终端运行对应的python解释器，打印sys.path，发现/usr/lib/python3/dist-packages，而source devel/setup.bash后，再打印，发现sys.path中/usr/lib/python3/dist-packages，导致import时优先找到了系统装的numpy，目前不知道如何避免这一点，只能在python结点的开头加上：
```python
#!/home/hanyx/.conda/envs/airsim/bin/python
import sys
sys.path = [p for p in sys.path if not p.startswith('/usr/lib')]  # 去掉系统路径
```
