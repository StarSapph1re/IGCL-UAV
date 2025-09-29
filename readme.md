# IGCL-UAV
本仓库记录配置Amov P450无人机的步骤

## 环境准备
系统：Ubuntu 20.04，安装ROS-noetic  

## 代码
p450机上代码 p450_experiment  
p450-px4仿真代码 Prometheus

## 使用fishros工具一键安装ROS
### 更换系统源
```
wget http://fishros.com/install -O fishros && . fishros
```
执行以上命令，并依次选择5 2 1
如果出现证书、数字签名无效等问题，执行以下命令
```
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```

### 安装ros-noetic
```
wget http://fishros.com/install -O fishros && . fishros
```
执行以上命令，并依次选择1 2 3 1
### 验证安装
打开一个终端，启动ros核心
```
roscore
```
再打开一个终端，运行海龟仿真结点
```
rosrun turtlesim turtlesim_node
```
如有正常画面显示，则ros安装成功

