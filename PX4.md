### 融合vision pose
必须设置PX4中的参数EKF2_AID_MASK，并且发布到话题/mavros/vision_pose/pose给PX4融合
可以在终端执行：
param set EKF2_EV_CTRL 15
param save
save完了以后，下次启动也是这个值

如果PX4这样启动：make px4_sitl none_iris  
可以在/home/hanyx/usr/PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes/下找到10016_none_iris  
，即启动脚本，可以在里面设置启动参数，其他名称与设置同理

### 相关命令
listener vehicle_visual_odometry 查看视觉里程计状况

```
TOPIC: vehicle_visual_odometry
 vehicle_visual_odometry
    timestamp: 1757728444868867 (0.009006 seconds ago)
    timestamp_sample: 1757728444868867 (0 us before timestamp)
    position: [4.32368, 2.46479, -1.71765]
    q: [0.70697, 0.00669, -0.00373, 0.70720] (Roll: 0.2 deg, Pitch: -0.8 deg, Yaw: 90.0 deg)
    velocity: [nan, nan, nan]
    angular_velocity: [nan, nan, nan]
    position_variance: [0.00000, 0.00000, 0.00000]
    orientation_variance: [0.00000, 0.00000, 0.00000]
    velocity_variance: [nan, nan, nan]
    pose_frame: 1
    velocity_frame: 0
    reset_counter: 0
    quality: 0
```