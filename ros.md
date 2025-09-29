### rosbag 播放
rosbag play my_data.bag -r 1.5 --loop # r为速度倍率，--loop表示循环播放

### 编译通过，roslaunch可以正常补全，但是报错Cannot locate node of type
首先检查包名和结点名是否写错  
其次还有可能是CmakeLists.txt中没有声明这个包，通常在include_directories之前加上
```
catkin_package(
  CATKIN_DEPENDS roscpp rospy sensor_msgs std_msgs
  DEPENDS xxx
)

```
### Airsim如何修改Imu，相机图像等等的ros话题发布频率？
在airsim_node.launch中有对应的配置参数：
```
<param name="update_airsim_img_response_every_n_sec" type="double" value="0.05" /> 
<param name="update_airsim_control_every_n_sec" type="double" value="0.005" />
<param name="update_lidar_every_n_sec" type="double" value="0.01" />
```